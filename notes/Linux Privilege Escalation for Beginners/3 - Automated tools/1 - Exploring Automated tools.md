# LinPEAS
We just need to execute:
```bahs
./linpeas.sh
```


On the legend we can see:
- ![[Pasted image 20250116102746.png]]

In this specific case it shows us the Kernel version in red, so we need to look at it:
- ![[Pasted image 20250116102846.png]]


Other things we can see are:
- ![[Pasted image 20250116103108.png]]
- ![[Pasted image 20250116103231.png]]


We have a lot of things we can do to try to do a priv escalation.


# Linux exploit suggester
To run it we can do:
```bash
./linux-exploit-suggester.sh
```


It takes so long to do, but it gives us the possible exploit we can use to escalate to root:
- ![[Pasted image 20250116103430.png]]

