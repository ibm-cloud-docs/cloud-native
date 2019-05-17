---

copyright:
  years: 2019
lastupdated: "2019-02-10"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Controllo dell'integrità
{: #healthcheck}

I controlli dell'integrità forniscono un semplice meccanismo nei sistemi automatizzati per esaminare lo stato di integrità di una singola istanza. Il sistema risponde quindi a questi eventi di ispezione dell'integrità eseguendo un'azione, come ad esempio la sostituzione di un'istanza malfunzionante, l'aggiornamento delle tabelle di instradamento o la comunicazione degli stati di integrità risultanti all'utente.
{:shortdesc}

Kubernetes definisce due meccanismi integrali per controllare l'integrità di un contenitore:

* Viene utilizzato un probe di disponibilità per indicare se il processo può gestire le richieste (è instradabile). Kubernetes non instrada il lavoro a un contenitore con un probe di disponibilità in errore. Un probe di disponibilità dovrebbe avere esito negativo se un servizio non ha finito l'inizializzazione o se è in qualche altro modo occupato, sovraccaricato o non in grado di elaborare le richieste.
* Viene utilizzato un probe di attività per indicare se il processo deve essere riavviato. Kubernetes arresta e riavvia un contenitore con un probe di disponibilità in errore per assicurarsi che i pod che sono in uno stato che indica che sono oramai obsoleti vengano terminati e sostituiti. Un probe di attività dovrebbe avere esito negativo se il servizio è in uno stato irreversibile, ad esempio se si verifica una condizione di memoria esaurita. Dei semplici controlli di attività che restituiscono sempre una risposta OK possono identificare i contenitori in uno stato incongruente, cosa che può verificarsi quando si verifica un arresto anomalo del processo che serve le richieste ma il contenitore è ancora in esecuzione.

I probe di disponibilità e attività sono entrambi definiti utilizzando una struttura simile che include intervalli temporali di nuovi tentativi e ritardi, periodi di tolleranza agli errori, timeout e la definizione dell'implementazione del probe. Il probe può essere implementato eseguendo un comando, controllando un endpoint TCP per la connettività oppure eseguendo un richiamo HTTP. La stessa implementazione del probe può spesso essere utilizzata sia per scopi di disponibilità che di attività ma gli intervalli temporali di nuovi tentativi e ritardi devono essere regolati per lo specifico scopo.

## Descrizione e applicazione dei probe
{: #kubernetes-probes}

Fondamentalmente, lo sviluppo di applicazioni native del cloud si base sul principio che per i processi del contenitore si verificano in effetti delle condizioni di errore ma tali processi vengono prontamente sostituiti da un nuovo contenitore. Ciò si verifica sia in risposta ad eventi imprevisti come una condizione di errore del contenitore o della macchina ma anche a causa di eventi operativi come un ridimensionamento orizzontale e delle distribuzioni di nuove immagini dell'applicazione. I controlli della disponibilità sono importanti perché misurano se le nuove istanze del contenitore sono pronte a ricevere del lavoro prima di instradare il traffico e gli stessi controlli impediscono l'instradamento del traffico a istanze che sono state chiuse o di cui è in corso l'eliminazione.

Quando non sono disponibili dei controlli della disponibilità, Kubernetes non ha molte informazioni per determinare se un'istanza del contenitore è pronta a gestire del traffico e instrada il traffico immediatamente dopo che il processo del contenitore è stato avviato. Senza i controlli della disponibilità, ci sono maggiori probabilità che le applicazioni riscontrino dei timeout della connettività e delle risposte di rifiuto dalla connessione quando il lavoro viene instradato a un'istanza che non è pronta a soddisfare la richiesta. I controlli della disponibilità riducono, pur non eliminandoli del tutto, gli errori di connettività client

Anche se le modifiche alle destinazioni dell'instradamento dell'istanza sono normali durante il ciclo di vita di un'applicazione abilitata ai contenitori, gli stati del processo che è previsto vengano identificati dai controlli di attività sono meno frequenti e rappresentano un'eccezione, piuttosto che la norma. Quando passa a uno stato da cui non è possibile alcun ripristino, un processo diventa in effetti non operativo. Qualche esempio di perché ciò potrebbe accadere includono le condizioni di memoria esaurita o un deadlock causato da un errore di programmazione. Il modo migliore per il ripristino da situazioni come queste consiste nel terminare il contenitore, terminando così di conseguenza anche l'eventuale elaborazione attualmente in corso nel contenitore. Ciò crea anche la possibilità di terminare o riavviare i loop nell'applicazione, laddove i contenitori non sono in grado di passare completamente online prima di essere terminati e sostituiti.

I probe di disponibilità e attività hanno un impatto sul sistema in modi diversi. Si può pensare a ciò in termini di transizione dello stato, in cui lo stato positivo di un controllo della disponibilità è instradabile e lo stato negativo non è instradabile. In modo analogo, lo stato positivo di un controllo dell'attività rappresenta un contenitore in esecuzione normalmente e lo stato negativo è non operativo. Quando un contenitore viene avviato, lo stato di disponibilità è inizialmente negativo e passa a uno stato positivo solo dopo che il contenitore viene rilevato come integro. Un controllo dell'attività inizia in uno stato positivo e passa a uno stato negativo solo quando il processo diventa non operativo.

La configurazione di un controllo della disponibilità in modo molto aggressivo, ad esempio con un basso ritardo iniziale, ha uno scarso effetto poiché eseguire il probe troppo presto non causa una modifica dello stato del controllo della disponibilità. D'altro canto, un controllo dell'attività aggressivo, in cui il probe viene attivato troppo presto, causa una modifica di transizione dello stato, portando così il sistema a terminare il contenitore prima del previsto.

## Prassi ottimali per la configurazione dei probe
{: #probe-recommendation}

Quando implementi un probe di integrità utilizzando HTTP, considera i seguenti codici di stato HTTP per disponibilità, attività e integrità.

| Stato    |  Disponibilità        |  Attività             |
|----------|-----------------------|-----------------------|
|          | Non OK provoca nessun caricamento  | Non OK provoca il riavvio  |
|Avvio in corso | 503 - Non disponibile         | 200 - OK                   |
|Attivo | 200 - OK                   | 200 - OK                   |
|Arresto in corso | 503 - Non disponibile           | 200 - OK               |
|Inattivo | 503 - Non disponibile           | 503 - Non disponibile          |
| In errore  | 500 - Errore server          | 500 - Errore server            |
{: caption="Tabella 1. Codici di stato HTTP" caption-side="bottom"}

Gli endpoint di controllo dell'integrità non devono richiedere l'autorizzazione o l'autenticazione. Poiché queste protezioni non sono implementate sugli endpoint di probe di integrità, limita le eventuali implementazioni di probe HTTP alle richieste GET che non modificano alcun dato. Non restituire mai dei dati che identificano specifiche sull'ambiente, come il sistema operativo, la lingua di implementazione o le versioni software poiché possono essere utilizzate come un vettore di attacco.

Un probe di attività deve essere molto ponderato in merito a ciò che controlla poiché un malfunzionamento causa un'immediata terminazione del processo. Evita metriche ambigue che solo a volte indicano un processo malfunzionante, ad esempio un endpoint HTTP semplice che restituisce sempre `{"status": "UP"}` con un codice di stato 200. I processi host che sono in uno stato di non operativo non superano questo controllo, attivando pertanto correttamente un riavvio.

I controlli dell'integrità si verificano a intervalli frequenti, e questo può causare un ulteriore sovraccarico. I probe di disponibilità e attività devono testare solo l'applicabilità dei servizi di supporto come i database o altri microservizi nel loro risultato quando non c'è un fallback accettabile. Per un probe di attività, un controllo di supporto deve essere incluso solo se un risultato di non riuscito causerebbe il passaggio del contenitore locale a uno stato irreversibile. Un probe di disponibilità deve verificare un servizio di supporto solo quando il contenitore locale non è in grado di gestire le richieste se si verifica un suo malfunzionamento ma la condizione è reversibile.

Quando si configura il ritardo temporale iniziale, un probe di disponibilità deve utilizzare il valore più basso probabile e un controllo dell'attività deve utilizzare il valore temporale più grande probabile. Ad esempio, se un server delle applicazioni tende ad avviarsi in 30 secondi, un tipico ritardo di disponibilità è 10 secondi. Il controllo dell'attività utilizza un valore di 60 secondi per garantire che l'avvio del server sia sempre stato completato prima di controllare l'eventuale presenza di stati che possono essere terminati.

L'attributo *periodSeconds* per le decisioni di instradamento è di norma configurato so un valore di una singola cifra, a condizione che l'implementazione del probe sia relativamente leggera. Ad esempio, un probe HTTP che restituisce uno stato 200 OK senza una sostanziale elaborazione su lato server ha un carico del processore minimo e può essere prontamente ripetuto ogni 1-5 secondi.

## Configurazione dei probe in Kubernetes
{: #probe-config}

Dichiara i probe di attività e disponibilità accanto alla distribuzione di Kubernetes nell'elemento contenitore. Entrambi i probe utilizzano gli stessi parametri di configurazione.

| Parametro | Descrizione |
|-----------|-------------|
| *initialDelaySeconds* | Il lasso di tempo per cui il kubelet attende dopo la creazione del contenitore prima del primo probe. |
| *periodSeconds* | La frequenza con cui il kubelet esegue il probe del servizio. Il valore predefinito è 1. |
| *timeoutSeconds* | Con che frequenza il probe va in timeout. Il valore predefinito, e minimo, è 1.  |
| *successThreshold* | Il numero di volte per cui il probe deve avere esito positivo dopo un malfunzionamento. Il valore predefinito e minimo è 1. Il valore deve essere 1 per i probe di attività. |
| *failureThreshold* | Il numero di volte per cui Kubernetes proverà a riavviare il pod prima di rinunciare quando il pod viene avviato e il probe non riesce (vedi la nota). Il valore minimo è 1 e il valore predefinito è 3. |

  Per un probe di attività, rinunciare significa riavviare il pod. Per un probe di disponibilità, rinunciare significa contrassegnare il pod come non pronto.
  {: note}

Per evitare dei cicli di riavvio, imposta il parametro `livenessProbe.initialDelaySeconds` in modo che sia prudentemente più lungo del tempo necessario per inizializzare il tuo servizio. Puoi quindi utilizzare un valore più breve per l'attributo `readinessProbe.initialDelaySeconds` per instradare le richieste al servizio appena è pronto.

Una configurazione di esempio potrebbe essere qualcosa di simile a quanto segue (nota i valori di percorso e porta):

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

Per ulteriori informazioni, vedi il documento relativo alla [configurazione di probe di attività (Liveness) e disponibilità (Readiness) in Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").
