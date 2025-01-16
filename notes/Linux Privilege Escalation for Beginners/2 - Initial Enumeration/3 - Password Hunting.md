This process is used to see if we have access to passwords and sensitive files.

# Searching the word password in all files
A simple command we can use to search in all files the string `Password` is:
```bash
grep --color=auto -rnw '/' -ie "PASSWORD" --color=always 2> /dev/null
```
- it will color the results in red
- ![[Pasted image 20250115170934.png]]

It is difficult to find something but we can try with `PASSWORD=`
- ![[Pasted image 20250115171013.png]]
Useful is also `pass=`


# Searching for password in file names

In this case we want to search:
```bash
locate password | more
```
- ![[Pasted image 20250115171125.png]]

Or maybe `pass`:
- ![[Pasted image 20250115171143.png]]

# Find for authorized key files
In this case we can search:
```bash
find / -name id_rsa 2> /dev/null
```
- ![[Pasted image 20250115171346.png]]
- This could be very interesting
