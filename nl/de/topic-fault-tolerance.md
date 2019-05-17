---

copyright:
  years: 2019
lastupdated: "2019-02-11"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Fehlertoleranz
{: #fault-tolerance}

Die Fehlertoleranz ist die Fähigkeit eines Systems, bei einem partiellen Ausfall weiterhin zu funktionieren. Die Erstellung eines ausfallsicheren Systems stellt Anforderungen an alle Services in diesem System. Die dynamische Natur von Cloudumgebungen erfordert, dass Services so geschrieben sind, dass sie auf unerwartete Ereignisse vorbereitet sind und damit umgehen können. Zu diesen Ereignissen zählen der Empfang fehlerhafter Daten, die Nichterreichbarkeit eines erforderlichen Unterstützungsservice oder Konflikte aufgrund gleichzeitig ablaufender Aktualisierungen in einem verteilten System.  

Fehlertoleranzlösungen konzentrieren sich in der Regel auf Zeitlimits, Fallbacks, Bulkheads und Circuit Breakers. 

In einigen Umgebungen können Fehlertoleranzmechanismen durch Infrastrukturkomponenten wie Istio bereitgestellt werden. Unabhängig davon, ob die Infrastruktur aushilft, muss ein Service davon ausgehen, dass der ferne Anruf fehlschlagen kann. Er sollte mit entsprechenden Fallback-Aktionen vorbereitet werden. 

## Zeitlimits

Die erste Verteidigungslinie gegen partielle Ausfälle ist die Verwendung von Zeitlimits. Zeitlimits stellen sicher, dass Ihre Anwendung einen Fehler empfängt, wenn ein Unterstützungsservice nicht antwortet, sodass sie die Bedingung mit dem entsprechenden Fallback-Verhalten handhaben kann. Dies bedeutet nicht unbedingt, dass die angeforderte Operation fehlgeschlagen ist. Zeitlimits legen fest, wie lange ein Client, der eine Anforderung sendet, auf eine Antwort wartet; sie wirken sich nicht auf das Verarbeitungsverhalten des Zielservice aus. 

In vielen Sprachbibliotheken wird ein Standardzeitlimit für Anforderungen verwendet. Dies ist auch in Istio der Fall. Standardmäßig lässt der Sidecar-Proxy die Anforderung fehlschlagen, wenn er nicht innerhalb von 15 Sekunden eine Antwort empfängt. Dieser Wert kann geändert werden, indem eine Zeitlimitrichtlinie für die Route in der VirtualService-Definition festgelegt wird. Für einen Service, der Börsennotierungen zurückgibt, kann die Richtlinie beispielsweise wie folgt aussehen: 

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: stock-quote-service
spec:
  hosts:
  - 'stock-quote-service'
  http:
  - route:
    - destination:
        host: stock-quote-service
    timeout: 30s
```
{: codeblock}

Bei dieser Konfiguration warten Anforderungen, die über den Sidecar-Proxy oder das Ingress-Gateway an den Service für Börsennotierung gesendet werden, anstelle der standardmäßigen 15 Sekunden 30 Sekunden lang, bevor die Anforderung fehlschlägt. 

Der interessante Aspekt bei der Optimierung von Zeitlimits ist, dass das Zeitlimit auf alle Anforderungen angewendet wird, die die Route verwenden. Dies ist eine von Istio bereitgestellte Basisebene der Sicherheit: Selbst wenn ein Anwendungsframework oder eine Bibliothek keine Zeitlimits erzwingt, wird nicht ewig gewartet, da Istio Zeitlimits erzwingt. Zeitlimits auf Anwendungsebene gelten dennoch. Im obigen Beispiel wurde das Zeitlimit für den Service für Börsennotierung auf der Infrastrukturebene auf 30 Sekunden erweitert. Wenn die Anwendungsbibliothek ein Zeitlimit von 5 Sekunden festlegt, schlägt die Anforderung der Anwendung immer noch mit einer Zeitlimitüberschreitung fehl. 

## Fallbacks

In einer Anwendung sollte definiert sein, was geschieht, wenn ein Unterstützungsservice fehlschlägt. Es gibt mehrere Optionen; das Ziel ist jedoch eine Leistungsverschlechterung und kein Fehler, wenn diese Services nicht zeitnah antworten. Wenn ein ferner Service fehlschlägt, ist es möglich, die Anforderung zu wiederholen, eine andere Anforderung zu versuchen oder stattdessen zwischengespeicherte Daten zurückzugeben. 

Auf den ersten Blick ist die Wiederholung der Anforderung der einfachste Fallback-Mechanismus. Nicht so offensichtlich ist jedoch, dass die Wiederholung von Anforderungen zu kaskadierenden Systemfehlern beitragen kann (sogenannten "Retry Storms", einer Variante des [Thundering-Herd-Problems](https://en.wikipedia.org/wiki/Thundering_herd_problem)). Der Code auf der Anwendungsebene verfügt nicht über genügend Informationen zum System- oder Netzzustand und die richtige Anwendung von Exponential-Backoff-Algorithmen ist schwierig. 

Istio kann Wiederholungen wesentlich effektiver ausführen. Die Plattform ist bereits direkt am Anforderungsrouting beteiligt und stellt eine konsistente, von der Sprache unabhängige Implementierung für Wiederholungsrichtlinien zur Verfügung. Zum Beispiel könnten wir eine Richtlinie wie die folgende für unseren Service für Börsennotierung definieren: 

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: stock-quote-service
spec:
  hosts:
  - 'stock-quote-service'
  http:
  - route:
    - destination:
        host: stock-quote-service
    retries:
      attempts: 3
      perTryTimeout: 5s
```
{: codeblock}

Mit dieser einfachen Konfiguration werden Anforderungen, die über einen Istio-Sidecar-Proxy oder ein Ingress-Gateway an den Service für Börsennotierung gesendet werden, bis zu 3 Mal wiederholt und das Zeitlimit für jeden Versuch beträgt 5 Sekunden. Mithilfe [zusätzlicher Abgleichsregeln für Routen](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#HTTPMatchRequest) könnte diese Wiederholungsrichtlinie beispielsweise auf `GET`-Anforderungen beschränkt werden. 

Hierbei ist ein Detail leicht zu übersehen: Sie geben das Wiederholungsintervall nicht an. Das Sidecar legt das Intervall zwischen Wiederholungen fest und fügt absichtlich Abweichungen zwischen Versuchen ein, um zu vermeiden, dass überlastete Services mit Anforderungen bombardiert werden. 

## Bulkheads

In der Schifffahrt ist ein Bulkhead eine Trennwand, die verhindert, dass ein Leck in einem Bereich dazu führt, dass das ganze Schiff sinkt. Dieses Prinzip wird in Cloudumgebungen zu einem ähnlichen Zweck verwendet. Die Anwendung kann auf unterschiedliche Weise erfolgen. 

Bei Multithread-Sprachen wie Java können interne Bulkheads intern verwendet werden, um einzuschränken oder zu steuern, wie Ressourcen für die Kommunikation mit fernen Ressourcen verwendet werden. Dabei werden entweder Warteschlangen oder Semaphormechanismen genutzt: 

- Bei der Verwendung einer Warteschlange ordnet der Service eine bestimmte Anzahl an Threads einer bestimmten Warteschlange zu. Alle Anforderungen, die gesendet werden, wenn die Warteschlange voll ist, empfangen schnell eine Fehlernachricht. In Java könnte dies beispielsweise `ThreadPoolExecutor` zusammen mit `BlockingQueue` sein. 
- Der Semaphormechanismus arbeitet mit einer festgelegten Anzahl von Genehmigungen. Für eine ausgehende Anforderung ist eine Genehmigung erforderlich. Nachdem eine Anforderung erfolgreich ausgeführt wurde, wird die Genehmigung freigegeben und kann von einer anderen Anforderung verwendet werden. 

Sie können auch mit einer Istio-DestinationRule Bulkheads zwischen Services definieren, um den Verbindungspool für einen vorgelagerten Service zu beschränken. Verwenden Sie ähnlichen Code wie im folgenden Beispiel: 

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: stock-quote-service
spec:
  host: stock-quote-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 10
        connectTimeout: 30s
```
{: codeblock}

Diese Konfiguration begrenzt die maximale Anzahl gleichzeitiger Verbindungen, die zu den einzelnen Instanzen des Service für Börsennotierung hergestellt werden können, auf 10. Services, die innerhalb von 30 Sekunden keine Verbindung herstellen können, erhalten die Antwort `503 -- Service nicht verfügbar`. Mit diesem Typ des Bulkhead kann z. B. verhindert werden, dass ein rechenintensiver Service mehr Anforderungen empfängt als er verarbeiten kann. 

## Circuit Breakers

Circuit Breakers werden verwendet, um das Verhalten ausgehender Anforderungen zu optimieren, wenn Fehler auftreten. Anstatt wiederholt Anforderungen an einen nicht reagierenden Service zu senden, beobachtet ein Circuit Breaker die Anzahl der Fehler, die innerhalb eines bestimmten Zeitraums aufgetreten sind. Wenn die Fehlerrate einen Schwellenwert überschreitet, trennt der Circuit Breaker den Schaltkreis. Dadurch schlägt die Anforderung fehl. Alle nachfolgenden Anforderungen schlagen ebenfalls fehl, bis der Schaltkreis wieder geschlossen wird. 

Circuit Breakers werden ebenfalls mithilfe einer Istio-DestinationRule definiert: 

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: stock-quote-service
spec:
  host: stock-quote-service
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 3
      interval: 5s
      baseEjectionTime: 5m
      maxEjectionPercent: 100
```
{: codeblock}

Diese Konfiguration legt Einschränkungen für Anforderungen fest, die andere Services an den Service für Börsennotierung senden. Die angegebene Datenverkehrsrichtlinie `outlierDetection` wird auf jede einzelne Instanz angewendet. Eine Beschreibung der obigen Konfiguration in einem Satz lautet wie folgt: "Entferne jede Instanz des Service für Börsennotierung, die mindestens 5 Minuten lang 3 Mal in 5 Sekunden fehlschlägt; zudem können alle Instanzen entfernt werden." Die letzte Einstellung `maxEjectionPercent` bezieht sich auf den Lastausgleich. Istio verwaltet einen Lastausgleichspool und entfernt fehlgeschlagene Instanzen aus diesem Pool. Standardmäßig werden maximal 10 % aller verfügbaren Instanzen aus dem Lastausgleichspool entfernt. 

Falls Sie mit anderen Circuit Breaker-Mechanismen vertraut sind: Bei Istio gibt es keinen halbgeöffneten Zustand. Stattdessen wird eine einfache mathematische Operation angewendet: Die Zeit, über die eine Instanz außerhalb des Pools verbleibt, wird mit `baseInjectionTime * <number of times it has been ejected>` berechnet. Auf diese Weise können Instanzen mit vorübergehenden Fehlern wiederhergestellt werden, während Instanzen, die permanent fehlschlagen, außerhalb des Pools verbleiben. 

