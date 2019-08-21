---

copyright:
  years: 2019
lastupdated: "2019-07-18"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Osservabilità, telemetria e monitoraggio
{: #observability-cn}

Per quanto riguarda il monitoraggio, si sta imponendo un cambiamento che vede un orientamento di natura nativa del cloud. Anche se si prevede che le applicazioni sia in ambienti in loco che native del cloud siano altamente disponibili e resilienti alle condizioni di errore, i metodi utilizzati per raggiungere questi obiettivi sono differenti. Di conseguenza, lo scopo del monitoraggio cambia, passando dal monitoraggio per evitare le condizioni di errore alla gestione di tali condizioni. 
{:shortdesc}

Negli ambienti in loco, il provisioning di infrastruttura e middleware viene eseguito in base a pattern di capacità e di alta disponibilità pianificati, ad esempio attivo-attivo o attivo-passivo. Degli errori imprevisti in questo ambiente possono essere complessi e richiedere un notevole sforzo per determinare qual è il problema e risolverlo. Il monitoraggio esterno viene eseguito da agent che esaminano l'utilizzo delle risorse per evitare classi note di errori. Come un esempio, considera la regolazione della dimensione dell'heap, dei timeout e delle politiche di raccolta dei dati obsoleti per le applicazioni Java.

Un'applicazione nativa del cloud è formata da microservizi indipendenti e servizi di supporto richiesti. Anche se un'applicazione nativa del cloud nel suo insieme deve rimanere disponibile e continuare a funzionare, le singole istanze del servizio verranno avviate o arrestate per adattarsi ai requisiti di capacità o per eseguire ripristini da condizioni di malfunzionamento.  

## Osservabilità
{: #observability}

Il monitoraggio di questo sistema fluido richiede che ogni partecipante sia *osservabile*. Ogni entità deve produrre i dati appropriati per supportare il rilevamento dei problemi automatizzato e gli avvisi, il debug manuale quando necessario, e l'analisi dell'integrità del sistema (tendenze e analisi cronologiche).

Che tipi di dati deve produrre un servizio per essere osservabile?

* I **controlli dell'integrità** (spesso endpoint HTTP personalizzati) aiutano gli orchestratori, come Kubernetes o Cloud Foundry, a eseguire azioni automatizzate per la manutenzione dell'integrità complessiva del sistema.
* Le **metriche** sono una rappresentazione numerica dei dati raccolti a intervalli in una serie temporale. I dati numerici delle serie temporali sono facili da archiviare e interrogare, il che aiuta quando si cercano delle tendenze cronologiche. Su un periodo più lungo, i dati numerici possono essere compressi in aggregati meno granulari, ad esempio giornalieri o settimanali. 
* Le **voci di log** rappresentano degli eventi discreti. Le voci di log sono essenziali per il debug poiché spesso includono delle tracce di stack e altre informazioni contestuali che possono aiutare a identificare la causa principale dei malfunzionamenti osservati.
* La **traccia distribuita, richiesta o end-to-end** acquisisce il flusso end-to-end di una richiesta attraverso il sistema. La traccia essenzialmente acquisisce sia le relazioni tra i servizi (i servizi che sono stati toccati dalla richiesta) che la struttura di lavoro per il sistema (elaborazione sincrona o asincrona, relazioni di subordinazione (child-of) o derivazione (follows-from)). 

## Telemetria
{: #telemetry}

Le applicazioni native del cloud fanno affidamento sull'ambiente per la *telemetria*, che è la raccolta e la trasmissione automatiche dei dati a ubicazioni centralizzate per una successiva analisi. Ciò è sottolineato da uno dei dodici fattori, secondo il quale i log vanno trattati come flussi di eventi, e si estende a tutti i dati prodotti da un microservizio per garantire che possa essere osservato.

Kubernetes ha delle funzionalità di telemetria integrate, come Heapster, ma è più probabile che la telemetria venga fornita da altri sistemi che si integrano con il piano di controllo Kubernetes. Per fare un esempio, due dei componenti di Istio, Mixer ed Envoy, agiscono insieme per [raccogliere la telemetria dalle applicazioni distribuite](https://istio.io/docs/concepts/policies-and-telemetry/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") in modo trasparente.

I malfunzionamenti non sono più eventi rari che causano interruzioni del servizio. Suddividere un'applicazione monolitica in microservizi spinge una parte maggiore del percorso della linea principale sulla rete, aumentando l'impatto della latenza e di altri problemi di rete. Le richieste raggiungono anche dei processi che non sono pronti per il lavoro per una serie di motivi. I servizi vengono automaticamente riavviati se esauriscono le risorse e le strategie di tolleranza agli errori consentono al sistema nel suo insieme di continuare a funzionare. Un intervento manuale per i singoli malfunzionamenti non è utile o fattibile in questo tipo di ambiente. 

## Monitoraggio
{: #monitoring}

I concetti di osservabilità e telemetria aiutano ad evidenziare alcune differenze significative nel modo in cui vengono monitorate le applicazioni native del cloud in sistemi distribuiti su larga scala. Ricordati che i processi negli ambienti nativi del cloud sono temporanei. Tre dei dodici fattori (processi, simultaneità e disponibilità) sottolineano questo punto. I processi monolitici preassegnati e a lunga esecuzione sono sostituiti o circondati da molti più processi di breve durata che sono avviati o arrestati in risposta al carico per il ridimensionamento orizzontale o se non stanno funzionando correttamente. La telemetria è critica quando i dati devono essere raccolti e conservati in modo persistente da qualche altra parte per evitare che vadano persi quando i processi (contenitori) vengono creati ed eliminati in modo permanente. La telemetria è spesso richiesta anche per motivi di conformità. 

Per tutti i motivi sopra descritti, il monitoraggio ha ora un altro punto focale: invece di monitorare la modalità di funzionamento e l'integrità delle risorse (singoli processi o singole macchine) lo stato del sistema viene monitorato nel suo insieme. Ogni singolo servizio produce dati che alimentano questa vista aggregata.

## Confronto tra traccia, registrazione e metriche
{: #trace-log-metrics}

:FIXME -- Riscrivi il confronto come riepilogo dell'argomento:

La registrazione viene utilizzata quando lo sviluppatore vuole emettere esplicitamente alcuni messaggi in modo che qualcuno li veda. Vengono codificati direttamente nella classe Java, incluso il passaggio insieme ai valori delle variabili rilevanti. Quando si verificano dei problemi, i log sono utili per scopi di debug, i quali mostrano dove si è verificato un errore, come ad esempio una traccia di stack di un'eccezione che è stata generata. Utilizzi *Kibana* per visualizzare una vista federata di tali log tra i pod/microservizi.

Il tracciamento si verifica automaticamente, per cui lo sviluppatore non deve effettuare alcuna azione. Ad esempio, puoi configurare Liberty in modo che invii i record di traccia a server di traccia conforme alla traccia aperta se viene richiamato un qualsiasi metodo di annotazione JAX-RS. In questo modo hai un record di controllo su cosa viene richiamato quando, da chi e quanto tempo è stato necessario. Puoi anche aumentare tale traccia, ad esempio includendo le informazioni su quali metodi privati sono stati richiamati nel tuo codice, aggiungendo le annotazioni di traccia aperta ai metodi che vengono tracciati. 

Puoi utilizzare uno strumento come *Zipkin* o *Jaeger* per visualizzare una vista federata delle tracce tra i pod/microservizi. Una *rete di servizi* può anche fornire il tracciamento automatico delle chiamate passate tramite un sidecar per il tuo contenitore.  

Le metriche vengono utilizzate per tenere traccia dei valori aggregati. Invece di guardare attraverso una traccia o un log per vedere quanto spesso qualcuno crea dei portafogli, puoi farlo con una metrica di contatore personalizzata. Puoi etichettare la tua distribuzione come se ne "venisse eseguito lo scraping" da *Prometheus*, come ad esempio controllando periodicamente l'URI /metrics sui tuoi pod. Puoi quindi utilizzare uno strumento come *Grafana* per ottenere una vista federata delle metriche tra i pod/microservizi.

Puoi utilizzare il tracciamento per scoprire come "questo metodo è stato richiamato". Ma utilizzerai la registrazione per scoprire "cosa è successo all'interno di tale metodo quando è stato richiamato". E utilizzerai le metriche per vedere "quante volte è stato richiamato". Normalmente il tracciamento è implicito, si verifica automaticamente senza alcuno sforzo da parte dello sviluppatore. Invece la registrazione è esplicita, richiede allo sviluppatore di codificare le informazioni inviate che potrebbero essere importanti per le analisi post-mortem di un problema. Anche le metriche sono esplicite, nella misura in cui devi aggiungere l'annotazione al metodo appropriato del tuo codice (anche se ci sono spesso metriche predefinite di basso livello disponibili senza sforzo da parte del programmatore, come utilizzo di memoria, utilizzo di CPU e numero di thread).

Normalmente le metriche sono utili per le analisi, invece la registrazione è più utile per scopi di determinazione del problema e la traccia è più utile nella comprensione del flusso di controllo da microservizio a microservizio.
