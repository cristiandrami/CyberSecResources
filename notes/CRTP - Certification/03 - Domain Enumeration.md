For domain enumeration we can use the tools:
- ActiveDirectory Powershell module (MS signed and works even in Powershell CLM)
	- [https://github.com/samratashok/ADModule]
	- `Import-Module \AD\Tools\AdModule-master\Microsoft.ActiveDirectory.Management.dll`
	- `Import-Module \AD\Tools\AdModule-master\ActiveDirectory.psd1`
- BloodHoud (that is a C# and powershell collectors)
	- [https://github.com/specterops/bloodhound]
- PowerView 8powershell)
	- [https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1]
	- or `\AD\Tools\PowerView.ps1`
- Sharpview (C#)
	- [https://github.com/tevora-threat/SharpView]

==🔴In any case  we need to be safe so we run:==
1. `C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat`
2. `C:\AD\Tools\PowerView.ps1`
## Domain Enumeration 

### Domain infos
Get current Domain
- `Get-Domain` (powerview)
	- ![[Pasted image 20261116122319.png|386]]
- `Get-ADDomain` (Active directory module)
	- ![[Pasted image 20261116122452.png|575]]
	- Gives us also the SID
#### Domain objects
Get object of another domain
- `Get-Domain -Domain enterprise.local` (powerV)
- `Get-ADDomain -Identity enterprise.local` (AD module)

### Domain SID
Get domain SID for the current domain:
- `Get-DomainSID` (PwerV)
- `(GetADDOmain).DomainSID` (AD module)

### Domain policy
Get domain Policy for the current domain JUST IN powerview:
- `Get-DomainPolicyData` 
	- ![[Pasted image 20261116123134.png]]
- `(Get-DomainPolicyData).systemaccess`
Get domain Policy for another domain:
- `(Get-DomainPolicyData  -domain enterprise.local).systemaccess` 

### Domain controllers
Get domain controllers for current domain
- `Get-DomainController` (powerV)
	- ![[Pasted image 20261116151437.png]]
- `Get-ADDomainController` (AD module)

Get domain controllers for another domain
- `Get-DomainController -Domain enterprise.local` (powerV)
- `Get-ADDomainController -DomainName enterprise.local -Discover` (AD module)

### Domain users
Get domain Users in current domain
- `Get-DomainUser` (powerV)
	- We can filter using `Get-DomainUser | select userpricipalname` to get only the name of the users
	- For a single user `Get-DomainUser -Identity username`
- `Get-ADUser -Filter * -Properties *` (AD module)
	- For a single user `Get-ADUser -Idnetity username -Properties *` (AD module)`

Get domain Users Properties in current domain 
- `Get-DomainUser -identity username -Properties *` (powerV)
	- We can filter using `Get-DomainUser | select userpricipalname` to get only the name of the users
	- For a single user `Get-DomainUser -Identity username`
- `Get-ADUser -Filter * -Properties *` (AD module)
	- For a single user `Get-ADUser -Idnetity username -Properties *` (AD module)


Some interesting info can be found.

For example we can found the logoncount (the number of logon performed by an account).
==An account with a **high number in logoncount is very interesting** because probably it could be a job that performs some tasks==

To do it we can use:
- `Get-DomainUser | select samaccountname, logoncount`
	- ![[Pasted image 20261116152520.png|363]]



We can search for a particular string in a user attribute using:
- `Get-Domainuser -LDAPFilter "Description=*built*" | select name, Description`
	- We are searching in the description
- `Get-ADUser -Filter "Description -like *built*" -Properties Description |  select name, Description`

==Description is very interesting to understand info on the account and what it does. ==


### Domain Computers
Get domain computers  for current domain
- `Get-DomainComputer | select samaccountname ` (powerV)
	- ![[Pasted image 20261116155024.png|526]]
	- `Get-DomainComputer -OperatingSystem "*Server 2022*" `
	- `Get-DomainComputer -Ping `

==Also here is interesting to get the logoncount==:
- `Get-DomainComputer | select samaccountname, logoncount `
- ![[Pasted image 20261116155230.png|520]]

- `Get-ADComputer -Filter *  select Name` (AD module)
	-  `Get-ADComputer -Filter * -Properties *`
	-  `Get-ADComputer -Filter 'OperatingSystem -like "*Server 2022*"' -Properties OperatingSystem | select Name, OperatingSystem`
	-  `Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName`


### Domain Groups
Get domain groups  for current domain
- `Get-DomaingGroup | select name ` (powerV)

Get domain groups  for external domain
- `Get-DomainGroup -Domain <target> `
	

==Also here is interesting to get the logoncount==:
- `Get-DomainGroup | select samaccountname, logoncount `

For example we can search all groups that contain the word "admin":
- `Get-DomainGroup *admin* | select cn`
	- ![[Pasted image 20261116155640.png|447]]
- `Get-DomainGroup -Domain moneycorp.local *admin* | select cn`
	- ![[Pasted image 20261116155839.png|454]]

- `Get-ADGroup -Filter *  select Name` (AD module)
	-  `Get-ADGroup -Filter * -Properties *`

### Domain Admins Group
To get all the members of the Domain Admins Group we can easily do:
- `Get-DomainGroupMember -Identity "Domain Admins" -Recurse` (Powerview)
	- ![[Pasted image 20261116160356.png|414]]

To get a single goup membership (o a single user):
- `Get-DomainGroup -username "username"`

- `Get-ADGroupMember -Identity "Domain Admins" -Recursive` (AD module)
- `Get-ADPrincipalGroupMembership -Identity "username"`


==🔴The built-in ADMIN has always a sid that terminates with **500**, also if the name is different==
- ![[Pasted image 20261116160615.png|515]]


#### Local groups
==**To list local groups you need administrator privileges. If you are in the domain controller you don't need administrator privileges**==

To list it we need:
- `Get-netLocalGroup -ComputerName dcorp-dc`


To list all the members of the local group "Administrators" on a machine:
- `Get-NetLocalGroupMember -ComputerName dcorp-dc -GroupName Administrators`
	- ![[Pasted image 20261116161048.png|620]]


### Logged users
==**To list logged users you need administrator privileges. If you are in the domain controller you don't need administrator privileges**==

To list the actively logged 
- `Get-netLoggedOn -Computername dcorp-adminsrv`

To list locally logged users (needs remote registry on the target started by default on server OS)
- `Get-LoggedonLocal -Computername dcorp-adminsrv`


To list last logged users on a computer (needs administrative rights and remote registry on the target)
- `Get-LastLoggedOn -Computername dcorp-adminsrv`

### Enumerate Shares
To enumerate Shares we need a tool called PowerHuntShares [https://github.com/NetSPI/Powerhuntshares]

It can discover shares, sensitive files, ACL from shares, networks, computers, identities etc. It generates also an HTML report.

A share is a shared space that can involve networks, files etc.

To use it:
- `Invoke-HuntSMBShares -NoPing -OutputDirectory C:\AD\Tools -HostList C:\AD\Tools\servers.txt`
	- In `servers.txt` we have to avoid domain controllers to not have problems


`servers.txt` doesn't include the ddomain controller for  abetter OPSEC
The report includes "ShareGraph" that can be used to explore share relationships on your host (not on the exam host).


## Blood Hound
Bloohoud provides a GUI for AD entities and relashionships for the data collectors by its ingestors.

It uses Graph Theory for providing the capability of mapping shortest path for interesting things like domain Admins.

It has builtin queires for frequently used actions and also supports custo cypher queries.

There are 2 free verisons:
- Legacy [https://github.com/SpecterOps/BloodHound-Legacy]
- Community edition [https://github.com/SpecterOps/BloodHound]

It is present in `C:\AD\Tools` of the student VM

We can have the access to Read-Only prep-populated BloodHound CE 



==🔴To collect data we can do:==
- `C:\AD\Tools\Loader.exe -Path C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.exe -args --collecionmethods All`
	- `All` uses all the methods ==better to remove some (see later)==
	- The data collected can be uploaded to Bloodhoud legacy
	- In the lab we can have read only access to web UI of Bloodhound


Cypher Queries can be used in the Bloodhound query library in the Web UI [https://queries.specterops.io]
- ![[Pasted image 20261116180951.png|552]]

### Stealthy BloodHound
To make BloodHound stealth we can remove noisy collection methods like RDP, DCOM, PSremote, LocalAdmin.

We can also avoid detection by MDI (microsoft defender by identity) by using `-ExcludeDCs` (exclude domain controllers).


==🔴So we can do:==
- `C:\AD\Tools\Loader.exe -Path C:\AD\Tools\SharpHound\SharpHound.exe -args --collecionmethods Group,GPOLocalGroup,Session,Trusts,ACl,Container,Objectprops,SPNTargets,CertServices --excludedcs`
	- `CertServices` must be removed when using Bloodhound legacy collector.

This gives us files. Then we upload these files to Bloodhound dashboard.


## SOAPHound

SoapHound is used for a even more stealth.

It talks to AD Web Service (port 9389) in place of sending LDAP queries, just like the AD Module.
- Almost no network based detection (like MDI)
- it gets info about all objects and then process them. So we have limited queries and less chance of endpoint detection.

To biuld a cache that includes basic info about domain objects we can do:
- `SOAPHound.exe --buildcache -c C:\AD\Tools\cache.txt`

To collect data compatible with bloodhound:
`SOAPHound.exe -c C:\AD\Tools\cache.txt --bhdump -o C:\AD\Tools\bhoutput --nolaps`
