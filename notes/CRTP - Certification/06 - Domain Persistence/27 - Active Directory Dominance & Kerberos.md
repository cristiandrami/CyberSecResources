Once we have domain admin privileges new avenues of persistence, ecalation to Enterprise Admins and attacks across trust open up.

# Kerberos
Kerberos is the basis of authentication in a AD environment.

Clients need to obtain tickets from Key Distribution Center (KDC) which is a service running on the domain Controller.
- These tickets are basically the clients credentials.
- They are also a very interesting target of abuse.


## How it works

Let's say we want to access with our client to a server in the network that hosts a service:
1. the client sends a timestamp and the secret  (AES key or NTLM has) of the user to the KDC.
2. The KDC (Key Distribution Center) sends back a message called TGT (Ticket Granting), that is encrypted using the secret (aes key or ntlm hash) of the krbtgt account.
3. The TGT (Ticket Granting Ticket) message is used by the client to send a request to the KDC. The request can be a Service Ticket (ST) or a TGS (_Ticket Granting Service_) ticket.
4. The ST or the TGS is encrypted with the secret of the target service.
5. The client connect to the target service sending the ST or TGS.
6. The target service replies after checking the validity of the TGS or ST.

![[Pasted image 20261123144849.png|626]]
