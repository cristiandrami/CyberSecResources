These attacks are used to break the authentication and to be recognized from the server as legitimate and authenticated users.

In general we try to steal passwords and to bypass logins.
# Brute force Auth 0x01
In this case we try to bruteforce the password of a known user on the sever.

This is possible if the server doen's implement anti-brute force mechanisms, such as disabling the user or stopping the login attempt for a certain period of time.

## Using burp
First of all we need to send the request in burp to the Intruder and select the password as target:
- ![[Pasted image 20250106163323.png]]
- using `add` on the right

On the payload we add a wordlist:
```text
/usr/share/seclists/Passwords/xato-net-10-million-passwords-1000.txt
```
- this is the common one
- or also the oen with `100000`, but is slower

At this point we look at the responses and we filter then based on the length:
- ![[Pasted image 20250106163614.png]]



So we use it and we login:
- ![[Pasted image 20250106163642.png]]


## Usinf ffuf
First of all we need to copy the `POST`request:
- ![[Pasted image 20250106163748.png]]
- we put in password field `FUZZ`

At this point we can use the command:
```bash
ffuf -request request.txt -request-proto http -w /usr/share/seclists/Passwords/xato-net-10-million-passwords-1000.txt  
```


But we need to filter on the size looking at the results:
- ![[Pasted image 20250106164021.png]]

==**So the final command is:**==
```bash
ffuf -request request.txt -request-proto http -w /usr/share/seclists/Passwords/xato-net-10-million-passwords-1000.txt -fs SIZE  
```
- in this case `1814`
- ![[Pasted image 20250106164112.png]]


We can also use `hydra`.

# Auth 0x02

Here we want to login as `jeremy` and we have the credentials `jessamy:pasta`

After login with the password it asks us for the MFA code.
- ![[Pasted image 20250106173747.png]]



So if we follow the link we retrieve the `jassamy` MFA code:
- ![[Pasted image 20250106173835.png]]


It is not so long, just 6 numbers. So potentially we can bruteforce it.

However if we use it we can login easily...


The idea here is to open burp and the proxy intercept, login as `jessamy` and then analyze the request done when we put the MFA code:
- ![[Pasted image 20250106174204.png]]


So we just try to change `jessamy` to `jeremy` to see what happens.
- ![[Pasted image 20250106174238.png]]


So we forward the request and we can see that we login as jeremy...
- ![[Pasted image 20250106174314.png]]
In the past this vulnerability was very common. Now not so much!




# Challenge 0x03

In this specific case we have a protection mechanism for the brute-force, in fact the account will be locked after 5 attempts.
- ![[Pasted image 20250106174649.png]]

## Using ffuf
We will use `ffuf` to perform the attack, but we need to get the response length to filter the results.

First step we create a file with 4 passwords (common ones):
```text
123456
password
letmein
teashop
```

At this point we copy the login request in a file using `FUZZUSER` for the username and `FUZZPASS` for the pasword:
- ![[Pasted image 20250106175103.png]]


So we use:
```bash
ffuf -request request.txt -request-proto http -mode clusterbomb -w password.txt:FUZZPASS -w /usr/share/seclists/Usernames/top-usernames-shortlist.txt:FUZZUSER -fs SIZE
```
- this `mode` takes the first username and tests all the password on it, the second username and so on
- we can perform it also in burp with the intruder cluster bomb

In our case looking at burp we can see the length of the wrong response:
```shell
ffuf -request request.txt -request-proto http -mode clusterbomb -w password.txt:FUZZPASS -w /usr/share/seclists/Usernames/top-usernames-shortlist.txt:FUZZUSER -fs 3376
```


In this case we obtain some wrong results with size `3256`, but we can see a result of length `3378`:
- ![[Pasted image 20250106175717.png]]

And we login as admin:
- ![[Pasted image 20250106175735.png]]

Now we can do it with burp. Let's reset the db using `localhost/init.php`


## Burp solving
We go on the intruder and we add 2 target on username and password, using `Cluster bomb`:
- ![[Pasted image 20250106180004.png]]


At this point we put in the payload the usernames in the `payload set 1`:
- ![[Pasted image 20250106180156.png]]
- ![[Pasted image 20250106180103.png]]

Going on `payload set 2` we set the passwords:
- ![[Pasted image 20250106180222.png]]
- ![[Pasted image 20250106180242.png]]


Here we filter on the length and we have done:
- ![[Pasted image 20250106180331.png]]

