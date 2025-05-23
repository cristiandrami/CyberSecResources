By default, all the machines that join a domain (except for the DCs) will be put in the container called "Computers". 

If we check our DC, we will see that some devices are already there:
- ![[Pasted image 20250128140750.png]]

We can see some servers, some laptops and some PCs corresponding to the users in our network.

While there is no golden rule on how to organise your machines, an excellent starting point is segregating devices according to their use. 
- In general, you'd expect to see devices divided into at least the three following categories:

**1. Workstations**
This is the device users will use to do their work or normal browsing activities. 
These devices should never have a privileged user signed into them.  

**2. Servers**
Servers are generally used to provide services to users or other servers.

**3. Domain Controllers**
Domain Controllers allow you to manage the Active Directory Domain. 
These devices are often deemed the most sensitive devices within the network as they contain hashed passwords for all user accounts within the environment.

Since we are tidying up our AD, let's create two separate OUs for `Workstations` and `Servers` (Domain Controllers are already in an OU created by Windows). 

We will be creating them directly under the `thm.local` domain container. In the end, you should have the following OU structure:
- ![[Pasted image 20250128141037.png]]
- ![[Pasted image 20250128141111.png]]

Now we move the personal computers in `workstations` and the server computers in `servers`
- ![[Pasted image 20250128141328.png]]
- ![[Pasted image 20250128141250.png]]




