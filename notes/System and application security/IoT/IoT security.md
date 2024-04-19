
# What is the Internet of Things?
The internet of things is basically **objects with computing devices that are able to connect to each other and exchange data using the Intenet**

![[Pasted image 20240413160821.png]]




# The "S" in IoT Stands for Security

Seemingly harmless devices but they open up to whole new attacks (such as DDoS).

**==There are no standardized approaches or security best practices.==**
- documentation can also be misleading resulting in misconfigured servers.

*For example, AWS provided in their offical developer guide 38 best practices and 26 of them introduced some vulnerabilities*



## Mirai (2016)
Mirai is a malware that infects smart devices that run on ARC processors. It:
- **scans for vulnerable devices**
- **tries to authenticate via telnet using default username and passwords**

When devices are infected, they are used as bots for DDoS attacks.

Source code = [Mirai source code](https://github.com/jgamblin/Mirai-Source-Code)


## CloudPets (2017)
It allowed parents and kids to record and receive audio files using Internet connection and toys.

**Data was stored in a MongoDB that was in a public network segment without any authentication.**


In addition if we look at the app, the profile image was stored in an Amazon S3 bucket which was publicly accessible and contains Personally Identifiable Information (PII) about the kids. 
- if I get the URL of the bucket then I have access to all the information


CloudPets had no password strenght rules. These password could be cracked, also if they were hashed with bcrypt.




# Why IoT so insecure?

1. A lot of **Communication protocols**
	1. BLE, CoAP, MQTT etc
2. A lot of **Hardware Architectures** 
	1. there are different processors (ARM, AVR)
	2. and some peripheral devices (sensors/actuators)
3. A lot of **Communication means**
	1. Wired (CAN for cars, Ethernet)
	2. WAN (WiFi, 3G, LoRaWAN)
	3. Low Range Networks (Zigbee, Bluetooth)

In addition IoT have  limited resources.





# OWASP top 10 for IoT
It consists of best-practices and things to avoid instead of actual vulnerabilities.

## 1. Weak, Guessable or Hardcoded Passwords

**==So basically it opens the doors to bruteforce attacks or the using of known vocabularies.==**



## 2. Insecure Network Services

==**This involves all network services used that compromise the confidentiality, integrity/authenticity or availability of information or allow unauthorized remote control.**==

*Example using SSH, TelNet*


## 3. Insecure Ecosystem Interfaces
==**This involves all insecure web, backend API, cloud, or mobile interfaces  used in the ecosystem that allows to compromise the device or the components.**==

*Example: lack of authentication/authorization, weak encryption, and a lack of input and output filtering.*


## IoTFlow 
![[Pasted image 20240413163548.png]]

Every IoT device has his own ecosystem.


If we want to perform an analysis of these devices we can focus on 

### Value Set analysis
In this case we identify all potential values that a variable might take at a specific point in a program.


### Dataflow analysis 
Here we determine where data coems from, or where data is going to.

![[Pasted image 20240413163945.png]]


## 4. Lack of secure update mechanism
**==This involves all problems related to the security in updates.==** 

*For example we could have lack of firmware validation of device, lack of secure delivery (un-encrypted in transit), lack of anti-rollback mechanism and lack of notifications of security changes due to updates.*


## 5. Insecure Ecosystem Interfaces

**==This involves the usage of deprecated or insecure software components/libraries that could compromise the device.==**

*For example it involves insecure customization of operating system platforms and the use of third-party software or hardware components form a compromised supply chain.*



## 6. Insufficient Privacy Protection

**==This involves the storing of personal data into devices or ecosystems that are insecure==.**

## 7. Insecure Ecosystem Interfaces 

**==This involves the missing of encryption or missing of access control on sensitive data (at rest, in transit or during the processing of that data).==**

*For example: lack in cryptography, mismanagement of keys, inefficient access controls etc*

## 8. Lack of Device Management

**==It involves lack of security support for deployed devices.==** 

*For example: no security update, no systems monitoring etc*

## 9. Insecure Default Settings

==**It involves all the devices that have insecure default settings or all devices that cannot be configured to be more secure.**==


## 10. Lack of Physical Hardening

It involves all devices that are not secure in a physical way,  that could help an attacker to gain sensitive information for a future remote attack or to take a local control of the device.

*For example:no anti-tampering defenses or system integrity checking*




## CASE STUDY: SMART CAMERA

```java
byte[] Key = {88, 99, … , OTHER_VARIABLE, -107, … }; void Login(String user, String password) { 

String imei = GlobalArea.getPhoneID(); 

byte[] user_buf = DEES3.encrypt_3des(Key, user.getBytes()); 

String md5pw = DEES3.Md5_String32(password); 

byte[] pw_buf = DEES3.encrypt_3des(Key, md5pw.getBytes());

String xml = "" + DEES3.byte2Hex(user_buf) + "" + DEES3.byte2Hex(pw_buf) + "" + imei + ""; 

this.glsocketclient.sendData(xml); }
```

We can see there that the **IMEI is used for authentication**.
- we should never use it because it is a unique code that cannot be changed and if it is theft then we are fucked up


**The password is hashed with MD5 (a weak algorithm that can be reverted)**

**Username and password are encrypted with 3DES. (weak)**


## CASE STUDY: SMART GRILL

```java
void initiateIoT() { 
	//... 
	this.topicUpdate = "$aws/things/" + this.deviceName + "/update"; initMQTTConnection(); 
} 

public MQTTMessenger(DeviceListener listener) {
	this.credentialsProvider = new CognitoCredentialsProvider("us:xyz");
	 this.mqttManager = new AWSIotMqttManager(UUID.randomUUID(), "xyz.iot.us.aws.com"); 
}
```

Here we can see that the data is sent to an US AWS endpoint and that:
- Cognito credential provider is used
- there is no access control in MQTT




# IoT Communication Architectures

![[Pasted image 20240413170123.png]]



# HyperText Transfer Protocol (HTTP)

This protocol is Connectionless and Stateless.

The architecture used is the REST one and the model used is the client/server.

It supports password and certificate based authentication.

![[Pasted image 20240413170258.png]]


# Message Queue Telemetry Transport (MQTT)

This protocol works over TCP/IP or WebSockets.

The architecture is Publish/Subscribe.

It is ideal for communication between devices with limited resources over low-bandwidth and unreliable networks.

It supports Access Control Lists, password and certificate-based authentication.


![[Pasted image 20240413170524.png]]


**The Broker receives and sends messages
The Publisher publish messages on a topic
The subscriber receives the messages published on topics on which it is subscribed.**


![[Pasted image 20240413170613.png]]

It can have different types of messages:
- CONNECT ( and CONNACK ), in it we can specify URLs, port and credentials
- SUBSCRIBE ( and SUBACK ), to subscribe on a topic
- PUBLISH ( with the various ACKs-PUBACK, PUBREC, etc), to publish a message on a topic
- DISCONNECT


It has a specification called QoS (Quality of Service) that is used to guarantee some levels of performance and other information about the quality of data transmission:
- **at-most-once (level 0), the message is sent to the recipient at most one time**
	- messages could be lost because they are sent only one time
	- **i sent it but i don't care about the ACK**
- **at-least-once (level 1), the message is sent to the recipient at least one time**
	- the message will be delivered at leat one time but there could be duplication
	- **i send the message until i get a ACK back**
- **exactly-once (level 2), the message is sent to the recipient at exactly one time**
	- more reliable (the message is delivered exactly one time )
	- **I send the message and i get form the broker an ACK, then i send an ACK for the ACK and i wait for the confirmation of that one**

It supports also two wildcards:
- **+**, single level
	- Ad esempio, se ci iscriviamo a "casa/salone/+", riceviamo i messaggi pubblicati a "casa/salone/illuminazione", "casa/salone/temperatura", "casa/salone/presa", e così via.
- **#** , multilevel
	- Ad esempio, se ci abboniamo a "casa/#", riceveremo i messaggi pubblicati a "casa/salone/illuminazione", "casa/cucina/temperatura", "casa/garage/presa", e così via.






## MTTQ SECURITY 

### Unauthorized MQTT messages
There are 2 different unauthorized messages:
- **Retained messages** that is sent to every new subscriber (the future subscribers) on a given topic
	- a malicious ex-user can send a Retained message if he has still access to the device
- **Will messages** that is sent to every client subscribed to a topic when the client who sent this message disconnects.
	- a ex user can use the Will flag to later trigger a command when he disconnect (for example in an hotel it could open a smart door locked)



### Handling of topic strings
In this protocol when a client fails to validate an disallowed UTF-8 it will close the connection.

It opens the door to a DoS attack because
1. broker doesn't check for disallowed UTF-8
2. Only clients checks for it
So a malicious client can disconnect other clients by sendinf invalid strings.

We cannot change the configuration of the Broker, only if we revert the code and rewrite 


## Nexx Garage Door Vulnerability (2023)

There are 5 security issues disclosed publicly. 

The issues here was that there was an Universal MQTT password for all registered devices via the Nexx Home mobile app. This password was disclosed in:
- the API data exchange
- the firmware as an hardcoded string

![[Pasted image 20240413172206.png]]




# Constrained Application Protocol (CoAP)

It is based over UDP but can also be extended to TCP and WebSockets.

It is based on REST architecture and client/server model.

It is designed for resource-constrained devices (microcontrollers) in constrained networks.

It supports password and certificate-based authentication with DTLS.


## Messages 

The **request messages** are equal to HTTP and contain:
- method (GET, POST, DELETE, PUT, FETCH, PATCH)
- resource identifier
- a payload
- optional metadata
- message identifier
- message type

*For example*
```yaml
CON GET 
MID: 12345 
URI-Path: resource 
Accept: application/json
```



The **response messages** contain:
- a response code (for example 4.03 Bad Option)
- a payload
- mesage type
- message identifier

*For example:*
```yaml
ACK 2.05 Content
MID: 12345
Content-Format: application/json
Payload: {"temperature": 25, "humidity": 50}

```



There are 4 types of messages:
1. **Confirmable (CON)**, the client waits for an ACK or continues to send the message
2. **Non-Confirmable (NON)**, the delivery is not ensured, no ACK
3. **Acknowledgement (ACK)**
4. **Reset (RST)**, when a recipient cannot process the request.


## Security modes

There are 4 differents security modes in CoAP:
1. **No Security (NoSec)**
2. **Pre-Shared Key (PSK)**, here DTLS is enables (TLS for UDP). Symmetric key for encryption is used. 
	1. these keys are previously distributed and secured safely in storage.
3. **Raw Public Key (RPK)**, here asymmetric key is used. They are previously exchanged. It works as a certificate without the public key validation.
4. **Certificates**, here we use Certificate authority to validate certificates.


### IP address spoofing attacks
In UDP is not possible to verify if the packet is originated by the host with the IP used in the header. So we can perform spoofing attacks:
- **Reset message spoofing**, the attacker can close other client connections
- **ACK message spoofing**, the attacker can block the retrasmission
- **Response spoofing**, the attacker can change the real response with a crafted one
- **Multicast request spoofing**, the attacker can perform DoS attacks and network collapse

<mark style="background: #FF5582A6;">Always avoid NOSEC</mark>


### Amplification 
The size of the response packet can be larger that the request one, so the attacker can send a small request to generate large responses.

This allows him to perform DoS attacks.





# eXtensible Messaging and Presence Protocol (XMPP)

It works over TCP/IP.

It is based on client/server model and uses the Direct-to-Device communication (D2D).

It is designed as a near real-time instan messaging protocol, if fact is used in chat application like Facebok.

Messages are XML Stanzas: 
- parts of the XML document

It supports Access Control Lists, password and certificate based authentication.



The most important features of this protocol are:
- Openness, because we have many open-source servers and clients
- Extensibility, because with the XMPP Extension Protocols (XEP) we can extend the core specification 
	- *For example we can have multi-user chat or publish/subscribe architecture*
- Flexibility, because it uses XML, that is a markup language based on domain and needs.


![[Pasted image 20240414131354.png]]

## XMPP addresses
XMPP addresses are unique and we have 3 types of addresses:
- Sever Address (JID), that is a domain
	- *For example: `xmpp.server.com`*
- Client Address (bare JID), that can be a local ID, usually is the username:
	- *For example: `username@xmpp.server.com`*
- Resource Address (full JID), that contains a resource name 
	- *For example `username@xmpp.server.com/resource`*


Each message is exchanged using XML stanzas.
We have 3 types of stanzas:
- Message, the tag is `<message/>` and is used to send information to other entities
- Presence, the tag is `<presence/>` and is used to broadcast the presence of an entity in the network
- Info/Query (IQ), the tag is `<iq/>`, it is the request/response mechanism 




## XMPP Security 
The XMPP protocol implements these security mechanis:
- Session encryption with TLS
- Authentication with Simple Authentication and Security Layers (SASL) that needs the fully qualified domain name, and identifier, a username and a passphrase
- Access Control mechanism



XMPP uses TLS with STARTTLS extension
- ovvero una estensione che permette di avviare la sessione TLS dopo che la connessione è stata stabilita tra i client, senza dover usare una porta separata per le comunicazioni criptate

This TLS with STARTTLS extension is vulnerable to downgrade attacks that enable Man-in-the-Middle attacks.
- It performs the SASL negotiation ONLY after a successful TLS connection; 
- the receiving entity provides a list with supported SASL mechanisms so a MITM could intercept the message and exchange it with a list containing only weak mechanisms



There is also a lack of support for end-to-end encryption. We have no guarantees that all streams are encrypted.
- *Esempio: se io comunico con il server con TLS e il messaggio viene inviato ad un altro client dal server senza usare TLS*

### Hanwha SmartCam (2020)
The command message was generated in the application and sent to the camera using XMPP. The fact is that it:
- uses HTTP for firmware update (so we can tamper it)
- uses a weak password policy when registering the camera on  the server `xmpp.samsungsmartcam.com`

The entire cloud was ONE Jabber server with "rooms" (that contain cameras). An attacker can register an arbitrary account on the Jabber server and have the access to all rooms of the server.



# Hybrid Broadcast Broadband TV (HbbTV)

It is used to aggregate the traditional television transmission (broadcast) with internet services (broadband). It allows users to access the normal tv transmission and additional internet services such as streaming on demand and so on.


We have here 2 connections in parallel:
- Broadcast Digital Video Broadcasting (DVB)
- Internet connection via BroadBand interface

![[Pasted image 20240414131519.png]]



The TV receives application data and stream events via the **Broadcast interface**
- it's basically a **web application that is shown as a layer over the tv channel**


The data is sent to the Runtime Environment (RE) of the TV where the browser is located.

HbbTV apps are embedded as URLs in the broadcast stream.

The **TV’s browser presents and executes the HbbTV.** The Internet Protocol Processing component parses and passes the information to the RE.

It is written with standard web techniques (HTML, CSS, JavaScript).

![[Pasted image 20240414131854.png]]



**L'idea fondamentale è che tutto arriva con un singolo segnale dall'antenna o dal cavo che contiene:**

![[Pasted image 20240414214322.png]]
**AIT contiene tutte le info sull'applicazione HbbTV.**

Il segnale viene:
1. inviato alla tv
2. la tv prende AIT e lo invia al Runtime Environment 
3. il Runtime environment processa AIT e effettua le comunicazioni con gli URL


## HbbTV Security 
1. TLS is suggested but not mandatory, it depends on manufacturers and content providers decide.
2. HbbTV application run inside built-in browser displaying HTML content and running js code
	1. An attacker can replace the URL pointing to the HbbTV application through a DVB/DMS-CC injection
	2. Attackers can trick users to click malicious links, or display them fake news banners

### DVB Hijacking
DVB hijacking refers to the unauthorized access and control of Digital Video Broadcasting (DVB) signals. 

DVB is a set of international standards for digital television transmission. 

Hijacking these signals can involve various methods, i**ncluding intercepting and manipulating the broadcast content or taking over the transmission channel to broadcast unauthorized content**.


To do it we need:
1. HiDes UT100C modulator
	1. it's like an usb but It is designed to modulate digital video and audio signals into a format suitable for transmission over various mediums, such as cable or satellite
2. Antenna
3. C++ TSDuck library
	1. The C++ TSDuck library is a software library designed for processing MPEG transport streams. Transport Stream (TS) is a standard format for transmission and storage of audio, video, and data on various media such as terrestrial broadcast systems, cable television, and satellite systems.
4. Laptop


![[Pasted image 20240414214143.png]]


The idea here is to **extract the HbbTV app URL form the DVB stream. The URL is contained in the Application Information Table (AIT)**


![[Pasted image 20240414214322.png]]
In generale possiamo accederci direttamente dal browser che ha il supporto HbbTV. Basta avviare l'app e accedere alle info di debug tramite il menu del browser.




**If we extract the Program ID where the acronym AIT is present and convert it into XML we see the AIT for HbbTV with different application URLs.**
- This is what we need to change if we are attackers

![[Pasted image 20240414214639.png]]



**If I want to show my injected signal to the TV i need an Antenna to overlap the normal signal.**

# SHODAN
Shodan is search engine for Internet-connected devices [SHODAN](https://www.shodan.io/)

In fact if we search for `has_screenshot:true port:554` we can see screenshots of devices that stream in real time and that are not secured.

**Port 554 is used for Real Time Streaming Protocol (RTSP) in IP cameras.**
