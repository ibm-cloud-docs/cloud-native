---

copyright:
  years: 2019
lastupdated: "2019-07-16"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Cosa si intende per nativo del cloud?
{: #overview}

Gli ambienti di calcolo cloud sono dinamici, con un'assegnazione e un rilascio dinamici delle risorse da un pool virtualizzato condiviso. Questi ambienti elastici consentono delle opzioni di ridimensionamento più flessibili rispetto all'assegnazione di risorse anticipata che viene di norma utilizzata nei data center in loco tradizionali.
{:shortdesc}

Secondo la [Cloud Native Computing Foundation](https://github.com/cncf/foundation/blob/master/charter.md){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"), i sistemi nativi del cloud hanno i seguenti attributi:

- Applicazioni o processi sono eseguiti in contenitori software come unità isolate.
- I processi sono gestiti da processi di orchestrazione centrali per migliorare l'utilizzo delle risorse e ridurre i costi di manutenzione.
- Le applicazioni o i servizi (microservizi) sono debolmente accoppiati con dipendenze descritte esplicitamente.

Questi attributi descrivono un sistema altamente dinamico che è composto da processi indipendenti che lavorano insieme per fornire un valore di business: un sistema distribuito.

L'elaborazione distribuita è un concetto che risale a decenni fa. [Fallacies of Distributed Computing](http://www.rgoarchitects.com/Files/fallacies.pdf){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") riporta le supposizioni di seguito indicate, espresse da architetti e designer, che, alla fine si sono rivelate errate. 

* La rete è affidabile.
* La rete è sicura.
* La rete è omogenea.
* La latenza è zero.
* La larghezza di banda è infinita.
* La topologia non cambia.
* C'è un solo amministratore.
* Il costo di trasporto è zero.

Le tecnologie cloud come Kubernetes e Istio intendono rispondere a queste preoccupazioni nell'infrastruttura stessa.

## Dodici fattori
{: #twelve-factors}

La metodologia dell'[applicazione a dodici fattori](https://12factor.net){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") è stata elaborata dagli sviluppatori di Heroku. Le caratteristiche menzionate nei dodici fattori non sono specifiche per un provider, una piattaforma o un linguaggio cloud. I fattori rappresentano una serie di linee guida o prassi ottimali per applicazioni portatili e resilienti che hanno negli ambienti cloud il loro ambiente ideale (in particolare l'applicazione SaaS (Software as a Service)). I dodici fattori sono forniti nel seguente elenco:

1. C'è un'associazione uno-a-uno tra una codebase con controllo delle versioni, ad esempio un repository Git, e un servizio distribuito. La stessa codebase viene utilizzata per molte distribuzioni.
2. I servizi dichiarano esplicitamente tutte le dipendenze e non fanno affidamento sulla presenza di librerie o strumenti a livello di sistema.
3. La configurazione che varia tra gli ambienti di distribuzione è memorizzata nell'ambiente, in modo specifico nelle variabili di ambiente.
4. Tutti i servizi di supporto sono trattati come risorse collegate, che sono gestite (collegate e scollegate) dall'ambiente di esecuzione.
5. La pipeline di distribuzione ha delle fasi nettamente separate: creazione, rilascio, esecuzione.
6. Le applicazioni sono distribuite come uno o più processi senza stato. In modo specifico, i processi temporanei sono senza stato e non condividono nulla. I dati indicati come persistenti sono archiviati in un servizio di supporto appropriato.
7. I servizi completi si rendono disponibili per gli altri servizi restando in ascolto su una specifica porta.
8. La simultaneità viene ottenuta ridimensionando i singoli processi (ridimensionamento orizzontale).
9. I processi sono disponibili: delle modalità di funzionamento che prevedono un avvio rapido e un arresto normale producono un sistema più solido e resiliente.
10. Tutti gli ambienti, dallo sviluppo locale alla produzione, sono il più simili possibile.
11. Le applicazioni producono dei log come flussi di eventi, ad esempio, scrivendo in `stdout` e `stderr`, e fanno affidamento sull'ambiente di esecuzione per aggregare i flussi.
12. Se sono necessarie attività di gestione una tantum, sono tenute nel controllo origine e assemblate a fianco all'applicazione per garantire che siano eseguite con lo stesso ambiente dell'applicazione.

Non devi rispettare in modo rigoroso questi fattori per ottenere un ambiente di microservizi di qualità. Ma con essi puoi creare e gestire applicazioni o servizi portatili in ambienti di distribuzione continua.

## Applicazioni
{: #apps-intro}

Un'applicazione comprende codice, dati, servizi e toolchain. Ad esempio, l'applicazione mobile {{site.data.keyword.cloud_notm}} contiene il codice dispositivo, insieme alla logica di backend, all'archiviazione dei dati e ai servizi di analisi e di sicurezza ed è configurata per la fornitura continua. 

![Riutilizzo](images/garage_reuse2.png "Con Developer Experience, puoi riutilizzare ed evitare di reinventare") 

Puoi creare e gestire un'applicazione utilizzando qualsiasi portale sviluppatori {{site.data.keyword.cloud_notm}} o {{site.data.keyword.dev_cli_notm}}. 

Puoi creare direttamente delle applicazioni vuote semplici o crearne di più complesse utilizzando i nostri kit starter. Se scegli di creare delle applicazioni vuote senza l'aiuto di un kit starter, puoi farlo dal [dashboard {{site.data.keyword.cloud_notm}} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")](https://{DomainName}){: new_window} senza visitare un portale. 

Puoi utilizzare un modello di codice per creare rapidamente la tua applicazione e distribuirla a {{site.data.keyword.cloud_notm}}. Dal [sito web di IBM Developer ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")](https://developer.ibm.com/patterns/){:new_window}, scegli un modello di codice. Puoi visualizzare il codice in GitHub o creare e compilare un'applicazione su {{site.data.keyword.cloud_notm}}, dove puoi utilizzare una toolchain DevOps per distribuire automaticamente la tua applicazione.

## Microservizi
{: #microservices}

Un *microservizio* è un insieme di piccoli componenti dell'architettura indipendenti che comunicano su una leggera API comune. Ogni microservizio nel semplice esempio riportato di seguito è un'applicazione a dodici fattori che utilizza i servizi di supporto sostituibili per archiviare i dati e passare i messaggi:

![Un'applicazione di microservizi](images/microservice.png "Un'applicazione di microservizi")

I microservizi sono indipendenti. L'agilità è uno dei vantaggi delle architetture di microservizi ma è un fatto solo quando i servizi possono essere riscritti senza disturbare gli altri servizi.  

Cancellare i confini delle API dà ai team che lavorano su un servizio la massima flessibilità di evolvere l'implementazione. Questa caratteristica è quanto abilita la persistenza e la programmazione poliglotta.

I microservizi sono resilienti. La stabilità delle applicazioni dipende dalla resistenza dei singoli microservizi ai malfunzionamenti. Questa è una notevole differenza rispetto alle architetture tradizionali, in cui l'infrastruttura di supporto gestisce i malfunzionamenti per tuo conto. Ogni servizio deve applicare dei pattern di isolamento, come ad esempio degli interruttori di circuito o dei bulkhead, per contenere i malfunzionamenti e definire le modalità di funzionamento di fallback appropriate per proteggere i servizi upstream.

I microservizi sono senza stato e vengono archiviati in servizi cloud di supporto esterni, come Redis. Una modalità di funzionamento che prevede avvii rapidi e arresti normali consente ulteriormente ai microservizi di funzionare bene in ambienti automatizzati che creano e rimuovono istanze in risposta al carico o per mantenere l'integrità del sistema.

### Il significato di "piccolo"
{: #small-microsvc}

L'uso della parola "piccolo", così come viene applicato a un microservizio, significa che ha uno scopo dedicato. Molte descrizioni fanno dei paralleli tra i ruoli dei singoli microservizi e i comandi concatenati sulla riga di comando Unix:

```
ls | grep 'service' | sort -r
```
{:pre}

Questi comandi UNIX eseguono ognuno delle attività nettamente diverse e puoi concatenarli indipendentemente dal linguaggio di programmazione o dalla quantità di codice.

## Applicazioni poliglotte: scelta dello strumento giusto per il lavoro
{: #polyglot-apps}

Il poliglottismo è un vantaggio spesso citato delle architetture basate sui microservizi. Crea un equilibrio tra i due fattori creando un elenco di tecnologie supportate da cui scegliere, con una politica definita per estendere l'elenco con nuove tecnologie nel corso del tempo. Assicurati che i requisiti non funzionali o regolamentari come la gestibilità, la controllabilità e la sicurezza dei dati possano essere soddisfatti, conservando al tempo stesso l'agilità e supportando l'innovazione mediante la sperimentazione.

... :FIXME: unpack? Language table here?:...

## REST e JSON
{: #rest-json}

Le applicazioni poliglotte sono possibili solo con protocolli indipendenti dai linguaggi. I pattern dell'architettura REST definiscono le linee guida per creare delle interfacce uniformi che separano la rappresentazione dei dati in transito dall'implementazione del servizio.

JSON, nelle architetture di microservizi, è il formato di trasmissione preferito per i dati basati su testo, rimpiazzando XML grazie alla semplicità e sinteticità che offre rispetto a esso. Per fare un confronto, il seguente esempio è un record di base che contiene i dati su un dipendente in JSON:

```json
{
  "name": "Marley Cassin",
  "serial": 228264,
  "title": "Senior Software Engineer",
  "address": {
    "office": "501-B101",
    "street": "3858 Kuvalis Pass",
    "city": "East Craig",
    "state": "NC",
    "zip": "64519-8934"
  }
}
```
{: codeblock}

E il seguente esempio è lo stesso record di dipendente in XML:

```xml
<person>
  <name>Marley Cassin</name>
  <serial>228264</serial>
  <title>Senior Software Engineer</title>
  <address>
    <office>501-B101</office>
    <street>3858 Kuvalis Pass</street>
    <city>East Craig</city>
    <state>NC</state>
    <zip>64519-8934</zip>
  </address>
</person>
```
{: codeblock}

JSON utilizza le coppie attributo-valore per rappresentare gli oggetti di dati in una sintassi concisa che conserva le informazioni su alcuni tipi di base, come i numeri, le stringhe, gli array e gli oggetti. Sia JSON sia XML rappresentano chiaramente l'oggetto di indirizzo indicato negli esempi precedenti ma hai bisogno dello schema XML associato per determinare il tipo dell'elemento `serial`. In JSON, la sintassi rende chiaro che il valore di `serial` è un numero e non una stringa.
