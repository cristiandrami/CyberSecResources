In this case we want to hunting for ss keys in order to use them to escalate.

In order to do it we want to use `Payload all the things`.

First of all we want to search for for:
```bash
find / -name authorized_keys 2> /dev/null
```
-  but no results in this case

The we pass to:
```bash
find / -name id_rsa 2> /dev/null
```
- ![[Pasted image 20250116142649.png]]


When we setup a user in a `ssh` server we use `ssh-keygen` to generate a public and a private key.
- the public one is copied on the server ssh (in the authorized keys folder)
- the private one is stored on the client and is used to authenticate it on the connection 


An `id_rsa` is the private key used to authenticate on a `ssh` server.

So in this case we access to it:
- ![[Pasted image 20250116151616.png]]

At this point we copy it and in a new window we create a file `id_rsa` with that key:
- ![[Pasted image 20250116151659.png]]


So we can try to access to the server as root in order to understand if it is possible:
```bash
chmod 600 id_rsa
ssh -i id_rsa root@OUR_HOST_IP
```
- ![[Pasted image 20250116151826.png]]
- in this case we have done


Now as root if we access to the `.ssh` folder we can see the authorized keys (we can see that there is our public key not the `id_rsa`):
- ![[Pasted image 20250116151916.png]]

