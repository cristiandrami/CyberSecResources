
> [!info] Learning objectives
> 1. Enumerate the ACL of the `Domain Admins` group.
> 2. Find interesting ACL entries associated with our student account or its groups.
> 3. Interpret our account's relationships in BloodHound.

## 1. Prepare our PowerShell environment

We begin from our Student VM, preparing PowerShell and loading PowerView as in the previous learning objective:

```powershell
. C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat

. C:\AD\Tools\PowerView.ps1
```
  
## 2. Inspect the `Domain Admins` group ACL
  
Now we want to understand who has permissions over the `Domain Admins` group. We could enumerate all the object ACLs:

```powershell
Get-DomainObjectACL
```


However, this produces too much output. We narrow the target:

```powershell
Get-DomainObjectACL -Identity "Domain Admins"
```

![[Pasted image 20261118085242.png|492]]


We can also first identify the LDAP Distinguished Name of the group:

```powershell
Get-DomainGroup | select name,distinguishedname
```

![[Pasted image 20261118085020.png]]


Then we can query its ACL using the LDAP path :

```powershell
Get-DomainObjectACL -SearchBase "LDAP://CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local" -ResolveGUIDs -Verbose
```

![[Pasted image 20261118085412.png]]

The `-ResolveGUIDs` flag helps us interpret GUID-based permission information in a human-readable form where resolution is available.


> [!note]
> L'ACL del gruppo descrive **chi può operare sull'oggetto gruppo**. 
> 
> Non coincide con la lista dei membri del gruppo. 
> 
> Per capire se un permesso è interessante dobbiamo identificare il trustee (utente/gruppo), il diritto concesso, l'eventuale attributo specifico e se l'ACE è di tipo Allow o Deny.

  

> [!warning] HINTS
> When reading an ACL, focus on **who owns or can modify the object**, not only on who is a group member. 
> 
> Especially relevant permissions include `GenericAll`, `WriteDacl`, `WriteOwner`, and a scoped right to change group membership. 
> 
> But always identify the **exact target object**: control over an unrelated object is not control over `Domain Admins`.

  

## 3. Find interesting permissions for `student555`

We now want to identify ACLs that may affect our user:

```powershell
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "student555"}
```

This searches the ACL enumeration results for ACEs where the principal name matches `student555`.


> [!note] 
> Cerchiamo se `student555` compare esplicitamente come soggetto che possiede permessi interessanti su oggetti del dominio. 
> 
> Un risultato vuoto non ci autorizza a concludere che non esistano permessi sfruttabili, perché questi potrebbero essere ereditati o attribuiti a un gruppo del quale facciamo parte.


> [!warning] HINTS
> Direct-user filters are **not enough**. 
> 
> If we do not see a useful ACE for `student555`, the next question should be: *Which groups is student555 a member of, including nested groups?* 
> 
> Then inspect ACEs whose trustees correspond to those groups. This often explains BloodHound edges that do not explicitly mention our username.

## 4. Expand our group memberships

Now we investigate the groups to which our account belongs:

```powershell
Get-DomainGroup -UserName "student555"
```

![[Pasted image 20261118090401.png|507]]


`RDPUsers` was one relevant group. We therefore filter interesting ACLs for that group:

```powershell
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```

![[Pasted image 20261118090802.png|492]]

> [!note] 🇮🇹 Spiegazione in italiano
> L'appartenenza a un gruppo può attribuirci permessi che non appaiono nelle ACL filtrate per il solo nome del nostro account. 
> 
> Se `student555` appartiene a `RDPUsers`, dobbiamo studiare anche le ACE assegnate a `RDPUsers`. 
> 
> La presenza del gruppo nell'ACL di un oggetto non implica automaticamente un diritto amministrativo: bisogna leggere quali operazioni sono concesse.

  

> [!warning] HINTS
> In BloodHound, check for chains of the form **controlled user → member of group → group has control rights over an object**. P
> 
> rioritize cases where the controlled object is itself privileged or leads to a privileged machine or account. 
> 
> Distinguish `MemberOf`, `AdminTo`, `CanRDP`, and ACL-control edges: their prerequisites and consequences are different.


## 5. Interpret relationships in BloodHound

We now open the BloodHound interface provided by the lab and select the appropriate shortest-path analysis from our student account toward `Domain Admins`.

![[Pasted image 20261118091133.png|542]]

We inspect the edge descriptions and permission information shown on the right side of the graph:
  
![[Pasted image 20261118091207.png|277]]

![[Pasted image 20261118091237.png|382]]


Here we end with a statement about **`student145`** having local administrator rights on `dcorp-adminsrv`. 
- This because bloodhound is a read only version in the lab. So we have information about the lab that are a representaion of the real info.
- `student145` is basically equal to `student555`


> [!note] 
> Apriamo il grafo per verificare il significato delle relazioni, non soltanto per trovare un percorso visivamente corto. 
> 
> Negli screenshot finali gli appunti parlano di `student145`, mentre il nostro utente è `student555`. Il lab ha una versione `giocattolo` di bloodhound ma le ralizioni sono quelle reali della rete. `student145` è uguale a `student555`.


> [!warning] HINTS
> For every possible path, we should answer these questions before relying on it:
> - **Source:** Does the first node represent `student555` or a group we actually belong to?
> - **Control:** Which exact edge grants a usable right (`GenericWrite`, `WriteDacl`, `AddMember`, `AdminTo`, etc.)?
> - **Target:** Is the target a user, group, computer, OU, or GPO?
> - **Conditions:** Does the edge imply ownership, an active session, or an already available credential?
> - **Validation:** What output would prove the edge is currently true?
> 
> A **shorter** path is not always a **more practical** path. Prefer a path with fewer unverified assumptions.


## 6. Understand common ACL rights

| Right / BloodHound relationship | What it generally means                             | Why we care                                                 |
| ------------------------------- | --------------------------------------------------- | ----------------------------------------------------------- |
| `GenericAll`                    | Broad control over the target object                | Could be critical if the target is sensitive                |
| `GenericWrite`                  | Ability to modify many target-object properties     | Depends strongly on the object type and writable attributes |
| `WriteDacl`                     | Ability to change an object's DACL                  | Can change which principals receive permissions             |
| `WriteOwner`                    | Ability to modify ownership in applicable contexts  | Ownership can influence future control of the object        |
| `AddMember`                     | Ability to change membership of a target group      | Relevant when the group grants important privileges         |
| `AdminTo`                       | A principal has administrative rights on a computer | Stronger than simply having a remote-login capability       |
| `CanRDP`                        | An RDP access relationship exists                   | Does not imply local admin rights                           |
| `HasSession`                    | A user session was observed on a computer           | Does not itself provide access to that session              |

  
> [!note] 
> Un privilegio ha significato solo rispetto all'oggetto su cui viene esercitato. 
> 
> Ad esempio `GenericWrite` su un utente non produce automaticamente gli stessi effetti di `GenericWrite` su una GPO. 
> 
> Allo stesso modo, `CanRDP` indica la possibilità di accesso remoto secondo determinate condizioni, non il possesso di una sessione amministrativa.


> [!warning] HINTS
> **CRTP mindset:** Instead of asking *“Where is GenericAll?”*, ask **“What object can we control, and what trust relationship does that object influence?”** 
> 
> Then move from a graph hypothesis to a verified permission, one edge at a time.


