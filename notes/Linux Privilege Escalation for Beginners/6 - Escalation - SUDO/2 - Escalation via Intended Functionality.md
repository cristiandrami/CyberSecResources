**==In this case we will exploit==** `sudo` **==leveraging on intended functionalities.==**

We start always from :
```bash
sudo -l
```
- ![[Pasted image 20250116153510.png]]


In this case we will leverage on `apache2` that is not present in `GTFOBins`.


So we search in google `apache sudo privilege escalation`:
- ![[Pasted image 20250116153606.png]]

In this case the idea is to access the `/etc/shadow` file with `apache2` in order to leak the hash value of `root` password:
```bash
sudo apache2 -f /etc/shadow
```
- ![[Pasted image 20250116153748.png]]



# TryHackMe resource Sunday

We can find it here:
- [https://veteransec.com/2018/09/29/hack-the-box-sunday-walkthrough/](https://veteransec.com/2018/09/29/hack-the-box-sunday-walkthrough/)


This is a machine that has a particular privilege escalation based on the using of `wget`:
- ![[Pasted image 20250116154015.png]]


The idea is to send privilege files in the post request to our malicious host:
- ![[Pasted image 20250116154114.png]]
- this on the attacker

Now from the victim that can run `wget` as `sudo`:
- ![[Pasted image 20250116154220.png]]


