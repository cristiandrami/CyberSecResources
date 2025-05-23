The core of any Windows Domain is the **Active Directory Domain Service (AD DS)**. This service acts as a catalogue that holds the information of all of the "objects" that exist on your network.

# Objects 

## Users
Users are one of the most common object types in Active Directory. 

Users are one of the objects known as **security principals**, meaning that they can be authenticated by the domain and can be assigned privileges over **resources** like files or printers.

Users can be used to represent two types of entities:
- **People:** persons in the organisation that need to access the network, like employees.
- **Services:** like IIS or MSSQL. Every single service requires a user to run, but service users are different from regular users as they will only have the privileges needed to run their specific service.


## Machines 
For every computer that joins the Active Directory domain, a **machine object will be created**. Machines are also considered "security principals" and are assigned an account just as any regular user.

The machine accounts themselves are local administrators on the assigned computer.

If you have the password, you can use it to log in, but they are supposed to don't be accessed.

They follow a specific naming scheme.
For example, a machine named `DC01` will have a machine account called `DC01$`.

## Security groups
If you are familiar with Windows, you probably know that you can define user groups to assign access rights to files or other resources to entire groups instead of single users.

Groups can have both users and machines as members. If needed, groups can include other groups as well.

Several groups are created by default in a domain that can be used to grant specific privileges to users.


Some of the most important are:
- ![[Pasted image 20250128120440.png]]




## Active directory Users and Computers
To configure users, groups or machines in Active Directory, we need to log in to the Domain Controller and run "Active Directory Users and Computers" from the start menu:
- ![[Pasted image 20250128120802.png]]



This will open up a window where you can see the hierarchy of users, computers and groups that exist in the domain. 

These objects are organised in **Organizational Units (OUs)** which are container objects that allow you to classify users and machines. 

Checking our machine, we can see that there is already an OU called `THM` with four child OUs for the IT, Management, Marketing and Sales departments. 

It is very typical to see the OUs mimic the business' structure, as it allows for efficiently deploying baseline policies that apply to entire departments. 
![[Pasted image 20250128121054.png]]

If you open any OUs, you can see the users they contain and perform simple tasks like creating, deleting or modifying them as needed. You can also reset passwords if needed (pretty useful for the helpdesk):
- ![[Pasted image 20250128121131.png]]


You can notice there are other default containers apart from the THM OU. 
These containers are created by Windows automatically and contain the following:
- **Builtin:** Contains default groups available to any Windows host.
- **Computers:** Any machine joining the network will be put here by default.
- **Domain Controllers:** Default OU that contains the DCs in your network.
- **Users:** Default users and groups that apply to a domain-wide context.
- **Managed Service Accounts:** Holds accounts used by services in your Windows domain.

## Security Groups vs OUs
While both are used to classify users and computers, their purposes are entirely different:
- **OUs** are handy for **applying policies** to users and computers, which include specific configurations that pertain to sets of users depending on their particular role in the enterprise. Remember, a user can only be a member of a single OU at a time.
- **Security Groups**, on the other hand, are used to **grant permissions over resources**. 