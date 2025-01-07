First of all we need to download the docker here  [https://cdn.fs.teachablecdn.com/CbIyLkOuS4GUH7TNFTFg](https://cdn.fs.teachablecdn.com/CbIyLkOuS4GUH7TNFTFg)

At this point we can setup it using these commands:
```bash
sudo apt update

sudo apt upgrade

sudo apt install docker.io

sudo apt install docker-compose
```

At this point we can restart our Kali linux.

Then we can run:
```bash
tar -xf peh-web-labs.tar.gz

cd labs

sudo docker-compose up
```


At this point we have a docker running on the machine.

The first time it will be slow becuase it needs to download some stuff.

One the databases are `ready to connections` then the container is setup:
- ![[Pasted image 20241223145319.png]]

Now we just need to setup the permissions to the server:
```bash
./set-permissions.sh
```

Finally we can browse to our localhost `https://localhost`

Each time we need to run the container we need to execute:
```bash
sudo docker-compose up 
```