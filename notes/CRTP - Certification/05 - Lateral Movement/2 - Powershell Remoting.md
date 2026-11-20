PSRemoting uses Windows Remote Management (WinRM) which is Microsoft implementation of WS-Management.

It is enabled by default on Server 2012 and after with a firewall exception.

It uses WinRM and listens by default on 5985 (http) and 5986 (https).

It is the recommended way to manage Windows Core servers.

==🔴The remoting process is executed as high integrity process. So you get an elevated shell.==


## One to one
It can be One-to-One so we create a PSSession and we interact with one machine per time.

A PSSession is interactive and runs into a new process (wsmprovhost)
- It is stateful

Useful cmdlets are:
- `New-PSSession`
- `Enter-PSSession`

#### Example
- `Enter-PSSession -ComputerName dcorp-ci`
	- ![[Pasted image 20261119160748.png|445]]
## One to many
It could also be One-to-Many.
So you control multiple machine at one time.

It is known as Fan-out rempting and it is ==Non-Interactive==.
- You can execute commands parallely 

Useful cmdlets is:
- `Invoke-Command`

It is very useful to lateral movement.
- If you compromise an user and you want to use it in multiple machine you can use `Invoke-Command` to use the command on multiple machines parallely.


To execute commands or script blocks:
- `Invoke-Command -Scriptblock {<command ex: Get-Process>} -ComputerName (Get-Content <list_of_servers>)`

To execute scripts from files:
- `Invoke-Command -FilePath <script path ex: C:\Scripts\Get-PassHashes.ps1> -ComputerName (Get-Content <list_of_servers>)`

You can also run local functions on the remote machines:
- `Invoke-Command -Scriptblock ${function:<function ex: Get-PassHashes>} -ComputerName (Get-Content <list_of_servers>)`
	- if you add `-ArgumentList` you can pass also arguments


You can execute also "Stateful" commands using `Invoke-Command`:
```
$Sess = New-PSSession -Computername <name>
Invoke-Command -Session $Sess -ScriptBlock {$Proc = Get-Process}
Invoke-Command -Session $Sess -ScriptBlock {$Proc.Name}
```
- Useful to mantain the state of the connection


#### Example
- `Invoke-Command -Scriptblock {ls env:} -ComputerName dcorp-ci`
	- ![[Pasted image 20261119161325.png|444]]

To store the session in avariable:

```
$dcorpci = New-PSSesssion -ComputerName dcorp-ci
Invoke-Command -Session $dcorpci -ScriptBlock {Get-Process}
```
- ![[Pasted image 20261119161646.png|579]]


# WinRS
Powershell remoting supports system-wide transcripts and deep scrip block logging.
- so PSRemote could block our scripts 

We can use winrs in plce of PSRemoting to evade logging and still leverage on the benefit of 5985 allowed between hosts:
- `winrs -remote:<computer_name> -u:<computer_name\username ex:server1\administrator> -p:<password> <command>`


Example:
- `winrs -remote:dcorp-ci cmd /c "set computername && set username`
	- ![[Pasted image 20261119160639.png]]

You can also use `winrm.vbs` or COM object of WSMAN object [https://github.com/bohops/WSMan-WinRM]
