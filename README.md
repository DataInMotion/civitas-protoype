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

Im Wesentlichen kommen die geforderten EMF basierten Komponenten zum Einsatz. Der Einsatz dieser Technologie, insbesondere EMF und Ecore als Meta-Modell betrachten wir aus Governance Gründen als vergleichbare Formate wie XML-Schema oder Json-Schema. Beide Schemas sind Beschreibungen für Datenaustauschformate. Ihre Aufgabe ist nicht Modelle zu erstellen. Der Unterschied liegt in den umfangreichen semantischen Möglichkeiten die Ecore bietet , um insbesondere fachliche Sachverhalte zu modellieren. 

Trotzdem soll noch einmal festgehalten werden, dass wir nicht der Meinung sind, dass bspw. QVT zwangsweise für jeden Use-Case das optimale Werkzeug sein kann. Vielmehr sollten für verschieden Use-Cases unterschiedlich Tools zum Einsatz kommen dürfen. Die Forderung, nicht zu viele Tools warten zu müssen, ist natürlich verständlich. Sie sollte, unser Meinung nach, nicht einer optimalen Umsetzung der Plattform übergeordnet sein. Eine zu starke Einschränkung der Tools / Bibliotheken kann auch die Gefahr verstärken, diese Hilfsmittel umzubiegen oder für Anwendungsfälle zu nutzen, für die sie nicht gedacht sind. Dies führt zur einer höheren Wahrscheinlichkeit von Artificial Complexity, was wiederum dem Wartungsthema nicht zuträglich wäre. 

Wir halten es für angebracht sich an wichtigen Stellen im System auf die Governance zu berufen, während sie an anderen Stellen eher einer pragmatischen Umsetzung (bspw. Transformation auf Json-Basis) Platz machen kann. Mit der Verwendung von Ecore ist dies keine Fragen von “Entweder-Oder” sondern “Und?”. Eine EMF De-/Serialisierung von und zu Json ist unproblematisch und wird im POC auch angewendet.

Ein letzter Hinweis für die Verwendung von Ecore in Python oder TypeScript soll darauf aufmerksam machen, dass diese Technologie nicht auf Java beschränkt ist. Insbesondere in der Frontendentwicklung ist es hilfreich, wenn die UI sich nur mit dem Model Atlas verbinden muss, um Modell nachzuladen. 

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
Password: <siehe-zulip->Prototyp 2: Einreichung & Rückfragen>
```

Starten des Systems

```bash
cd cloudatlas
docker compose up -d
```

## Allgemein

DIe Modelle aus dem *workspace/models*-Ordner werden in das System geladen und registriert. 

**ACHTUNG: Der Dummy Generator erzeugt relativ viel Load. Daher sollte man darauf achten, wie lange man das System laufen lässt, da die Datenbank ggfs. schnell viele Daten erzeugt und System-Resource belegt ;-).**

### Einordnung Fennec / Geckoprojects Bibliotheken

Ziel unserer Architektur ist ein maximal integrativer Ansatz, wobei möglichst die original Bibliotheken (EMF. QVT, OCL) verwenden werden und ein Konfiguration-Layer, bzw. Dynamik-Layer (für die Verwendung in OSGi) eingezogen wurde.

In der Vergangenheit sind unser Anforderungen an einen alternative Verwendung in die Bibliotheken aufgenommen worden, sodass sie nahtlos mit unseren Wrappern funktionieren.

Kommt eine Verwendung von OSGi nicht in Betracht, würde man auf die originalen Billiotheken zurückgreifen und ggfs. eine Konfigurationsschicht einziehen. 

## Allgemeine Data-Sources und Data-Sinks

In dem Beispiel kommen MinIO, MQTT sowie Datenbankadapter zum Einsatz. Diese funktionieren für OSGi genauso wie für normales Java. Daneben gibt es weitere Adapter für AMQP oder Kafka. Dabei haben wir für Kafka einen De-/Serialisierungs Adapter, der ein und ausgehende Daten direkt bspw. aus Json Payload in EObjekte de-/serialisiert. Dadurch ist kein weiterer Boilerplate-Code nötig und man bekommt die EMF objekte direkt aus dem Kafka-Client. 

Für Datenbanken kommt hier ein relationaler Adapter auf Basis von Eclipselink zum Einsatz. Dieser verwendet im Wesentlichen die JPA (Java Persistence API) um EMF Objekte über diese Standard API laden und speichern zu können. Beide Fälle kommen im POC zum Einsatz, so beim Speichern der Meter Target Model Instanzen die aus den Datenbankabfragen und CSV Dateien gemergt wurden.

Der SensiNact Broker ist ebenso als Data-Source oder Data-Sink denkbar. In seinem Kern verwendet auch er den modell-zentrierten Datenfluss.

Grundsätzlich ist EMF mit allen denkbaren Konnektoren verwendbar. Da es in EMF eine Trennung von Objektdaten, Transport-Protokoll und Transportformat gibt, ist es extrem vielseitig anwendbar.

## Model Atlas

Der Model Atlas gibt Informationen über Modelle und, sofern aktiviert, auch Daten. Das Laden von Daten aus der Postgres ist für das *glt* und Meter *target* Model aktiviert. Entsprechend die Links zu den Objekten. 

* Modell-Liste - [http:localhost:8080/atlas/rest/models](http:localhost:8080/atlas/rest/models)
  * Modellübersicht und Link zu Dokumentation
  * ggfs. Link zu Datenendpunkten, sofern konfiguriert

Für folgende Modelle wurde die Daten API aktiviert:

* GLT:
  * GLT XMI - http://localhost:8080/glt/rest/glt/
  * GLT Building Daten - http://localhost:8080/atlas/rest/glt/Building (Json: http://localhost:8080/atlas/rest/glt/Building?mediaType=application/json)
  * GLT Contact Daten - http://localhost:8080/atlas/rest/glt/Contact (Json:http://localhost:8080/atlas/rest/glt/Contact?mediaType=application/json )
  * GLT Mermaid - http://localhost:8080/atlas/rest/glt/documentation/html/mermaid
  * GLT Datenschutz - http://localhost:8080/atlas/rest/models/gdpr?nsUri=https://civitas.org/glt/1.0.0 (hier werden die Modell-Annotation dargestellt, alle a)
* Meter:
  * Meter Target XMI - http://localhost:8080/glt/rest/target/
  * Meter Target Mermaid - http://localhost:8080/atlas/rest/target/documentation/html/mermaid
  * Meter Target Plants - http://localhost:8080/atlas/rest/target/Plant 
  * Meter Target MeteringPoints - http://localhost:8080/atlas/rest/target/MeteringPoint
  * Mater Target MeterReading - http://localhost:8080/atlas/rest/target/MeterReading (http://localhost:8080/atlas/rest/target/MeterReading?limit=10&skip=20)

Bestehende Json-Schema sollen in ihrem beschränkten Rahmen in Ecore referenziert und konvertiert werden können. Diese Feature ist im POC nicht umgesetzt aber im Generellen eingeschränkt möglich.

## Data Atlas

Unser Modell-zentrierter Ansatz schreibt keine Präferenzen vor, wie ETL abzulaufen hat. Wie bereits erwähnt lassen sich die entsprechen den Bibliotheken integrieren und konfigurieren.

Im POC sind für die beiden Use-Cases Pipelines definiert. Jeder Pipeline-Step ist eine Konfiguration. Dabei gibt es ein allgemeiners Konfigurationsmodell.

```xml
<?xml version="1.0" encoding="ASCII"?>
<pipeline:Pipeline xmi:version="2.0" xmlns:xmi="http://www.omg.org/XMI" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:ecore="http://www.eclipse.org/emf/2002/Ecore" xmlns:emfattacherconfig="https://civitas.org/handler/emf/attacher/config/1.0.0" xmlns:mqtthandler="http://civitas.org/mqtt/handler/1.0" xmlns:mqttreceiver="http://civitas.org/mqtt/receiver/1.0" xmlns:pipeline="http://civitas.org/pipline/model/1.0" xmlns:qvthandler="http://civitas.org/handler/qvt/1.0" xmlns:scheduledloaderconfig="https://civitas.org/db/loader/scheduledloader/config/1.0.0" xmlns:validationhandlerconfig="https://civitas.org/handler/validation/config/1.0" id="GLT_Pipeline">
  <steps xsi:type="mqttreceiver:MqttReceiverConfig" pid="SensorReadingReceiver" id="SensorReadingReceiver" outputs="sensor_reading_attacher" mqttTopic="buildings/#" mqttServiceTarget="(id=local)">
    <payloadEclassuri href="http://models.civitas.org/models/building/sensor/1.0#//SensorReading"/>
  </steps>
  <steps xsi:type="scheduledloaderconfig:ScheduledLoaderConfig" pid="building-glt" id="building-glt" outputs="reading_attacher" repoTarget="(repo_id=assets)" loaderName="building-glt">
    <package href="https://civitas.org/glt/1.0.0#/"/>
    <eclass href="https://civitas.org/glt/1.0.0#//Building"/>
  </steps>
  <steps xsi:type="emfattacherconfig:EMFAttacherHandlerConfig" pid="reading_attacher" id="reading_attacher" inputs="building-glt" repoTarget="(repo_id=inmem)">
    <targetEClassUri href="https://civitas.org/glt/intermediate/1.0.0#//BuildingSensorReading"/>
    <incomingEClassUri href="https://civitas.org/glt/1.0.0#//Building"/>
    <targetReferenceUri href="https://civitas.org/glt/intermediate/1.0.0#//BuildingSensorReading/building"/>
    <foreignKeyFeatureUri xsi:type="ecore:EAttribute" href="https://civitas.org/glt/1.0.0#//Building/id"/>
  </steps>
  ...
</pipeline:Pipeline>
```

Im Beispiel kann man erkennen, dass es mehrerer konkrete Konfigurationsmodelle gibt, die dann die Parameter für einen bestimmten Zweck entgegennehmen. So enthält die `MqttReceiverConfig`:

* eine Modell-Referenz, um zu wissen, welche Daten für welchen Typ de-serialisiert werden darf (Governance-Validierung)
* eine Topic auf die sich der Receive subscribed
* eine Referenz auf einen generischen MQTT Service (*id=local*). Dieser wurde separat gegen seinen MQTT Broker konfiguriert. Ein modellbasierte Konfiguration wäre auch hier möglich.

Im gleichen Stil gibt es noch weitere, eingebettete Konfiguration verschiedener konkreter Art.

### Pipeline Engine

Exemplarisch wurde im POC eine einfache, konfigurierbare Pipeline implementiert. Diese arbeitet Event-basiert über eine In-VM Broker. Entsprechend der *meter.pipeline* und *glt.pipeline* Dateien, werden die Pipeline Steps, von der Datenquelle bis zur Datensenke, nacheinander aufgerufen. Zwischen den Steps werden die EMF Objekte per Event-Bus weitergereicht. Die Topics werden dabei autoamtisch generiert und registriert. 

Die in den Pipeline Steps verwendeten Komponenten sind Service bzw. Bibliotheken und können genau so in Tools wie Apache Camel eingesetzt werden. Diese Funktionen können dann als Camel Processor oder Camel Endpoint implementiert werden. 

## Fazit

Im Wesentlichen dreht sich der modell-zentrierten Datenfluss um die Governance:  “*Wie stellen wir sicher, das nur erlaubt Modelle in unserem System agieren dürfen?*”. Ziel ist es nicht bestimmte Frameworks zur Datentransformation zu erzwingen. QVT ist bspw. eine sehr typisierte Variante, die man verwenden kann aber nicht muss. Bei komplexen Mapping ist QVT gut geeignet, bei einfachen Mappings kann der Einsatz übertrieben sein. Wichtig ist am Ende das die Validierung gegen Schemas an bestimmten Punkten notwendig sein sollte. Dafür sollte ein entsprechende Framework zur Verfügung stehen. EMF ist dafür bestens geeignet.

