Burp suite is used to intercept traffic.

# Firefox as browser on Burp
To add firefox as browser of burp suite we can go to `Preferences` -> `Network Proxy` -> `Settings` and set a manual proxy:
- ![[Pasted image 20250110171053.png]]


Now we need to visit `https://burp` to downlaod the certificate. 
We have to click on CA Certificate:
- ![[Pasted image 20250110171211.png]]


Once downloaded we go on `Preferences` -> `Privacy and Security` -> `View Certificates`:
- ![[Pasted image 20250110171303.png]]

At this point we need to upload burp certificate:
- ![[Pasted image 20250110171326.png]]


Now we can start to use burp.

# visiting websites with burp
We visit websites with the proxy browser and we can see them in the target section:
- ![[Pasted image 20250110171508.png]]

From there we can see all the content visited, which there is on the website and so on.

We can also see info in the responses:
- ![[Pasted image 20250110171605.png]]

