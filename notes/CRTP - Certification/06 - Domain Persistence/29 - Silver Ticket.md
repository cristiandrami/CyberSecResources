In the golden ticket account what we do is to steal the secret of `krbtgt` user account and to use it to forge `TGT` tickets.

In the silver ticket attack what we do is:
- Steal the `target service` AES key or NTLM hash
- Use the `target service` secret to force directly a `ST` or `TGS` (it is signed using the `target service` secret)

>[!Important] 
>Here we don't need to interact with the Domain Controller.
>
>We forge directly the `TGS` or  the `ST` and we use it directly with the `target service`.

![[Pasted image 20261123154424.png]]
## Where we can find the service account secret?
In case of windows almost every service uses as `service account` the `machine service account`.


**By default, in Active Directory, a machine account password automatically rotates every 30 days.**
- The persistence is limited compared to the golden ticket.

# Forging a silver ticket from Rubeus

To generate a golden ticket with attributes similar to a normal TGT we could use Rubeus:
```powershell
C:\AD\Tools\Rubeus.exe silver  /service:http/dcorp-dc.dollarcorp.moneycorp.local /rc4:<service_rc4> /sid:<domain_sid> /ldap /user:Administrator /ptt
```
- `/ldap` is used to retrieve as many information of the doamin as possible, that are needed to create the ticket
- `/user:Administrator` is used to generate a ticket that will be used by `Administrator`, so we impersonate the default `Administrator` account.
- `/ptt` injects the current process (a shell)
- `/service` is used to define which service we would access 
	- It can be `http`, `host`, `rpcss`, `cifs` etc
