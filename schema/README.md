Stand: 2026-10-18 (YYYY-MM-DD)

# Datenformat `watt-daten`

WATT exportiert die erfassten Daten im Format `watt-daten` als JSON oder als XML. Hier findest du die Schemas aller veröffentlichten Formatversionen, den Zusammenhang zwischen XML und JSON, eine Anleitung zum Prüfen einer Datei und die Änderungen je Version.

## Versionen

| Version | Schemata | Genutzt von | JSON einlesbar bis |
| --- | --- | --- | --- |
| 17 | `watt-daten-17.schema.json` | WATT 0.8.100 „Bergamotte“ | WATT 0.8.100 „Bergamotte“ |
| 16 | `watt-daten-16.schema.json` | WATT 0.8.94 „Apfel“ | WATT 0.8.100 „Bergamotte“ |

„Genutzt von“ nennt die Fassungen von WATT, die diese Version schreiben, „JSON einlesbar bis“ die neueste Fassung, die eine JSON-Datei dieser Version einlesen kann.

Nachfolgend die allgemeinen Regeln für beide Formate und die Änderungen je Version.

## Zusammenhang von XML und JSON

Die XML-Datei bildet das JSON eins zu eins ab:

- Das JSON-Objekt der Datei wird zum Wurzelelement `wattDaten`; jedes Feld eines Objekts wird zu einem gleichnamigen Kindelement in der Reihenfolge des JSON.
- Eine Liste wird zu einem Element mit dem Feldnamen, jedes Listenelement zu einem Kindelement mit dem Namen im Singular (z. B. `eintraege` → `eintrag`, `protokoll` → `glied`, `kette` → `glied`, `raster` → `stunde`); eine leere Liste ist ein leeres Element.
- Das Verzeichnis `antworten` mit freien Schlüsseln wird zu einem Element mit einem Kindelement `antwort` je Paar; der Schlüssel steht im Attribut `schluessel`.
- Text steht als Inhalt des Elements, Zahlen und Wahrheitswerte in der Schreibweise des JSON (`22.5`, `true`); ein fehlender Wert (`null`) ist ein leeres Element mit `xsi:nil="true"`.

XML und JSON können aus WATT direkt exportiert werden, importiert werden kann aber nur JSON.

## Prüfen

JSON Schema, z. B. mit [ajv](https://ajv.js.org/):

```
npx ajv-cli validate -s watt-daten-<Version>.schema.json -d <Datei>.json --spec=draft2020
```

XML Schema, z. B. mit xmllint:

```
xmllint --noout --schema watt-daten-<Version>.xsd <Datei>.xml
```

## Änderungen je Version

### Version 17

- Neue Felder: `kalenderlink`, `kalenderZuordnungen`, `kalenderEntscheidungen`.
- Neue Felder in `stundenplan`: `quelle`; in `stundenplan.eintraege[]`: `ausKalenderlink`.
- Neuer Wert `KALENDERLINK` in `eintraege[].quelle`.

### Version 16

- Erste veröffentlichte Version.

## Lizenz

Die Dateien in diesem Ordner (Schemas und diese Beschreibung) stehen unter der Lizenz [Creative Commons Namensnennung 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/deed.de). Namensnennung: „Hans-Joachim Steudel, WATT – Workload Analysis & Time-Tracking for Teachers“. WATT selbst ist proprietär; es gelten die Nutzungsbedingungen in [`LICENSE_WATT`](../LICENSE_WATT).
