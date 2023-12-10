# Principali tipi di DNS records
| RECORD | DESCRIZIONE |
| -------- | -------- | 
| A | Restituisce l'indirizzo IPv4 di un dominio| 
| AAAA | Restituisce l'indirizzo IPv6 di un dominio | 
| CNAME | Associa un alias ad un nome di dominio canonico| 
| NS | Definisce quale è il nome autorevole per un dominio | 
| CAA | Definisce le Certificate Authority che possono dare un certificato ad un dominio | 


Alla fine, il server DNS autoritario restituisce al client un insieme di Resource Records (RR) con il formato: nome, TTL, classe, tipo, dati.

Il DNS supporta anche i record di risorse con l'etichetta *, come ad esempio *.example.com. 

I record con caratteri jolly non vengono confrontati se per il nome richiesto è stato definito un RR se per il nome richiesto è stato definito un RR esplicito. 

<mark style="background: #BBFABBA6;">In generale, gli RR con caratteri jolly hanno una priorità inferiore rispetto agli RR standard. </mark>

Per esempio, dato un record A con caratteri jolly *.example.com e un record A per a.example.com, le richieste a b.example.com e a b.example.com e c.b.example.com vengono risolte dal carattere jolly, mentre le richieste a a.example.com vengono soddisfatte dal corrispondente record A. 

Si noti che c.a.example.com non è risolvibile in questo caso 


### Public suffix list
I suffissi elencati nella PSL sono chiamati domini di primo livello effettivi (eTLD) e definiscono il confine tra i nomi che possono essere registrati dai privati e i nomi privati.

Un nome di dominio con una sola etichetta a sinistra di un suffisso pubblico viene comunemente definito dominio registrabile, eTLD+1 o dominio di vertice. 
==**I domini che condividono lo stesso eTLD+1 sono detti appartenere allo stesso sito.**==


![[Pasted image 20231208150847.png]]

# Related Domain Attacker

### Threat model
Nella sua definizione originale, l'attaccante di domini correlati è un aggressore web che gestisce un sito web dannoso ospitato su un dominio correlato al sito web bersaglio

==Due domini sono correlati se condividono un suffisso che non è incluso nel PSL. 
Ad esempio, si consideri il sito target example.com: tutti i suoi sottodomini sono correlati al target, oltre a essere correlati tra loro. (a.example.com e b.example.com)==

Possibili mosse che possono essere eseguite da un related-domain attacker:
![[Pasted image 20231208152025.png]]

| Capability | DESCRIZIONE |
| -------- | -------- | 
| headers | accedere e modificare gli headers http| 
| js | eseguire codice js | 
| html | moditifacre l'html senza poter usare però js| 
| content | modificare il contenuto del sito senza modificare tag e usare js | 
| file | hostare file arbitrari | 
| https | ottenere un  certificato valido per il sito malevolo |


### Dangling DNS records
==I record DNS pendenti si riferiscono ai record dei server DNS autorevoli di un dominio che puntano a risorse scadute.==

### Expired domains
Un record DNS CNAME mappa un nome di dominio (alias) a un altro chiamato nome canonico. 
==Se il nome canonico è scaduto, una terza parte può semplicemente registrare il dominio e servire contenuti arbitrari sotto il dominio alias.==

<mark style="background: #FF5582A6;">Qui tutte le capability sono abilitate per l'attaccante, ad eccezione di https perchè alcuni domini possono ottenere certificati solo da alcuni specifici CA (quindi più sicurezza), evitando l'accesso a Certificati automatici come quelli di Let's encrypt</mark>



### Discontinued Services
I servizi di terze parti sono ampiamente utilizzati per estendere le funzionalità di un sito web. 
I proprietari di domini possono integrare piattaforme ricche rendendole accessibili sotto un sottodominio della loro organizzazione, ad esempio blog.example.com potrebbe mostrare un blog ospitato da WordPress e shop.example.com potrebbe essere un e-shop gestito da Shopify. 

Per mappare un (sotto)dominio a un servizio, un integratore deve in genere 
	1) configurare un record DNS per il (sotto)dominio, come A/AAAA, CNAME o NS, in modo che punti a un server controllato dal fornitore del servizio
	2) rivendicare la proprietà del (sotto)dominio nelle impostazioni dell'account del servizio. 

<mark style="background: #FF5582A6;">Se il fornitore del servizio non verifica esplicitamente la proprietà del dominio, ossia un record DNS che punta al servizio è l'unica condizione richiesta per rivendicare la proprietà di un (sotto)dominio, un utente malintenzionato potrebbe mappare sul proprio account qualsiasi (sotto)dominio non rivendicato con un record DNS valido.</mark>


==I record dangling possono verificarsi anche a causa della presenza di caratteri jolly DNS. ==Si consideri, ad esempio, la configurazione da parte di un gestore di un sito di un carattere jolly DNS come ==* .example.com== che punta all'IP di un fornitore di servizi per consentire a più siti web di essere ospitati sotto i sottodomini di example.com. 
==Un utente malintenzionato potrebbe associare un sottodominio di sua scelta, ad esempio evil.example.com, a un nuovo account sul provider di servizi. ==



Alcuni service provider non verificano la proprietà di un sottodominio anche se il dominio padre è già stato mappato su un account esistente. In pratica, ==ciò consente a un aggressore di rivendicare evil.proj.example.com anche in presenza di un binding legittimo per proj.example.com.==


### Corporate networks and roaming services
<mark style="background: #BBFABBA6;">Le grandi organizzazioni spesso assegnano nomi di dominio completamente qualificati (FQDN) ai dispositivi della rete. 
</mark>
Questa pratica consente di fare riferimento staticamente alle risorse della rete, indipendentemente dall'assegnazione degli indirizzi IP che possono cambiare nel tempo. 

==Sebbene gli host possano essere inaccessibili dall'esterno della rete dell'organizzazione, gli utenti interni si trovano in una posizione di attacco al dominio correlato con tutte le funzionalità==, ad eccezione di https che dipende dalla configurazione di rete dell'organizzazione.


### Compromised Hosts/Websites
Un altro modo per ottenere una posizione di aggressore di dominio correlato è lo sfruttamento di host e siti web vulnerabili. 

<mark style="background: #FF5582A6;">Se la vulnerabilità sfruttata è un XSS, gli aggressori potrebbero sfruttare la capacità di eseguire codice JavaScript da una posizione privilegiata.</mark>

# Web Threats
## Minacce intrinseche
Qui gli attaccanti si trovano sullo stesso sito dell'applicazione web bersaglio. Questo è più debole della condivisione della stessa origine dell'obiettivo, che è il confine tradizionale della sicurezza web, ma è sufficiente per abusare della fiducia riposta dai fornitori di browser e dagli utenti finali nei contenuti dello stesso sito. 

### Fiducia negli utenti finali
<mark style="background: #FF5582A6;">Gli utenti finali potrebbero fidarsi maggiormente dei sottodomini di siti che conoscono bene rispetto a siti esterni arbitrari.
**Esempio evil.facebook.com** </mark> 

==Questo è particolarmente pericoloso su alcuni browser mobili, che visualizzano solo la parte più a destra del dominio a causa delle dimensioni ridotte del display, per cui un sottodominio lungo potrebbe apparire erroneamente come il sito principale. ==


### Isolamento del sito 
Site Isolation è un'architettura per browser che tratta siti diversi come processi di rendering separati.

==I related-domain attackers possono annullare i vantaggi di questa architettura di sicurezza in quanto un sotto dominio è visto come stesso sito.==
![[Pasted image 20231208155659.png]]

### Same Site Request Forgery
SameSite è una proprietà che può essere impostata nei cookie HTTP per prevenire gli attacchi Cross Site Request Forgery.

<mark style="background: #FF5582A6;">Un attaccante che è in possesso di un sotto dominio può però eseguire un Same Site Request Forgery semplicemente includendo un elemento HTML che punta al sito web di destinazione in una delle sue pagine web.</mark>


## Cookie Confidentiality and Integrity
I cookie possono essere emessi con l'attributo Domain impostato su un antenato del dominio che li imposta, in modo da condividerli con tutti i suoi sottodomini. 

<mark style="background: #FF5582A6;">Esempio good.foo.com imposta un cookie per foo.com così che possa essere condiviso con ogni sotto dominio del dominio foo.com</mark>. 

==**Quindi evil.foo.com può impostare cookie per good.foo.com per poter eseguire attachi.**==

==Anche l’integrità dei cookie impostati per essere condivisi con solo l'host è danneggiata, perché un utente malintenzionato del dominio correlato può montare il cookie shadowing, ovvero impostare un cookie di dominio con lo stesso nome di un cookie solo host per confondere il server web==

### Capabilities on cookies
Le capabilities di un attaccante dipendono dai flag settati sui cookies:
- se il cookie è HttpOnly, non può essere esfiltrato tramite JavaScript 
- se il flag Secure è abilitato, il cookie viene inviato solo tramite HTTPS

Per quanto riguarda l'integrità
- tutti i cookie senza il prefisso __ Host hanno una bassa integrità contro un attaccante che nel sottodominio che comanda può eseguire js
- i cookie che utilizzano il prefisso __ Secure- hanno una bassa integrità solo contro gli aggressori che hanno la funzionalità https


## Bypassing CSP
<mark style="background: #BBFABBA6;">La Content Security Policy (CSP) è un meccanismo di difesa lato client progettato per mitigare attacchi di injection ed altri, ad esempio il click-jacking.</mark>

Esempio:
```
script-src foo.com *.bar.com; //1
frame-ancestors *.bar.com; //2
default-src https: //3
```
Permette di
- caricare script solo dal dominio foo.com am anche da tutti i sottodomini di bar.com (* .bar.com) //1
- essere inclusa in frame in ogni pagina di tutti i sotto domini di bar.com //2
- includere qualsiasi contenuto che non sia script da ogni host che usa https //3

<mark style="background: #FF5582A6;">Se un attaccante prende il possesso di evil.bar.com riuscirebbe ad aggirare quasi tutte le protezioni di sopra</mark>


### Capabilities
Se l'attaccante può nel sotto dominio:
- eseguire js allora può eseguire XSS sul dominio con la CSP
- modificare html allora può effettuare attachi CSRF o comunque includere in un frame la pagina
- usare https allora può modificare il DOM della pagina


## Abusing CORS 

<mark style="background: #BBFABBA6;">Cross-Origin Resource Sharing (CORS) è l'approccio standard per allentare le restrizioni imposte da SOP sulle comunicazioni cross-origin.</mark>


I related-domain attackers possono abusare di CORS per aggirare le restrizioni di sicurezza messe in atto da SOP quando i suddetti controlli di autorizzazione lato server sono troppo rilassati, ovvero l'accesso in lettura è concesso a sottodomini arbitrari.

Ad esempio, se https://api.example.com fosse disposto a concedere l'accesso multiorigine a qualsiasi sottodominio di example.com oltre a www.example.com, un utente malintenzionato del dominio correlato potrebbe ottenere accesso illimitato ai suoi dati. 


### Capabilities
Per sfruttare le configurazioni errate di CORS, un utente malintenzionato ha bisogno della funzionalità js per inviare richieste tramite API JavaScript come recuperare e accedere al contenuto della risposta. 

La funzionalità https potrebbe essere richiesta a seconda della politica CORS implementata dall'operatore del sito.


## Abusing postMessage API
<mark style="background: #BBFABBA6;">L'API postMessage supporta la comunicazione multiorigine tra finestre (ad esempio, tra frames o tra una pagina e il popup da essa aperto).</mark>

<mark style="background: #FFB86CA6;">Quando si inviano dati confidenziali, si dovrebbe sempre specificare l'origine del destinatario previsto nell'invocazione postMessage. 
Quando si ricevono dati, invece, è opportuno verificare l'origine del mittente e validare il contenuto dei messaggi.</mark>



Gli attaccanti possono tentare di abusare della propria posizione per annullare i controlli sull'origine e comunicare con destinatari disattenti che potrebbero elaborare i messaggi in modo non sicuro.

### Capabilities
Se l'attaccante può eseguire js allora potrebbe comunicare con una pagina vulnerabile untilizzando postMessage API.



## Abusing domain relaxation
<mark style="background: #BBFABBA6;">Il rilassamento dei domini è il modo tradizionale per implementare la comunicazione tra finestre i cui domini condividono un antenato comune.</mark>

Supponiamo che una pagina su a.example.com apra una pagina su b.example.com all'interno di un frame.

==I due frame possono comunicare rilassando la loro proprietà document.domain su un antenato comune tipo example.com . ==

Il rilassamento del dominio può essere abusato da related-domain attackers, che possono cercare pagine disposte a impegnarsi in un meccanismo di comunicazione così pericoloso e abusarne.

### Capabilities
L'attaccante deve avere accesso a js per modificare il DOM dell'altra pagina e devono avere accesso allo stesso protocollo dell'altra pagina, quindi può essere richiesto https.


# Analysis Methodology
![[Pasted image 20231209160603.png]]
## DNS data collection
La fase di enumerazione si è basata su amass, un tool.

Nella nostra configurazione, abbiamo estratto i sottodomini utilizzando i seguenti approcci:
- recuperare dati da fonti disponibili al pubblico, come Censys, registri di trasparenza dei certificati , motori di ricerca, ecc.; 
- tentare il trasferimento di zona DNS per ottenere l'elenco completo degli RR definiti per una determinata zona DNS; 
- ispezionare i campi dei certificati TLS, ad esempio Nome alternativo soggetto e Nome comune.

Per i record CNAME e NS, eseguiamo ricorsivamente una risoluzione DNS finché non viene rilevato un RR A/AAAA.

Per garantire la correttezza dei risultati, ricalcoliamo le catene di risoluzione DNS non terminate utilizzando l'utilità dig.


## RDScan
==Dopo aver popolato un database con i record DNS dei sottodomini scoperti, il framework rileva i record pendenti e verifica che siano soddisfatte tutte le precondizioni per sferrare un attacco di subdomain takeover. ==

### Expired domains
Il rilevamento dei domini scaduti viene effettuato secondo la seguente procedura: 
- <mark style="background: #BBFABBA6;">data una catena risolutiva che inizia con un record CNAME, il nostro strumento verifica se punta a una risorsa irrisolvibile ed estrae l'eTLD+1 del nome canonico al fine della catena</mark>
- <mark style="background: #BBFABBA6;">se il comando whois sul dominio apex non restituisce alcuna corrispondenza, RDScan interroga GoDaddy per rilevare se il dominio può essere acquistato.</mark>


### Discontinued Services
Questo passaggio viene implementato in base alla documentazione fornita dai singoli servizi e in genere si basa sulla verifica della presenza di 
- (i) un record A che si risolve in un indirizzo IP specifico, 
- (ii) il nome canonico di un CNAME record corrispondente a un dato host 
- (iii) l'esistenza di un record NS che punta al server DNS di un servizio. 

I (sotto)domini associati ai servizi vengono quindi controllati per verificare se sono presenti collegamenti tra gli account utente e i (sotto)domini.

RDScan esegue anche il rilevamento dei caratteri jolly DNS. 
Un carattere jolly DNS per un dominio come test.example.com può essere facilmente rilevato tentando di risolvere un record CNAME o A DNS per nonce.test.example.com

### Deprovisioned cloud instances
Abbiamo raccolto l'insieme di intervalli IP di 6 principali provider: Amazon AWS, Google Cloud Platform, Microsoft Azure, Hetzner Cloud, Linode e OVHcloud.

Abbiamo testato ciascun (sotto)dominio nel nostro set di dati per verificare se l'IP indicato fosse incluso in uno qualsiasi degli intervalli IP del cloud.

Come ultimo passaggio, eseguiamo un'analisi dell'attività per determinare se l'IP è in uso. Questo viene fatto eseguendo un ping sull'IP: se non viene ricevuta alcuna risposta, facciamo una scansione dell'intero intervallo IPv4 su 148 porte (128 TCP, 20 UDP).

## Web Analyzer
### Analysis of cookies
Abbiamo utilizzato l'API Puppeteer per raccogliere i cookie impostati tramite intestazioni HTTP e JavaScript.

Contrassegniamo un cookie come affetto da problemi di riservatezza se, tra i relativi domini vulnerabili al takeover, esiste un dominio dom tale che:
- dom è un sottodominio dell'attributo Domain del cookie; 
- usando dom, l'aggressore ha acquisito le capacità necessarie per diffondere il cookie.

Contrassegniamo un cookie come affetto da problemi di integrità se: 
- il nome del cookie non inizia con __ Host-

Ci basiamo anche su un'euristica proposta da Bugliesi per identificare staticamente cookie che potrebbero essere rilevanti per la gestione delle sessioni utente.

Per compromettere l'integrità è necessaria la funzionalità js o le intestazioni e, se viene utilizzato il prefisso __ Secure-, è necessario anche https.


### CORS analysis
Per valutare la sicurezza della politica CORS implementata da un sito Web, eseguiamo più richieste con valori Origin diversi e ispezioniamo le intestazioni HTTP nella risposta per capire se CORS è stato abilitato dal server.

Vengono testate 3 cose:
- il dominio in possesso è correlato a quello che stiamo testando
- il dominio inizia come quello testato
- il dominio termina come quello testato

Per ogni test controlliamo se l'intestazione Access-Control-Allow-Origin è presente nella risposta e se il suo valore è * o quello dell'intestazione Origin contenuta nella richiesta. 

Controlliamo anche se l'intestazione Access-Control-Allow-Credentials = true (quando Access-Control-Allow-Origin è diverso da *  perchè è disabilitato in quel caso) per identificare i casi in cui le richieste con credenziali sono consentite.

### postMessage handler analysis
Si usa il framework PMForce per estrarre i vincoli sul contenuto del messaggio (ad esempio, la presenza di una determinata stringa nel messaggio) che portano a tracce di esecuzione in cui il messaggio entra in un pericoloso sink che consente l'esecuzione del codice (ad esempio , eval) o l'alterazione dello stato del browser (es. document.cookie). 

Abbiamo integrato PMForce nella nostra pipeline e l'abbiamo modificato per generare, per ciascun handler, più messaggi di exploit con gli stessi contenuti ma una proprietà di origine diversa, ad esempio un'origine di dominio correlato e un'origine intersito generata casualmente. 

Consideriamo una pagina vulnerabile a qualsiasi attacco web se uno qualsiasi dei suoi gestori è sfruttabile da una posizione tra siti. 
Consideriamo invece una pagina sfruttabile da un utente malintenzionato del dominio correlato se i suoi gestori possono essere sfruttati solo da un dominio correlato.


### Domain relaxation analysis
Come primo passo, l'analizzatore rileva se la proprietà document.domain è impostata dopo il caricamento della pagina. 

Questo compito è semplice tranne nel caso in cui la pagina imposta la proprietà al suo valore originale poiché questo non può essere rilevato semplicemente leggendo il valore di document.domain. 

Per identificare questo caso particolare, sfruttiamo le API Puppeteer per:
- iniettare un frame da un sottodominio (generato casualmente) della pagina in analisi
- intercettare la richiesta di rete in uscita e fornire come risposta una pagina con uno script che esegue il rilassamento del dominio e tenta di accedere al frame genitore, cosa che riesce solo se il genitore ha impostato document.domain. 

​





## PICCOLA SINTESI
==I domini scaduti vengono banalmente verificati controllando se il dominio di destinazione può essere acquistato. ==

==Per i servizi fuori servizio, abbiamo creato account di prova personali su ciascun servizio considerato nell'analisi e abbiamo utilizzato questi account per sondare la mappatura tra il sottodominio di destinazione e il servizio. 
Se rileviamo tutte le condizioni necessarie per associare il sottodominio al nostro account, lo consideriamo vulnerabile.==




## SECURITY EVALUATION
Abbiamo valutato i servizi considerati rispetto alle trappole di sicurezza: 
- (i) jolly, la verifica della proprietà del dominio consente agli aggressori di rivendicare sottodomini di un dominio già mappato, ad esempio, a causa della presenza di un carattere jolly voce DNS; 
- (ii) reindirizzamento, se il sottodominio www di un dominio mappato reindirizza automaticamente al dominio principale, ad esempio www.shop.example.com reindirizza a shop.example.com, se il primo può essere rivendicato da un account diverso; 
- (iii) PSL, se il servizio consente agli utenti di creare un sito web sotto uno specifico sottodominio, se il dominio principale del sito web assegnato è incluso nel PSL.




# CONCLUSIONI
<mark style="background: #BBFABBA6;">La toolchain è costituita da un modulo di analisi per l'acquisizione di sottodomini che 
identifica quali sottodomini possono essere violati da un utente malintenzionato. </mark>

<mark style="background: #BBFABBA6;">Successivamente, il modulo di sicurezza web quantifica quanti domini correlati possono essere attaccati dai domini scoperti nel passaggio precedente. </mark>


Abbiamo eseguito un'analisi su larga scala sui 50.000 domini più popolari e abbiamo identificato le vulnerabilità in 887 di essi, inclusi i principali siti Web come cnn.com e cisco.com. 

==Quindi, abbiamo correlato per la prima volta l’impatto di queste vulnerabilità sulla sicurezza delle applicazioni web, dimostrando che gli aggressori dei domini correlati ottengono un ulteriore vantaggio rispetto agli aggressori web che va oltre i tradizionali problemi legati ai cookie.==






<mark style="background: #FF5582A6;">L'unico modo efficace per migliorare l'integrità dei cookie in questa impostazione è l'adozione del prefisso __ Host</mark>

<mark style="background: #FF5582A6;">Gli aggressori di domini correlati sono più potenti dei tradizionali aggressori web per i CSP del mondo reale, essendo in grado di aggirare il meccanismo di protezione.</mark>

<mark style="background: #FF5582A6;">Le implementazioni CORS sono significativamente più a rischio contro gli aggressori dei domini correlati piuttosto che contro gli aggressori web tradizionali. </mark>

<mark style="background: #FF5582A6;">Lo sfruttamento del rilassamento del dominio pone un aggressore del dominio correlato nella stessa origine dell'applicazione web di destinazione, aggirando quindi tutti i confini della sicurezza web.</mark>

<mark style="background: #FF5582A6;">L'analisi su postMessage mostra che tutti i siti che soffrono di pratiche di programmazione non sicure sono già vulnerabili contro gli aggressori web, vale a dire che per questo specifico vettore di attacco gli aggressori legati ai domini non sono più potenti degli aggressori web tradizionali.</mark>
