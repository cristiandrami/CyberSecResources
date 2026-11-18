In an AD environment, there are multiple scenarios which lead to privilege escalation.

Generally it can be done with:
- Hunting for Local Admin access on other machines
- Hunting fo rhigh privilege domain accounts (like a domain administrator)

![[Pasted image 20261117154953.png|548]]


There are various ways of locally escalating privileges on Windows Box:
- Missing patches and exploits
- Automated deplyment and Autologon passswords in clear text
- AwlaysInstallElevated (Any user can run MSI as SYSTEM)
- Misconfigured Services
- DLL hijacking and more
- Kerberos and NTLM relaying


The tools for the complete coverage are:
- PowerUP [https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1]
- PrivescCheck [https://github.com/itm4n/PrivescCheck]
- WinPEAS (The best one) [https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS]

## PowerUp services

With PowerUP we can get Services with unqouted paths and a space in their name:
- `Get-ServiceUnquoted -verbose`
	- this because with unqouted services, if we are able to write on the folder we can create an .exe that is triggered when the service is restarted
		- ![[Pasted image 20261117160830.png]]
		- If we create on `C:\WebServer` a file called `Abyss.exe` when we restart the service it will be triggered
		- it is possible only if we have permission to write in `C:\WebServer`
	- ![[Pasted image 20261117161244.png]]
	  This is not vulnerable because is quoted, so it's taken as it is

We can also get the services where the current user can write to its binary path or change arguments to the binary
- `Get-ModifiableServiceFile -Verbose`
	- This because if the user can modify the path of the service binary he can put a malicious one
	- ![[Pasted image 20261117161526.png]]
		- The first one is not vulnerable, the second one yes, you can see WD (everyone has that permissions)
		- ![[Pasted image 20261117161610.png]]
		- `sc.exe config SNMPTrap binpath=` can be used to change the binary path


We can get the services whose configuration current user can modify:
- `Get-ModifiableService -verbose`



# Run all checks
We can run all checks using:
- PowerUP
	- `Invoke-AllChecks`
- PrivescCheck
	- `Invoke-PrivEscCheck`
- PEASS-ng
	- `winPEASx65.exe`
