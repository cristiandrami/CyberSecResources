Local Security Authority (LSA) is responsible for authentication on a Windows machine.
Local Security Authority Subsystem Service (LSASS) is its service.

LSASS storese credentials in multiple forms
- NT hashes
- AES 
- Kerberos tickets and so on

Credentials are stored by LSASS when a user logs on to a local session or using rdp, uses RunAs, runs a windows service, runs a scheduled tasj or batch job, uses a Remote Admin tool.

LSASS is a very attractive target service. But t is the most monitored process on windows.

There are some credentials taht can be extracted without touchin LSASS 
- SAM hive (registry), and so the local creds
- LSA secrets/Security Hive (registry), and so service account passwords, domain cached credentials.
	- Very interesting one.
- DPAPI Protected Credentials (Disk), and so Credentials manager/vault, Browser cookies, Certificates, Azure tokens etc.


==🔴LSASS must be the last option.==

## Console history
You can also search for interesting stuff in the consol history.

To see where the history is saved you can do:
- `Get-PSReadLineOption`
	- ![[Pasted image 20261119162941.png|554]]
	- ![[Pasted image 20261119163008.png|559]]


