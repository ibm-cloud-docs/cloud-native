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

# Configurazione:
{: #configuration}

Le applicazioni native del cloud devono essere portatili. Puoi utilizzare la stessa risorsa fissa per la distribuzione a più ambienti senza modificare il codice o esercitare percorsi di codice altrimenti non testati.
{:shortdesc}

Tre fattori dalla [metodologia a dodici fattori](https://12factor.net/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") sono direttamente correlati a questa prassi.

* Il primo fattore raccomanda una correlazione 1-a-1 tra un servizio in esecuzione e una codebase con il controllo delle versioni. Crea una risorsa di distribuzione fissa, come un'immagine Docker, da una codebase con il controllo delle versioni che può essere distribuita, immutata, a più ambienti.
* Il terzo fattore raccomanda una separazione tra la configurazione specifica per l'applicazione, che fa parte della risorsa fissa, e la configurazione specifica per l'ambiente, che viene fornita al servizio in fase di distribuzione.
* Il decimo fattore raccomanda di mantenere tutti gli ambienti simili il più possibile. I percorsi di codice specifici per l'ambiente sono difficili da testare e aumentano il rischio di malfunzionamenti man mano che esegui la distribuzione ad ambienti differenti. Ciò vale anche per i servizi di supporto. Se esegui attività di sviluppo e test con un database in memoria, potrebbero verificarsi dei malfunzionamenti imprevisti negli ambienti di test, preparazione o produzione perché utilizzano un database che ha una modalità di funzionamento differente.

## Origini di configurazione
{: #config-inject}

La configurazione specifica per l'applicazione fa parte della risorsa fissa. Ad esempio, le applicazioni in esecuzione su WebSphere Liberty definiscono un elenco di funzioni installate che controllano i file binari e i servizi attivi nel runtime. Questa configurazione è specifica per l'applicazione e viene inclusa nell'immagine Docker. Le immagini Docker definiscono anche la porta di ascolto o esposta, poiché l'ambiente di esecuzione gestisce l'associazione di porte all'avvio del contenitore. 

Una configurazione specifica per l'ambiente, come l'host e la porta utilizzati per comunicare con altri servizi, gli utenti del database o i vincoli di utilizzo delle risorse, viene fornita al contenitore dall'ambiente di distribuzione. La gestione della configurazione del servizio e delle credenziali può variare in modo significativo.

* Kubernetes memorizza i valori di configurazione (JSON sotto forma di stringa o attributi semplici) in ConfigMap o segreti. Possono essere passati all'applicazione inserite nel contenitore come variabili di ambiente o montaggi del file system virtuale. Il meccanico utilizzato da un servizio è specificato nei metadati di distribuzione, in YAML Kubernetes o nel grafico helm).
* Gli ambienti di sviluppo locale sono spesso varianti semplificate che utilizzano semplici variabili di ambiente chiave-valore.
* Cloud Foundry memorizza gli attributi di configurazione e i dettagli di bind del servizio in oggetti JSON sotto forma di stringa che vengono passati all'applicazione come variabile di ambiente, ad esempio `VCAP_APPLICATION` e `VCAP_SERVICES`.
* L'utilizzo di un servizio di supporto, come etcd, hashicorp Vault, Netflix Archaius o Spring Cloud Config, per memorizzare e richiamare gli attributi di configurazione specifici per l'ambiente è anch'esso un'opzione in qualsiasi ambiente.

Nella maggior parte dei casi, un'applicazione elabora una configurazione specifica per l'ambiente in fase di avvio. Il valore delle variabili di ambiente, ad esempio, non può essere modificato dopo l'avvio di un processo. Tuttavia, Kubernetes e i servizi di configurazione di supporto forniscono dei meccanismi per le applicazioni per rispondere in modo dinamico agli aggiornamenti della configurazione. Questa è una funzionalità facoltativa. Nel caso di processi temporanei e senza stato, il riavvio del servizio è spesso sufficiente.


Molti linguaggi e framework forniscono delle librerie standard per aiutare le applicazioni sia in configurazioni specifiche per l'applicazione che specifiche per l'ambiente in modo da consentirti di concentrarti sulla logica principale della tua applicazione e astrarre queste funzionalità fondamentali.

### Gestione delle credenziali del servizio
{: #portable-credentials}

La gestione della configurazione e delle credenziali del servizio (bind del servizio) varia tra le piattaforme. Cloud Foundry memorizza i dettagli di bind del servizio in un oggetto JSON sotto forma di stringa che viene passato all'applicazione come una variabile di ambiente `VCAP_SERVICES`. Kubernetes memorizza i bind di servizio come JSON sotto forma di stringa o attributi `ConfigMaps` o `Secrets` semplici, che possono essere passati all'applicazione inserita nel contenitore come variabili di ambiente o montati con un volume temporaneo. Nel caso di sviluppo locale, che ha una propria configurazione, l'esecuzione di test in locale è spesso una versione semplificata di qualsiasi cosa sia in esecuzione nel cloud. Lavorare tra queste variazioni in modo portatile senza avere percorsi di codice specifici per l'ambiente può essere difficile.

Negli ambienti Cloud Foundry e Kubernetes, puoi utilizzare i [broker di servizio](https://cloud.ibm.com/apidocs/ibm-cloud-osb-api){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") per gestire il bind a un servizio di supporto e inserire le credenziali associate nell'ambiente dell'applicazione. Ciò può influenzare la portatilità delle applicazioni poiché le credenziali potrebbero non essere fornite all'applicazione nello stesso modo in ambienti diversi.

{{site.data.keyword.IBM}} ha diverse librerie open source che operano con un file `mappings.json` per associare la chiave utilizzata dall'applicazione per richiamare le informazioni sulle credenziali a un elenco ordinato di possibili origini. Supporta tre tipi di pattern di ricerca:

* `cloudfoundry`: cerca un valore nella variabile di ambiente dei servizi di Cloud Foundry (VCAP_SERVICES).
* `env`: cerca un valore che è associato a una variabile di ambiente.
* `file`: cerca un valore in un file JSON.

Nel seguente file `mappings.json` di esempio, `cloudant-password` è la chiave utilizzata dal codice applicativo per cercare le credenziali di password. Una libreria specifica per il linguaggio esegue l'iterazione nell'array `searchPatterns` in un ordine specifico finché non viene trovata una corrispondenza.

```json
{
   "cloudant_password": {
      "searchPatterns": [
         "cloudfoundry:$['cloudant'][0].credentials.password",
         "env:cloudant_password",
         "file:/server/localdev-config.json:$.cloudant_password"
      ]
   }
}
```
{: codeblock}

La libreria ricerca la password cloudant nelle seguenti ubicazioni:

* Il percorso JSON `['cloudant'][0].credentials.password` nella variabile di ambiente di Cloud Foundry `VCAP_SERVICES`.
* Una variabile di ambiente non sensibile a maiuscole/minuscole denominata cloudant_password.
* Un campo JSON **cloudant_password** in un file **`localdev-config.json`** che viene mantenuto in un'ubicazione delle risorse specifica per il linguaggio.

Per ulteriori informazioni, vedi:

* [{{site.data.keyword.Bluemix}} Environment for Go](https://github.com/ibm-developer/ibm-cloud-env-golang){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
* [{{site.data.keyword.Bluemix_notm}} Environment for Node](https://github.com/ibm-developer/ibm-cloud-env){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
* [{{site.data.keyword.Bluemix_notm}} Service Bindings For Spring Boot](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")

## Valori di configurazione in Kubernetes
{: #config-kubernetes}

I Kubernetes forniscono alcuni modi diversi per definire le variabili di ambiente e assegnare i loro valori.

### Valori letterali
{: #config-literal}

Il modo più semplice per definire una variabile di ambiente è quello di includerla direttamente nel file `deployment.yaml` per il servizio. Nel seguente [esempio Kubernetes di base](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"), l'applicazione diretta funziona bene quando il valore è congruente tra gli ambienti.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"
```
{: codeblock}

### Variabili Helm
{: #config-helm}

Helm utilizza i template per creare i grafici in modo che i valori possano essere successivamente sostituiti. Puoi raggiungere lo stesso risultato dell'esempio precedente, con una maggiore flessibilità tra gli ambienti, utilizzando questo esempio nel template di file `mychart/templates/pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: {{ .Values.greeting }}
    - name: DEMO_FAREWELL
      value: {{ .Values.farewell }}
```
{: codeblock}

E il seguente esempio in un file `mychart/values.yaml`:

```yaml
greeting: "Hello from the environment"
farewell: "Such a sweet sorrow"
```
{: codeblock}

Il seguente output viene prodotto quando Helm riproduce il template:

```bash
$ helm template mychart
---
# Source: mychart/templates/pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: Hello from the environment
    - name: DEMO_FAREWELL
      value: Such a sweet sorrow
```
{:screen}

  C'è una piccola differenza tra questi due esempi. Nel primo esempio e nel file `values.yaml` di esempio, un essere umano ha aggiunto delle virgolette. Le virgolette non sono necessarie per le stringhe in YAML. Quando Helm riproduce il template, le virgolette vengono escluse.
  {: note}

### ConfigMap
{: #kubernetes-configmap}

Una ConfigMap è una risorsa Kubernetes univoca che definisce i dati come un insieme di coppie chiave-valore. Una ConfigMap per le variabili di ambiente mostrate negli esempi precedenti può avere un aspetto simile al seguente esempio:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: envar-demo-config
  labels:
    purpose: demonstrate-envars
data:
  DEMO_GREETING: Hello from the environment
  DEMO_FAREWELL: Such a sweet sorrow
```
{: codeblock}

La definizione di pod iniziale viene quindi modificata per utilizzare i valori dalla ConfigMap nel seguente modo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    envFrom:
    - configMapRef:
        name: envar-demo-config
```
{: codeblock}

La ConfigMap è ora una risorsa separata dal pod. Può avere un ciclo di vita differente. In questo caso, puoi aggiornare o modificare i valori nella ConfigMap senza dover ridistribuire il pod. Puoi anche aggiornare e manipolare una ConfigMap direttamente dalla riga di comando, cosa che può essere utile nel ciclo di sviluppo/test/debug.

In caso di utilizzo con Helm, puoi servirti delle variabili nella tua dichiarazione di ConfigMap. Queste variabili vengono risolte come al solito quando viene distribuito il grafico.

Per ulteriori informazioni, vedi la sezione relativa alle [ConfigMap di Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

### Credenziali e segreti
{: #kubernetes-secrets}

La configurazione viene generalmente fornita ai contenitori in esecuzione in Kubernetes mediante variabili di ambiente o ConfigMap. In entrambi i casi, i valori di configurazione possono essere rilevati in modo abbastanza rapido. Ecco perché Kubernetes utilizza i segreti per memorizzare le informazioni riservate.

I segreti sono oggetti indipendenti che contengono valori con codifica base64:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```
{: codeblock}

I segreti possono quindi essere utilizzati come file in un volume montato su uno o più dei contenitori del pod.

```yaml
containers:
- name: myservice
  image: myimage:latest
  volumeMounts:
  - name: foo
    mountPath: "/etc/foo"
    readOnly: true
volumes:
- name: foo
  secret:
    secretName: mysecret
```
{: codeblock}

In questo caso, per un montaggio come un volume in questo modo, ciascuna chiave nella mappa dei dati del segreto diventa un nome file sotto ai `mountPath`: `/etc/foo/username` e `/etc/foo/password` specificati.

I segreti possono anche essere utilizzati per definire le variabili di ambiente:

```yaml
containers:
- name: mypod
  image: myimage
  env:
  - name: SECRET_USERNAME
      valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
  - name: SECRET_PASSWORD
    valueFrom:
        secretKeyRef:
          name: mysecret
          key: password
```
{: codeblock}

Kubernetes esegue la codifica base64 per tuo conto. Il contenitore in esecuzione nel pod riconosce il valore con codifica base64 quando richiama la variabile di ambiente.

Come con le ConfigMap, i segreti possono essere creati e manipolati dalla riga di comando, cosa che torna utile quando si ha a che fare con certificati SSL.

Per ulteriori informazioni, vedi il documento relativo ai [segreti Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

<!-- SSL EXAMPLE -->
