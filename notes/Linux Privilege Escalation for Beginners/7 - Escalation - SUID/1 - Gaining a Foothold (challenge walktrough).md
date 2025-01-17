In this specific case we will do the `Vulnversity` machine on tryhackme:
- [https://tryhackme.com/r/room/vulnversity](https://tryhackme.com/r/room/vulnversity)


First of all we need to execute a `NMAP` scan:
- ![[Pasted image 20250117110305.png]]
- ![[Pasted image 20250117110329.png]]

So we try to visit first the port `3333` on the browser:
- ![[Pasted image 20250117110435.png]]



We use `dirsearch` to enumerate folders and files:
```bash
python dirsearch.py -u http://IP:3333/ -e html,php -x 400,401,403
```
- ![[Pasted image 20250117110748.png]]
- `/internal` seems interesting

We can see that we have a file upload functionality:
- ![[Pasted image 20250117110851.png]]


We download this reverse shell [https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)
- Remember to change IP and ports


Let's try to upload the `php` shell but it is not allowed `.php` extension, so we try with burp to bypass the block:

We send the request to the `intruder` and we fuzz on the extension:
- ![[Pasted image 20250117111253.png]]

In the payloads we put:
- ![[Pasted image 20250117111320.png]]

In `options` we filter on the error message:
- ![[Pasted image 20250117111404.png]]

We start the attack and we notice that `phtml` is allowed.
- we upload the file with `.phtml` extension


So we run a `nc` on our host with the port we insert in the shell file:
- ![[Pasted image 20250117111655.png]]


At this point we go on `/internal/uploads` and we find the file, we click on it and we connect in reverse shell:
- ![[Pasted image 20250117111718.png]]
- ![[Pasted image 20250117111742.png]]


We are in as `www-data` so we want to escalate.