The idea is to create a symlink (an alias that points to another file) in order to point to a privileged binary file in order to force the execution of the symlink and so internally the execution of the privileged file.

*For example: we have a file with `SUID` that accesses to a file during the execution, we can sobstitute it with a symlink in order to point to a file with malicious code.*


# Nginx example
In this case the vulnerability leverages on the permissions of the logs created by `nginx` 

In this specific case we have to login in the machine (linux room of the course in tryhackme) as `www-data`:
- ![[Pasted image 20250117141318.png]]

# automatic scan

At this point we run `linux-exploit-suggester`:
```bash
./linux-exploit-suggester.sh
```
- ![[Pasted image 20250117141432.png]]
- here we can see that we have a possible privilege escalation

The exploit of this vulnerability can be found on this page:
- [https://legalhackers.com/advisories/Nginx-Exploit-Deb-Root-PrivEsc-CVE-2016-1247.html](https://legalhackers.com/advisories/Nginx-Exploit-Deb-Root-PrivEsc-CVE-2016-1247.html)

# manual scan
We can perform a manual scan in order to see if it is vulnerable:
```bash
dpkg -l | grep nginx
```
- ![[Pasted image 20250117141739.png]]
- the version `1.6.2-5` is vulnerable to this attack

To exploit it we need that `sudo` has the `SUID` set.
# find for files with `SUID` set

As always we can search for:
```bash
find / -type f -perm -04000 -ls 2>/dev/null
```
- ![[Pasted image 20250117142007.png]]
- so `sudo` has `SUID` set and `nginx` version is vulnerable

We can do the attack.

Now we can notice that we have access to the `nginx` log directory:
```bash
ls -la /var/log/nginx
```
- ![[Pasted image 20250117142139.png]]


So the idea is to overwrite the log file with a `symlink`. In order to force `nginx` to write on the log privileged files.

# Symlink creation with exploit
We can easily do the symlink creation using the exploit:
- [https://legalhackers.com/advisories/Nginx-Exploit-Deb-Root-PrivEsc-CVE-2016-1247.html](https://legalhackers.com/advisories/Nginx-Exploit-Deb-Root-PrivEsc-CVE-2016-1247.html)

Running:
```bash
./nginxed-root.sh /var/log/nginx/error.log
```
- ![[Pasted image 20250117142751.png]]

So it creates a `.so` library containing a shell and injecting it into the log file as a `symlink`.


Each time the server `nginx` is restarted it will invoke this shared library, because it will use the `error.log` that is a `symlink` to the malicious library.

# restart the nginx to see the result

Now we need the restart of the server `nginx` to gain the root shell.

So for testing purposes we need to login as root in the machine and restart it manually:
- ![[Pasted image 20250117143146.png]]


```bash
invoke-rc.d nginx rotate >/dev/null 2>&1
```
- this restarts the server


Now we go back to the terminal in which we run the exploit and we can see:
- ![[Pasted image 20250117143319.png]]

<mark style="background: #BBFABBA6;">We gain the root access!</mark>

