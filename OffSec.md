
> [!important] Try Harder
At OffSec, we encourage learners to adopt the _Try Harder_ mindset. To better understand this mindset, let's quickly consider two potential perspectives in a moment of "failure."
> 
> 1. If my attack or defense fails, it represents a truth about my current skills/processes/configurations/approach as much as it is a truth about the system.
> 2. If my attack or defense fails, this allows me to learn something new, change my approach, and do something differently.

# Proofs

## Linux

```
echo " ";echo "uname -a:";uname -a;echo " ";echo "hostname:";hostname;echo " ";echo "id";id;echo " ";echo "ifconfig:";/sbin/ifconfig -a;echo " ";echo "proof:";cat /home/aero/local.txt 2>/dev/null; cat /Desktop/proof.txt 2>/dev/null;echo " "
```

## Windows

```
echo. & echo. & echo whoami: & whoami 2> nul & echo %username% 2> nul & echo. & echo Hostname: & hostname & echo. & ipconfig /all & echo. & echo proof.txt: &  type "C:\Documents and Settings\Administrator\Desktop\proof.txt"
```

