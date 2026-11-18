It is very interesting to discover info about users.

## Local Admin Access check
To find all the machines on which **current user** user has local admin access we can do:
- `Find-localAdminAccess -verbose`

> This function queries the Domain controller of the current or provided domain for a list of computer (Get-NetComputer) and then use multi-thread `Invoke-CheckLocalAdminAccess` on each one of these machines
> 
> ![[Pasted image 20261117112325.png|283]]

This can be done also with tools like WMI and Poweshell remoting. It's useful if ports (RPC and SMB) used by `Find-localAdminAccess` are blocked.


`Find-LocalAdminAccess` can generate significant network traffic and authentication events, especially when querying many domain computers concurrently. 
WMI and PowerShell Remoting provide alternative methods for checking local administrative privileges, particularly when SMB or RPC connectivity is restricted. 

However, these methods are not inherently stealthier, as they can also generate security events and endpoint telemetry.

See the scripts:
- `Find-WMILocalAdminAccess.ps1`
- `Find-PSRemotingLocalAdminAccess.ps1`

## Enumerate computers with an active domain admin session

We can find computers where a domain admin (or specified group/user) jas sessions:
- `Find-DomainUserLocation -verbose`
- `Find-DomainUserLocation -userGroupIdentity "RDPUsers"`

>This function queries the DCof the current or provided domain for members of the given froup (Domain Admins by defaukt) using `Get-DomainGroupMember`, gets a list of compputers (`Get-DomainComputer`) and list sessions and logged on users (`Get-NetSession/Get-NetLoggedon`) from each machine.
>
>![[Pasted image 20261117113420.png|395]]

From windows server 2019 and greater you need admin privilege to list sessions.
- I you can use it you are a local admin


## Domain in which current is admin and there is domain admin session
To find computers where a domain admin session is available and the current user has admin acce we can do.
- `Find-DomainUserLocation -CheckAccess`

To find computers (File servers and distributed file servers) where a domain admin session is available we cna do:
- `Find-DomainUserLocation -Stealth`

## Stealth tool
We can list sessions on remote machine using [https://github.com/Leo4j/Invoke-SessionHunter] with:
- `Invoke-SessionHunter -Failsafe`

> It doesn't need admin access on remote machines. It uses Remote Registry and queries HKEY_USERS hive.

An opsec friendly command would be (to avoid connections to all the target machines by specyfing targets):
- `Invoke-SessionHunter -NoPortScan -Targets C:\AD\Tools\Servers.txt`
	- `servers.txt` is the list of computers in the domain
	- ![[Pasted image 20261117114609.png|455]]


To import it:
- `. C:\AD\Tools\Invoke-SessionHunter.ps1`