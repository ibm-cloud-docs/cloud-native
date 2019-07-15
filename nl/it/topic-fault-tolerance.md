---

copyright:
  years: 2019
lastupdated: "2019-06-06"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:external: target="_blank" .external}

# Tolleranza agli errori
{: #fault-tolerance}

La tolleranza agli errori è la capacità di un sistema di continuare a funzionare nel caso in cui si verifichi un parziale malfunzionamento. La creazione di un sistema resiliente impone dei requisiti su tutti i servizi in esso. La natura dinamica degli ambienti cloud esige che i servizi siano scritti in modo da aspettarsi l'imprevisto e rispondere in modo normale quando ciò avviene, come ad esempio la ricezione di dati non validi, l'impossibilità di raggiungere un servizio di supporto necessario o la gestione di conflitti dovuti ad aggiornamenti simultanei in un ambiente distribuito. 

Le soluzioni di tolleranza agli errori di norma si concentrano su timeout, failback, bulkhead e interruttori di circuito.

In alcuni ambienti, i meccanismi di tolleranza agli errori potrebbero essere forniti da componenti dell'infrastruttura come Istio. Indipendentemente dall'eventuale supporto offerto dall'infrastruttura, un servizio deve presumere che la chiamata remota possa non riuscire e deve essere pronto con le appropriate azioni di fallback.

## Timeout

La prima linea di difesa contro malfunzionamenti parziali consiste nell'utilizzo di timeout. I timeout garantiscono che la tua applicazione riceva un errore quando un servizio di supporto non risponde, consentendole di gestire la condizione con l'adeguata modalità di funzionamento di fallback. Ciò non significa necessariamente che l'operazione richiesta non sia riuscita. I timeout modificano il tempo di attesa di una risposta di un client che effettua una richiesta e non hanno alcun impatto sulla modalità di funzionamento dell'elaborazione del servizio di destinazione.

Molte librerie di linguaggio utilizzano un timeout predefinito per le richieste e anche Istio lo fa. Per impostazione predefinita, il proxy sidecar indica la richiesta come non riuscita se non riceve una risposta entro 15 secondi. Questo valore può essere modificato impostando una politica di timeout per l'instradamento nella definizione VirtualService, che si presenta simile a quanto di seguito indicato, utilizzando un servizio che restituisce delle quotazioni azionarie, ad esempio:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: stock-quote-service
spec:
  hosts:
  - 'stock-quote-service'
  http:
  - route:
    - destination:
        host: stock-quote-service
    timeout: 30s
```
{: codeblock}

Con questa configurazione, le richieste fatte al servizio di quotazione azionaria mediante il proxy sidecar o il gateway ingress attendono 30 secondi prima di indicare come non riuscita la richiesta invece del valore predefinito di 15 secondi.

L'aspetto interessante della regolazione dei timeout in questo modo è che tutte le richieste che utilizzano l'instradamento hanno il timeout applicato. Questo è un livello di base di sicurezza fornito da Istio: anche se una libreria o un framework dell'applicazione non impone un timeout, non attenderà per sempre perché se ne occupa Istio. I timeout a livello dell'applicazione sono comunque validi. Dato l'esempio precedente, il timeout a livello dell'infrastruttura per il servizio di quotazione azionaria è stato esteso a 30 secondi. Se la libreria dell'applicazione imposta un timeout per 5 secondi, la richiesta dell'applicazione sarà comunque indicata come non riuscita con un timeout.

## Fallback

Un'applicazione deve definire cosa succede quando una richiesta a un servizio di supporto non riesce. Ci sono diverse opzioni ma l'obiettivo è quello di procedere a una riduzione non brusca quando tali servizi non rispondono in modo tempestivo. Quando un servizio remoto non riesce, è possibile ritentare la richiesta, provare una richiesta differente o restituire invece i dati memorizzati nella cache.

Ritentare la richiesta è, a prima vista, il meccanismo di fallback più facile. Quello che non è così evidente è che le richieste di nuovi tentativi possono contribuire a condizioni di errore del sistema a cascata (le cosiddette "tempeste di nuovi tentativi" (retry storm), una variazione del problema di collasso per attivazione simultanea di un elevato numero di thread di esecuzione ([thundering herd problem](https://en.wikipedia.org/wiki/Thundering_herd_problem){: external}). Il codice a livello dell'applicazione non rileva con sufficiente precisione l'integrità del sistema o della rete ed è difficile ottenere degli algoritmi di backoff esponenziale validi.

Istio può eseguire i nuovi tentativi in modo molto più efficace. È già direttamente coinvolto nell'instradamento delle richieste e fornisce un'implementazione congruente e indipendente dai linguaggi per le politiche di nuovi tentativi. Ad esempio, possiamo definire una politica come la seguente per il nostro servizio di quotazione azionaria:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: stock-quote-service
spec:
  hosts:
  - 'stock-quote-service'
  http:
  - route:
    - destination:
        host: stock-quote-service
    retries:
      attempts: 3
      perTryTimeout: 5s
```
{: codeblock}

Con questa semplice configurazione, le richieste fatte al servizio di quotazione azionaria mediante un gateway ingress o un proxy sidecar Istio verranno ritentate fino a 3 volte, con un timeout di 5 secondi per ogni tentativo. Delle [regole di corrispondenza degli instradamenti aggiuntive](https://istio.io/docs/reference/config/networking/#HTTPMatchRequest){: external} possono limitare ulteriormente questa politica di nuovi tentativi alle richieste `GET`, ad esempio.

C'è una sfumatura che facilmente potrebbe sfuggire, qui: non stai specificando l'intervallo dei nuovi tentativi. Il sidecar determina l'intervallo tra i nuovi tentativi e introduce deliberatamente il "jitter" tra i tentativi per evitare di bombardare i servizi sovraccaricati.

## Bulkhead

Il termine bulkhead si traduce letteralmente come paratia, che nel trasporto marittimo indica una partizione che impedisce che una falla in un compartimento affondi l'intera nave. Il pattern viene applicato negli ambienti cloud per ottenere uno scopo simile e viene realizzato in vari modi diversi

Per i linguaggi multithread come Java, i bulkhead interni possono essere utilizzati internamente per limitare o controllare la modalità di utilizzo delle risorse per le comunicazioni a risorse remote, utilizzando le code o i meccanismi si semaforo.

- Per utilizzare una coda, il servizio associa uno specifico numero di thread a una specifica coda. Tutte le richieste effettuate una volta che la coda è piena ottengono un errore rapido. In Java, può essere un `ThreadPoolExecutor` associato a una `BlockingQueue`, ad esempio.
- Il meccanismo di semaforo funziona con la disponibilità di un numero specifico di autorizzazioni. Una richiesta in uscita richiede un'autorizzazione. Dopo che una richiesta è stata completata con esito positivo, l'autorizzazione viene rilasciata per l'utilizzo da parte di un'altra richiesta.

Puoi definire dei bulkhead tra i servizi utilizzando una DestinationRule Istio per vincolare il pool di connessioni per un servizio di upstream utilizzando qualcosa come il seguente esempio:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: stock-quote-service
spec:
  host: stock-quote-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 10
        connectTimeout: 30s
```
{: codeblock}

Questa configurazione limita il numero massimo di connessioni simultanee fatte a ciascuna istanza del servizio di quotazione azionaria a 10. I servizi che non riescono a stabilire una connessione entro 30 secondi otterranno una risposta `503 -- Service Unavailable`. Questo tipo di bulkhead può essere utilizzato per evitare che un servizio che si sta occupando di un notevole carico di calcolo riceva più richieste di quante ne possa gestire, ad esempio.

## Interruttori di circuito

Gli interruttori di circuito vengono utilizzati per ottimizzare la modalità di funzionamento delle richieste in uscita quando si verificano dei malfunzionamenti. Invece di eseguire ripetutamente delle richieste a un servizio che non risponde, un interruttore di circuito osserva il numero di malfunzionamenti che si sono verificati entro uno specifico periodo di tempo. Se la frequenza degli errori supera una soglia, l'interruttore di circuito apre il circuito e indica la richiesta e tutte quelle successive come in errore finché il circuito non viene nuovamente chiuso.

Gli interruttori di circuito possono essere definiti anche utilizzando una DestinationRule Istio:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: stock-quote-service
spec:
  host: stock-quote-service
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 3
      interval: 5s
      baseEjectionTime: 5m
      maxEjectionPercent: 100
```
{: codeblock}

Questa configurazione pone dei vincoli sulle richieste che gli altri servizi stanno facendo al servizio di quotazione azionaria. La politica di traffico `outlierDetection` specificata viene applicata a ogni singola istanza. Per formulare la configurazione precedente come una frase: "Rimuovi qualsiasi istanza del servizio di quotazione azionaria che non riesce 3 volte in 5 secondi per almeno 5 minuti; inoltre, tutte le istanze possono essere rimosse". L'ultima impostazione `maxEjectionPercent` è correlata al bilanciamento del carico. Istio mantiene un pool di bilanciamento del carico e rimuove le istanze malfunzionanti da tale pool. Per impostazione predefinita, rimuove un massimo del 10% di tutte le istanze disponibili dal pool di bilanciamento del carico.

Per gli utenti che hanno dimestichezza con altri meccanismi di interruzione del circuito, Istio non ha uno stato semiaperto. Applica invece una semplice formula: un'istanza resta rimossa dal pool per `baseInjectionTime * <number of times it has been ejected>`. Ciò consente un ripristino da condizioni di errore temporanee delle istanze e, al tempo stesso, mantiene fuori dal pool le istanze che sono costantemente in errore.

