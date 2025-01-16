This is very important to understand the structure of the network and to see which ports are exposed to the extern and open.

To understand our network adapter IP and othe info we can run:
```bash
ifconfig
```
- or `ip a` that is the new ifconfig
- ![[Pasted image 20250115165852.png]]

To see if our packets are sent to another network we can use:
```bash
ip route
```
- ![[Pasted image 20250115165954.png]]

To see al the neighbors and which of them are reachable from our machine we can do:
```bash
ip neigh
```
- ![[Pasted image 20250115170307.png]]


# Open ports on our host

To see all the open ports on our host we can do:
```bash
netstat -ano
```
- ![[Pasted image 20250115170359.png]]
Here we can also see whic machine are in connection with our:
- ![[Pasted image 20250115170435.png]]


Important info are which connection are open in local host:
- ![[Pasted image 20250115170519.png]]
- WHAT WE ARE DOING IN UDP ON THE PORT 961?

