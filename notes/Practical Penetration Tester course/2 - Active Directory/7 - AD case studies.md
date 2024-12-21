# Case study 1
We have this scenario:
- ![[Pasted image 20241221162524.png]]
- So we cannot use responder, we cannot use IPv6 attacks 

We could try to use smb relay attacks.
We cannot guess or crack the password because there is CyberArk but we could relay the password.


So we use a dmb relay attack:
- ![[Pasted image 20241221162801.png]]

So we have access to the machine and dump the SAM hashes.

We cna see that the admin account and tech accounts uses the same password (same hash).
-  Password reuse


So we try to crack the password:
- ![[Pasted image 20241221163123.png]]



So we login in all the machines and try to dump the hashes.

In one of them we found a hash that was able to grant the access to the domain controller:
- ![[Pasted image 20241221163257.png]]


More info here [https://tcm-sec.com/pentest-tales-001-you-spent-how-much-on-security/](https://tcm-sec.com/pentest-tales-001-you-spent-how-much-on-security)

In this case we have to:
- enable SMB signing (disable LLMNR)
- use least privilege
- use account tiering
- don't reuse passwords




# Case study 2
We have an AD with these security mechanisms:
- ![[Pasted image 20241221210754.png]]

So we cannot do any easy entry level attacks.

So what we can do is to scan all the web portarls of the environment.

We can see a iDRAC server and searching for the default credentials of an iDRRAC server we find `root:calvin`

So searching we can login in a tool:
- ![[Pasted image 20241221211019.png]]

So we have a password for a "local admin" so we try to access to every machine as local admin using `crackmapexec`:
- ![[Pasted image 20241221211145.png]]

At this point we are in a machine so we can dump all hashes using `secretsdump.py`:
- ![[Pasted image 20241221211244.png]]

We can see that the `admin` account has the same password of the `Administator` account.

So we start to spray on each machine the hash and dump data with `secretsdump.py`

We are able to capture a administrator password stored in clear text:
- ![[Pasted image 20241221211507.png]]

At this point we can access to the domain controller with these credentials.

## Mitigation
1. Don't use default credentials
2. Turn off WDigest
3. Don't give access to service accounts as Admin
4. Don't reuse passwords


More here [https://tcm-sec.com/pentest-tales-002-digging-deep](https://tcm-sec.com/pentest-tales-002-digging-deep)



# Case study 3
In this case we are able to use NTLM but we cannot escalate because each local admin has a unique password.

First of all we captured hashes:
- ![[Pasted image 20241221212548.png]]


Then we cracked it:
- ![[Pasted image 20241221212610.png]]


We noticed that a user we can have access to was not admin but has access to file shares.
- But the file shares was all account fileshares so we have access to everything


So we accessed to the public share and we found a setup guide:
- ![[Pasted image 20241221212817.png]]

On the MacBook setup procedure we found:
- ![[Pasted image 20241221212912.png]]

At this point we try to access with these credentials as admin in all machines and we get access:
- ![[Pasted image 20241221213333.png]]

So we execute `secretsdump` and we get on this machine:
- ![[Pasted image 20241221213437.png]]

It runs a service and we can find his password in clear text as we can seee on the screenshot.

So we can access with this credentials in the domain controller and we are in.

So, we get it because we enumerated, so ALWAYS ENUMERATE!!!

