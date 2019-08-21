---

copyright:
  years: 2019
lastupdated: "2019-07-19"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Registrazione
{: #logging}

I messaggi di log sono stringhe con informazioni contestuali pertinenti allo stato e all'attività di un microservizio quando viene creata la voce di log. I log sono richiesti per diagnosticare come e perché i servizi hanno esito negativo. Giocano un ruolo di supporto alle metriche nel monitoraggio dell'integrità dell'applicazione.
{:shortdesc}

Assicurati di scrivere le voci di log direttamente in flussi di errore e di output standard. Ciò rende le voci di log visualizzabili utilizzando gli strumenti della riga di comando e consente ai servizi di inoltro dei log configurati a livello dell'infrastruttura di gestire la raccolta dei log e la gestione dei dati. 

La gestione dei file di log richiede una più attenta valutazione se non è possibile configurare un'applicazione inserita in un contenitore per scrivere i log nel flusso di output standard o di errori standard.

* Un'opzione consiste nell'utilizzare un volume per i dati di log, sia esso un montaggio di bind semplice per lo sviluppo e i test locali o un appropriato volume persistente come parte di una distribuzione Kubernetes. Un [agent di registrazione o un sidecar dedicato](https://kubernetes.io/docs/concepts/cluster-administration/logging/#sidecar-container-with-a-logging-agent){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") può leggere da un volume condiviso per inoltrare i log a un aggregatore centralizzato. La rotazione dei log deve essere configurata esplicitamente per controllare la quantità di dati di log archiviata nei volumi. 
* Un'altra opzione consiste nell'utilizzare librerie o agent dell'applicazione per inoltrare i log direttamente agli aggregatori. Questa opzione può comportare una certa complessità della configurazione negli ambienti di sviluppo.

## Registrazione JSON
{: #json-logging}

Man mano che la tua applicazione si evolve nel tempo, la natura di ciò che registri può cambiare. Utilizzando un formato di log JSON, ottieni i seguenti vantaggi:

* I log sono indicizzabili, il che rende molto più semplice la ricerca in un corpo aggregato di log.
* I log sono resilienti alla modifica, poiché l'analisi non è dipendente dalla posizione degli elementi in una stringa.

Se da una parte i log con formattazione JSON sono più facili da analizzare per le macchine, dall'altra sono più difficili da leggere per gli esseri umani. Valuta l'utilizzo di variabili di ambiente per alternare il formato di log tra testo semplice per lo sviluppo e il debug locale e i log con formattazione JSON per archiviazione e aggregazione a più lungo termine. 

I parser JSON della riga di comando, come lo strumento JSON Query (jq), possono essere utilizzati per creare delle viste leggibili di log con formattazione JSON. Nel seguente esempio, viene eseguito il piping dei log mediante grep per garantire che il campo di messaggio sia presente prima che jq analizzi la riga: 

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | \
  grep message | \
  jq .message -r
```
{: pre}

## Visualizzazione dei log utilizzando `kubectl` 
{: #view-logs-kube}

I log inviati ai flussi di errori e di output standard possono essere visualizzati in Kubernetes mediante la console oppure utilizzando i comandi `kubectl` che sono nel formato: `kubectl logs <podname>`.

Se utilizzi uno spazio dei nomi personalizzato, come stock-trader, includilo nel comando, `kubectl logs -n stock-trader <podname>`. 

Se ci sono più contenitori per ogni pod, come con i sidecar istio, devi anche specificare il contenitore. Nel seguente esempio, lo spazio dei nomi stock-trader viene utilizzato per visualizzare i log dal contenitore `portfolio` nel pod `portfolio-54b4d579f7-4zvzk`.

```bash
kubectl logs -n stock-trader portfolio-54b4d579f7-4zvzk -c portfolio
```
{: pre}

Per i log in formato JSON, puoi utilizzare `jq` per estrarre un campo di messaggio, ad esempio:

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | grep message | jq .message -r
```
{: pre}

Delle voci di log viene eseguito il piping mediante `grep` per garantire che `jq` analizzi le righe con un campo di messaggio.
{: note}
