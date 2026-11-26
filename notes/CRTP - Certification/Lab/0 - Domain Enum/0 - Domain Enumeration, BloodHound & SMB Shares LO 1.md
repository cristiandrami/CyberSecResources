
> [!INFO] Learning objectives
> In this lab, we want to:
> 1. Enumerate domain users, computers, Domain Admins and Enterprise Admins.
> 2. Identify potential paths to Domain Admins using BloodHound.
> 3. Find a file share where our student account has write access.
>
> Our current lab identity is `student555` (the course sometimes uses `studentx` as a generic placeholder).

## 1. Prepare our PowerShell environment

First, we need to load the tools provided in the CRTP lab on our Student VM.

```powershell
. C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat

. C:\AD\Tools\PowerView.ps1
```

We use **PowerView** to enumerate Active Directory. The lab also uses **InviShell**, which is associated with altering PowerShell logging and inspection behavior. We should not confuse this with making the shell intrinsically safe or undetectable.

  

> [!NOTE]
> Iniziamo dalla Student VM. 
> 
> PowerView ci permette di interrogare utenti, gruppi, computer, GPO e relazioni di dominio. InviShell è presente nell'ambiente del corso per modificare alcuni comportamenti di logging: non significa che ogni comando diventi invisibile. 
> 
> Caricare PowerView con `. <percorso>` rende disponibili le sue funzioni nella sessione PowerShell.

## 2. Identify the current domain

Now we need to understand which domain our shell belongs to and obtain the basic domain properties.

```powershell
Get-Domain
```

![[Pasted image 20261117184427.png|427]]


We should record the domain name and domain controller information before continuing our enumeration.

> [!NOTE] 
> `Get-Domain` recupera informazioni sull'oggetto dominio tramite Active Directory. 
> 
> Ci serve per evitare di confondere il dominio figlio `dollarcorp.moneycorp.local` con il dominio radice della foresta `moneycorp.local`. 
> 
> La presenza di più domini è importante quando cerchiamo gruppi privilegiati.

> [!warning] HINTS
> If a privileged group is not found in the current domain, check whether it belongs to the **forest root domain**. In our environment, this is particularly relevant for `Enterprise Admins`.



## 3. Enumerate domain users

Now we can list the users in the current domain.

```powershell
Get-DomainUser
```

The full output contains many attributes, so we select only the fields we need.

```powershell
Get-DomainUser | Select UserPrincipalName
```

![[Pasted image 20261117184946.png|359]]


> [!NOTE] 
> Elenchiamo gli account presenti nel dominio. 
> 
> `Get-DomainUser` restituisce numerosi attributi LDAP; usando `Select-Object` limitiamo la visualizzazione a `UserPrincipalName`, che normalmente identifica l'utente nel formato `utente@dominio`. 
> 
> Un UPN può anche non essere valorizzato: non è un requisito per l'esistenza dell'account.

  
> [!warning] HINTS
> While reviewing users, pay attention to **privileged identities, service accounts and accounts associated with important machines**. 
> 
> An interesting username alone does not prove we can access it; we need an actual relationship such as a group membership, ACL permission or session edge.


## 4. Enumerate domain computers

Next, we need to identify the domain-joined machines.

```powershell
Get-DomainComputer | select name
```

![[Pasted image 20261117185205.png|414]]

We can use this inventory to identify servers and domain workstations that may be relevant in later learning objectives.


> [!NOTE] 
> `Get-DomainComputer` enumera gli oggetti computer registrati in Active Directory, non necessariamente le macchine attualmente accese o raggiungibili. 
> 
> Il filtro `select name` rende l'elenco più leggibile. Questo inventario ci servirà per individuare sistemi come `dcorp-adminsrv`, `dcorp-ci` e `dcorp-dc`.

> [!warning] HINTS
> Machines linked to **administrative sessions**, **delegated administration**, or **modifiable GPOs** deserve closer attention in an authorized lab. 
> 
> But an `AdminTo` or `HasSession` edge must be validated: graph data can be incomplete or outdated.



## 5. Enumerate Domain Admins

Before enumerating privileged members, we can inspect the available groups.

```powershell
Get-DomainGroup | select name
```

![[Pasted image 20261117185618.png|345]]

Now we need to enumerate the members of the `Domain Admins` group, including nested group membership.

```powershell
Get-DomainGroupmember -Identity "Domain Admins" -recurse
```

  

![[Pasted image 20261117190034.png|533]]

`-Recurse` asks PowerView to expand nested group memberships rather than returning only direct group members.


> [!NOTE] 
> I Domain Admins sono un gruppo privilegiato **nel rispettivo dominio**. 
> 
> Prima individuiamo i gruppi e poi espandiamo `Domain Admins`. 
> 
> Il parametro `-Recurse` è importante perché un account può ottenere privilegi attraverso un gruppo annidato. 

  

> [!warning] HINTS
> In BloodHound, we should inspect not only **direct membership** in Domain Admins but also paths through **nested groups**, **control over a group**, or **administration of a machine hosting a privileged session**. 
> 
> Always read each edge label instead of assuming that every connection grants direct access.


## 6. Enumerate Enterprise Admins in the forest root

At this point, we also want to identify the Enterprise Admins group. 

In our lab, it is located in the forest root domain, `moneycorp.local`, rather than the child domain `dollarcorp.moneycorp.local`.

In fact if we run
```powershell
Get-DomainGroupMember -domain moneycorp.local -Identity "Enterprise Admins"
```
- It takes our current domain and gives no result.

![[Pasted image 20261117190616.png|551]]


First, we query the groups in the root domain.

```powershell
Get-DomainGroup -domain moneycorp.local | select name
```

![[Pasted image 20261117190720.png|477]]

Then we enumerate the group members.

```powershell
Get-DomainGroupMember -domain moneycorp.local -Identity "Enterprise Admins"
```

![[Pasted image 20261117190923.png|566]]

  
> [!NOTE]
> Il gruppo Enterprise Admins è normalmente presente nel dominio radice della foresta e ha un ambito diverso da Domain Admins. 
> 
> Per questo specifichiamo `-Domain moneycorp.local`. 
> 
> Se cercassimo il gruppo soltanto nel dominio figlio, potremmo non individuarlo. Il comando di enumerazione legge l'appartenenza al gruppo, non ci attribuisce i relativi privilegi.

> [!warning] HINTS
> If we see a path toward a principal from another domain, verify **which domain owns the group** and whether the trust relationship supports the required authentication and authorization. 
> 
> Being in the same forest does not mean all cross-domain actions are automatically allowed.



## 7. Identify potential paths to Domain Admins with BloodHound

Now we want to understand how users, groups, sessions, machines and permissions are related. We can use SharpHound to collect data for BloodHound.

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.exe -args --collectionmethods Group,GPOLocalGroup,Session,Trusts,ACl,Container,Objectprops,SPNTargets,CertServices --excludedcs
```

![[Pasted image 20261117193554.png|562]]

The collector produces structured data (including JSON) that BloodHound can analyze. 

`--excludedcs` excludes domain controllers from relevant collection activity, so some relationships or session data may be absent.

In our lab, the BloodHound dashboard does not allow file upload. 
We can instead use the lab-provided BloodHound instance and the data already available there.


Now, inside the graph interface, we choose the query **Shortest Paths to Domain Admins**.

![[Pasted image 20261117193754.png]]

The screenshots show a potential path involving our student account, `RDPUsers`, `dcorp-adminsrv`, `dcorp-dc`, `Administrator`, and `Domain Admins`.

![[Pasted image 20261117193929.png|540]]

![[Pasted image 20261117194023.png|428]]


![[Pasted image 20261117194253.png|328]]

  
We need to inspect the **exact edge labels** for each relationship rather than assuming that all links represent the same privilege.

  
> [!NOTE] 🇮🇹 Spiegazione in italiano
> SharpHound raccoglie relazioni da Active Directory e, in base al tipo di raccolta, anche informazioni ottenute interrogando computer del dominio. 
> 
> BloodHound le rappresenta come un grafo: i nodi sono oggetti (utenti, gruppi, computer, GPO), gli archi indicano relazioni e privilegi. 
> 
> Un percorso breve segnala **un possibile cammino di escalation**, non dimostra che abbiamo già accesso a tutte le macchine. Dobbiamo interpretare gli archi, ad esempio `MemberOf`, `AdminTo`, `CanRDP` o altri diritti, senza trattarli come equivalenti.

> [!warning] HINTS
> If the graph does not show a promising path, first check **collection coverage**: were ACLs, sessions, group memberships, GPO links and relevant hosts actually collected? 
> 
> Missing edges may indicate incomplete data, not the absence of an escalation path. Compare paths from the current student account, not just global shortest paths.
  

> [!warning] HINTS
> When choosing which path to investigate, look for:
> - **A starting node we actually control** (`student555` or one of its groups).
> 
> - **Strong control edges** such as `GenericAll`, `GenericWrite`, `WriteDacl`, `WriteOwner`, or permissions to change group membership.
> 
> - **Machines where our identity is already local administrator** and whether those machines are related to privileged accounts.
> 
> - **GPOs linked to important OUs** where our account or its groups can modify relevant settings.
> 
> - **Sessions of privileged users**, while remembering that discovering a session is not equivalent to compromising it.
> 
> - **Alternative paths with fewer dependencies**, not only the mathematically shortest one.
>
> For each candidate path, we should write down: *What do we control? What must be true for the next edge? What evidence will confirm it?*

## 8. Find SMB shares writable by our account

Next, we want to find a share where our student identity may have write access. We start by loading PowerHuntShares.

```powershell
Import-Module C:\AD\Tools\PowerHuntShares.psm1
```

We then need a target list. We can obtain the DNS hostnames of domain computers and use them to prepare `C:\AD\Tools\servers.txt`.

```powershell
Get-DomainComputer | select dnshostname
```

![[Pasted image 20261117195059.png|559]]


Now we can run the share enumeration using that host list.

```powershell
Invoke-HuntSMBShares -NoPing -OutputDirectory C:\AD\Tools -HostList C:\AD\Tools\servers.txt
```

`-NoPing` skips the initial ping check; it does not guarantee that every target is reachable. 
I suggest to reopen PowerShell if the scan fails.

> [!NOTE] 
> `PowerHuntShares` analizza condivisioni SMB e relative autorizzazioni. 
> 
> Prima prepariamo l'elenco degli host; `Select-Object DnsHostName` da solo stampa i nomi, quindi dobbiamo assicurarci che `servers.txt` contenga realmente **un hostname per riga**. 
> 
> `-OutputDirectory` definisce dove scrivere il report. Un permesso di scrittura efficace dipende sia dai permessi della share sia da quelli NTFS, oltre che dall'identità usata per il collegamento.


> [!warning] HINTS
> A writable share is **not automatically an escalation**. 
> 
> We should ask: *Who reads or executes files from this location? Is it used by a privileged service, job, or administrative workflow?* 
> 
> Also distinguish share permissions from underlying NTFS permissions and confirm the **effective** access. 
> 
> The risk increases when a writable location is consumed by a more privileged process.



## 9. Review the report and record the finding

Once the scan finishes, we need to review the generated report. In this lab, we could not open it directly in the Student VM, so we transferred it to our local computer using the lab's file-transfer mechanism.

![[Pasted image 20261117195602.png|551]]

![[Pasted image 20261117203807.png|543]]


When we open the report, we find a folder named `AI` for which the report indicates write access for `Everyone`, along with other potentially interesting shares.

![[Pasted image 20261117204413.png|526]]

We should treat that as a finding to validate against effective permissions, not as a guarantee that every anonymous or domain account can write.

  
> [!NOTE]
> La cartella `AI` è interessante perché potrebbe consentire il deposito di file a molti utenti.
> 
> Tuttavia `Everyone` nelle ACL non significa automaticamente che l'accesso anonimo sia permesso: contano il token dell'utente, i permessi SMB e quelli NTFS. 
> Il valore dell'enumerazione consiste nel trovare **relazioni e risorse da verificare**, non nel presumere che ogni controllo sia già stato superato.

  

> [!TIP] CRTP exam reminder
> Domain enumeration, BloodHound edges and SMB permissions answer three different questions: **who exists**, **who controls what**, and **where can our identity write**. 
> 
> We should not infer successful exploitation from enumeration alone.

