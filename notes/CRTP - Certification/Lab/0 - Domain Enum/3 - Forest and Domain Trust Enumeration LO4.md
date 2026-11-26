

> [!info] Learning objectives
> 1. Enumerate all domains in the `moneycorp.local` forest.
> 2. Map the trusts of `dollarcorp.moneycorp.local`.
> 3. Examine trusts involving other forests/domains.
> 4. Understand what information we can and cannot enumerate across a trust boundary.


## 1. Prepare our PowerShell environment

We begin on our Student VM and load the lab environment and PowerView:

```powershell
. C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat

. C:\AD\Tools\PowerView.ps1
```
  
## 2. Enumerate all domains in the forest

Now we need to understand the forest structure:

```powershell
Get-ForestDomain -verbose
```

![[Pasted image 20261118095321.png]]


The results indicate three domains:
- `moneycorp.local` — forest-root domain.
- `dollarcorp.moneycorp.local` — a child domain.
- `us.dollarcorp.moneycorp.local` — another domain beneath `dollarcorp`.

> [!note] 
> Una foresta può contenere più domini collegati da relazioni di trust. 
> 
> Con `Get-ForestDomain` recuperiamo i domini noti alla foresta corrente. 
> 
> Distinguiamo il root domain dai child domain perché gruppi, SID e alcuni ruoli amministrativi possono avere scope diversi.


> [!warning] HINTS
> When an interesting account or group is not found in our current domain, first check **which domain owns it**. 
> 
> This also matters for SID interpretation and for investigating whether a path crosses a domain boundary.


## 3. Inspect trusts of dollarcorp.moneycorp.local
  
Next, we want to map the relationships involving our current domain.

```powershell
Get-DomainTrust -Domain "dollarcorp.moneycorp.local"
```

![[Pasted image 20261118095706.png|538]]


We inspect the trusted domain, direction, type and attributes of each returned relationship.


> [!note] 
> Enumeriamo i trust del dominio `dollarcorp.moneycorp.local`. 
> 
>Per interpretare un trust servono più campi: tipo, direzione e attributi. Vedere un dominio nell'elenco non prova di poter autenticarsi o amministrarlo.


> [!warning] HINTS
> A useful trust analysis asks **which side trusts which other side**, not merely whether a line exists between domains. 
> 
> A trust can allow authentication pathways, but permissions on the target are a separate requirement. 
> 
> Direction, trust type, SID filtering and selective authentication can change the practical meaning of a path.


## 4. Look for trusts with SID filtering

Now we want to inspect trust attributes and identify relationships that may have SID filtering enabled.

The command iterates through the domains in our forest and applies a narrow filter:

```powershell
Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name} | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
```


![[Pasted image 20261118100451.png]]

This returned one matching relationship.


**Important correction:** `FILTER_SIDS` describes a **trust attribute associated with SID filtering**, not a universal definition of an external trust. Also, an equality test can miss trust records whose attributes contain multiple flags.

A broader inspection is easier to interpret:

```powershell
Get-ForestDomain | ForEach-Object {
    Get-DomainTrust -Domain $_.Name
} | Select-Object SourceName, TargetName, TrustDirection, TrustType, TrustAttributes
```


We should review the full results before classifying a relationship.

> [!note] 
> Negli appunti cercavamo solo i trust con `FILTER_SIDS`. 
> 
> Ma questo valore non significa automaticamente “external trust”: SID filtering è una proprietà di sicurezza che può essere presente su determinate relazioni di trust. 
> 
> Inoltre `TrustAttributes` può contenere più flag, per cui il filtro con `-eq` rischia di escludere risultati utili. È meglio osservare prima la tabella completa.

  

> [!warning] HINTS
> In a cross-domain BloodHound path, we should inspect **trust direction and security restrictions** before considering it viable. 
> 
> An interesting relationship may be stopped by SID filtering, selective authentication, missing permissions or simple network reachability. 
> 
> Don't treat a trust edge as proof of lateral movement.


## 5. Investigate the trust with eurocorp.local

Next, we want to examine the trust involving the separate `eurocorp.local` forest. 

```powershell
Get-DomainTrust -Domain eurocorp.local | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
```

![[Pasted image 20261118101032.png]]


> [!note]
> Stiamo interrogando i trust dal punto di vista di `eurocorp.local`. 
> 
> Il filtro continua a cercare solo gli attributi uguali a `FILTER_SIDS`, quindi non dimostra che altri trust siano assenti. 
> 
> La possibilità di interrogare l'altra foresta dipende anche dalla connettività e dai permessi a disposizione.

> [!warning] HINTS
> If the goal is to map cross-forest relationships, separate **trust discovery** from **remote enumeration**. 
> 
> A trust may exist even when the remote domain cannot be enumerated using our current account or from our current host.

## 6. Attempt to enumerate domains in eurocorp.local

Now we attempt to query the other forest and enumerate trust relationships for its domains

```powershell
Get-ForestDomain -Forest eurocorp.local | %{Get-DomainTrust -Domain $_.Name}
```

![[Pasted image 20261118101154.png|592]]

It produces an error and shows us only the relation with our domain  `dollarcorp.moneycorp.local` and his child `eu.eurocorp.local`

**This because there is no transitivity trust with external forests, so we can see only the relation with us.**

  

## 7. Understand forest security boundaries

Finally, we need to understand the meaning of a forest boundary.

![[Pasted image 20261118101345.png|498]]

An Active Directory forest is a fundamental security boundary. 

Trusts can enable selected authentication and resource access between different security scopes, but they do not automatically merge their authorization models.
  

> [!note] 
> La foresta costituisce un confine di sicurezza importante in Active Directory. 
> 
> Anche in presenza di trust, l'accesso effettivo dipende da autorizzazioni e impostazioni di sicurezza. 
> 
> Non possiamo concludere che il controllo di un dominio comporti automaticamente il controllo dell'altra foresta.

  

> [!warning] HINTS
> **CRTP mindset:** We should draw two maps, not one: 
> (1) the **trust topology** showing where authentication relationships exist; 
>
> (2) the **privilege graph** showing what our account can actually control. Escalation hypotheses become meaningful only where the two maps intersect under verified conditions.

