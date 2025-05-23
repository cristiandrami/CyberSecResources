Let's imagine you have to change the organization structure to fit this chart:
- ![[Pasted image 20250128122106.png]]

# Deleting extra OUs and users
The first thing you should notice is that there is an additional department OU in your current AD configuration that doesn't appear in the chart. 

If you try to right-click and delete the OU, you will get the following error:
- ![[Pasted image 20250128122246.png]]

By default, OUs are protected against accidental deletion. To delete the OU, we need to enable the **Advanced Features** in the **View** menu:
- ![[Pasted image 20250128122316.png]]

This will show you some additional containers and enable you to disable the accidental deletion protection:
- ![[Pasted image 20250128122512.png]]
- ![[Pasted image 20250128122352.png]]

So now we can delete the Organizational Unit:
- ![[Pasted image 20250128122606.png]]


# Delegation

You can give to specific users some control over some OUs. 

This process is known as **delegation** and allows you to grant users specific privileges to perform advanced tasks on OUs without needing a Domain Administrator to step in.

One of the most common use cases for this is granting `IT support` the privileges to reset other low-privilege users' passwords. 

For this example, we will delegate control over the Sales OU to Phillip. 
To delegate control over an OU, you can right-click it and select **Delegate Control**:
- ![[Pasted image 20250128123043.png]]
- ![[Pasted image 20250128123109.png]]

Now we can give the permission to reset passwords:
- ![[Pasted image 20250128123141.png]]



At this point we can connect to `phillip` account usign this command in the powershell and using the tryhackme ip:
```bash
mstsc
```
- ![[Pasted image 20250128125046.png]]
	- ![[Pasted image 20250128125108.png]]
	- ![[Pasted image 20250128125155.png]]
- ![[Pasted image 20250128124207.png]]


Now we change sophie password using the powershell:
```bash
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose
```
- ![[Pasted image 20250128124928.png]]

And force sophie to change it on the next login:
```bash
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose
```
- we set `Password123!`

Now we access as sophie:

```bash
mstsc
```
- ![[Pasted image 20250128125046.png]]
	- ![[Pasted image 20250128125108.png]]
	- ![[Pasted image 20250128125628.png]]

