# Find file with `SUID` mechanism

<mark style="background: #BBFABBA6;">First of all we need to run:</mark>
```bash
find / -perm -u=s -type f 2 > /dev/null
```
- ![[Pasted image 20250117112018.png]]
- We get some results

We can see `systemctl`

# GTFOBins
We go on [https://gtfobins.github.io/](https://gtfobins.github.io/) and we search for the files we have with the `SUID` set:
- ![[Pasted image 20250117112355.png]]
- So `systemctl` is the one we want to use to escalate

Now we want to modify the script:
- we change the file to read into `/root/root.txt`
- we modify the `systemctl` path with the correct one `/bin/systemctl`
- ![[Pasted image 20250117112602.png]]


Now, we run the commands one by one:
- ![[Pasted image 20250117112632.png]]

So we gain the access to privileged file `root.txt` because we copied it into `/tmp/output`



# Steps to do SUID priv escalation

- [ ] **List files with the SUID bit set**
- `find / -perm -u=s -type f 2>/dev/null`

- [ ] **Verify if they are executable**
- `file filename`

- [ ] **Search in GTFOBins as `SUID`**
- [https://gtfobins.github.io/](https://gtfobins.github.io/)

- [ ] **Execute the commands given by GTFOBins**

- [ ] **Clean the history**
- `echo "" > ~/.bash_history`
