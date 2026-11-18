1. Enumerate following for the dollarcorp domain:
	1. ACL for Domain Admins group
	2. ACLs where studentx has interesting permissions
2. Analyze the permissions for student in BloodHound UI

# Make the shell safe 
To make our work safe we can run:
- `. C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat`

# Run Powerview
- `. C:\AD\Tools\PowerView.ps1`
# Enumertae ACLs for Domain Admins

At this point we can enumerate the ACLs:
- `Get-DomainObjectACL`
	- They are too much

So we filter for `Domain Admins`:
- `Get-DomainObjectACL -Identity "Domain Admins"`
	- ![[Pasted image 20261118085242.png|492]]

We can also search by LDAP:
1. `Get-DomainGroup | select name,distinguishedname `
	1. ![[Pasted image 20261118085020.png]]
2. `Get-DomainObjectACL -SearchBase "LDAP://CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local" -ResolveGUIDs -Verbose`
	1. ![[Pasted image 20261118085412.png]]


# Enumerate ACL where studentx has interesting permission
My student is student555.

So try to use:
- `Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "student555"}`

Or in case, we need to understand in which group we are and then see the acl:
- `Get-DomainGroup -UserName "student555"`
	- ![[Pasted image 20261118090401.png|507]]
- `Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}`
	- ![[Pasted image 20261118090802.png|492]]




# Analyze the permissions for student in BloodHound UI

To do it we have to login in the BloodHound UI on the dashboard.

In this case we just need to look ath the graph and choose  in `Cypher` `Shortest Path to Domain Admins` the user:
- ![[Pasted image 20261118091133.png|542]]
- On the right we can see the permissions it has
	- ![[Pasted image 20261118091207.png|277]]
	- ![[Pasted image 20261118091237.png|382]]

So `student145` has Local Admin Privileges on `dcorp-adminsrv`