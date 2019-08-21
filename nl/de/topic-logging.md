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

# Protokollierung
{: #logging}

Protokollnachrichten sind Zeichenfolgen mit Kontextinformationen zu Status und Aktivität eines Mikroservice zu dem Zeitpunkt, zu dem der Protokolleintrag erstellt wird. Protokolle sind erforderlich, um zu diagnostizieren, wie und warum Services fehlschlagen. Sie unterstützen die Metriken, die bei der Überwachung des Zustands von Anwendungen erfasst werden.
{:shortdesc}

Stellen Sie sicher, dass Protokolleinträge direkt in die standardmäßigen Ausgabe- und Fehlerdatenströme geschrieben werden. Dies bewirkt, dass Protokolleinträge mit Befehlszeilentools angezeigt werden können und dass auf der Infrastrukturebene konfigurierte Protokollweiterleitungsservices die Protokollerfassung und das Datenmanagement ausführen können.

Die Behandlung von Protokolldateien ist schwieriger, wenn eine containerisierte Anwendung nicht so konfiguriert werden kann, dass sie Protokolle in den Standardausgabe- oder Standardfehlerdatenstrom schreibt.

* Eine Option ist die Verwendung eines Datenträgers für Protokolldaten. Dies kann ein einfacher Bindemount für die lokale Entwicklung und den lokalen Test oder ein ordnungsgemäßer persistenter Datenträger als Bestandteil einer Kubernetes-Bereitstellung sein. Ein [dediziertes Sidecar oder ein Protokollierungsagent](https://kubernetes.io/docs/concepts/cluster-administration/logging/#sidecar-container-with-a-logging-agent){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") kann Daten von einem gemeinsam genutzten Datenträger lesen, um Protokolle an einen zentralen Aggregator weiterzuleiten. Die Protokollrotation muss explizit konfiguriert werden, um die Menge der Protokolldaten zu steuern, die auf Datenträgern gespeichert werden.
* Eine andere Option ist die Verwendung von Anwendungsbibliotheken oder Agenten, um Protokolle direkt an Aggregatoren weiterzuleiten. Die Konfiguration dieser Option kann über verschiedene Bereitstellungsumgebungen hinweg komplex sein.

## JSON-Protokollierung
{: #json-logging}

Wenn Ihre Anwendung sich mit der Zeit weiterentwickelt, können die protokollierten Daten sich ändern. Die Verwendung eines JSON-Protokollformats bietet die folgenden Vorteile:

* Protokolle sind indexierbar, wodurch die Suche in den aggregierten Protokollen wesentlich vereinfacht wird.
* Die Protokolle tolerieren Änderungen, da das Parsing nicht von der Position der Elemente in einer Zeichenfolge abhängig ist.

Während JSON-formatierte Protokolle für Maschinen einfacher zu parsen sind, sind sie für Menschen schwerer zu lesen. Ziehen Sie die Verwendung von Umgebungsvariablen in Betracht, um das Protokollformat zwischen einfachem Text für die lokale Entwicklung und Fehlerbehebung und JSON-formatierten Protokollen für die längerfristige Speicherung und Aggregation umzuschalten.

Mit JSON-Parsern für die Befehlszeile wie das Tool JSON Query (jq) können lesbare Ansichten aus JSON-formatierten Protokollen erstellt werden. Im folgenden Beispiel werden die Protokolle mit grep über eine Pipe geleitet, um sicherzustellen, dass das Nachrichtenfeld vorhanden ist, bevor jq die Zeile parst:

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | \
  grep message | \
  jq .message -r
```
{: pre}

## Protokolle mithilfe von `kubectl` anzeigen
{: #view-logs-kube}

An den standardmäßigen Ausgabe- und Fehlerdatenstrom gesendete Protokolle können in Kubernetes über die Konsole oder mit `kubectl`-Befehlen im folgenden Format angezeigt werden: `kubectl logs <podname>`.

Wenn Sie einen angepassten Namensbereich verwenden, z. B. stock-trader, schließen Sie diesen in den Befehl ein, z. B. `kubectl logs -n stock-trader <podname>`.

Sind für jeden Pod mehrere Container vorhanden (wie beispielsweise bei Istio-Sidecars), müssen Sie auch den Container angeben. Im folgenden Beispiel wird der Namensbereich 'stock-trader' zum Anzeigen von Protokollen aus dem Container `portfolio` im Pod `portfolio-54b4d579f7-4zvzk` verwendet:

```bash
kubectl logs -n stock-trader portfolio-54b4d579f7-4zvzk -c portfolio
```
{: pre}

Für Protokolle im JSON-Format können Sie mit `jq` ein Nachrichtenfeld extrahieren. Beispiel:

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | grep message | jq .message -r
```
{: pre}

Protokolleinträge werden mit `grep` über eine Pipe geleitet, um sicherzustellen, dass `jq` Zeilen parst, die ein Nachrichtenfeld enthalten.
{: note}
