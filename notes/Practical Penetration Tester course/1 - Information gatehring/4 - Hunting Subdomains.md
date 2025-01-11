In this case we are searching to get all subdomains of a domain to perform a enumeration.

# Sublist3r
First of all we can use `sublist3r` that is a useful tool we can install with 
```bash
sudo apt install sublist3r
```

Then we can use it just running:
```bash
sublist3r -d DOMAIN
```
- ![[Pasted image 20250110165208.png]]



# crt.sh
We can also use `crt.sh`
- ![[Pasted image 20250110165657.png]]
- ![[Pasted image 20250110165706.png]]


The website can be accessed through [https://crt.sh/](https://crt.sh/)

# OWASP Amass
We can download it on github:
- [https://github.com/owasp-amass/amass](https://github.com/owasp-amass/amass)
It is a poweful tool that can do a lot of things but it requires a lot of time.

# Analyzing results
## sublist3r
As we can see there are some interesting results:
- ![[Pasted image 20250110165929.png]]
- like `sso-dev.tesla.com` `www-test.tesla.com` `www-dev.tesla.com`

They are very very important.

To make it faster we can specify `-t` for the threads.
```bash
sublist3r -d DOMAIN -t 100
```
- ![[Pasted image 20250110170206.png]]


# OWASP Amass
We can download it on github:
- [https://github.com/owasp-amass/amass](https://github.com/owasp-amass/amass)
It is a poweful tool 