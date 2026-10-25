Stand: 2026-10-25 (YYYY-MM-DD)

# Datenformat `watt-daten`

WATT exportiert die erfassten Daten im Format `watt-daten` als JSON oder als XML. Hier findest du die Schemas aller veröffentlichten Formatversionen, den Zusammenhang zwischen XML und JSON, eine Anleitung zum Prüfen einer Datei und die Änderungen je Version.

## Versionen

| Version | Schemata | Genutzt von | JSON einlesbar bis |
| --- | --- | --- | --- |
| 19 (Vorabfassung) | `watt-daten-19.schema.json`, `watt-daten-19.xsd` | noch keine ausgelieferte Fassung | – |
| 18 | `watt-daten-18.schema.json`, `watt-daten-18.xsd` | WATT 0.8.109 „Cherimoya“ | WATT 0.8.109 „Cherimoya“ |
| 17 | `watt-daten-17.schema.json` | WATT 0.8.100 „Bergamotte“ | WATT 0.8.109 „Cherimoya“ |
| 16 | `watt-daten-16.schema.json` | WATT 0.8.94 „Apfel“ | WATT 0.8.109 „Cherimoya“ |

Version 19 ist eine Vorabfassung für die nächste Fassung von WATT; sie ist noch nicht ausgeliefert und kann sich bis dahin ändern. Erst mit der Auslieferung gilt sie als veröffentlicht und unveränderlich.

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

### Version 19 (Vorabfassung)

- `nachweise.anzeigen[].anhaenge`: fremde PDFs und Fotos zur Anzeige (`id`, `art`, `name`, `groesse`, `erstelltAm`, optional `teil` bzw. `dateiname`); im XML `anhang`.
- Änderungs-Glieder von `eintrag` und `abwesenheit` tragen das optionale Feld `begruendung` (Kommentar zur Änderung) in `daten`.
- `protokoll[].neustarts` und `protokoll[].autoZeit` (Uhrstand, nur bei neuen Gliedern; gehen in die Prüfsumme ein).
- Neue Gliedarten `uhr_verstellt` und `schluessel`; das Glied `wiederherstellung` nennt `signaturen` und gegebenenfalls `signaturenUngueltigAb`.
- Neue Listen `kettenschluessel` (Fingerabdruck, Zertifikatskette) und `kettensignaturen` (seq, Fingerabdruck, Signatur); im XML `schluessel`, `zertifikat`, `signatur`.

Einzelheiten: [Technische Erläuterung](https://github.com/Aves83/watt_public/blob/main/doc/technische-erlaeuterung.md), Abschnitt 4.7, und [Datenformat](https://github.com/Aves83/watt_public/blob/main/doc/datenformat.md).

### Version 18

- Erste Version mit XML-Export; dazu das XML Schema `watt-daten-18.xsd`.
- Neues Feld `umfang` (`VOLLSTAENDIG` oder `BUCHUNGEN`): Mit `BUCHUNGEN` enthält die Datei nur die Buchungen eines Zeitraums; `profil`, `stundenplan`, `einstellungen`, `laufenderTimer`, `nachweise` und `kalenderlink` sind dann `null`, `protokoll`, `funktionen`, `kalenderZuordnungen` und `kalenderEntscheidungen` leer. Fehlt das Feld, gilt `VOLLSTAENDIG`.
- Neues Feld `importiertAm` in `eintraege[]` und `abwesenheiten[]` (Zeitpunkt des Einlesens aus einer Datei mit `umfang` `BUCHUNGEN`, sonst `null`).
- `profil.geburtMonat` als Monat `JJJJ-MM`, sobald eine Regel des Landes ihn braucht.
- Neuer Wert `ARZTBESUCH` in `abwesenheiten[].art` mit den neuen Feldern `beginn` und `ende`.
- Neuer Wert `SCHULFAHRT` in `sonderTage[].grund`.
- `protokoll[].objektArt`: neuer Wert `uhr_abweichung`; `daten` des Glieds `neubeginn` zusätzlich mit `vorherAnzahl`, `vorherLetzterHash`, `vorherZeitpunkt`, `eintraege` und `abwesenheiten`, `daten` des Glieds `wiederherstellung` zusätzlich mit `vorgeschichte`.
- `kalenderlink` ist im Klartext-Export immer `null`.

### Version 17

- Neue Felder: `kalenderlink`, `kalenderZuordnungen`, `kalenderEntscheidungen`.
- Neue Felder in `stundenplan`: `quelle`; in `stundenplan.eintraege[]`: `ausKalenderlink`.
- Neuer Wert `KALENDERLINK` in `eintraege[].quelle`.
- Korrektur (20.10.2026): Ferienarten `WINTER` und `EINZELTAG` in `ferien[].art` ergänzt, die WATT schon schrieb (JSON Schema und XSD).

### Version 16

- Erste veröffentlichte Version.
- Korrektur (20.10.2026): Ferienarten `WINTER` und `EINZELTAG` in `ferien[].art` ergänzt, die WATT schon schrieb.

## Lizenz

Die Dateien in diesem Ordner (Schemas und diese Beschreibung) stehen unter der Lizenz [Creative Commons Namensnennung 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/deed.de). Namensnennung: „Hans-Joachim Steudel, WATT – Workload Analysis & Time-Tracking for Teachers“. WATT selbst ist proprietär; es gelten die [Nutzungsbedingungen](../doc/license.txt).
