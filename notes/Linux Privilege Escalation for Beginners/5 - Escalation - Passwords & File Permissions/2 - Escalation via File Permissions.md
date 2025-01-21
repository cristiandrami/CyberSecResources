In this case what we want to do is to access to files that are run or accessed as root and try to modify them to gain the privilege.

<mark style="background: #BBFABBA6;">For example to see our permissions on files we can do:</mark>
```bash
ls -la file
```
- ![[Pasted image 20250116120343.png]]
- in this case we can read both file but not write on them (we have to see the last 2 `-`)

If we had the possibility to change the `/etc/passwd` file we had the possibility to insert a password for root and access with it.
We just need to change the `x` after the username:
- ![[Pasted image 20250116134904.png]]

Or we can change the group of our user.


# Shadow and Passwd files (to crack password)
One thing we can do is to copy and paste the `/etc/passwd` and the `/etc/shadow` files into 2 new files:
- ![[Pasted image 20250116135500.png]]
- ![[Pasted image 20250116135510.png]]

## Unshadow tool
<mark style="background: #BBFABBA6;">At this point with a tool called `unshadow` we can retrieve the hash of the password we want to crack:</mark>
```bash
unshadow passwd_created shadow_created
```
- this files are the files we copy and paste before
- ![[Pasted image 20250116135655.png]]

We can see that it changed the `x` in the `passwd` with the hash contained in `shadow`.


At this point we copy the unshadowed file (removing the users we don't want):
- ![[Pasted image 20250116135806.png]]


So we copy the remaining strings into a file `credentials.txt` (TCM and root here) in order to use hashcat:
```bash
hashcat -m 1800 credentials.txt rockyou.txt -O
```
- ![[Pasted image 20250116140259.png]]
- it cracked the password `password123`


We can look at the mode we need to use from this website [https://hashcat.net/wiki/doku.php?id=example_hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)
- ![[Pasted image 20250116140025.png]]
- in our case it seems `1800` since it starts with `$6$`



