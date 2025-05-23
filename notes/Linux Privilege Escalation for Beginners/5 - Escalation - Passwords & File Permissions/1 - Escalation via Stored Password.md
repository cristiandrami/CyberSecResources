<mark style="background: #BBFABBA6;">The first thing we need to do, to see if there are passwords on the terminal is:</mark>
```bash
history
```
- ![[Pasted image 20250116105036.png]]

We can also do:
```bash
cat .bash_history
```



So now we know there is a password `password123` used for `mysql`


So we try to use the password `password123`:
```bash
su root
```
- and we are root
- ![[Pasted image 20250116105241.png]]


To speed up the search we can use:
```bash
history | grep pass
```
- ![[Pasted image 20250116110953.png]]


# Search for passw with Payload all the things

Useful strategy and payloads can be found here [https://swisskyrepo.github.io/PayloadsAllTheThings/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation/](https://swisskyrepo.github.io/PayloadsAllTheThings/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation/)


From it we can use for example:
```bash
find . -type f -exec grep -i -I "PASSWORD" {} /dev/null \;
```
- it will give us all the files in which there is the word `password` in it
- ![[Pasted image 20250116105713.png]]



# LinPEAS for hunting passwords

When we run:
```bash
./linpeas.sh
```

It will show us a lot of information and will show us all the files/variables/history in which we have the word `passw`
- ![[Pasted image 20250116105929.png]]


# Searching in specific files

In our case we cna search into a file called `myvpn.ovpn`

In it we can see:
- ![[Pasted image 20250116110614.png]]


So what we can access to that file `/etc/openvpn/auth.txt`
- ![[Pasted image 20250116110739.png]]

But this user doesn't exists anymore


# STEP
- [ ] In command prompt type: **cat ~/.bash_history | grep -i passw**

