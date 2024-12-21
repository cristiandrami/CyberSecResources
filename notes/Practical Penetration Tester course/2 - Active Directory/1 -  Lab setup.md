# ISO needed
- Windows 10 enterprise 
- Windows Server 2025 (8 gb RAM)
	- 



# Change windows server name 
Digit name and change it

# Make Windows Server a Domain controller 

1. Go to Server Manager
2. Manage -> Add Roles and features
3. Role-based and feature based
4. Click on AD Domain Services -> Add features
5. Restart the destination server automatically if required
6. Click Promote this server to a Domain controller


# Add forest 
Cllick on Add new forest and use root domain "CRISTIAN.local" or what you want.

Now we need to set the password for DSRM.


# Add authentication with tickets (to attack later)

1. Go to Server Manager
2. Manage -> Add Roles and features
3. Role-based and feature based
4. Click on AD Ceertificate Services -> Add features
5. Restart the destination server automatically if required
6. Configure AD Certificate services
7. Check Certification Authority
8. All next, on Validity put 99

Restart Server



# Modify Server

1. Go to Tools -> Add AD users and computer
2. on the domain on left click new -> Organization Unit -> call it Groups
3. copy all users in Users into Groups (not Administrator and Guest)

Create a new admin:
1. click on admin and click copy 
2. give name and use initial name for login name -> password never espires

Create a SQL server account:
1. click on admin and click copy 
2. give name "SQL" 
3. User login name use SQLServer
4. password never espires
5. double click on it and in description write "The password is PASSWORD"

It's very common people write password in it



Add a normal user:
1. create a user

Add another normal user



# Change file and storage services

On the left in the Server manager click on `Files and storage services`.

Click on Shares -> Tasks -> New share -> Add a share name "hackme" -> Allow caching of share



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



# setup goup policy

On menu search Group Policy
1. click on CRISTIAN.local -> create GPO 
2. call disable Windows Defender
3. Edit policy created
4. go to policies -> administrative templates -> windows components -> Microsoft Windows Defender Antivirus -> enabled
5. right click on policy Enforce


# check for ip
`cmd`-> `ifconfig`


go to network -> change adapter options -> ipv4 -> put one static (the one retrieved by ifconfig)



# access to the domain with windows 10

search for `Access work or school` -> + -> access a local active directory -> `CRISTIAN.local` -> `administrator:password`

Do the same thing for both vMs


## See if it worked 
Go to the server and Tools -> AD Users and computers -> domain -> Computers
- we must see the two



## setup the machines
Go on search `Edit local users` -> on `users` see `administrator` right click and set password

==**Do this on both vms and the password must be the identical one**==


Now go in the section `groups` -> `administrators`:
1. in the first machine add both 2 normal users
2. in the other one add only one of them

(Use checknames to choose the correct one)

Now go to folders, on the left network -> enable network on the popup
- do it in both machines


## login as the user john in machine
Change user on the machine with both users and login as the user of Active directory "John".

## login to share space
At this point go to folders -> `this pc` -> `computer` on top -> `map network drive` -> in folder put:
- `\\JohnWick\hackmebastard`

