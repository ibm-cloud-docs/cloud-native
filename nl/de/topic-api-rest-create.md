---

copyright:
  years: 2019
lastupdated: "2019-03-26"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# REST-konforme Mikroservices erstellen
{: #rest-api}

Cloudnative Anwendungen erzeugen und konsumieren APIs, unabhängig davon, ob es sich um eine Mikroservicearchitektur handelt. Einige APIs werden als intern oder privat und andere als extern betrachtet.
{:shortdesc}

Interne APIs werden nur innerhalb von Umgebungen mit Firewall für die Kommunikation zwischen den Back-End-Services verwendet. Externe APIs stellen einen einheitlichen Einstiegspunkt für Konsumenten dar und werden häufig von Tools wie {{site.data.keyword.apiconnect_long}} **verwaltet**, die eine Durchsatzbegrenzung oder andere Nutzungsbegrenzungen erzwingen können. Ein Beispiel für eine solche API ist die [GitHub Developer API](https://developer.github.com/v3/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link"). Sie stellt eine einheitliche API mit konsistenter Verwendung von HTTP-Verben und Rückkehrcodes, Paginierungsverhalten usw. bereit, ohne interne Implementierungsdetails offenzulegen. Diese API kann von einer monolithischen Anwendung oder durch eine Sammlung von Mikroservices unterstützt werden; diese Information wird dem Konsumenten nicht offengelegt. So bleibt GitHub frei, seine internen Systeme nach Bedarf weiterzuentwickeln. 

## Bewährte Verfahren für REST-konforme APIs
{: #bps-apis}

REST-APIs sollten HTTP-Standardverben für die Operationen zum Erstellen, Abrufen, Aktualisieren und Löschen (Create, Retrieve, Update, Delete - CRUD) verwenden, wobei besonderes Augenmerk auf die Idempotenz der Operation liegen sollte (d. h., die Operation kann problemlos mehrfach wiederholt werden). 

* POST-Operationen können verwendet werden, um Ressourcen zu erstellen oder zu aktualisieren. POST-Operationen können nicht wiederholt aufgerufen werden. Wenn zum Beispiel eine POST-Anforderung zum Erstellen von Ressourcen verwendet wird und sie mehrfach aufgerufen wird, wird als Ergebnis jedes Aufrufs eine neue eindeutige Ressource erstellt. 
* GET-Operationen müssen sich wiederholt aufrufen lassen, ohne Nebeneffekte zu verursachen. Sie sollten nur zum Abrufen von Informationen verwendet werden. GET-Anforderungen mit Abfrageparametern sollten nicht verwendet werden, um Informationen zu ändern oder zu aktualisieren. Verwenden Sie stattdessen die Operationen POST, PUT oder PATCH. 
* PUT-Operationen können verwendet werden, um Ressourcen zu aktualisieren. PUT-Operationen enthalten in der Regel eine vollständige Kopie der zu aktualisierenden Ressource, sodass die Operation mehrfach aufgerufen werden kann. 
* Mit PATCH-Operationen können Ressourcen teilweise aktualisiert werden. Abhängig davon, wie das Delta angegeben und anschließend auf die Ressource angewendet wird, können sie wiederholt aufgerufen werden. Wenn z. B. eine PATCH-Operation angibt, dass ein Wert von A in B geändert werden soll, kann sie wiederholt aufgerufen werden. Sie hat keine Auswirkungen, wenn sie mehrfach aufgerufen wird und der Wert bereits B lautet. 
* DELETE-Operationen können mehrfach aufgerufen werden, da eine Ressource nur einmal gelöscht werden kann. Der Rückkehrcode ist jedoch unterschiedlich, da die erste Operation erfolgreich ist (`200` oder `204`), während die Ressource bei nachfolgenden Aufrufen nicht gefunden wird (`404` oder `410`). 

### Maschinenfreundliche, beschreibende Ergebnisse
{: #rest-results}

Da APIs von Software und nicht von Menschen aufgerufen werden, sollte darauf geachtet werden, dass Informationen möglichst effektiv und effizient an die aufrufende Software kommuniziert werden. 

Verwenden Sie relevante und nützliche HTTP-Statuscodes, wie in der folgenden Tabelle beschrieben:  

| HTTP-Fehlercode | Verwendungshinweise|
|-----------------|----------------|
| `200 (OK)` | Zu verwenden, wenn alles in Ordnung ist und es Daten gibt, die zurückgegeben werden können. |
| `204 (NO CONTENT)` | Zu verwenden, wenn alles in Ordnung ist, aber keine Antwortdaten vorhanden sind. |
| `201 (CREATED)` | Für POST-Anforderungen zu verwenden, die zur Erstellung einer Ressource führen, unabhängig davon, ob ein Antworthauptteil vorhanden ist. |
| `409 (CONFLICT)` | Zu verwenden, wenn gleichzeitige Änderungen miteinander in Konflikt stehen. |
| `400 (BAD REQUEST)` | Zu verwenden, wenn Parameter fehlerhaft sind. |

Weitere Informationen finden Sie in [Response status codes](https://tools.ietf.org/html/rfc7231#section-6){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").  

Sie sollten auch überlegen, welche Daten in Ihren Antworten zurückgegeben werden sollten, um die Kommunikation effizient zu gestalten. Wenn z. B. eine Ressource mit einer POST-Anforderung erstellt wird, sollte die Antwort die Position der neu erstellten Ressource in einem Header 'Location' enthalten. Häufig wird auch die erstellte Ressource in den Antworthauptteil aufgenommen, um die zusätzliche GET-Anforderung zum Abrufen der erstellten Ressource zu eliminieren. Dasselbe gilt für PUT-  und PATCH-Anforderungen. 

### REST-konforme Ressourcen-URIs
{: #rest-uris}

Es gibt unterschiedliche Meinungen zu einigen Aspekten von REST-konformen Ressourcen-URIs. Im Allgemeinen besteht Übereinstimmung darüber, dass Ressourcen Substantive und keine Verben sein sollten und dass Endpunkte im Plural stehen sollten. Daraus ergibt sich eine klare Struktur für CRUD-Operationen: 

* `POST /accounts`: Ein neues Konto erstellen. 
* `GET /accounts`: Eine Liste der Konten abrufen. 
* `GET /accounts/16`: Ein bestimmtes Konto abrufen. 
* `PUT /accounts/16`: Ein bestimmtes Konto aktualisieren. 
* `PATCH /accounts/16`: Ein bestimmtes Konto aktualisieren. 
* `DELETE /accounts/16`: Ein bestimmtes Konto löschen. 

Beziehungen werden mit hierarchischen URIs modelliert, z. B. ` /accounts/16/credentials` für die Verwaltung von Berechtigungsnachweisen, die einem Konto zugeordnet sind. 

Weniger Übereinstimmung besteht darüber, wie mit der Ressource verknüpfte Operationen zu behandeln sind, die nicht in diese übliche Struktur passen. Es gibt nicht das eine korrekte Verfahren zur Verwaltung dieser Operationen: Überlegen Sie, was für den Konsumenten der API am besten ist. 

### Zuverlässigkeit und REST-konforme APIs
{: #robust-api}

Das [Zuverlässigkeitsprinzip](https://tools.ietf.org/html/rfc1122#page-12){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") ist die beste Anleitung: "Seien Sie liberal bei den Daten, die Sie akzeptieren, und konservativ bei den Daten, die Sie senden." Gehen Sie davon aus, dass APIs sich mit der Zeit weiterentwickeln, und seien Sie tolerant bei den Daten, die Sie nicht verstehen. 

#### APIs erzeugen
{: #robust-producer}

Bei der Bereitstellung einer API für externe Clients müssen Sie beim Akzeptieren von Anforderungen und Zurückgeben von Antworten zwei Punkte beachten.  

* Akzeptieren Sie unbekannte Attribute als Bestandteil der Anforderung.
    > Wenn ein Service Ihre API mit nicht notwendigen Attributen aufruft, werfen Sie diese Werte einfach weg. Wenn in diesem Szenario ein Fehler zurückgegeben wird, kann dies zu unnötigen Störungen führen, die den Endbenutzer beeinträchtigen. 
* Geben Sie nur die Attribute zurück, die Ihre Konsumenten benötigen.
    > Vermeiden Sie es, interne Servicedetails zugänglich zu machen. Machen Sie nur Attribute zugänglich, die Konsumenten als Bestandteil der API benötigen. 

#### APIs konsumieren
{: #robust-consumer}

Validieren Sie die Anforderung nur anhand der Variablen oder Attribute, die Sie benötigen. 

    > Führen Sie keine Validierung anhand von Variablen aus, nur weil sie angegeben werden. Wenn Sie die Variablen nicht als Bestandteil Ihrer Anforderung verwenden, sollten Sie sich nicht darauf verlassen, dass sie vorhanden sind. 

Akzeptieren Sie unbekannte Attribute als Bestandteil der Antwort. 

    > Geben Sie keine Ausnahmebedingung aus, wenn Sie eine unerwartete Variable empfangen. Solange die Antwort die Informationen enthält, die Sie benötigen, spielen die zusätzlich gesendeten Informationen keine Rolle. 

Diese Richtlinien sind besonders relevant für stark typisierte Sprachen wie Java, während die Serialisierung und Deserialisierung von JSON häufig indirekt auftritt, z. B. über die Jackson-Bibliotheken oder JSON-P/JSON-B. Suchen Sie nach Sprachmechanismen, mit denen Sie ein großzügigeres Verhalten wie das Ignorieren von unbekannten Attributen angeben oder die zu serialisierenden Attribute definieren oder filtern können. 

### Versionssteuerung für REST-konforme APIs
{: #version-api}

Einer der Hauptvorteile von Mikroservices ist die Tatsache, dass Services unabhängig voneinander weiterentwickelt werden können. Vor dem Hintergrund, dass Mikroservices andere Services aufrufen, bringt dies einen großen Nachteil mit sich: In der API können keine Änderungen vorgenommen werden, die zu Fehlern bei anderen Services führen. 

Wenn das Zuverlässigkeitsprinzip befolgt wird, kann es lange dauern, bis eine Änderung erforderlich ist, die zu Fehlern bei anderen Services führt. Wenn dieser Zeitpunkt schließlich gekommen ist, können Sie sich entscheiden, einen ganz neuen Service zu entwickeln und den ursprünglichen Service später zurückzuziehen. 

Falls es doch erforderlich ist, API-Änderungen an einem vorhandenen Service vorzunehmen, die zu Fehlern bei anderen Services führen, müssen Sie entscheiden, wie Sie diese Änderungen verwalten: Soll der Service alle Versionen der API verarbeiten, sollen unabhängige Versionen des Service für die Unterstützung jeder Version der API verwaltet werden oder soll Ihr Service nur die neueste Version der API unterstützen, während andere adaptive Schichten die Konvertierung in und aus der älteren API übernehmen? 

Nachdem Sie festgelegt haben, wie die Änderungen verwaltet werden, müssen Sie nur noch das viel einfachere Problem lösen, wie die Version in Ihrer API widergespiegelt werden soll. Es gibt im Allgemeinen drei Möglichkeiten, um eine REST-Ressource zu versionieren: 

* Schließen Sie die Version in den URI-Pfad ein. 
* Schließen Sie die Version in den HTTP-Accept-Header ein und verlassen Sie sich auf die Inhaltsvereinbarung. 
* Verwenden Sie einen angepassten Anforderungsheader. 

#### Version in den URI-Pfad einschließen
{: #version-path}

Das einfachstes Verfahren zum Angeben einer Version besteht darin, sie in den Pfad des URI einzuschließen. Dieser Ansatz hat Vorteile: Die Version ist klar ersichtlich, die Angabe der Version bei der Entwicklung des Service in Ihrer Anwendung ist einfach und das Verfahren ist kompatibel mit Browsing-Tools wie Swagger sowie mit Befehlszeilentools wie `curl` usw. 

Wenn Sie die Version in den Pfad des URI einschließen, sollte die Version für Ihre gesamte Anwendung gelten, z. B. sollte `/api/v1/accounts` statt `/api/accounts/v1` verwendet werden. HATEOS (Hypermedia as the Engine of Application State) ist eine der Möglichkeiten, um API-Konsumenten URIs zur Verfügung zu stellen, so dass sie nicht selbst für die Erstellung von URIs verantwortlich sind. Beispielsweise stellt GitHub aus diesem Grund [Hypermedia-URLs](https://developer.github.com/v3/\#hypermedia){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") in den Antworten zur Verfügung. HATEOAS wird schwierig, wenn nicht gar unmöglich, wenn verschiedene Back-End-Services unterschiedliche Versionen in ihren URIs enthalten können. 

#### Version in den Accept-Header einschließen
{: #version-accept}

Der Accept-Header ist auf den ersten Blick für die Definition einer Version geeignet. Hierbei ist der Test jedoch extrem schwierig. Der Accept-Header ist auch häufig die Landing-Position für Funktionsumschalter. Für die Angabe von HTTP-Headern sind detailliertere API-Aufrufe erforderlich. 

#### Angepassten Anforderungsheader hinzufügen
{: #version-custom}

Sie können einen angepassten Anforderungsheader hinzufügen, um die API-Version anzugeben. Wie beim Accept-Header können Sie auch angepasste Header verwenden, um den Datenverkehr an bestimmte Back-End-Instanzen weiterzuleiten. Bei dieser Methode treten dieselben Probleme mit der Benutzerfreundlichkeit wie beim Accept-Header auf, mit der zusätzlichen Anforderung, dass die Konsumenten Informationen zu diesem Header erhalten müssen. 

Weitere Informationen finden Sie in [Your API versioning is wrong, which is why I decided to do it 3 different wrong ways](https://www.troyhunt.com/your-api-versioning-is-wrong-which-is/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link"). 

## APIs erstellen und generieren
{: #create-api}

[OpenAPI V3](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") ist die offizielle Spezifikation für REST-konforme Services, die von der [OpenAPI Initiative](https://www.openapis.org/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") erstellt wurde; einem Zusammenschluss von Unternehmen unter der Linux Foundation. 

Sie können eine der folgenden Methoden zum Erstellen einer API verwenden: 

  * Beginnen Sie mit einer OpenAPI-Definition (Top-down): Bei diesem Ansatz beginnen Sie mit der Erstellung einer OpenAPI-Definition in einem sprachunabhängigen Format (normalerweise YAML). Anschließend verwenden Sie einen Codegenerator, um ein Gerüst zu erstellen und dieses als Ausgangspunkt für Ihre Serviceimplementierung zu verwenden. Dieses Muster wird normalerweise von Unternehmen verwendet, die über ein zentrales API-Entwicklungsteam verfügen. Es ermöglicht parallele Entwicklungs- und Testaktivitäten. 
  * Beginnen Sie mit dem Code (Bottom-up): Ihr Code ist die Quelle Ihrer API-Definition. Dieser Ansatz funktioniert gut für neue Anwendungen mit einem experimentellen Aspekt, da Ihre API-Definition sich weiterentwickelt, wenn Sie ein besseres Verständnis der Funktionen erlangen, die Ihr Service ausführen muss. Dieser Ansatz funktioniert außerdem in einigen Sprachen besser als in anderen, da Tools benötigt werden, die aus Ihrem Code eine OpenAPI-Definition generieren. Java bietet beispielsweise eine ausgezeichnete Unterstützung für die Generierung von OpenAPI-Dokumenten aus annotationsbasierten REST-Frameworks. 

In jedem Fall kann die Arbeit mit einer OpenAPI-Definition dazu beitragen, Bereiche zu identifizieren, in denen die API aus Konsumentensicht inkonsistent oder schwer zu verstehen ist. Veröffentlichte oder versionsgesteuerte OpenAPI-Definitionen können auch von Build-Tools verwendet werden, um Änderungen zu markieren, die zu Fehlern bei den Konsumenten führen würden. 

### API aus einer OpenAPI-Definition erstellen
{: #openapi-first}

Sie können Ihre OpenAPI-YAML-Datei in einem beliebigen Tool erstellen. Die Verwendung eines einfachen Texteditors kann jedoch fehleranfällig sein. Einige Editoren verfügen über eine Basisunterstützung für YAML und einige verfügen möglicherweise über zusätzliche Erweiterungen für die Unterstützung von OpenAPI-Definitionen. Beispielsweise können Sie Visual Studio Code-Erweiterungen wie [Swagger Viewer](https://marketplace.visualstudio.com/items?itemName=Arjun.swagger-viewer){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") oder [OpenAPI Preview](https://marketplace.visualstudio.com/items?itemName=zoellner.openapi-preview){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") verwenden, um Ihre OpenAPI-Definition anhand einer angegebenen Spezifikationsversion zu validieren und eine Webansicht im Vorschaufenster auszugeben: 

![OpenAPI Preview](images/create-api-image1.png "OpenAPI Preview") 

Es gibt auch eine Vielzahl von browserbasierten Live-Parsing-Editoren, die Sie entweder online oder lokal verwenden können. Einige Beispiele: 

* Das [Projekt OpenAPI-GUI](https://github.com/Mermade/openapi-gui){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") unterstützt sowohl V2 als auch V3 der OpenAPI-Spezifikation und kann eine OpenAPI-Definition mit V2 für Sie in V3 migrieren. 
* [Swagger Editor von SmartBear](https://editor.swagger.io){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") unterstützt ebenfalls V2 und V3 von OpenAPI. 
* [{{site.data.keyword.apiconnect_short}}](https://cloud.ibm.com/catalog/services/api-connect){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") stellt eine Reihe von Editoren und Tools für die API-Modellierung und -Erstellung zur Verfügung. 

### API-Implementierung generieren
{: #code-first}

Sie können die Open-Source-Software [OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") verwenden, um einen Projektentwurf für Ihre Serviceimplementierung aus einer OpenAPI-Definition zu erstellen. Sie können die Sprache oder das Framework für das Gerüst über die Befehlszeile angeben. Geben Sie beispielsweise Folgendes an, um ein Java-Projekt für die Beispiel-API PetStore zu erstellen, bei der Annotationen nach der generischen JAX-RS-Methode verwendet werden: 

```bash
openapi-generator generate -g jaxrs-cxf-cdi -i ./petstore.yaml -o petstore --api-package=com.ibm.petstore
```
{: pre}

