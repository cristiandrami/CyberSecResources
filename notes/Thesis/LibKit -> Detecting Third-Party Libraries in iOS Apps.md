
# Abstract 
Presentiamo LibKit, il primo approccio e strumento per rilevare il nome e la versione delle librerie di terze parti (Third-Party Libraries) presenti nelle app iOS. 

LibKit crea automaticamente le fingerprints per 86K versioni di librerie disponibili attraverso il gestore delle dipendenze CocoaPods e le confronta con gli eseguibili decriptati delle app per identificare le Third-Party Libraries (nome e versione) utilizzate da un'app iOS. 

LibKit supporta le app scritte in Swift e Objective-C, rileva le librerie collegate staticamente e dinamicamente e affronta problemi quali le librerie parzialmente incluse e le diverse versioni e configurazioni del compilatore che producono varianti della stessa versione di libreria. 


# Introduction

Negli ecosistemi iOS e Android, la maggior parte delle app mobili e molte Third-Party Libraries sono closed-source, quindi gli approcci di identificazione della Third-Party Libraries devono operare sui pacchetti di app rilasciati (IPA per iOS, APK per Android) e dovrebbero essere in grado di identificare le Third-Party Libraries anche se il codice della libreria sorgente non è disponibile. 

L'identificazione della Third-Party Libraries comporta tre problemi. 
- <mark style="background: #BBFABBA6;">Identificazione dei confini</mark> -> dividere il codice di un'app (ad esempio, eseguibile Mach-O per iOS, bytecode DEX per Android) in componenti in cui un componente corrisponde al codice dell'app e c'è un componente aggiuntivo per ogni Third-Party Libraries. 
- <mark style="background: #BBFABBA6;">Identificazione della libreria</mark> -> restituire i nomi di tutti i Third-Party Libraries utilizzati da un'app. 
- <mark style="background: #BBFABBA6;">identificazione della versione della libreria</mark> -> restituire il nome e la versione di tutte le Third-Party Libraries utilizzate da un'app. 

Alcuni lavori hanno proposto approcci di identificazione della Third-Party Libraries iOS. Tuttavia, questi approcci utilizzano tecniche basate sul clustering che affrontano solo il problema dell'identificazione dei confini (ad esempio, CriOS) o sono specifici per Third-Party Libraries disponibili sia in Android che in iOS. 

LibKit segue i precedenti approcci di identificazione Third-Party Libraries Android che operano in due fasi:
- generazione della fingerprint della libreria 
- e rilevamento della libreria. 

Il vantaggio principale rispetto alle tecniche basate sul clustering iOS è che il nostro approccio può nominare una libreria e identificarne la versione, oltre a identificare i confini tra il codice dell'app e quello del Third-Party Libraries. 

Ad alto livello il nostro approccio funziona come segue: 
1. ==**Prende come input il repository della libreria di CocoaPods, uno dei principali gestori di dipendenze per progetti Swift e Objective-C, utilizzato da oltre 3 milioni di app iOS.**==
2. ==**Genera una fingerprint della versione della libreria per ciascuna versione della libreria nel repository CocoaPods.** ==

Una fingerprint della versione della libreria è un insieme distintivo di funzionalità che catturano le proprietà univoche di una versione della libreria. 

<mark style="background: #FF5582A6;">Nel nostro approccio, la fingerprint di una versione della libreria comprende un insieme di fingerprints di classe, ciascuna delle quali cattura caratteristiche sintattiche di una classe che fa parte della versione della libreria.
</mark>

==**Data un'app:**
1. ==**LibKit decodifica il codice binario dell'app**==
2. ==**analizza i suoi binari per ottenere le caratteristiche della classe**==
3. ==**produce un insieme di fingerprints di classe, una per ciascuna classe nell'app.** ==
4. ==**Quindi, cerca le fingerprints della classe dell'app corrispondenti nelle fingerprints della classe della libreria nel database.**==


LibKit restituisce l'elenco dei nomi e delle versioni delle librerie identificate nell'app. 

<mark style="background: #BBFABBA6;">Progettiamo le nostre fingerprints per lavorare sulle informazioni sulla classe disponibili nel codice nativo di iOS.</mark>

Le nostre fingerprints sfruttano un hash di somiglianza per abbinare fingerprints di classe simili, ma non identiche. 

L’algoritmo di corrispondenza mira alla massima copertura possibile delle classi dell’app e consente una copertura parziale della libreria, ovvero identifica le Third-Party Libraries anche quando, a causa dell’eliminazione del codice morto, solo una parte del codice Third-Party Libraries entra nel codice nativo dell’app. 


 
 Applichiamo LibKit su 1.500 app dell'iTunes Store per le quali non disponiamo di dati concreti. Questo esperimento simula il modo in cui LibKit potrebbe supportare il lavoro di un'analisi della sicurezza che deve identificare quali Third-Party Libraries include un insieme di applicazioni, per identificare le app che includono librerie dannose note o versioni di librerie note vulnerabili. 
 
 LibKit rileva 47.015 versioni di librerie, con una media di 25,5 librerie per app. Riportiamo le prime 10 librerie identificate e mostriamo che le app più popolari contengono versioni di librerie precedenti. 



# Background

## iOS development
Il linguaggio ufficiale originale per lo sviluppo di app iOS era ObjectiveC, un superset del linguaggio C che aggiunge un livello e un runtime orientati agli oggetti. 

Nel 2014, Apple ha rilasciato Swift, un nuovo linguaggio di programmazione per sostituire Objective-C. Swift è stato progettato per mantenere la compatibilità con Objective-C. 

<mark style="background: #BBFABBA6;">Per questo motivo, anche un'app Swift pura contiene parti del runtime Objective-C e ogni classe Swift è anche una classe Objective-C. 
</mark>

### Building. 
Gli sviluppatori di app iOS in genere utilizzano l'IDE Xcode ufficiale come ambiente di sviluppo. Xcode gestisce tutti i passaggi per la creazione di un'app, inclusi compilazione, collegamento e passaggi post-compilazione come la firma e il confezionamento. 

Tutto il codice eseguibile prodotto nel processo di compilazione si trova nei file eseguibili Mach-O. Questi includono il codice dell'app, le librerie dinamiche, le librerie collegate staticamente e le librerie vendute, ovvero librerie open source il cui codice sorgente viene copiato direttamente nella base sorgente dell'app. 

==**Le librerie dinamiche sono distribuite come Framework che comprendono il codice della libreria in un eseguibile Mach-O e tutte le risorse richieste dalla libreria.** ==

<mark style="background: #BBFABBA6;">I framework in genere contengono il proprio nome nel percorso dell'eseguibile, rendendone più semplice l'identificazione. 

Tuttavia, i file Framework non possono essere nominati con il nome della libreria conosciuta e la versione non viene divulgata. </mark>

Il codice binario delle librerie collegate staticamente e delle librerie vendute verrà invece incluso nell'eseguibile principale dell'app. 

Al giorno d'oggi, le app iOS in genere contengono più eseguibili Mach-O: un eseguibile per l'app principale (incluso il codice dell'app, le librerie vendute e le librerie collegate staticamente) e un eseguibile per ciascun Framework che non sia una libreria di sistema preinstallata in iOS. 

### Packaging. 
Le app iOS sono distribuite come contenitori IPA, ovvero file ZIP con una struttura specifica. 
La struttura è:
![[Pasted image 20240320161609.png]]

- **==Info.plist==** contiene tutte le informazioni sulla configurazione dell'app e i permessi di quest'ultima
- **==Frameworks==**, contiene tutti i framework e quindi le librerie in file eseguibile Mach-O
- ==**MyApp**== è il binario dell'applicazione
- further resources contiene file per i linguaggi, icone, oggetti GUI, video ecc


### Distribuzione. 
Per pubblicare un'app tramite iTunes Store ufficiale, uno sviluppatore deve creare un account sviluppatore. 
Le app nell'iTunes Store sono firmate da Apple. 

La protezione FairPlay DRM utilizzata da Apple crittograferà il codice eseguibile utilizzando una chiave specifica dello sviluppatore prima di pubblicarlo nello store, lasciando gli altri file (ad esempio, risorse) non crittografati.


## CocoaPods 
==**Gli sviluppatori iOS possono utilizzare un gestore di pacchetti (PM) per gestire le dipendenze della propria app, ad esempio scaricare librerie di terze parti e aggiungerle al progetto dell'app.** ==

Esistono tre PM iOS: 
- CocoaPods, 
- Carthage 
- Swift Package Manager (Swift PM). 

Tutti e tre supportano le librerie distribuite come codice sorgente o precompilate. Una differenza fondamentale tra loro è che CocoaPods utilizza un repository centrale per archiviare le specifiche (chiamate Podspec) delle versioni della libreria disponibili. Questo repository centrale consente agli sviluppatori di librerie di rendere visibili le proprie Third-Party Libraries agli sviluppatori di app, che possono facilmente trovare le Third-Party Libraries da utilizzare. 

Quando uno sviluppatore desidera pubblicare in CocoaPods una nuova libreria o una nuova versione di una libreria già disponibile, invia un file Podspec al repository pubblico. CocoaPods presuppone il controllo delle versioni semantico per le versioni della libreria (MAJOR.MINOR.PATCH). 

![[Pasted image 20240328113551.png]]
- contiene informazioni generali sulla libreria come il nome, la versione e la fonte da cui è possibile scaricare la libreria (ad esempio, l'URL di un repository o di un archivio). 
- Nell'esempio, la libreria ha due specifiche secondarie (Core, AdMob), ma per impostazione predefinita verrà installata solo la specifica secondaria Core predefinita. 



Per includere le librerie in un'app, ==**lo sviluppatore genera un Podfile che indica le librerie da cui dipende l'app.**== 


==**CocoaPods utilizza il Podfile per scaricare automaticamente le librerie specificate (e le relative dipendenze) e per incorporarle nel progetto Xcode dell'app in modo che vengano compilate (se distribuite come codice sorgente) e collegate durante la creazione dell'app.** ==
![[Pasted image 20240328113730.png]]

<mark style="background: #BBFABBA6;">Tramite use_frameworks!  le librerie sono costruite (se possibile) come Framework separati. </mark>

Ciò fa sì che l'app contenga quattro file binari Mach-O, uno con il codice dell'app di test e uno per ciascuna libreria. 

Per Objection, poiché lo sviluppatore non ha specificato alcuna versione, CocoaPods installerà la versione più recente. Per SSZipArchive, lo sviluppatore ha utilizzato l'operatore ottimistico ∼> 2.2, che equivale a range (2.2.0, 2.3.0). 

==**L'installazione di CocoaPods produce un file Podfile.lock che risolve le dipendenze in versioni concrete della libreria.** ==
- Nell'esempio, il file Podfile.lock specificherà che Firebase 4.7.0, Objection 1.6.1 e SSZipArchive 2.2.3 sono stati inclusi nel progetto dell'app. 

<mark style="background: #BBFABBA6;">Sfortunatamente, il file Podfile.lock non fa parte dell'app creata. Pertanto, è disponibile solo quando si crea l'app dal codice sorgente e non può essere utilizzato per il rilevamento Third-Party Libraries nel nostro scenario in cui l'input è il codice binario dell'app.</mark>



# Approach
L'idea chiave alla base di qualsiasi tecnica di identificazione basata sulle fingerprints risiede nel rappresentare un artefatto analizzato in una fingerprint che cattura le caratteristiche uniche presenti in tale artefatto. 

Tale fingerprint può quindi essere utilizzata per un'identificazione affidabile basata su queste caratteristiche distintive in altri artefatti. 


LibKit è una tecnica basata sulle fingerprints per identificare i Third-Party Libraries nelle app iOS e pertanto comprende due fasi: 
- generazione della fingerprint della libreria
- rilevamento della libreria. 


![[Pasted image 20240328114204.png]]



## COME VENGONO GENERATE LE FINGERPRINT PER IL DATASET

1. Nello specifico, ==**la generazione della fingerprint della libreria recupera una libreria ottenendo il suo Podspec CocoaPods e la crea con tutte le sue dipendenze**==. 
	- Prendendo come esempio la libreria Firebase nella Figura 1, LibKit recupererebbe il codice sorgente Firebase dalla posizione specificata, quindi recupererebbe tutte le dipendenze (ad esempio Firebase/Core, FirebaseAnalytics, FirebaseCore, Google-Mobile-Ads-SDK) in modo ricorsivo da CocoaPods. 

2. LibKit ==**nell'esempio usa un'app prototipo che include esclusivamente la libreria Firebase come dipendenza per produrre i file binari MachO eseguibili della libreria**==. 
	- Utilizziamo un'applicazione modello perché CocoaPods è pensato per essere integrato con un progetto applicativo e questo processo può produrre i file binari da analizzare indipendentemente dal fatto che la libreria, o una qualsiasi delle sue dipendenze, sia distribuita come codice sorgente o precompilata come libreria dinamica o statica. 

3. ==**Ogni file binario MachO viene quindi analizzato dal modulo Feature Extraction, che utilizza un'analisi statica automatizzata per estrarre le funzionalità del codice sintattico**==. 

Tali funzionalità ==**vengono poi utilizzate per produrre una fingerprint della libreria da archiviare nel database**==. 

Durante l'estrazione delle funzionalità, LibKit utilizza una lista nera per scartare le funzionalità che potrebbero influire negativamente sulla qualità dell'fingerprint.

4. ==**LibKit ricorre a simhash per calcolare le fingerprints delle caratteristiche estratte.**== 
	- Il vantaggio di Simhash è che utilizza un metodo probabilistico per generare fingerprints simili per oggetti simili.

La fase di generazione dell'fingerprint della libreria deve essere eseguita solo una volta per ciascuna versione della libreria contenuta nel repository CocoaPods. 


## COME VENGONO RILEVATE LE LIBRERIE DALLE APP
Nella fase di rilevamento della libreria (parte inferiore della Figura 3), <mark style="background: #BBFABBA6;">LibKit analizza un'app iOS closed source con l'obiettivo di identificare qualsiasi impronta Third-Party Libraries nota nei file binari dell'app</mark>. 

1. A tal fine, deve prima ==**decrittografare il codice eseguibile che viene crittografato a causa delle politiche di Apple**==. 

2. Quindi ==**analizza ciascun file binario con la stessa tecnica usata per estrarre funzionalità e creare fingerprints per le librerie nella fase di generazione delle fingerprints**==. 

3. LibKit ==**ricerca i candidati Third-Party Libraries la cui fingerprint corrisponda alle caratteristiche trovate nell'app**== (le più simili).
	- Il codice eseguibile di una Third-Party Libraries in un'app potrebbe essere stato prodotto a seguito di un diverso processo di compilazione, che potrebbe includere l'eliminazione del dead code (non usato) e altre ottimizzazioni. Ciò porta ad avere impronte diverse anche per la stessa esatta versione della libreria. 

Grazie a simhash, tuttavia, le fingerprints della stessa versione della libreria dovrebbero essere almeno simili.


## Library Fingerprint Generation
La parte superiore della Figura 3 descrive l'architettura della generazione delle fingerprints della libreria. 

<mark style="background: #BBFABBA6;">Prende come input il repository della libreria CocoaPods e produce un database di fingerprints della versione della libreria. </mark>

Dato il file Podspec di una versione specifica di una libreria in CocoaPods, LibKit segue tre passaggi per generare l'fingerprint della versione della libreria: 
- creazione, 
- estrazione delle funzionalità 
- generazione della fingerprint. 

### Building.
1. ==**Data una versione della libreria di destinazione in CocoaPods, LibKit crea un'app modello che la include.**  ==

L'uso di un'app modello è necessario perché CocoaPods non crea librerie autonome, ma piuttosto le include nel progetto di un'app. 

L'utilizzo di un'app modello ci consente inoltre di produrre i file binari della libreria da analizzare indipendentemente dal fatto che la libreria, o una qualsiasi delle sue dipendenze, sia distribuita come codice sorgente o precompilata come libreria dinamica o statica. 

<mark style="background: #BBFABBA6;">Per creare l'app modello, LibKit crea un nuovo progetto Xcode e produce un Podfile per l'app modello che richiede la versione della libreria di destinazione. </mark>

Esempio:
![[Pasted image 20240328115547.png]]
- Il Podfile imposta iOS 9 come piattaforma di destinazione. 
- Il use_frameworks! dice a CocoaPods di provare a creare le dipendenze dell'app come Framework. 
- Tuttavia, alcune librerie come Crashlytics sono distribuite come archivi .a precompilati, costringendo Xcode a collegarle staticamente al binario principale dell'app. 

2.==**LibKit utilizza quindi CocoaPods per installare la versione della libreria nel Podfile nel progetto dell'app.** ==

CocoaPods include la versione della libreria e tutte le sue dipendenze nello script di build del progetto dell'app modello. 

L'installazione di CocoaPods produce un file Podfile.lock che risolve le dipendenze nel Podspec della versione della libreria di destinazione in versioni concrete della libreria da includere nel progetto dell'app. 

3. **==Successivamente, LibKit sfrutta Xcode per creare l'app modello.==** 
	- <mark style="background: #BBFABBA6;">Poiché l'app modello include solo la libreria, ma non la utilizza realmente, LibKit utilizza la modalità debug di Xcode per crearla, disabilitando l'eliminazione del dead code. </mark>

4. ==**Come ultimo passaggio, i file binari compilati e il file Podfile.lock vengono elaborati per generare un registro di compilazione. Ovvero mappa ciascun eseguibile alle versioni della libreria a cui corrisponde.** ==

Include anche un albero delle dipendenze con le dipendenze della versione della libreria dichiarata nei Podspec di tutte le librerie installate. 

### Feature extraction
Il processo di estrazione delle funzionalità prende come input i file binari compilati e il log di compilazione. 

==**Per ogni binario Mach-O, utilizza il parser dsdump per estrarre l'elenco delle classi in esso contenute.** ==

Durante questo processo filtra le classi scheletriche che appartengono all'app modello utilizzando una lista nera di classi. 

**==Per ogni classe rimanente vengono estratte 4 proprietà:==**
- **==nome della classe,==** 
- **==linguaggio della classe (Swift o Objective-C),==** 
- ==**elenco delle variabili di istanza (solo per classi Objective-C)** ==
- ==**elenco dei metodi della classe.**== 



**==Per ogni metodo di una classe vengono estratte 3 proprietà:==** 
- ==**nome del metodo,** ==
- ==**tipo di metodo**== (metodo di classe o di istanza)
- **==la sua interfaccia==** (per i metodi Objective-C, per i metodi Swift l'interfaccia è codificata nel nome). 
	- L'interfaccia di un metodo è una stringa alterata che definisce il tipo restituito e il tipo dei suoi parametri. 



**==Per ogni variabile di istanza Objective-C vengono estratte 2 proprietà:==** 
- **==nome==**
- **==tipo==**

Esempio:
![[Pasted image 20240328120532.png]]


### Class fingerprints 

<mark style="background: #BBFABBA6;">Una fingerprint di classe è un valore Simhash a 64 bit. </mark>

Simhash è un hash di somiglianza utilizzato in molte tecniche di rilevamento basate sulle fingerprints grazie alla sua capacità di generare fingerprints simili per oggetti simili.

Nello specifico, prende come input un insieme di hash e produce un valore hash di dimensione fissa con la proprietà che input simili producono valori hash simili.

==**Nel nostro caso un hash copre la concatenazione del nome della classe, del linguaggio della classe, del numero di variabili di istanza e del numero di metodi.** ==

Viene prodotto un hash aggiuntivo per ciascun metodo Objective-C, per ciascun metodo Swift e per ciascuna variabile di istanza ObjectiveC. 

**==L'hash di ciascun metodo e variabile di istanza include il nome della classe per evitare corrispondenze spurie con classi non correlate che potrebbero avere metodi e variabili con nomi simili.==** 

Nella Figura 4, il fingerprint della classe NSDuck sarebbe il Simhash a 64 bit di tre hash: uno per il nome della classe (NSDuck), la lingua (Objective-C), il numero di metodi e il numero di variabili di istanza ; 

un altro per il metodo quackWithVolume; 
e un'ultima per la variabile dell'istanza flying. 


<mark style="background: #BBFABBA6;">Il vantaggio di questa progettazione è che può identificare la stessa classe indipendentemente dalle modifiche al codice dovute alle diverse configurazioni del compilatore. </mark>

Questo è importante perché LibKit genera le fingerprints della classe utilizzando le impostazioni del compilatore che potrebbero non corrispondere a quelle utilizzate dagli sviluppatori delle app che utilizzano la libreria. 

<mark style="background: #FF5582A6;">Uno svantaggio è che due versioni della stessa classe con metadati identici ma contenenti differenze di codice, ad esempio una patch che aggiunge solo un controllo del puntatore NULL in un metodo, avranno la stessa fingerprint della classe e quindi non potranno essere differenziate. </mark>

<mark style="background: #FF5582A6;">Un altro svantaggio è che le nostre fingerprints non sono resistenti alla ridenominazione dei simboli. </mark>

Tuttavia hanno identificato solo lo 0,06% di un milione di app iOS raccolte dall'iTunes Store ufficiale come offuscate utilizzando la ridenominazione dei simboli. 

L'uso di Simhash ci consente di identificare la stessa classe nonostante piccole modifiche nei suoi metadati.

Cambiamenti importanti come un pesante refactoring, la ridenominazione delle classi o l'aggiunta di un gran numero di metodi produrrebbero fingerprints della classe significativamente diverse. 

Se la libreria presa in considerazione non è nel database di Libkit, allora Libkit mira a identificare la versione più vicina della libreria presente nel database.


### Library version fingerprint

**==Una fingerprint della versione della libreria comprende un insieme di fingerprints di classe, una per ciascuna classe della libreria.==**** 

<mark style="background: #BBFABBA6;">Più specificamente, la fingerprint della versione della libreria comprende l'insieme di fingerprints di classe per tutte le classi in tutti i file binari prodotti durante la creazione dell'app modello (relativi a quella libreria)</mark>. 

È possibile che più versioni della libreria abbiano la stessa fingerprint, il che le rende indistinguibili l'una dall'altra. 

Ciò accade per due ragioni principali: 
1. versioni vicine della stessa libreria che hanno metadati di classe identici e differiscono solo nel codice; 
2. librerie che sono fork o cloni esatti l'una dell'altra. 

**==Una volta popolato il database, LibKit identifica le versioni della libreria con la stessa fingerprint e le inserisce in una classe di equivalenza.==** 

Per ogni classe di equivalenza identifica un leader. 

Il ruolo del leader è quello di essere l'output della versione della libreria nei risultati. 

La selezione leader conta quante volte una versione della libreria dipende da un'altra nella classe di equivalenza. La libreria con il maggior numero di dipendenti viene scelta come leader.



## Library detection
La parte inferiore della Figura 3 descrive l'architettura di rilevamento della libreria. 

==**Prende come input l'IPA di un'app e restituisce l'elenco delle versioni della libreria utilizzate dall'app.** ==

1. **==Il primo passaggio è la decrittografia dell'app.==** 

2. Dopo aver decrittografato l'app, ==**il passaggio successivo è,** ==**==per ciascun binario decrittografato, analizzarlo, estrarre le funzionalità della classe e del metodo e generare le fingerprints della classe==**.

3. Le fingerprints della classe vengono inserite nella ricerca dei candidati, il cui compito è ==**trovare per ciascuna fingerprint della classe nell'app, un insieme di fingerprints di classe simili nel database, che chiamiamo candidati.** ==

La selezione dei candidati identifica quindi le migliori librerie candidate. 

Infine, l'inclusione delle dipendenze aggiunge le dipendenze tra le versioni della libreria rilevate, seleziona un leader per ciascuna classe di equivalenza e produce l'elenco finale delle versioni della libreria identificate.


### Candidate search
Ogni file binario nell'app `A` è composto da un set di fingerprints di classe. 

Per ogni fingerprint di classe `c` nel file binario dell'app, la funzione di ricerca dei candidati `S` restituisce come candidato `v1` qualsiasi libreria nel database contenente almeno una fingerprint di classe `f` più simile di una soglia di somiglianza `Ts` alla fingerprint di classe `c` nell'app. 

`S(c) = {v1 : (v1 , f ) ∈ D, 𝑠𝑖𝑚𝑖𝑙(c, f ) ≥ Ts }` 

La funzione `simil` è la somiglianza normalizzata tra due fingerprints di classe `s`, uno proveniente dall'app (`c`) e l'altro da una libreria nel database (`f`). 

Poiché `c` e `f` sono valori Simhash a 64 bit, LibKit calcola la loro somiglianza in base alla distanza di Hamming delle loro stringhe di bit, quindi `simil(c, f) = 64−hamming(c,f ) / 64 `. 

La soglia di somiglianza `Ts` cattura la somiglianza minima affinché le fingerprints di due classi possano essere considerate una corrispondenza.

==**Quindi per ogni fingerprint di classe dell'applicazione che stiamo analizzando si prende un set di librerie dal database che hanno fingerprints di classe simili a questa (sopra una soglia di similitudine Ts)**==

### Candidate filtering

**==La ricerca dei candidati seleziona qualsiasi versione della libreria che abbia almeno una classe in comune con l'app.==** 

Pertanto, potrebbe restituire centinaia o addirittura migliaia di candidati. 

Non ha molto senso considerare una libreria se il numero di classi trovate nell'app rappresenta un rapporto molto piccolo rispetto all'intera libreria. 

Per ogni `v1` identificata durante il passaggio precedente, LibKit mantiene solo quelle che soddisfano `(L(v1 )−M(v1) / L(v1)) ≥ Tm`
- `L(v1)` è il numero di fingerprints della classe che la versione di quest a libreria ha 
- `M(v1)` il numero di classi in `v1` che non è stato possibile trovare in A a partire dal nome. 

<mark style="background: #FF5582A6;">Quindi vengono rimossi i candidati con un rapporto ( numero di classi presenti nel candidato / numero di classi del candidato che non si trovano nell'applicazione A ) minore di Tm.</mark>

La funzione di filtraggio è quindi:
`F = {v1 : v1 ∈ S(C), C ∈ A, L(v1) − M(v1) / L(V1) ≥ 𝑇𝑚 }`


### Framework filtering. 
Le librerie compilate come Framework hanno il nome nel percorso del bundle dell'applicazione quindi si può usare un metodo ottimizzato. 

Per tutte le versioni della libreria distribuite come sorgente, il database mantiene una mappatura tra la versione della libreria e i binari del Framework prodotti durante la sua elaborazione nella generazione della fingerprints della libreria. 

==**Quando il binario in analisi è un Framework, LibKit quindi estrae direttamente il suo nome dal percorso e lo confronta con la mappatura.**==

LibKit mantiene in `F` solo le versioni della libreria che corrispondono. 

Pertanto, se un Framework è stato rinominato, nessuna libreria candidata lo corrisponderà e il rilevamento procederà senza questa ottimizzazione.



### Candidate selection

Al termine della fase di rilevamento della libreria, ciascuna fingerprints di classe trovata nell'app dovrebbe essere assegnata al massimo a una versione candidata della libreria. 

Tenendo presente che i Framework contengono solo una libreria per binario, mentre il binario principale può contenere molte librerie collegate staticamente, oltre al codice dell'app, LibKit ricorre a due diversi algoritmi per selezionare la migliore corrispondenza. 


==**In entrambi i casi, LibKit classifica innanzitutto l'elenco delle versioni**== `F` ==**della libreria filtrata in base a un punteggio**== `p(v1)`. 

Questo punteggio viene calcolato come moltiplicazione di tre valori:
1. **==coverage score==** -> il numero di classi che il candidato matcha moltiplicato per il rapporto tra:  (le classi che il candidato matcha nell'app A e il numero totale di classi nella fingerprint della versione della libreria scelta come candidato.) 
2. ==**average normalized similarity**== delle classi matchate dal candidato. 
3. ==**il numero di diverse versioni della libreria contenute dal candidato in F, che chiamiamo popolarità della libreria** ==
	1. L'intuizione alla base di questo punteggio di popolarità è che, <mark style="background: #BBFABBA6;">poiché le versioni simili di una libreria non differiscono molto, avranno fingerprints di versione della libreria simili. </mark>
	2. Pertanto, è normale che molte versioni delle librerie che hanno migliore corrispondenza appaiano in F. 
	
`p(1) = ((|C(v1)|^2 )/ L(𝑣 𝑙 ) ) × 𝑠𝑖𝑚𝑖𝑙(v1) × 𝑝𝑜𝑝(v1) `

Nel caso del binario principale, LibKit risolve il problema della copertura dell'insieme con l'algoritmo:
![[Pasted image 20240328143007.png]]
1. `cov` alla linea 2 è inizializzata con tutte le classi nell'app che devono essere coperte da un candidato in F. 
2. Partendo dal candidato con il `p(v1)` più alto, LibKit 
	1. rimuove da `cov` le classi che copre `v1`, 
	2. aggiunge `v1` alle librerie selezionate  `selection`, 
	3. e infine rimuove da F le librerie che non coprono alcuna classe rimanente in `cov`. 
	4. Questo processo continua finché l'elenco in F è vuoto. 

<mark style="background: #FF5582A6;">Quindi in pratica seleziona tutte le librerie che hanno almeno un metodo in comune con l'app.</mark>

==**Quando il binario è un Framework, invece, LibKit seleziona prima i candidati con il miglior punteggio di copertura.** ==
- Da questo sottoinsieme seleziona il candidato con la migliore somiglianza media normalizzata e lo aggiunge all'elenco delle librerie selezionate.


### Dependency inclusion 
Alla fine per ogni libreria selezionata, LibKit riporta tutte le loro dipendenze nella lista finale delle librerie trovate nell'app.



# Datasets (Da qui meno interessante solo info)
## Library database.
Il dataset contiene 86.597 versioni di librerie appartenenti a 14.043 librerie. 

Il numero totale delle classi è 8.714.001, di cui il 63,7% sono classi Objective-C e il 36,3% classi Swift. 
Queste 8.714.001 classi contengono 106.993.407 metodi, una media di 10,2 metodi per classe.

![[Pasted image 20240328144037.png]]



## App collection
La nostra pipeline di raccolta di app replica quella proposta da CriOS. 
1. ==Scarica le app utilizzando il client Windows iTunes, ==
2. ==le installa su un dispositivo iPhone== 
3. ==scarica la memoria utilizzando Frida una volta completata la decrittazione. ==

Utilizziamo la pipeline di raccolta app per scaricare e decrittografare 1.500 app.

## Ground truth
Abbiamo creato due set di dati ground Truth: 
- uno con 43 app per le quali abbiamo identificato le versioni delle librerie che utilizzano (GT43) 
- un altro con 95 app per le quali abbiamo identificato solo i nomi delle librerie che utilizzano, ma non la loro versione (GT95). 


### GT43. 
Per creare i nostri set di dati concreti, abbiamo iniziato cercando su GitHub app iOS open source, che hanno prodotto 140 app. 

==Per ottenere i dati concreti desiderati per un'app, dobbiamo ottenere il relativo file Podfile.lock, che indica le versioni della libreria incluse da CocoaPods durante la creazione dell'app. ==
- Ad esempio, se il podfile dell'app definisce un intervallo di versioni di librerie compatibili, il file Podfile.lock indicherà la versione specifica scelta da CocoaPods in tale intervallo (ovvero, quella che LibKit dovrebbe rilevare). 

==Sfortunatamente, Podfile.lock è disponibile solo quando si crea l'app dal sorgente o se gli sviluppatori lo hanno inserito nel repository di origine dell'app dopo aver creato l'app. ==

Siamo riusciti a compilare solo 10 delle 140 app. Inoltre, altre 33 app (disgiunte dalle 10 che siamo riusciti a compilare) avevano il file Podfile.lock disponibile nel loro repository. 

Per queste 43 app conosciamo le versioni della libreria CocoaPods incluse nell'app. Tuttavia, l'app potrebbe includere anche alcune librerie vendute, ovvero il sorgente della libreria è stato copiato nel sorgente dell'app e quindi viene compilato come parte dell'app. Per identificarle, abbiamo esaminato manualmente il codice sorgente dell'app nel repository. 

Complessivamente le 43 app contengono 511 versioni di libreria appartenenti a 347 librerie, per un totale di 26.160 classi. 

Utilizziamo GT43 per misurare la precisione con cui LibKit rileva le versioni della libreria. 

### GT95. 
Tra le 140 app, c'erano 95 app (le 33 con un Podfile.lock nel loro repository e altre 62 non in GT43) disponibili su iTunes Store. 

Possiamo analizzare il podfile nel repository di origine per identificare i nomi dei Third-Party Libraries utilizzati dall'app. Poiché il podfile è necessario per creare l'app open source, gli sviluppatori generalmente lo aggiungono al repository, a differenza del Podfile.lock che è opzionale. 

Tieni presente che il file pod è disponibile solo per le app open source che utilizzano CocoaPods, ma non per le app proprietarie nell'iTunes Store. 


Di queste 95 app conosciamo solo i nomi delle librerie utilizzate dalle app, ma non la versione della libreria. 

Usiamo GT95 per misurare la precisione con cui LibKit rileva le librerie. 

Le 95 app contengono 1.066 librerie con 37.283 classi. 


# Evaluation
## RQ1: Library Identification. What is the accuracy of LibKit for detecting libraries? 

Per valutare l'accuratezza di LibKit nel rilevamento delle librerie, eseguiamo LibKit su GT95 più volte utilizzando parametri diversi. 

Per ogni esecuzione confrontiamo le librerie identificate (ignorando la versione identificata) con l'elenco delle librerie nel GT. 

In particolare, calcoliamo 
- il numero di librerie rilevate nel GT (Veri Positivi),
- il numero di librerie rilevate non nel GT (Falsi Positivi) e
- il numero di librerie non rilevate nel GT (Falsi Negativi). 

Abbiamo suddiviso i falsi negativi in quelli dovuti alle librerie presenti e non presenti nel nostro database delle librerie, in modo da poter separare la copertura del database dall'accuratezza del rilevamento. 


Il rilevamento della libreria ha due parametri: 
- la soglia di somiglianza (𝑇𝑠 ) e 
- la soglia di copertura (𝑇𝑚). 

I migliori risultati si ottengono utilizzando una soglia di somiglianza 𝑇𝑠 = 0,8. 


Selezioniamo 𝑇𝑠 = 0,8 e 𝑇𝑚 = 0,35 come valori dei parametri per il resto della valutazione. 

I risultati del rilevamento della libreria sono: precision di 0,911, recall di 0,839 e F1 score di 0,874. 

Successivamente stimiamo l'impatto della copertura del database delle librerie. 

In GT95, 248 (22%) delle librerie non sono in CocoaPods. 

Queste librerie sono distribuite attraverso altri mezzi. Per coprire tali librerie potremmo incorporare altre fonti di libreria come GitHub. 

Altre 164 librerie (15%) si trovano in CocoaPods, ma LibKit non è riuscito a generare una fingerprint per esse. 

La copertura di queste librerie richiederebbe miglioramenti alla nostra pipeline di creazione automatizzata, ad esempio il supporto dei framework iOS meno recenti. 

### False negative

Più della metà di questi FN sono limitazioni della nostra GT che non cattura le dipendenze dovute alle librerie fornite. 

### Falsi positivi. 
Un motivo comune per i falsi positivi sono in realtà i falsi negativi, ciò che chiamiamo coppie FN-FP. Accade spesso che quando LibKit manca una libreria, introduce un FP selezionando una libreria diversa che contiene la libreria mancante. 

Abbiamo anche riscontrato alcuni casi causati dall'utilizzo di altri linguaggi nello sviluppo di app. 
- Ad esempio, ci sono due app React Native che contengono il runtime React Native, che non è nel nostro database. Invece, LibKit identifica una particolare libreria che contiene il runtime React Native.

## RQ2: Comparison with State of the Art. How does LibKit compare to the state of the art?

Confrontiamo LibKit con CRiOS, che consideriamo lo stato dell'arte nel rilevamento Third-Party Libraries di iOS. 
Dal rilascio di CRiOS, si sono verificati molti cambiamenti in iOS, che causano il fallimento di CRiOS su 72 app GT95 (45 app che contengono Swift e 27 app che utilizzano versioni Objective-C più recenti). Per risolvere questo problema, abbiamo modificato CRiOS per sostituire il suo parser Mach-O di dump di classe con il parser dsdump utilizzato da LibKit. 


Con questa correzione potremmo eseguire CRiOS su tutte le app in GT95. La tabella 2 riporta i risultati del confronto su tutte le app in GT95, e anche sulle 50 app basate interamente su Objective-C poiché CRiOS non è stato progettato per supportare Swift. 
![[Pasted image 20240328151211.png]]


I risultati mostrano che LibKit batte CRiOS in entrambi i set di dati e in tutti i parametri. 

Inoltre, LibKit è in grado di etichettare automaticamente le librerie (e le versioni) nell'app.



## RQ3: Library Version Identification. What is the accuracy of LibKit for detecting library versions?

Per valutare il rilevamento delle versioni della libreria eseguiamo LibKit sul set di dati GT43 utilizzando i valori dei parametri determinati in RQ1. 

LibKit rileva 421 versioni della libreria: 370 che corrispondono esattamente a quella della GT (TPs) e 51 che differiscono dalla GT (FPs). 

Mancano 84 versioni della libreria (FN). 

Pertanto, le metriche di precisione per il rilevamento della versione della libreria sono: precision di 0,879, recall di 0,815 e F1 score di 0,846.



## RQ4: Library Detection on Store Apps. What libraries does LibKit detect on apps from the iTunes Store?

Eseguiamo il rilevamento della libreria sulle 1.500 app raccolte da iTunes Store. 

LibKit rileva 47.015 versioni di libreria in tutte le 1.500 app. 

Di questi, 31.041 (66%) sono collegati staticamente (o vendute), mentre il restante 34% sono Framework. 

Ciò evidenzia l'utilità di LibKit per l'analisi delle app iOS. 

Esaminando semplicemente il nome dei file Framework negli App bundle mancherebbero due terzi delle librerie. 

Inoltre, per i Framework, LibKit è in grado di fornire la versione, che non fa parte del nome del Framework. 

Tra i 47.015 rilevamenti, ci sono 11.704 versioni di libreria univoche (ovvero il 13,5% di quelle nel nostro DB) e 1.592 nomi di libreria univoci (l'11,3% di quelle nel nostro DB). 

In 131 app, LibKit non ha rilevato alcuna libreria e il maggior numero di librerie rilevate in un'app è 106. La tabella 3 mostra le prime 10 librerie più diffuse identificate sulle 1.500 app. 
![[Pasted image 20240328152822.png]]

La libreria più popolare è la libreria pubblicitaria AdMob di Google, presente in 494 app.
La seconda è la sottospec AppDelegateSwizzler della libreria GoogleUtilities presente in 493 app. 
Il terzo è FBSDKCoreKit, che consente di integrare Facebook in un'app iOS e si trova in 445 app. 

La colonna più a destra nella Tabella 3 mostra che le app contengono un gran numero di versioni per la stessa libreria. Ad eccezione di GoogleAnalytics, il numero di versioni uniche per ciascuna libreria varia da 20 in Alamofire fino a 78 per AdMob. Dato che le app sono state raccolte entro 1,2 giorni, ciò indica che molte app eseguivano versioni precedenti delle librerie. Un esempio è l'app Documenti (Office Docs) di Savy Soda, con 4,6K valutazioni nel mercato italiano. 

 I risultati evidenziano che le vecchie versioni delle librerie sono comuni tra le app più popolari nell'iTunes Store e come LibKit possa essere utilizzato per identificare le app con dipendenze così obsolete. 
 
 
### Duration. 

In media, LibKit impiega 8 minuti per rilevare le librerie in un'app. 


# Related Works
Gli approcci di rilevamento Third-Party Libraries per le app mobili possono essere suddivisi tra
- approcci basati su clustering che deducono le librerie identificando componenti di codice condivisi da più app di input 
- e quelli che creano un database di fingerprints di libreria direttamente dal codice della libreria. 


Gli approcci basati sul clustering hanno il vantaggio di poter identificare Third-Party Libraries precedentemente sconosciuti, ma non possono nominare la libreria specifica o la versione della libreria rappresentata da un cluster, a meno che non venga creata una mappatura separata (in genere manualmente).


CriOS ha proposto per primo un approccio basato sul clustering che raggruppa classi con lo stesso prefisso del nome e coesione di classe in librerie. 

Come parte del loro approccio per identificare le vulnerabilità nei servizi di rete iOS, hanno proposto una tecnica basata sul clustering che ottiene dinamicamente lo stack di chiamate di un'app quando richiama la funzione di `bind` e identifica l'associazione di wrapper Third-Party Libraries raggruppando stack di chiamate simili. 

Tuttavia, il loro approccio si applica solo alle librerie di rete. 


Diversi lavori hanno proposto tecniche di analisi iOS per altre applicazioni. PiOS identifica staticamente i metodi utilizzati da un binario Objective-C per rilevare violazioni della privacy. 

Joorabchi e Mesbah applicano il riconoscimento delle immagini per rilevare quando l'interfaccia utente di un'app iOS è cambiata durante l'esecuzione. 

Altri lavori identificano vulnerabilità nelle app iOS come quelle introdotte dall'uso improprio di API crittografiche e credenziali. 

Altri lavori studiano le dipendenze iOS in CocoaPods.



# Discussion 

## Fingerprint uniqueness

==Le versioni della libreria con la stessa fingerprints (ovvero, gli stessi metadati della classe) non possono essere differenziate e vengono aggiunte a una classe di equivalenza. ==

==Questi casi sono in gran parte dovuti a versioni consecutive della libreria, di cui circa due terzi sono dovuti a versioni consecutive a livello di patch e un altro 25% è dovuto a versioni minori consecutive. ==

Osserviamo anche che le nostre fingerprints producono collisioni più elevate nelle librerie Swift rispetto alle librerie Objective-C. 

Ciò è dovuto al codice Swift compilato che contiene meno informazioni rispetto al codice Objective-C compilato, ad esempio non contiene variabili di istanza, che sono incluse solo nelle nostre fingerprints per Objective-C. 

Per rendere le fingerprints più uniche potremmo aggiungere più caratteristiche estratte dal codice binario, ad esempio da funzioni e variabili che non appartengono a classi e dal codice dei metodi e delle funzioni. 
- però, esistono strumenti limitati per analizzare il codice nativo iOS. 
- inoltre, le popolari piattaforme di analisi binaria come Angr hanno poco supporto per i binari iOS e Mach-O

In secondo luogo, alcuni problemi in cui l'analisi del codice sarebbe molto utile, come la gestione dell'offuscamento, non sono ancora prevalenti in iOS, solo lo 0,06% delle app nell'iTunes Store sono offuscate. 

## Database coverage.

Sebbene l'analisi del codice aiuterebbe a distinguere alcune versioni di libreria vicine, i nostri risultati mostrano che il basso recall è dovuto principalmente alla copertura limitata nel database delle versioni della libreria. 

Ciò accade anche se il nostro database delle versioni della libreria comprende 86.000 versioni delle librerie, sette volte più grandi di quelle utilizzate nel rilevamento Third-Party Libraries di Android. 


# Conclusion

Abbiamo presentato LibKit, uno strumento di identificazione Third-Party Libraries ==completamente automatizzato che è il primo a identificare il nome e la versione dei Third-Party Libraries presenti nelle app iOS==. LibKit supporta app sviluppate in Swift, Objective-C o una combinazione di entrambi.

==Rileva le librerie collegate staticamente e dinamicamente.==

LibKit crea automaticamente fingerprints per le 6K versioni delle librerie disponibili tramite CocoaPods e le abbina agli eseguibili dell'app decrittografati. 

Valutiamo LibKit per i problemi di identificazione della libreria e di identificazione della versione della libreria, dimostrando che la sua precisione si confronta positivamente con gli strumenti Android più performanti. 

LibKit supera significativamente CRiOS, il precedente strumento all'avanguardia per le app iOS, per quanto riguarda il problema del rilevamento dei limiti delle classi Third-Party Libraries nelle app iOS. 

Come lavoro futuro vorremmo affrontare alcune delle limitazioni di LibKit come:
- aggiungere il supporto per codice scritto in altri linguaggi (ad esempio C/C++); 
- rafforzare le fingerprints contro l'offuscamento incorporando funzionalità a livello di istruzione o di CFG; 
- aumentare la copertura del nostro database delle librerie; 
- e aumentare il tasso di successo della pipeline di compilazione (ad esempio, supportando versioni iOS precedenti).

