Now we have compromise a user, so what we can do?

# ldapdomaindump
This tool is used to enumerate the domain from internally.

We have to create first a dir in our attacker machine to collect data:
```bash
mkdir domain.name
```

Then we can run the command:
```bash
sudo ldapdomaindumo ldaps://DC_IP -u 'DOMAIN\username' -p Password 
```


Now in the folder we have a lot of information.
==**Take a look on the descriptions of the users**==


# bloodhound

First of all we need to install it:
```bash
sudo pip install bloodhoud
```

Then we run neo4j that is required to run bloodhoud:
```bash
sudo neo4j console
```
- username and password are `neo4j:neo4j` 
- at first login we need to change it `neo4j1` is ok


So now we can run bloodhoud:
```bash
sudo bloodhoud 
```

Then on `localhost:7474`  we  have a console.

At this point we need to run another command into a folder:
```bash
sudo bloodhoud-python -d DOMAIN -u username -p Password -ns DOMAIN CONTROLLER IP -c all  
```
- `-c` is to collect all


**==Now we take all the files created by the command and we put it on the web interface.==**


# plumhoud
We can get it from github:
- https://github.com/PlumHound/PlumHound

Once cloned it we need all requirements with:
```bash
sudo pip3 install -r requirements.txt
```


At this point we can use it:
```bash
sudo python3 PlumHoud.py -x --easy -p neo4j1
```
- so we need to start neo4j first

To get more information we need to execute:
```bash
sudo python3 PlumHoud.py -x tasks/default.tasks -p neo4j1
```


At this point we have a report in the `reports` folder.


#
