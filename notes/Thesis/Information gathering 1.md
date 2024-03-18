
# When Wells Run Dry: The 2020 IPv4 Address Market


Ad oggi, il mondo ha quasi esaurito gli indirizzi IPv4 non allocati per soddisfare la domanda sempre crescente di nuovi indirizzi. 

Tre RIR, vale a dire ARIN, LACNIC e RIPE NCC, hanno esaurito il loro pool di indirizzi IPv4 non allocati e gli ultimi due RIR, APNIC e AFRINIC, attualmente allocano rispettivamente dall'ultimo blocco di indirizzi /10 e /11. 

<mark style="background: #BBFABBA6;">Tutti i RIR hanno ridotto la dimensione dei blocchi IPv4 che un nuovo membro può ricevere</mark> (ad esempio, il RIPE NCC assegna solo /24 prefissi IPv4).

Di conseguenza, le reti potrebbero dover utilizzare metodi alternativi per acquisire lo spazio degli indirizzi IPv4: leasing e acquisto. 

Sebbene tecniche come il NAT carrier-grade (CGN) riducano la necessità di indirizzi IP pubblici, non la eliminano. 

REMINDER:
- **==RIR -> Regional Internet Registries==**
## Getting IP resources

Per partecipare attivamente all’ecosistema di routing BGP di Internet, le organizzazioni necessitano di risorse IP. 

<mark style="background: #BBFABBA6;">L'Internet Assigned Numbers Authority (IANA) gestisce le allocazioni gerarchiche di indirizzi IP e numeri AS ai cinque Regional Internet Registries (RIR). </mark>

Questi RIR, o
- I RIR AFRINIC (regione africana), 
- APNIC (regione Asia-Pacifico), 
- ARIN (regione americana), 
- LACNIC (regione dell’America Latina)
- RIPE NCC (regione europea e mediorientale) 
sono responsabili dell’assegnazione degli indirizzi, contabilità e sostegno alla comunità.

Esistono tre opzioni per ottenere nuove risorse IPv4: 
- **==aderire a un RIR come membro e richiedere un nuovo spazio di indirizzi==** 
- **==aderire a una RIR e acquistare indirizzi==**
- ==**leasing**== 


## RIR membership
I RIR sono organizzazioni basate sull'appartenenza: se un'organizzazione riceve uno spazio di indirizzi da un RIR, in genere diventa membro di quel RIR, un registro Internet locale (LIR).

I RIR sviluppano le loro politiche operative attraverso discussioni tra i loro membri.
- I LIR possono influenzare le politiche del loro RIR. Per diventare e rimanere una LIR, un'organizzazione deve pagare una quota associativa annuale più quote a seconda del numero di risorse richieste. 


## Verso l'esaurimento degli indirizzi IPV4
Tutti i RIR mantengono pool di indirizzi IPv4 e IPv6. 
==**Ricevono indirizzi da IANA, altri RIR o organizzazioni che non necessitano più dei loro IP allocati.** 
**Da quando IANA ha assegnato gli ultimi blocchi di indirizzi IPv4 rimanenti ad APNIC il 31 gennaio 2011, i pool RIR non possono più ricevere indirizzi IPv4 riservati.** 

==**Di conseguenza, i RIR hanno presto raggiunto il loro ultimo /8 .**==
![[Pasted image 20240317111156.png]]

Tutti i RIR hanno ora politiche di assegnazione più restrittive.




## Esaurimento IPV4
Attualmente tutti i RIR recuperano lo spazio degli indirizzi IP se un'organizzazione chiude o se i criteri originali per l'assegnazione iniziale non sono più soddisfatti. 

APNIC contatta attivamente i membri che hanno ricevuto deleghe che non sono almeno parzialmente visibili nel sistema di routing globale. 

<mark style="background: #BBFABBA6;">Dopo aver recuperato lo spazio degli indirizzi IP e rimosso gli oggetti associati dal database, la maggior parte dei RIR mette i blocchi in un periodo di quarantena di sei mesi prima di ridistribuirli nuovamente. 
</mark>

Di conseguenza, le politiche di assegnazione sono diventate più restrittive e la maggior parte dei RIR ha introdotto liste di attesa. 

Oggi, AFRINIC, ARIN  e LACNIC limitano lo spazio degli indirizzi assegnabili per organizzazione a /22. Per APNIC è un /23 e per RIPE un /24. 
- Ciò non significa che i RIR obblighino le organizzazioni a restituire lo spazio degli indirizzi se in precedenza hanno ricevuto un blocco di indirizzi più grande. 


## Acquisto di indirizzi IP
Molti LIR hanno iniziato ad acquistare ulteriori spazi di indirizzi IPv4 (i diritti di utilizzo). 

Durante una transazione, un LIR acquirente paga il LIR venditore in modo tale che quest'ultimo invoca un trasferimento di risorse dello spazio degli indirizzi all'acquirente. 
- Questo trasferimento di risorse può comportare pagamenti aggiuntivi al RIR. 
- Dopo il trasferimento, il LIR acquirente è responsabile dei costi annuali di mantenimento delle risorse del RIR. 

<mark style="background: #BBFABBA6;">I broker IPv4 certificati aiutano a facilitare questo processo. </mark>
- <mark style="background: #BBFABBA6;">Collegano l'acquisto e la vendita dei LIR, li aiutano nella negoziazione dei prezzi e spesso gestiscono le formalità dei trasferimenti di indirizzo.
</mark>

## Leasing di IPV4
Mentre gli Internet Service Provider (ISP) affittano lo spazio degli indirizzi ai propri clienti per molto tempo, recentemente si è registrato un aumento del numero di organizzazioni che affittano il proprio indirizzo a qualsiasi organizzazione indipendentemente dagli accordi di routing o connettività.

**==I fornitori di leasing sono LIR che delegano temporaneamente i diritti di utilizzo per parte del loro spazio di indirizzi a un cliente.==** 
- ==**Sebbene il leasing non comporti alcun trasferimento di risorse presso il RIR, può comprendere l'alterazione di oggetti nel database WHOIS, un database che contiene informazioni su risorse Internet, organizzazioni e persone di contatto.** ==

Un contratto di leasing può limitare l'utilizzo dell'indirizzo e può includere accordi di hosting o connettività di rete, o entrambi. Per gli hoster, lo spazio degli indirizzi affittato si trova solitamente ancora nel proprio AS. 


## Non tutti gli IP sono uguali

Le black lists basate su IP sono diventate molto popolari per mitigare le attività dannose, ad esempio lo spam tramite posta elettronica o gli attacchi di tipo Flooding. 

Una volta che un indirizzo IP bloccato appare in una lista nera, può essere difficile rimuoverlo nuovamente: l'indirizzo IP è contaminato. 

I blocchi di indirizzi IP che non sono mai comparsi in una lista nera e non sono associati ad alcuna attività dannosa sono noti come "IP puliti". 

==**Per mantenere puliti i blocchi di indirizzi, i fornitori di leasing spesso richiedono informazioni su come un potenziale cliente intende utilizzare le risorse noleggiate.** ==

==**Inoltre, i fornitori di leasing spesso installano dati di registro, ad esempio record condivisi del progetto WHOIS (noti anche come record SWIP ), per proteggere il loro spazio di indirizzi rimanente dall'essere inserito nella lista nera quando viene rilevato spam in un blocco delegato.** ==

Allo stesso modo, la maggior parte dei LIR controlla la “reputazione” dei blocchi di indirizzi prima di acquistarli per garantire che gli indirizzi siano raggiungibili a livello globale.



## Reseller di IP
La Figura 2 mostra il numero di trasferimenti aggregati nell'arco di tre mesi per ciascuna regione da ottobre 2009 a giugno 2020.
![[Pasted image 20240317113049.png]]


Per comprendere il costo di acquisto di indirizzi IPv4, utilizziamo le informazioni sui prezzi disponibili pubblicamente da IPv4.Global nonché le informazioni sui prezzi private ottenute da Brander Group, IPTrading.com e IPv4 Market.

In totale, abbiamo ottenuto informazioni sui prezzi per 2,9mila transazioni tra il 1° gennaio 2016 e il 25 giugno 2020. 

La Figura 1 mostra le informazioni sui prezzi come box plot raggruppati per dimensione del prefisso, regione e intervallo di tre mesi. 
![[Pasted image 20240317113400.png]]


Innanzitutto osserviamo che non vi è alcuna differenza statistica nei prezzi tra le regioni, vale a dire che il fatto che un prefisso sia assegnato alla regione APNIC, ARIN o RIPE NCC non ha un impatto significativo sul suo prezzo. 

<mark style="background: #BBFABBA6;">Al contrario, acquistare indirizzi IP in blocchi /24 o /23 è più costoso che acquistare blocchi di indirizzi più grandi; quando un broker decide di vendere un grande blocco di indirizzi in piccole parti separate, i costi secondari associati aumentano. </mark>

Il broker non solo deve trovare più acquirenti, ma deve anche avviare più trasferimenti separati. 

Nel complesso, osserviamo che i prezzi, indipendentemente dalla dimensione o dalla regione effettiva del prefisso, sono raddoppiati dal 2016, il che è correlato alla diminuzione della disponibilità di blocchi di indirizzi non allocati dai RIR.

A partire dalla primavera del 2019, il mercato IPv4 sembra essere entrato in una fase di consolidamento, ovvero in uno stato in cui il prezzo di mercato cambia poco e in cui il numero di trasferimenti non corrisponde più alla domanda effettiva. 


## IPV4 Leasing market
lll leasing di IP è spesso disponibile in meno di un giorno e richiede solo pagamenti mensili. 
Per comprendere fino a che punto viene attualmente utilizzata l'opzione di leasing, analizzano due fonti di dati:
1. ==**deducono accordi di leasing dalle informazioni di instradamento rivisitando il concetto di delegazioni BGP**==
2  ==**utilizzano un'istantanea del database WHOIS di RIPE e query sul suo database RDAP per tenere traccia delle deleghe di blocco degli indirizzi.** ==



### Deduzione delle deleghe BGP. 

==**Affittare lo spazio degli indirizzi IP è economicamente utile solo se l'organizzazione annuncia il prefisso anche all'interno dell'ecosistema BGP.** ==

Pertanto, la maggior parte dello spazio di indirizzi affittato dovrebbe essere visibile come spazio di indirizzi delegato per cui il fornitore di leasing può ancora annunciare un prefisso meno specifico. 

Diciamo che un delegante AS S possiede un prefisso P e delega un sottoprefisso più specifico P′ a un delegato AS T . Deduciamo una delega P'ST se osserviamo che S e T originano P e P ′ , rispettivamente. 

Per studiare tali deleghe:
1. ==**Otteniamo l'insieme di tutte le coppie prefisso-origine**==
2. ==**Rimuoviamo tutte le coppie viste da meno della metà di tutti i monitor BGP per garantire visibilità globale:==** ciò limita l'impatto, ad esempio, di errate configurazioni locali o dirottamenti BGP diffusi localmente
3. **==Rimuoviamo le coppie per le quali il rispettivo prefisso è originato da un AS_SET o più di un AS==**
4. Basandosi sulla mappatura AS-to-Organization di CAIDA, rimuoviamo le deleghe tra AS della stessa organizzazione all'interno del successivo snapshot disponibile.
5. **==Si aggiungono le deleghe temporaneamente non presenti sulla base di approfondimenti derivanti dall'analisi della coerenza delle deleghe in RPKI==**: se osserviamo la stessa delegazione a dieci giorni di distanza senza osservare una delega in conflitto (ad es. , osserviamo che P viene delegato ad un altro delegato AS T ′ ) presumiamo che la delega esista anche per tutti i giorni intermedi. 

## Limitazioni
Nonostante prenda precauzioni contro i dirottamenti, l'algoritmo può comunque considerare una delega tra l'AS vittima e l'AS dirottante se il dirottamento viene eseguito utilizzando un prefisso più specifico. 

L'algoritmo potrebbe dedurre erroneamente deleghe in combinazione con servizi di scrubbing basati su BGP (ovvero servizi che annunciano i prefissi dei clienti, analizzano e eliminano il traffico dannoso in entrata e incanalano il restante traffico "pulito" verso i clienti). 


## Delegazioni BGP
Applicano l'algoritmo di inferenza alle informazioni di routing raccolte da RIPE RIS, Route Views e Isolario tra il 1 gennaio 2018 e il 1 giugno 2020. 

hanno aggregato i dati quotidianamente
Per pulire i dati, hanno rimosso tutti i percorsi per lo spazio degli indirizzi privati e riservati , i prcorsi che contengono AS attualmente riservati da IANA e i percorsi che contengono un loop nel loro AS-PATH. 

La loro tecnica riduce significativamente il numero di deleghe dedotte ma elimina la grande varianza prodotta dall'approccio precedente
![[Pasted image 20240317114939.png]]





## Delegazioni RDAP. 
<mark style="background: #BBFABBA6;">Alcuni RIR mantengono interfacce RDAP (Registration Data Access Protocol) accessibili al pubblico progettate per sostituire eventualmente il protocollo WHOIS.</mark> Come WHOIS, questo database contiene informazioni di registrazione ma è più ampio. 

Se un LIR assegna lo spazio degli indirizzi a un "end-host" , il campo *parentHandle* contiene un identificatore univoco RIR per la rete madre: questo può essere utilizzato per dedurre le deleghe. 

<mark style="background: #BBFABBA6;">Poiché le interfacce RDAP non consentono richieste di caratteri jolly o di intervallo, si affidano agli oggetti inetnum di un'istantanea WHOIS corrente come spazio di input per le query RDAP. </mark>

Innanzitutto, selezionano tutti gli oggetti inetnum dal database WHOIS di RIPE con i tipi relativi alla delega: 
- il tipo PA SUB-ALLOCATO si riferisce allo spazio di indirizzi sub-allocato a un'altra organizzazione
- il tipo PA ASSEGNATO si riferisce allo spazio di indirizzi assegnato da un LIR a un end-host.

Dopo il filtraggio restano 181.000 delegazioni basate su RDAP.

## Delegazioni BGP e delegazioni RDAP
Confrontando le delegazioni identificate tramite BGP nel giugno 2020 con quelle delle delegazioni RDAP osserviamo che: 
- le delegazioni BGP coprono solo circa l'1,85% degli IP delegati da RDAP mentre le delegazioni RDAP coprono circa il 65,7% degli IP delegati da BGP nel Regione RIPE (utilizzando). 

Questa copertura limitata delle delegazioni BGP implica che il mercato del leasing è significativamente più ampio di quanto previsto da lavori precedenti. 

La copertura limitata delle deleghe BGP può essere dovuta a:
1. il presupposto che il prefisso delegato e il prefisso di copertura siano annunciati all'interno dell'ecosistema BGP
2. anche se il delegato lo annuncia, il prefisso più specifico può essere aggregato e non è più globalmente visibile
3. i grandi LIR spesso delegano blocchi di indirizzi di medie dimensioni agli ISP. Questi ISP utilizzano parte dello spazio degli indirizzi ma riservano parti significative per i futuri clienti. Quest'ultimo è invisibile in BGP. 

Sebbene le organizzazioni abbiano incentivi a inserire i propri contratti di leasing nei database WHOIS e RDAP (ad esempio, per ridurre il rischio di inserimento nella lista nera a causa di attività dannose in un prefisso affittato ), non tutti i fornitori di leasing richiedono voci. 


==**Le deleghe RDAP sono complementari alle deleghe BGP: la prima cattura i processi amministrativi, la seconda l'utilizzo effettivo.** ==
Nessuno dei due può catturare tutti i contratti di leasing. Pertanto, la combinazione di entrambi i tipi di dati è essenziale per stimare la dimensione del mercato del leasing IPv4. 


REMINDER:
- **==le deleghe RDAP sono utilizzate per gestire le informazioni di registrazione degli indirizzi IP (quindi a chi "appartiene" l'IP)**
- ==**le deleghe BGP sono utilizzate per controllare gli annunci di routing per gli indirizzi IP assegnati (chi ha il permesso di inviare annunci BGP per questi indirizzi IP e quali prefissi di rete sono inclusi in questi annunci)==**


## Prezzi di leasing 

Anche se alcuni siti web offrono sconti fino al 10% quando si noleggiano prefissi di dimensioni maggiori o si stipulano contratti di più mesi, hanno considerato i prezzi per il noleggio a /24 per un singolo mese. 
![[Pasted image 20240317121114.png]]

In generale, osserviamo che i prezzi variano sostanzialmente: da $ 0,30 a $ 2,33 per IP al mese.


## RELATED WORK
Nel 2011, Osterweil et al. hanno discusso le implicazioni di possibili regolamenti per i trasferimenti IPv4. Hanno sostenuto l’utilizzo di ulteriori record DNS inversi per convalidare la proprietà IP. Giotsas et al. hanno segnalato incongruenze tra i dati di trasferimento RIR e le informazioni dedotte dai set di dati BGP. Sottolineano inoltre che i blocchi trasferiti di frequente hanno maggiori probabilità di essere coinvolti in attività dannose.




## Discussion

La domanda di indirizzi IPv4 attualmente supera l’offerta. Di conseguenza, i potenziali clienti sono attualmente disposti a pagare prezzi più alti.le deleghe RDAP sono utilizzate per gestire le informazioni di registrazione degli indirizzi IP (quindi a chi "appartiene" l'IP)**
- ==**le deleghe BGP sono utilizzate per controllare gli annunci di routing per gli indirizzi IP assegnati (chi ha il permesso di inviare annunci BGP per questi indirizzi IP e quali prefissi di rete sono inclusi in questi annunci)==

Il modo in cui un'organizzazione interagisce con i mercati del leasing e dei trasferimenti è spesso fortemente correlato al suo modello di business: 
- i fornitori di servizi Internet (ISP) spesso acquistano blocchi più grandi di /20 con l'intento di affittarne parti a (potenziali) clienti
- i clienti a lungo termine acquistano uno spazio di indirizzi inferiore a /20 per soddisfare le proprie esigenze di indirizzamento e rescindono i contratti di locazione degli indirizzi con, ad esempio, un ISP

Molti provider VPN affittano continuamente lo spazio degli indirizzi ma spesso "ruotano" gli IP effettivi in modo tale che sia più difficile bloccare il loro servizio. 

==**Infine, gli spammer utilizzano spesso contratti di leasing di breve durata di varie dimensioni, garantendo al tempo stesso che il proprio spazio di indirizzi rimanga pulito quando intraprendono attività dannose.** ==

Un'altra strategia che abbiamo riscontrato nelle discussioni con i broker è l'acquisto e il lease back: in questo modello le organizzazioni che possiedono più spazio di indirizzi IPv4 di quello che utilizzano attualmente (ad esempio, l'ISP) vendono questo spazio di indirizzi a un broker e, in cambio, affittano solo la quantità che necessità con termini precedentemente concordati nel caso in cui avessero bisogno di spazio aggiuntivo. 

Le politiche attive richiedono la restituzione degli indirizzi non utilizzati, ma l'attuale situazione del mercato fornisce pochi incentivi a liberare le risorse acquisite. 

Pertanto, il numero di indirizzi IPv4 disponibili toccherà presto il fondo, al punto che l’implementazione mondiale di IPv6 diventerà inevitabile per i servizi futuri. Con i progressi nelle implementazioni IPv6, l’attenzione potrebbe spostarsi dagli indirizzi IPv4. 



## Appenxid


### Delegations and RPKI
Per osservare le deleghe nei dati BGP è necessario annunciare lo spazio degli indirizzi delegati. 

Poiché l'implementazione della convalida dell'origine del percorso è aumentata in modo significativo, il database Resource Public Key Infrastructure (RPKI) è diventato una fonte preziosa per dedurre le deleghe. 

Invece di prendere gli annunci di P e P ′ , ora controlliamo se tali prefissi hanno autorizzazioni di origine del percorso (ROA) assegnate a diversi AS. 

Le deleghe basate su RPKI forniscono una visione diversa sulla coerenza della delega: se S ha una ROA assegnata per P e delega P′ a T, allora T ha bisogno continuamente di avere una ROA per P durante l’intero periodo di delega, altrimenti molti Gli AS filtrerebbero e non propagherebbero il prefisso delegato P ′. 

Se osserviamo una delega il giorno X e il giorno X + M, la delega esiste anche per tutti i giorni tranne N nel mezzo.

Nella Figura 5, è presente il tasso di fallimento per regole con valori diversi per N sull'asse y rispetto a valori crescenti di M (ovvero, intervallo di tempo crescente) sull’asse x. 

![[Pasted image 20240317140636.png]]

Innanzitutto, osserviamo che circa il 90% delle delegazioni viste ad almeno 90 giorni di distanza sono visibili per l'intero intervallo di tempo.

Infine, vediamo che anche quando N è 0 (ovvero, la delega deve essere visibile per tutti i giorni all'interno dell'intervallo temporale) la regola viene violata solo per circa il 5% di tutte le possibilità quando l'intervallo temporale è di 10 giorni.

Hanno deciso di applicare la seguente regola di coerenza a tutte le deleghe BGP: quando osserviamo la stessa delega per dieci giorni e non c'è una delega in conflitto (ovvero osserviamo che P viene delegato a un altro delegato AS T ′ ) allora la delega esiste anche per tutti i giorni intermedi.

### Delegations
La Figura 6 mostra il numero di deleghe e la quantità di spazio di indirizzi delegati dedotto dagli algoritmi precedenti. 
![[Pasted image 20240317141121.png]]



