

> [!info] Learning objectives
> 1. List all Organizational Units in the dollarcorp domain.
> 2. Identify computers inside the DevOps OU.
> 3. Enumerate Group Policy Objects and their links.
> 4. Identify the DevOps-related GPO.
> 5. Inspect permissions associated with DevOps and AppLocker GPOs.


## 1. Prepare our PowerShell environment

We begin from our Student VM, preparing the lab session and loading PowerView:

```powershell
. C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat

. C:\AD\Tools\PowerView.ps1
```

## 2. Enumerate Organizational Units

Now we want to understand how the domain objects are organized. We first list the OUs:

```powershell
Get-DomainOU
```

![[Pasted image 20261118091919.png|413]]

To make the results more readable, we display their names and distinguished names:

```powershell
Get-DomainOU | select name, distinguishedname
```

![[Pasted image 20261118092037.png]]

An Organizational Unit groups AD objects and may also serve as a scope for Group Policy links and administrative delegation.


> [!note]
> Una OU è un contenitore logico usato per organizzare oggetti Active Directory. 
> 
> Sapere in quale OU si trova un computer è importante perché i criteri di gruppo possono essere collegati a OU, dominio o sito; le OU possono anche avere autorizzazioni amministrative delegate.


> [!warning] HINTS
> In the graph, we should look for **OUs containing important computers** and ask whether we or one of our groups have permission to modify the OU itself or a GPO linked to it. 
> 
> The OU's name alone tells us little about its effective security configuration.


## 3. Identify computers in the DevOps OU

Now we want to identify the computers that belong to the `DevOps` OU. Our original command was:

```powershell
(Get-DomainOU -Identity <OUname>).distinguishedname | %{Get-DomainComputer -SearchBase $_} |select name
```

`<OUname>` is a placeholder and must be replaced with the actual OU identity. For our lab example, the command becomes:

```powershell
(Get-DomainOU -Identity 'DevOps').distinguishedname | ForEach-Object { Get-DomainComputer -SearchBase $_ } | Select-Object Name
```

![[Pasted image 20261118092453.png|488]]

We can inspect whether `dcorp-ci` or other relevant servers belong to this OU, using the actual results rather than assuming their membership.
  


> [!note] 🇮🇹 Spiegazione in italiano
> Recuperiamo il `distinguishedName` della OU e lo usiamo come base della ricerca dei computer.
> 
>In questo modo limitiamo la ricerca agli oggetti che ricadono nello scope LDAP selezionato. 
>Il risultato dipende dallo scope di ricerca e dalle OU figlie, quindi dobbiamo osservare i DN dei computer se serve una conferma precisa.

  
> [!warning] HINTS
> A GPO that influences a computer can become relevant when we later discover **write permissions over that GPO**.
> 
> Before drawing conclusions, verify the computer's actual OU, linked policies and effective filtering.

  
## 4. Enumerate all GPOs

Next, we list the available Group Policy Objects:

```powershell
Get-DomainGPO | select displayname
```

![[Pasted image 20261118092626.png]]

Each GPO has its own object and identifier. We should distinguish the name of a GPO from the GUID used to reference it.


> [!warning] HINTS
> When reviewing GPOs, give special attention to policies affecting **administrative groups, remote access, application allowlisting or important servers**. 
> 
> Their significance depends on both **where the GPO applies** and **who can edit it**.


## 5. Find GPOs linked to the DevOps OU

Now we need to inspect the `gPLink` attribute of the DevOps OU. Our original lab used:

```powershell
Get-NetOU | select name, gplink
```

![[Pasted image 20261118093629.png]]

Alternatively, we can query the specific OU

```powershell
Get-DomainOU -Identity "DevOps" | select name, gplink
```

We can identify the following GPO GUID associated with the DevOps policy:

```powershell
Get-DomainGPO -Identity "{0BF8D01C-1F62-4BDC-958C-57140B67D147}"
```

![[Pasted image 20261118093837.png]]


We should interpret `gPLink` as a **link between a site/domain/OU and a GPO**, not as the file path containing the Group Policy Template.


> [!note] 
> `gPLink` indica quali GPO sono collegate alla OU. 
> 
> Il GUID ci permette di interrogare una GPO specifica.
> 
> Non va confuso con `gPCFileSysPath`, che indica il percorso dei file della policy. 
> 

  

> [!warning] HINTS
> **Two questions must always be answered together:**
> 1. *Can we modify this GPO?*
> 2. *Does this GPO actually apply to the machine or users we care about?*
>
> A powerful permission on an unrelated GPO may have no relevance to the path we are investigating.

  
## 6. Inspect DevOps and AppLocker GPO permissions

We now want to identify who can modify the relevant policies. 

```powershell
Get-NetOU | select name, gplink

Get-DomainGPO -Identity "{0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E}"
```

![[Pasted image 20261118094018.png]]


We can inspect the target GPO's permissions with the PowerView command below, which is the missing ACL-enumeration step in our original notes:

```powershell
Get-DomainObjectAcl -Identity "{0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E}" -ResolveGUIDs
```

![[Pasted image 20261126162134.png]]

This returns ACEs associated with that GPO object; we must inspect their trustees and permissions. 

  

> [!warning] HINTS
> Pay attention to **`GenericWrite`, `GenericAll`, `WriteDacl`, and `WriteOwner`** on important GPOs. 
> 
> These rights concern control over the policy object, but a meaningful escalation hypothesis still requires an **applicable target** and **effective policy processing**. 
> 
> A GPO linked to `DevOps` or `Applocked` may be especially relevant if the associated computers perform privileged tasks.


## 7. Correlate GPO permissions in BloodHound

We can use BloodHound to make the permission relationships easier to visualize

![[Pasted image 20261118094627.png]]


The graph showed that:
- `RDPusers` (in which there is our `student555` user) has `GenericWrite` permission on `AppLocker` policy.
- `Domain Admins` has also `GenericWrite` permission on `AppLocker` policy.


`Applocker` policy affects `dcorp-adminsrv`. Very interssting since with `student555` we can write on that policy.
![[Pasted image 20261126162704.png]]


These are the relations of `DevOps` Policy.
![[Pasted image 20261118094837.png|551]]

  


> [!warning] HINTS
> When BloodHound displays a GPO-related path, work through it in this order:
> - **Principal:** Do we control the user/group that has the right?
> - **Permission:** Is the edge actually a write/control relationship?
> - **Policy:** Which exact GPO object is involved?
> - **Scope:** Which OU/computer is linked to it?
> - **Effective application:** Are security filtering, WMI filtering, inheritance and processing compatible with the expected effect?
>
> This can help distinguish **a real candidate escalation path** from a graph link that only looks promising.

  

## 8. Recap: gPLink vs gPCFileSysPath vs ACL

| Property               | What it means                                        | What we should ask                 |
| ---------------------- | ---------------------------------------------------- | ---------------------------------- |
| `gPLink`               | Associates a GPO with a site, domain or OU           | Where is the policy linked?        |
| `gPCFileSysPath`       | Points to the GPO's file-system template             | Where are the policy files stored? |
| GPO ACL                | Defines who can read, edit or control the GPO object | Who is authorized to modify it?    |
| Security/WMI filtering | Helps determine which targets process the GPO        | Does it actually apply?            |

  

> [!warning] HINTS
> **CRTP mindset:** A potentially dangerous GPO path generally requires the combination of **control over the policy** and **a policy that affects an interesting target**. 
> 
> Neither condition alone proves an escalation.

