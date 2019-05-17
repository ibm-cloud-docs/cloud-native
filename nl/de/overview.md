---

copyright:
  years: 2019
lastupdated: "2019-04-08"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Was ist cloudnativ? 
{: #overview}

Cloud-Computing-Umgebungen sind dynamisch und umfassen die Zuordnung und Freigabe von Ressourcen aus einem virtualisierten, gemeinsamen Pool nach Bedarf. Im Vergleich zur Ressourcenzuordnung im Vorfeld, die typischerweise in traditionellen On-Premises-Rechenzentren verwendet wird, ermöglichen diese elastischen Umgebungen flexiblere Skalierungsoptionen.
{:shortdesc}

Gemäß der [Cloud Native Computing Foundation](https://cncf.io/about/charter){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") weisen cloudnative Systeme die folgenden Attribute auf: 

- Anwendungen oder Prozesse werden in Softwarecontainern als isolierte Einheiten ausgeführt. 
- Prozesse werden von zentralen Orchestrierungsprozessen verwaltet, um die Ressourcennutzung zu verbessern und die Wartungskosten zu reduzieren. 
- Anwendungen oder Services (Mikroservices) sind lose mit explizit beschriebenen Abhängigkeiten gekoppelt. 

Diese Attribute beschreiben ein hochdynamisches System, das aus unabhängigen Prozessen besteht, die zusammenarbeiten, um einen geschäftlichen Nutzen bereitzustellen: ein verteiltes System. 

Die verteilte Datenverarbeitung ist ein Konzept mit Wurzeln, die Jahrzehnte zurückreichen. In [Fallacies of Distributed Computing](http://www.rgoarchitects.com/Files/fallacies.pdf){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") sind die folgenden Annahmen von Architekten und Entwicklern für verteilte Systeme aufgeführt, die sich langfristig als falsch erweisen.  

* Das Netz ist zuverlässig. 
* Das Netz ist sicher. 
* Das Netz ist homogen. 
* Die Latenz ist null. 
* Die Bandbreite ist unendlich. 
* Die Topologie ändert sich nicht. 
* Es gibt einen einzigen Administrator. 
* Die Transportkosten sind null. 

Cloudtechnologien wie Kubernetes und Istio zielen darauf ab, diese Probleme in der Infrastruktur selbst zu lösen. 

## Zwölf Faktoren
{: #twelve-factors}

Die Methodik der [Zwölf-Faktor-Anwendung](http://12factor.net){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") wurde von Entwicklern bei Heroku entworfen. Die in den zwölf Faktoren genannten Merkmale sind nicht für einen Cloud-Provider, eine Plattform oder eine Sprache spezifisch. Die Faktoren stellen eine Reihe von Richtlinien oder bewährten Verfahren für portierbare, ausfallsichere Anwendungen dar, die für Cloudumgebungen geeignet sind (insbesondere Software-as-a-Service-Anwendungen). Die zwölf Faktoren sind in der folgenden Liste angegeben: 

1. Es gibt eine Eins-zu-eins-Zuordnung zwischen einer versionsgesteuerten Codebasis wie einem Git-Repository und einem bereitgestellten Service. Dieselbe Codebasis wird für viele Bereitstellungen verwendet. 
2. In den Services werden alle Abhängigkeiten explizit deklariert und das Vorhandensein von Tools oder Bibliotheken auf der Systemebene ist nicht erforderlich. 
3. Die Konfigurationsdaten, die zwischen Bereitstellungsumgebungen variieren, werden in der Umgebung gespeichert, insbesondere in Umgebungsvariablen. 
4. Alle Unterstützungsservices werden als angehängte Ressourcen behandelt, die durch die Ausführungsumgebung verwaltet (angehängt und abgehängt) werden. 
5. Die Delivery Pipeline beinhaltet strikt voneinander getrennte Phasen: Build, Release, Ausführung. 
6. Anwendungen werden in Form eines statusunabhängigen Prozesses oder mehrerer statusunabhängiger Prozesse bereitgestellt. Insbesondere transiente Prozesse sind statusunabhängig und verwenden keine Ressourcen gemeinsam. Persistente Daten werden in dem jeweiligen Unterstützungsservice gespeichert. 
7. Eigenständige Services machen sich selbst für andere Services verfügbar, indem sie an einem bestimmten Port empfangsbereit sind. 
8. Nebenläufigkeit wird durch die Skalierung einzelner Prozesse (horizontale Skalierung) erzielt. 
9. Prozesse sind verfügbar: Der schnelle Start und die ordnungsgemäße Beendigung führen zu einem stabileren und ausfallsichereren System. 
10. Alle Umgebungen, von der lokalen Entwicklung bis zur Produktion, sind so ähnlich wie möglich. 
11. Anwendungen erzeugen Protokolle als Ereignisströme; sie schreiben z. B. in `stdout` und `stderr` und überlassen der Ausführungsumgebung die Aggregierung der Datenströme. 
12. Wenn einmalige Verwaltungstasks erforderlich sind, werden sie in der Quellcodeverwaltung aufbewahrt und zusammen mit der Anwendung gepackt, um sicherzustellen, dass sie mit derselben Umgebung wie die Anwendung ausgeführt werden. 

Sie müssen diese Faktoren nicht strikt einhalten, um eine qualitativ hochwertige Mikroserviceumgebung zu erzielen. Die Orientierung an diesen Faktoren hilft Ihnen jedoch, portierbare Anwendungen und Services in Continuous Delivery-Umgebungen zu erstellen und zu pflegen. 

## Mikroservices
{: #microservices}

Ein *Mikroservice* ist eine Gruppe von kleinen, unabhängigen Architekturkomponenten, die jeweils einen einzigen Zweck haben und über eine gemeinsame einfache API kommunizieren. Jeder Mikroservice im folgenden einfachen Beispiel ist eine Zwölf-Faktor-Anwendung, die austauschbare Unterstützungsservices verwendet, um Daten zu speichern und Nachrichten zu übergeben. 

![Eine Mikroserviceanwendung](images/microservice.png "Eine Mikroserviceanwendung")

Mikroservices sind unabhängig. Agilität ist einer der Vorteile von Mikroservicearchitekturen. Sie ist jedoch nur vorhanden, wenn Services vollständig neu geschrieben werden können, ohne dass andere Services beeinträchtigt werden. Das ist wahrscheinlich nur selten der Fall, dient aber zur Erklärung der Anforderung. Klare API-Grenzen sorgen dafür, dass das Team, das einen Service bearbeitet, eine sehr hohe Flexibilität bei der Weiterentwicklung der Implementierung genießt. Dieses Merkmal ermöglicht die polyglotte Programmierung und Persistenz. 

Mikroservices sind ausfallsicher. Die Anwendungsstabilität hängt davon ab, dass die einzelnen Mikroservices gegen Fehler widerstandsfähig sind. Dies ist der große Unterschied zu traditionellen Architekturen, in denen die unterstützende Infrastruktur Fehler für die Anwendung verarbeitet. Jeder Service muss Isolierungsmuster wie Circuit Breakers und Bulkheads anwenden, um Fehler einzugrenzen und ein geeignetes Fallback-Verhalten zu definieren und dadurch vorgelagerte Services zu schützen. 

Mikroservices sind statusunabhängige, transiente Prozesse. Dies bedeutet nicht, dass Mikroservices keinen Status haben können! Es bedeutet, dass der Status in externen, unterstützenden Cloud-Services wie Redis und nicht speicherintern gespeichert werden sollte. Der schnelle Start und die ordnungsgemäße Beendigung tragen zudem dazu bei, dass Mikroservices sich gut für automatisierte Umgebungen eigenen, in denen Instanzen als Reaktion auf Last oder zur Aufrechterhaltung eines einwandfreien Systemstatus erstellt und gelöscht werden. 

### Bedeutung von "klein"
{: #small-microsvc}

Das Wort "klein" bedeutet im Zusammenhang mit einem Mikroservice, dass er fokussiert ist: er dient einem einzigen Zweck und soll diese Aufgabe gut erledigen. In vielen Beschreibungen werden Parallelen zwischen den Rollen einzelner Mikroservices und verketteten Befehlen in der UNIX-Befehlszeile gezogen: 

```
ls | grep 'service' | sort -r
```
{:pre}

Diese UNIX-Befehle führen jeweils ganz unterschiedliche Aufgaben aus und können unabhängig von Programmiersprache oder Codemenge miteinander verkettet werden. 

## Polyglotte Anwendungen: das richtige Tool für die Aufgabe auswählen
{: #polyglot-apps}

Die Mehrsprachigkeit ist ein häufig genannter Vorteil einer Architektur, die auf Mikroservices basiert. Einerseits kann die Auswahl der geeigneten Sprache oder des geeigneten Datenspeichers für die Funktion, die ein Service bereitstellt, sehr leistungsfähig sein und die Effizienz enorm steigern. Andererseits kann die Verwendung von undurchsichtigen Technologien die Langzeitpflege des Codes und den Wechsel von Entwicklern in ein anderes Team erschweren.  

Finden Sie einen Mittelweg, indem Sie zu Anfang eine Liste der zur Auswahl stehenden unterstützten Technologien erstellen und eine Richtlinie für die spätere Erweiterung der Liste mit neuen Technologien definieren. Stellen Sie sicher, dass nicht funktionale Anforderungen oder die Einhaltung gesetzlicher Bestimmungen wie Wartbarkeit, Überprüfbarkeit und Datensicherheit gewährleistet sind, während gleichzeitig die Agilität erhalten bleibt und Innovation durch Experimente unterstützt wird. 

## REST und JSON
{: #rest-json}

Polyglotte Anwendungen sind nur mit sprachunabhängigen Protokollen möglich. REST-Architekturmuster definieren Richtlinien für die Erstellung einheitlicher Schnittstellen, die die Darstellung der Daten während der Übertragung von der Implementierung des Service trennen. 

Bei Mikroservicearchitekturen hat JSON sich als das Sendeformat der Wahl für textbasierte Daten entwickelt und mit seiner relativen Einfachheit und Kompaktheit XML verdrängt. Zum Vergleich enthält das folgende Beispiel einen Basisdatensatz mit Daten zu einem Mitarbeiter in JSON: 

  ```json
{
  "name": "Marley Cassin",
  "serial": 228264,
  "title": "Senior Software Engineer",
  "address": {
    "office": "501-B101",
    "street": "3858 Kuvalis Pass",
    "city": "East Craig",
    "state": "NC",
    "zip": "64519-8934"
  }
}
```
{: codeblock}

Und das folgende Beispiel zeigt denselben Mitarbeiterdatensatz in XML:

```xml
<person>
  <name>Marley Cassin</name>
  <serial>228264</serial>
  <title>Senior Software Engineer</title>
  <address>
    <office>501-B101</office>
    <street>3858 Kuvalis Pass</street>
    <city>East Craig</city>
    <state>NC</state>
    <zip>64519-8934</zip>
  </address>
</person>
```
{: codeblock}

In JSON werden Attribut/Wert-Paare für die Darstellung von Datenobjekten in einer prägnanten Syntax verwendet, die Informationen zu einigen Basistypen wie Zahlen, Zeichenfolgen, Arrays und Objekten beibehält. Sowohl in JSON als auch in XML ist das verschachtelte Adressobjekt im obigen Beispiel deutlich zu erkennen; jedoch ist das zugehörige XML-Schema erforderlich, um den Typ des Elements 'serial' zu bestimmen. In JSON ist aus der Syntax ersichtlich, dass der Wert von 'serial' eine Zahl und keine Zeichenfolge ist.
