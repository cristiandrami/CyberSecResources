
## What is?
Directory service is developed to manage windows domain networks.
It stores information related to objects (computers, users, printers).

The authentication is based on Kerberos tickets.
- Also non windows devices (linux hosts, firewalls etc) can authenticate to Active directory via RADIUS or LDAP

Basically it is a identity management service.

## Why?
It is the most commonly used identity management service in the world.

It can be exploited also without attacking patchable exploits.
- we can abuse features, trusts, components and more


# Components of AD

## Physical Components

### Domain controller
The domain controller is a server with  that is used to manage al the settings of the Active Directory.

It:
- hosts a copy of the AD DS directory store
- provides the authentication and the authorizations
- replicates updates to the other domain controllers in the domain
- allows administrative access to manage user accounts and network resources

### Data Store
It contains all the database files and processes that store and manage directory information for users, services and applications.

It is a single file `Ntds.dit` stored by default in `%SystemRoot%\NTDS` folder on all domain controllers.

It is accessible only through  the domain controller processes and protocols.

## Logical Components

### AD Schema
It defines all the type of objects that can be stored in the directory.
It enforces rules regarding object creation and configuration.
![[Pasted image 20241216144058.png]]

### Domains
They are used to group and manage objects in the organization.
1. to apply policies to groups of objects 
2. to replicate data between domain controllers
3. to limit the scope of access to resources (authentication and authorization)

### Tree
A domain tree is a hierarchy of domains in AD.

All domains in a tree:
- share a contigous namespace with the parent domain
- can have additional child domains
- create two-way transitive trust with other domains
![[Pasted image 20241216145204.png]]


### Forest 
A forest is a collection of one or more domains.

A Forest:
- share a common schema
- share a common configuration partition 
- share a common global catalog to enable searching
- enable trusts between all domains in the forest 
- share the `enterprise admins` and `schema admins group`


### Organizational Units (OUs)

They are containers that can contain users, groups, computers and other OUs.

Are used to:
- represent the organization hierarchically and logically 
- manage a collection of objects in a consistent way 
- delegate permissions to administer groups of objects
- apply ploicies


### Trusts 
Trusts are used to allow users to gain access to resources in other domains.

*For example*:
- all domains in a forest trust all domain in the forest
	- trusts can be extended also out of forest

![[Pasted image 20241216150023.png]]


### Objects 
Objects are the organizational units:
- ![[Pasted image 20241216150240.png]]