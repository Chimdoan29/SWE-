
# Smart Home Dashboard

# Inhaltsverzeichnis

```text
1. Vorstellung des Produktes ........................................ 5
    1.1 Einleitung ................................................... 5
        1.1.1 Abstract ............................................... 5
        1.1.2 Produktbeschreibung und Zielgruppe ...................... 6

    1.2 Systemarchitektur ............................................ 7
        1.2.2 Prozessdiagramm ........................................ 11
        1.2.3 Beschreibung der persistenten Daten ..................... 15
        1.2.4 Beschreibung der transienten Daten und Schnittstellen ... 26

    1.3 Mockups ...................................................... 30
        1.3.1 Anmeldemaske ........................................... 30
        1.3.2 Hauptseite nach der Anmeldung ........................... 31
                Bild 12: Hauptseite nach der Anmeldung ................. 31
        1.3.3 Location anzeigen ....................................... 32
                Bild 13: Location anzeigen .............................. 32
        1.3.4 Smart Devices aufgelistet anzeigen ...................... 33
        1.3.4 Verbrauchsübersicht ..................................... 33
        1.3.5 Analyse ................................................. 35
                Bild 16: Analyse ........................................ 35

    1.4 Lastenheft ................................................... 36
        1.4.1 Nicht-Funktionale Anforderungen ......................... 36
                NFA1: Benutzerfreundlichkeit (Usability) ................ 36
                NFA2: Performance ....................................... 36

        1.4.2 Funktionale Anforderungen ............................... 37
        1.4.2.1 Erfassung & Übertragung von Messdaten ................. 37
        1.4.2.2 Validierung von Messdaten ............................. 39
        1.4.2.3 Speicherung von Messdaten ............................. 41
        1.4.2.4 Echtzeitvisualisierung ................................ 43

2. Qualitätssicherung ................................................ 45
    2.1 Beschreibung der Umgebungen und CI-Pipeline ................... 45
        2.1.1 CI-Pipeline (Continuous Integration) .................... 45
    2.2 Testverfahren ................................................. 46
    2.3 Testvorlagen .................................................. 47
    2.4 Mögliche Defekte .............................................. 51

3. Projektstruktur und Ressourcenplanung .............................. 52
    3.1 Vorgehensweise ................................................ 52
    3.2 Work Breakdown ................................................ 52
        3.2.1 Stakeholder und Verantwortlichkeiten .................... 52
        3.2.2 RACI-Matrix ............................................. 54
                Legende der Rollen ...................................... 54
                Beschreibung der Phasenübergänge (Quality Gates) ........ 55
                Übergang 1 – Quality Gate 1 ............................. 55
                Übergang 2 – Quality Gate 2 ............................. 55
                Übergang 3 – Quality Gate 3 ............................. 55
                Übergang 4 – Quality Gate 4 ............................. 55

    3.3 Sprint-Planung und agiler Methodeneinsatz ..................... 56
    3.4 Entwicklungs- und Testanforderungen ........................... 57
    3.5 Marktabschätzung und Finanzierung ............................. 58
```
## 1. Einleitung & Produktvorstellung
### 1.1 Einleitung
#### 1.1.1 Abstract
Das Projekt **Smart Home Dashboard** befasst sich mit der Konzeption einer modernen, webbasierten Softwarelösung zur Überwachung, Analyse und Visualisierung des Energieverbrauchs in privaten Haushalten.

<img width="700" alt="Mindmap" src="https://github.com/user-attachments/assets/f363a06b-4bb4-4682-9efd-8708568ea114" />

Bild: Mindmap

Ziel der Anwendung ist es, Nutzer:innen eine transparente und verständliche Übersicht über ihren aktuellen sowie historischen Energieverbrauch bereitzustellen. Dadurch soll ein bewussteres und nachhaltigeres Energieverhalten gefördert werden.

**Kernfunktionen:**
* **Datenerfassung:** Intelligente Messgeräte (Smart Plugs, Temperatursensoren) erfassen Daten in regelmäßigen Intervallen.
* **Übertragung:** Nutzung standardisierter Protokolle wie MQTT oder HTTP.
* **Verarbeitung:** Serverseitige Validierung, Speicherung (JSON) und Aufbereitung.
* **Visualisierung:** Übersichtliche Diagramme, zeitbasierte Vergleiche (Tag/Woche/Monat) und Durchschnittswerte.
* **Optimierung:** Erkennung hoher Verbräuche und Identifikation von Einsparpotenzialen zur Steigerung der Energieeffizienz und Kostensenkung.

#### 1.1.2 Produktbeschreibung
Das Ergebnis ist ein vollständig ausgearbeitetes Konzept inklusive Systemarchitektur, Datenflüssen und Anforderungen. Das Dashboard dient als zentrale Informationsgrundlage für Nutzer zur Optimierung ihres Energieverbrauchs.

### 1.2 Systemarchitektur
<img width="700" alt="Screenshot 2026-01-22 132318" src="https://github.com/user-attachments/assets/c1597861-2303-4ec2-b067-176f40282235" />

Bild 1: Systemarchitektur Diagramm

Das System ist in vier logische Schichten unterteilt, um eine skalierbare Datenverarbeitung von der Hardware bis zum Frontend zu gewährleisten.

#### Home Network (IoT Devices)
Die IoT-Geräte erfassen physikalische Messwerte im lokalen Heimnetzwerk.
* **Hardware:** ESP32-basierte Sensoren oder Smart Plugs (ggf. mit RIOT-OS).
* **Aufgaben:** Periodische Erfassung von Spannung, Leistung, Energieverbrauch oder Temperatur.
* **Kommunikation:** Via WLAN unter Nutzung von **Matter** oder **Zigbee** zum Gateway.
* **Output:** Rohmessdaten.

#### Gateway Layer (Umsetzer)
Das Gateway fungiert als Brücke zwischen den IoT-Protokollen und dem HTTP-basierten Backend.
* **Aufgaben:**
    * Übersetzung von Matter/Zigbee zu HTTP.
    * Vereinheitlichung der Datenformate und Normalisierung von Zeitstempeln.
    * Mapping in ein einheitliches Datenmodell.
* **Kommunikation:** Push via HTTP (JSON) an das Backend.

#### Backend (Verarbeitungssystem)
Das Herzstück der Anwendung, unterteilt in mehrere Module:

* **API Layer:**
    * `POST /ingest`: Empfang von JSON-Messdaten vom Gateway.
    * `GET /measurements`, `/devices`, `/alerts`: Datenabruf für das Frontend.
* **Service Layer:** Durchführung von Schema-Validierung, Plausibilitätsprüfungen und Aggregationen (Mittelwerte, Zeitfenster).
* **Alerting-Komponente:** Überwachung von Schwellenwerten (z. B. >, <, ≥) und Auslösung von Ereignissen bei Regelverletzung.
* **Persistenz (Datenbanken):**
    * **TSDB (Time Series Database):** Für hochfrequente Zeitreihendaten.
    * **RDBMS (Relationale DB):** Für Stammdaten (Nutzer, Geräte, Regeln, Abonnements).

#### Client (Dashboard)
Die Benutzeroberfläche für den Endnutzer.
* **Funktionen:** Live-Monitoring, Historische Analyse, Alarmvisualisierung.
* **Kommunikation:** REST-Requests (JSON) an das Backend.

#### 1.2.1 Datenmodell (Klassendiagramm)

<img width="700" height ="400" alt="KlassenDia" src="https://github.com/user-attachments/assets/14027e96-07ce-4a20-adf1-d55b0f681ee1" />

Bild 2: Klassendiagramm

#### Benutzer & Haushalte
* **User** (`user_id`, `email`, `name`, `created_at`)
    * *Beziehung:* Ein Benutzer kann Mitglied in mehreren Haushalten sein.
* **Household** (`household_id`, `name`, `created_at`)
    * *Beschreibung:* Ein Haushalt gruppiert Geräte und Sensoren.
* **HouseholdMember** (`household_id`, `user_id`, `role`, `joined_at`)
    * *Funktion:* Realisierung einer n:m-Beziehung zwischen User und Household (Rollen: `owner`, `member`).

#### Geräte & Sensordaten
* **Device** (`device_id`, `household_id`, `name`, `device_type`, `protocol`, `location`, `is_active`)
    * *Beziehung:* Ein Gerät gehört zu genau einem Haushalt.
    * *Details:* Protokolle (Matter, Zigbee), Typen (Smart Plug, Sensor).
* **Sensor** (`sensor_id`, `device_id`, `metric_type`, `unit`, `sampling_rate_s`)
    * *Beziehung:* Ein Gerät kann mehrere Sensoren besitzen (z. B. Leistung in W, Energie in kWh).
* **Measurement** (`measurement_id`, `sensor_id`, `ts`, `value`, `quality_flag`)
    * *Funktion:* Speichert Zeitreihenmessungen.
    * *Flags:* `ok`, `estimated`, `outlier`.

#### Abrechnung (Subscription)
* **Subscription** (`subscription_id`, `user_id`, `plan_name`, `included_households`, `status`, `started_at`)
    * *Status:* `active`, `paused`, `canceled`.
* **SubscriptionAddOn** (`addon_id`, `subscription_id`, `addon_type`, `quantity`)
    * *Funktion:* Kostenpflichtige Erweiterungen eines Abonnements.

#### Alarmsystem
* **AlertRule** (`rule_id`, `household_id`, `sensor_id`, `rule_name`, `threshold`, `operator`, `duration_s`)
    * *Logik:* Definiert Schwellenwerte (Operatoren: `>`, `<`, `≥`, `≤`).
* **AlertEvent** (`event_id`, `rule_id`, `sensor_id`, `ts`, `observed_value`, `status`)
    * *Status:* `open`, `acknowledged`, `resolved`.
* **Benutzer & Haushalte**

    * **User** (`user_id`, `email`, `name`, `created_at`)
        * *Beziehung:* Ein Benutzer kann Mitglied in mehreren Haushalten sein.

    * **Household** (`household_id`, `name`, `created_at`)
        * *Beschreibung:* Ein Haushalt gruppiert Geräte und Sensoren.

    * **HouseholdMember** (`household_id`, `user_id`, `role`, `joined_at`)
        * *Funktion:* Realisierung einer n:m-Beziehung zwischen User und Household (mit Rollen wie `owner`, `member`).

* **Geräte & Sensordaten**

    * **Device** (`device_id`, `household_id`, `name`, `device_type`, `protocol`, `location`, `is_active`, `created_at`)
        * *Beziehung:* Ein Gerät gehört zu genau einem Haushalt.
        * *Typen:* z. B. `smart_plug`, `sensor`. Protokolle: `matter`, `zigbee`.

    * **Sensor** (`sensor_id`, `device_id`, `metric_type`, `unit`, `sampling_rate_s`, `created_at`)
        * *Beziehung:* Ein Gerät kann mehrere Sensoren besitzen.

    * **Measurement** (`measurement_id`, `sensor_id`, `ts`, `value`, `quality_flag`)
        * *Funktion:* Speichert Zeitreihenmessungen eines Sensors.
        * *Qualitäts-Flags:* `ok`, `estimated`, `outlier`.

* **Abrechnung (Subscription)**

    * **Subscription** (`subscription_id`, `user_id`, `plan_name`, `included_households`, `status`, `started_at`)
        * *Status:* `active`, `paused`, `canceled`.

    * **SubscriptionAddOn** (`addon_id`, `subscription_id`, `addon_type`, `quantity`)
        * *Funktion:* Erweitert ein bestehendes Abonnement.

* **Alarmsystem**

    * **AlertRule** (`rule_id`, `household_id`, `sensor_id`, `rule_name`, `threshold`, `operator`, `duration_s`, `is_active`, `created_at`)
        * *Logik:* Definiert Schwellenwerte (Operatoren: `>`, `<`, `≥`, `≤`).

    * **AlertEvent** (`event_id`, `rule_id`, `sensor_id`, `ts`, `observed_value`, `status`)
        * *Status:* `open`, `acknowledged`, `resolved`.

#### Zentrale Beziehungen
Das Entity-Relationship-Modell beschreibt die persistente Datenstruktur des Systems. Es ist logisch in **Stammdaten**, **Bewegungsdaten** und **Geschäftsdaten** gegliedert und unterstützt folgende Kernfunktionen:
* **Multi-Tenant-Fähigkeit:** Verwaltung von Benutzern und Haushalten.
* **Gerätemanagement:** Verwaltung von Smart-Home-Geräten und deren Sensoren.
* **Zeitreihen:** Speicherung hochfrequenter Messdaten in einer spezialisierten TSDB.
* **Alarmierung:** Definition von Regeln und Verwaltung von Ereignissen.
* **Billing:** Abbildung von Abonnements und Zusatzfunktionen.

### 1.2.2 Prozessdiagramm

<img width="700" alt="ProzessDia" src="https://github.com/user-attachments/assets/55c8d956-a443-493e-9c93-7d33efef86d9" />

Bild: Prozessdiagramm

#### Beschreibung

Der Prozess beschreibt den vollständigen Ablauf von der Erfassung der Messdaten auf dem IoT-Gerät bis zur Visualisierung und Alarmanzeige im Dashboard.

#### IoT Device

Das IoT-Gerät erfasst physikalische Messwerte aus der Umgebung.

**Aufgaben:**
- Erfassung von Messwerten (z. B. Temperatur, Leistung, Energieverbrauch)
- Zeitliche Abtastung der Sensorwerte
- Bereitstellung der Rohdaten zur Übertragung

**Output:**
- Rohmessdaten

#### Gateway / Converter

Der Gateway empfängt die Rohdaten vom IoT-Gerät und bereitet sie für die Backend-Verarbeitung auf.

**Aufgaben:**
- Empfang der Messdaten über WiFi mit Matter/Zigbee
- Normalisierung der gerätespezifischen Daten
- Mapping auf ein einheitliches JSON-Format
- Weiterleitung der Daten per HTTP POST an `/ingest`

**Output:**
- Vereinheitlichter JSON-Payload

#### Backend API

Die Backend API dient als zentrale Schnittstelle für die Datenannahme und Weiterleitung.

**Aufgaben:**
- Entgegennahme der Messdaten vom Gateway
- Weiterleitung der Daten an die Service Layer
- Asynchrone Bestätigung des Empfangs (`202 Accepted`)
- Fehlerweitergabe bei ungültigen Anfragen

**Output:**
- HTTP-Statuscodes (`202 Accepted`, `400 Bad Request`)

#### Service Layer

Die Service Layer verarbeitet und validiert die eingehenden Messdaten.

**Aufgaben:**
- Schema-Validierung der JSON-Payloads
- Plausibilitätsprüfung der Messwerte
- Optionale Aggregation (z. B. Zeitfenster, Mittelwerte)
- Vorbereitung der Daten für die Persistenz
- Auflösung von Geräte- und Sensorzuordnungen

**Output:**
- Validierte und aufbereitete Messdaten

#### Persistenz

Das System nutzt zwei unterschiedliche Datenbanksysteme zur optimierten Speicherung.

**TSDB (Time-Series Database):**
- Speicherung hochfrequenter Zeitreihendaten
- Persistenz von Messwerten (Sensor-ID, Zeitstempel, Wert, Einheit)

**RDBMS (Relationale Datenbank):**
- Speicherung von Stammdaten (Benutzer, Haushalte, Geräte, Sensoren)
- Speicherung von Alarmregeln und Alarmereignissen

#### Alerting-Komponente

Die Alerting-Komponente überwacht Messwerte und erkennt Regelverletzungen.

**Aufgaben:**
- Laden aktiver Alarmregeln pro Sensor und Haushalt
- Prüfung von Schwellenwerten und Zeitbedingungen
- Erzeugung von Alarmereignissen bei Regelverletzung
- Persistenz der Alarmereignisse im RDBMS

**Output:**
- Alarmereignisse für die Dashboard-Visualisierung


### Dashboard (Client)

Das Dashboard visualisiert Messdaten und Alarmereignisse für den Benutzer.

**Aufgaben:**
- Abfrage von Messdaten (`GET /measurements`)
- Abfrage von Alarmen (`GET /alerts`)
- Darstellung von Live- und historischen Daten
- Anzeige von Warnungen und Alarmen

**Output:**
- Visualisierte Messdaten und Alarmmeldungen


#### 1.2.3 Entity Relationship Modell

<img width="700" alt="ERDia" src="https://github.com/user-attachments/assets/c9685d63-8e54-454e-96f5-b8014b020761" />

Bild 3: ER Modell
#### Zweck des Datenmodells
* **User ↔ Household:** n:m (realisiert über `HouseholdMember`).
* **Household → Device:** 1:n
* **Device → Sensor:** 1:n
* **Sensor → Measurement:** 1:n
* **User → Subscription:** 1:n
* **Household → AlertRule:** 1:n

#### Domänenobjekte (Entities)

* **USER**
    * *Beschreibung:* Repräsentiert einen registrierten Plattformnutzer.
    * *Verantwortung:* Identität, Kontaktinformationen, Vertragsbeziehungen (Abonnements) und Haushalts-Mitgliedschaften.

* **HOUSEHOLD**
    * *Beschreibung:* Repräsentiert eine organisatorische Einheit (z. B. Wohnung oder Gebäude).
    * *Verantwortung:* Gruppierung von Geräten/Sensoren und Zugriffskontrolle.

* **HOUSEHOLD_MEMBER**
    * *Beschreibung:* Assoziationsentität für die n:m-Beziehung zwischen User und Household.
    * *Verantwortung:* Rollenverwaltung (z. B. Admin, Member) und Nachvollziehbarkeit der Mitgliedschaft.

* **DEVICE**
    * *Beschreibung:* Repräsentiert ein physisches Smart-Home-Gerät.
    * *Verantwortung:* Technische Identifikation, Protokollzuordnung (Matter, Zigbee) und Zuordnung zum Haushalt.

* **SENSOR**
    * *Beschreibung:* Repräsentiert eine messende Einheit innerhalb eines Geräts.
    * *Verantwortung:* Definition der Messgröße (Metrik, Einheit), Abtastrate und Quelle für die Alarmüberwachung.

* **MEASUREMENT**
    * *Beschreibung:* Repräsentiert einen einzelnen Messwert eines Sensors.
    * *Verantwortung:* Persistenz zeitbasierter Sensordaten als Grundlage für Visualisierung und Analyse.
    * *Technik:* Optimiert für hochfrequente Schreibzugriffe (Time-Series-Datenbank).


* **HOUSEHOLD_MEMBER**
    * *Beschreibung:* Assoziationsentität zur Abbildung der N:M-Beziehung zwischen USER und HOUSEHOLD.
    * *Verantwortung:*
        * Rollenverwaltung (z. B. Admin, Member)
        * Nachvollziehbarkeit der Mitgliedschaft

* **DEVICE**
    * *Beschreibung:* Repräsentiert ein physisches Smart-Home-Gerät.
    * *Verantwortung:*
        * Technische Identifikation des Geräts
        * Protokollzuordnung (Matter, Zigbee, etc.)
        * Zuordnung zu einem Haushalt

* **SENSOR**
    * *Beschreibung:* Repräsentiert eine messende Einheit innerhalb eines Geräts.
    * *Verantwortung:*
        * Definition der Messgröße (Metrik, Einheit)
        * Abtastrate
        * Quelle für Messdaten und Alarmüberwachung

* **MEASUREMENT**
    * *Beschreibung:* Repräsentiert einen einzelnen Messwert eines Sensors.
    * *Verantwortung:*
        * Persistenz zeitbasierter Sensordaten
        * Grundlage für Visualisierung und Analyse
    * *Hinweis:* Diese Entität ist für hochfrequente Schreibzugriffe optimiert und wird typischerweise in einer Time-Series-Datenbank gespeichert.

* **ALERT_RULE**
    * *Beschreibung:* Repräsentiert eine Regel zur automatisierten Überwachung von Sensorwerten.
    * *Verantwortung:*
        * Definition von Schwellenwerten und Zeitbedingungen
        * Aktivierung / Deaktivierung von Regeln
        * Auslöselogik für Alarmereignisse

* **ALERT_EVENT**
    * *Beschreibung:* Repräsentiert ein konkretes ausgelöstes Alarmereignis.
    * *Verantwortung:*
        * Protokollierung von Regelverletzungen
        * Statusverfolgung (offen, bestätigt, behoben)

* **SUBSCRIPTION**
    * *Beschreibung:* Repräsentiert ein Abonnement eines Benutzers.
    * *Verantwortung:*
        * Abbildung von Tarifmodellen
        * Steuerung verfügbarer Systemfunktionen

* **SUBSCRIPTION_ADDON**
    * *Beschreibung:* Repräsentiert optionale Erweiterungen eines Abonnements.
    * *Verantwortung:*
        * Erweiterung von Funktionsumfang oder Kapazitäten

#### Beziehungen und fachliche Abhängigkeiten

**Benutzer und Haushalte**
* USER und HOUSEHOLD stehen in einer N:M-Beziehung über HOUSEHOLD_MEMBER.
* Rollen steuern Berechtigungen innerhalb eines Haushalts.

**Gerätehierarchie**
* Ein HOUSEHOLD besitzt mehrere DEVICE.
* Ein DEVICE enthält mehrere SENSOR.
* Diese Hierarchie bildet die physische Struktur des Smart-Home-Systems ab.

**Messdatenfluss**
* Ein SENSOR erzeugt kontinuierlich MEASUREMENT-Datensätze.
* Messdaten werden als Zeitreihen gespeichert und für Analyse und Visualisierung verwendet.

**Alarmverarbeitung**
* Ein SENSOR wird durch eine oder mehrere ALERT_RULE überwacht.
* Wird eine Regel verletzt, wird ein ALERT_EVENT erzeugt.
* Alarmereignisse referenzieren sowohl Regel als auch Sensor.

**Abonnementmodell**
* Ein USER kann mehrere SUBSCRIPTION besitzen.
* Eine SUBSCRIPTION kann mehrere SUBSCRIPTION_ADDON enthalten.
* Abonnements steuern verfügbare Funktionen und Limits.

#### Persistenz- und Speicherstrategie

Das Datenmodell unterstützt eine hybride Speicherstrategie:

**Relationale Datenbank (RDBMS)**

Speichert:
* USER
* HOUSEHOLD
* HOUSEHOLD_MEMBER
* DEVICE
* SENSOR
* ALERT_RULE
* ALERT_EVENT
* SUBSCRIPTION
* SUBSCRIPTION_ADDON

**Begründung (RDBMS):**
* Starke Konsistenz
* Referentielle Integrität
* Komplexe Abfragen und Beziehungen

**Time-Series-Datenbank (TSDB)**

Speichert:
* MEASUREMENT

**Begründung (TSDB):**
* Hohe Schreibfrequenz
* Zeitbasierte Aggregationen
* Effiziente Kompression und Retention

#### Qualitätsmerkmale und Architekturtreiber

| Aspekt | Beitrag des Datenmodells |
| : | : |
| **Skalierbarkeit** | Trennung von Messdaten und Stammdaten |
| **Erweiterbarkeit** | Entkoppelte Entitäten (Sensor, Rule, Subscription) |
| **Wartbarkeit** | Klare Domänengrenzen |
| **Performance** | Optimierte Speicherung von Zeitreihen |
| **Sicherheit** | Rollenbasierte Zugriffskontrolle |
| **Datenkonsistenz** | Relationale Integrität |
| **Nachvollziehbarkeit** | Historisierung von Events und Messdaten |


#### Abgrenzung und Annahmen

* **Authentifizierung und Autorisierung** sind nicht Teil des ER-Modells, sondern werden auf Applikationsebene umgesetzt.
* **Physische Gerätekommunikation** ist nicht Bestandteil des Datenmodells.
* **Historische Archivierung und Datenlöschung** erfolgen über separate Retention-Policies.


#### 1.2.3 Beschreibung der transienten Daten und Schnittstellen

<img width="700" alt="Ingestion" src="https://github.com/user-attachments/assets/e04e1480-c4ff-44d5-b80a-d355efd2e6e7" />

Bild 4: Messdatenübertragung (Ingestion)

**Beschreibung des Sequenzdiagramm:**
* **Schritt 1:** IoT-Gerät erfasst einen Messwert über das lokale Netzwerk (WiFi mit Matter/Zigbee).
* **Schritt 2:** Der Gateway / Umsetzer empfängt die Rohdaten vom Gerät.
* **Schritt 3:** Der Gateway normalisiert die Daten und mappt sie auf ein einheitliches JSON-Format.
* **Schritt 4:** Der Gateway sendet die Messdaten per HTTP POST an den Backend-API-Endpunkt `/ingest`.
* **Schritt 5:** Das Backend bestätigt den Empfang mit `202 Accepted` und stellt die Daten zur weiteren Verarbeitung in eine Warteschlange.

<img width="700" alt="Validierung" src="https://github.com/user-attachments/assets/53f0f600-3517-4c0a-b858-299b55374d73" />

Bild 5: Validierung eingehender Messdaten

**Beschreibung des Sequenzdiagramm:**
* **Schritt 1:** Der Gateway sendet einen JSON-Payload an den Backend-API-Endpunkt `/ingest`.
* **Schritt 2:** Die Backend API leitet den Payload zur Validierung an die Service Layer weiter.
* **Schritt 3:** Die Service Layer prüft:
    * Vollständigkeit der Pflichtfelder
    * Datentypen und Format
    * Plausibilität der Werte
* **Schritt 4:**
    * Bei gültigem Payload wird die Verarbeitung bestätigt und die API antwortet mit `202 Accepted`.
    * Bei ungültigem Payload wird die Validierung abgelehnt und die API antwortet mit `400 Bad Request` inklusive Fehlerdetails.

<img width="700" height="500" alt="Speicherung" src="https://github.com/user-attachments/assets/30f78dd9-dbd7-47f7-a4c7-2959320b6d4e" />

Bild 6: Speicherung validierter Messdaten

**Beschreibung des Sequenzdiagramm:**
* **Schritt 1:** Die Backend API übergibt einen validierten Payload an die Service Layer zur Verarbeitung.
* **Schritt 2:** Optional erfolgt ein Lookup der Geräte- und Sensor-Stammdaten in der relationalen Datenbank.
* **Schritt 3:** Die Service Layer schreibt den Messwert mit Zeitstempel und Einheit in die Time-Series-Datenbank.
* **Schritt 4:** Die Datenbank bestätigt den erfolgreichen Schreibvorgang.
* **Schritt 5:** Die Backend API bestätigt die erfolgreiche Verarbeitung und kann optional Metriken oder Logs erfassen.

<img width="700" alt="Screenshot 2026-01-22 140611 (1)" src="https://github.com/user-attachments/assets/2ac6828a-9d34-4f20-a900-33f0ef6adb63" />

Bild 7: Auffälligen Verbrauch erkennen und Alert auslösen

**Beschreibung des Sequenzdiagramm:**
* **Schritt 1:** Die Service Layer übergibt neue Messwerte an das Alerting-Modul zur Auswertung.
* **Schritt 2:** Das Alerting lädt die aktiven Alarmregeln für den betroffenen Sensor und Haushalt aus der Datenbank.
* **Schritt 3:** Die Regeln werden gegen den aktuellen Messwert und die definierte Zeitbedingung geprüft.
* **Schritt 4:**
    * Bei Regelverletzung wird ein Alarmereignis in der Datenbank gespeichert.
    * Bei keiner Regelverletzung wird kein Ereignis erzeugt.
* **Schritt 5:** Das Dashboard ruft regelmäßig die aktuellen Alarmereignisse über die Backend API ab und visualisiert sie.
  
### 1.3 Mockups
#### 1.3.1 Anmeldemaske

<p align="center">
  <img src="https://github.com/user-attachments/assets/8a9decdf-706e-4810-8f02-5cb1ebc08b82" width="200" alt="App Anmeldung Screen">
  <br>
  <em>Bild 8: Anmeldemaske
</p>

#### 1.3.2 Hauptseite nach der Anmeldung

<p align="center">
  <img src="https://github.com/user-attachments/assets/8c49478e-2bb2-4475-b096-2f74f54de1b7" width="200" alt="App Anmeldung Screen">
  <br>
  <em>Bild 9: Anmeldung - Energie Monitor App</em>
</p>

Bild 9: Anmeldung

#### 1.3.3 Location anzeigen
<p align="center">
  <img src="https://github.com/user-attachments/assets/3cc1e0ca-7c27-44d1-95f0-a184b8ca7bd7" width="200" alt="Mockup_Location">
  <br>
  <em>Bild 10: Location </em>
</p>

#### 1.3.4 Smart Devices aufgelistet anzeigen
<p align="center">
  <img src="https://github.com/user-attachments/assets/a5f5db7e-e721-4bf0-980c-c05be8b9cd19" width="200" alt="Mockup_SmartDevices">
  <br>
  <em>Bild 11: SmartDevices</em>
</p>

#### 1.3.5 Verbrauchsübersicht
<p align="center">
  <img src="https://github.com/user-attachments/assets/d066d2f7-6fc1-4d3e-bb26-2103fa211f7b" width="200" alt="Mockup_SmartDevices">
  <br>
  <em>Bild 12: Verbrauchsübersicht</em>
</p>

#### 1.3.6 Analyse
<p align="center">
  <img src="https://github.com/user-attachments/assets/51e4d582-61aa-40b6-8eb5-bd6dc95d2bb3" width="200" alt="Mockup_Analyse">
  <br>
  <em>Bild 13: Analyse</em>
</p>

### 1.4 Lastenheft

#### 1.4.1 Nicht-Funktionale Anforderungen

* **NFA1: Benutzerfreundlichkeit (Usability)**
    * Das System soll intuitiv bedienbar sein.
    * Die Benutzeroberfläche soll übersichtlich gestaltet sein.
    * Die Darstellung der Daten soll leicht verständlich sein.

* **NFA2: Performance**
    * Das System soll stabil und fehlerfrei laufen.
    * Diagramme sollen auch bei großen Datenmengen flüssig dargestellt werden.
    * Berechnungen sollen zeitnah erfolgen.

* **NFA3: Zuverlässigkeit**
    * Das System soll Daten ohne spürbare Verzögerung anzeigen.
    * Messdaten dürfen nicht verloren gehen.
    * Temporäre Verbindungsprobleme sollen abgefangen werden.

* **NFA4: Verfügbarkeit**
    * Das System soll eine hohe Verfügbarkeit aufweisen.
    * Wartungszeiten sollen minimal gehalten werden.

* **NFA 5: Sicherheit**
    * Daten sollen vor unbefugtem Zugriff geschützt sein.
    * Die Kommunikation zwischen Komponenten soll abgesichert erfolgen.

* **NFA 6: Datenschutz**
    * Personenbezogene Daten sollen gemäß geltender Datenschutzvorgaben verarbeitet werden.
    * Es sollen nur notwendige Daten gespeichert werden.

* **NFA 7: Skalierbarkeit**
    * Das System soll mit einer steigenden Anzahl von Geräten und Datenmengen umgehen können.
    * Eine spätere Erweiterung soll möglich sein.

* **NFA 8: Wartbarkeit**
    * Das System soll modular aufgebaut sein.
    * Änderungen und Erweiterungen sollen mit vertretbarem Aufwand möglich sein.

#### 1.4.2 Funktionale Anforderungen

<img width="700" alt="MindmapFAV3" src="https://github.com/user-attachments/assets/e049f4ac-4149-4d11-ab43-b02ad993affc" />
Bild: funktionale Anforderungen

##### 1.4.2.1 Funktionale Anforderung 1: Erfassung & Übertragung von Messdaten
 
**Allgemeine Informationen**

| Feld | Inhalt |
| :--- | :--- |
| **Name des Testfalls** | Validierung von Messdaten |
| **Version** | 1.0 |
| **Datum** | *(leer)* |
| **Autor** | Energie Monitor Team |
| **Kurzbeschreibung** | Sensoren erfassen Messdaten und übertragen sie an das Backend-System. |
| **Primärer Akteur** | Sensor / Mikrocontroller |
| **Sekundärer Akteur** | Backend-Server |
| **Auslöser / Vorbedingung** | Sensor ist aktiv, Netzwerkverbindung besteht |
| **Ergebnisse / Nachbedingungen** | Messdaten sind erfolgreich im Backend angekommen|
| **Technische Randbedingungen** |1.CoAP <br> 2.WLAN <br> 3.Embedded OS|
| **Allgemeine Bemerkungen** | Übertragung erfolgt automatisch |
| **Offene Punkte** | Pufferung bei Netzwerkausfall |

**Ablauf und Szenarien**

| Schritt | Akteur | Ablauf | Verzweigung |
| :--- | :--- | :--- | :--- |
| **1** | **Backend** | **Empfängt Messdaten** | |
| 1.1 | Sensor | Erfasst Messwert |  |
| 1.2 | Sensor | Bereitet Daten auf |  |
| 1.3 | Sensor | Sendet Daten an Backend |A1|
| 1.4 | Backend | Empfängt Daten | |
| 1.5 | Backend | Bestätigt Empfang | |
| | | | |
| **Ausnahmen** | | | |
| A1 | System | Ungültiges Datenformat | |
| | | | |
| **Erweiterungen** | | | |
| E1 | System | Automatische Korrektur einfacher Fehler | |

* **Beschreibung:** Das System muss Messdaten von Sensoren erfassen, die auf Mikrocontrollern mit Embedded-Betriebssystem betrieben werden, und diese über das Heimnetzwerk an ein zentrales Backend übertragen.
* **Wechselwirkungen:** Diese Anforderung steht in Wechselwirkung mit der Netzwerkstabilität sowie mit der Backend-Verarbeitung und Visualisierung.
* **Risiken:** Instabile Netzwerke oder begrenzte Hardware-Ressourcen können die Datenübertragung beeinträchtigen.
* **Vergleich mit bestehenden Lösungen:** Im Gegensatz zu vielen bestehenden Lösungen setzt das Projekt auf CoAP und eine bewusst schlanke Architektur.
* **Grobschätzung des Aufwands:** Der Aufwand wird als mittel eingeschätzt.

##### 1.4.2.2 Funktionale Anforderung 2: Validierung von Messdaten

**Allgemeine Informationen**

| Feld | Inhalt |
| :--- | :--- |
| **Name des Testfalls** | Validierung von Messdaten |
| **Version** | 2.0 |
| **Datum** | *(leer)* |
| **Autor** | Energie Monitor Team |
| **Kurzbeschreibung** | Eingehende Messdaten werden auf Plausibilität geprüft. |
| **Primärer Akteur** | Backend-System |
| **Sekundärer Akteur** | Datenbank |
| **Auslöser / Vorbedingung** | Neue Messdaten liegen vor. |
| **Ergebnisse / Nachbedingungen** | Daten sind validiert oder verworfen. |
| **Technische Randbedingungen** | 1. Validierungsregeln<br>2. Serverlogik |
| **Allgemeine Bemerkungen** | Validierung erfolgt automatisch. |
| **Offene Punkte** | Erweiterbare Prüfregeln |

**Ablauf und Szenarien**

| Schritt | Akteur | Ablauf | Verzweigung |
| :--- | :--- | :--- | :--- |
| **1** | **Backend** | **Empfängt Messdaten** | |
| 1.1 | Backend | Prüft Format | A1 |
| 1.2 | Backend | Prüft Wertebereich | A2 |
| 1.3 | Backend | Markiert Daten als gültig | |
| 1.4 | Backend | Übergibt an Speicherung | |
| | | | |
| **Ausnahmen** | **(System)** | | |
| A1 | System | Ungültiges Datenformat | |
| A2 | System | Unplausible Messwerte | |
| | | | |
| **Erweiterungen** | **(System)** | | |
| E1 | System | Automatische Korrektur einfacher Fehler | |

* **Beschreibung:** Eingehende Messdaten müssen auf Korrektheit, Vollständigkeit und Plausibilität geprüft werden.
* **Wechselwirkungen:** Die Validierung beeinflusst die Datenqualität und die Zuverlässigkeit der Visualisierung.
* **Risiken:** Fehlerhafte Validierung kann zu falschen Darstellungen führen.
* **Vergleich mit bestehenden Lösungen:** Bestehende Plattformen bieten oft generische Validierungsmechanismen, während hier ein projektspezifischer Ansatz verfolgt wird.
* **Grobschätzung des Aufwands:** Der Aufwand wird als gering bis mittel eingeschätzt.

##### 1.4.2.3 Funktionale Anforderung 3: Speicherung von Messdaten

**Allgemeine Informationen**

| Feld | Inhalt |
| :--- | :--- |
| **Name des Testfalls** | Speicherung von Messdaten |
| **Version** | 3.0 |
| **Datum** | *(leer)* |
| **Autor** | Energie Monitor Team |
| **Kurzbeschreibung** | Validierte Messdaten werden dauerhaft gespeichert. |
| **Primärer Akteur** | Backend-System |
| **Sekundärer Akteur** | Datenbank |
| **Auslöser / Vorbedingung** | Daten wurden validiert. |
| **Ergebnisse / Nachbedingungen** | Daten sind persistent gespeichert. |
| **Technische Randbedingungen** | 1. Dateisystem<br>2. Datenbank |
| **Allgemeine Bemerkungen** | Speicherung erfolgt chronologisch. |
| **Offene Punkte** | Backup-Strategie |

**Ablauf und Szenarien**

| Schritt | Akteur | Ablauf | Verzweigung |
| :--- | :--- | :--- | :--- |
| **1** | **Backend** | **Empfängt validierte Daten** | |
| 1.1 | Backend | Bereitet Speicherung vor | |
| 1.2 | Backend | Schreibt Daten in DB | A1 |
| 1.3 | Backend | Bestätigt Speicherung | |
| 1.4 | Datenbank | Aktualisiert Index | |
| | | | |
| **Ausnahmen** | **(System)** | | |
| A1 | System | Speicherfehler | |
| | | | |
| **Erweiterungen** | **(System)** | | |
| E1 | System | Automatisches Backup | |

* **Beschreibung:** Validierte Messdaten müssen persistent gespeichert werden, um historische Auswertungen zu ermöglichen.
* **Wechselwirkungen:** Die Speicherung bildet die Grundlage für Verlaufsdarstellungen und Analysen.
* **Risiken:** Wachsende Datenmengen können die Performance beeinträchtigen.
* **Vergleich mit bestehenden Lösungen:** Das Projekt verzichtet bewusst auf komplexe Datenbanksysteme zugunsten einer verständlichen Lösung.
* **Grobschätzung des Aufwands:** Der Aufwand wird als mittel eingeschätzt.

##### 1.4.2.4 Funktionale Anforderung 4: Echtzeitvisualisierung

**Allgemeine Informationen**

| Feld | Inhalt |
| :--- | :--- |
| **Name des Testfalls** | Speicherung von Messdaten |
| **Version** | 4.0 |
| **Datum** | *(leer)* |
| **Autor** | Energie Monitor Team |
| **Kurzbeschreibung** | Aktuelle Messdaten werden im Dashboard dargestellt. |
| **Primärer Akteur** | Nutzer:in|
| **Sekundärer Akteur** | Web-Frontend, Backend |
| **Auslöser / Vorbedingung** | Daten wurden angemeldet. |
| **Ergebnisse / Nachbedingungen** | Aktuelle Daten sind sichtbar |
| **Technische Randbedingungen** | 1. WebSocket / HTTP <br>2. Browser |
| **Allgemeine Bemerkungen** | Darstellung nahezu verzögerungsfrei |
| **Offene Punkte** | Anpassung der Aktualisierungsrate |

**Ablauf und Szenarien**

| Schritt | Akteur | Ablauf | Verzweigung |
| :--- | :--- | :--- | :--- |
| **1** | **Backend** | **Empfängt validierte Daten** | |
| 1.1 | Nutzer:in | Öffnet Dashboard | |
| 1.2 | Frontend | Fordert Daten an |  |
| 1.3 | Backend | Liefert aktuelle Daten| A1|
| 1.4 | Frontend | Aktualisiert Anzeige | |
| 1.5 | Nutzer:in | Betrachtet Werte | |
| | | | |
| **Ausnahmen** |  | | |
| A1 | System | Verzögerte Datenübertragung | |
| | | | |
| **Erweiterungen** | | | |
| E1 | System | Automatisches Refresh| |

* **Beschreibung:** Aktuelle Messdaten sollen nahezu in Echtzeit im Web-Dashboard dargestellt werden.
* **Wechselwirkungen:** Diese Anforderung hängt stark von der Übertragungs- und Backend-Performance ab.
* **Risiken:** Hohe Aktualisierungsraten können die Systemlast erhöhen.
* **Vergleich mit bestehenden Lösungen:** Im Vergleich zu umfangreichen Dashboards wird ein reduzierter Ansatz verfolgt.
* **Grobschätzung des Aufwands:** Der Aufwand wird als mittel eingeschätzt.

## 2. Qualitätssicherung

### 2.1 Beschreibung der Umgebungen und CI-Pipeline

Die Qualitätssicherung des Smart Home Dashboards erfolgt in verschiedenen Umgebungen, um sicherzustellen, dass die Verarbeitung der Energiedaten und die Visualisierung in jeder Phase stabil funktionieren.

* **Entwicklungsumgebung:**
    * Lokale Entwicklungsumgebung auf den Rechnern der Entwickler.
    * Verwendung von **Docker-Containern**, um die Backend-Dienste (Node.js/Python) und Datenbanken konsistent zu halten.
    * **Tools:** Visual Studio Code, Git für Versionskontrolle.
    * **Simulation:** Einsatz von Skripten zur Simulation von MQTT-Datenströmen (simulierte smarte Steckdosen).

* **Testumgebung (Staging):**
    * Cloud-basierte Staging-Umgebung (z. B. auf Render, AWS oder Azure).
    * **Datenbank:** InfluxDB oder PostgreSQL zur Speicherung von Zeitreihendaten.
    * **Lasttests:** Prüfung der Verarbeitungskapazität bei hoher Frequenz von eingehenden JSON-Messwerten.

* **Produktionsumgebung:**
    * Hochverfügbare Infrastruktur mit SSL-Verschlüsselung.
    * Überwachung der Schnittstellen (Endpunkte für HTTP/MQTT).
    * **Sicherheit:** Schutz der privaten Verbrauchsdaten gemäß DSGVO.

#### 2.1.1 CI-Pipeline (Continuous Integration)

Die CI-Pipeline automatisiert den Build- und Testprozess bei jeder Code-Änderung:

* **Code-Integration:** Automatischer Trigger bei Push in das GitHub/GitLab Repository.
* **Automatisierte Tests:**
    * **Unit Tests:** Validierung der Berechnungslogik für Durchschnittswerte und Einsparpotenziale.
    * **Integrationstests:** Überprüfung der Kommunikation zwischen dem MQTT-Broker und dem Backend.
* **Code-Qualitätsprüfung:** Statische Analyse (z. B. ESLint) zur Einhaltung von Coding-Standards.
* **Deployment:** Automatisches Deployment auf den Staging-Server nach erfolgreichen Tests.

### 2.2 Testverfahren

Es werden sowohl **Blackbox-Tests** als auch **Whitebox-Tests** eingesetzt, um die Zuverlässigkeit der Datenverarbeitung zu garantieren.

* **Funktionale Tests:**
    * Überprüfung der Kernfunktionen: Datenempfang, Speicherung und Anzeige im Dashboard.

* **Blackbox-Tests (Perspektive des Endnutzers):**
    * **Anwendungsfall Datenvisualisierung:** Erscheinen die Verbräuche der smarten Steckdosen korrekt in den Diagrammen?
    * **Anwendungsfall Historischer Vergleich:** Funktioniert die Umschaltung zwischen Tages-, Wochen- und Monatsansicht fehlerfrei?
    * **Tools:** Selenium oder Cypress für automatisierte Oberflächentests.

* **Whitebox-Tests (Interne Logik):**
    * **JSON-Validierung:** Testen der Backend-Logik auf den Umgang mit fehlerhaften oder unvollständigen JSON-Paketen.
    * **API-Tests:** Überprüfung der HTTP-Schnittstellen für den Datenaustausch.
    * **Code Coverage:** Ziel ist eine Abdeckung von mindestens 80% der geschäftskritischen Logik.

**Akzeptanzkriterien für User Acceptance Tests (UATs):**

* **Datenaktualität:** Messwerte müssen innerhalb von maximal 2 Sekunden nach dem Empfang im Dashboard sichtbar sein.
* **Visualisierung:** Diagramme müssen bei Zeitbereichswechseln (Tag/Monat) ohne Darstellungsfehler laden.
* **Genauigkeit:** Die berechneten Durchschnittswerte müssen mit den Rohdaten in der Datenbank übereinstimmen.
* **Sicherheit:** Zugriff auf das Dashboard ist nur für autorisierte Nutzer möglich; Datenübertragung erfolgt verschlüsselt.

### 2.3 Testvorlagen

Zur Dokumentation werden standardisierte Vorlagen verwendet.

#### 2.3.1 Test Case: Erfassung und Übertragung von Messdaten
**Allgemeine Informationen**

| Feld | Inhalt |
| :--- | :--- |
| **Name des Testfalls** | Speicherung von Messdaten |
| **Version** | 1.0 |
| **Datum** | *(leer)* |
| **TestID** | TC-1.001 |
| **Test Suite(s)** | Datenerfassung und Netzwerk|
| **Priorität** | Hoch|
| **Erforderliche Hardware** | Sensor, Mikrocontroller, Testserver |
| **Erforderliche Software** | Embedded OS, CoAP-Client, Backend |
| **Testdauer** | 2 Stunden |
| **Aufwand** | 1 Stunde |
| **Setup** | Sensor einrichten, Netzwerk konfigurieren |
| **Rückbau** | Sensor zurücksetzen, Logs sichern |

**Ablauf und Szenarien**
| ID | Testschritt | Resultat | Defekt-Nr | Impact |
| :--- | :--- | :--- | :--- |:---|
| 1.000| Sensor starten| OK | |
| 1.001 | Messwert erfassen | OK |  |
| 1.002 | Daten an Backend senden | Fail| BUG-010| Critical|
| 1.003 | Empfang im Backend prüfen | OK | |
| 1.004 | Übertragungslog auswerten | OK | |
| | | | |

|**Zusammenfassung**| |
|:---|:---|
| Feld | Wert |
| Status | Pass |
| Tester | Projektteam |
| Test beendet am|- |
| Aufwand | 2 Stunden |
| Durchführungszeit | 2 Stunden |

#### 2.3.2 Test Case: Validierung von Messdaten

**Allgemeine Informationen**

| Feld | Inhalt |
| :--- | :--- |
| **Name des Testfalls** | Validierung von Messdaten |
| **Version** | 4.0 |
| **Datum** | *(leer)* |
| **TestID** | TC-2.001 |
| **Test Suite(s)** | Datenvalidierung |
| **Priorität** | Hoch|
| **Erforderliche Hardware** | Backend-Server |
| **Erforderliche Software** | Datenbank, Validierungsmodul |
| **Testdauer** | 1,5 Stunden |
| **Aufwand** | 1 Stunde |
| **Setup** | Validierungsregeln konfigurieren |
| **Rückbau** | Testdaten löschen |

**Ablauf und Szenarien**
| ID | Testschritt | Resultat | Defekt-Nr | Impact |
| :--- | :--- | :--- | :--- |:---|
| 2.000| Testdaten erzeugen| OK | |
| 2.001 | Korrekte Daten einspeisen | OK |  |
| 2.002 | Fehlerhafte Daten einspeisen | Warning| BUG-011/BUG-012| Major|
| 2.003 | Validierungsergebnis prüfen | OK | |
| 2.004 | Log-Dateien analysieren | OK | |
| | | | |

|**Zusammenfassung**| |
|:---|:---|
| Feld | Wert |
| Status | Pass |
| Tester | Projektteam |
| Test beendet am|- |
| Aufwand | 1,5 Stunden |
| Durchführungszeit | 1,5 Stunden |


#### 2.3.3 Test Case: Speicherung von Messdaten

**Allgemeine Informationen**

| Feld | Inhalt |
| :--- | :--- |
| **Name des Testfalls** | Speicherung von Messdaten |
| **Version** | 4.0 |
| **Datum** | *(leer)* |
| **TestID** | TC-3.001 |
| **Test Suite(s)** | Datenspeicherung |
| **Priorität** | Hoch|
| **Erforderliche Hardware** | Server mit Speicher |
| **Erforderliche Software** | Datenbank, Backend |
| **Testdauer** | 2 Stunden |
| **Aufwand** | 1 Stunde |
| **Setup** | Datenbank intialisieren |
| **Rückbau** | Testdaten löschen |

**Ablauf und Szenarien**
| ID | Testschritt | Resultat | Defekt-Nr | Impact |
| :--- | :--- | :--- | :--- |:---|
| 3.000| Nutzer:in | OK | |
| 3.001 | Frontend | OK |  |
| 3.002 | Backend | Warning| BUG-013| Medium|
| 3.003 | Frontend | OK | |
| 3.004 | Nutzer:in | OK | |
| | | | |

|**Zusammenfassung**| |
|:---|:---|
| Feld | Wert |
| Status | Pass |
| Tester | Projektteam |
| Test beendet am|- |
| Aufwand | 2 Stunden |
| Durchführungszeit | 2 Stunden |

#### 2.3.4 Test Case: Echtzeitvisuallisierung von Messdaten

**Allgemeine Informationen**

| Feld | Inhalt |
| :--- | :--- |
| **Name des Testfalls** | Echtzeitvisualisierung von Messdaten|
| **Version** | 4.0 |
| **Datum** | *(leer)* |
| **TestID** | TC-4.001 |
| **Test Suite(s)** | Frontend und Dashboard|
| **Priorität** | Hoch|
| **Erforderliche Hardware** | PC / Tablet |
| **Erforderliche Software** | Datenbank, Backend, Webbrowser |
| **Testdauer** | 2 Stunden |
| **Aufwand** | 1 Stunde |
| **Setup** | Nutzer anmelden, Dashboard öffnen |
| **Rückbau** | Session beenden |

**Ablauf und Szenarien**
| ID | Testschritt | Resultat | Defekt-Nr | Impact |
| :--- | :--- | :--- | :--- |:---|
| 4.000| Nutzer anmelden | OK | |
| 4.001 | Dashboard öffnen| OK |  |
| 4.002 | Live-Daten abrufen| OK |  |
| 4.003 | Backend | Warning| BUG-021/BUG-002| High|
| 4.004 | Reaktionszeit messen | OK | |
| | | | |

|**Zusammenfassung**| |
|:---|:---|
| Feld | Wert |
| Status | Pass |
| Tester | Projektteam |
| Test beendet am|- |
| Aufwand | 2 Stunden |
| Durchführungszeit | 2 Stunden |

#### 2.3.5 Mögliche Defekte

| Use Case | ID | Szenario / Schritt | Möglicher Fehler | Impakt |
| :--- | :--- | :--- | :--- | :--- |
| **1. Login & App Start** | **BUG-001** | App Starten (1.0) | **Crash on Startup:** App stürzt sofort nach dem Tippen auf das Icon ab. | **Blocker** |
| | **BUG-002** | Verbindung (1.2) | **Infinite Loading:** Ladebalken läuft endlos, kein Timeout, keine Fehlermeldung bei fehlendem Internet. | **Major** |
| | **BUG-003** | Authentifizierung (1.3) | **False Positive:** Nutzer kann sich mit falschem Passwort einloggen. | **Critical** |
| | **BUG-004** | Profil laden (1.4) | **Data Mismatch:** Nach Login wird das Profil eines anderen Nutzers (oder Testdaten) angezeigt. | **Critical** |
| | **BUG-005** | UI Initialisierung (1.1) | **Layout Broken:** Login-Button ist auf kleinen Bildschirmen abgeschnitten oder nicht klickbar. | **Major** |
| | | | | |
| **2. Erfassen & Speichern** | **BUG-010** | Daten senden (1.1) | **Data Loss:** Smart Device sendet Daten, aber Backend empfängt nichts (Paketverlust ohne Retry). | **Critical** |
| | **BUG-011** | Validierung (1.3) | **Validation Error:** Backend akzeptiert ungültiges JSON-Format und speichert korrupte Daten („Garbage Data“). | **Major** |
| | **BUG-012** | Validierung (1.3) | **Server Error:** Senden von Buchstaben im Zahlenfeld (z.B. „20kWh“ statt „20“) verursacht HTTP 500 Fehler statt 400 Bad Request. | **Major** |
| | **BUG-013** | Speichern (1.4) | **Duplicate Entries:** Ein Messwert wird doppelt in die Datenbank geschrieben. | **Medium** |
| | **BUG-014** | Protokollierung (1.5) | **Missing Logs:** Erfolgreiche Übertragung taucht nicht in den System-Logs auf. | **Low** |
| | | | | |
| **3. Anzeigen & Analysieren** | **BUG-020** | Daten anfordern (1.1) | **Empty State:** Bei leerer Datenbank stürzt die App ab, statt „Keine Daten verfügbar“ anzuzeigen. | **Major** |
| | **BUG-021** | Diagramme erstellen (1.3) | **Rendering Fail:** Diagramm wird nicht gezeichnet (weiße Fläche), obwohl Daten vom Backend geliefert wurden. | **High** |
| | **BUG-022** | Trends anzeigen (1.5) | **Wrong Calculation:** Der angezeigte Durchschnittswert stimmt nicht mit den Rohdaten überein (Rechenfehler). | **High** |
| | **BUG-023** | Zeitraum wählen (1.4) | **UI Freeze:** App friert für 5+ Sekunden ein, wenn man von „Woche“ auf „Jahr“ umschaltet (Performance). | **Medium** |
| | **BUG-024** | UI/UX | **Overlap:** Beschriftungen der X-Achse (Datum) überlappen sich und sind unlesbar. | **Low** |

## 3. Projektstruktur und Ressourcenplanung

### 3.1 Vorgehensweise

Für die Entwicklung des Smart Home Dashboards wird die **Scrum-Methode** verwendet. Scrum ist ein agiles Framework, das eine iterative und inkrementelle Vorgehensweise ermöglicht, um flexibel auf Anforderungen und Feedback reagieren zu können. Das Projekt ist in **Sprints** unterteilt, die jeweils zwei Wochen dauern. Jeder Sprint hat ein klar definiertes Ziel, und am Ende jedes Sprints wird ein funktionsfähiges Inkrement der Software geliefert.

| Sprint | Zeitraum | Fokus / Ziel | Wichtige Lieferergebnisse (Deliverables) |
| :--- | :--- | :--- | :--- |
|**Sprint 1**| Woche 1-2 | **Konzept & Analyse** | Requirements (Lastenheft), Systemarchitektur-Diagramm, JSON-Definition. |
|**Sprint 2**| Woche 3-4 | **Design & Prototyping** | UI-Mockups (Figma), Datenbank-Schema, Aufsetzen der Entwicklungsumgebung. |
|**Sprint 3**| Woche 5-6 | **Backend Core** | MQTT-Broker Setup, Python-Skript zum Empfang der Sensordaten, Speicherung in DB. |
|**Sprint 4**| Woche 7-8 | **Frontend Development** | Entwicklung des Dashboards (React), Visualisierung der Echtzeit-Werte (Tacho). |
|**Sprint 5**| Woche 9-10 | **Integration & Testing** | Verknüpfung von Sensor und Frontend, Test der Warnmeldungen (Alerts), Bugfixing. |
|**Sprint 6**| Woche 11-12 | **Finalisierung** | Dokumentation, User Guide, Abschlusspräsentation, Deployment. |


### 3.2 Work Breakdown

#### 3.2.1 Stakeholder und Verantwortlichkeiten

Die folgenden Stakeholder sind am Projekt SmartHome Dashboard beteiligt, und ihre Verantwortlichkeiten sind wie folgt definiert:

* **Projektleitung & Organisation (PM):**
    * Verantwortlich für die Gesamtkoordination des Projekts und die Kommunikation als Gruppensprecherin.
    * Überwachung des Projektfortschritts mittels Gantt-Diagramm und Einhaltung der Meilensteine.
    * Verantwortlich für die organisatorische Struktur sowie die fristgerechten Abgaben.

* **Systemarchitektur, Entwicklung & Prozessmanagement (Tech):**
    * **Technische Konzeption:** Design der gesamten Systemarchitektur, einschließlich der Integration von Frontend, Backend und den Datenbank-Strukturen (InfluxDB/PostgreSQL).
    * **Implementierung:** Verantwortlich für die operative Softwareentwicklung und die Definition der JSON-Schnittstellen zur Verarbeitung der Sensordaten (MQTT/HTTP).
    * **Agiles Projektmanagement:** Festlegung und Überwachung des Projektablaufs nach dem SCRUM-Modell, inklusive der Leitung von Sprint-Planungen und Retrospektiven.
    * **Anforderungsanalyse:** Mitwirkung bei der Erfassung funktionaler Anforderungen, um diese in technische User Stories zu übersetzen.

* **Technische Dokumentation, Qualitätssicherung & QA:**
    * **Qualitätsmanagement:** Sicherstellung der Code-Qualität durch regelmäßige Code-Reviews sowie die Durchführung umfassender Testreihen (Unit Tests, Integrationstests und Systemtests).
    * **Dokumentation:** Verantwortlich für die Ausarbeitung der technischen Projektstruktur sowie die Erstellung und Pflege der gesamten Projektdokumentation (z. B. Lastenheft, Systemhandbuch).
    * **Validierung & Review:** Unterstützung bei der Modellierung der Datenflüsse und Durchführung von Revisionen, um sicherzustellen, dass alle Lieferergebnisse die definierten Akzeptanzkriterien erfüllen.
    * **Fehlermanagement:** Systematische Dokumentation von Softwarefehlern (Bugs) und proaktive Überwachung deren Behebung zur Gewährleistung einer stabilen Release-Version.

* **Private Haushalte (Kunden / Endnutzer):**
    * Bereitstellung von funktionalen Anforderungen zur Energieüberwachung und -einsparung.
    * Feedback zur Benutzerfreundlichkeit des Dashboards und der Visualisierungen.
    * Zielgruppe für die Verwertbarkeitsstudie im Rahmen des Business Model Canvas.

* **Hardware-Partner & Service-Provider:**
    * Bereitstellung der IoT-Infrastruktur (Sensoren, Smart Plugs) zur Datenerfassung.
    * Sicherstellung der Datenübertragung über Protokolle wie MQTT oder HTTP.
    * Bereitstellung externer Datenquellen (z. B. Wetter-API, Strompreise).

#### 3.2.2 RACI-Matrix

| Aktivität / Arbeitspaket | PM | Tech | QA | Private Haushalte | Hardware Partner |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Projektmanagement & Anmeldung** | A/R | C | I | I | I |
| **Anforderungsanalyse (User Stories)** | A | C | R | C/I | I |
| **Home Assistant Architektur-Design** | I | A/R | C | I | C |
| **Hardware-Bereitstellung & Support** | I | C | I | I | A/R |
| **Frontend & Widget Entwicklung** | A | R | C | I | I |
| **Integration (API & Home Assistant)** | I | A/R | C | I | C |
| **Qualitätssicherung & Testing** | A | C | R | C | I |
| **User Acceptance Test (UAT)** | I | I | R | A/R | I |
| **Dokumentation & Finalisierung** | A | C | R | I | I |

**Legende der Rollen:**
* **R (Responsible / Verantwortlich):** Die Person, die letztendlich die Verantwortung für den Erfolg der Aufgabe trägt und die Entscheidungsgewalt hat (im Sinne von "Accountable").
* **A (Accountable / Ausführend):** Die Person, welche die Aufgabe operativ durchführt oder maßgeblich daran arbeitet.
* **C (Consulted / Konsultierend):** Personen, die Fachwissen beisteuern oder deren Meinung vor der Fertigstellung eingeholt wird.
* **I (Informed / Informiert):** Personen, die über den Fortschritt oder das Ergebnis informiert werden.


### Beschreibung der Phasenübergänge (Quality Gates)

#### Übergang 1: Von der Initialisierung zur offiziellen Anmeldung (Quality Gate 1)
* **Inhalt:** Definition des Projektziels (Fokus auf Home Assistant) und Festlegung des funktionalen Rahmens.
* **Bedingung:** Das Team hat sich auf einen eindeutigen Projekttitel geeinigt. Die Privaten Haushalte (Kunden) wurden konsultiert, um die relevantesten Anwendungsfälle (z. B. Licht, Heizung, Sicherheit) zu priorisieren.
* **Verantwortung:** PM stellt sicher, dass die Anmeldung fristgerecht erfolgt. Tech bestätigt die technische Machbarkeit der Kundenwünsche innerhalb des Home Assistant Ökosystems.

#### Übergang 2: Von der Anmeldung zur detaillierten Planung (Quality Gate 2)
* **Inhalt:** Erstellung der operativen Roadmap inklusive Zeitplan (Gantt-Diagramm) und Ressourcenallokation.
* **Bedingung:** Vorliegen eines Meilensteinplans, der sowohl die Software-Sprints als auch die Bereitstellungszyklen der Hardware-Partner berücksichtigt. Die Arbeitspakete für die Integration der Sensoren sind definiert.
* **Verantwortung:** PM validiert die Zeitplanung; Tech und QA liefern die notwendigen Arbeitspakete zu. Die Hardware-Partner bestätigen die Lieferfähigkeit der benötigten Komponenten (Gateways/Aktoren).

#### Übergang 3: Von der Planung zur technischen Konzeption (Quality Gate 3)
* **Inhalt:** Ausarbeitung der Systemarchitektur und Definition der Schnittstellen (YAML-Konfiguration, WebSocket-API, REST).
* **Bedingung:** Das technische Konzept ist mit den Spezifikationen der Hardware-Partner abgeglichen. Das Geschäftsmodell (BMC) ist konsistent und berücksichtigt die Bedürfnisse der Privaten Haushalte (z. B. Datensicherheit und lokale Speicherung).
* **Verantwortung:** Tech übernimmt die Führung und gibt die Architektur frei. QA prüft die Dokumentation auf Konsistenz und stellt sicher, dass alle technischen Anforderungen für die Home Assistant Instanz dokumentiert sind.

#### Übergang 4: Von der technischen Konzeption zur Finalisierung des Proposals (Quality Gate 4)
* **Inhalt:** Zusammenführung aller Proposal-Bestandteile (WBS, RACI, Lastenheft-Entwurf) und abschließende Qualitätskontrolle.
* **Bedingung:** Durchführung eines finalen Reviews. Ein simulierter User-Acceptance-Test (UAT) oder ein Review durch die Privaten Haushalte bestätigt, dass das Dashboard-Konzept intuitiv und benutzerfreundlich ist. Alle Dokumente sind formal fehlerfrei.
* **Verantwortung:** QA führt die finale Qualitätssicherung durch. Erst nach ihrer Freigabe und der Bestätigung durch die Stakeholder erfolgt die Einreichung des gesamten Proposals durch PM.



### 3.3 Sprint-Planung und agiler Methodeneinsatz

Die Umsetzung erfolgt nach dem Scrum-Framework, um maximale Flexibilität bei der Integration verschiedener SmartHome-Komponenten zu gewährleisten. Zu Beginn jedes Sprints findet ein Sprint Planning Meeting statt, an dem das Kernteam teilnimmt. Die Planung umfasst folgende Kernschritte:

* **Backlog-Auswahl:** Basierend auf der Priorisierung der Privaten Haushalte (Kunden) werden die relevantesten User Stories (z. B. „Steuerung der Philips Hue Leuchten via Dashboard“) aus dem Product Backlog in das Sprint Backlog überführt.
* **Task-Granularisierung:** Die User Stories werden in technische Sub-Tasks unterteilt (z. B. Konfiguration der REST-API, UI-Widget-Design, YAML-Integration).
* **Aufwandsschätzung:** Das Team schätzt die Komplexität der Tasks mittels Story Points. Hierbei werden technische Abhängigkeiten von Hardware-Partnern besonders berücksichtigt.
* **Ressourcenallokation:** Die Tasks werden entsprechend der RACI-Matrix zugewiesen: Linh übernimmt die technische Umsetzung, während QA parallel die Testfälle für das Quality Gate vorbereitet.
* **Definition of Done (DoD):** Ein Task gilt erst als abgeschlossen, wenn er erfolgreich in der Home Assistant Umgebung getestet wurde und die Dokumentation vorliegt.

### 3.4 Entwicklungs- und Testanforderungen

Um die Stabilität und Sicherheit des Dashboards im produktiven Heimbetrieb zu garantieren, werden strikte Qualitätsstandards definiert:

#### Entwicklungsanforderungen

* **Modularität & Skalierbarkeit:** Der Code wird so strukturiert, dass neue Geräteklassen der Hardware-Partner (z. B. Matter-Sensoren) ohne Architekturänderung integriert werden können.
* **Versionskontrolle & Workflow:** Nutzung von Git mit einem definierten Branching-Modell. Code-Reviews erfolgen durch Linh, um die Einhaltung von Clean-Code-Prinzipien sicherzustellen.
* **Home Assistant Compliance:** Verwendung offizieller APIs und Einhaltung der Konfigurationsstandards für YAML- und Lovelace-Komponenten.
* **Dokumentation:** Kontinuierliche Pflege der Systemarchitektur und Schnittstellenbeschreibung durch QA.

#### Testanforderungen

* **Automatisierte Tests:** Durchführung von Unit-Tests für die Logik-Engine sowie Integrationstests zur Validierung der Kommunikation zwischen Dashboard und Home Assistant Core.
* **Hardware-Integrationstests:** Validierung der physischen Schaltvorgänge in Zusammenarbeit mit den Hardware-Spezifikationen der Partner.
* **Nutzerakzeptanztests (UAT):** Regelmäßige Überprüfung der Usability durch Private Haushalte, um sicherzustellen, dass das Interface intuitiv bedienbar ist.
* **Datensicherheit & DSGVO:** Da sensible Verhaltensdaten erfasst werden, erfolgt eine strikte Trennung von lokalem Netzwerk und externem Zugriff. Die Datensouveränität der Haushalte hat oberste Priorität.

### 3.5 Marktabschätzung und Finanzierung

<img width="800" alt="BMC" src="https://github.com/user-attachments/assets/3331a5b4-1723-4377-899d-2f30276fc3af" />

Bild 14: Business Model Canvas

**Beschreibung des Business Model Canvas:**

Das Business Model Canvas des Smart Home Energy Dashboards bietet eine klare Übersicht über die Schlüsselelemente unseres Geschäftsmodells. Es zeigt, wie wir durch technologische Innovation und Benutzerorientierung einen messbaren Mehrwert für Privathaushalte schaffen. Die wichtigsten Aspekte sind:

* **Key Partners:** Wir kooperieren mit Herstellern von Smart-Home-Hardware (z. B. Shelly, Tuya) und Cloud-Providern, um eine stabile Datenerfassung und -speicherung zu gewährleisten. Langfristig streben wir Partnerschaften mit lokalen Energieversorgern an.
* **Value Propositions:** Unsere Plattform bietet Echtzeit-Transparenz über den Stromverbrauch, identifiziert ineffiziente Geräte ("Stromfresser") und ermöglicht den direkten Vergleich mit ähnlichen Haushalten, um Kosten effektiv zu senken.
* **Customer Segments:** Unsere primäre Zielgruppe umfasst kostenbewusste Privathaushalte (Mieter und Eigentümer) sowie technikaffine Smart-Home-Nutzer, die ihren ökologischen Fußabdruck minimieren möchten.
* **Revenue Streams:** Das Geschäftsmodell basiert auf einem Freemium-Ansatz. Grundfunktionen sind kostenlos, während detaillierte historische Analysen und Export-Funktionen über ein Premium-Abonnement angeboten werden.

Das Modell zeigt, dass durch die Kombination aus Kosteneinsparung für den Nutzer und skalierbaren Einnahmequellen ein nachhaltiger wirtschaftlicher Erfolg gesichert wird.




