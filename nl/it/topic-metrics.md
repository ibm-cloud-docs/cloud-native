---

copyright:
  years: 2019
lastupdated: "2019-03-26"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Metriche
{: #metrics}

Le metriche sono semplici misurazioni numeriche acquisite come coppie chiave-valore. Alcune metriche sono dei contatori incrementali, altre eseguono delle aggregazioni, come la somma di tutti i valori raccolti nell'ultimo minuto o il tempo medio trascorso nell'ultimo minuto. Alcune metriche sono dei semplici misuratori che restituiscono qualsiasi valore sia quello ultimo osservato. L'acquisizione e l'elaborazione delle metriche può aiutarti a identificare e rispondere a potenziali problemi prima che si accrescano e causino problemi più seri.
{:shortdesc}

Ci sono tre fattori generali quando si parla di metriche in un sistema distribuito: produttori, aggregatori e processori. Ci sono delle combinazioni abbastanza comuni di questi fattori, come ad esempio l'utilizzo di Prometheus come aggregatore con le metriche raccolte dall'elaborazione Grafana per la visualizzazione nei dashboard grafici oppure l'utilizzo di StatsD con Graphite.

![I tre fattori nelle metriche di sistema distribuite](images/metrics-systems.png "I tre fattori nelle metriche di sistema distribuite")

Il produttore è, ovviamente l'applicazione stessa. In alcuni casi, l'applicazione è direttamente coinvolta nella produzione delle metriche. In altri casi, gli agent o un'altra infrastruttura osservano passivamente oppure strumentano attivamente l'applicazione per produrre delle metriche per loro conto. Quello che succede dopo dipende dall'aggregatore.

Le metriche sono trasferite dal produttore all'aggregatore mediante meccanismi di "push" o "pull". Alcuni aggregatori, come StatsD, prevedono che l'applicazione (oppure un agent per suo conto) si connetta all'aggregatore per trasmettere i dati. Ciò richiede delle informazioni di connessione perché l'aggregatore venga distribuito a tutti i processi applicativi che devono essere misurati. Altri aggregatori, come Prometheus, si connettono periodicamente a un endpoint noto per raccogliere i dati delle metriche o eseguirne lo scraping. Ciò richiede che il produttore definisca e fornisca un endpoint di cui può essere eseguito lo scraping e che all'aggregatore venda indicato dove si trovano gli endpoint. Quando è utilizzato con Kubernetes, Prometheus può rilevare gli endpoint in base alle annotazioni del servizio.

Per finire, il processore è qualcosa che fa uso di tutti questi dati aggregati. Come detto in precedenza, i servizi come Grafana elaborano le metriche aggregate per la visualizzazione utilizzando i dashboard. Grafana supporta anche gli avvisi in base alle regole memorizzate e valutate indipendentemente dal dashboard.

## Rilevamento automatico degli endpoint Prometheus in Kubernetes
{: #prometheus-kubernetes}

Il modello basato su pull è stato creato nel tuo ecosistema. Altri aggregatori, come Sysdig, possono anch'essi eseguire lo scraping di dati delle metriche dagli endpoint Prometheus. Questo potrebbe significare che alcuni sistemi utilizzano le metriche Prometheus senza utilizzare il server Prometheus.

Negli ambienti Kubernetes, le annotazioni vengono utilizzate per il rilevamento degli endpoint Prometheus. Ad esempio, un servizio che fornisce un endpoint `/metrics` su HTTP sulla porta 8080 aggiunge le seguenti annotazioni alla definizione del servizio:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ...
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /metrics
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

Qualsiasi aggregatore compatibile con Prometheus può quindi rilevare questi endpoint utilizzando la API Kubernetes applicando filtri sull'annotazione.

## Confronto tra le metriche di applicazione di piattaforma
{: #app-platform-metrics}

La raccolta delle metriche va ponderata. Quali punti di dati devi raccogliere? Quando valuti un'applicazione nativa del cloud, puoi grosso modo dividere le metriche in due categorie, applicazione e piattaforma:

* Le metriche di applicazione si concentrano sulle entità di dominio delle applicazioni. Ad esempio, quanti accessi sono stati completati negli ultimi cinque minuti? Quanti utenti sono attualmente connessi? Quante transazioni commerciali sono state eseguite nell'ultimo secondo? La raccolta di metriche specifiche per le applicazioni richiede del codice personalizzato per raccogliere e pubblicare le informazioni. Molti linguaggi hanno delle librerie di metriche per semplificare l'aggiunta di metriche personalizzate.
* Le metriche di piattaforma, invece, sono delle entità del dominio host. Ad esempio, quanto tempo ci vuole per richiamare un servizio? Quanto tempo impiega questa query del database? Sono allineate alla modalità di transito di traffico e lavoro nel sistema e possono spesso essere misurate senza alcuna modifica all'applicazione. Alcuni framework di applicazioni forniscono anche un supporto integrato per misurare queste problematiche, come ad esempio i tempi di esecuzione delle query.

Queste categorie non sono sufficienti da sole per decidere davvero cosa hai bisogno di misurare. Ricordati che, in un sistema distribuito, sono molte le cose che producono delle metriche Ci sono alcuni metodi ben noti che provano a sintetizzare il vasto pool di metriche a quelle essenziali che devono essere controllate.

* I [quattro segnali d'oro](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/#xref_monitoring_golden-signals){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") sono le metriche essenziali identificate dal team SRE (Site Reliability Engineering) di Google per l'osservabilità a livello di servizio quando si monitora un sistema distribuito. Secondo questo team, se puoi misurare solo quattro metriche del tuo sistema orientato all'utente, concentrati su queste quattro. Le quattro metriche sono latenza, traffico, errori e saturazione.
* Il [metodo USE (Utilization Saturation and Errors](http://www.brendangregg.com/usemethod.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") è progettato come un elenco di controllo di emergenza per analizzare le prestazioni di un sistema. Il metodo USE può essere riepilogato in una frase, "Per orni risorsa, controlla utilizzo, saturazione ed errori". Per risorsa, in questo caso, si intendono risorse fisiche o logiche con limiti materiali, come le CPU o i dischi. Per ogni risorsa finita, misuri l'utilizzo, la saturazione e gli errori. 
* Il [metodo RED](https://thenewstack.io/monitoring-microservices-red-method/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") è un promemoria derivato da quattro segnali d'oro che definisce tre metriche chiave che devi misurare per ogni microservizio nella tua architettura. Questo metodo è particolarmente incentrato sulle richieste, soprattutto rispetto al metodo USE, che ben si allinea con il design e l'architettura di molte applicazioni native del cloud. Le metriche sono frequenza, errori e durata. 

Ci sono alcuni elementi comuni tra questi metodi, e tutti tracciano la frequenza degli errori, ad esempio. Ci sono però anche delle notevoli differenze. Il metodo USE si concentra sulle metriche dell'infrastruttura (utilizzo/saturazione). Gli ambienti nativi del cloud sono progettati per fare un uso migliore dell'hardware fisico (o virtuale). Queste misurazioni dell'infrastruttura appurano se il tuo sistema sta gestendo in modo appropriato il carico oppure no. Il metodo RED si concentra esclusivamente sulle metriche delle richieste (latenza/durata, traffico/frequenza), che possono indicare problemi con l'infrastruttura (problemi di rete) o con le tue applicazioni (configurazioni errate, deadlock, malfunzionamenti ripetuti dei servizi e così via), soprattutto quando combinato con le frequenze di errore osservate. I segnali d'oro comprendono entrambi, portando insieme le metriche dell'infrastruttura e delle richieste in una vista olistica.

## Definizione delle metriche
{: #defining-metrics}

Il monitoraggio delle metriche raccolte da un singolo servizio possono offriti un'idea del suo utilizzo delle risorse ma, se ci sono più istanze del servizio (a causa del ridimensionamento orizzontale o delle distribuzioni multiregione), devi distinguere tra esse per isolare i problemi nella massa di dati simili che arriva. Ciò alla fine si riduce alla denominazione. In alcuni casi, il sistema di metriche che stai usando potrebbe imporre una struttura sulle tue metriche. Prometheus, ad esempio, raccomanda qualcosa come `namespace_subsystem_name`, altri raccomandano `namespace.subsystem.targetNoun.actioned`.

Ad esempio, se volevi tenere traccia del numero di "transazioni" eseguito da un'applicazione di transazioni azionarie, potresti acquisirlo in una proprietà denominata `stock.trades`. Per distinguere tra le istanze, potesti anteporre alla proprietà l'ID istanza: `<instanceid>.stock.trades`. Ciò supporta la raccolta di singoli valori di istanza nonché dei dati aggregati utilizzando `*.stock.trades`. Cosa succede però se esegui la distribuzione a più data center e vuoi analizzare le metriche in questo modo? Puoi aggiornare il tuo nome in modo che sia `<datacenter>.<instanceid>.stock.trades` ma questo interromperebbe qualsiasi segnalazione che utilizza il carattere jolly precedente di `*.stock.trades`. Dovrebbe invece usare `*.*.stock.trades`. 

Se non si presta attenzione, utilizzare solo proprietà denominate gerarchicamente può portare a fragili pattern di caratteri jolly collegati alla struttura arbitraria della tua strategia di denominazione che non ti aiutano a osservare le informazioni di cui hai bisogno per assicurati che la tua applicazione stia funzionando bene.

I sistemi di metriche che supportano i dati dimensionali ti consentono di associare ulteriori etichette di identificazione, o tag, ai dati delle metriche. Puoi quindi filtrare, raggruppare o analizzare le metriche raccolte utilizzando queste dimensioni aggiuntive senza fare affidamento sui caratteri jolly e sulle convenzioni di denominazione. Le tipiche etichette includono il nome di endpoint o servizio, il data center, il codice risposta, l'ambiente di host (produzione/preparazione/sviluppo/test) o gli identificativi di runtime (versione Java, informazioni sul server delle applicazioni).

Utilizzando lo stesso esempio di transazioni azionarie, la metrica `stock.trades` è associata a diverse etichette `{ instanceid=..., datacenter=... }`, il che consente al valore aggregato di essere filtrato o raggruppato in base a `instanceid` o `datacenter` senza fare affidamento sui caratteri jolly. Esiste un equilibrio tra il traffico denominato (`stock.trades`) e le etichette associate (confronta ciò all'esempio gerarchico di `<datacenter>.<instanceid>.stock.trades`): ciascuna metrica deve acquisire dei dati significativi, con le etichette che consentono la disambiguazione, dove appropriato.

Inoltre, sii cauto quando definisci le etichette. In Prometheus, ad esempio, ogni combinazione univoca di coppie di etichette chiave-valore viene trattata come una serie temporale separata. Una prassi ottimale per garantire una buona modalità di funzionamento delle query e una raccolta dei dati delimitata consiste nell'utilizzare le etichette con un numero finito di valori consentiti. Ad esempio, se usi una metrica che conteggia il numero di errori, puoi utilizzare il codice di restituzione come un'etichetta, dove i valori rientrano entro una serie ragionevole (401, 404, 409, 500, ... ) ma non desideri un'etichetta per l'URL malfunzionante poiché si tratta di una serie non delimitata (qualsiasi URL di richiesta non riuscito per qualsiasi motivo, incluso l'essere non valido).

Per ulteriori informazioni sulle prassi ottimali per la denominazione di metriche ed etichette, vedi il documento relativo alla [denominazione di metriche ed etichette ](https://prometheus.io/docs/practices/naming/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

## Considerazioni aggiuntive
{: #metrics-considerations}

Quando raccogli le tue metriche, ricordati che un percorso di malfunzionamento è spesso molto diverso da un percorso di esito positivo. Ad esempio, una risposta di errore su una risorsa HTTP può richiedere molto più tempo di una risposta di esito positivo se il malfunzionamento coinvolgeva dei timeout e una raccolta della traccia di stack. Conteggia e tratta i percorsi di errore separatamente dalle richieste riuscite.

Un sistema distribuito ha delle variazioni naturali in alcune misurazioni. Degli errori occasionali sono normali poiché le richieste potrebbero essere indirizzate a processi che sono in pieno avvio o arresto. Filtra i dati non elaborati da acquisire quando questa variazione naturale inizia a superare un intervallo valido. Ad esempio, suddividi le metriche in bucket. Categorizza la durata delle richieste in categorie come 'smallest/quickest', 'medium/normal' e 'longest/largest', come osservato entro un intervallo di tempo considerato. Se le durate delle richieste vanno a finire regolarmente nel bucket "longest/largest", puoi identificare un problema. Per questo tipo di dati vengono di norma utilizzate le metriche di riepilogo o istogramma. Per ulteriori informazioni, vedi il documento relativo a [istogrammi e riepiloghi](https://prometheus.io/docs/practices/histograms/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

In qualità di sviluppatore delle applicazioni, assicurati che le tue applicazioni o i tuoi servizi stiano emettendo metriche con nomi ed etichette che rispettano le convenzioni a livello dell'organizzazione per supportare gli sforzi di monitoraggio concentrati sui percorsi end-to-end centrali per la tua attività di business. Per ulteriori informazioni, vedi il documento relativo al [monitoraggio dei sistemi distribuiti](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").
