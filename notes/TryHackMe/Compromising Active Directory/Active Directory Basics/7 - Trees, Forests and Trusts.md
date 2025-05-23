# Trees


Active Directory supports integrating multiple domains so that you can partition your network into units that can be managed independently.

If you have two domains that share the same namespace (`thm.local` in our example), those domains can be joined into a **Tree**.

If our `thm.local` domain was split into two subdomains for UK and US branches, you could build a tree with a root domain of `thm.local` and two subdomains called `uk.thm.local` and `us.thm.local`, each with its AD, computers and users:
- ![[Pasted image 20250128150038.png]]

This partitioned structure gives us better control over who can access what in the domain. The IT people from the UK will have their own DC that manages the UK resources only.

A new security group needs to be introduced when talking about trees and forests. 
- The **Enterprise Admins** group will grant a user administrative privileges over all of an enterprise's domains. Each domain would still have its Domain Admins with administrator privileges over their single domains and the Enterprise Admins who can control everything in the enterprise.


# Forests
The domains you manage can also be configured in different namespaces. 
The union of several trees with different namespaces into the same network is known as a **forest**.
- ![[Pasted image 20250128150252.png]]


# Trust Relationships
Having multiple domains organised in trees and forest allows you to have a nice compartmentalised network in terms of management and resources. To share resources, domains arranged in trees and forests are joined together by **trust relationships**.

In simple terms, having a trust relationship between domains allows you to authorise a user from domain `THM UK` to access resources from domain `MHT EU`.

The simplest trust relationship that can be established is a **one-way trust relationship**. 

In a one-way trust, if `Domain AAA` trusts `Domain BBB`, this means that a user on BBB can be authorised to access resources on AAA:
- ![[Pasted image 20250128150427.png]]


The direction of the one-way trust relationship is contrary to that of the access direction.


**Two-way trust relationships** can also be made to allow both domains to mutually authorise users from the other. By default, joining several domains under a tree or a forest will form a two-way trust relationship.

It is important to note that having a trust relationship between domains doesn't automatically grant access to all resources on other domains. 

Once a trust relationship is established, you have the chance to authorise users across different domains, but it's up to you what is actually authorised or not.