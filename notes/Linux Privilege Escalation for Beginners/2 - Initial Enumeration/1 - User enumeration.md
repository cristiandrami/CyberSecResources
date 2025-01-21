We need to do a user enumeration in order to understand who we are, what we can do and which are our permissions on the system.

**==To discover who we are:==**
```bash
whoami
```
- ![[Pasted image 20250115164728.png]]

**==To see in which group we are:==**
```bash
id
```
- ![[Pasted image 20250115164751.png]]
- ==**It is very important, maybe we are in sudoers group**==



# Which command we can run as sudo?
It is very useful to understand which commeand we can use as `sudo`.

This because using this website:
- [https://gtfobins.github.io/](https://gtfobins.github.io/)
- we can easily see if the command we can use can be used to escalate to root

<mark style="background: #BBFABBA6;">To enumerate the commands we can use as sudo we can do:</mark>
```bash
sudo -l
```
- ![[Pasted image 20250115165023.png]]
- All can be executed with no password, veery useful because we don't require any password at all


# /etc/passwd
**==This is a useful file we can look in order to get info about the users in the system and about their permissions:==**
```bash
cat /etc/passwd
```
- ![[Pasted image 20250115165300.png]]


We can also retrieve all the users line by line using:
```bash
cat /etc/passwd | cut -d : -f 1
```
- ![[Pasted image 20250115165239.png]]



# /etc/shadow
If we have access to `/etc/passwd` and to `/etc/shadow` then we have access to all passwords in the system so always check it:
```bash
cat /etc/shadow
```
- ![[Pasted image 20250115165433.png]]


# In general
**==We have to check our permissions and check to which file (maybe sensitive we can access)==**


<mark style="background: #BBFABBA6;">Another important thing is to see the last executed commands:</mark>
```bash
history
```
- ![[Pasted image 20250115165559.png]]


