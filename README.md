# CIVITAS POC Data In Motion

## Aufgabenstellung

Entsprechend der Anforderungen in:

https://gitlab.com/civitas-connect/civitas-core/requirements/-/issues?sort=updated_desc&state=opened&label_name%5B%5D=poc%3A%3Amodelling-technology&first_page_size=100

wurde eine exemplarische Anwendung entwickelt, die die beiden nachfolgenden Use-Cases umsetzt:

* [#212](https://gitlab.com/civitas-connect/civitas-core/civitas-core-v2/civitas-core-platform/-/issues/212)
* [#206](https://gitlab.com/civitas-connect/civitas-core/civitas-core-v2/civitas-core-platform/-/issues/206)

Als Grundlage diente das bereitgestellte *docker-compose* Setup mit

* MinIO
* Postgres
* Mosquitto

## Stellungnahme Kriterienkatalog

Auf Basis der definierten Anforderungen aus obigen Link wurde eine [Stellungnahme zu den Anforderung an den Prototyp](Anforderungen-Prototyp.md) erstellt. Sie beschreibt die Beweggründe diese Technologie zu wählen.

## Umsetzung

Neben den bereitgestellten Komponenten sind in unserem Setup zusätzlich zwei weiter Komponenten im Einsatz:

* Model Atlas - Single Source of Truth für Modelle (ecore)
* Data Atlas - konfigurierbare ETL Pipeline

Im Wesentlichen kommen die geforderten EMF basierten Komponenten zum Einsatz. Der Einsatz dieser Technologie, insbesondere EMF und Ecore als Meta-Modell betrachten wir aus Governance Gründen als vergleichbare Formate wie XML-Schema oder Json-Schema. Beide Schemas sind Beschreibungen für Datenaustauschformate. Ihre Aufgabe ist nicht Modelle zu erstellen. Der Unterschied liegt in den umfangreichen semantischen Möglichkeiten die Ecore bietet , um insbesondere fachliche Sachverhalte zu modellieren. Deshalb heißt das ni

Trotzdem soll noch einmal festgehalten werden, dass wir nicht der Meinung sind, dass bspw. QVT zwangsweise für jeden Use-Case das optimale Werkzeug sein kann. Vielmehr sollten für verschieden Use-Cases unterschiedlich Tools zum Einsatz kommen dürfen. Die Forderung, nicht zu viele Tools warten zu müssen, ist natürlich verständlich. Sie sollte, unser Meinung nach, nicht einer optimalen Umsetzung der Plattform übergeordnet sein. Eine zu starke Einschränkung der Tools / Bilbiotheken kann auch die Gefahr verstärken, diese Hilfsmittel umzubiegen oder für Anwendungsfälle zu nutzen, für die sie nicht gedacht sind. Dies führt zur einer höheren Wahrscheinlichkeit von Artificial Complexity, was wiederum dem Wartungsthema nicht zuträglich wäre. 

Wir halten es für angebracht an wichtigen Stellen im System auf die Governance zurückzufallen, während sie an anderen Stellen eher einer pragmatischen Umsetzung (bspw. Transformation auf Json-Basis) Platz machen kann. Mit der Verwendung von Ecore ist dies keine Fragen von “Entweder-Oder” sondern “Und?”. EIne EMF De-/Serialisierung von und zu Json ist unproblematisch und wird im POC auch angewendet.

## Use-Case 1 - Meter

## Use-Case 2 - GLT