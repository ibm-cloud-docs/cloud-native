---

copyright:
  years: 2019
lastupdated: "2019-04-09"

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

In Bezug auf die Überwachung bringt der Wechsel zu cloudnativen Anwendungen einen Änderung der Kultur mit sich. Obwohl sowohl in On-Premises-Umgebungen als auch in cloudnativen Umgebungen erwartet wird, dass Anwendungen hoch verfügbar und ausfallsicher sind, unterscheiden sich die Methoden erheblich, die verwendet werden, um diese Ziele zu erreichen. Dies hat zur Folge, dass der Zweck der Überwachung sich verändert: Die Überwachung wird nicht ausgeführt, um Fehler zu vermeiden, sondern um Fehler zu verwalten.
{:shortdesc}

In On-Premises-Umgebungen wird Infrastruktur und Middleware auf der Basis der geplanten Kapazität und der Hochverfügbarkeitsmuster bereitgestellt, z. B. aktiv-aktiv oder aktiv-passiv. Unerwartete Fehler können in dieser Umgebung komplex sein und einen erheblichen Aufwand für die Fehlerbestimmung und Wiederherstellung erfordern. Die externe Überwachung wird von Agenten ausgeführt, die die Ressourcenauslastung untersuchen, um bekannte Klassen von Fehlern zu vermeiden. Ein Beispiel dafür ist die Optimierung der Heapspeichergröße, Zeitlimitwerte und Garbage-Collection-Richtlinien für Java-Anwendungen. 

Eine cloudnative Anwendung setzt sich aus unabhängigen Mikroservices und erforderlichen Unterstützungsservices zusammen. Auch wenn eine cloudnative Anwendung als Ganzes verfügbar bleiben und weiter funktionieren muss, werden einzelne Serviceinstanzen nach Bedarf gestartet oder gestoppt, um die Anwendung an die Kapazitätsanforderungen anzupassen oder die Wiederherstellung nach einem Fehler auszuführen.  

## Beobachtbarkeit
{: #observability}

Die Überwachung dieses fließenden Systems erfordert, dass jeder Teilnehmer *beobachtbar* ist. Jede Entität muss geeignete Daten erstellen, um die automatisierte Problemerkennung und Alertausgabe, das manuelle Debugging, falls notwendig, und die Analyse des Systemzustands (Langzeittrends und Analysefunktionen) zu unterstützen. 

Welche Arten von Daten sollte ein Service erzeugen, damit er beobachtbar ist? 

* **Statusprüfungen** (häufig angepasste HTTP-Endpunkte) helfen Orchestrierungssystemen wie Kubernetes oder Cloud Foundry, automatisierte Aktionen auszuführen, um den allgemeinen Systemzustand zu verwalten. 
* **Metriken** sind numerische Darstellungen von Daten, die in Intervallen erfasst und in eine Zeitreihe eingeordnet werden. Numerische Zeitreihendaten lassen sich einfach speichern und abfragen, was bei der Suche nach Langzeittrends hilft. Über längere Zeiträume können numerische Daten in weniger differenzierte Aggregate zusammengefasst werden, z. B. in Tagen oder Wochen. 
* **Protokolleinträge** stellen diskrete Ereignisse dar, die im Lauf der Zeit aufgetreten sind. Protokolleinträge sind für das Debugging von entscheidender Bedeutung, da sie häufig Stack-Traces und andere Kontextinformationen enthalten, die bei der Ermittlung der eigentlichen Ursache von festgestellten Fehlern hilfreich sind. 
* Mit **verteiltem, Anforderungs- oder End-to-End-Tracing** wird der durchgängige Fluss einer Anforderung durch das System erfasst. Durch das Tracing werden im Wesentlichen sowohl die Beziehungen zwischen Services (die Services, die die Anforderung berührt hat) als auch die Struktur der Arbeit erfasst, die durch das System fließt (synchrone oder asynchrone Verarbeitung, Child-of- oder Follows-from-Beziehungen). 

## Telemetrie
{: #telemetry}

Cloudnative Anwendungen sollten der Umgebung die *Telemetrie* überlassen. Dies ist die automatische Erfassung und Übertragung von Daten an zentrale Speicherorte für die nachfolgende Analyse. Dies wird durch einen der zwölf Faktoren hervorgehoben, der angibt, dass Protokolle als Ereignisdatenströme zu behandeln sind, und gilt für alle von einem Mikroservice erstellten Daten, um sicherzustellen, dass er beobachtet werden kann. 

Kubernetes verfügt über einige integrierte Telemetriefunktionen wie Heapster, aber es ist wahrscheinlicher, dass Telemetrie von anderen Systemen zur Verfügung gestellt wird, die in die Kubernetes-Steuerebene integriert sind. Beispielsweise arbeiten die beiden Istio-Komponenten Mixer und Envoy zusammen, um transparent [Telemetrie von bereitgestellten Anwendungen zu erfassen](https://istio.io/docs/concepts/policies-and-telemetry/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link"). 

Fehler sind nicht mehr seltene, störende Vorkommnisse. Bei der Aufspaltung einer monolithischen Anwendung in Mikroservices wird ein größerer Teil des Hauptpfads auf das Netz verlagert und die Auswirkungen der Latenz und anderer Netzprobleme verstärken sich. Anforderungen erreichen außerdem Prozesse, die aus einer ganzen Reihe von Gründen nicht für die Verarbeitung bereit sind. Services werden automatisch neu gestartet, wenn keine Ressourcen mehr zur Verfügung stehen, und durch Fehlertoleranzstrategien kann das System als Ganzes weiterhin funktionieren. Manuelle Eingriffe für einzelne Fehler sind in dieser Art der Umgebung nicht sehr sinnvoll oder durchführbar. 

## Überwachung
{: #monitoring}

Die Konzepte der Beobachtbarkeit und Telemetrie verdeutlichen einige wichtige Unterschiede bei der Überwachung von cloudnativen Anwendungen in umfangreichen verteilten Systemen. Vergessen Sie nicht, dass Prozesse in cloudnativen Umgebungen transient sind. Drei der zwölf Faktoren (Prozesse, Nebenläufigkeit und Verfügbarkeit) unterstreichen diesen Punkt. Vorab zugeordnete, lange andauernde, monolithische Prozesse werden durch viele weitere kurzlebige Prozesse ersetzt oder umschlossen, die bei hoher Last für die horizontale Skalierung oder bei nicht ordnungsgemäß funktionierenden Prozessen gestartet und gestoppt werden. Telemetrie ist kritisch, wenn Daten gesammelt und an anderer Stelle gespeichert werden müssen, um zu verhindern, dass sie verloren gehen, wenn Prozesse (Container) erstellt und gelöscht werden. Telemetrie ist oft auch aus Compliancegründen erforderlich.  

Aufgrund der oben beschriebenen Ursachen ändert sich der Fokus bei der Überwachung: Anstatt das Verhalten und den Zustand von Ressourcen (einzelnen Prozessen oder Maschinen) zu überwachen, wird der Status des Systems als Ganzes überwacht. Jeder einzelne Service erzeugt Daten, die in diese zusammengefasste Ansicht eingespeist werden. 

