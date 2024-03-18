
# IRRegularities in the Internet Routing Registry paper

Il Registro di instradamento Internet (IRR) è un insieme di database distribuiti
utilizzati dalle reti per registrare le informazioni sulle politiche di instradamento e per
messaggi ricevuti nel Border Gateway Protocol (BGP).
- BGP è il protocollo che indica alle richieste di dati in arrivo quale percorso devono seguire per raggiungere il _server_ dove si trovano i dati di interesse, **minimizzando i tempi di attesa**.


**==L'IRR manca di uno standard di validazione rigoroso e il limitato coordinamento tra i diversi fornitori di database può portare a imprecisioni.==**

Inoltre, <mark style="background: #FF5582A6;">le reti possono registrare le informazioni di instradamento in più database IRR ma conservarne solo un sottoinsieme, contribuendo ulteriormente all’incoerenza tra i diversi IRR.</mark>

Non siamo a conoscenza di alcuno sforzo per risolvere le incongruenze tra i database IRR, né di una procedura standardizzata per convalidare se un AS registrato nell'IRR ha l'autorizzazione ad annunciare i suoi prefissi registrati. 


Tali vulnerabilità hanno permesso a soggetti malintenzionati di falsificare i record nell'IRR e successivamente di originare falsamente il prefisso in BGP. 


### RPKI 
RPKI (Resource Public Key Infrastructure) è un framework di sicurezza progettato per migliorare la sicurezza del BGP (Border Gateway Protocol). 

<mark style="background: #BBFABBA6;">RPKI si basa su un sistema di firme digitali e certificati per verificare l'autenticità dei percorsi BGP e garantire che i prefissi IP siano annunciati solo da parte dei legittimi titolari. </mark>
- Funziona associando le risorse IP agli AS attraverso l'emissione di certificati firmati digitalmente. 
- Questi certificati vengono quindi utilizzati per autorizzare i router a eseguire il routing di determinati prefissi IP.


## Internet Routing Register 
Due dei database IRR più grandi e più vecchi sono RADB e RIPE IRR. Negli ultimi decenni sono emersi nuovi database IRR gestiti da società commerciali (ad esempio Lumen, NTT), registri Internet regionali (RIR) e registri Internet locali (LIR).

I cinque RIR (RIPE, ARIN, APNIC, AFRINIC e LACNIC) **==gestiscono database IRR autorevoli==**. <mark style="background: #BBFABBA6;">Le informazioni di instradamento registrate in tali database IRR sono sottoposte a un processo di convalida rispetto alle informazioni sulla proprietà dell'indirizzo per garantirne la correttezza.</mark> 

I <mark style="background: #FF5582A6;">database IRR gestiti da altre istituzioni sono database IRR non autorevoli e non sono convalidati</mark>.


Per registrare le informazioni di instradamento nell'IRR, un'organizzazione deve prima registrare le proprie informazioni di autenticazione e l'e-mail dell'operatore di rete in un oggetto manutentore (mntner).

L'organizzazione può quindi creare e modificare record IRR come l'oggetto del percorso. **L'oggetto del percorso contiene il prefisso IP e l'ASN che l'organizzazione intende utilizzare per originare il prefisso in BGP**. 

Gli **IRR autorevoli contengono l'oggetto inetnum**, che contiene informazioni sulla proprietà dell'indirizzo, ma questo oggetto generalmente non è presente in altri IRR (quelli non autorevoli).




### IRR falsificati
Gli attaccanti inseriscono falsi record IRR nel tentativo di aumentare la probabilità di ottenere un attacco di dirottamento BGP.

In genere gli attaccanti registrano route objects contenenti i prefissi che vogliono dirottare, utilizzando l'AS di questi prefissi.
- REMINDER: ==**Un AS è un gruppo di reti IP che sono sotto il controllo di una singola entità amministrativa, come un Internet service provider (ISP) o un'organizzazione aziendale.**==


## Data set utilizzati
- **Archivio IRR** -> istantanee giornaliere dei database IRR tra novembre 2021 e maggio 2023. 
	- Nel novembre 2021 -> hanno avuto accesso a 21 database IRR da 17 server FTP, ma solo a 18 database nel maggio 2023. Tre fornitori IRR (ARIN-NONAUTH, OPENFACE, RGNET) hanno ritirato i loro database durante il periodo di raccolta dei dati e i loro elenchi sono stati rimossi.
	- ![[Pasted image 20240316155456.png]]
- **Set di dati BGP** -> con lo strumento CAIDA BGPView hanno letto gli aggiornamenti BGP raccolti da Routeviews e RIPE RIS.
	- Hanno creato istantanee BGP con incrementi di 5 minuti con il quale hanno costruito un dataset
- **Archivio RPKI** -> RIPE NCC pubblica elenchi giornalieri di payload ROA (Route Origin Authorization) convalidati dai cinque trust Anchor RPKI (APNIC, ARIN, RIPE NCC, AFRINIC, LACNIC)
- **Dirottatori BGP seriali**-> dei ricercatori hanno fornito un elenco di dirottatori seriali BGP in base al loro comportamento di instradamento.
- **Set di dati di supporto** -> set di dati CAIDA AS Rank, set di dati CAIDA AS Relationship  e set di dati CAIDA AS-toOrganizations (as2org) per analizzare gli AS registrati nei database IRR.



# Methodology
## Caratteristiche dei IRR
### Consistenza inter-IRR
Si confrontano il route object di una coppia IRR a e b:

IRR-a ha una route R-a che ha un prefisso P-a e un origine AS-a, il route object viene classificato così:
1. si trova in ==**IRR-b una lista di route object R-b1 ... R-bn con i prefissi P-b1 ... P-bn dove ogni prefisso P-bi deve essere uguale al prefisso P-a**==
2. se questa lista non esiste allora <mark style="background: #BBFABBA6;">R-a è classificato come no overlap</mark> (nessuna sovrapposizione)
3. se la lista esiste ed abbiamo che per ogni AS-bi corrispondente a R-bi,<mark style="background: #BBFABBA6;"> se AS-a è uguale ad almeno uno di AS-bi allora R-a è considerato consistente con IRR-b</mark>
	1. se questo non è vero allora viene usato il dataset CAIDA as2org e AS Relationship per capire se sono correlati in qualche modo. 
4. Altrimenti è considerato non coerente.

IRR-a e IRR-b sono consistenti quanti più oggetti coerenti hanno tra loro.


### BGP Overlap 
Per ogni database IRR si conta il numero di route objects con lo stesso identico prefisso e stessa AS. ==**La percentuale viene calcolata su tutti gli oggetti del percorso.**==



## Identificazione dei Route Objects irregolari
### Mismatching tra origini AS e IRR autoritari

<mark style="background: #BBFABBA6;">Le informazioni nei database IRR autorevoli sono considerate più affidabili.</mark>

Un route object è classificato come irregolare se c'è una mancata corrispondenza con il suo oggetto corrispondente nell'IRR autorevole. 

Si usa la stessa tecnica di prima dove IRR-a è qualsiasi database IRR non autorevole e IRR-b sono i 5 database IRR autorevoli mixati. 

## Matching tra IRR objects e BGP

==**Un attaccante che vuole evadere il firltraggio di BGP inserisce nei IRR falsificati il proprio route object. Perciò si controlla che per gli oggetti classificati come incoerenti (vedi sopra) i loro prefissi sono già apparsi in BGP nello stesso periodo di tempo.**==

Gli oggetti incoerenti sono di 3 tipi:
- i route objects che hanno prefissi assegnati allo stesso set di ASN sia nei dataset IRR che in quelli BGP sono detti **completamente sovrapposti**
- i route objects che hanno che hanno prefissi assegnati a diversi set di ASN che hanno comunque una sovrapposizione parziale di ASN sono detti **parzialmente sovrapposti**
	- Ad esempio questi due IRR (P, AS1) e (P, AS2) che corrispondono a BGP (P, AS2), (P, AS3). 
	- (P, AS2 ) è parzialmente sovrapposto
- i route objects che hanno prefissi assiciati a insiemi disgiunti di ASN nell'IRR e nel BGP **non sono considerati sovrapposti**.

## Validazione dei route object irregolari 

Se un oggetto del percorso ottenuto dai passaggi precedenti ha un record RPKI corrispondente nel set di dati RPKI, viene rimosso dall'elenco di route objects irregolari. 

==**Gli AS dei route objects irregolari vengono verificati con quelli dei dirottatori seriali in modo da vedere quali sono effettivamente malevoli.**==


# IRR Baseline Characteristics

## Consistenza tra database IRR 
Nella maggior parte dei casi i vari IRR non sono consistenti tra loro:
![[Pasted image 20240316164639.png]]

Ci sono casi in cui una società ha registrato oggetti di percorso in più database IRR, ma ha aggiornato i record solo in un database IRR, causando incoerenze tra IRR. 

Si è notato anche una mancata corrispondenza di record tra coppie di database IRR autorevoli. 

## Consistenza con i RPKI
Riassunto in questa foto:
![[Pasted image 20240316170746.png]]

## Overlap con gli annunci BGP 
![[Pasted image 20240316171050.png]]




# Route Object irregolari

## RADB Analysis

### Filtraggio di objects irregolari
Sono stati filtrati route object di RADB.
Sono stati trovati 196.664 prefissi univoci che hanno un'origine AS non corrispondente con uno qualsiasi dei cinque IRR autorevoli (RIPE, ARIN, APNIC, AFRINIC, LACNIC).

Si sono concentrati sui prefissi parzialmente sovrapposti perché avevano conflitti AS multi-origine (MOAS) in BGP, che è stata una metrica utilizzata per identificare il dirottamento BGP.
- ==**MOAS -> uno stesso prefisso IP viene annunciato nello stesso momento da due AS diversi**==



### Validazione 

Hanno scoperto che dei 34.199 route objetcs irregolari, 20.523 sono coerenti, 4.082 hanno un ASN non corrispondente, 144 hanno un prefisso troppo specifico e 9.450 non hanno un ROA corrispondente in RPKI.

==REMINDER:
- ==**irregolari** -> hanno conflitti o sono inconsistenti con altri dati di routing come BGP o RPKI
- ==**coerenti** -> sono allineati con altri dati di routing
- ==**ASN non corrispondente** -> quando l'ASN associato ad un prefisso IP non matcha le info ASN contenute in BGP o RPKI==

Hanno anche confrontato i route objects con l'elenco di dirottatori seriali e hanno trovato 5.581 oggetti di percorso registrati da 168 AS dirottatori seriali.


### False fonti di deduzione: Società che fanno leasing di IP

<mark style="background: #FF5582A6;">Il 30,4% (10.408 / 34.199) di route objects irregolari erano registrati da ipxo.com, una società di leasing IP. </mark>

<mark style="background: #FF5582A6;">La società ha registrato 738 AS sotto diversi manutentori in RADB.</mark>

<mark style="background: #FF5582A6;">Questi AS mostravano attività BGP sporadiche, con una durata degli annunci che andava da 10 minuti a più di 500 giorni.</mark>


## ALTDB Analysis

Sono stati trovati 1.206 prefissi univoci che non erano coerenti con i database IRR autorevoli. 
Di questi,  918 completamente sovrapposti con BGP, 5 parzialmente sovrapposti e 12 senza sovrapposizione. 

Sono stati mappati i 5 prefissi parzialmente sovrapposti su 11 origini di prefissi in BGP. Degli 11 prefissi IP studiati 5 erano sospetti.

Uno aveva un prefisso registrato da AS58202, una rete georgiana senza clienti, fornitori o peer, hanno annunciato il prefisso, che faceva parte di un prefisso più ampio di proprietà di Sprint, in BGP per sole 14 ore. 

Gli altri 4 prefissi facevano parte dello spazio degli indirizzi di Verizon, ma sono stati annunciati da AS non correlati per meno di 1 giorno. 



# Discussion

La mancanza di un meccanismo di convalida standardizzato e di un coordinamento tra i fornitori di IRR consente la persistenza di record obsoleti e configurati in modo errato, offrendo agli hacker l’opportunità di falsificare record IRR per scopi dannosi. 

I crescenti incentivi ad abusare del sistema di routing Internet (ad esempio rubando criptovalute) hanno motivato tecniche di dirottamento BGP sempre più complicate 

I database IRR sono soggetti a obsolescenza ed errori, confermando l'importanza della transizione degli operatori al filtraggio basato su RPKI. 

Sono state riscontrate incoerenze tra i database IRR, suggerendo opportunità per un migliore coordinamento tra i fornitori IRR per migliorare la sicurezza del routing. 

Infine, hanno descritto la difficoltà di dedurre la sospettità di tali oggetti irregolari e compilato un elenco di 6.373 route objects sospetti. 

