SMB is an important service used to file sharing.
It is generally on port `139` and is importnat to scan it in order to gain useful info.

It is generally used in work environments.

# Metasploit
We will use this tool to enumerate `smb`

To access to metasploit we have to use the console
```bash
msfconsole
```
- ![[Pasted image 20250111172950.png]]

To analyse `smb` we can use:
```bash
search smb
```
- ![[Pasted image 20250111173043.png]]




To start enumerating `smb` we have to use `auxiliary/scanner/smb/smb_version` :
```bash
use auxiliary/scanner/smb/smb_version
```
- ![[Pasted image 20250111173408.png]]

At this point we have to set the exploit in order to scan the victim IP:
```bash
set RHOSTS VICTIM_IP
run
```
- ![[Pasted image 20250111173543.png]]
- here we can see that the version is `smb 2.2.1a` (very specific)

**We have to write it into the notes (it's important).**


# testing for anonymus access
If `smb` is wrongly configured we can access to it as `anonymous` and have access to files.

To test it we can use `smbclient`:
```bash
smbclient -L \\\\VICTIM_IP\\
```
- `-L` lists all files
- `\\` at the end is important because otherwise will obtain no results

![[Pasted image 20250111173854.png]]
- without password we have listed the file shares

So try to connect to `ADMIN$` with
```bash
smbclient \\\\VICTIM_IP\\SHARE_NAME
```
- ![[Pasted image 20250111174003.png]]
- in this case we cannot acces without password

So try for `IPC$`:
- ![[Pasted image 20250111174047.png]]
- we are in!


But commands are not allowed for us.



## solving Protocol negotiation failed: NT_STATUS_IO_TIMEOUT

When we use `enum4linux` or `smbclient` and we obtain `Protocol negotiation failed: NT_STATUS_IO_TIMEOUT`

We have to follow this guide:
- On Kali, edit `/etc/samba/smb.conf`
- Add the following under global:
	- `client min protocol = CORE`
	- `client max protocol = SMB3`
