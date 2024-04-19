# COME AVVIENE IL LEASING?


# iptrading.com

(questi dati sono stati ottenuti chiedendo ad iptrading.com come funziona il loro modo di effettuare il leasing)

## lettera di agenzia
==**L'accesso agli indirizzi affittati viene concesso al locatore tramite una Lettera di Agenzia (LOA)**== che descrive il locatore che concede al locatario l'autorizzazione a utilizzare gli indirizzi.

La LOA:
- <mark style="background: #BBFABBA6;"> è un documento legale che conferisce al locatario il diritto di utilizzare gli indirizzi IP specifici assegnati dal locatore o dal provider di servizi Internet (ISP) </mark>per scopi specifici, come ad esempio l'utilizzo su una rete aziendale.
- <mark style="background: #BBFABBA6;">descrive i dettagli del contratto di locazione, inclusi gli indirizzi IP assegnati, la durata della locazione, le responsabilità del locatario e del locatore</mark> ecc
- L'obiettivo principale della LOA è stabilire chiaramente i diritti e le responsabilità delle parti coinvolte per garantire un uso corretto e legale degli indirizzi IP.

## Route Origination Authorization (ROA) tramite RPKI

==**Inoltre, è consuetudine generare anche una Route Origination Authorization (ROA) tramite RPKI che specifica l'ASN del locatario per pubblicizzare il percorso.**==


- <mark style="background: #BBFABBA6;">RPKI (Resource Public Key Infrastructure) è un sistema di sicurezza utilizzato per garantire l'autenticità e l'integrità delle informazioni di routing su Internet</mark>, incluso l'assegnazione di blocchi di indirizzi IP.
- <mark style="background: #BBFABBA6;">Una ROA è un certificato digitale firmato che specifica l'autorizzazione per un'autorità di routing (AS, Autonomous System) a pubblicizzare un blocco di indirizzi IP</mark>.

==**Quando un locatario riceve indirizzi IP affittati, può generare una ROA per quegli indirizzi tramite RPKI. Questa ROA specifica l'ASN (Autonomous System Number) del locatario che può pubblicizzare il percorso degli indirizzi IP assegnati.**==
- Questo processo aiuta a garantire che solo il locatario autorizzato possa annunciare il percorso degli indirizzi IP assegnati, riducendo così il rischio di abusi o manomissioni nel routing degli indirizzi.


## Uso degli indirizzi IP 
==**Si noti che offriamo locazioni in cui il locatario deve pubblicizzare e utilizzare gli indirizzi sulla propria rete** ==(o su quella dell'ISP a monte).

- Nelle locazioni di indirizzi IP, è spesso richiesto che il locatario utilizzi gli indirizzi assegnati sulla propria rete o sulla rete dell'ISP a monte, anziché tramite tunneling.
- <mark style="background: #BBFABBA6;">Ciò significa che il locatario è responsabile della corretta configurazione e gestione degli indirizzi IP assegnati all'interno della propria infrastruttura di rete.</mark>




# IPXO

**==Quando affittiamo degli indirizzi IP in IPXO viene generato una LOA anche qui.==**




### IPXO acquisto in leasing
L'idea è registrarsi effettuato un test KYC, ovvero inserendo informazioni che verranno usate da IPXO per capire se l'acquirente è "sicuro".

![[Pasted image 20240319111657.png]]


![[Pasted image 20240319111745.png]]
![[Pasted image 20240319111903.png]]

![[Pasted image 20240319111921.png]]






Dobbiamo prima assegnare loro un ASN, altrimenti non sono utilizzabili.

![[Pasted image 20240319111145.png]]


==**A questo punto possiamo far generare un ROA, un oggetto WHOIS, e un route object su RADb.**==

![[Pasted image 20240319151345.png]]





==**Alla fine del processo possiamo scaricare il LOA e utilizzare la subnet affittata che ha l'ASN assegnato.**==

![[Pasted image 20240319111434.png]]







### IPXO vendita di IP

![[Pasted image 20240319112155.png]]![[Pasted image 20240319112215.png]]




A questo punto ci viene chiesto di inserire le subnet che vogliamo dare in leasing:
![[Pasted image 20240319112313.png]]![[Pasted image 20240319112325.png]]

Andando avanti si possono impostare i dettagli del leasing:

![[Pasted image 20240319112415.png]]

![[Pasted image 20240319112432.png]]


Per alcune potrebbe essere necessario inserire username e password (come per esempio le reti gestite da APNIC):
![[Pasted image 20240319112608.png]]
Un venditore può fornire la possibilità di dare questi servizi entro 48 ore:
![[Pasted image 20240319112708.png]]



