The `SUID` is a mechanism that allows an user to execute a file or program with the privilege of the owner of the file.
- *For example the command `passwd` allows a normal user to modify his password (that are stored in a file of property of `root`)*

We can see if there is this mechanism on a file when we see a `s` in the output of `ls -l`:
- ![[Pasted image 20250116180016.png]]



# Permissions on files

When we look at the privileges on files using `ls -la` we obtian an output like:
```bash
-rw-rw-r--
```
- the first 3 are for the owner of the group (here `read and write`) 
- the second 3 are for the groups in the machine (here `read and write)
- the last 3 for all other users (here `read`)

*For example when we use `chmod +x file` we are modifying the 7th bit of all (that is the x)*

In the same way when we use:
```bash
chmod 777 file
```
- we are modifying the 7th bit of each type of user

This because we have `rwx` the `r` part is 4 bits, `w` part 2 bits and `x` part 1 bit
- the 7th bit is `x`



When we have the `SUID` mechanism of a file we will see in the `x` part a `s` (for owner and group, a `t` for all users)


# Find files with SUID set on them

**==To find all file with the permission== `s` ==we can use:==**
```bash
find / -perm -u=s -type f 2>/dev/null
```
- in this case we want all files of `root` with the `s` set (`SUID`) 
- `type f` means that we want files
- ![[Pasted image 20250116181029.png]]



<mark style="background: #BBFABBA6;">So basically we are searching all files that our user can run as root.</mark>


*For example we can see:*
- ![[Pasted image 20250116181107.png]]
- there is a `s` set


## GTFOBins
At this point we have the files we can execute as root and we search for them in `GTFOBins`:
- [https://gtfobins.github.io/](https://gtfobins.github.io/)

**Here we go on the `SUID` filter and search for the files we have access to in the section `SUID`.**





# Challenge
Do this machine by your own:
- [https://tryhackme.com/r/room/vulnversity](https://tryhackme.com/r/room/vulnversity)

