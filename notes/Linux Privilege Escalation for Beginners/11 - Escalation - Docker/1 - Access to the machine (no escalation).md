# NMAP scan

First of all we need to execute a NMAP scan:
```bash
nmap -p- -A -T4 HOST_IP
```
- ![[Pasted image 20250118164316.png]]


# Enumerating the website
We can see that there is an `apache` sever so we start to enumerate the website:
- on port `8081` that is a `Node,js Express Framework`
- on port `31331` that is a `apache` server


So we visit both:
- ![[Pasted image 20250118164510.png]]
- ![[Pasted image 20250118164521.png]]


We use `Burp suite` to analyze the website:
- ![[Pasted image 20250118164610.png]]

But nothing interesting



So we try to access to `robots.txt` if it exists on port `31331`:
- ![[Pasted image 20250118164705.png]]

We found this interesting file and we look at it:
- ![[Pasted image 20250118164739.png]]

So we start to enumerate the information.


On `/partners.html` we can see a login page (port `31331`):
- ![[Pasted image 20250118164813.png]]

We look at the source code and we can notice a `api.js`:
- ![[Pasted image 20250118164907.png]]

We click on it and we start to look for info:
- ![[Pasted image 20250118165107.png]]
- it does a ping to the `window.location.hostname`



So basically we try to see what it does, we know that the IP of the API is `10.10.186.101` so we search for:
```bash
http://10.10.186.101:8081/ping?ip=127.0.0.1
```
- in `localhost` just to see
- ![[Pasted image 20250118165305.png]]

So we are doing a ping and so `code execution`...


So we can try to inject using the backticks:
- this because everytime a backtick is used what inside is executed first
- so if we have ping \`ls\` the `ls` is executed before the ping

So we try to inject:
```bash
http://10.10.186.101:8081/ping?ip= `ls`
```
- ![[Pasted image 20250118165822.png]]
- we gain a result


So we can try to `cat` this file:
```
http://10.10.186.101:8081/ping?ip= `cat utech.db.sqlite`
```
- ![[Pasted image 20250118165932.png]]

So we copy the hash of `r00t` and we go on crack station [https://crackstation.net/](https://crackstation.net/)
- ![[Pasted image 20250118170020.png]]
We cracked the `r00t` hash that is `n100906` password.



# ssh access on the victim

So now we can try to access usint `r00t:n100906`:
```bash
ssh r00t@10.10.186.101 (TryHackMe IP)
```
- ![[Pasted image 20250118170227.png]]

We got a low level access!


Now we need to escalate.

