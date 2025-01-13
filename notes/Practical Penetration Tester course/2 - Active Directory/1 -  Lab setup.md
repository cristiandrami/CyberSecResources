# ISO needed
- Windows 10 enterprise 
- Windows Server 2025 (8 gb RAM)



# Change windows server name 
<mark style="background: #BBFABBA6;">On search windows bar digit name and change it</mark>
- ![[Pasted image 20250113121928.png]]
# Make Windows Server a Domain controller 

1. Go to `Server Manager`
2. `Manage` -> `Add Roles and features`
	1. ![[Pasted image 20250113122016.png]]
3. `Role-based and feature based`
	1. ![[Pasted image 20250113122034.png]]
4. Click on `AD Domain Services` -> `Add features`
	1. ![[Pasted image 20250113122059.png]]
	2. ![[Pasted image 20250113122110.png]]
5. Restart the destination server automatically if required
6. Click `Promote this server to a Domain controller`
	1. ![[Pasted image 20250113122144.png]]


Continue on next section...
# Add forest 
Click on `Add new forest` and use root domain "`CRISTIAN.local`" or what you want.
- ![[Pasted image 20250113122216.png]]


Now we need to set the password for `DSRM`.
- ![[Pasted image 20250113122245.png]]



# Add authentication with tickets (to attack later)

1. Go to `Server Manager`
2. `Manage` -> `Add Roles and features`
	1. ![[Pasted image 20250113122016.png]]
3. `Role-based and feature based`
	1. ![[Pasted image 20250113122034.png]]
4. Click on `AD Certificate Services` -> `Add features`
	1. ![[Pasted image 20250113122350.png]]
	2. ![[Pasted image 20250113122430.png]]
5. Restart the destination server automatically if required
6. `Configure AD Certificate services`
	1. ![[Pasted image 20250113122451.png]]
7. `Check Certification Authority`
	1. ![[Pasted image 20250113122519.png]]
8. All next, on Validity put 99

Restart Server


# Create host machine
We just need to install windows ISO and add a user.

We have to do it for all the pc we want to add to the domain.

We have also to rename the PC:
- ![[Pasted image 20250113122753.png]]
- the name can be what we want

# Modify Server DC

1. Go to `Tools` -> `Add AD users and computer`
	1. ![[Pasted image 20250113122847.png]]
2. on the domain on left click `New` -> `Organization Uni`t -> call it Groups
	1. ![[Pasted image 20250113122938.png]]
3. copy all users in `Users` into `Groups` (not Administrator and Guest)
	1. ![[Pasted image 20250113123000.png]]
	2. in this way the only user we can access on the domain is the Admin account

Create a new admin:
1. click on admin user and click `copy` 
	1. ![[Pasted image 20250113123104.png]]
2. give name and use initial name for login name -> password never expires
	1. ![[Pasted image 20250113123147.png]]
	2. ![[Pasted image 20250113123219.png]]

Create a SQL server account:
1. click on admin user and click `copy` 
	1. ![[Pasted image 20250113123104.png]]
2. give name "SQL" 
3. User login name use SQLServer
	1. ![[Pasted image 20250113123306.png]]
4. password never espires
	1. ![[Pasted image 20250113123336.png]]
5. double click on it and in description write "The password is PASSWORD"
	1. ![[Pasted image 20250113123353.png]]

It's very common people write password in it



Add a normal user:
1. create a user
	1. ![[Pasted image 20250113123432.png]]
	2. ![[Pasted image 20250113123451.png]]

Add another normal user.



# Change file and storage services

On the left in the `Server manager` click on `Files and storage services`.
1. ![[Pasted image 20250113123633.png]]

Click on `Shares` -> `Tasks` -> `New share` -> `Add a share name` "hackme" -> `Allow caching of share`
1. ![[Pasted image 20250113123645.png]]
2. ![[Pasted image 20250113123710.png]]



# Service account
Digit `cmd` -> as administrator

```
setspn -a domainname/SQLService.CRISTIAN.local:60111 CRISTIAN\SQLService
```
- domain name in the pc name of server

```
setspn -T CRISTIAN.local -Q */*
```
- if we see Existing SPN found everything is ok



# setup group policy

On menu search Group Policy
1. click on `CRISTIAN.local` -> `create GPO `
	1. ![[Pasted image 20250113123931.png]]
	2. ![[Pasted image 20250113123951.png]]
2. call `disable Windows Defender`
	1. ![[Pasted image 20250113124027.png]]
3. `Edit policy` created
	1. ![[Pasted image 20250113124050.png]]
4. go to `policies` -> `administrative templates `-> `windows components` -> `Microsoft Windows Defender Antivirus` -> `Turn off windows...` -> `enabled`
	1. ![[Pasted image 20250113124133.png]]
5. right click on policy `Enforced`
	1. ![[Pasted image 20250113124207.png]]


# check for ip
`cmd`-> `ifconfig`


go to `network` -> `change adapter options` -> `ipv4` -> put one static (the one retrieved by ifconfig)
1. ![[Pasted image 20250113124230.png]]



# access to the domain with windows 10 (add our host to the domain)

search for `Access work or school` -> `+ Connect` -> `access a local active directory` -> `CRISTIAN.local` -> `administrator:password`
1. ![[Pasted image 20250113124359.png]]
2. ![[Pasted image 20250113124440.png]]
3. ![[Pasted image 20250113124456.png]]
4. ![[Pasted image 20250113124505.png]]
5. 

Do the same thing for both vMs


## See if it worked 
Go to the server and `Tools` -> `AD Users and computers` -> `domain` -> `Computers`
- we must see the two
- ![[Pasted image 20250113124523.png]]



## setup the machines (host)
Go on search `Edit local users` -> on `users` see `administrator` right click and `set password`
1. ![[Pasted image 20250113124629.png]]
2. ![[Pasted image 20250113124700.png]]
3. ![[Pasted image 20250113124719.png]]


Then we have to double click on at and check `Account is disabled`:
- ![[Pasted image 20250113124856.png]]

==**Do this on both vms and the password must be the identical one**==
- `Passwordadmin`


Now go in the section `groups` -> `administrators`:
1. ![[Pasted image 20250113125019.png]]
2. ![[Pasted image 20250113125141.png]]
In the first machine **==add both 2 normal users==**

In the second machine **==add only one of them==**

(Use checknames to choose the correct one)
1. ![[Pasted image 20250113125320.png]]


Now go to `folders`, on the left `network` -> `enable network` on the popup
- do it in both machines
- ![[Pasted image 20250113125402.png]]
- ![[Pasted image 20250113125424.png]]




## login as the user john in machine (host with both users)
**==Change user on the machine with both users==** and login as the user of Active directory "John".
- ![[Pasted image 20250113125538.png]]
- 

## login to share space
At this point go to folders -> `this pc` -> `computer` on top -> `map network drive` -> in folder put:
- `\\JohnWick\hackmebastard`
1. ![[Pasted image 20250113125620.png]]
2. ![[Pasted image 20250113125649.png]]
	1. (connect using different credentials)
3. ![[Pasted image 20250113125716.png]]


