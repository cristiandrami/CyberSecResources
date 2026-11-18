Group Policy provides the ability to manage configuration and changes easily and centrally in AD.

**==A policy setting may only affect a computer or a user.==**
- For Computers we can set security settings, shutdown or startup scripts, assigned applications etc.
- For users we can set security settings, logon and logoff scripts, assigned apps etc.


Group policy settings are contained into a **Group Policy Object (GPO)**.
- it is a virtual collection of policy settings, security permissions and scope of management (SOM) that can be applied to users or computer.

GPO can be linked to organizational units (OU) or to sites and domains.

GPOs can be abnused for several attacks like privsec, backdoors, persistence, etc.

# Organizationals Units (OUs)
Organizations use organizationals units (OUs) for delegating administration.

An OU is the lowest-level AD container to which GPO can be applied.

Attacks at the OU level due to misconfigured or overly permissive GPOs is most common in enterprise environments.

To list them:
- `Get-DomainOU`

To get computers in a OU:
- `(Get-DomainOU -Identity <OUname>).distinguishedname | %{Get-DomainComputer -SearchBase $_} |select name`
	- we use searchbase passing the result of the domainOU distinguishedname
# Enumeration GPO
==**It is not possible with powerview to read the GPO. We can only list them.**==

To list the GPO in current domain:
- `Get-DomainGPO | select displayname`
	- ![[Pasted image 20261117095917.png|403]]
- `Get-DomainGPO -ComputerIdentity dcorp-student1`
	- We read the GPO of the computer dcorp-student1

We can get GPO wich use Resticted Groups or groups.xml for interesting users with:
- `Get-DomainGPOLocalGroup`
	- ![[Pasted image 20261117100141.png]]
	- In this case we don't have local groups

To get the users that are in a local group of a machine using GPO:
- `Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity dcorp-student1`

To get Machines where the given user is member of a specific group:
- `Get-DomainGPOUserLocalgroupMapping -identity student1 -Verbose`

## Enumerating OU

To get the OUs in a domain we can use:
- `Get-DomainOU`
	- ![[Pasted image 20261117100646.png|427]]
In AD module:
`Get-ADOrganizationalUnit -Filter * -Properties *`


To get GPO applied on an OU. 
Read GPOname from gplink attribute from Get-NetOU
- `Get-NetOU | select name, gplink`
	- ![[Pasted image 20261118093629.png]]

Then we can get th epolicies related to the OU:
- `Get-DomainGPO -Identity "{0BF8D01C-1F62-4BDC-958C-57140B67D147}"`
	- ![[Pasted image 20261118093843.png]]
	- this is the gplink of the GPO applied on a OU


