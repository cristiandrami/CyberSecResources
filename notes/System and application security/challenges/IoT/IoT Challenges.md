### Cristian Domenico Dramisino (12338532)

## Introduction

The challenge consists of 3 parts, where in each part we have to retrieve a flag.

For these challenges we will use some interesting tools:
- [Mosquitto](https://github.com/eclipse/mosquitto)  ( focusing more on  `mosquitto_sub`  ) 
- [Wireshark](https://www.wireshark.org/)
- [TSDuck](https://tsduck.io/)

# Challenge part 1 

## Introduction

This challenge can be accessed through this [link.](https://sas.hackthe.space/#/challenges/IoT%20part%201)

From the description of the challenge we can retrieve some useful information that we could take in mind:
```
Ben is an Airbnb host in the wonderful city of Bergen, Norway. He is in his 60s and decided to install a super technological door lock working over MQTT (mosquitto.sas.hackthe.space, port 8883)! 
He secured it following the top security practices. He had a problem with the [firmware](https://owncloud.tuwien.ac.at/index.php/s/k3njyCirLnmNnGU) (password: mqtt_chal_1) and posted it on a forum. 

One day he received a complaint from one of his guests saying that someone had opened the door and stolen all their belongings! With utter surprise, Ben discovered it was a previous malicious guest! 

Can you help Ben find out what happened? **WILL** he be able to solve the problem? 

Hint: The user was able to find the flaw quickly, detailed reversing is not necessary.
```
- There is a MQTT server on the domain **mosquitto.sas.hackthe.space on the port 8883**
- The MQTT server **is secured with "top security practices"**.
- We could download the firmware from this [link](https://owncloud.tuwien.ac.at/index.php/s/k3njyCirLnmNnGU) using the password **mqtt_chal_1**
- The **firmware was posted on a forum** so the attacker could have obtained some information from it in order to understand how to open the door.


## Connecting to the MQTT broker 
We know that there is a MQTT broker on  **mosquitto.sas.hackthe.space on the port 8883** so what we can try to do is to connect to it.

In order to do that we need to install `mosquitto`. To install it we just need to follow the guide on the official website -> [Download guide](https://mosquitto.org/download/)


MQTT is a protocol that follows the architecture Publish/Subscribe. 
This means that it works on topics on which clients:
- can be subscribed, and so can read all the messages published on them
- can publish messages that will be send to all clients subscribed

What we want to do there is to try to subscribe to all topics in order to see if we could get some useful information useful for the flag getting.



So we could try to use this command:
```shell
mosquitto_sub -h mosquitto.sas.hackthe.space -p 8883 -t '#'
```
- `-h` is used to define the the `hostname` on which we want to connect, in this case `mosquitto.sas.hackthe.space`
- `-p` is used to define the the `port` on which we want to connect, in this case `8883`
- `-t` is used to define the the `topic` on which we want to subscribe
	- since we don't know which topics are present on the MQTT server, we use the wildcard `#` that allows us to subscribe to all the topics



Unfortunately we receive an error message that says:
`Error: A TLS error occurred.`

This means that probably we have not the authorization to connect to the MQTT server. 
So maybe, we need a valid certificate to connect to server, because we know that the MQTT server **is secured with "top security practices"**. 


At this point we could try to analyze the firmware file we can download from this [link](https://owncloud.tuwien.ac.at/index.php/s/k3njyCirLnmNnGU) using the password `mqtt_chal_1`


## Firmware analysis

What we can start to do is to open the firmware file that is a `.bin` file using a text editor, to see if there are useful information on it.

![[Pasted image 20240419152155.png]]


We can see that there are some log information and it is pretty readable, but to make it more readable we could use this command:
```shell
strings firmware_2024.bin > firmware_strings.txt
```
- `strings` is a command that searchs in files or inputs sequences of bytes that represent a valid string 


So now we have a readable file, and we can start to analyze it.
If we analyze it in a detailed way we can see that it contains an interesting line:
```
-----BEGIN CERTIFICATE----- MIIDnzCCAoegAwIBAgIUV8tXyfkAERUe4ajvN9S1AiQU3RowDQYJKoZIhvcNAQEL BQAwXjELMAkGA1UEBhMCQVQxDTALBgNVBAgMBFdpZW4xDTALBgNVBAcMBFdpZW4x DzANBgNVBAoMBlRVV2llbjEMMAoGA1UECwwDU0FTMRIwEAYDVQQDDAkxMjcuMC4w LjEwIBcNMjIxMTA3MDkxMzA0WhgPMzAyMjAzMTAwOTEzMDRaMF4xCzAJBgNVBAYT AkFUMQ0wCwYDVQQIDARXaWVuMQ0wCwYDVQQHDARXaWVuMQ8wDQYDVQQKDAZUVVdp ZW4xDDAKBgNVBAsMA1NBUzESMBAGA1UEAwwJMTI3LjAuMC4xMIIBIjANBgkqhkiG 9w0BAQEFAAOCAQ8AMIIBCgKCAQEAub9wOf/kgUptJ4F5J3gO6cxXpenX/j2+0lKo DJ7dXytCiUIH5yREfhqGxeCtR1b0/3rkK3wEkT8N3DbRtiGrGVFgwgegLtufrYLz yvUvm1p6JI1TPmv7wpvXjplq0T8G1e6P2UeMf2BZCAs8kGIYFARE0GlmZhiGha3R ElnFq6iTSr1FiBu7nQbxa/OwfdbVrRmHhO9qRhsaDurKa/MdH8CgJdOHzwKM7Jcn +azYwNPmkUskfNsAe6+9u5UJ70gAY2pS2gRNevlXWE5azoZC5GKPpNVpdjBXbMcc ZfT7jZ8KGdbOm2jOE40HoFanNBDSrrfjVaIRFvb4Bwr1Ud2cpwIDAQABo1MwUTAd BgNVHQ4EFgQUpwkz73DstzC221sS8WAgbrKf54kwHwYDVR0jBBgwFoAUpwkz73Ds tzC221sS8WAgbrKf54kwDwYDVR0TAQH/BAUwAwEB/zANBgkqhkiG9w0BAQsFAAOC AQEAW/h8RWIAkS1UiggSUqhbwBGQA4HTSEmKhe/4GeMQxMz8WAl4+oeSqAErgPdg ec8b2IT7iZcvwv9mpkCXPBb9vIcBFGn+IEYWV66dVxVW1oVYzTzFvw+0pqNsog9z FTzDDWBkhz3+LIaagoSLX1/LV/iiakxfUDJi+Xft59bf+f4A84p8+Ub2IzeyeTE2 cxcj3TD6rq5jGJAJZjkR6xIfivZ7rv/v+f0B+sN5cR4XOLResBxNhsh0S70FgIJB JuJembC8m36Axycy7D0g52WWdfbF5Q3bR3BWuxKX9WFXb93WpRbJlu1rKiD1lfGZ Ryzh2D4QlEOmRg6HtQFC1MWYTQ== -----END CERTIFICATE-----

```


It represents for sure a certificate and it is very strange that it is present on this firmware file. So maybe it is a valid certificate that could allow us to connect to the MQTT server.

At this point we rewrite the certificate into a txt file following the certificate standards and we obtain this:
```
-----BEGIN CERTIFICATE-----
MIIDnzCCAoegAwIBAgIUV8tXyfkAERUe4ajvN9S1AiQU3RowDQYJKoZIhvcNAQEL
BQAwXjELMAkGA1UEBhMCQVQxDTALBgNVBAgMBFdpZW4xDTALBgNVBAcMBFdpZW4x
DzANBgNVBAoMBlRVV2llbjEMMAoGA1UECwwDU0FTMRIwEAYDVQQDDAkxMjcuMC4w
LjEwIBcNMjIxMTA3MDkxMzA0WhgPMzAyMjAzMTAwOTEzMDRaMF4xCzAJBgNVBAYT
AkFUMQ0wCwYDVQQIDARXaWVuMQ0wCwYDVQQHDARXaWVuMQ8wDQYDVQQKDAZUVVdp
ZW4xDDAKBgNVBAsMA1NBUzESMBAGA1UEAwwJMTI3LjAuMC4xMIIBIjANBgkqhkiG
9w0BAQEFAAOCAQ8AMIIBCgKCAQEAub9wOf/kgUptJ4F5J3gO6cxXpenX/j2+0lKo
DJ7dXytCiUIH5yREfhqGxeCtR1b0/3rkK3wEkT8N3DbRtiGrGVFgwgegLtufrYLz
yvUvm1p6JI1TPmv7wpvXjplq0T8G1e6P2UeMf2BZCAs8kGIYFARE0GlmZhiGha3R
ElnFq6iTSr1FiBu7nQbxa/OwfdbVrRmHhO9qRhsaDurKa/MdH8CgJdOHzwKM7Jcn
+azYwNPmkUskfNsAe6+9u5UJ70gAY2pS2gRNevlXWE5azoZC5GKPpNVpdjBXbMcc
ZfT7jZ8KGdbOm2jOE40HoFanNBDSrrfjVaIRFvb4Bwr1Ud2cpwIDAQABo1MwUTAd
BgNVHQ4EFgQUpwkz73DstzC221sS8WAgbrKf54kwHwYDVR0jBBgwFoAUpwkz73Ds
tzC221sS8WAgbrKf54kwDwYDVR0TAQH/BAUwAwEB/zANBgkqhkiG9w0BAQsFAAOC
AQEAW/h8RWIAkS1UiggSUqhbwBGQA4HTSEmKhe/4GeMQxMz8WAl4+oeSqAErgPdg
ec8b2IT7iZcvwv9mpkCXPBb9vIcBFGn+IEYWV66dVxVW1oVYzTzFvw+0pqNsog9z
FTzDDWBkhz3+LIaagoSLX1/LV/iiakxfUDJi+Xft59bf+f4A84p8+Ub2IzeyeTE2
cxcj3TD6rq5jGJAJZjkR6xIfivZ7rv/v+f0B+sN5cR4XOLResBxNhsh0S70FgIJB
JuJembC8m36Axycy7D0g52WWdfbF5Q3bR3BWuxKX9WFXb93WpRbJlu1rKiD1lfGZ
Ryzh2D4QlEOmRg6HtQFC1MWYTQ==
-----END CERTIFICATE-----
```


Now we can change the extension of the txt file into `.pem` that is a format used to store certificates and cryptographic keys.
![[Pasted image 20240419153048.png]]

If we double click on it, we can see the details of the certificate and we can understand that it is a valid certificate:
![[Pasted image 20240419153141.png]]



## Connecting to the MQTT broker using the certificate found

Now we can retry to connect to the MTTQ server using the found certificate. 
The procedure is pretty similar to the previous one, and in fact we can just use the command:
```shell
mosquitto_sub -h mosquitto.sas.hackthe.space -p 8883 -t '#' --cafile certificate.pem
```

But also this time we get the error:
`Error: A TLS error occurred.`

This is a little bit strange, because the certificate we have has an high probability to be the correct one for connecting to the MQTT server. 

So we try to analyze it and we can see that the `CN` (Common name) is set to `127.0.0.1` that means we can connect only to `127.0.0.1` using this certificate:
![[Pasted image 20240419153653.png]]

But we can try to find a way to bypass the check on the `CN` field. So we search on the `mosquitto_sub` documentation ([here](https://mosquitto.org/man/mosquitto_sub-1.html)) and we can find on the `TLS` section an interesting argument we can use: `--insecure`

Description of `--insecure` argument:
```
When using certificate based encryption, this option disables verification of the server hostname in the server certificate. This can be useful when testing initial server configurations but makes it possible for a malicious third party to impersonate your server through DNS spoofing, for example. Use this option in testing _only_. If you need to resort to using this option in a production environment, your setup is at fault and there is no point using encryption.
```


Let's try it:
```shell
mosquitto_sub -h mosquitto.sas.hackthe.space -p 8883 -t '#' --cafile certificate.pem --insecure
```
- it works, and allows us to connect to the server.

As soon as we connect to the MQTT server we receive a message that is the flag.

So the flag is: ``




# Challenge part 2

## Introduction

This challenge can be accessed through this [link.](https://sas.hackthe.space/#/challenges/IoT%20part%202)

From the description of the challenge we can retrieve some useful information that we could take in mind:
```
Bob is a famous artist and, on the side, very passionate about technology! 
He is experimenting new things with his art, such as sharing his masterpieces over the MQTT protocol! (mosquitto.sas.hackthe.space, port 1883)! 

One day an unfinished painting got public, can you figure out how that happened? 

Hint: there is not a single MQTT version..
```
- There is a MQTT server on the domain **mosquitto.sas.hackthe.space on the port 1883**
- Bob **shares his paintings over MQTT**
- **Hint: there is not a single MQTT version.** So maybe we have to work with different protocol versions


## Connecting to the MQTT broker 
We know that there is a MQTT broker on  **mosquitto.sas.hackthe.space on the port 1883 so what we can try to do is to connect to it.

What we want to do there is to try to subscribe to all topics in order to see if we could get some useful information useful for the flag getting.


So we try to use:
```shell
mosquitto_sub -h mosquitto.sas.hackthe.space -p 1883 -t '#'
```


We are able to connect but nothing happens:
![[Pasted image 20240419161251.png]]

But we know from the hint tha there are different protocol versions. So let's get a look on the `mosquitto_sub` documentation ([here](https://mosquitto.org/man/mosquitto_sub-1.html)).

There we can find an useful argument that is `--protocol-version`

Description of `--protocol-version` argument:
```
Specify which version of the MQTT protocol should be used when connecting to the remote broker. Can be `5`, `311`, `31`, or the more verbose `mqttv5`, `mqttv311`, or `mqttv31`. Defaults to `311`.
```


So we can try all of them and see what happens:
![[Pasted image 20240419161820.png]]
- But also with different protocol versions nothing happens, at least nothing directly visible.


So we can do a deeper analysis using `wireshark`.

What we need to do here is to:
1. run wireshark using `sudo wireshark`
2. start a capture using the network device, in my case `wlo1`
	1. ![[Pasted image 20240419162046.png]]
3. start, as before, a connection with the MQTT server with all different MQTT protocol versions:
	1. ![[Pasted image 20240419161820.png]]
4. filter the connection on wireshark using `mqtt` filter
	1. ![[Pasted image 20240419162227.png]]




At this point we can start to analyze the different connection. 
In general what we need to do is to analyze the response we obtain to the connection request and so in this case the `Connect ACK` packets.


So we take a look:
1. `Protocol version 31` Connect ACK packet
	1.  ![[Pasted image 20240419162511.png]]
	2. Nothing interesting there
2. `Protocol version 311` Connect ACK packet
	1. ![[Pasted image 20240419162617.png]]
	2. Nothing interesting there
3. `Protocol version 5` Connect ACK packet
	1. ![[Pasted image 20240419162652.png]]
	2. **Very interesting!**



So the protocol version 5 gives us some useful information:
- `flag_picture` is the topic we need to connect to
- `bob:ross` are the credentials we can use to connect to it


## Subscription to the MQTT topic using found credentials
At this point we can subscribe to the topic `flag_picture` using the credentials `bob:ross`.

We use this command:
```shell
mosquitto_sub -h mosquitto.sas.hackthe.space -p 1883 -t 'flag_picture' -u bob -P ross
```
- `-u` is used to define the username 
- `-P` is used to define the password

The result is:
![[Pasted image 20240419163225.png]]

Not comprensible at all. But we know that Bob does paintings and the topic is called `flag_picture`. So maybe this binary represents a picture.


What we really need is the HEX representation of this binary. And we can easily find it using wireshark, capturing the message exchange:
![[Pasted image 20240419163402.png]]

So we can now copy the message received:
![[Pasted image 20240419163449.png]]

At this point we can past it into a file `flag.txt` and write a simple python script that converts HEX strings to images or use the website [CodePen](https://codepen.io/abdhass/full/jdRNdj) :
![[Pasted image 20240419163708.png]]


The  python script to do the same thing could be:
```python
import binascii
with open('flag.txt', 'r') as file:
    data = file.read().strip()

data = binascii.a2b_hex(data)
with open('image.jpg', 'wb') as image_file:
    image_file.write(data)
```



The flag is: `FLG_PT2{B0BR0SS_TH3_B3ST}`



# Challenge part 3

## Introduction

This challenge can be accessed through this [link.](https://sas.hackthe.space/#/challenges/IoT%20part%203)


From the description of the challenge we can retrieve some useful information that we could take in mind:
```
Knodel war!

I'm filling good.

As with almost every recipe, the original recipe for knodels is also disputed among a bunch of countries: Germany, Italy, Austria... And this opens up a crucial debate... 
BUT WAIT! A group of malicious chefs is showing on TV the wrong recipe! Can you find out which recipe? 

[You have a recording of the stream](https://owncloud.tuwien.ac.at/index.php/s/cSu6orfR8PlMco7), password: hbbtv_chal.
```
- We can download the record of the stream [here](https://owncloud.tuwien.ac.at/index.php/s/cSu6orfR8PlMco7) using the password `hbbtv_chal`
	- Maybe the attacker injected the HbbTV signal



## Analyzing the stream recording file

Using the link provided by the challenge we will download the stream recording file that is a `.ts` file.

A TS file is a video file saved in the Video Transport Stream (TS) format, which stores video data compressed with standard MPEG-2 (MPEG) video compression.
- This format is commonly used for transmitting video data over networks, including broadcasting over the air or via satellite.


To analyze it we need to use the `TSDuck` tool that could be dowloaded [here](https://tsduck.io/download/tsduck/).
- TSDuck is a e tool used for analyzing and manipulating MPEG transport streams and it provides various functionalities for inspecting, modifying, and generating transport streams.


With the usage of this tool we are able to analyze the file and the content of the signal that contains **application data and stream events**. 
An attacker can manipulate this signal in order to inject unauhtorized data into the AIT **Application Information Table**, so we could be pretty sure that the attacker modified it to perform the injection.
- An Application Information Table (AIT) is a specific type of table used to convey application-related information it contains information about interactive applications, such as their identifiers, descriptors, and other metadata.


You could get more information about **HbbTV** from this article: [link](https://medium.com/@dimitri.reifschneider/hbbtv-all-about-streams-230bd0ac05fb)

### AIT information extracting
What we really need to do is to analyze the AIT using `TSDuck` and we can do that using this command:
```shell 
tstables hacked_2024.ts -a -b tables_in_binary
```

This command collects MPEG tables or individual sections from a transport stream and so it takes all the information about the AIT from the stream recording file `hacked_2024.ts` and stores it into a binary file called `tables_in_binary`

From the documentation:
- `-a`
	- Display/save all sections, as they appear in the stream. By default, collect complete tables, with all sections of the tables grouped and ordered and collect each version of a table only once. Note that this mode is incompatible with all forms of XML and JSON output since valid XML and JSON structures may contain complete tables only
- `-b` 
	- Save the sections in raw binary format in the specified output file name. If the file name is empty or '-', the binary sections are written to the standard output


### AIT information converting into XML
At this point we need to convert the binary file into a readable format. So we convert it into a `XML` format using `TSDuck`.

In this specific case we can use this command:
```shell
tstabcomp --decompile tables_in_binary -o tables_in_XML
``` 

### AIT analyzing
Now, we can open the XML file and analyze it.

It is a very huge file, but we can start to search for keywords such as: `recipe`, `ricetta` (since it is in italian), `knodels`, `knodel`.

And, in fact, if we search for `knodel` we can see that there are interesting sections in the file that are:
![[Pasted image 20240419172333.png]]

Here we can easily see the string `RkxHX1BUM3tQMW4zQXBwbDNfS24wZDNsfQ==`, by the format and the final `==` it could be a base64 string.

So we try to decrypt it using the command:
`echo 'RkxHX1BUM3tQMW4zQXBwbDNfS24wZDNsfQ==' | base64 -d`
- it basically passes the string echo `RkxHX1BUM3tQMW4zQXBwbDNfS24wZDNsfQ==` to the command `base64 -d` that decrypts it


![[Pasted image 20240419172513.png]]


Or we could use this website: [base64decode.org](https://www.base64decode.org/).


The flag of part 3 is: `FLG_PT3{P1n3Appl3_Kn0d3l}`