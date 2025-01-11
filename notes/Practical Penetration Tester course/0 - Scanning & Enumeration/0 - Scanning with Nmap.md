# Detecting hosts with arp-scan
There is a simple tool in kali linux that can be used to understand which hosts are connected in our network.

It is `arp-scan`. We can use it in this way:
```bash
arp-scan -l
```
- ![[Pasted image 20250111164534.png]]

# Detecting hosts with netdiscover

Another tool is `netdiscover`. Here we need to know the `CIDR` of the network:
```bash
netdiscover -r IP/MASK
```
- `-r` stands for range
- ![[Pasted image 20250111164744.png]]
- ![[Pasted image 20250111164759.png]]



# Scan with Nmap
The idea of `nmap` is that it starts a `SYN` connection with the host on all the possible ports in order to understand which ones are open.

There are different flags we can use:
- `-sS` stands for **stealthy**, in this case nmap sends a `SYN` request and then sends a `RST` so the connection is never set, no one can see it
- `-T4` speeds up the  scanning
- `-p-` is used to scan all ports from `0` to `65535`
- `-A` stands for everything, so it does all the checks (OS, versions and so on)
- ...

If we don't use `-p-` it scans the first 1000 ports. We can also define the ports we want `-p 80,443,53`

You can see al the possible flags with 
```bash
nmap -help
```


*For example we can run*:
```bash
nmap -T4 -p- -A IP_ADDRESS
```
- ![[Pasted image 20250111165516.png]]


One of the most useful is:
```bash
sudo nmap -p- -sC -sV -T4 -O IP_ADDRESS
```
