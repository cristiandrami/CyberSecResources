It is a very powerful tool that can be downloaded from [https://www.tenable.com/downloads/nessus](https://www.tenable.com/downloads/nessus)

To install it we can just do:
```bash
dpkg -i Package.deb
```
- ![[Pasted image 20250111175704.png]]



To start it we just need to do:
```bash
/etc/init.d/nessusd start
```
- or what the installation says

Now we just need to go on:
```url
https://localhost:8834
```
- or what the installation said

At this point we just need to install the essentials one and activate it with the email.

At this point we just need to do the scans using the dashboard.
- ![[Pasted image 20250111175913.png]]

We can configure the scans as we want and then start them:
- ![[Pasted image 20250111180245.png]]
- ![[Pasted image 20250111180001.png]]


When it finishes it provides us a report
- ![[Pasted image 20250111180049.png]]

