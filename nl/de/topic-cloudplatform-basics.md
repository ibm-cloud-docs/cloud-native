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

# Cloudplattformkonzepte
{: #platform}

Hier finden Sie eine kurze Übersicht über die wichtigsten Technologien und Konzepte, mit denen Entwickler bei der Erstellung von cloudnativen Anwendungen interagieren, beginnend mit Containern, Kubernetes, Helm und Istio.
{:shortdesc}

## Container
{: #containers}

Container sind ein Standardmechanismus zum Packen einer Anwendung mit allen zugehörigen Abhängigkeiten in eine einzelne, eigenständige Einheit. Container lösen das Portierbarkeitsproblem. Das Containerartefakt (Image) stellt sicher, dass alle Elemente, die eine Anwendung für die Ausführung benötigt, am richtigen Ort sind. Container-Engines können sich auf die Ausführung von Containern als isolierte Prozesse in einer effizienten, sicheren und geschützten Weise konzentrieren.

Container-Images werden in der Regel aus einer Liste von Anweisungen erstellt, die in einer `Dockerfile` definiert sind. Container-Images werden fast immer aus anderen Container-Images erstellt (sie sind im Wesentlichen eine Fortsetzung der Anweisungen aus dem bekannten vorherigen Status). Sie können das folgende Snippet verwenden, um ein eigenes Open Liberty-Image zu erstellen. Beispiel:

```yaml
FROM open-liberty:kernel
COPY server.xml /config/
```
{: codeblock}

Nachdem ein Image erstellt wurde, kann es ausgeführt werden. Engines für die Containerausführung wie Docker oder [Container](https://containerd.io/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") verwenden diese Imagedefinition und führen den definierten Einstiegspunkt als ressourcenisolierten Prozess direkt auf dem Hostbetriebssystem aus. Dadurch entfällt der Systemaufwand für virtuelle Maschinen.

Container-Images werden in *Registrys* gespeichert. Die bekannteste Registry ist die öffentliche Docker-Hub-Registry. Images werden in oder aus Container-Registrys mit Zugriffssteuerung wie {{site.data.keyword.registryshort_notm}} übertragen, die enger mit der Infrastruktur und den CI/CD-Pipelines verknüpft sind.

## Kubernetes
{: #kubernetes}

IBM Cloud-Plattformen verwenden Kubernetes für die Orchestrierung von Containern. Daher ist es wichtig, dass Entwickler nicht nur die Grundlagen der Containertechnologie beherrschen, sondern auch mit den Grundlagen von Kubernetes einschließlich der wichtigsten Befehle und Bereitstellungsartefakte vertraut sind. In der folgenden Tabelle sind einige wichtige Kubernetes-Konzepte aufgeführt:

| Konzept | Beschreibung |
|---------|-------------|
| Pod | Eine lokalisierte Gruppe von Containern, die zusammen als Einheit bereitgestellt werden. Pods sind relativ unveränderlich, sodass der ursprüngliche Pod ersetzt werden muss, um Änderungen an verschiedenen Attributen des Pod vornehmen zu können. Eine typische Anwendung verfügt über einen Container mit der zentralen Geschäftslogik und zusätzliche Pods, die Plattformfunktionen auf der differenzierten Ebene bereitstellen. |
| Deployment (Bereitstellung) | Eine wiederholt anwendbare Vorlage für einen statusunabhängigen Pod, die dem Konzept des Pod eine Dimension für die Skalierung hinzufügt. Darüber hinaus kann die Definition in der Vorlage aktualisiert und die zugrunde liegenden Podinstanzen können ersetzt werden. Eine Kubernetes-Bereitstellungskonfiguration wird von einem Kubernetes Deployment Controller überwacht, um sicherzustellen, dass die deklarierte Anzahl von Pods für eine Bereitstellung eingehalten wird. Eine Bereitstellung wird als `kind: Deployment` in `.yaml`-Dateien angezeigt. |
| Service | Ein bekannter Name, der eine Gruppe von relativ instabilen Pod-IP-Adressen darstellt. Ein Service kann nur im privaten Netz des Clusters vorhanden oder extern über eine für den Cloud-Provider spezifische Lastausgleichsfunktion zugänglich sein. Ein Service wird als `kind: Service` in `.yaml`-Dateien angezeigt. |
| Ingress | Die Möglichkeit, eine einzelne Netzadresse mit mehreren Services über ein virtuelles Hosting oder ein kontextbasiertes Routing gemeinsam zu nutzen. Ein Ingress kann auch Verwaltungsaktivitäten für die Netzverbindung wie die TLS-Beendigung ausführen. Ein Ingress wird als `kind: Ingress` in `.yaml`-Dateien angezeigt. |
| Secret (geheimer Schlüssel) | Ein Objekt, das sensible Informationen für die Verwendung zur Podlaufzeit speichert und die bereitstellungsspezifischen Informationen von dem Container-Image oder der Orchestrierung trennt. Ein Secret kann einem Pod während der Laufzeit entweder über Umgebungsvariablen oder über virtuelle Dateisystemmounts zugänglich gemacht werden. Ohne Secrets werden sensible Daten entweder im Container-Image oder in der Orchestrierung gespeichert, die beide mehr Raum für Sicherheitslücken oder unbeabsichtigten Zugriff schaffen. |
| ConfigMap | Trennt ebenso wie Secrets bereitstellungsspezifische Informationen von der Containerorchestrierung. Eine ConfigMap ist jedoch eine allgemeine Konfigurationsstruktur. Sie wird verwendet, um Informationen wie Befehlszeilenargumente, Umgebungsvariablen und andere Konfigurationsartefakte zur Laufzeit an die Container und Systemkomponenten des Pod zu binden. | 
{: caption="Tabelle 1. Kubernetes-Konzepte" caption-side="bottom"}

Alle Ressourcen werden im Kubernetes-Ressourcenmodell definiert. Dieses kann entweder über die REST-konforme API oder über Konfigurationsdateien konfiguriert werden, die über die `kubectl`-Befehlszeile übergeben werden.

Weitere Informationen finden Sie auf der Website mit den [grundlegenden Informationen zu Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link"), auf der Website mit den Informationen zum [Kubernetes-Objektmodell](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") und auf der Website zur [Befehlszeile `kubectl`](https://kubernetes.io/docs/reference/kubectl/overview/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link"). 

## Helm
{: #helm}

Helm ist ein Paketmanager, mit dem für Kubernetes erstellte Software auf einfache Weise gesucht, gemeinsam genutzt und verwendet werden kann. Helm bietet auch eine Lösung für eine häufige Benutzeranforderung: die Bereitstellung derselben Anwendung für mehrere Umgebungen. In Helm werden *Diagramme* (Charts) verwendet; Sammlungen von Vorlagen, die zur Installationszeit gültige Kubernetes-Objekte (YAML) erzeugen. Diese Diagramme werden aus einer Vorlagensprache erstellt, die Unterstützung für Variablen, Bereichsoperationen und andere Mechanismen enthält, die die Verwaltung von Metadaten für Kubernetes-Bereitstellungen erheblich vereinfachen.

Weitere Informationen finden Sie in [Helm](https://helm.sh/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

## Service-Mesh Istio
{: #istio}

Istio ist eine Open-Source-Plattform für die Verwaltung und den Schutz von Mikroservices. Sie arbeitet mit Orchestrierungssystemen wie Kubernetes zusammen und ermöglicht die Verwaltung und Steuerung der Kommunikation zwischen Services.

In Istio wird ein sogenanntes Sidecar-Modell verwendet. Ein Sidecar (ein Envoy-Proxy) ist ein separater Prozess, der neben der Anwendung besteht. Das Sidecar verwaltet die gesamte ein- und ausgehende Kommunikation des Service und wendet eine allgemeine Funktionalitätsstufe auf alle Services an, unabhängig von der Programmiersprache oder dem Framework, mit der bzw. dem der Service erstellt wurde. Praktisch bietet Istio einen Mechanismus für die zentrale Konfiguration von Routing- und Sicherheitsrichtlinien und die dezentrale Anwendung dieser Richtlinien über Sidecars.

Anstelle von ähnlichen Funktionen, die von einzelnen Programmiersprachen oder Frameworks bereitgestellt werden, wird die Verwendung der von Istio bereitgestellten Funktionen empfohlen. Richtlinien für den Lastausgleich und andere Routing-Richtlinien beispielsweise werden jedoch durch die Infrastruktur in konsistenterer Weise definiert, verwaltet und umgesetzt.

In einigen Fällen, z. B. beim dezentralen Tracing, ergänzen sich Istio und Bibliotheken auf der Anwendungsebene. Sie können die Abläufe verbessern, wenn Sie beides verwenden. Beim dezentralen Tracing kann Istio lediglich sicherstellen, dass Trace-Header vorhanden sind; Anwendungsbibliotheken stellen den wichtigen Kontext zu den Beziehungen zwischen Anforderungen zur Verfügung. Ihr Verständnis des Systems als Ganzes verbessert sich, wenn Istio und die unterstützenden Bibliotheken oder Frameworkbibliotheken gemeinsam verwendet werden.

Auf der höchsten Ebene erweitert Istio die Kubernetes-Plattform und stellt weitere Managementkonzepte, Transparenz und Sicherheit bereit. Die Funktionen von Istio können in die folgenden vier Kategorien eingeteilt werden:

* Datenverkehrsmanagement: Steuerung des Datenverkehrs zwischen den Mikroservices, um Datenverkehrssplitting, Fehlerbehebung und Canary-Releases auszuführen.
* Sicherheit: Bereitstellung einer starken, identitätsbasierten Authentifizierung, Autorisierung und Verschlüsselung zwischen den Mikroservices.
* Beobachtbarkeit: Erfassung von Metriken und Protokollen für eine höhere Transparenz der Anwendungen, die im Cluster ausgeführt werden.
* Richtlinien: Erzwingung von Zugriffsbeschränkungen, Durchsatzbegrenzungen und Kontingenten, um die Anwendungen zu schützen.

Weitere Informationen finden Sie in [What is Istio?](https://istio.io/docs/concepts/what-is-istio/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").



