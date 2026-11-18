1. Enumerate all domains in the `moneycorp.local` forest. x
2. Map the trusts of the `dollarcorp.moneycorp.local` domain. x
3. Map external trusts in moneycorp.local forest. x
4. Identify external trusts of dollarcorp domain. Can you enumerate trusts for a trusting forest?

# Make the shell safe 
To make our work safe we can run:
- `. C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat`

# Run Powerview
- `. C:\AD\Tools\PowerView.ps1`

# Enumerate all domains in the `moneycorp.local` forest.
First we need to understand the forest:
- `Get-ForestDomain -verbose`
	- ![[Pasted image 20261118095321.png]]
	- We have 3  domains `dollarcorp.moneycorp.local`, `moneycorp.local`, `us.dollarcorp.moneycorp.local`

# Map the trusts of `dollarcorp.moneycorp.local`
It's easy. We just need to do :
- `Get-DomainTrusts -Domain "dollarcorp.moneycorp.local"`
	- ![[Pasted image 20261118095706.png|538]]

# Map the external trusts of `dollarcorp.moneycorp.local`
To do it we can see at the `FILTER_SIDS` attribute.
But we can do it for each `Domain` in the current forest:
- `Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name} | ?{$_.TrustAttributes -eq "FILTER_SIDS"}`
	- ![[Pasted image 20261118100451.png]]
	- Just one in this case.
	- So we get all domains, we pass it tu Get-DomainTrust and then we filter the ones with FilterAttributes equal to "Filter_SIDS"

# Identify external trusts of dollarcorp domain. Can you enumerate trusts for a trusting forest?

We don't have transitivity trust between forests.

So what we could see is, the trusts of the second forest with us and the trusts with its children. But no other infos.

In fact we can see:
- `Get-DomainTrust -Domain eurocorp.local | ?{$_.TrustAttributes -eq "FILTER_SIDS"}`
	- ![[Pasted image 20261118101032.png]]
	- Only with us

We can try to see all:
- `Get-ForestDomain -Forest eurocorp.local | %{Get-DomainTrust -Domain $_.Name}`
	- ![[Pasted image 20261118101154.png|592]]
	- So we have an error and we can see only the trust with us and with the children of `eurocorp.local`



Forest are security bounds:
- ![[Pasted image 20261118101345.png|498]]