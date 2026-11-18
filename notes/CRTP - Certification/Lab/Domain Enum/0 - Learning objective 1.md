1. Enumerate following for the dollarcorp domain:
	1. users x
	2. computers x
	3. Domain Administrators x
	4. Enterprise Administrators x

2. Use BloodHound to identify the shortest path to Domain Admins in the dollar corp domain x
3. Find a file share where the studentx has write permissions


# Make the shell safe 
To make our work safe we can run:
- `. C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat`

# Run Powerview
- `. C:\AD\Tools\PowerView.ps1`
# Get info about the domain
We can do:
- `Get-Domain`
	- ![[Pasted image 20261117184427.png|427]]

# Get current domain users
To get the current domain users:
- `Get-DomainUser`

But it is too much, so we have to 
- `Get-DomainUser | select userpricipalname`
	- ![[Pasted image 20261117184946.png|359]]


# Get current domain computers
To get the computers in the current domain we can do:
- `Get-DomainComputer | select name`
	- ![[Pasted image 20261117185205.png|414]]


# Get Domain administrators 
To get the domain Administrators we need first to get the groups:
- `Get-DomainGroup | select name`
	- ![[Pasted image 20261117185618.png|345]]


To get now the users in the `Domain Admins`:
- `Get-DomainGroupmember -Identity "Domain Admins" -recurse`
	- ![[Pasted image 20261117190034.png|533]]


# Get Enterprise Admins
Now we need the enterprise admins.

The Enterprise Admins are not in the domain `dollarcorp.moneycorp.local`:
- ![[Pasted image 20261117190616.png|551]]

but we can see if in the forest of our domain there is a group called "Enterprise":
- `Get-DomainGroup -domain moneycorp.local | select name`
	- ![[Pasted image 20261117190720.png|477]]

So now we can see the users in it:
- `Get-DomainGroupMember -domain moneycorp.local -Identity "Enterprise Admins"`
	- ![[Pasted image 20261117190923.png|566]]


# BloodHound for shortest path to Domain Admins in the dollarcorp.moneycorp.local domain

to collect data to put in Bloodhound we have to use:
- `C:\AD\Tools\Loader.exe -Path C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.exe -args --collectionmethods Group,GPOLocalGroup,Session,Trusts,ACl,Container,Objectprops,SPNTargets,CertServices --excludedcs`
	- ![[Pasted image 20261117193554.png|562]]
	- It contains some json that are the result of the analysis
However Bloodhound doesn't allow us to upload files in this version (the dashboard).

But we can use it, with no terminal using the credentials of the lab.

So from cypher we can choose `Shortest path to Domain Admins`:
- ![[Pasted image 20261117193754.png]]


From it we can see that each student is connected to `RDPUsers` -> `Dcorp-adminsrv...` -> `Dcorp-dc` -> `Administrator` -> `Domain Admins`:
- ![[Pasted image 20261117193929.png|540]]
- ![[Pasted image 20261117194023.png|428]]
- ![[Pasted image 20261117194253.png|328]]


There are different paths. 


# Find a file share where the studentx has write permissions
First we need to load PowerHuntShares:
- `Import-Module C:\AD\Tools\PowerHuntShares.psm1`
then we have to create a list of servers and we can do copying the result of:
- `Get-DomainComputer | select dnshostname`
	- ![[Pasted image 20261117195059.png|559]]


Now we can do:
- `Invoke-HuntSMBShares -NoPing -OutputDirectory C:\AD\Tools -HostList C:\AD\Tools\servers.txt`
	- IF IT DOESN'T WORK CLOSE THE POWERSHELL AND RE OPEN IT.

Now we have a report, we have to take it in our machine. The student vm cannot open it:
- ![[Pasted image 20261117195602.png|551]]

We have to move it on our pc to open the report.

To do it we just need to copy the folder into:
- ![[Pasted image 20261117203807.png|543]]
	- This allows us to put files into our pc

We open the report and we can see that everyone can write on a folder called `AI` and others:
- ![[Pasted image 20261117204413.png|526]]
