# ED
Ericsson password decoder

![alt text](https://github.com/billchaison/ED/blob/master/er.png)

Ericsson network equipment stores passwords using a simple bitwise encoding scheme in configuration files.  The following shell script can be used to decode them.  Create a bash script named `ericsson-pwdecoder.sh` as follows:

```bash
#!/bin/bash

if [ "$#" -ne 1 ]
then
   echo "You must supply a hex-encoded string."
   echo "(e.g.) ericsson-pwdecoder.sh 798909896149095131"
   exit 1
fi

EPW=${1,,}

if [ ${#EPW} -ge 2 ]
then
   if ! (( ${#EPW} % 2 ))
   then
      if [[ $EPW =~ ^[0-9a-f]+$ ]]
      then
         BYTES=( $(echo $EPW | fold -2) )
      else
         echo "You must supply a hex-encoded string."
         echo "(e.g.) ericsson-pwdecoder.sh 798909896149095131"
         exit 1
      fi
   else
      echo "You must supply a hex-encoded string."
      echo "(e.g.) ericsson-pwdecoder.sh 798909896149095131"
      exit 1
   fi
else
   echo "You must supply a hex-encoded string."
   echo "(e.g.) ericsson-pwdecoder.sh 798909896149095131"
   exit 1
fi

declare -A XLAT
XLAT=( [0]=0 [1]=8 [2]=4 [3]=c [4]=2 [5]=a [6]=6 [7]=e [8]=1 [9]=9 [a]=5 [b]=d [c]=3 [d]=b [e]=7 [f]=f )
PT=""

for B in "${BYTES[@]}"
do
   N=( $(echo $B | fold -1 ) )
   X="$((0xf ^ 0x${N[1]}))"
   Y=$(printf "%x\n" $X)
   Z=${XLAT[$Y]}
   PT+=$Z
   X="$((0xf ^ 0x${N[0]}))"
   Y=$(printf "%x\n" $X)
   Z=${XLAT[$Y]}
   PT+=$Z
done

echo $PT | xxd -r -p
echo
```

Execute the bash script supplying the hex-encoded password string as the argument, e.g. `ericsson-pwdecoder.sh 798909896149095131`

The decoded password will be displayed:<br />
![alt text](https://github.com/billchaison/ED/blob/master/er2.png)
