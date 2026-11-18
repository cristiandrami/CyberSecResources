ACLs are the Access Control Model.

They enables control on the ability of a process to access objects and other resources in active directory based on:
- Access Tokens (defines the identity and privs of users). This is the security context of a process.
- Security descriptors (SID of the owner, Discretionary ACL (DACLS) and System ACL (SACL))


An ACL has a list of permissions, that defines who is allowed to do what:
- ![[Pasted image 20261117092030.png|247]]

Each Process has Access Token that contains some info:
- ![[Pasted image 20261117092158.png|220]]

When a process ties to access an object then the permissions are compared with the rules of the ACL.

Example:
- ![[Pasted image 20261117093616.png|533]]
	- Thread A cannot access the object because in the rules we have "Andrew - Read, write, execute - Access Denied".

## Discretionary ACL (DACLs)
DACLs defines the permissions trustees a user or a group has on an object.

## System ACL (SACLs)
SACLs logs success and failure audit messages when an object is accessed.


If an object has in the ACL a full control on a DC, even if it is not an Administrator it can do everything such as password reset of other users etc.:
- ![[Pasted image 20261117092642.png|484]]




# Enumeration of ACL

To understand the ACLs associated with a specific object:
- `Get-DomainObjectACL -SamAccountName <name> -ResolveGUIDs`

To filter by a group:
- `Get-DomainObjectACL -Identity "Domain Admins" -ResolveGUIDs -Verbose`

==To get the ACLs associated with specified prefix to be used for a search:==
- `Get-DomainObjectACL -SearchBase "LDAP://CN=Domain Admins, CN=Users, DC=dollarcorp,DC=moneycorp,DC=local" -ResolveGUIDs -Verbose`
	- In questo caso stimao leggendo le ACL associate al gruppo domain admins
	- Stiamo cercando solo un percorso LDAP "Domain admins che sta nel dominio dollarcorp.moneycorp.local"



With AD module we can use:
- `(Get-ACL 'AD:\CN:Administrator, CN=Users, DC=dollarcorp, DC=moneycorp, DC=local').Access`

## Interesting ACEs
**==🔴Searching for interesting ACEs:==**
- `Find-InterestingDomainAcl -ResolveGUIDs`

To filter by username:
- `Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "student555"}`

But using the gui is more usable.

Get the ACLs associated with the specified path:
- `Get-PathACL -Path "\\dcorp-dc.dollarcorp.moneycorp.local\sysvol"`