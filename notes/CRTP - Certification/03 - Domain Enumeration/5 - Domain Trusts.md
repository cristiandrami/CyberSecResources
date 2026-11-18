In AD a domain trust is a relationship between two domains or forsest which allows users of one domain/frest to access resources on the other domain/forest.

Trust can be
- automatic (parent-child, same forest etc)
- established (forest, external)

Trusted domain objects (TDOs) represent the trust relationships in a domain.

## trust directions
There are 2 types of directions:
- one way direction (unidirectional)
	- users in a domain can access resources of another domain but the reverse is not true
	- ![[Pasted image 20261117104506.png|425]]
- two way direction
	- users in a domain can access resources of another domain and reverse
	- ![[Pasted image 20261117104516.png|432]]



## trust transitivity
A transitive trust can be extended to establish trust relationships with other domains.
- all the defaault intra-forest trust relationships (tree-rotot, parent-child) between domains within a same forest are transitive two-way trust.

A non transitive trust cannot be extended to other domains in the forest. It can be two or one way.
- this is by default (external trust) between two domains in different forests when forests do not have a trust relationship.
- 
![[Pasted image 20261117104821.png|373]]
- domain A is 2 way trusted with B and B with C, so C is two way trusted with A by transitivity


## Default / Automatic Trusts
The Parent-Child trust is created automatically between the new domain and the domain that precedes it in the namespace hierarchy, whenever another domaain is added in the tree.
- dollarcorp.moneycorp.local is child of moneycorp.local

The tree-root trust is created automatically between whenever a new domain tree is added to a forest root. This is alway two way transitive.

![[Pasted image 20261117105331.png|617]]
- if we compromise C we can move to each one other domain, since everything is two way and we have transitivity.


## External trusts
An external trust happens we have relations between two domains in different forests when forests don not have a trust relationship.
- it can be one or two way but is **NONTRANSITIVE**

![[Pasted image 20261117105627.png|463]]
- if C is compromised, an attacker cannot go to Z
- Z can go to B but not to A or C

### Forest trusts
A forest trust can be done between forest root domain.

It cannot be extended to a third forest (no implicit trust here).

It can be one  or two way.

![[Pasted image 20261117110315.png|627]]
- here there is no trust between Forest 1 and forest 2 (no implicit trust).


# Forest enumeration
To see al the domains in the current forest we can use:
- `Get-ForestDomain -verbose`
# Enumeration of trusts
to enumerate the trusts of a domain we can use:
- `Get-DomainTrust`
- `Get-DomainTrust -Domain <target-domain>`

Or Ad module:
- `get-AdTrust`
- `Get-ADTrust -identity <target-domain>`


## enumerate an external trust
To enumerate an external trust we can do:
- `Get-DomainTrust`
	- If in the result you found `TrustAttribute FILTER_SIDS` then it's an external trust.
	- ![[Pasted image 20261118100023.png|530]]
