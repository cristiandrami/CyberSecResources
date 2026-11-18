1. Enumerate following for teh dollarcorp domain:
	1. List all OUs x 
	2. List all computers in the DevOps OU x
	3. List the GPOs x
	4. Enumerate GPO appplied on DevOps OU x
	5. Enumerate ACLs for Applocker and DevOps GPOs x

# Make the shell safe 
To make our work safe we can run:
- `. C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat`

# Run Powerview
- `. C:\AD\Tools\PowerView.ps1`

# List all OUs
To get all the OUs:
- `Get-DomainOU`
	- ![[Pasted image 20261118091919.png|413]]
- `Get-DomainOU | select name, distinguishedname`
	- ![[Pasted image 20261118092037.png]]


# List all the computers in DevOps OU
To list the computers in DevOps OU:
- `(Get-DomainOU -Identity <OUname>).distinguishedname | %{Get-DomainComputer -SearchBase $_} |select name`
	- ![[Pasted image 20261118092453.png|488]]


# List GPOs
To list GPOs we just need to do:
- `Get-DomainGPO | select displayname`
	- ![[Pasted image 20261118092626.png]]

# List GPO associated with DevOps

First of all we need to get the gplink code associated to the DevOps OU:
- `Get-NetOU | select name, gplink`
	- ![[Pasted image 20261118093629.png]]
	or `Get-DomainOU -Identity "DevOps" | select name, gplink`

Then we can get th epolicies related to the OU:
- `Get-DomainGPO -Identity "{0BF8D01C-1F62-4BDC-958C-57140B67D147}"`
	- ![[Pasted image 20261118093837.png]]
	- GPOs applied on a "DevOps"

# Enumerate ACLs for Applocker and DevOps GPOs
Its easy.
Get the GPO of Applocker and DevOps:
- `Get-NetOU | select name, gplink`
- `Get-DomainGPO -Identity "{0BF8D01C-1F62-4BDC-958C-57140B67D147}"`
- `Get-DomainGPO -Identity "{0BF8D01C-1F62-4BDC-958C-57140B67D147}"`
	- ![[Pasted image 20261118094018.png]]

Now we have to seach for the ACL.

We can do:




We can just use BloodHound:
- ![[Pasted image 20261118094627.png]]
	- Searching the node we can see that  `RDPUsers` and `Domain Admins` have `GenericWrite`, `(MoneyCorp.local)` has writeowner
- ![[Pasted image 20261118094837.png|551]]