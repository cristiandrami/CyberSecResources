
## Blood Hound
Bloohoud provides a GUI for AD entities and relashionships for the data collectors by its ingestors.

It uses Graph Theory for providing the capability of mapping shortest path for interesting things like domain Admins.

It has builtin queires for frequently used actions and also supports custo cypher queries.

There are 2 free verisons:
- Legacy [https://github.com/SpecterOps/BloodHound-Legacy]
- Community edition [https://github.com/SpecterOps/BloodHound]

It is present in `C:\AD\Tools` of the student VM

We can have the access to Read-Only prep-populated BloodHound CE 



==🔴To collect data we can do:==
- `C:\AD\Tools\Loader.exe -Path C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.exe -args --collecionmethods All`
	- `All` uses all the methods ==better to remove some (see later)==
	- The data collected can be uploaded to Bloodhoud legacy
	- In the lab we can have read only access to web UI of Bloodhound


Cypher Queries can be used in the Bloodhound query library in the Web UI [https://queries.specterops.io]
- ![[Pasted image 20261116180951.png|552]]

### Stealthy BloodHound
To make BloodHound stealth we can remove noisy collection methods like RDP, DCOM, PSremote, LocalAdmin.

We can also avoid detection by MDI (microsoft defender by identity) by using `-ExcludeDCs` (exclude domain controllers).

==🔴So we can do:==
- `C:\AD\Tools\Loader.exe -Path C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.exe -args --collectionmethods Group,GPOLocalGroup,Session,Trusts,ACl,Container,Objectprops,SPNTargets --excludedcs`
	- `CertServices` must be removed when using Bloodhound legacy collector.

This gives us files. Then we upload these files to Bloodhound dashboard.


## SOAPHound

SoapHound is used for a even more stealth.

It talks to AD Web Service (port 9389) in place of sending LDAP queries, just like the AD Module.
- Almost no network based detection (like MDI)
- it gets info about all objects and then process them. So we have limited queries and less chance of endpoint detection.

To biuld a cache that includes basic info about domain objects we can do:
- `SOAPHound.exe --buildcache -c C:\AD\Tools\cache.txt`

To collect data compatible with bloodhound:
`SOAPHound.exe -c C:\AD\Tools\cache.txt --bhdump -o C:\AD\Tools\bhoutput --nolaps`
