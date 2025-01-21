Now we are in the machine and we want to escalate to `root`


# System information

First of all we want to understand which is the name of the system:
```bash
hostname
```
- ![[Pasted image 20250115163607.png]]


For more information we need to run:
```bash
uname -a
```
- ![[Pasted image 20250115163626.png]]
- **==This is very useful to search for any possible exploit==**


Here we will copy the `debian 2.6.32-5-amd64`.


Other useful information can be retrieved by:
```bash
cat /proc/version

cat /etc/issue
```
- ![[Pasted image 20250115163900.png]]

# CPU info
**==To get info about the CPU we will use:==**
```bahs
lscpu
```
- ![[Pasted image 20250115164152.png]]

It is useful because some exploit can be done only on certain cpus.


# Listing processes
**==Another important information is the processes that runs on the system:==**
```bash
ps aux
```
- ![[Pasted image 20250115164248.png]]

ALWAYS SEARCH FOR:
- `/usr/sbin/cron`
- `apache` and other http servers 
- file sharing services


**==To see all the pocesses run as root we can do:==**
```bash
ps aux | grep root
```

