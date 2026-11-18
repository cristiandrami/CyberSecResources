What we do in this certification is feature abuse.

In windows many enterprise applications need their Administrative privileges or SYSTEM privileges making them a great target for privesc.

# Example Jenkins
On dcorp-ci (172.16.3.11) on port 8080 there is a old and vulnerable version of Jenkins. We will use it to run commands on the system.

In jenkins Maaster there are two ways of executing commands.
If you have admin access (defualt installation before 2.x) we can go to http://<jenkins_server>/script

And in the script conole we can execute groovy:
- ![[Pasted image 20261117164301.png|454]]

But if you don't have admin access but you can add or edit build steps in the buil configuration, you can add a build step "Execute Windows Batch Command" and enter:
- `powershell -c <command>`
	- so you can do everything like download scripts etc.


# Relaying 
In a relaying attack the target credentials are not captured.
Instead they are forwarded to a local or remote service or an endpoint for authentication.

There are two types based on auth:
- NTLM relaying
- Kerberos relaying

LDAP and AD CS (Certificate Service) are the two most abused services for relaying.