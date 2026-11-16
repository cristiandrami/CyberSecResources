
# Active directory.
Active directory is a Directory Service used to manage Windows networks.
It enables centralized, secure management of an entire network, which might span a building, a city or multiple locations."

Everything is defined as an object.

It looks like:
- ![[Pasted image 20261116094941.png|415]]


## Components 
The components of active directory are:
- ==Schema==, that defines object and their attributes
- ==Query and index mechanism==, that provides searching and pusblication of objects and their properties.
- ==Global catalog==, that contains information about every object in the directory.
- ==Replication Service==, that distributes info across domain controllers.


## Structure

Forests, domains and organizational units (OU) are basic building blocks of any activity directory structure.

A forest, which is a security boundary, may contain multiple domains and each domain may contain multiple OU.
- ![[Pasted image 20261116100326.png]]