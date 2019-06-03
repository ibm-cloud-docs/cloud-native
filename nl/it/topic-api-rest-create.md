---

copyright:
  years: 2019
lastupdated: "2019-05-20"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Creazione di microservizi RESTful
{: #rest-api}

Le applicazioni native del cloud producono e consumano le API, in un'architettura di microservizi o meno. Alcune API sono considerate interne, o private, e alcune sono considerate esterne. 
{:shortdesc}

Le API interne sono utilizzate solo all'interno di un ambiente protetto da firewall per consentire ai servizi di backend di comunicare tra loro. Le API esterne presentano un punto di ingresso unificato per i consumatori e sono spesso **gestite** da strumenti come {{site.data.keyword.apiconnect_long}}, che possono imporre una limitazione della frequenza o altri vincoli di utilizzo. Un esempio di una API del genere è l'[API GitHub Developer](https://developer.github.com/v3/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"). Fornisce una singola API unificata con un utilizzo congruente dei codici di restituzione e dei verbi HTTP, della modalità di funzionamento della paginazione e così via, senza esporre i dettagli dell'implementazione interna. Questa API può essere supportata da una singola applicazione unica oppure può essere supportata da una raccolta di microservizi: questo dettaglio non è esposto al consumatore, che lascia che GitHub sia libera di evolvere i suoi sistemi interni come necessario.

## Prassi ottimali per le API RESTful
{: #bps-apis}

Le API REST devono utilizzare i verbi HTTP standard per le operazioni CRUD (Create, Retrieve, Update, and Delete), prestando speciale attenzione al fatto che l'operazione sia idempotente, ossia che sia possibile ripeterla più volte senza alcun problema.

* Le operazioni POST possono essere utilizzate per creare o aggiornare le risorse. Le operazioni POST non possono essere richiamate ripetutamente. Ad esempio, se una richiesta POST viene utilizzata per creare le risorse ed è richiamata più volte, viene creata una nuova risorsa univoca come risultato di ciascun richiamo.
* Le operazioni GET devono poter essere richiamate ripetutamente e non devono causare effetti collaterali. Devono essere utilizzate solo per richiamare le informazioni. Le richieste GET con i parametri di query non devono essere utilizzate per modificare o aggiornare le informazioni. Utilizza invece le operazioni POST, PUT o PATCH.
* Le operazioni PUT possono essere utilizzate per aggiornare le risorse. Le operazioni PUT di norma includono una copia completa della risorsa da aggiornare, rendendo così possibile richiamare l'operazione più volte.
* Le operazioni PATCH consentono l'aggiornamento parziale delle risorse. Possono essere richiamate ripetutamente, a seconda di come viene specificato il delta, e poi applicate alla risorsa. Ad esempio, se un'operazione PATCH indica che un valore deve essere modificato da A a B, può essere richiamata ripetutamente. Non succede niente se viene richiamata più volte e il valore è già B.
* Le operazioni DELETE possono essere richiamate più volte, in quanto una risorsa può essere eliminata una sola volta. Tuttavia, il codice di restituzione varia, poiché la prima operazione ha esito positivo (`200` o `204`), mentre i richiami successivi non trovano la risorsa (`404` o `410`).

### Risultati descrittivi e adatti per le macchine
{: #rest-results}

Tenuto conto del fatto che le API vengono richiamate da software piuttosto che da esseri umani, è necessario prestare attenzione a comunicare informazioni al chiamante nel modo più efficace ed efficiente possibile.

Utilizza i codici di stato HTTP pertinenti e utili, come descritto nella seguente tabella: 

| Codice errore HTTP | Guida all'uso |
|-----------------|----------------|
| `200 (OK)` | Da utilizzare quando va tutto bene e non ci sono dati da restituire |
| `204 (NO CONTENT)` | Da utilizzare quando va tutto bene ma non ci sono dati di risposta |
| `201 (CREATED)` | Da utilizzare per le richieste POST che determinano la creazione di una risposta, che sia presente un corpo della risposta o meno |
| `409 (CONFLICT)` | Da utilizzare quando delle modifiche simultanee sono in conflitto |
| `400 (BAD REQUEST)` | Da utilizzare quando i parametri sono in formato non corretto |

Per ulteriori informazioni, vedi [Response Status Codes](https://tools.ietf.org/html/rfc7231#section-6){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"). 

Devi anche valutare quali dati restituire nella tua risposta per rendere efficienti le comunicazioni. Ad esempio, quando una risorsa viene creata con una richiesta POST, la risposta deve includere l'ubicazione della risorsa appena creata in un'intestazione Location. La risorsa creata è spesso inclusa anche nel corpo della risposta per eliminare la richiesta GET supplementare per recuperare la risorsa creata. Lo stesso vale per le richieste PUT e PATCH.

### URI di risorsa RESTful
{: #rest-uris}

Ci sono opinioni diverse su alcuni aspetti degli URI di risorsa RESTful. In generale, si è d'accordo che le risorse debbano essere dei nomi, non dei verbi, e che gli endpoint siano plurali. Ciò produce una struttura chiara per le operazioni CRUD.

* `POST /accounts`: crea un nuovo account.
* `GET /accounts`: richiama un elenco di account.
* `GET /accounts/16`: richiama uno specifico account.
* `PUT /accounts/16`: aggiorna uno specifico account.
* `PATCH /accounts/16`: aggiorna uno specifico account.
* `DELETE /accounts/16`: elimina uno specifico account.

Le relazioni sono modellate utilizzando degli URI gerarchici, ad esempio ` /accounts/16/credentials` per la gestione delle credenziali associate a un account.

Si è meno d'accordo su cosa dovrebbe accadere con le operazioni associate alla risorsa che non rientrano in questa solita struttura. Non c'è un singolo modo corretto per gestire queste operazioni: fai quello che funziona meglio per il consumatore della API.

### Solidità e API RESTful
{: #robust-api}

Il principio di solidità, in inglese [Robustness Principle](https://tools.ietf.org/html/rfc1122#page-12){: new_window}, ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") fornisce l'orientamento migliore, invitandoti a "essere liberale con quanto accetti e conservativo con quanto invii." Presumi inoltre che le API si evolveranno nel corso del tempo e sii tollerante con i dati che non comprendi.

#### Produzione di API
{: #robust-producer}

Quando fornisci una API a client esterni, ci sono due cose che devi fare quando accetti richieste e restituisci risposte: 

* Accetta attributi sconosciuti come parte della richiesta.
    
    - Se un servizio richiama la tua API con attributi non necessari, semplicemente sbarazzati di questi valori. La restituzione di un errore in questo scenario può causare delle condizioni di errore non necessarie, con ripercussioni negative sull'utente finale.
* Restituisci solo gli attributi richiesti dai tuoi consumatori 
    - Evita di esporre dettagli del servizio interno. Esponi solo gli attributi di cui i consumatori hanno bisogno come parte della API.

#### Consumo di API
{: #robust-consumer}

Quando utilizzi le API:

* Convalida la richiesta solo rispetto alle variabili o agli attributi di cui hai bisogno.
    
    - Non eseguire una convalida rispetto a delle variabili solo perché vengono fornite. Se non le stai usando come parte della tua chiesta, non fare affidamento su fatto che ci siano.
* Accetta valori sconosciuti come parte della risposta.
    
    - Non emettere un'eccezione se ricevi una variabile imprevista. È sufficiente che la risposta contenga le informazioni di cui hai bisogno, il resto non conta.

Queste linee guida sono particolarmente rilevanti per linguaggi fortemente tipizzati come Java, dove la serializzazione e la deserializzazione JSON spesso si verifica in modo indiretto, ad esempio mediante librerie Jackson, o JSON-P/JSON-B. Cerca i meccanismi di linguaggio che ti consentono di specificare una modalità di funzionamento più generosa, come ad esempio ignorare gli attributi sconosciuti, o di definire o filtrare quali attributi devono essere serializzati.

### Controllo delle versioni delle API RESTful
{: #version-api}

Uno dei vantaggi principali dei microservizi è la capacità di consentire ai servizi di evolversi in modo indipendente. Tenuto conto del fatto che i microservizi richiamano altri servizi, questa indipendenza porta con sé un importante monito: non puoi causare delle modifiche importanti nella tua API.

Se viene rispettato il principio di solidità, puoi volerci del tempo prima che sia necessaria una modifica importante. Quando infine arriva il momento di una modifica importante, puoi scegliere di creare un servizio del tutto differente e ritirare gradualmente quello originale.

Nel caso in cui tu debba apportare delle modifiche importanti alla API per un servizio esistente, decidi come gestire queste modifiche: sarà il servizio a gestire tutte le versioni della API o manterrai delle versioni indipendenti del servizio per supportare ciascuna versione della API oppure sarà il tuo servizio a supportare solo la versione più recente della API e farà affidamento sugli altri livelli adattivi per eseguire la conversione verso e dalla API meno recente?

Dopo che hai determinato come gestire le modifiche, il problema molto più facile da risolvere è come riflettere la versione nella tua API. Ci sono generalmente tre modi per controllare la versione di una risorsa REST:

* Includere la versione nel percorso URI.
* Includi la versione nell'intestazione HTTP Accept e fare affidamento sulla negoziazione del contenuto.
* Utilizzare un'intestazione della richiesta personalizzata.

#### Inclusione della versione nel percorso URI
{: #version-path}

Il modo più semplice per specificare una versione è quello di includerla nel percorso dell'URI. Questo approccio presenta dei vantaggi: è chiaro, è facile da raggiungere quando crei i servizi nella tua applicazione ed è compatibile con gli strumenti di esplorazione delle API come Swagger e gli strumenti di riga di comando come `curl` e così via.

Se includerai la versione nel percorso dell'URI, la versione deve essere valida per la tua applicazione nel suo insieme, ad esempio `/api/v1/accounts` invece di `/api/accounts/v1`. HATEOAS (Hypermedia as the Engine of Application State) è un modo per fornire gli URI ai consumatori di API in modo che non siano responsabili essi stessi della creazione degli URI. GitHub, ad esempio, fornisce [URL ipermediali](https://developer.github.com/v3/#hypermedia){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") nelle risposte per questo motivo. HATEOAS diventa difficile, se non impossibile, da ottenere se diversi servizi di backend possono avere delle versioni che variano in modo indipendente nei loro URI.

#### Modifica dell'intestazione Accept per includere la versione
{: #version-accept}

L'intestazione Accept è il posto ovvio dove definire una versione ma è uno dei pi difficili da testare. L'intestazione Accept è anche un frequente punto di approdo per le attivazioni/disattivazioni di funzioni. La specifica di intestazioni HTTP richiede dei richiami API più dettagliati.

#### Aggiunta di un'intestazione di richiesta personalizzata
{: #version-custom}

Puoi aggiungere un'intestazione di richiesta personalizzata per indicare la versione API. Come con l'intestazione Accept, puoi anche utilizzare le intestazioni personalizzate per instradare il traffico a specifiche istanze di backend. Con questo metodo, riscontri gli stessi problemi di facilità d'uso che riscontri con il metodo di intestazione Accept, con il requisito aggiuntivo che i consumatori debbano avere una formazione in merito a questa intestazione.

Per ulteriori informazioni, vedi [Your API versioning is wrong, which is why I decided to do it 3 different wrong ways](https://www.troyhunt.com/your-api-versioning-is-wrong-which-is/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

## Creazione e generazione di API
{: #create-api}

[OpenAPI v3](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") è la specifica ufficiale per i servizi RESTful, regolamentata dalla [OpenAPI Initiative](https://www.openapis.org/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"), un'associazione di aziende che fa capo alla Linux Foundation.

Puoi utilizzare uno dei seguenti modi per creare una API:

  * Inizia con una definizione OpenAPI (dall'alto verso il basso): in questo approccio, inizi creando una definizione OpenAPI in un formato indipendente dal linguaggio (di norma YAML). Utilizzi quindi un generatore di codice per creare una struttura di base e, da essa, sviluppi la tua implementazione del servizio. Questo pattern viene di norma adottato dalle società che hanno un team di progettazione di API centrale e consente un avanzamento in parallelo di sviluppo e test.
  * Inizia con il codice (dal basso verso l'alto): il tuo codice è il sorgente della tua definizione API. Questo approccio funziona bene per le nuove applicazioni che presentano un aspetto sperimentale poiché la tua definizione di API si evolve man mano che acquisisci una maggiore comprensione di quello che il tuo servizio deve fare. Questo approccio funziona anche meglio in alcuni linguaggi piuttosto che in altri poiché si basa su strumenti che generano una definizione OpenAPI dal tuo codice. Java, ad esempio, ha un supporto eccellente per la generazione di documenti OpenAPI da framework REST basati su annotazioni.

In entrambi i casi, lavorare con una definizione OpenAPI può aiutare a identificare le aree dove la API è incongruente o difficile da capire dal punto di vista di un consumatore. Le definizioni OpenAPI pubblicate o con il controllo delle versioni possono anche essere utilizzate per creare strumenti di ausilio nell'indicazione di modifiche importanti che avrebbero ripercussioni sui consumatori.

### Creazione di una API da una definizione OpenAPI
{: #openapi-first}

Puoi creare il tuo file YAML OpenAPI nello strumento che preferisci. Tuttavia, l'utilizzo di un editor di testo semplice può esporre al rischio di errori. Alcuni editor hanno un supporto di base per YAML e qualcun altro può avere delle estensioni aggiuntive per supportare le definizioni OpenAPI. Ad esempio, puoi utilizzare le estensioni Visual Studio Code come [Swagger Viewer](https://marketplace.visualstudio.com/items?itemName=Arjun.swagger-viewer){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") oppure [OpenAPI Preview](https://marketplace.visualstudio.com/items?itemName=zoellner.openapi-preview){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") per convalidare la tua definizione OpenAPI rispetto a una determinata versione specificata e riprodurre una vista web nel riquadro di anteprima.

![OpenAPI Preview](images/create-api-image1.png "OpenAPI Preview") 

Sono disponibili anche diversi editor di analisi dal vivo e basati sui browser che puoi utilizzare online oppure localmente. Alcuni esempi includono:

* Il [progetto OpenAPI-GUI](https://github.com/Mermade/openapi-gui){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") supporta sia la v2 che la v3 della specifica OpenAPI e può migrare una definizione OpenAPI v2 alla v3 per tuo conto.
* [Swagger Editor di SmartBear](https://editor.swagger.io){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") supporta anch'esso sia la v2 che la v3 di OpenAPI.
* [{{site.data.keyword.apiconnect_short}}](https://cloud.ibm.com/catalog/services/api-connect){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") fornisce una serie di editor e strumenti per la modellazione e la creazione di API.

### Generazione dell'implementazione API
{: #code-first}

Puoi utilizzare un [generatore OpenAPI](https://github.com/OpenAPITools/openapi-generator){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") open-source per creare un progetto con una struttura di base per la tua implementazione del servizio da una definizione OpenAPI. Puoi specificare il linguaggio o il framework per la struttura di base dalla riga di comando. Ad esempio, per creare un progetto Java per la API PetStore di esempio che utilizza le annotazioni del metodo JAX_RS generiche, specifichi il seguente comando:

```bash
openapi-generator generate -g jaxrs-cxf-cdi -i ./petstore.yaml -o petstore --api-package=com.ibm.petstore
```
{: pre}

