---

copyright:
  years: 2019
lastupdated: "2019-02-18"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Concetti della piattaforma cloud
{: #platform}

Questa sezione fornisce una breve panoramica delle tecnologie e dei concetti principali con cui interagiscono gli sviluppatori quando creano applicazioni native del cloud, iniziando con i contenitori, Kubernetes, Helm e Istio.
{:shortdesc}

## Contenitori
{: #containers}

I contenitori sono un meccanismo standard per assemblare un'applicazione e tutte le sue dipendenze in una singola unità completa. I contenitori risolvono il problema della portatilità: la risorsa contenitore (immagine) garantisce che tutto quanto occorre a un'applicazione per l'esecuzione sia nel posto giusto; i motori dei contenitori possono quindi concentrarsi sull'esecuzione dei contenitori come processi isolati in un modo efficiente, protetto e sicuro.

Le immagini del contenitore sono di norma create da un elenco di istruzioni definite in un `Dockerfile`. Le immagini del contenitore sono quasi sempre create da altre immagini del contenitore (essenzialmente come una continuazione delle istruzioni da uno stato precedente noto). Puoi utilizzare il seguente frammento di codice per creare una tua immagine Open Liberty, ad esempio:

```yaml
FROM open-liberty:kernel
COPY server.xml /config/
```
{: codeblock}

Dopo essere stata creata, un'immagine può essere eseguita. I motori di esecuzione dei contenitori, come Docker o [containerd](https://containerd.io/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"), prendono tale definizione dell'immagine ed eseguono il punto di ingresso definito come un processo isolato dalle risorse direttamente in aggiunta al sistema operativo host, eliminando il sovraccarico delle macchine virtuali.

Le immagini del contenitore sono archiviate in *registri*. Quello più noto è il registro pubblico Docker Hub ma, più comunemente, esegui il push e il pull delle immagini verso e da registri dei contenitori con controllo dell'accesso, come {{site.data.keyword.registryshort_notm}}, che sono più strettamente associate alla tua infrastruttura e alle pipeline CI/CD.

## Kubernetes
{: #kubernetes}

Le piattaforme cloud di IBM si avvalgono di Kubernetes per l'orchestrazione dei contenitori. Pertanto, oltre a conoscere i concetti di base dei contenitori, è importante che gli sviluppatori abbiano dimestichezza con gli aspetti fondamentali di Kubernetes, compresi i comandi di base e le risorse di distribuzione. La seguente tabella include alcuni importanti concetti di Kubernetes.

| Concetto | Descrizione |
|---------|-------------|
| Pod | Un gruppo localizzato di contenitori che vengono distribuiti insieme come una singola unità. I pod sono relativamente immutabili ed è necessario sostituire il pod originale per apportare modifiche a diversi attributi del pod. Un'applicazione tipica ha un contenitore con la logica aziendale di base e, facoltativamente, dei pod aggiuntivi che forniscono funzionalità di piattaforma al livello granulare del pod. |
| Distribuzione | Un template ripetibile per un pod senza stato, che aggiunge una dimensione di scala al concetto di pod. È inoltre possibile aggiornare la definizione in formato template e sostituire le istanze pod sottostanti. Una configurazione di distribuzione Kubernetes viene monitorata da un controller della distribuzione Kubernetes per garantire che venga mantenuto il numero dichiarato di pod per una specifica distribuzione. Una distribuzione viene visualizzata come `kind: Deployment` nei file `.yaml`. |
| Servizio | Un nome noto che rappresenta una serie di indirizzi IP pod relativamente instabili. Un servizio può esistere solo sulla rete privata del cluster oppure essere esposto esternamente, di norma utilizzando un programma di bilanciamento del carico specifico per il provider cloud. Un servizio viene visualizzato come `kind: Service` nei file `.yaml`. |
| Ingress | Consente di condividere un singolo indirizzo di rete con più servizi mediante l'hosting virtuale o l'instradamento basato sul contesto. Un Ingress può anche eseguire delle attività di gestione delle connessioni di rete come la terminazione TLS. Un Ingress viene visualizzato come `kind: Ingress` nei file `.yaml`. |
| Segreto | Un oggetto che archivia informazioni sensibili per l'utilizzo di runtime del pod e che separa le informazioni specifiche per la distribuzione dall'orchestrazione o dall'immagine del contenitore. Un segreto può essere esposto come un pod al runtime mediante variabili di ambiente o montaggi VFS (virtual file system). Senza i segreti, i dati sensibili sono archiviati nell'orchestrazione o nell'immagine del contenitore, ed entrambe creano maggiori opportunità di un'esposizione accidentale o di un accesso non previsto. |
| ConfigMap | Ha un ruolo simile ai segreti in quanto separa le informazioni specifiche per la distribuzione dall'orchestrazione del contenitore. Tuttavia, una ConfigMap è una struttura di configurazione per utilizzo generico. Viene utilizzata per eseguire il bind delle informazioni, come gli argomenti della riga di comando, delle variabili di ambiente e di altre risorse di configurazione ai componenti di sistema e ai contenitori del tuo pod in fase di runtime. | 

Tutte le risorse sono definite nel modello di risorsa Kubernetes, che può essere configurato mediante la API RESTful o mediante i file di configurazione inoltrati servendosi della riga di comando `kubectl`.

Per ulteriori informazioni, vedi [Kubernetes basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"), [Kubernetes Object Model](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") e [`kubectl` command line](https://kubernetes.io/docs/reference/kubectl/overview/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"). 

## Helm
{: #helm}

Helm è un gestore pacchetti che offre un modo facile per trovare, condividere e utilizzare software creato per Kubernetes. Helm risolve anche un'esigenza comune dell'utente: distribuire la stessa applicazione a più ambienti. Helm utilizza i *grafici*, che sono delle raccolte di template che producono oggetti Kubernetes validi (YAML) in fase di installazione. Questi grafici sono creati da un linguaggio di template che include il supporto per variabili, operazioni di intervallo e altri elementi che riducono in notevole misura lo sforzo necessario per gestire i metadati di distribuzione di Kubernetes.

Per ulteriori informazioni, vedi [Helm](https://helm.sh/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

## Mesh di servizio Istio
{: #istio}

Istio è una piattaforma open-source per gestire e proteggere i microservizi. Funziona con gli agenti di orchestrazione come Kubernetes, fornendo un modo per gestire e controllare le comunicazioni tra i servizi.

Istio funziona utilizzando un modello sidecar. Un sidecar (un proxy Envoy) è un processo separato che si affianca alla tua applicazione. Il sidecar gestisce tutte le comunicazioni verso e dal tuo servizio e applica un livello comune di funzionalità a tutti i servizi indipendente dal framework o dal linguaggio di programmazione con cui è stato creato il servizio. In effetti, Istio fornisce un meccanismo per configurare in modo centralizzato politiche di instradamento e di sicurezza pur lasciando che tali politiche vengano applicate mediante dei sidecar in un modo decentralizzato.

Consigliamo, perlopiù, di utilizzare le funzionalità fornite da Istio invece delle funzionalità simili fornite dai singoli framework o linguaggi di programmazione. Per fare un esempio, il bilanciamento del carico e altre politiche di instradamento sono definiti in modo più congruente e implementati dall'infrastruttura.

In alcuni casi, come con la traccia distribuita, Istio e le librerie a livello delle applicazioni sono complementari. Puoi migliorare le operazioni utilizzando entrambi insieme. In caso di traccia distribuita, Istio può solo garantire che le intestazioni di traccia siano presenti; le librerie delle applicazioni forniscono l'importante contesto in merito alle relazioni tra le richieste. Hai una migliore comprensione del sistema come un insieme quando sia Istio che le librerie di supporto o framework vengono utilizzati insieme.

Al livello più alto, Istio estende la piattaforma Kubernetes, fornendo ulteriori concetti di gestione, visibilità e sicurezza. Le funzioni di Istio possono essere suddivise nelle seguenti quattro categorie:

* Gestione del traffico: il controllo del traffico tra i tuoi microservizi per eseguire la suddivisione del traffico, il ripristino da condizioni di errore e le release canary.
* Sicurezza: la fornitura di crittografia, autorizzazione e autenticazione basata sull'identità avanzate tra i microservizi.
* Osservabilità: la raccolta di metriche e log per una migliore visibilità nelle applicazioni in esecuzione nel tuo cluster.
* Politiche: l'implementazione di controlli dell'accesso, limiti della frequenza e quote per proteggere le tue applicazioni.

Per ulteriori informazioni, vedi [What is Istio?](https://istio.io/docs/concepts/what-is-istio/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").



