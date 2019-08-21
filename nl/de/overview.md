---

copyright:
  years: 2019
lastupdated: "2019-07-16"

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

Gemäß der [Cloud Native Computing Foundation](https://github.com/cncf/foundation/blob/master/charter.md){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") weisen cloudnative Systeme die folgenden Attribute auf:

- Anwendungen oder Prozesse werden in Softwarecontainern als isolierte Einheiten ausgeführt.
- Prozesse werden von zentralen Orchestrierungsprozessen verwaltet, um die Ressourcennutzung zu verbessern und die Wartungskosten zu senken.
- Anwendungen oder Services (Mikroservices) sind lose mit explizit beschriebenen Abhängigkeiten gekoppelt.

Diese Attribute beschreiben ein hochdynamisches System, das sich aus unabhängigen Prozessen zusammensetzt, die zusammenarbeiten, um einen geschäftlichen Nutzen zu bieten: ein verteiltes System.

Die verteilte Datenverarbeitung ist ein Konzept mit Wurzeln, die Jahrzehnte zurückreichen. In [Fallacies of Distributed Computing](http://www.rgoarchitects.com/Files/fallacies.pdf){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") sind die folgenden Annahmen von Architekten und Entwicklern für verteilte Systeme aufgeführt, die sich am Ende als falsch erweisen. 

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

Die Methodik der [Zwölf-Faktor-Anwendung](https://12factor.net){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") wurde von Entwicklern bei Heroku entworfen. Die in den zwölf Faktoren genannten Merkmale sind nicht für einen Cloud-Provider, eine Plattform oder eine Sprache spezifisch. Die Faktoren stellen eine Reihe von Richtlinien oder bewährten Verfahren für portierbare, ausfallsichere Anwendungen dar, die für Cloudumgebungen geeignet sind (insbesondere Software-as-a-Service-Anwendungen). Die zwölf Faktoren sind in der folgenden Liste angegeben:

1. Es gibt eine Eins-zu-eins-Zuordnung zwischen einer versionsgesteuerten Codebasis wie einem Git-Repository und einem bereitgestellten Service. Dieselbe Codebasis wird für viele Bereitstellungen verwendet.
2. In den Services werden alle Abhängigkeiten explizit deklariert und das Vorhandensein von Tools oder Bibliotheken auf der Systemebene ist nicht erforderlich.
3. Die Konfigurationsdaten, die zwischen Bereitstellungsumgebungen variieren, werden in der Umgebung gespeichert, insbesondere in Umgebungsvariablen.
4. Alle Unterstützungsservices werden als angehängte Ressourcen behandelt, die durch die Ausführungsumgebung verwaltet (angehängt und abgehängt) werden.
5. Die Delivery Pipeline beinhaltet strikt voneinander getrennte Stages: Build, Release, Ausführung.
6. Anwendungen werden in Form eines statusunabhängigen Prozesses oder mehrerer statusunabhängiger Prozesse bereitgestellt. Insbesondere transiente Prozesse sind statusunabhängig und verwenden keine Ressourcen gemeinsam. Persistente Daten werden in dem jeweiligen Unterstützungsservice gespeichert.
7. Eigenständige Services machen sich selbst für andere Services verfügbar, indem sie an einem bestimmten Port empfangsbereit sind.
8. Nebenläufigkeit wird durch die Skalierung einzelner Prozesse (horizontale Skalierung) erzielt.
9. Prozesse sind verfügbar: Der schnelle Start und die ordnungsgemäße Beendigung führen zu einem stabileren und ausfallsichereren System.
10. Alle Umgebungen, von der lokalen Entwicklung bis zur Produktion, sind so ähnlich wie möglich.
11. Anwendungen erzeugen Protokolle als Ereignisströme; sie schreiben z. B. in `stdout` und `stderr` und überlassen der Ausführungsumgebung die Aggregierung der Datenströme.
12. Wenn einmalige Verwaltungstasks erforderlich sind, werden sie in der Quellcodeverwaltung aufbewahrt und zusammen mit der Anwendung gepackt, um sicherzustellen, dass sie mit derselben Umgebung wie die Anwendung ausgeführt werden.

Sie müssen diese Faktoren nicht strikt einhalten, um eine qualitativ hochwertige Mikroserviceumgebung zu erzielen. Sie helfen Ihnen jedoch, portierbare Anwendungen und Services in Continuous Delivery-Umgebungen zu erstellen und zu pflegen.

## Anwendungen
{: #apps-intro}

Eine App bietet Code, Daten, Services und Toolchains. Beispielsweise enthält die mobile {{site.data.keyword.cloud_notm}}-App Einheitencode sowie Back-End-Logik, Analyse- und Sicherheitsservices und ist für Continuous Delivery eingerichtet.

![Wiederverwenden](images/garage_reuse2.png "Mit Developer Experience können Sie Funktionalität wiederverwenden und eine Neuerfindung vermeiden")

Sie können eine App über ein beliebiges {{site.data.keyword.cloud_notm}}-Entwicklerportal oder über die {{site.data.keyword.dev_cli_notm}} erstellen und verwalten.

Sie können entweder einfache leere Apps direkt erstellen oder mithilfe von Starter-Kits komplexere Apps generieren. Wenn Sie leere Apps ohne die Hilfe eines Starter-Kits erstellen möchten, können Sie dies über das [{{site.data.keyword.cloud_notm}}-Dashboard ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")](https://{DomainName}){: new_window} tun, ohne dass Sie ein Portal aufrufen müssen.

Sie können ein Codemuster verwenden, um Ihre App rasch zu erstellen und in {{site.data.keyword.cloud_notm}} bereitzustellen. Wählen Sie auf der [IBM Developer-Website ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")](https://developer.ibm.com/patterns/){:new_window} ein Codemuster aus. Sie können den Code entweder in GitHub anzeigen oder eine App auf {{site.data.keyword.cloud_notm}} erstellen und dort ihren Build erstellen. Außerdem können Sie dort Ihre App mithilfe einer DevOps-Toolchain automatisch bereitstellen.

## Mikroservices
{: #microservices}

Ein *Mikroservice* ist eine Gruppe von kleinen, unabhängigen Architekturkomponenten, die über eine gemeinsame einfache API kommunizieren. Jeder Mikroservice im folgenden einfachen Beispiel ist eine Zwölf-Faktor-Anwendung, die austauschbare Unterstützungsservices verwendet, um Daten zu speichern und Nachrichten zu übergeben:

![Eine Mikroserviceanwendung](images/microservice.png "Eine Mikroserviceanwendung")

Mikroservices sind unabhängig. Agilität ist einer der Vorteile von Mikroservicearchitekturen. Sie ist jedoch nur vorhanden, wenn Services neu geschrieben werden können, ohne dass andere Services beeinträchtigt werden. 

Klare API-Grenzen sorgen dafür, dass die Teams, die einen Service bearbeiten, eine sehr hohe Flexibilität bei der Weiterentwicklung der Implementierung genießen. Dieses Merkmal ermöglicht die polyglotte Programmierung und Persistenz.

Mikroservices sind ausfallsicher. Die Anwendungsstabilität hängt davon ab, dass die einzelnen Mikroservices gegen Fehler widerstandsfähig sind. Dies ist ein bedeutsamer Unterschied zu traditionellen Architekturen, in denen die unterstützende Infrastruktur Fehler für die Anwendung verarbeitet. Jeder Service muss Isolierungsmuster wie Circuit Breakers und Bulkheads anwenden, um Fehler einzugrenzen und ein geeignetes Fallback-Verhalten zu definieren und dadurch vorgelagerte Services zu schützen.

Mikroservices sind statusunabhängig und werden in externen, unterstützenden Cloud-Services wie Redis gespeichert. Der schnelle Start und die ordnungsgemäße Beendigung tragen zudem dazu bei, dass Mikroservices sich gut für automatisierte Umgebungen eigenen, in denen Instanzen als Reaktion auf Last oder zur Aufrechterhaltung eines einwandfreien Systemstatus erstellt und entfernt werden.

### Bedeutung von "klein"
{: #small-microsvc}

Das Wort "klein" bedeutet im Zusammenhang mit einem Mikroservice, dass er fokussiert ist. In vielen Beschreibungen werden Parallelen zwischen den Rollen einzelner Mikroservices und verketteten Befehlen in der UNIX-Befehlszeile gezogen:

```
ls | grep 'service' | sort -r
```
{:pre}

Diese UNIX-Befehle führen jeweils ganz unterschiedliche Aufgaben aus und können unabhängig von Programmiersprache oder Codemenge miteinander verkettet werden.

## Polyglotte Anwendungen: das richtige Tool für die Aufgabe auswählen
{: #polyglot-apps}

Die Mehrsprachigkeit ist ein häufig genannter Vorteil einer Architektur, die auf Mikroservices basiert. Finden Sie einen Mittelweg, indem Sie eine Liste der zur Auswahl stehenden unterstützten Technologien erstellen und eine Richtlinie für die spätere Erweiterung der Liste mit neuen Technologien definieren. Stellen Sie sicher, dass nicht funktionale Anforderungen oder die Einhaltung gesetzlicher Bestimmungen wie Wartbarkeit, Überprüfbarkeit und Datensicherheit gewährleistet sind, während gleichzeitig die Agilität erhalten bleibt und Innovation durch Experimente unterstützt wird.

... :FIXME: unpack? Language table here?:...

## REST und JSON
{: #rest-json}

Polyglotte Anwendungen sind nur mit sprachunabhängigen Protokollen möglich. REST-Architekturmuster definieren Richtlinien für die Erstellung einheitlicher Schnittstellen, die die Darstellung der Daten während der Übertragung von der Implementierung des Service trennen.

Bei Mikroservicearchitekturen hat JSON sich als das Sendeformat der Wahl für textbasierte Daten etabliert und mit seiner relativen Einfachheit und Kompaktheit XML verdrängt. Zum Vergleich enthält das folgende Beispiel einen Basisdatensatz mit Daten zu einem Mitarbeiter in JSON:

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

In JSON werden Attribut/Wert-Paare für die Darstellung von Datenobjekten in einer prägnanten Syntax verwendet, die Informationen zu einigen Basistypen wie Zahlen, Zeichenfolgen, Arrays und Objekten beibehält. Sowohl in JSON als auch in XML ist das verschachtelte Adressobjekt im obigen Beispiel deutlich zu erkennen; jedoch ist das zugehörige XML-Schema erforderlich, um den Typ des Elements `serial` zu bestimmen. In JSON ist aus der Syntax ersichtlich, dass der Wert von `serial` eine Zahl und keine Zeichenfolge ist.
