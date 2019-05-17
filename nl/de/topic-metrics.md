---

copyright:
  years: 2019
lastupdated: "2019-04-30"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Metriken
{: #metrics}

Metriken sind einfache numerische Messungen, die als Schlüssel/Wert-Paare erfasst werden. Einige Metriken erhöhen Zähler, andere führen Aggregationen aus, wie die Summe aller Werte, die in der letzten Minute erfasst wurden, oder die durchschnittliche abgelaufene Zeit in der letzten Minute. Einige Metriken sind nur einfache Messanzeigen, die den letzten beobachteten Wert zurückgeben. Die Erfassung und Verarbeitung von Metriken kann Ihnen helfen, auf potenzielle Probleme zu reagieren, bevor sie zu Fehlern werden und größere Probleme verursachen.
{:shortdesc}

Bei Metriken in einem verteilten System gibt es drei allgemeine Faktoren: Produzenten, Aggregatoren und Verarbeiter. Bestimmte Kombinationen aus diesen Faktoren sind relativ gängig, z. B. die Verwendung von Prometheus als Aggregator zusammen mit Grafana für die Verarbeitung der erfassten Metriken und die Anzeige in grafischen Dashboards oder die Verwendung von StatsD zusammen mit Graphite.

![Die drei Faktoren bei Metriken in verteilten Systemen](images/metrics-systems.png "Die drei Faktoren bei Metriken in verteilten Systemen")

Der Produzent ist natürlich die Anwendung selbst. In einigen Fällen ist die Anwendung direkt an der Erstellung von Metriken beteiligt. In anderen Fällen können Agenten oder andere Infrastrukturen die Anwendung entweder passiv beobachten oder aktiv instrumentieren, um Metriken für sie zu erzeugen. Die nachfolgenden Vorgänge sind vom Aggregator abhängig.

Metriken werden vom Produzenten über Push- oder Pull-Mechanismen an den Aggregator übertragen. Einige Aggregatoren wie StatsD erwarten, dass die Anwendung (oder stattdessen ein Agent) eine Verbindung zum Aggregator herstellt, um Daten zu übertragen. Dies erfordert, dass die Verbindungsinformationen für den Aggregator an alle Anwendungsprozesse verteilt werden, die gemessen werden sollen. Andere Aggregatoren wie Prometheus stellen in regelmäßigen Abständen eine Verbindung zu einem bekannten Endpunkt her, um Metrikdaten (eventuell per Scraping) zu sammeln. Dies setzt voraus, dass der Produzent einen Endpunkt für das Scraping definiert und bereitstellt und dass dem Aggregator mitgeteilt wird, wo die Endpunkte sich befinden. Bei der Verwendung mit Kubernetes kann Prometheus Endpunkte auf der Basis von Serviceannotationen erkennen.

Der Verarbeiter schließlich nutzt alle aggregierten Daten. Wie bereits erwähnt, verarbeiten Services wie Grafana aggregierte Metriken für die Visualisierung mithilfe von Dashboards. Grafana unterstützt auch Alerts auf der Basis von Regeln, die unabhängig vom Dashboard gespeichert und ausgewertet werden.

## Automatische Erkennung von Prometheus-Endpunkten in Kubernetes
{: #prometheus-kubernetes}

Das Pull-basierte Modell hat sein eigenes Ökosystem geschaffen. Andere Aggregatoren wie Sysdig können ebenfalls Metrikdaten von Prometheus-Endpunkten per Scraping erfassen. Dies kann bedeuten, dass in einigen Systemen Prometheus-Metriken ohne den Prometheus-Server verwendet werden.

In Kubernetes-Umgebungen werden Annotationen für die Erkennung von Prometheus-Endpunkten verwendet. Beispiel: Ein Service, der den Endpunkt `/metrics` über HTTP an Port 8080 bereitstellt, fügt der Servicedefinition die folgenden Annotationen hinzu:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ...
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /metrics
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

Jeder Prometheus-kompatible Aggregator kann dann diese Endpunkte mithilfe der Kubernetes-API erkennen, indem er nach der Annotation filtert.

## Anwendungs- oder Plattformmetriken?
{: #app-platform-metrics}

Die Erfassung von Metriken muss sorgfältig durchdacht werden. Welche Datenpunkte sollten erfasst werden? Bei cloudnativen Anwendungen können die Metriken grob in zwei Kategorien eingeteilt werden, Anwendungsmetriken und Plattformmetriken:

* Anwendungsmetriken konzentrieren sich auf Entitäten in der Anwendungsdomäne. Beispiele: Wie viele Anmeldungen wurden in den letzten fünf Minuten ausgeführt? Wie viele Benutzer sind momentan angemeldet? Wie viele Handelsaktivitäten wurden in der letzten Sekunde ausgeführt? Für die Erfassung von anwendungsspezifischen Metriken ist angepasster Code erforderlich, um die Informationen zu erfassen und zu veröffentlichen. Die meisten Sprachen haben Metrikbibliotheken, um das Hinzufügen von benutzerdefinierten Metriken zu vereinfachen.
* Plattformmetriken sind hingegen Entitäten in der Hosting-Domäne. Beispiele: Wie lange dauert es, einen Service aufzurufen? Wie lange dauert diese Datenbankabfrage? Sie sind am Fluss des Datenverkehrs und der Arbeit durch das System ausgerichtet und können oft ohne Änderungen an der Anwendung gemessen werden. Einige Anwendungsframeworks bieten auch integrierte Unterstützung für die Messung dieser Aspekte, z. B. der Abfrageausführungszeiten.

Diese Kategorien alleine reichen nicht aus, um zu entscheiden, was gemessen werden muss. Vergessen Sie nicht, dass in einem verteilten Systemen viele Elemente vorhanden sind, die Metriken erzeugen. Mit einigen bekannten Methoden wird versucht, den riesigen Pool an Metriken auf wenige wichtige Metriken einzuschränken, die überwacht werden sollten:

* Die [vier goldenen Signale](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/#xref_monitoring_golden-signals){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") sind die essenziellen Metriken, die das Google-Team für Site Reliability Engineering (SRE) für die Beobachtbarkeit des Service-Levels bei der Überwachung eines verteilten Systems identifiziert hat. Das Team drückt es wie folgt aus: "Wenn nur vier Metriken Ihres Benutzersystems gemessen werden können, konzentrieren Sie sich auf diese vier." Die vier Metriken sind Latenz, Datenverkehr, Fehler und Sättigung.
* Die [USE-Methode (Utilization, Saturation and Errors)](http://www.brendangregg.com/usemethod.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") ist als Checkliste für den Notfall konzipiert, um die Leistung eines Systems zu analysieren. Die USE-Methode lässt sich in einem Satz zusammenfassen: "Überprüfen Sie für jede Ressource die Auslastung, die Sättigung und die Fehler." Eine Ressource ist in diesem Fall eine physische oder logische Ressource mit festen Grenzwerten wie CPUs oder Platten. Für jede endliche Ressource messen Sie die Auslastung, die Sättigung und die Fehler. 
* Die [RED-Methode](https://thenewstack.io/monitoring-microservices-red-method/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") wurde von den vier goldenen Signalen abgeleitet. Ihr Name setzt sich aus den Anfangsbuchstaben der drei wichtigen Metriken zusammen, die Sie für jeden Mikroservice in Ihrer Architektur messen sollten. Diese Methode konzentriert sich stark auf die Anforderungen, insbesondere im Vergleich zur USE-Methode, die gut zum Design und zur Architektur vieler cloudnativer Anwendungen passt. Die Metriken sind Rate (R), Fehler (Errors - E) und Dauer (D). 

Alle diese Methoden weisen einige Gemeinsamkeiten auf; beispielsweise verfolgen alle die Fehlerrate. Aber es gibt auch erhebliche Unterschiede. Die USE-Methode konzentriert sich auf die Infrastrukturmetriken (Auslastung/Sättigung). Cloudnative Umgebungen sind so konzipiert, dass sie die physische (oder virtuelle) Hardware besser nutzen können. Mit diesen Infrastrukturmessungen wird überprüft, ob Ihr System die Arbeitslast ordnungsgemäß verarbeitet. Die RED-Methode konzentriert sich ausschließlich auf Anforderungsmetriken (Latenz/Dauer, Datenverkehr/Rate). Diese können auf Probleme mit der Infrastruktur (Netzprobleme) oder mit den Anwendungen (falsche Konfigurationen, Deadlocks, wiederholte Servicefehler usw.) hinweisen, insbesondere wenn sie mit beobachteten Fehlerraten kombiniert werden. Die goldenen Signale umfassen beide Bereiche und vereinen Infrastruktur- und Anforderungsmetriken in einer ganzheitlichen Sicht.

## Metriken definieren
{: #defining-metrics}

Mit der Überwachung von Metriken, die für einen einzigen Service erfasst werden, erhalten Sie Informationen über die Ressourcennutzung dieses Service. Sind jedoch (aufgrund der horizontalen Skalierung oder durch Bereitstellungen für mehrere Regionen) mehrere Instanzen des Service vorhanden, müssen Sie diese voneinander unterscheiden können, um bei der Masse ähnlicher Daten, die ankommen, Probleme einzugrenzen. Dieses Problem löst die Namensgebung. In einigen Fällen gibt das von Ihnen verwendete Metriksystem eine bestimmte Struktur für Ihre Metriken vor. Prometheus empfiehlt beispielsweise eine Struktur wie `namespace_subsystem_name`, andere empfehlen `namespace.subsystem.targetNoun.actioned`.

Wenn Sie z. B. die Anzahl der Handelsaktivitäten verfolgen möchten, die eine Börsenhandelsanwendung ausgeführt hat, können Sie diese in einer Eigenschaft mit dem Namen `stock.trades` erfassen. Um zwischen den Instanzen zu unterscheiden, können Sie der Eigenschaft die Instanz-ID voranstellen: `<instanceid>.stock.trades`. Dies unterstützt die Erfassung einzelner Instanzwerte sowie aggregierter Daten mithilfe von `*.stock.trades`. Was geschieht jedoch, wenn Sie die Anwendung in mehreren Rechenzentren bereitstellen und die Metriken nach Rechenzentrum analysieren möchten? Sie können den Namen in `<datacenter>.<instanceid>.stock.trades` ändern. Dann funktionieren jedoch die Berichte nicht mehr, die das vorherige Platzhalterzeichen `*.stock.trades` verwenden. Es müsste stattdessen `*.*.stock.trades` verwendet werden. 

Die Verwendung von hierarchisch benannten Eigenschaften alleine kann leicht zu problematischen Mustern mit Platzhaltern führen, die mit der willkürlichen Struktur der Namensgebung verbunden sind und Ihnen nicht die Informationen bereitstellen, die Sie benötigen, um sicherzustellen, dass Ihre Anwendung gut funktioniert.

Mit Metriksystemen, die Dimensionsdaten unterstützen, können Sie Metrikdaten zusätzliche identifizierende Bezeichnungen oder Tags zuordnen. Anschließend können Sie die erfassten Metriken anhand dieser zusätzlichen Dimensionen filtern, gruppieren oder analysieren, ohne dass Sie Platzhalterzeichen verwenden müssen oder von der Namenskonvention abhängig sind. Typische Bezeichnungen sind der Endpunkt- oder Servicename, das Rechenzentrum, der Antwortcode, die Hosting-Umgebung (prod/staging/dev/test) oder die Laufzeit-IDs (Java-Version, App-Server-Informationen).

Bei Verwendung desselben Beispiels für den Börsenhandel ist die Metrik `stock.trades` verschiedenen Bezeichnungen zugeordnet: `{ instanceid=..., datacenter=... }`. Daher kann der aggregierte Wert nach `instanceid` oder `datacenter` gefiltert oder gruppiert werden, ohne Platzhalterzeichen zu verwenden. Es besteht ein Gleichgewicht zwischen der benannten Metrik (`stock.trades`) und den zugehörigen Bezeichnungen (vergleichen Sie dies mit dem hierarchischen Beispiel `<datacenter>.<instanceid>.stock.trades`): Jede Metrik sollte aussagekräftige Daten erfassen, wobei die Bezeichnungen bei Bedarf für die eindeutige Zuordnung sorgen.

Gehen Sie beim Definieren von Bezeichnungen vorsichtig vor. In Prometheus wird beispielsweise jede eindeutige Kombination aus Schlüssel/Wert-Bezeichnungspaaren als separate Zeitreihe behandelt. Ein bewährtes Verfahren, um ein gutes Abfrageverhalten und eine begrenzte Datenerfassung zu gewährleisten, ist die Verwendung von Bezeichnungen mit einer endlichen Anzahl zulässiger Werte. Wenn Sie beispielsweise eine Metrik verwenden, die die Anzahl der Fehler zählt, können Sie den Rückkehrcode als Bezeichnung verwenden, da sich die Werte innerhalb einer angemessenen Gruppe befinden (401, 404, 409, 500, ... ). Eine Bezeichnung für eine fehlgeschlagene URL wäre jedoch nicht sinnvoll, da es sich hierbei um eine unbegrenzte Gruppe handelt (jede Anforderungs-URL, die aus irgendeinem Grund fehlgeschlagen ist, u. a. weil sie ungültig ist).

Weitere Informationen zu bewährten Verfahren für die Benennung von Metriken und Beschriftungen (Labels) finden Sie unter [Metric and Label Naming](https://prometheus.io/docs/practices/naming/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

## Weitere Hinweise
{: #metrics-considerations}

Vergessen Sie beim Erfassen der Metriken nicht, dass ein Fehlerpfad sich häufig wesentlich von einem Erfolgspfad unterscheidet. So kann eine Fehlerantwort für eine HTTP-Ressource erheblich länger dauern als eine erfolgreiche Antwort, wenn der Fehler Zeitlimitüberschreitungen und die Stack-Trace-Erfassung beinhaltete. Zählen und behandeln Sie Fehlerpfade und erfolgreiche Anforderungen separat.

Ein verteiltes System weist natürliche Variationen bei bestimmten Messungen auf. Gelegentliche Fehler sind normal, da Anforderungen an Prozesse geleitet werden können, die gerade gestartet oder beendet werden. Filtern Sie die Rohdaten, um zu erkennen, wann diese natürlichen Variationen einen gültigen Bereich überschreiten. Teilen Sie Metriken beispielsweise in Buckets auf. Kategorisieren Sie die in einem gleitenden Zeitfenster gemessene Anforderungsdauer in Kategorien wie 'sehr kurz/schnell', 'mittel/normal' und 'sehr lang/langsam'. Wenn die Anforderungsdauer konsistent im Bucket "sehr lang/langsam" landet, gibt es ein Problem. Für diese Art der Daten werden häufig Histogramme oder Zusammenfassungsmetriken verwendet. Weitere Informationen finden Sie in [Histograms and Summaries](https://prometheus.io/docs/practices/histograms/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

Stellen Sie als Anwendungsentwickler sicher, dass Ihre Anwendungen oder Services Metriken mit Namen und Bezeichnungen ausgeben, die den organisationsweiten Konventionen entsprechen, um Überwachungsmaßnahmen zu unterstützen, die sich auf die End-to-End-Pfade konzentrieren, die für Ihr Unternehmen zentral sind. Weitere Informationen finden Sie in [Monitoring distributed systems](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").
