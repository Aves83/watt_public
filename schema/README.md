Stand: 2026-10-17 (YYYY-MM-DD)

# Schnittstellen von WATT: Datenformat `watt-daten` als JSON und XML

WATT (Workload Analysis & Time-Tracking for Teachers) exportiert alle Daten einer Lehrkraft auf Wunsch als Klartext im offenen Datenformat `watt-daten`, wahlweise als JSON oder als XML (in der App unter „Mehr → Daten, Sicherheit & Export → Export → Klartext“). Beide Dateien haben denselben Inhalt und dieselbe Formatversion. Dieser Ordner enthält die Schemas dazu, damit sich die Dateien ohne WATT prüfen und in Tabellenkalkulationen, Skripten oder anderen Programmen weiterverarbeiten lassen.

| Datei | Inhalt |
| --- | --- |
| `watt-daten-18.schema.json` | JSON Schema (Draft 2020-12) der Formatversion 18 (seit 17.10.2026: Arztbesuch mit Beginn und Ende, Tagesmarkierung „Schulfahrt“) |
| `watt-daten-17.schema.json` | JSON Schema (Draft 2020-12) der Formatversion 17 (12.10.2026 bis 17.10.2026, Fassung „Bergamotte“) |
| `watt-daten-16.schema.json` | JSON Schema (Draft 2020-12) der Formatversion 16 (11.10.2026 bis 12.10.2026, Fassung „Apfel“) |
| `watt-daten-18.xsd` | XML Schema (XSD 1.0) der XML-Darstellung der Formatversion 18 |
| `watt-daten-17.xsd` | XML Schema (XSD 1.0) der XML-Darstellung der Formatversion 17 (seit 16.10.2026 die erste Version mit XML-Export) |

Die vollständige Beschreibung der Felder, der Hash-Ketten und der verschlüsselten Sicherung steht im Repository von WATT in `docs/DATENFORMAT.md`; die öffentliche Kopie dieses Ordners liegt unter https://github.com/Aves83/watt_public/tree/main/schema.

## Was die Dateien enthalten

Eine Datei enthält den gesamten Datenbestand: Profil und Beschäftigungsumfang, Kategorien, Ferien, unterrichtsfreie Tage, Stundenplan, alle erfassten Tätigkeiten, Abwesenheiten, das Änderungsprotokoll der Arbeitszeiten (Arbeitszeitkette), den laufenden Timer, Einstellungen der Auswertung, Funktionen, die Nachweise (Überlastungsanzeigen samt PDF-Dokumenten als Base64, Gesprächsnotizen, Nachweiskette) und seit Version 17 den Kalenderlink mit Zuordnungen und Entscheidungen; seit Version 18 tragen Abwesenheiten der Art `ARZTBESUCH` Beginn und Ende, und unterrichtsfreie Tage kennen die Art `SCHULFAHRT`. Der Klartext-Export ist nicht für Dritte gedacht; WATT weist vor dem Export darauf hin.

## Grundregeln

- Zeitpunkte stehen als ISO 8601 in UTC, z. B. `2026-09-29T08:00:00.123Z`; Kalendertage als `JJJJ-MM-TT` und Uhrzeiten als `HH:MM`, beide in der Zeitzone `Europe/Berlin`.
- Aufzählungen stehen als Name der Konstante, z. B. `KRANKHEIT`.
- Der Export enthält alle Felder, auch leere (`null`); WATT übergeht beim Einlesen unbekannte Felder. Die Schemas beschreiben deshalb genau die Felder, die WATT schreibt (`additionalProperties: false`); Pflichtfelder sind die, ohne die WATT eine Datei ablehnt.
- Die Hash-Ketten lassen sich ohne WATT nachrechnen: Für jedes Glied werden die Felder in der dokumentierten Reihenfolge jeweils als `<Länge in UTF-8-Bytes>:<Wert>` aneinandergehängt; `hash` ist SHA-256 darüber, `prev` der `hash` des Vorgängers (beim ersten Glied `0`).

## XML-Darstellung

Die XML-Datei bildet das JSON eins zu eins ab:

- Das JSON-Objekt der Datei wird zum Wurzelelement `wattDaten`; jedes Feld eines Objekts wird zu einem gleichnamigen Kindelement in der Reihenfolge des JSON.
- Eine Liste wird zu einem Element mit dem Feldnamen, jedes Listenelement zu einem Kindelement mit dem Namen im Singular (z. B. `eintraege` → `eintrag`, `protokoll` → `glied`, `kette` → `glied`, `raster` → `stunde`); eine leere Liste ist ein leeres Element.
- Das Verzeichnis `antworten` mit freien Schlüsseln wird zu einem Element mit einem Kindelement `antwort` je Paar; der Schlüssel steht im Attribut `schluessel`.
- Text steht als Inhalt des Elements, Zahlen und Wahrheitswerte in der Schreibweise des JSON (`22.5`, `true`); ein fehlender Wert (`null`) ist ein leeres Element mit `xsi:nil="true"`.

WATT liest XML nicht ein; Daten kommen nur über die verschlüsselte Sicherung oder den JSON-Klartext zurück.

## Prüfen

JSON Schema, z. B. mit [ajv](https://ajv.js.org/):

```
npx ajv-cli validate -s watt-daten-18.schema.json -d WATT_2026-10-17.json --spec=draft2020
```

XML Schema, z. B. mit xmllint:

```
xmllint --noout --schema watt-daten-18.xsd WATT_2026-10-17.xml
```

## Versionen

Jede Formatversion ab 16 erhält ein eigenes JSON Schema und jede Formatversion ab 17 ein eigenes XML Schema (`watt-daten-<Version>.schema.json`, `watt-daten-<Version>.xsd`); die Schemas älterer Versionen bleiben dauerhaft liegen, eine neue Formatversion bringt neue Dateien dazu. Ältere Dateien bleiben lesbar (siehe `docs/DATENFORMAT.md`, Tabelle der Versionen). Zu jeder an Testende ausgelieferten Fassung von WATT liegt im Repository eine Beispieldatei, die ein Test gegen das JSON Schema und, ab Version 17, gegen das XML Schema ihrer Version prüft.

## Lizenz

Die Dateien in diesem Ordner (Schemas und diese Beschreibung) stehen unter der Lizenz [Creative Commons Namensnennung 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/deed.de). Namensnennung: „Hans-Joachim Steudel, WATT – Workload Analysis & Time-Tracking for Teachers“. WATT selbst ist proprietär; die Nutzungsbedingungen stehen in der App unter „Mehr → Rechtliches → Nutzungsbedingungen“.
