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

# Beobachtbarkeit, Telemetrie und Überwachung
{: #observability-cn}

In Bezug auf die Überwachung bringt der Wechsel zu cloudnativen Anwendungen einen Änderung der Kultur mit sich. Obwohl sowohl in On-Premises-Umgebungen als auch in cloudnativen Umgebungen erwartet wird, dass Anwendungen hoch verfügbar und ausfallsicher sind, unterscheiden sich die Methoden, die verwendet werden, um diese Ziele zu erreichen. Dies hat zur Folge, dass der Zweck der Überwachung sich verändert: Die Überwachung wird nicht ausgeführt, um Fehler zu vermeiden, sondern um Fehler zu verwalten. 
{:shortdesc}

In On-Premises-Umgebungen wird Infrastruktur und Middleware auf der Basis der geplanten Kapazität und der Hochverfügbarkeitsmuster bereitgestellt, z. B. aktiv-aktiv oder aktiv-passiv. Unerwartete Fehler können in dieser Umgebung komplex sein und einen erheblichen Aufwand für die Fehlerbestimmung und Wiederherstellung erfordern. Die externe Überwachung wird von Agenten ausgeführt, die die Ressourcenauslastung untersuchen, um bekannte Klassen von Fehlern zu vermeiden. Ein Beispiel dafür ist die Optimierung der Heapspeichergröße, Zeitlimitwerte und Garbage-Collection-Richtlinien für Java-Anwendungen.

Eine cloudnative Anwendung setzt sich aus unabhängigen Mikroservices und erforderlichen Unterstützungsservices zusammen. Auch wenn eine cloudnative Anwendung als Ganzes verfügbar bleiben und weiter funktionieren muss, werden einzelne Serviceinstanzen gestartet oder gestoppt, um die Anwendung an die Kapazitätsanforderungen anzupassen oder die Wiederherstellung nach einem Fehler auszuführen. 

## Beobachtbarkeit
{: #observability}

Die Überwachung dieses fließenden Systems erfordert, dass jeder Teilnehmer *beobachtbar* ist. Jede Entität muss geeignete Daten erstellen, um die automatisierte Problemerkennung und Alertausgabe, das manuelle Debugging, falls notwendig, und die Analyse des Systemzustands (Langzeittrends und Analysefunktionen) zu unterstützen.

Welche Arten von Daten sollte ein Service erzeugen, damit er beobachtbar ist?

* **Statusprüfungen** (häufig angepasste HTTP-Endpunkte) helfen Orchestrierungssystemen wie Kubernetes oder Cloud Foundry, automatisierte Aktionen auszuführen, um den allgemeinen Systemzustand zu verwalten.
* **Metriken** sind numerische Darstellungen von Daten, die in Intervallen erfasst und in eine Zeitreihe eingeordnet werden. Numerische Zeitreihendaten lassen sich einfach speichern und abfragen, was bei der Suche nach Langzeittrends hilft. Über längere Zeiträume können numerische Daten in weniger differenzierte Aggregate zusammengefasst werden, z. B. in Tagen oder Wochen.
* **Protokolleinträge** stellen diskrete Ereignisse dar. Protokolleinträge sind für das Debugging von entscheidender Bedeutung, da sie häufig Stack-Traces und andere Kontextinformationen enthalten, die bei der Ermittlung der eigentlichen Ursache von festgestellten Fehlern hilfreich sind.
* Mit **verteiltem, Anforderungs- oder End-to-End-Tracing** wird der durchgängige Fluss einer Anforderung durch das System erfasst. Durch das Tracing werden im Wesentlichen sowohl die Beziehungen zwischen Services (die Services, die die Anforderung berührt hat) als auch die Struktur der Arbeit erfasst, die das System durchläuft (synchrone oder asynchrone Verarbeitung, Child-of- oder Follows-from-Beziehungen).

## Telemetrie
{: #telemetry}

Cloudnative Anwendungen überlassen der Umgebung die *Telemetrie*. Dies ist die automatische Erfassung und Übertragung von Daten an zentrale Speicherorte für die nachfolgende Analyse. Dies wird durch einen der zwölf Faktoren hervorgehoben, der angibt, dass Protokolle als Ereignisdatenströme zu behandeln sind, und gilt für alle von einem Mikroservice erstellten Daten, um sicherzustellen, dass er beobachtet werden kann.

Kubernetes verfügt über einige integrierte Telemetriefunktionen wie Heapster, aber es ist wahrscheinlicher, dass Telemetrie von anderen Systemen zur Verfügung gestellt wird, die in die Kubernetes-Steuerebene integriert sind. Beispielsweise arbeiten die beiden Istio-Komponenten Mixer und Envoy zusammen, um transparent [Telemetrie von bereitgestellten Anwendungen zu erfassen](https://istio.io/docs/concepts/policies-and-telemetry/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

Fehler sind nicht mehr seltene, störende Vorkommnisse. Bei der Aufspaltung einer monolithischen Anwendung in Mikroservices wird ein größerer Teil des Hauptpfads auf das Netz verlagert und die Auswirkungen der Latenz und anderer Netzprobleme verstärken sich. Anforderungen erreichen außerdem Prozesse, die aus einer ganzen Reihe von Gründen nicht für die Verarbeitung bereit sind. Services werden automatisch neu gestartet, wenn keine Ressourcen mehr zur Verfügung stehen, und durch Fehlertoleranzstrategien kann das System als Ganzes weiterhin funktionieren. Manuelle Eingriffe für einzelne Fehler sind in dieser Art der Umgebung nicht sinnvoll oder durchführbar.

## Überwachung
{: #monitoring}

Die Konzepte der Beobachtbarkeit und Telemetrie verdeutlichen einige wichtige Unterschiede bei der Überwachung von cloudnativen Anwendungen in umfangreichen verteilten Systemen. Vergessen Sie nicht, dass Prozesse in cloudnativen Umgebungen transient sind. Drei der zwölf Faktoren (Prozesse, Nebenläufigkeit und Verfügbarkeit) unterstreichen diesen Punkt. Vorab zugeordnete, lange andauernde, monolithische Prozesse werden durch viele weitere kurzlebige Prozesse ersetzt oder umschlossen, die bei hoher Last für die horizontale Skalierung oder bei nicht ordnungsgemäß funktionierenden Prozessen gestartet und gestoppt werden. Telemetrie ist kritisch, wenn Daten gesammelt und an anderer Stelle gespeichert werden müssen, um zu verhindern, dass sie verloren gehen, wenn Prozesse (Container) erstellt und gelöscht werden. Telemetrie ist oft auch aus Compliancegründen erforderlich. 

Aufgrund der oben beschriebenen Ursachen ändert sich der Fokus bei der Überwachung: Anstatt das Verhalten und den Zustand von Ressourcen (einzelnen Prozessen oder Maschinen) zu überwachen, wird der Status des Systems als Ganzes überwacht. Jeder einzelne Service erzeugt Daten, die in diese zusammengefasste Ansicht eingespeist werden.

## Tracing im Vergleich zu Protokollierung und Metriken
{: #trace-log-metrics}

:FIXME -- rephrase comparison as topic summary:

Die Protokollierung wird verwendet, wenn der Entwickler eine Nachricht explizit für jemanden ausgeben möchte. Sie wird direkt in die Java-Klasse codiert, einschließlich der Weitergabe von Werten relevanter Variablen. Wenn Probleme auftreten, sind die Protokolle für das Debugging hilfreich. Sie zeigen an, wo ein Fehler aufgetreten ist, wie z. B. ein Stack-Trace für eine Ausnahmebedingung, die ausgelöst wurde. Mit *Kibana* können Sie eine föderierte Sicht solcher Protokolle über Pods/Mikroservices hinweg anzeigen.

Das Tracing erfolgt automatisch; dies bedeutet, dass der Entwickler nicht tätig wird. Sie können Liberty beispielsweise so konfigurieren, dass Tracesätze an einen OpenTracing-konformen Trace-Server gesendet werden, wenn eine JAX-RS-annotierte Methode aufgerufen wird. Auf diese Weise verfügen Sie über einen Protokolleintrag mit Informationen darüber, was von wem und wie lange aufgerufen wurde. Sie können diesen Trace auch erweitern, beispielsweise mit Informationen darüber, welche privaten Methoden in Ihrem Code aufgerufen wurden, indem Sie OpenTracing-Annotationen zu solchen Methoden hinzufügen, für die ein Trace erstellt wird. 

Sie können ein Tool wie *Zipkin* oder *Jaeger* verwenden, um eine föderierte Sicht von Traces über Pods/Microservices anzuzeigen. Ein *Service-Netz* kann auch ein automatisches Tracing für Aufrufe bereitstellen, die über ein Sidecar für Ihren Container übergeben werden.  

Metriken werden zur Überwachung von Aggregatwerten verwendet. Anstelle sich durch einen Trace oder ein Protokoll zu arbeiten, um zu sehen, wie oft jemand Portfolios erstellt hat, können Sie dies zu einer benutzerdefinierten Zählermetrik machen. Sie können Ihre Bereitstellung so kennzeichnen, dass sie von *Prometheus* 'per Scraping erfasst' wird, z. B. wenn Sie in regelmäßigen Abständen den '/metrics'-URI für Ihre Pods aufrufen. Sie können dann mit einem Tool wie *Grafana* eine föderierte Sicht der Metriken über Pods/Mikroservices hinweg anzeigen.

Beim Tracing erfahren Sie, dass "diese Methode aufgerufen wurde". Die Protokollierung verrät Ihnen hingegen, "was in dieser Methode passiert ist, als sie aufgerufen wurde". Die Metriken wiederum zeigen an, "wie oft die Methode aufgerufen wurde". Das Tracing ist in der Regel implizit und erfolgt automatisch ohne jeglichen Eingriff seitens des Entwicklers. Die Protokollierung hingegen ist explizit und der Entwickler muss das Senden von Informationen codieren, die für die Post-Mortem-Analyse eines Problems von Bedeutung sein können. Metriken sind auch explizit, da Sie die Annotation zu der entsprechenden Methode Ihres Codes hinzufügen müssen (allerdings gibt es oft Low-Level-Standardmetriken, die ohne Aufwand seitens des Programmierers zur Verfügung stehen, z. B. für Speicherbelegeung, CPU-Auslastung und Threadanzahl).

In der Regel sind Metriken für die Analyse hilfreich, während die Protokollierung für Problembestimmungszwecke von Nutzen ist und das Tracing zum besseren Verständnis des Steuerungsablaufs zwischen verschiedenen Mikroservices dient.
