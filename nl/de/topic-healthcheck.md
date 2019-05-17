---

copyright:
  years: 2019
lastupdated: "2019-02-10"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Statusprüfung
{: #healthcheck}

Mit Statusprüfungen kann innerhalb von automatisierten Systemen auf einfache Weise der Allgemeinzustand einer einzelnen Instanz untersucht werden. Das System reagiert anschließend auf diese Statusprüfungsereignisse durch Ausführen einer Aktion. Beispielsweise kann das System eine fehlgeschlagene Instanz austauschen, die Routing-Tabellen aktualisieren oder die festgestellten Status an den Benutzer kommunizieren.
{:shortdesc}

In Kubernetes sind zwei integrierte Mechanismen für die Überprüfung des Status eines Containers definiert:

* Mit einen Readiness-Test (Bereitschaftstest) wird geprüft, ob der Prozess Anforderungen verarbeiten kann (Routing-fähig ist). Kubernetes leitet keine Arbeit an einen Container weiter, bei dem der Readiness-Test fehlgeschlagen ist. Ein Readiness-Test sollte fehlschlagen, wenn ein Service die Initialisierung noch nicht beendet hat oder aus anderen Gründen belegt, überlastet oder nicht in der Lage ist, Anforderungen zu verarbeiten.
* Mit einem Liveness-Test (Aktivitätstest) wird geprüft, ob der Prozess erneut gestartet werden muss. Kubernetes stoppt einen Container mit fehlgeschlagenem Liveness-Test und startet ihn erneut, um sicherzustellen, dass die Pods in einem nicht funktionsfähigen Status beendet und ausgetauscht werden. Ein Liveness-Test sollte fehlschlagen, wenn ein Service sich in einem nicht behebbaren Status befindet, wenn beispielsweise eine abnormale Speicherbedingung vorliegt. Mit einfachen Liveness-Tests, die immer die Antwort OK zurückgeben, können Container in einem inkonsistenten Status identifiziert werden. Dies kann vorkommen, wenn der Prozess, der Anforderungen bedient, abgestürzt ist, während der Container noch aktiv ist.

Readiness- und Liveness-Tests werden mit einer ähnlichen Struktur definiert, die Zeitverzögerung und Wiederholungsintervalle, Fehlertoleranzzeiträume, Zeitlimitüberschreitungen sowie die Definition der Testimplementierung enthält. Der Test kann implementiert werden, indem ein Befehl ausgeführt, ein TCP-Endpunkt auf eine Verbindung überprüft oder ein HTTP-Aufruf ausgeführt wird. Dieselbe Testimplementierung kann häufig sowohl für Readiness- als auch für Liveness-Tests verwendet werden, aber die Zeitverzögerung und die Wiederholungsintervalle müssen für den jeweiligen Zweck angepasst werden.

## Tests verstehen und anwenden
{: #kubernetes-probes}

Im Grunde basiert die Entwicklung von cloudnativen Anwendungen auf dem Prinzip, dass Containerprozesse tatsächlich fehlschlagen können, aber auf einfache Weise durch einen neuen Container ersetzt werden. Dies geschieht in Reaktion auf unerwartete Ereignisse wie Container- oder Maschinenfehler, aber auch aufgrund von Betriebsereignissen wie der horizontalen Skalierung und neuen Rollouts des Anwendungsimage. Readiness-Prüfungen sind wichtig, da sie sicherstellen, dass neue Containerinstanzen für Arbeit bereit sind, bevor Datenverkehr an diese Instanzen weitergeleitet wird. Außerdem verhindern diese Prüfungen, dass  Datenverkehr an Instanzen weitergeleitet wird, die beendet wurden oder gelöscht werden.

Sind keine Readiness-Prüfungen definiert, ist Kubernetes nicht bekannt, ob eine Containerinstanz für die Verarbeitung von Datenverkehr bereit ist, und leitet Datenverkehr sofort weiter, nachdem der Containerprozess gestartet wurde. Ohne Readiness-Prüfungen ist es wahrscheinlicher, dass bei Anwendungen Zeitlimitüberschreitungen für die Verbindung und abgelehnte Verbindungen auftreten, wenn Arbeit an eine Instanz weitergeleitet wird, die nicht bereit ist, die Anforderung zu bedienen. Readiness-Prüfungen verringern Clientverbindungsfehler, beseitigen sie aber nicht vollständig.

Obwohl Änderungen an den Routing-Zielen der Instanzen im Lebenszyklus einer containerfähigen Anwendung normal sind, definiert der Prozess, dass Liveness-Prüfungen, die für die Identifizierung konzipiert sind, seltener vorkommen und eher eine Ausnahme als den Normalfall darstellen. Wenn ein Prozess in einen Status eintritt, bei dem keine Wiederherstellung möglich ist, ist der Prozess praktisch nicht funktionsfähig. Dies kann beispielsweise bei abnormalen Speicherbedingungen oder einem Deadlock aufgrund eines Programmierfehlers vorkommen. Die beste Wiederherstellung bei diesen Situationen ist die Beendigung des Containers. Dabei wird auch sämtliche Verarbeitung beendet, die zu dieser Zeit in dem Container ausgeführt wird. Dies schafft auch die Möglichkeit, Schleifen in der Anwendung zu beenden oder neu zu starten, wenn Container nicht vollständig online gehen können, bevor sie beendet und ersetzt werden.

Readiness- und Liveness-Tests wirken sich auf unterschiedliche Weise auf das System aus. Dies lässt sich gut durch einen Statusübergang erklären: Der positive Status einer Readiness-Prüfung bedeutet, dass die Instanz Routing-fähig ist; der negative Status, dass sie nicht Routing-fähig ist. Ebenso stellt der positive Status einer Liveness-Prüfung einen Container dar, der normal ausgeführt wird, und der negative Status ist nicht funktionsfähig. Beim Start eines Containers ist der Readiness-Status zunächst negativ und der Container tritt erst in einen positiven Status ein, wenn er sich in einwandfreiem Zustand befindet. Eine Liveness-Prüfung beginnt mit dem positiven Status und tritt erst in einen negativen Status ein, wenn der Prozess funktionsunfähig wird.

Eine sehr aggressive Konfiguration einer Readiness-Prüfung, z. B. mit einer niedrigen Anfangsverzögerung, hat kaum Auswirkungen, da die zu frühe Ausführung des Tests nicht bewirkt, dass der Status der Readiness-Prüfung sich ändert. Andererseits bewirkt eine aggressive Liveness-Prüfung, bei der zu früh ein Fehler gemeldet wird, dass der Status sich ändert und das System den Container früher beendet als beabsichtigt.

## Bewährte Verfahren für die Konfiguration von Tests
{: #probe-recommendation}

Beachten Sie bei der Implementierung eines Statustests über HTTP die folgenden HTTP-Statuscodes für Readiness, Liveness und Status:

| Status    |  Readiness            |  Liveness             |
|----------|-----------------------|-----------------------|
|          | Nicht OK bewirkt keine Last | Nicht OK bewirkt Neustart |
| Wird gestartet | 503 - Nicht verfügbar     | 200 - OK              |
| Aktiv       | 200 - OK              | 200 - OK              |
| Wird gestoppt | 503 - Nicht verfügbar     | 200 - OK              |
| Inaktiv     | 503 - Nicht verfügbar     | 503 - Nicht verfügbar     |
| Fehlerhaft  | 500 - Serverfehler    | 500 - Serverfehler    |

Endpunkte für die Statusprüfung dürfen keine Berechtigung oder Authentifizierung erfordern. Da diese Schutzmechanismen nicht auf Endpunkten für Statusprüfungen implementiert werden, schränken Sie alle HTTP-Testimplementierungen auf GET-Anforderungen ein, die keine Daten ändern. Geben Sie keine Daten zurück, die die Besonderheiten der Umgebung identifizieren, wie das Betriebssystem, die Implementierungssprache oder Softwareversionen, da diese zum Erstellen eines Angriffsvektors verwendet werden können.

Für einen Liveness-Test muss sorgfältig überlegt werden, was geprüft wird, da ein Fehler zur sofortigen Beendigung des Prozesses führt. Vermeiden Sie mehrdeutige Metriken, die nur in manchen Fällen einen fehlgeschlagenen Prozess angeben, z. B. einen einfachen HTTP-Endpunkt, der immer `{"status": "UP"}` mit dem Statuscode 200 zurückgibt. Bei den meisten Prozessen, die sich im nicht funktionsfähigen Status befinden, schlägt diese Prüfung fehl, wodurch korrekterweise ein Neustart ausgelöst wird.

Statusprüfungen werden häufig ausgeführt. Dies kann zusätzlichen Systemaufwand verursachen. Mit Readiness- und Liveness-Tests sollte nur die Funktionsfähigkeit von Unterstützungsservices wie Datenbanken oder anderen Mikroservices in ihrem Ergebnis geprüft werden, wenn kein akzeptabler Fallback vorhanden ist. Für einen Liveness-Test sollte eine Prüfung der Unterstützungsservices nur eingeschlossen werden, wenn der lokale Container bei einem nicht erfolgreichen Ergebnis in einen nicht behebbaren Status eintreten würde. Bei einem Readiness-Test sollte ein Unterstützungsservice nur geprüft werden, wenn der lokale Container Anforderungen nicht verarbeiten kann, wenn er fehlschlägt; die Bedingung jedoch behebbar ist.

Bei der Konfiguration der ursprünglichen Zeitverzögerung sollte für einen Readiness-Test der niedrigste wahrscheinliche Wert und für eine Liveness-Prüfung der größte wahrscheinliche Zeitwert verwendet werden. Beispiel: Wenn ein Anwendungsserver normalerweise 30 Sekunden für den Start benötigt, beträgt eine typische Readiness-Verzögerung 10 Sekunden. Für die Liveness-Prüfung wird der Wert 60 Sekunden verwendet, um sicherzustellen, dass der Serverstart immer abgeschlossen ist, bevor auf Beendigung geprüft wird.

Das Attribut *periodSeconds* für Routing-Entscheidungen wird in der Regel mit einem einstelligen Wert konfiguriert, wenn die Testimplementierung relativ schlank ist. Beispiel: Ein HTTP-Test, der den Status '200 OK' ohne wesentliche serverseitige Verarbeitung zurückgibt, hat eine minimale Prozessorlast und kann problemlos alle 1 bis 5 Sekunden wiederholt werden.

## Tests in Kubernetes konfigurieren
{: #probe-config}

Deklarieren Sie Liveness- und Readiness-Tests neben Ihrer Kubernetes-Bereitstellung im Containerelement. Für beide Tests werden dieselben Konfigurationsparameter verwendet:

| Parameter | Beschreibung |
|-----------|-------------|
| *initialDelaySeconds* | Wie lange das Kubelet nach der Erstellung des Containers bis zum ersten Test wartet. |
| *periodSeconds* | Wie oft das Kubelet den Service testet. Der Standardwert ist 1. |
| *timeoutSeconds* | Wie schnell bei dem Test eine Zeitlimitüberschreitung auftritt. Der Standard- und Mindestwert ist 1. |
| *successThreshold* | Wie oft der Test nach einem Fehler erfolgreich verlaufen muss. Der Standard- und Mindestwert ist 1. Für Liveness-Tests muss der Wert 1 sein. |
| *failureThreshold* | Wie oft Kubernetes versucht, einen Pod erneut zu starten, bevor aufgegeben wird, wenn der Pod startet und der Test fehlschlägt (siehe Hinweis). Der Mindestwert ist 1 und der Standardwert ist 3. |

  Für einen Liveness-Test bedeutet 'aufgeben', dass der Pod erneut gestartet wird. Für einen Readiness-Test bedeutet 'aufgeben', dass der Pod als nicht bereit markiert wird.
  {: note}

Setzen Sie den Parameter `livenessProbe.initialDelaySeconds` auf einen Wert, der auf jeden Fall größer ist als die Zeit, die Ihr Service für die Initialisierung benötigt, um Neustartzyklen zu vermeiden. Sie können dann einen kürzeren Wert für das Attribut `readinessProbe.initialDelaySeconds` verwenden, um Anforderungen an den Service weiterzuleiten, sobald er bereit ist.

Eine Beispielkonfiguration könnte wie im folgenden Beispiel aussehen (beachten Sie die Pfad- und Portwerte):

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

Weitere Informationen finden Sie in [Configure Liveness and Readiness Probes in Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").
