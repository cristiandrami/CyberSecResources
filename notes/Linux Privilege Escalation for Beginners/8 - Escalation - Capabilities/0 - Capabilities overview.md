Capabilities are used to divide the privileges there are usually given to `super user` into distinct units.

They can be enabled and disabled.

genrally we are passing from `SUID` to them because they are more secure.


# Hunting for capabilities

To find the avaliable capabilities in the system we need to run:
```bash
getcap -r / 2>/dev/null
```
- ![[Pasted image 20250117153831.png]]
- here we have 1 capability set
- `+ep` is for permit everything

So all capabilities are enabled.



# Resources
Linux Privilege Escalation using Capabilities - [https://www.hackingarticles.in/linux-privilege-escalation-using-capabilities/](https://www.hackingarticles.in/linux-privilege-escalation-using-capabilities/)

SUID vs Capabilities - [https://mn3m.info/posts/suid-vs-capabilities/](https://mn3m.info/posts/suid-vs-capabilities/)

Linux Capabilities Privilege Escalation - [https://medium.com/@int0x33/day-44-linux-capabilities-privilege-escalation-via-openssl-with-selinux-enabled-and-enforced-74d2bec02099](https://medium.com/@int0x33/day-44-linux-capabilities-privilege-escalation-via-openssl-with-selinux-enabled-and-enforced-74d2bec02099)

