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

Im Wesentlichen kommen die geforderten EMF basierten Komponenten zum Einsatz. Der Einsatz dieser Technologie, insbesondere EMF und Ecore als Meta-Modell betrachten wir aus Governance Gründen als vergleichbare Formate wie XML-Schema oder Json-Schema. Beide Schemas sind Beschreibungen für Datenaustauschformate. Ihre Aufgabe ist nicht Modelle zu erstellen. Der Unterschied liegt in den umfangreichen semantischen Möglichkeiten die Ecore bietet , um insbesondere fachliche Sachverhalte zu modellieren. Deshalb heißt das ni

Trotzdem soll noch einmal festgehalten werden, dass wir nicht der Meinung sind, dass bspw. QVT zwangsweise für jeden Use-Case das optimale Werkzeug sein kann. Vielmehr sollten für verschieden Use-Cases unterschiedlich Tools zum Einsatz kommen dürfen. Die Forderung, nicht zu viele Tools warten zu müssen, ist natürlich verständlich. Sie sollte, unser Meinung nach, nicht einer optimalen Umsetzung der Plattform übergeordnet sein. Eine zu starke Einschränkung der Tools / Bilbiotheken kann auch die Gefahr verstärken, diese Hilfsmittel umzubiegen oder für Anwendungsfälle zu nutzen, für die sie nicht gedacht sind. Dies führt zur einer höheren Wahrscheinlichkeit von Artificial Complexity, was wiederum dem Wartungsthema nicht zuträglich wäre. 

Wir halten es für angebracht an wichtigen Stellen im System auf die Governance zurückzufallen, während sie an anderen Stellen eher einer pragmatischen Umsetzung (bspw. Transformation auf Json-Basis) Platz machen kann. Mit der Verwendung von Ecore ist dies keine Fragen von “Entweder-Oder” sondern “Und?”. EIne EMF De-/Serialisierung von und zu Json ist unproblematisch und wird im POC auch angewendet.

## Umsetzung

Neben den bereitgestellten Komponenten sind in unserem Setup zusätzlich zwei weiter Komponenten im Einsatz:

* Model Atlas - Single Source of Truth für Modelle (ecore)
* Data Atlas - konfigurierbare ETL Pipeline

### Repository-Struktur

*README.md*
*cloudatlas/docker-compose*
*workspace/*
	*models/*
		*units.ecore*
		*glt/*
			*alarm.ecore*
			*building_sensor.ecore*
			*civitas_base.ecore*
			*glt.ecore*
			*glt-intermediate.ecore*
		*meter/*
			*meter-intermediate.ecore*
			*meter-source.ecore*
			*meter-target.ecore*
	*pipelines/*
			*glt.pipeline*
			*meter.pipeline*
			*mqttexample.pipeline*
	*trafos/*
		*glt/*
			**.qvto*
		*meter/*
			**.qvto*

Vor dem Ausführen des *docker-compose* muss sich noch auf unserer Docker-Registry angemeldet werden:

```bash
docker login devel.data-in-motion.biz:6000
Username: civitas
Password: core_v2
```

Starten des Systems

```bash
cd cloudatlas
docker compose up -d
```

## Allgemein

DIe Modelle aus dem *workspace/models*-Ordner werden in das System geladen und registriert. Modelle mit der  

### Model Atlas

Der Model Atlas ist erreichbar unter: 

* Modell-Liste - [http:localhost:8080/atlas/rest/models](http:localhost:8080/atlas/rest/models)
  * Modellübersicht und Link zu Dokumentation

* GLT:
  * GLT XMI - http://localhost:8080/glt/rest/alarm/
  * GLT Building Daten - http://localhost:8080/atlas/rest/glt/Building (Json: http://localhost:8080/atlas/rest/glt/Building?mediaType=application/json)
  * GLT Contact Daten - http://localhost:8080/atlas/rest/glt/Contact (Json:http://localhost:8080/atlas/rest/glt/Contact?mediaType=application/json )
  * GLT Mermaid - http://localhost:8080/atlas/rest/glt/documentation/html/mermaid
  * GLT Datenschutz - http://localhost:8080/atlas/rest/glt/documentation/html/gdpr
* Meter:
  * Meter Source XMI - http://localhost:8080/atlas/rest/metersource/
  * Meter Source Mermaid - http://localhost:8080/atlas/rest/metersource/documentation/html/mermaid
  * Meter Target XMI - http://localhost:8080/glt/rest/target/
  * Meter Target Mermaid - http://localhost:8080/atlas/rest/target/documentation/html/mermaid
  * Meter Target Plants - http://localhost:8080/atlas/rest/target/Plant 
  * Meter Target MeteringPoints - http://localhost:8080/atlas/rest/target/MeteringPoint
  * Mater Target MeterReading - http://localhost:8080/atlas/rest/target/MeterReading (http://localhost:8080/atlas/rest/target/MeterReading?limit=10&skip=20)




## Use-Case 1 - Meter

## Use-Case 2 - GLT