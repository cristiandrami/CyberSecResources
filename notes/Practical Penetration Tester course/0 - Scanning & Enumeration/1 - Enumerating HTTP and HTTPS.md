The first thing to do is to use Burp suite, in order to caputre the traffic.

Basically we have to analyze the website in http and also in https in order to understand if bot are used.

# Navigating trough the website
One important thing is to navigate and use all the functionalities of the website.

This because generally we can retrieve a lot of information and in some cases also information leakages as in this example:
- ![[Pasted image 20250111170014.png]]
- here we can see that there is `Apache/1.3.20`


# Scanning with nikto
Nikto is a tool used to perform web vulnerabilities scanning:
```bash
nikto -h website_to_analyze
```
- ![[Pasted image 20250111170223.png]]


In general if the website has a good security of firewalls then it could be unuseful at all, because it could be blocked.


# Directory busting (find files and dirs)
This is a very useful activity to do, to understand which files and directories are accessible on the website using a wordlist.

# Dirbuster
Dirbuster is a tool created by OWASP. It uses a bruteforce approach using a wordlist:
- ![[Pasted image 20250111171014.png]]
- **important: we have to add the port at the end of the Target URL**

Generally we can use the `/usr/share/wordlists/dirbuster/directory-list2.3-small.txt` wordlist and file extension as `php,txt,html,zip,rar,pdf,docx,csv,xlsx` and so on.

More file extensions means more time to scan.

It can give us interesting results:
- ![[Pasted image 20250111171101.png]]


# Look at comments in the page
Another important thing is to have a look on the html source code of the pages we are analising because sometimes we can find interesting things:
- ![[Pasted image 20250111171216.png]]



# Using burp
Burp is very important because with it we can access to the requests and the responses of our HTTP communications with the server.

It sometimes give us information:
- ![[Pasted image 20250111171347.png]]
- here the server version

In addition we can manipulate the requests and test for different inputs.

