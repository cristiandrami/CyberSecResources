A GPO with overly permissive ACL can be abused fro multiple attacks.
- ![[Pasted image 20261119151400.png|597]]

We could do:
- EvilGPO  (immediate schedule task) NOT IN THIS COURSE
- Add a lcoal admin
- Modify group policy


# GPOddity
This combines NTLM relaying and modification of group policy container.

By relaying credentials of a user who has WriteDACL on GPO, we can modify the path (gPCFileSysPath) of the group policy template (that has as default SYSVOL).
- so we could change the group policy at all by loading malicous template from a location we can control


With Powerview and:
- `Get-DomainGPO` we can see that eaach GPO has a gpcfilesyspath:
	- ![[Pasted image 20261119151855.png|585]]

So the idea is to change this path to force the reading of a malicious policy.
- Very noisy, there is no direct detection but it is noisy.


To do it we:
1. Upload a malicious LNK on a share
2. A user executes LNK to trigger the authentication
3. We relay the authentication and delegation GPO write permissions of the user (send to DC)
4. We use the GPOddity to point the gPCFileSysPath to SMB server hosting our malicious GP  Tempalte (send to DC)
5. The domain controller updates the GPO after executing gpupdate
6. We fetch the new malicious GPT

![[Pasted image 20261119152400.png|572]]



We can see who has the permissions on a GPO using bloodhound:
- ![[Pasted image 20261119152626.png]]

And on who it is applied:
- ![[Pasted image 20261119152752.png|536]]