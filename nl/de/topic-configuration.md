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

# Konfiguration
{: #configuration}

Cloudnative Anwendungen müssen portierbar sein. Sie können dasselbe feste Artefakt für die Bereitstellung in mehreren Umgebungen verwenden, ohne Code zu ändern oder anderweitig nicht getestete Codepfade zu nutzen.
{:shortdesc}

Drei Faktoren aus der [Zwölf-Faktor-Methodik](https://12factor.net/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") beziehen sich direkt auf diesen Aspekt:

* Der erste Faktor empfiehlt eine 1-zu-1-Korrelation zwischen einem aktiven Service und einer versionsgesteuerten Codebasis. Erstellen Sie ein festes Bereitstellungsartefakt, wie z. B. ein Docker-Image, aus einer versionierten Codebasis, das unverändert in mehreren Umgebungen bereitgestellt werden kann.
* Der dritte Faktor empfiehlt eine Trennung zwischen der anwendungsspezifischen Konfiguration, die Teil des festen Artefakts ist, und eine umgebungsspezifische Konfiguration, die dem Service zur Bereitstellungszeit zur Verfügung gestellt wird.
* Der zehnte Faktor empfiehlt, dass alle Umgebungen so ähnlich wie möglich sein sollten. Umgebungsspezifische Codepfade sind schwierig zu testen und erhöhen das Risiko von Fehlern bei der Bereitstellung in unterschiedlichen Umgebungen. Dies gilt auch für Unterstützungsservices. Wenn Sie die Entwicklung und den Test mit einer speicherinternen Datenbank ausführen, können in Test-, Staging- oder Produktionsumgebungen unerwartete Fehler auftreten, da in diesen Umgebungen eine andere Datenbank verwendet wird, die sich anders verhält.

## Konfigurationsquellen
{: #config-inject}

Die anwendungsspezifische Konfiguration ist Teil des festen Artefakts. In unter WebSphere Liberty ausgeführten Anwendungen wird beispielsweise eine Liste der installierten Funktionen definiert, die die in der Laufzeit aktiven Binärdateien und Services steuern. Diese Konfiguration ist für die Anwendung spezifisch und ist im Docker-Image enthalten. In Docker-Images ist auch der empfangsbereite oder zugängliche Port definiert, da die Ausführungsumgebung beim Starten des Containers die Portzuordnung ausführt. 

Die umgebungsspezifische Konfiguration (z. B. der Host und der Port, der für die Kommunikation mit anderen Services oder Datenbankbenutzern verwendet wird, oder Einschränkungen für die Ressourcennutzung) wird dem Container von der Bereitstellungsumgebung zur Verfügung gestellt. Die Verwaltung der Servicekonfiguration und der Berechtigungsnachweise kann erheblich variieren:

* Kubernetes speichert Konfigurationswerte (in Zeichenfolgen konvertierte JSON- oder unstrukturierte Attribute) entweder in ConfigMaps oder in Secrets. Diese können als Umgebungsvariablen oder virtuelle Dateisystemmounts an die containerisierte Anwendung übergeben werden. Das von einem Service verwendete Verfahren wird in den Bereitstellungsmetadaten angegeben, entweder in einer Kubernetes-YAML-Datei oder im Helm-Diagramm.
* Lokale Entwicklungsumgebungen sind häufig vereinfachte Varianten, die einfache Schlüssel/Wert-Umgebungsvariablen verwenden.
* Cloud Foundry speichert Konfigurationsattribute und Details zur Servicebindung in JSON-Objekten, die in Zeichenfolgen konvertiert wurden und als Umgebungsvariablen an die Anwendung übergeben werden, z. B. `VCAP_APPLICATION` und `VCAP_SERVICES`.
* Die Verwendung eines Unterstützungsservice wie etcd, hashicorp Vault, Netflix Archaius oder Spring Cloud config zum Speichern und Abrufen von umgebungsspezifischen Konfigurationsattributen ist ebenfalls in jeder Umgebung möglich.

In den meisten Fällen verarbeitet eine Anwendung eine umgebungsspezifische Konfiguration beim Start. Der Wert von Umgebungsvariablen kann beispielsweise nicht geändert werden, nachdem ein Prozess gestartet wurde. Kubernetes und unterstützende Konfigurationsservices stellen jedoch Mechanismen zur Verfügung, die Anwendungen die dynamische Reaktion auf Konfigurationsaktualisierungen ermöglichen. Dies ist eine optionale Funktion. Bei statusunabhängigen, transienten Prozessen ist der Neustart des Service häufig ausreichend.

Viele Sprachen und Frameworks stellen Standardbibliotheken zur Verfügung, um Anwendungen in anwendungsspezifischen und umgebungsspezifischen Konfigurationen zu unterstützen, sodass Sie sich auf die zentrale Logik Ihrer Anwendung konzentrieren und diese grundlegenden Funktionen abstrahieren können.

### Mit Serviceberechtigungsnachweisen arbeiten
{: #portable-credentials}

Die Verwaltung der Servicekonfiguration und der Berechtigungsnachweise (Servicebindungen) variiert zwischen den Plattformen. Cloud Foundry speichert Details zur Servicebindung in einem JSON-Objekt, das in eine Zeichenfolge konvertiert wurde und als Umgebungsvariable `VCAP_SERVICES` an die Anwendung übergeben wird. Kubernetes speichert Servicebindungen als in Zeichenfolgen konvertierte JSON-Attribute oder unstrukturierte `ConfigMaps`- oder `Secrets`-Attribute, die als Umgebungsvariablen an die containerisierte Anwendung übergeben oder als temporärer Datenträger angehängt werden können. Im Fall der lokalen Entwicklung mit eigener Konfiguration weisen die lokalen Tests häufig eine vereinfachte Version der jeweiligen Umgebung auf, die in der Cloud ausgeführt wird. Die Arbeit mit diesen verschiedenen Varianten unter Sicherstellung der Portierbarkeit und ohne dass umgebungsspezifische Codepfade verwendet werden, kann eine große Herausforderung sein.

In Cloud Foundry- und Kubernetes-Umgebungen können Sie [Service-Broker](https://cloud.ibm.com/apidocs/ibm-cloud-osb-api){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") verwenden, um die Bindung an einen Unterstützungsservice zu verwalten und die zugehörigen Berechtigungsnachweise in die Umgebung der Anwendung einzufügen. Dies kann die Portierbarkeit der Anwendung beeinträchtigen, da die Berechtigungsnachweise in verschiedenen Umgebungen möglicherweise nicht auf dieselbe Art und Weise bereitgestellt werden.

{{site.data.keyword.IBM}} verfügt über mehrere Open-Source-Bibliotheken, die mit einer Datei `mappings.json` zusammenarbeiten, um den Schlüssel zuzuordnen, den die Anwendung zum Abrufen von Berechtigungsnachweisinformationen in eine geordnete Liste der möglichen Quellen verwendet. Drei Suchmustertypen werden unterstützt:

* `cloudfoundry`: Nach einem Wert in der Cloud Foundry-Umgebungsvariable für Services (VCAP_SERVICES) suchen.
* `env`: Nach einem Wert suchen, der einer Umgebungsvariable zugeordnet ist.
* `file`: Nach einem Wert in einer JSON-Datei suchen.

In der folgenden Beispieldatei `mappings.json` ist `cloudant-password` der Schlüssel, den der Anwendungscode verwendet, um den Kennwortberechtigungsnachweis zu suchen. Eine sprachspezifische Bibliothek durchläuft das Array `searchPatterns` in einer bestimmten Reihenfolge, bis eine Übereinstimmung gefunden wird.

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

Die Bibliothek durchsucht die folgenden Positionen nach dem Cloudant-Kennwort:

* Den JSON-Pfad `['cloudant'][0].credentials.password` in der Cloud Foundry-Umgebungsvariable `VCAP_SERVICES`.
* Eine von der Groß-/Kleinschreibung unabhängige Umgebungsvariable mit dem Namen 'cloudant_password'.
* Das JSON-Feld **cloudant_password** in einer Datei **`localdev-config.json`**, die an einer sprachspezifischen Ressourcenposition gespeichert ist.

Weitere Informationen siehe:

* [{{site.data.keyword.Bluemix}}-Umgebung für Go](https://github.com/ibm-developer/ibm-cloud-env-golang){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
* [{{site.data.keyword.Bluemix_notm}}-Umgebung für Node](https://github.com/ibm-developer/ibm-cloud-env){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
* [{{site.data.keyword.Bluemix_notm}}-Servicebindungen für Spring Boot](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")

## Konfigurationswerte in Kubernetes
{: #config-kubernetes}

Kubernetes stellt mehrere verschiedene Möglichkeiten zur Verfügung, um Umgebungsvariablen zu definieren und ihnen Werte zuzuordnen.

### Literalwerte
{: #config-literal}

Das einfachste Verfahren für die Definition einer Umgebungsvariable ist ihre direkte Angabe in der Datei `deployment.yaml` für den Service. Im folgenden [grundlegenden Kubernetes-Beispiel](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") funktioniert die direkte Angabe sehr gut, wenn der Wert in allen Umgebungen konsistent ist:

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

### Helm-Variablen
{: #config-helm}

Helm verwendet Vorlagen, um Diagramme zu erstellen, sodass Werte später ersetzt werden können. Sie erzielen dasselbe Ergebnis wie beim vorherigen Beispiel mit mehr Flexibilität über die Umgebungen hinweg, indem Sie das folgende Beispiel in der Dateivorlage `mychart/templates/pod.yaml` verwenden:

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

Verwenden Sie außerdem das folgende Beispiel in einer Datei `mychart/values.yaml`:

```yaml
greeting: "Hello from the environment"
farewell: "Such a sweet sorrow"
```
{: codeblock}

Die folgende Ausgabe wird erzeugt, wenn Helm die Vorlage wiedergibt:

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

  Es gibt einen kleinen Unterschied zwischen diesen beiden Beispielen. Im ersten Beispiel und in der Beispieldatei `values.yaml` wurden von einem Menschen Anführungszeichen hinzugefügt. Anführungszeichen sind für Zeichenfolgen in YAML nicht erforderlich. Wenn Helm die Vorlage wiedergibt, werden die Anführungszeichen weggelassen.
  {: note}

### ConfigMap
{: #kubernetes-configmap}

Eine ConfigMap ist ein eindeutiges Kubernetes-Artefakt, das Daten als Gruppe von Schlüssel/Wert-Paaren definiert. Eine ConfigMap für die in den vorangegangenen Beispielen gezeigten Umgebungsvariablen kann wie im folgenden Beispiel aussehen:

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

Anschließend wird die ursprüngliche Poddefinition wie folgt geändert, damit Werte aus der ConfigMap verwendet werden:

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

Die ConfigMap ist jetzt ein vom Pod getrenntes Artefakt. Sie kann einen anderen Lebenszyklus haben. Sie können die Werte in der ConfigMap aktualisieren oder ändern, ohne dass in diesem Fall der Pod erneut bereitgestellt werden muss. Sie können eine ConfigMap auch direkt über die Befehlszeile aktualisieren und bearbeiten. Dies kann im Entwicklungs-/Test-/Debug-Zyklus sehr nützlich sein.

Bei der Verwendung mit Helm können Sie Variablen in Ihrer ConfigMap-Deklaration nutzen. Diese Variablen werden wie üblich aufgelöst, wenn das Diagramm bereitgestellt wird.

Weitere Informationen finden Sie in [Kubernetes ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

### Berechtigungsnachweise und Secrets
{: #kubernetes-secrets}

Die Konfiguration wird den in Kubernetes ausgeführten Containern im Allgemeinen über Umgebungsvariablen oder ConfigMaps zur Verfügung gestellt. In jedem Fall können Konfigurationswerte relativ schnell erkannt werden. Aus diesem Grund verwendet Kubernetes Secrets, um sensible Informationen zu speichern.

Secrets sind unabhängige Objekte, die Base64-codierte Werte enthalten:

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

Anschließend können Secrets als Dateien auf einem Datenträger verwendet werden, der in mindestens einem der Container des Pod angehängt wurde:

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

Werden die Secrets auf diese Weise als Datenträger angehängt, wird jeder Schlüssel in der Datenmap des Secret zu einem Dateinamen unter dem angegebenen `mountPath` (Mountpfad): in diesem Fall `/etc/foo/username` und `/etc/foo/password`.

Secrets können auch zum Definieren von Umgebungsvariablen verwendet werden:

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

Kubernetes führt die Base64-Decodierung für Sie aus. Der im Pod ausgeführte Container erkennt beim Abrufen der Umgebungsvariable den dekodierten Base64-Wert.

Wie ConfigMaps können Secrets über die Befehlszeile erstellt und bearbeitet werden. Dies ist bei der Verarbeitung von SSL-Zertifikaten sehr praktisch.

Weitere Informationen finden Sie in [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

<!-- SSL EXAMPLE -->
