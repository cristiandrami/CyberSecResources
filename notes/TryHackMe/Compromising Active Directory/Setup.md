# impacket

First of all we need to install `impacket`:


Required python > 3.7

First, you will need to clone the Impacket Github repo: 
```bash
git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket
```


To install the Python requirements for Impacket:

```bash
pip3 install -r /opt/impacket/requirements.txt
```

Once the requirements have finished installing, we can then run the python setup install script:

```
cd /opt/impacket/ && python3 ./setup.py install
```
  

**Installing Bloodhound and Neo4j**

Bloodhound is another tool that we'll be utilizing while attacking Attacktive Directory. 

```bash
sudo apt install bloodhound neo4j
```
 


  

# Troubleshooting

If you are having issues installing Bloodhound and Neo4j, try issuing the following command:

```bash
sudo apt update && sudo apt upgrade 
```