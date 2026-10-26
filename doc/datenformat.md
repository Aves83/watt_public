Stand: 2026-10-26 (YYYY-MM-DD)

# Datenformat von WATT

Stand: 25.10.2026 · Format `watt-daten`, Version 19, als JSON und XML · verschlüsselte Sicherung im Container der Version 3 · Schemas in `docs/schnittstelle/`, veröffentlicht in watt_public unter `schema/`

| Version | Datum | Änderung |
| --- | --- | --- |
| 1 | 29.09.2026 | Erste Fassung |
| 2 | 01.10.2026 | `symbolKey` je Kategorie (F-80). Dateien der Version 1 bleiben lesbar; ihre Kategorien erhalten das Standardsymbol `category` |
| 3 | 01.10.2026 | `erfassungsbeginn` im Profil (F-86). Dateien der Versionen 1 und 2 bleiben lesbar; ohne das Feld zählen alle Tage |
| 4 | 01.10.2026 | `nachweise` mit „Meine Daten“, Anzeigen samt Dokumenten (Base64), Notizen, Versandangaben und Wiedervorlagen, Gesprächsnotizen mit Nachträgen und der Nachweiskette (F-90). Dateien der Versionen 1 bis 3 bleiben lesbar; ohne das Feld lässt die Wiederherstellung die Nachweise des Geräts unverändert |
| 5 | 02.10.2026 | `ferien[].quelle` als `KMK`, `ICS` oder leer (F-96), `sonderTage[].grund` für die Art des unterrichtsfreien Tages (F-100), dienstfreie Tage als Abwesenheit `DIENSTFREI` statt als Sondertag (F-101). Dateien der Versionen 1 bis 4 bleiben lesbar: `VORBELEGUNG` und `EIGEN` werden zu `KMK`, wenn der Eintrag genau einem mitgelieferten Termin ohne eigene Bezeichnung entspricht, sonst zu leer; Sondertage `DIENSTFREI` werden zu Abwesenheiten `DIENSTFREI` (sie ersetzen eine Abwesenheit desselben Tages); unterrichtsfreie Tage erhalten den Grund `SONSTIGES` |
| 6 | 03.10.2026 | Verschlüsselte Sicherung im Container der Version 2: Inhalt mit Deflate komprimiert, zufälliger Sicherungsschlüssel, im Kopf zweifach umhüllt (Passphrase und Notfallblatt), Erstellungszeitpunkt im Kopf (F-106, F-107). Die Felder des JSON-Textes sind unverändert. Dateien der Versionen 1 bis 5 und Container der Version 1 bleiben lesbar |
| 7 | 04.10.2026 | `stundenplan.zykluslaenge` (1 bis 6 Wochen, F-119), `woche` der Plan-Einträge `A` bis `F`, dazu `fach`, `klasse` und `raum` je Plan-Eintrag (F-120). `abWochenAktiv` bleibt erhalten und heißt „Zyklus länger als eine Woche“. Dateien der Versionen 1 bis 6 bleiben lesbar: ohne `zykluslaenge` gilt 2 bei `abWochenAktiv` = `true`, sonst 1; Fach, Klasse und Raum sind dann leer |
| 8 | 04.10.2026 | `nachweise.anzeigen[].pruefverfahren` (1 oder 2): Verfahren der Prüfsumme je Anzeige (E-251); `nachweise.anzeigen[].kettenkopf` mit dem Kopf der Arbeitszeitkette beim Erzeugen (F-127); `profil.bezugszeitraumWochen` entfällt, die Bezugszeiträume sind feste Regeldaten (F-125). Dateien der Versionen 1 bis 7 bleiben lesbar; ohne `pruefverfahren` gilt Verfahren 1, ein `bezugszeitraumWochen` wird ignoriert |
| 9 | 05.10.2026 | `profil.schulart` ist `BERUFLICHE_SCHULEN` oder `GYMNASIUM`, dazu `profil.unterart` (F-135); `profil.deputat`, `anrechnungsstunden` und `ermaessigungsstunden` entfallen, die Unterrichtspflichtzeit steht als `nachweise.meineDaten.unterrichtspflichtzeit` in den Grunddaten (F-138). Dateien der Versionen 1 bis 8 bleiben lesbar: eine einzelne berufliche Schulart oder keine Angabe ergibt `BERUFLICHE_SCHULEN` ohne Unterart, Deputat, Anrechnungs- und Ermäßigungsstunden werden ignoriert |
| 10 | 06.10.2026 | `profil.erfassungsbeginn` entfällt: Der Erfassungsbeginn ist keine Einstellung mehr, sondern der Tag des ältesten Eintrags jeder Art. Dateien der Versionen 1 bis 9 bleiben lesbar; ein dort genannter Erfassungsbeginn wird ignoriert |
| 11 | 06.10.2026 | Schlagworte `eintraege[].fach` und `eintraege[].klasse` (F-06) und `einstellungen.schlagworteEingeblendet` für die Auswertung nach Fach und Klasse. Dateien der Versionen 1 bis 10 bleiben lesbar: Ohne die Felder haben die Einträge keine Schlagworte, und nichts ist zugeschaltet |
| 12 | 07.10.2026 | Amt im Profil: `profil.amt`, `profil.amtsform`, `profil.amtBezeichnung` (F-154) und `profil.funktionenBestaetigt` (F-153); Funktionen mit Gültigkeit und Anrechnungsstunden als eigene Liste `funktionen` (F-155); `nachweise.meineDaten.amtsbezeichnung` entfällt, die Amtsbezeichnung setzt sich aus Amt und Funktionen zusammen. Dateien der Versionen 1 bis 11 bleiben lesbar: ohne die Felder ist kein Amt gewählt, nichts bestätigt und keine Funktion vorhanden; ein `amtsbezeichnung` wird ignoriert |
| 13 | 08.10.2026 | `profil.amtsform` ist `WEIBLICH`, `MAENNLICH` oder `NEUTRAL` und gilt für jedes Amt außer `SONSTIGES` (F-160); `profil.vorbereitungsdienstBeginn` (F-159). Das Änderungsprotokoll enthält nur noch Arbeitszeit-Einträge, Abwesenheiten (ohne dienstfreie Tage) und Vorgänge am Protokoll; Glieder von Einträgen nennen Kategorie und Unterkategorie im Klartext (F-162). Dateien der Versionen 1 bis 12 bleiben lesbar: `DOPPELPUNKT` wird als `NEUTRAL` gelesen, eine Form gilt unabhängig vom Amt; ohne `vorbereitungsdienstBeginn` ist kein Beginn angegeben |
| 14 | 08.10.2026 | `vorbild` je Kategorie und Unterkategorie: „Ähnlich wie …“ (F-163); `eintraege[].plausibilitaetBestaetigt`: Vermerk „Plausibilität bestätigt“ (F-165). Beide stehen nicht im Änderungsprotokoll. Dateien der Versionen 1 bis 13 bleiben lesbar: ohne die Felder gilt „ohne Angabe“ bzw. nichts bestätigt |
| 15 | 10.10.2026 | Profil: `wochenstunden` und `urlaubstage` entfallen (Regeldaten, F-169); neu `geburtHalbjahr` (mit vorbereiteten `geburtMonat`, `geburtTag`), `gdbAnfang` und `gdbAenderungen`, `urlaubWahl`, `unterrichtVd`, `deputatStartwert`, `deputatAusgleiche`, `vorbereitungsdienstEnde` (F-170, F-171, F-175, F-176, F-181, F-182). `beschaeftigung[]` mit `pflichtstunden`, `pflichtstundenMinuten`, `teilzeitStunden` und `altersteilzeit` statt `prozent`. `funktionen[].instanz` und neue Arten (F-180). `nachweise.meineDaten.unterrichtspflichtzeit` entfällt (F-185). Glieder `eintrag` können `umgewandeltAus` tragen (F-179). Dateien der Versionen 1 bis 14 bleiben lesbar: Wochenstunden, Urlaubstage, Prozent und Unterrichtspflichtzeit werden ignoriert, die Beschäftigung gilt als Vollzeit ohne gewählte Pflichtstunden, Geburtszeitraum und GdB fehlen (Pflichtangaben, F-173); ältere Funktionsarten werden abgebildet |
| 16 | 11.10.2026 | Alle Länder außer Hamburg (F-187): `profil.bundesland`, `schulart`, `unterart` und `amt` sind Kennungen der Regeldaten des gewählten Landes (F-188, F-189), `schulart` darf `null` sein. Neu `profil.regionaleFeiertage` (Kennungen der eingeschalteten regionalen Feiertage, F-195) statt `mariaeHimmelfahrt` und `augsburgerFriedensfest`; `profil.antworten` statt `urlaubWahl`, zusätzlich mit den Antworten der Ja/Nein-Fragen und den Stunden der Ermäßigungen auf Antrag (F-190, F-192); `beschaeftigung[].pflichtstundenAnderer` für „Anderer Wert“ (F-191). Dateien der Versionen 1 bis 15 bleiben lesbar: Die beiden Feiertage werden zu `regionaleFeiertage`, `urlaubWahl` zu `antworten`; eine Schulart, die das Land nicht kennt, entfällt (vor Version 9: einzelne berufliche Schularten werden zu `BERUFLICHE_SCHULEN`) |
| 17 | 12.10.2026 | Kalenderlink (F-211): `kalenderlink` (Text oder `null`, der Link selbst, nur in der Sicherung und nie in Berichten oder Exporten), `stundenplan.quelle` (`HAND` oder `KALENDERLINK`), `stundenplan.eintraege[].ausKalenderlink` für Musterstunden aus dem Kalenderlink (F-212), `kalenderZuordnungen` (Zuordnung je Fach, F-212) und `kalenderEntscheidungen` (Entscheidungen zu Vorschlägen, F-213); `eintraege[].quelle` kennt zusätzlich `KALENDERLINK`. Der Zwischenstand der Termine und der Stand der Abrufe stehen nicht in der Datei. Dateien der Versionen 1 bis 16 bleiben lesbar: ohne die Felder gibt es keinen Link, der Plan ist von Hand, nichts ist zugeordnet oder entschieden (`KalenderlinkDienstTest`) |
| 18 | 17.10.2026 | `abwesenheiten[].beginn` und `ende` für den Arztbesuch (`art` `ARZTBESUCH`, stundenweise, F-229), `null` bei ganztägigen Arten; `sonderTage[].grund` kennt zusätzlich `SCHULFAHRT` (F-230). Das Glied `abwesenheit` eines Arztbesuchs trägt in `daten` nach `art` die Felder `beginn` und `ende` (UTC), ganztägige Glieder bleiben wie bisher. Kopffeld `umfang` (`VOLLSTAENDIG` oder `BUCHUNGEN`, seit 18.10.2026, F-234): Dateien mit `BUCHUNGEN` enthalten nur die Buchungsdaten eines Zeitraums und ergänzen beim Einlesen (F-235). `eintraege[].importiertAm` und `abwesenheiten[].importiertAm` für eingelesene Buchungen; Anlage-Glieder eingelesener Buchungen tragen `importDatei`. Dateien der Versionen 1 bis 17 bleiben lesbar: Sie enthalten weder Arztbesuche noch Schulfahrten, ihre Abwesenheiten sind ganztägig, und ohne `umfang` gilt `VOLLSTAENDIG` Seit 20.10.2026 schreibt WATT `profil.geburtMonat` (Hessen, F-170), bisher immer `null`. |
| 19 | 23.10.2026 | `nachweise.anzeigen[].anhaenge` (F-241): fremde PDFs und Fotos zur Anzeige mit `id`, `art`, `name`, `groesse`, `erstelltAm`; in der verschlüsselten Sicherung (Container der Version 3, 24.10.2026) `teil` mit der Nummer des Anhangsteils, im Klartext-Export mit Anhängen (ZIP) `dateiname`. Das Glied `geaendert` eines Eintrags oder einer Abwesenheit trägt wie das Lösch-Glied das optionale Feld `begruendung` („Kommentar zur Änderung“, F-240); es geht in die Prüfsumme ein. Seit dem 24.10.2026 (F-242, F-243, E-743): `protokoll[].neustarts` und `autoZeit` (Uhrstand, nur bei neuen Gliedern), Gliedarten `uhr_verstellt` und `schluessel`, Listen `kettenschluessel` und `kettensignaturen`; seit dem 25.10.2026 (E-752 bis E-756) Grenzen und Prüfungen beim Lesen von Schlüsseln und Signaturen, Befunde `WIDERLEGT` und `NICHT_PRUEFBAR` im Glied `wiederherstellung`, `bestandsform` 2 im Glied `neubeginn` und `schritte` von `uhr_verstellt` als Zahl der Messungen seit dem letzten Vermerk; seit dem 26.10.2026 (E-766) tragen Änderungs- und Lösch-Glieder von `eintrag` und `abwesenheit` mit Uhrstand am Ende von `daten` das Feld `importiertAm`, wenn die Buchung eingelesen wurde. Dateien der Versionen 1 bis 18 bleiben lesbar: ohne `anhaenge` hat eine Anzeige keine Anhänge, ohne Uhrstand rechnen Glieder mit sieben Feldern, ohne die Listen gibt es keine Signaturen |

WATT speichert alle Daten verschlüsselt auf dem Gerät. Damit sie der Lehrkraft gehören (N-05), lassen sie sich jederzeit vollständig in einem offenen Format exportieren und ohne Verlust wieder einlesen. Dieses Dokument beschreibt das Format. Der Code dazu steht in `core/data/src/main/kotlin/de/achim/watt/core/data/sicherung/`.

## Projektstatus und Kompatibilität

Derzeit **private Beta** (F-108).

Kompatibilität des Datenformats und der Datenbank nach Projektstatus (F-108):

- Private Beta: keine Zusage.
- Öffentliche Beta: Wir bemühen uns um Kompatibilität, ohne Zusage.
- Nach der öffentlichen Beta (stabile Version) verbindlich: Jede veröffentlichte Formatversion bleibt dauerhaft einlesbar, Datenbank-Migrationen von jeder veröffentlichten Version laufen ohne Datenverlust, und je veröffentlichter Formatversion liegt eine Beispieldatei im Repository, die ein Test einliest.

## Dateien

| Datei | Inhalt | Endung |
| --- | --- | --- |
| Klartext-Export (JSON) | JSON in UTF-8, wie unten beschrieben | `.json` |
| Klartext-Export (XML) | derselbe Inhalt als XML in UTF-8, Abbildung unter „XML-Darstellung“ (seit 16.10.2026, F-223); WATT liest XML nicht ein | `.xml` |
| Buchungsdaten (JSON oder XML) | Klartext mit `umfang` `BUCHUNGEN`: nur Einträge, Abwesenheiten, Tagesmarkierungen und Ferien eines Zeitraums (seit 18.10.2026, F-234); WATT liest nur die JSON-Fassung ein | `.json`, `.xml` |
| Verschlüsselte Sicherung | derselbe JSON-Text, komprimiert und verschlüsselt (Container, siehe unten) | `.wattbak`, bis 02.10.2026 `.wattbackup` |

Die Sicherung entsteht unter „Mehr → Daten, Sicherheit & Export → Sicherung“, der vollständige Klartext unter „Mehr → Daten, Sicherheit & Export → Export“ (Format JSON oder XML wählbar, vor dem Export die Rückfrage nach F-178), die Buchungsdaten im Export-Dialog des Überblicks (F-234, mit Zeitraum und Kategorienfilter, ebenfalls mit Rückfrage); Sicherung, JSON-Klartext und JSON-Buchungsdaten lassen sich unter „Sicherung“ wieder einlesen; die verschlüsselte Sicherung entsteht auch automatisch (F-105). Automatische Sicherungen heißen `WATT-Sicherung_JJJJ-MM-TT_HHMM.wattbak` (Ortszeit), die Datei für die Google-Sicherung liegt als `files/sicherung/WATT-Sicherung.wattbak` im App-Speicher; ist die vollständige Datei mit Anhängen 20 MB groß oder größer, liegt sie als `files/sicherung-geraet/WATT-Sicherung.wattbak` nur für die Geräteübertragung, und `files/sicherung/` enthält die Fassung ohne Anhangsinhalte (PB 2026-10-24 EFG E1, Abschnitt „Container Version 3“). Die Wiederherstellung ersetzt alle Daten auf dem Gerät und wird im Änderungsprotokoll vermerkt; enthält die Datei Nachweise, auch in der Nachweiskette (siehe `nachweise`). Eingespielt wird nur, wenn `eintraege` und `abwesenheiten` genau dem Stand entsprechen, den das nachgespielte `protokoll` der Datei beschreibt (F-237, Einzelheiten in der Technischen Erläuterung, Abschnitt 7); Dateien der Versionen 1 bis 15 sind lesbar, lassen sich aber nicht abgleichen und werden nicht mehr eingespielt. Die Vorgeschichte (Kette des Geräts gegen Kette der Datei) steht im Glied `wiederherstellung` (F-238). Der Kalenderlink steht nur in der verschlüsselten Sicherung; der Klartext und alle Prüfsummen enthalten ihn nie. Buchungsdaten ersetzen nichts, sondern ergänzen (Abschnitt „Buchungsdaten“).

## Grundregeln

- Zeitpunkte stehen als ISO 8601 in UTC, z. B. `2026-09-29T08:00:00.123Z`.
- Kalendertage stehen als `JJJJ-MM-TT`, Uhrzeiten als `HH:MM`. Kalendertage und Uhrzeiten gelten in der Zeitzone `Europe/Berlin`.
- Aufzählungen stehen als Name der Konstante, z. B. `KRANKHEIT`.
- Der Export enthält alle Felder, auch leere (`null`).
- Beim Einlesen werden unbekannte Felder übergangen. Fehlt ein Pflichtfeld oder ist ein Wert ungültig, wird die Datei abgelehnt und nichts geändert.
- Eine Datei mit höherer `version` wird abgelehnt; die App muss dann aktualisiert werden.

## Aufbau

| Feld | Typ | Bedeutung |
| --- | --- | --- |
| `format` | Text | immer `watt-daten` |
| `version` | Zahl | Version des Formats, derzeit 18 |
| `exportiertAm` | Zeitpunkt | Zeitpunkt des Exports |
| `appVersion` | Text | Version der App |
| `umfang` | `VOLLSTAENDIG` oder `BUCHUNGEN` | Seit 18.10.2026 (F-234): alle Daten bzw. nur die Buchungsdaten eines Zeitraums (Abschnitt „Buchungsdaten“); fehlt das Feld, gilt `VOLLSTAENDIG` |
| `profil` | Objekt oder `null` | Soll-Parameter; `null` bei `BUCHUNGEN` |
| `kategorien` | Liste | Oberkategorien in der Reihenfolge der Anzeige, jeweils mit Unterkategorien |
| `ferien` | Liste | Ferienzeiträume |
| `sonderTage` | Liste | unterrichtsfreie Tage (Tagesmarkierung); bis Version 4 auch dienstfreie Tage |
| `stundenplan` | Objekt oder `null` | Zeitraster, Plan-Einträge, Blöcke, Einstellungen; `null` bei `BUCHUNGEN` |
| `eintraege` | Liste | erfasste Tätigkeiten |
| `abwesenheiten` | Liste | Abwesenheitstage |
| `protokoll` | Liste | Änderungsprotokoll mit Hash-Kette |
| `laufenderTimer` | Objekt oder `null` | laufender Timer zum Zeitpunkt des Exports |
| `einstellungen` | Objekt oder `null` | Einstellungen der Oberfläche; `null` bei `BUCHUNGEN` |
| `nachweise` | Objekt | Nachweise mit Nachweiskette (seit Version 4); fehlt in älteren Dateien. Seit 19.10.2026 lehnt WATT eine vollständige Datei ab Version 4 ohne `nachweise` ab (B-6); in Buchungsdaten ist es `null` |
| `funktionen` | Liste | Funktionen mit Gültigkeit und Anrechnungsstunden (seit Version 12, F-155); fehlt in älteren Dateien |
| `kalenderlink` | Text oder `null` | Kalenderlink des Stundenplans (seit Version 17, F-211); fehlt in älteren Dateien. Nur die verschlüsselte Sicherung enthält ihn; der Klartext-Export (JSON und XML) schreibt immer `null`, und das Einlesen eines Klartexts lässt den Link des Geräts unverändert |
| `kalenderZuordnungen` | Liste aus `fach`, `kategorieId`, `unterkategorieId` (Text oder `null`), `nurVorschlag` (Wahrheitswert) | Zuordnung je Fach aus dem Kalenderlink (seit Version 17, F-212) |
| `kettenschluessel` | Liste aus `fingerabdruck` (SHA-256 über die SubjectPublicKeyInfo, hexadezimal), `zertifikate` (Liste, je Zertifikat Base64 DER, Blatt zuerst) | Schlüssel der Arbeitszeitkette mit der Zertifikatskette der Attestierung (seit 24.10.2026, F-243); fehlt ohne Schlüssel und in Buchungsdaten. Der private Schlüssel steht nirgends. Beim Lesen prüft WATT genau dies (seit 25.10.2026, E-756), sonst ist die Datei ungültig: höchstens 256 Schlüssel, je Schlüssel 1 bis 8 Zertifikate, je Zertifikat höchstens 8 KB (8192 Bytes), Base64 ohne Fremdzeichen (kein PEM, keine Zeilenumbrüche); jedes Zertifikat ist genau ein DER-Element ohne angehängte Bytes und als X.509 lesbar, nicht nur das Blatt; Fingerabdrücke eindeutig; der Fingerabdruck passt zum öffentlichen Schlüssel des Blatts. Ob die Attestierung zu WATT und zur Kette passt (Paket, App-Zertifikat, Challenge, `zertifikateSha256`, Signatur jedes Zertifikats durch das nächste), ist ein Befund der Vorschau und macht die Datei nicht ungültig. Ob der Schlüssel verfügbar war, steht nur im Glied `schluessel`. Zu jedem Glied `schluessel` mit `status` `verfuegbar` steht sein Schlüssel hier, auch nach Wiederherstellung und Neubeginn; fehlt er, ist das „nicht bestätigbar“, nicht „widerlegt“, und die Datei bleibt lesbar (U-24, E-765) |
| `kettensignaturen` | Liste aus `seq`, `fingerabdruck`, `signatur` (Base64, DER-kodiertes ECDSA) | Signaturen von Köpfen der Arbeitszeitkette (seit 24.10.2026, F-243): SHA256withECDSA über die ASCII-Nachricht `watt-kopf-1:<hash des Glieds seq>`; nicht im Hash. Fehlt ohne Signaturen und in Buchungsdaten. Beim Lesen prüft WATT (seit 25.10.2026), sonst ist die Datei ungültig: `seq` eindeutig und von 1 bis zur `seq` des letzten Glieds von `protokoll`, `signatur` Base64 ohne Fremdzeichen, nicht leer und höchstens 128 Bytes. Eine rechnerisch falsche Signatur macht die Datei nicht ungültig, sie ist ein Befund der Vorschau. Je Glied höchstens eine Signatur, nur zu Gliedern der Kette; mehrere Signaturen zu einem Glied schreibt WATT nie (U-23, E-765) |
| `kalenderEntscheidungen` | Liste aus `ref`, `stand` (`UEBERNOMMEN`, `VERWORFEN`, `VERFALLEN`), `zeitpunkt`, `tag`, `von`, `bis`, `fach`, `raum` | Entscheidungen zu Vorschlägen aus dem Kalenderlink (seit Version 17, F-213); `ref` ist `<UID>@<Beginn>` bzw. `ohne-uid:<Fach>@<Beginn>` |

### profil

| Feld | Typ | Bedeutung |
| --- | --- | --- |
| `wochenstunden`, `urlaubstage` | Zahl | nur bis Version 14: Wochenarbeitszeit und Urlaubstage; seit Version 15 kommen sie aus den Regeldaten des Landes und werden ignoriert (F-169) |
| `beschaeftigung` | Liste | Zeilen des Beschäftigungsumfangs ab einem Tag (`ab`): `pflichtstunden` (Zahl oder `null`, gewählte Pflichtstunden vor Ermäßigungen), `pflichtstundenMinuten` (Stundenlänge, Standard 45), `teilzeitStunden` (Zahl oder `null` für Vollzeit), `altersteilzeit` (Wahrheitswert), seit Version 15 (F-176). Seit Version 16 `pflichtstundenAnderer` (Wahrheitswert): die Pflichtstunden sind ein frei eingegebener „Anderer Wert“ (1 bis 40 in halben Stunden, F-191). Bis Version 14 `prozent` (0 bis 100); es wird ignoriert |
| `regionaleFeiertage` | Liste von Kennungen, z. B. `MARIAE_HIMMELFAHRT` | Eingeschaltete regionale Feiertage des Landes (seit Version 16, F-195); fehlt das Feld, gelten die beiden folgenden Felder bzw. der Standard des Landes |
| `mariaeHimmelfahrt`, `augsburgerFriedensfest` | Wahrheitswert | nur bis Version 15: 15. bzw. 8. August ist Feiertag (Bayern); seit Version 16 in `regionaleFeiertage`, nur gelesen |
| `bezugszeitraumWochen` | Zahl | nur bis Version 7: Bezugszeitraum des 48-Stunden-Schnitts; seit Version 8 entfällt das Feld, ältere Dateien werden ohne es gelesen (F-125) |
| `deputat` | Zahl | nur bis Version 8: Unterrichtsstunden je Woche; seit Version 9 entfällt das Feld und wird ignoriert (F-138) |
| `bundesland` | `BADEN_WUERTTEMBERG`, `BAYERN`, `BERLIN`, … `THUERINGEN` (Namen der Länder in Großbuchstaben, Umlaute als AE, OE, UE) | Land der Regeldaten (seit Version 16 jedes Land außer Hamburg wählbar, F-187); fehlt das Feld, gilt `BAYERN` |
| `schulart` | Kennung einer Schulart des Landes, z. B. `GYMNASIUM`, oder `null` | Seit Version 16 je Land (F-188); Schulart (seit Version 9, F-135); bestimmt die sichtbaren vorgepflegten Unterkategorien. Bis Version 8 eine einzelne berufliche Schulart oder `null`; beim Einlesen ergibt jeder andere Wert `BERUFLICHE_SCHULEN` |
| `unterart` | Kennung einer Unterart der Schulart oder `null` | Seit Version 16 je Land und Schulart (F-188); Unterart der beruflichen Schulen, nur Angabe (seit Version 9); passt sie nicht zur Schulart, wird sie ignoriert |
| `amt` | Kennung eines Amts des Landes, `SONSTIGES` oder `null` | Seit Version 16 je Land (F-189); Amt (seit Version 12, F-154); `null`, solange nicht gewählt (Pflichtangabe, F-153) |
| `amtsform` | `WEIBLICH`, `MAENNLICH`, `NEUTRAL` oder `null` | Form der Amtsbezeichnung in Anschreiben (seit Version 12): einmal gewählt, gilt sie für jedes Amt und bleibt beim Wechsel des Amts erhalten; bei `SONSTIGES` gilt `amtBezeichnung` (seit Version 13, F-160). Bis Version 12 `DOPPELPUNKT` statt `NEUTRAL` |
| `amtBezeichnung` | Text | Bezeichnung bei `SONSTIGES`, sonst leer (seit Version 12) |
| `funktionenBestaetigt` | Wahrheitswert | Die Funktionen wurden einmal bestätigt, auch „keine Funktion“ (seit Version 12, F-153); fehlt das Feld, gilt `false` |
| `vorbereitungsdienstBeginn` | Tag oder `null` | Beginn des Vorbereitungsdienstes bei `STUDIENREFERENDAR` (seit Version 13, F-159): ein Einstellungstermin des Landes (F-194); sonst `null`. Nach „Ja“ auf „Ist dein Vorbereitungsdienst beendet?“ stellt WATT das Amt auf das Eingangsamt des Landes um und leert das Feld (F-181, F-189) |
| `vorbereitungsdienstEnde` | Monatserster oder `null` | Erwartetes Ende nach „Nein, er dauert länger“ (seit Version 15, F-181); `null`: Ende nach der Regeldauer |
| `geburtHalbjahr` | `JJJJ-MM` oder `null` | Beginn des Halbjahresfensters der Geburt (`JJJJ-02` für Februar bis Juli, `JJJJ-08` für August bis Januar des Folgejahres, seit Version 15, F-170); `geburtMonat` (`JJJJ-MM`) ist gesetzt, sobald WATT den Geburtsmonat erfragt hat (seit 20.10.2026 in Hessen, F-170, E-624), sonst `null`; `geburtTag` ist vorbereitet und bisher `null` |
| `gdbAnfang` | `KEINER`, `GDB_30_40`, `GDB_30_40_GLEICHGESTELLT`, `GDB_50` bis `GDB_100` (in Zehnerschritten), `KEINE_ANGABE` oder `null` | GdB-Stufe ohne Datum („schon immer“, seit Version 15, F-171) |
| `gdbAenderungen` | Liste aus `seit` (`JJJJ-MM`) und `stufe` | Änderungen der GdB-Stufe ab einem Monat (seit Version 15) |
| `antworten` | Objekt aus Kennung und Zahl | Antworten zu Regeln des Landes (seit Version 16): Urlaubszuschläge auf Antrag (Tage) bzw. mit Bestandsschutz (0 oder 1), Ja/Nein-Fragen der Ermäßigungen (1 = Ja) und Stunden der Ermäßigungen auf Antrag (F-190, F-192); bei der Kennung `SB_BESCHEID` (Eintragung laut Bescheid, F-209) zählt die Zahl halbe Stunden |
| `urlaubWahl` | Objekt aus Regel-Kennung und Zahl | nur Version 15: Antworten zu Urlaubszuschlägen; seit Version 16 in `antworten`, nur gelesen |
| `unterrichtVd` | Liste aus `ab` (Tag) und `stunden` (0 bis 20 in halben Stunden) | Unterricht je Woche im Vorbereitungsdienst (seit Version 15, F-182) |
| `deputatStartwert` | Zahl | Startwert des Deputatskontos in Einzelstunden zum Erfassungsbeginn (seit Version 15, F-175) |
| `deputatAusgleiche` | Liste aus `id`, `tag`, `einzelstunden` (mit Vorzeichen), `notiz` | Eingetragene Ausgleiche des Deputatskontos (seit Version 15, F-175) |
| `anrechnungsstunden`, `ermaessigungsstunden` | Zahl | nur bis Version 8, seit Version 9 ignoriert (F-138) |
| `erfassungsbeginn` | Tag oder `null` | nur Version 3 bis 9: eingestellter erster Tag der Erfassung (F-86). Seit Version 10 entfällt das Feld und wird ignoriert: Der Erfassungsbeginn ist der Tag des ältesten Eintrags jeder Art (`eintraege[].tag`, `abwesenheiten[].tag`); Tage davor haben kein Soll und zählen in keiner Kennzahl |

### funktionen

Seit Version 12 (F-155). Jeder Eintrag ist ein Abschnitt einer Funktion; mehrere Abschnitte derselben Funktion (gleiche `art`, bei `WEITERE` gleiche `unterkategorieId`) dürfen sich nicht überschneiden.

| Feld | Typ | Bedeutung |
| --- | --- | --- |
| `id` | Text | Kennung des Abschnitts |
| `art` | `KLASSENLEITUNG`, `FACHBETREUUNG`, `KOORDINATION`, `AUSBILDUNG_MENTORING`, `SYSTEMBETREUUNG`, `BERATUNG`, `BEAUFTRAGUNG`, `INTERESSENVERTRETUNG`, `SCHULLEITUNG`, `STELLVERTRETENDE_SCHULLEITUNG`, `ERWEITERTE_SCHULLEITUNG`, `WEITERE` | Art der Funktion (seit Version 15, F-180). Bis Version 14 außerdem `SEMINARLEHRKRAFT` (gelesen als `AUSBILDUNG_MENTORING`), `OBERSTUFENKOORDINATION` (`KOORDINATION`), `BERATUNGSLEHRKRAFT` (`BERATUNG`) und `PERSONALRAT` (`INTERESSENVERTRETUNG`) |
| `instanz` | Text | Kennung eines Eintrags bei mehrfach anlegbaren Arten (`KOORDINATION`, `BERATUNG`, `BEAUFTRAGUNG`, `INTERESSENVERTRETUNG`), sonst leer (seit Version 15) |
| `gueltigAb` | Tag | erster Tag des Abschnitts |
| `gueltigBis` | Tag oder `null` | letzter Tag; `null` für offen |
| `anrechnungsstunden` | Zahl oder `null` | Anrechnungsstunden je Woche, 0 bis 30 mit höchstens zwei Nachkommastellen; `null` ohne Anrechnung |
| `anzahlKlassen` | Zahl oder `null` | Zahl der Klassen bei der Klassenleitung |
| `fach` | Text | Fach bzw. Fächer bei Fachbetreuung und Ausbildung und Mentoring, Bereich bzw. Freitext bei Koordination, Beratung, Beauftragung und Interessenvertretung, sonst leer |
| `bezeichnung` | Text | Bezeichnung einer weiteren Funktion, sonst leer |
| `unterkategorieId` | Text | Unterkategorie, in der die Zeit der Funktion verbucht wird: die vorgepflegte Zuordnung der Art (z. B. `verwaltung-3` für die Systembetreuung) oder bei `WEITERE` die eigene Unterkategorie `funktion-<Kennung>` unter `verwaltung` |

### kategorien

| Feld | Typ | Bedeutung |
| --- | --- | --- |
| `id` | Text | stabile Kennung; vorgepflegte Kategorien heißen z. B. `unterricht` |
| `name` | Text oder `null` | eigener Name; `null` heißt: Bezeichnung der App |
| `farbe` | Text | Farbe als `#AARRGGBB` |
| `hoechstdauerMinuten` | Zahl | Höchstdauer eines Timer-Laufs |
| `aktiv` | Wahrheitswert | erscheint als Kachel |
| `unterkategorien` | Liste aus `id`, `name`, `aktiv`, `vorbild` | in der Reihenfolge der Anzeige; `aktiv` ist die eigene Entscheidung der Lehrkraft. Ob eine vorgepflegte Unterkategorie zur Schulart gehört, steht nicht in der Datei, sondern folgt aus `profil.schulart` (F-135); auch Unterkategorien der anderen Schulart bleiben enthalten |
| `symbolKey` | Text oder `null` | Symbol als Name des Material Symbols, z. B. `co_present` (seit Version 2). Fehlt das Feld oder ist der Schlüssel der App unbekannt, gilt das Standardsymbol `category` |
| `vorbild` | Text oder `null` | „Ähnlich wie …“ (seit Version 14, F-163): Kennung einer vorgepflegten Unterkategorie, nach deren Vorbild WATT eine eigene Kategorie einordnet; `null` ohne Angabe (eigene Hauptkategorie: keine Zuordnung). Auch je Unterkategorie in `unterkategorien`; ohne Angabe gilt dort die Hauptkategorie. Unbekannte Kennungen gelten als ohne Angabe |

### ferien und sonderTage

| Feld | Typ | Bedeutung |
| --- | --- | --- |
| `ferien[].id` | Zahl | Kennung; das Änderungsprotokoll verweist darauf |
| `ferien[].art` | `SOMMER`, `ALLERHEILIGEN`, `WEIHNACHTEN`, `WINTER`, `FRUEHJAHR`, `OSTERN`, `PFINGSTEN`, `EINZELTAG`, `SONSTIGE` | Art der Ferien; `WINTER` und `EINZELTAG` (bewegliche Ferientage, F-197) schrieb WATT seit Version 16, die Beschreibung nennt sie seit 20.10.2026 (K-M06) |
| `ferien[].von`, `bis` | Tag | erster und letzter Ferientag |
| `ferien[].name` | Text oder `null` | eigene Bezeichnung |
| `ferien[].quelle` | `KMK`, `ICS` oder leerer Text | Herkunft (seit Version 5, F-96): Abruf bei der KMK bzw. mitgeliefert, iCal-Datei, leer für manuell. Bis Version 4 `VORBELEGUNG` oder `EIGEN` |
| `sonderTage[].tag` | Tag | Datum |
| `sonderTage[].art` | `UNTERRICHTSFREI` | Art; bis Version 4 auch `DIENSTFREI`, seit Version 5 eine Abwesenheit |
| `sonderTage[].name` | Text oder `null` | eigene Bezeichnung |
| `sonderTage[].grund` | `PAEDAGOGISCHER_TAG`, `PRUEFUNGSTAG`, `SCHULVERANSTALTUNG`, `SONSTIGES`, `SCHULFAHRT` | Art des unterrichtsfreien Tages (seit Version 5, F-100); fehlt das Feld in älteren Dateien, gilt `SONSTIGES`. `SCHULFAHRT` (seit Version 18, F-230) markiert jeden Kalendertag einer Schulfahrt, auch Wochenende und Ferien; zusammenhängende Tage bilden eine Fahrt |

### stundenplan

| Feld | Typ | Bedeutung |
| --- | --- | --- |
| `aktiv` | Wahrheitswert | Automatik an; `false` heißt rein manuelle Erfassung |
| `abWochenAktiv` | Wahrheitswert | Zyklus länger als eine Woche (`zykluslaenge` ≥ 2); bis Version 6 der Schalter für A/B-Wochen |
| `zykluslaenge` | Zahl | Wochen bis zur Wiederholung, 1 bis 6 (Wochen A bis F), seit Version 7; fehlt das Feld, gilt 2 bei `abWochenAktiv`, sonst 1 |
| `abBezugstag` | Tag oder `null` | ein Tag in der Referenzwoche (Woche A) |
| `abFerienwochenUeberspringen` | Wahrheitswert | Wochen ohne Unterrichtstag zählen beim Wechsel der Zykluswoche nicht mit |
| `uebernommenBis` | Tag oder `null` | letzter Tag, bis zu dem der Plan übernommen ist |
| `raster` | Liste aus `nummer`, `von`, `bis` | Zeitraster |
| `eintraege` | Liste | Plan-Einträge, siehe unten |
| `bloecke` | Liste aus `id`, `name`, `zeitraeume` (`von`, `bis`) | Blockunterricht |
| `ausgesetzteTage` | Liste von Tagen | Tage ohne Übernahme |
| `verworfen` | Liste aus `planRef`, `begruendung`, `zeitpunkt` | entfernte Plan-Einträge |
| `quelle` | `HAND`, `KALENDERLINK` | Quelle der wöchentlichen Stunden (seit Version 17, F-211); bei `KALENDERLINK` ruht ihre Übernahme, die Termine des Kalenderlinks zählen; fehlt das Feld, gilt `HAND` |

Plan-Eintrag:

| Feld | Typ | Bedeutung |
| --- | --- | --- |
| `id` | Text | Kennung |
| `typ` | `WOECHENTLICH`, `EINZELTERMIN` | Art |
| `wochentag` | Zahl oder `null` | 1 = Montag bis 7 = Sonntag, nur bei `WOECHENTLICH` |
| `tag` | Tag oder `null` | Datum, nur bei `EINZELTERMIN` |
| `stundeVon`, `stundeBis` | Zahl oder `null` | Stunden im Zeitraster |
| `beginn`, `ende` | Uhrzeit oder `null` | freie Uhrzeit, wenn keine Rasterstunden angegeben sind |
| `woche` | `A` bis `F` oder `null` | Woche im Zyklus; `null` heißt jede Woche, ebenso jede Woche bei `zykluslaenge` 1 |
| `blockId` | Text oder `null` | zugeordneter Block |
| `bezeichnung` | Text | freie Bezeichnung, z. B. aus älteren Plänen oder der CSV-Vorlage |
| `fach`, `klasse`, `raum` | Text | Angaben aus dem Wochenraster oder dem Datei-Import, seit Version 7; sonst leer |
| `kategorieId`, `unterkategorieId` | Text, Text oder `null` | Tätigkeit |
| `gueltigAb`, `gueltigBis` | Tag oder `null` | Gültigkeitszeitraum |
| `deaktiviert` | Wahrheitswert | wird nicht übernommen |
| `ausKalenderlink` | Wahrheitswert | Musterstunde aus dem Kalenderlink (seit Version 17, F-212), in der App nicht bearbeitbar; fehlt das Feld, gilt `false` |

### eintraege

| Feld | Typ | Bedeutung |
| --- | --- | --- |
| `id` | Text | Kennung |
| `tag` | Tag | Kalendertag, zu dem der Eintrag zählt (Tag des Beginns) |
| `kategorieId`, `unterkategorieId` | Text, Text oder `null` | Tätigkeit |
| `beginn`, `ende` | Zeitpunkt oder `null` | Zeitraum; beide `null` bei einem Tagesblock |
| `dauerMinuten` | Zahl | Dauer des Tagesblocks, sonst 0 |
| `notiz` | Text | Notiz |
| `quelle` | `MANUELL`, `TIMER`, `PLAN`, `KALENDERLINK` | Herkunft; `KALENDERLINK` seit Version 17 (F-212) |
| `planRef` | Text oder `null` | Herkunft im Stundenplan, `Plan-Eintrag@Tag@Beginn`; bei `KALENDERLINK` die Referenz des Termins `<UID>@<Beginn>` (F-212) |
| `erfasstAm` | Zeitpunkt | Zeitpunkt der Erfassung |
| `geaendertAm` | Zeitpunkt oder `null` | Zeitpunkt der letzten Änderung |
| `fach`, `klasse` | Text | Schlagworte Fach und Klasse (seit Version 11, F-06), leer ohne Angabe; bei Unterricht aus dem Stundenplan aus dem Plan-Eintrag übernommen |
| `plausibilitaetBestaetigt` | Wahrheitswert | Vermerk „Plausibilität bestätigt“ (seit Version 14, F-165): Die Lehrkraft hat eine Rückfrage zu ungewöhnlichen Werten mit „Ja, stimmt“ beantwortet. Nicht im Änderungsprotokoll; fehlt das Feld, gilt `false` |
| `importiertAm` | Zeitpunkt oder `null` | Zeitpunkt, zu dem der Eintrag aus einer Datei mit Buchungsdaten eingelesen wurde (seit 18.10.2026, F-235); `null` für Einträge dieses Geräts; fehlt das Feld, gilt `null`. Beim Einspielen muss er dem Zeitpunkt des Anlage-Glieds mit `importDatei` entsprechen (seit 20.10.2026, F-237); Änderungs-Glieder mit Uhrstand tragen ihn selbst in `daten` (`importiertAm`, seit 26.10.2026, E-766), Änderungs-Glieder ohne Uhrstand (bis Version 18) erben ihn, neue Anlagen ohne `importDatei` beginnen ohne. Fehlt er, obwohl das Protokoll das Einlesen belegt, ergänzt WATT ihn beim Einspielen aus dem Protokoll (seit 26.10.2026, E-763) |

Die Nachtrag-Kennzeichnung wird nicht gespeichert. Sie ergibt sich aus `ende` (bei Tagesblöcken dem Ende des Tages), `erfasstAm` und `geaendertAm`.

### abwesenheiten

| Feld | Typ | Bedeutung |
| --- | --- | --- |
| `tag` | Tag | Datum |
| `art` | `KRANKHEIT`, `SONDERURLAUB`, `DIENSTBEFREIUNG`, `DIENSTFREI`, `ARZTBESUCH` | Art; `DIENSTFREI` (Dienstfreier Tag, F-101) seit Version 5, `ARZTBESUCH` (stundenweise, F-229) seit Version 18 |
| `beginn`, `ende` | Zeitpunkt oder `null` | Beginn und Ende eines Arztbesuchs innerhalb des Kalendertags `tag` (seit Version 18, F-229); `null` bei ganztägigen Arten; fehlen die Felder in älteren Dateien, sind alle Abwesenheiten ganztägig |
| `importiertAm` | Zeitpunkt oder `null` | Zeitpunkt des Einlesens aus einer Datei mit Buchungsdaten (seit 18.10.2026, F-235); sonst `null`; beim Einspielen muss er zum Anlage-Glied mit `importDatei` passen (F-237) bzw. zum Feld `importiertAm` des letzten Änderungs-Glieds mit Uhrstand (E-766); er bleibt beim Ändern derselben Abwesenheit, auch beim Wechsel der Art; nach einem Lösch-Glied oder einem dienstfreien Tag beginnt eine neue Anlage ohne. Fehlt er, obwohl das Protokoll das Einlesen belegt, ergänzt WATT ihn beim Einspielen (seit 26.10.2026, E-763) |

### protokoll

| Feld | Typ | Bedeutung |
| --- | --- | --- |
| `seq` | Zahl | laufende Nummer ab 1 |
| `zeitpunkt` | Zeitpunkt | Zeitpunkt der Änderung |
| `aktion` | `ANGELEGT`, `GEAENDERT`, `GELOESCHT` | Art der Änderung |
| `objektArt` | Text | seit Version 13 (F-162) nur `eintrag`, `abwesenheit` (ohne dienstfreie Tage), `wiederherstellung`, `neubeginn`, `uhr_ruecksprung`, seit 20.10.2026 auch `uhr_abweichung` (F-239: Beginn eines Timer-Laufs nach der Monotonuhr korrigiert oder Neustart während des Laufs; `objektId` `timer`, `daten` `{"art":"korrektur"|"neustart","beginnUhr":…,"beginnMonoton":…|null,"zeitpunkt":…,"abweichungSekunden":n|null}`; `uhr_ruecksprung` mit `objektId` `timer` bei einem Timer (Start, Stopp, Wechsel, Vorverlegen, Höchstdauer) mit zurückgestellter Uhr, `nachweis` beim Festhalten eines Nachweises, seit dem 24.10.2026 ohne Anker auch `start` (Öffnen der App oder einer Seite) und `zeitsignal` (Android meldet eine Änderung der Uhrzeit); sonst die laufende Nummer des auslösenden Glieds; ein Vermerk zur Uhr erklärt seinen eigenen Rücksprung gegenüber dem vorigen Glied, gleich mit welcher `objektId`): Die Kette schützt das Ist, Einstellungen stehen nicht darin. Bis Version 12 außerdem `profil`, `funktion` (seit Version 12, F-155), `ferien`, `sondertag`, `kategorie`, `unterkategorie`, `stundenplan`; solche Glieder bleiben gültig und werden wie alle anderen geprüft. Seit dem 24.10.2026 auch `uhr_verstellt` (F-242: Änderung des Versatzes zwischen Wanduhr und Monotonuhr um mehr als 2 Minuten seit dem letzten Vermerk, am festen Anker je Gerätestart, seit 25.10.2026 E-753; `objektId` `glied` (vor dem folgenden Glied; nach „Sofort wieder löschen“ kann es fehlen), die laufende Nummer eines Glieds `neubeginn` bzw. `wiederherstellung` (der Vermerk folgt diesem), `zeitsignal`, `timer`, `nachweis` oder `start`; `daten` `{"vorher":…,"nachher":…,"abweichungSekunden":n,"richtung":"vor"|"zurueck"}`; `abweichungSekunden` mit Vorzeichen, negativ bei `zurueck`: `nachher` − `vorher` in ganzen Sekunden, zur Null hin gekürzt, also die Änderung des Versatzes seit dem letzten Vermerk; dazu `schritte`, wenn sich die Verstellung aus mehreren Messungen ergibt (Zahl der Messungen seit dem letzten Vermerk mit einer Änderung des Versatzes um mindestens eine Sekunde, ohne nachgeführte Netzzeit, ganze Zahl größer als 1; fehlt bei einem einzelnen Sprung), und `timerBeginnVorher`/`timerBeginnNachher`, wenn ein laufender Timer korrigiert wurde; bei leerer Kette kein Vermerk) und `schluessel` (F-243: `objektId` Fingerabdruck bzw. leer; `daten` `{"status":"verfuegbar","fingerabdruck":…,"zertifikateSha256":…,"sicherheitsstufe":"SOFTWARE"|"TEE"|"STRONGBOX"}`, bei leerer Kette dazu `challenge` (32 Bytes hexadezimal), bzw. `{"status":"nicht_verfuegbar","grund":…}`); `uhr_abweichung` mit `art` `neustart` entsteht seitdem nicht mehr |
| `objektId` | Text | Kennung des Objekts |
| `daten` | Text | Stand des Objekts als JSON-Text |
| `prev` | Text | `hash` des vorherigen Glieds, beim ersten `0` |
| `hash` | Text | SHA-256 über das Glied, hexadezimal in Kleinbuchstaben |
| `neustarts` | Zahl | Uhrstand (seit 24.10.2026, F-242): Neustarts des Geräts seit dem vorigen Glied, 0 beim ersten Glied einer Kette und bei den weiteren Gliedern eines Vorgangs, -1, wenn kein Bezug zum vorigen Glied besteht (erstes Glied nach dem Update, das Glied `wiederherstellung`, außer die eingespielte Kette endet mit demselben Kopf wie die des Geräts, Gerät ohne Monotonuhr oder Neustartzähler, Zähler zurückgegangen); `uhr_ruecksprung` folgt nur, wenn vor der Messung kein Anker bestand (Technische Erläuterung 4.7); fehlt bei älteren Gliedern |
| `autoZeit` | Wahrheitswert | Uhrstand (seit 24.10.2026, F-242): „Datum und Uhrzeit automatisch“ beim Anlegen; fehlt genau dann, wenn `neustarts` fehlt |

Glieder `eintrag` nennen seit Version 13 am Ende von `daten` die Tätigkeit im Klartext zum Zeitpunkt der Buchung: `kategorie` (Name der Kategorie) und `unterkategorie` (Name oder `null`), nach `begruendung` und gegebenenfalls `fach` und `klasse` (F-162). So bleibt jedes Glied ohne die Einstellungen eindeutig, und eine spätere Umbenennung verändert kein geprüftes Glied. Ältere Glieder haben die Felder nicht. Wandelt die Lehrkraft einen Eintrag vom Timer oder aus dem Stundenplan in einen Tagesblock um, trägt das Glied dieser Änderung zwischen `klasse` und `kategorie` das Feld `umgewandeltAus` mit der früheren Quelle (`TIMER`, `PLAN` oder seit Version 17 `KALENDERLINK`); der Eintrag hat danach die Quelle `MANUELL` und keine `planRef` mehr (seit Version 15, F-179). Sonst fehlt das Feld.

Lösch-Glieder (`GELOESCHT`) tragen in `daten` den letzten Stand des Objekts. Bei `eintrag` und `abwesenheit` steht dort zusätzlich die optionale Löschbegründung `begruendung` (Text oder `null`, F-117); bei Abwesenheiten ohne Begründung fehlt das Feld wie in älteren Dateien. Seit dem 23.10.2026 trägt auch ein Änderungs-Glied (`GEAENDERT`) von `eintrag` und `abwesenheit` dort den optionalen Kommentar zur Änderung `begruendung` (F-240, E-719): bei `eintrag` wie bisher immer vorhanden (`null` ohne Kommentar), bei `abwesenheit` nur mit Kommentar; die Form des Formats ändert sich dadurch nicht. Das Glied `neubeginn` (F-118, Aktion `ANGELEGT`, leere `objektId`) steht nur als erstes Glied einer Kette, die mit „Änderungsprotokoll löschen und neu beginnen“ begonnen wurde; `daten` ist `{"bestand":"<SHA-256>", …}` mit der Prüfsumme über den Klartext des Datenbestands ohne Protokoll, mit Zeitpunkt `1970-01-01T00:00:00Z` und leerer App-Version. Seit dem 19.10.2026 (Version 18, B-4) folgen `vorherAnzahl`, `vorherLetzterHash` und `vorherZeitpunkt` (Anzahl, Prüfsumme und Zeitpunkt des letzten Glieds der gelöschten Kette; ohne Kette `0`, `"0"` und `null`), `eintraege` (Objekt: Kennung → Bestandsprüfsumme) und `abwesenheiten` (Objekt: Tag → Bestandsprüfsumme, nur Abwesenheiten mit Gutschrift), beide nach Schlüssel sortiert. Die Bestandsprüfsumme sind die ersten 32 Hex-Zeichen der SHA-256 über die kanonische Form als JSON ohne Leerzeichen: bei Einträgen `id`, `tag`, `kategorieId`, `unterkategorieId`, `beginn`, `ende`, `dauerMinuten`, `notiz`, `quelle`, `planRef`, `erfasstAm`, `geaendertAm`, `fach`, `klasse` (Zeitpunkte auf Millisekunden, fehlende Werte `null`, fehlendes Fach und fehlende Klasse leer), bei Abwesenheiten `tag`, `art`, `beginn`, `ende` (Form 1, Rechenbeispiel B8 der Technischen Erläuterung). Seit dem 25.10.2026 folgt `bestandsform` mit dem Wert `2` (E-754): Dann steht in der kanonischen Form bei Einträgen und Abwesenheiten am Ende zusätzlich `importiertAm` (Zeitpunkt des Einlesens oder `null`, Rechenbeispiel B14); ohne `bestandsform` gilt Form 1. Der Abgleich (F-237) nimmt diesen Bestand als Ausgangsstand. Zum Nachrechnen von `bestand` werden `protokoll` durch `[]` ersetzt und `kettenschluessel` und `kettensignaturen` ganz entfernt. Ältere Start-Glieder haben nur `bestand`. Steht die Geräteuhr mehr als 2 Minuten vor dem letzten Glied, lehnt WATT den Neubeginn ab. Ein sofort wieder gelöschter Eintrag hinterlässt kein Glied (F-116). Das Glied `uhr_ruecksprung` (N-07, F-133, Aktion `ANGELEGT`) folgt unmittelbar auf ein Glied, dessen Zeitpunkt mehr als 2 Minuten vor dem seines Vorgängers liegt; `objektId` ist dessen `seq`, `daten` ist `{"vorher":"<Zeitpunkt>","nachher":"<Zeitpunkt>"}` in UTC. Diese Werte ändern das Format nicht; die Version bleibt. Das Glied `wiederherstellung` (Aktion `ANGELEGT`) trägt in `daten` `exportiertAm`, `verschluesselt` (`true` für eine Sicherung, `false` für einen Klartext), `vorherAnzahl`, `vorherLetzterHash`, gegebenenfalls `nachweise` und `nachweisketteUnterbrochenBei` seit dem 18.10.2026 `vorgeschichte` mit `UEBEREINSTIMMEND`, `ABWEICHEND` oder `NICHT_ABGLEICHBAR` (F-238; ältere Glieder ohne das Feld gelten als nicht abgeglichen) und seit dem 19.10.2026 `nachweisketteVorherAnzahl` und `nachweisketteVorherLetzterHash` mit Anzahl und letztem Hash der Nachweiskette, die vorher auf dem Gerät lag (B-6). Das Anlage-Glied (`ANGELEGT`) eines aus einer Datei mit Buchungsdaten eingelesenen Eintrags bzw. einer Abwesenheit trägt am Ende von `daten` das Feld `importDatei` mit der SHA-256 der eingelesenen Datei (seit 18.10.2026, F-235); sonst fehlt das Feld. Änderungs- und Lösch-Glieder (`GEAENDERT`, `GELOESCHT`) eines eingelesenen Eintrags bzw. einer eingelesenen Abwesenheit (auch Arztbesuch) mit Uhrstand tragen seit dem 26.10.2026 am Ende von `daten` das Feld `importiertAm` mit dem Zeitpunkt des Einlesens (UTC, E-766); sonst fehlt das Feld. Der Abgleich nimmt bei Gliedern mit Uhrstand diesen Wert (fehlt das Feld, gilt kein Vermerk), auch nach einem Neubeginn; nur Änderungs-Glieder ohne Uhrstand erben den Wert des Anlage-Glieds, und ändert ein solches Glied nach einem Neubeginn ein Objekt des Bestands, gilt der Wert der Datei. Die Kettenvermerke zählen diese Glieder bei `eintrag`.

Die Hash-Kette lässt sich ohne WATT prüfen. Für jedes Glied werden die Felder `seq`, `zeitpunkt`, `aktion`, `objektArt`, `objektId`, `daten` und `prev` in dieser Reihenfolge jeweils als `<Länge in UTF-8-Bytes>:<Wert>` aneinandergehängt; `hash` ist SHA-256 über diese Bytes. `seq` ist um 1 höher als beim Vorgänger, `prev` gleich dessen `hash`. Bei Gliedern mit Uhrstand (seit 24.10.2026) folgen nach `prev` ebenso `neustarts` (Dezimalzahl) und `autoZeit` (`true` oder `false`); ein Glied mit nur einem der beiden Felder ist ein Bruch. Ein Glied ohne Uhrstand nach dem ersten Glied mit Uhrstand ist kein Bruch, wenn sein `hash` stimmt; WATT schreibt es so aber nie, deshalb ist es „nicht bestätigbar“ und wird mit seiner `seq` genannt (seit 26.10.2026, E-764). Alle Werte werden mit UTF-8 kodiert; ein einzelnes Ersatzzeichen (ungültiges UTF-16) wird dabei zu `?` (0x3F). Das Glied `wiederherstellung` nennt seit dem 24.10.2026 den Befund der mitgelieferten Signaturen: `signaturen` (`GEPRUEFT`: alle rechnerisch gültig und die Prüfungen ohne Netz bestanden, `UNGUELTIG`, `KEINE`, seit dem 25.10.2026 auch `WIDERLEGT`: ein Schlüssel besteht die Prüfungen ohne Netz nicht, und `NICHT_PRUEFBAR`: die Prüfung war nicht möglich) und bei `UNGUELTIG` `signaturenUngueltigAb`. Nach jeder Wiederherstellung folgt ein neues Glied `schluessel` (E-738).

### laufenderTimer und einstellungen

| Feld | Typ | Bedeutung |
| --- | --- | --- |
| `laufenderTimer.kategorieId`, `unterkategorieId` | Text, Text oder `null` | Tätigkeit |
| `laufenderTimer.beginn` | Zeitpunkt | Beginn des Laufs |
| `einstellungen.unterkategorienEingeblendet` | Liste von Texten | Oberkategorien, deren Unterkategorien in Auswertung, Export und Bericht erscheinen |
| `einstellungen.schlagworteEingeblendet` | Liste aus `FACH`, `KLASSE` | Arten der Schlagworte, nach denen Auswertung, Export und Bericht zusätzlich je Oberkategorie summieren (seit Version 11, F-06) |

### nachweise

Seit Version 4 (F-90). Enthält alles aus dem Bereich „Nachweise“ (F-61 bis F-79). Zeitpunkte, Tage und Aufzählungen wie oben.

| Feld | Typ | Bedeutung |
| --- | --- | --- |
| `meineDaten.name`, `schule`, `adressat`, `empfaengerEmail` | Text | „Basisangaben zu Überlastungsanzeigen“ (zuvor „Grunddaten für Überlastungsanzeigen“, „Angaben für Anzeigen“ bzw. „Meine Daten“, F-62, F-185), leer, wenn nicht ausgefüllt |
| `meineDaten.kopien` | Liste aus `stelle` (`PERSONALRAT`, `SCHWERBEHINDERTENVERTRETUNG`, `GLEICHSTELLUNGSBEAUFTRAGTE`), `aktiv`, `email` | Kopie-Stellen, immer alle drei |
| `meineDaten.amtsbezeichnung` | Text | nur bis Version 11: Freitext „Amts- bzw. Funktionsbezeichnung“; seit Version 12 entfällt das Feld und wird ignoriert, die Bezeichnung entsteht aus `profil.amt` und `funktionen` (F-154, F-158) |
| `meineDaten.unterrichtspflichtzeit` | Zahl oder `null` | nur Version 9 bis 14: Unterrichtspflichtzeit für das Anschreiben (F-138); seit Version 15 entfällt das Feld und wird ignoriert, das Anschreiben nennt die Pflichtstunden (F-185) |
| `anzeigen` | Liste | Überlastungsanzeigen, Entwürfe und gesendete |
| `anzeigen[].id` | Text | UUID |
| `anzeigen[].erzeugtAm` | Zeitpunkt oder `null` | Erzeugung des Entwurfs; nach dem Markieren als gesendet `null` |
| `anzeigen[].stufe` | `AUSSAGE`, `ZUSAMMENFASSUNG`, `NACHWEIS` | Inhaltsstufe (F-65) |
| `anzeigen[].zeitraumVon`, `zeitraumBis` | Tag oder `null` | Zeitraum der Anzeige |
| `anzeigen[].angabenArt` | `PLATZHALTER`, `EINMALIG`, `MEINE_DATEN` | Herkunft der persönlichen Angaben (F-67) |
| `anzeigen[].angabenEntfernt` | Wahrheitswert | Angaben „nur für dieses Schreiben“ durch Vermerke ersetzt |
| `anzeigen[].empfaenger` | Text | E-Mail des Empfängers |
| `anzeigen[].kopien` | Liste aus `stelle`, `email` | Kopien dieser Anzeige |
| `anzeigen[].betreff`, `anschreiben` | Text | Betreff und Anschreiben |
| `anzeigen[].datum` | Tag | Datum des Schreibens |
| `anzeigen[].gesendetAn`, `gesendetAm`, `pruefsumme` | Text oder `null`, Zeitpunkt oder `null`, Text oder `null` | Versandangaben (F-73); `pruefsumme` ist der SHA-256 über den Gesamtstand beim Markieren, `null` bei Entwürfen |
| `anzeigen[].wiedervorlageAm`, `wiedervorlageErledigt` | Tag oder `null`, Wahrheitswert | Wiedervorlage (F-76) |
| `anzeigen[].wiederhergestelltAm`, `backupVom` | Zeitpunkt oder `null` | Vermerk „wiederhergestellt am … aus Backup vom …“ der letzten Wiederherstellung |
| `anzeigen[].dokumente` | Liste aus `art` (`ANSCHREIBEN`, `ZUSAMMENFASSUNG`, `DETAILNACHWEIS`), `dateiname`, `inhalt` | gespeicherte PDFs; `inhalt` ist Base64 (Standardalphabet mit Auffüllung) der unveränderten Datei |
| `anzeigen[].notizen` | Liste aus `id` (Zahl), `text`, `erstelltAm`, `fest`, `nachVersand` | Notizen (F-74); `fest` gehören zum Nachweis |
| `anzeigen[].kettenkopf` | Objekt aus `anzahl` (Zahl), `hash` (Text), `zeitpunkt` (Zeitpunkt) oder `null` | Kopf der Arbeitszeitkette beim Erzeugen, wie im Prüfvermerk (seit Version 8, F-127): laufende Nummer und Prüfsumme des jüngsten Glieds und dessen Zeitpunkt; bei leerem Protokoll `0`, `0` und der Zeitpunkt der Erzeugung. `null` bei älteren Anzeigen |
| `anzeigen[].pruefverfahren` | Zahl | Verfahren der Prüfsumme (seit Version 8): `1` für Anzeigen bis 04.10.2026, `2` für neue; fehlt das Feld, gilt `1`. Beschreibung beider Verfahren in `docs/app-texte/technische-erlaeuterung.md`, Abschnitt 5.1 |
| `anzeigen[].anhaenge` | Liste aus `id` (Zahl), `art` (`PDF`, `FOTO`), `name`, `groesse` (Zahl, Bytes), `erstelltAm` (Zeitpunkt), optional `dateiname`, optional `teil` (Zahl ab 0) | Anhänge der Anzeige (seit Version 19, F-241): fremde PDFs und Fotos, nicht in der Prüfsumme der Anzeige und nicht in der Nachweiskette; Name, Art, Größe und Datum gehen in die Bestandsprüfsumme beim Neubeginn ein. In der verschlüsselten Sicherung steht `teil`, die Nummer des Anhangsteils hinter dem Hauptteil (Container der Version 3, Reihenfolge der Anzeigen und ihrer Anhänge); im Klartext-Export fehlen beide Felder, mit „Externe Anhänge mitspeichern“ steht `dateiname`, der Pfad der Kopie im Ordner `anhaenge/` der ZIP-Datei (`<erste 8 Zeichen der Anzeige-ID>-<id>-<name>`, nur Buchstaben, Ziffern und Bindestrich aus der Kennung, keine Pfadzeichen und keine Punktfolgen). Fehlt die Liste (bis Version 18), hat die Anzeige keine Anhänge. Ein Anhang ohne `teil` und ohne `dateiname` bleibt beim Einspielen mit seinen Angaben, aber ohne Inhalt; ein gleicher Anhang auf dem Gerät bleibt mit Inhalt (PB 2026-10-24 EFG E4). `id` ist nur innerhalb der Datei eindeutig: Beim Einspielen vergibt WATT den Anhängen eigene Kennungen, behaltene Anhänge des Geräts behalten ihre (PB 2026-10-24 NP N1); Notizen behalten die Kennungen der Datei. `groesse` liegt zwischen 1 und 10 MB, `art` ist bekannt, `teil` nicht negativ, sonst ist die Datei ungültig (PB 2026-10-24 EFG E6) |
| `gespraeche` | Liste | Gesprächsnotizen (F-77) |
| `gespraeche[].id` | Text | UUID |
| `gespraeche[].art` | `TELEFONAT`, `PERSOENLICH`, `VIDEOKONFERENZ` | Art |
| `gespraeche[].zeitpunkt`, `dauerMinuten` | Zeitpunkt, Zahl oder `null` | Gespräch |
| `gespraeche[].partner`, `anwesende`, `anlass`, `inhalt`, `zusagen` | Text | Inhalt |
| `gespraeche[].frist` | Tag oder `null` | Frist aus den Zusagen |
| `gespraeche[].erfasstAm`, `pruefsumme` | Zeitpunkt, Text | Erfassung und SHA-256 der Notiz |
| `gespraeche[].wiedervorlageAm`, `wiedervorlageErledigt`, `wiederhergestelltAm`, `backupVom` | wie bei Anzeigen | |
| `gespraeche[].nachtraege` | Liste aus `id` (`<ID der Notiz>:<Kennung>`), `text`, `erfasstAm`, `pruefsumme` | Nachträge (F-78) |
| `kette` | Liste | Nachweiskette (F-64), alle Glieder in Reihenfolge |
| `kette[].seq` | Zahl | laufende Nummer ab 1 |
| `kette[].zeitpunkt` | Zeitpunkt | Zeitpunkt des Eintrags |
| `kette[].ereignis` | `ANZEIGE_GESENDET`, `ANZEIGE_GELOESCHT`, `GESPRAECHSNOTIZ_GESPEICHERT`, `NACHTRAG`, `GESPRAECHSNOTIZ_GELOESCHT`, `WIEDERHERSTELLUNG`, `WIEDERHERSTELLUNG_UNTERBROCHEN` | Ereignis |
| `kette[].referenzId` | Text | ID des Eintrags; bei Wiederherstellungen der Zeitpunkt des Backups, bei unterbrochener Kette in der Datei `<Zeitpunkt>#<Nummer des Glieds>` |
| `kette[].pruefsumme` | Text | Prüfsumme des Inhalts; bei Wiederherstellungen der SHA-256 der eingespielten Datei (verschlüsselt oder Klartext, wie gewählt) |
| `kette[].prev`, `hash` | Text | wie beim Änderungsprotokoll; kanonische Form: `seq`, `zeitpunkt`, `ereignis`, `referenzId`, `pruefsumme`, `prev` je als `<Länge in UTF-8-Bytes>:<Wert>`, `hash` SHA-256 darüber, erstes Glied mit `prev` `0` |

Beim Einlesen prüft WATT die Nachweiskette gegen die Inhalte der Datei (Verkettung und Nachrechnen der Prüfsummen) und zeigt das Ergebnis vor dem Ersetzen an. Die Wiederherstellung ersetzt die „Grunddaten für Überlastungsanzeigen“, Anzeigen, Gesprächsnotizen und die Kette, setzt an jeder Anzeige und Gesprächsnotiz den Vermerk der Wiederherstellung und hängt an die Kette ein Glied `WIEDERHERSTELLUNG` an (bei unterbrochener Kette `WIEDERHERSTELLUNG_UNTERBROCHEN` mit dem betroffenen Glied in der Referenz). Eine unterbrochene Kette wird trotzdem eingespielt; der Befund bleibt dauerhaft in der Kette und erscheint im Prüfvermerk jedes Aktenauszugs. Fehlt `nachweise` (Versionen 1 bis 3), bleiben die Nachweise des Geräts unverändert.

## Buchungsdaten

Seit 18.10.2026 (F-234, F-235) gibt es neben dem vollständigen Klartext eine Datei mit `umfang` `BUCHUNGEN`, z. B. um Buchungen eines zweiten Geräts zu übernehmen. Sie entsteht im Export-Dialog des Überblicks als JSON oder XML für einen Zeitraum und mit dem Kategorienfilter. Gefüllt sind nur, jeweils im gewählten Zeitraum:

- `eintraege` (nach dem Kategorienfilter), `abwesenheiten` (außer der Filter schließt sie aus), `sonderTage` und `ferien` (Abschnitte auf den Zeitraum begrenzt);
- `kategorien` nur mit den Ober- und Unterkategorien, die die exportierten Einträge benutzen, jeweils mit dem angezeigten Namen in `name`, auch bei vorgepflegten Kategorien ohne eigenen Namen.

Alle übrigen Listen sind leer, `profil`, `stundenplan`, `einstellungen`, `laufenderTimer`, `nachweise` und `kalenderlink` sind `null`. Die Datei enthält kein Feld zu Zeitraum oder Filter.

Einlesen (JSON, unter „Mehr → Daten, Sicherheit & Export → Sicherung → Sicherung einspielen“), ergänzend und alles oder nichts in einer Transaktion:

- Übernommen werden nur Einträge und Abwesenheiten, mit neuer Kennung; Ferien und Tagesmarkierungen der Datei nicht.
- Kategorien werden ausschließlich über den angezeigten Namen von Ober- und Unterkategorie zugeordnet, nie über die Kennung; fehlt eine, wird nichts übernommen.
- Gleiche Einträge (gleicher Beginn, gleiches Ende, gleiche Ober- und Unterkategorie; Tagesblöcke zusätzlich gleicher Tag und gleiche Dauer) und gleiche Abwesenheiten werden übersprungen und gezählt.
- Überschneidungen (F-07, Arztbesuch gegen Eintrag, eine zweite Abwesenheit am selben Tag) und mehr Zeit, als ein Kalendertag lang ist (F-03), verhindern die Übernahme; die Vorschau nennt Tag, Uhrzeit und Konflikt.
- `erfasstAm` und `geaendertAm` kommen aus der Datei und gelten „laut Datei“; jede übernommene Buchung erhält `importiertAm` und ein Anlage-Glied mit `importDatei`. Seit 20.10.2026 sind Einträge in der Zukunft (mit einer Stunde Spielraum; Abwesenheiten dürfen vorausgeplant sein), `erfasstAm` vor `beginn` bei `TIMER`, `erfasstAm` oder `geaendertAm` nach `exportiertAm`, ein `tag`, der nicht zum `beginn` passt, und ein `exportiertAm` mehr als eine Stunde in der Zukunft Konflikte; Einträge mit `quelle` `PLAN` oder `KALENDERLINK` werden ohne `planRef` übernommen.

Eine Datei mit `BUCHUNGEN` ersetzt nie alle Daten; das vollständige Einlesen lehnt sie ab. Eine vollständige Datei lässt sich nicht als Buchungsdaten einlesen.

## XML-Darstellung

Seit dem 16.10.2026 (F-223) exportiert WATT den Klartext wahlweise als XML (`XmlDatenformat`, ohne Bibliothek), mit derselben Formatversion wie das JSON. Ausgelieferte Fassungen exportieren XML ab Formatversion 18 (erste: WATT 0.8.109 „Cherimoya“); veröffentlicht ist ein XML Schema ab Version 18. Keine ausgelieferte Fassung schreibt XML der Versionen 16 und 17; die XSD der Version 17 in `docs/schnittstelle/` stammt aus der Entwicklung zwischen dem 16. und 17.10.2026 und dient nur `FormatbeispieleTest` (XML-Darstellung von `bergamotte-17`), sie ist nicht veröffentlicht. Eingelesen wird nur JSON. Die Abbildung ist mechanisch und verlustfrei (`SchnittstellenTest` führt das XML in das JSON zurück):

- Das JSON-Objekt der Datei wird zum Wurzelelement `wattDaten` (mit `xmlns:xsi`); jedes Feld eines Objekts wird zu einem gleichnamigen Kindelement in der Reihenfolge des JSON.
- Eine Liste wird zu einem Element mit dem Feldnamen, jedes Listenelement zu einem Kindelement mit dem Namen im Singular (`XmlDatenformat.SINGULAR`): `kategorien` → `kategorie`, `unterkategorien` → `unterkategorie`, `ferien` → `ferienabschnitt`, `sonderTage` → `sonderTag`, `eintraege` → `eintrag`, `abwesenheiten` → `abwesenheit`, `protokoll` → `glied`, `funktionen` → `funktion`, `kalenderZuordnungen` → `kalenderZuordnung`, `kalenderEntscheidungen` → `kalenderEntscheidung`, `beschaeftigung` → `abschnitt`, `regionaleFeiertage` → `feiertag`, `gdbAenderungen` → `gdbAenderung`, `unterrichtVd` → `abschnitt`, `deputatAusgleiche` → `deputatAusgleich`, `anzeigen` → `anzeige`, `gespraeche` → `gespraech`, `kette` → `glied`, `kopien` → `kopie`, `dokumente` → `dokument`, `notizen` → `notiz`, `anhaenge` → `anhang`, `nachtraege` → `nachtrag`, `raster` → `stunde`, `bloecke` → `block`, `zeitraeume` → `zeitraum`, `ausgesetzteTage` → `tag`, `verworfen` → `verwerfung`, `unterkategorienEingeblendet` → `kategorieId`, `schlagworteEingeblendet` → `art`, `kettenschluessel` → `schluessel`, `zertifikate` → `zertifikat`, `kettensignaturen` → `signatur`. Eine leere Liste ist ein leeres Element.
- Das Verzeichnis `antworten` mit freien Schlüsseln wird zu einem Element mit einem Kindelement `antwort` je Paar; der Schlüssel steht im Attribut `schluessel`.
- Text steht als Inhalt des Elements (die fünf XML-Sonderzeichen maskiert), Zahlen und Wahrheitswerte in der Schreibweise des JSON (`22.5`, `true`), ein fehlender Wert (`null`) als leeres Element mit `xsi:nil="true"`. Ein CR steht als `&#13;`, damit er beim Lesen nicht zu LF wird; Zeichen, die XML 1.0 nicht darstellen kann (Steuerzeichen außer Tab, LF und CR wie U+000B aus Word, einzelne Surrogate, U+FFFE, U+FFFF), stehen als U+FFFD, das JSON behält sie (seit 19.10.2026, C-19).

## Schemas und Beispieldateien

Die öffentliche Beschreibung des Formats liegt in `docs/schnittstelle/` (Lizenz CC BY 4.0, F-223): je Formatversion ab 16 ein JSON Schema (Draft 2020-12) `watt-daten-<Version>.schema.json`, je Formatversion ab 17 das XML Schema `watt-daten-<Version>.xsd` (veröffentlicht ab 18, siehe „XML-Darstellung“; jede bleibt dauerhaft liegen, E-544) und `README.md`. Die Schemas beschreiben genau die Felder, die WATT schreibt; Pflichtfelder sind die, ohne die WATT eine Datei ablehnt. `SchnittstellenTest` prüft den JSON- und den XML-Export gegen die Schemas, veröffentlicht sind nur die Formatversionen ausgelieferter AABs und dort genau die Dateien der Spalte „Schemata“ der Tabelle „Versionen“ in der README (E-574); ihre öffentliche Kopie liegt im Repository watt_public im Ordner `schema/` (https://github.com/Aves83/watt_public/tree/main/schema, `scripts/schnittstelle-oeffentlich.ps1`). Dort liegen genau die veröffentlichten Versionen mit ihrer README, gegebenenfalls dazu eine Vorabfassung der nächsten Version, als solche in der Tabelle „Versionen“ gekennzeichnet; eine Vorabfassung kann sich bis zur Auslieferung ändern (`.claude/rules/oeffentliche-dateien.md`). Die App verweist unter „Mehr → Hilfe → Schnittstellen (JSON, XML)“ darauf.

Zu jeder an Testende ausgelieferten AAB (`docs/RELEASES.md`) liegt unter `core/data/src/test/resources/formate/` eine Beispieldatei als Klartext `<name>-<version>.json` und als Sicherung `<name>-<version>.wattbak` (Passphrase `beispiel-<name>`), erzeugt mit dem Code des ausgelieferten Commits; jede Liste darin hat mindestens ein Element (F-224, F-108). `FormatbeispieleTest` liest alle Dateien mit dem aktuellen Code, prüft Version, Elementzahlen, beide Ketten, das JSON gegen das Schema seiner Version und ab Version 17 die XML-Darstellung gegen die XSD ihrer Version. Vorhanden: `apfel-16` (Commit 297ea40), `bergamotte-17` (Commit 32581af) und `cherimoya-18` (Commit c3ff686; mit Arztbesuch, Schulfahrt, Ferien jeder Art, Geburtsmonat, eingelesenen Einträgen und dem Glied `uhr_abweichung`).

## Verschlüsselte Sicherung

Der Schlüssel der Datenbank liegt im Android Keystore und lässt sich nicht exportieren. Die Sicherung hat deshalb einen eigenen Sicherungsschlüssel und lässt sich auf jedem Gerät mit der Passphrase oder mit dem Notfallblatt öffnen. Den Container beschreibt `BackupKrypto`; seine Version ist unabhängig von der Version des JSON-Textes.

### Container Version 3 (seit 24.10.2026)

Wie Version 2, aber mit eigenen verschlüsselten Teilen für die Anhänge an Anzeigen (F-241, PB 2026-10-24 EFG E1): Der JSON-Text nennt je Anhang nur die Nummer seines Teils (`teil`), die Inhalte folgen hinter dem Hauptteil, einer nach dem anderen. WATT schreibt und liest die Datei gestreamt, je Teil einer im Speicher; die Größe der Datei ist nur durch den Speicherplatz begrenzt (je Anhang höchstens 10 MB). Ohne Anhänge ist die Datei bis auf das Längenfeld und die Versionsnummer wie in Version 2.

| Teil | Länge | Inhalt |
| --- | --- | --- |
| Kennung | 8 Byte | `WATTBKUP` in ASCII |
| Version | 1 Byte | 3 |
| Merkmale | 1 Byte | Bit 0: Hauptteil mit Deflate komprimiert (immer gesetzt), Bit 1: Notfallhülle vorhanden, Bit 2: Anhangsteile folgen |
| Erstellt | 8 Byte, Big Endian | Zeitpunkt der Sicherung in Millisekunden seit 1970 (UTC) |
| Passphrase-Hülle | 4 + 16 + 12 + 48 Byte | wie Version 2 |
| Notfallhülle | 16 + 12 + 48 Byte | nur bei gesetztem Bit 1, wie Version 2 |
| IV des Hauptteils | 12 Byte | zufällig |
| Länge des Hauptteils | 4 Byte, Big Endian | Länge des Chiffrats des Hauptteils einschließlich der 16 Byte Prüfsumme |
| Hauptteil | Länge | AES-256-GCM über den mit Deflate komprimierten JSON-Text, mit 16 Byte Prüfsumme am Ende; zusätzliche Daten: der ganze Kopf bis einschließlich der Länge |
| Anhangsteil (je Anhang) | 4 + 4 + 12 + Länge Byte | Nummer (Big Endian, lückenlos ab 0), Länge des Chiffrats einschließlich Prüfsumme, IV (zufällig), AES-256-GCM über den unveränderten Inhalt des Anhangs; zusätzliche Daten: der Kopf ‖ `WATT-Teil` ‖ Nummer ‖ Länge (beide Big Endian) |

- Alle Teile verschlüsselt derselbe Sicherungsschlüssel, jeder mit eigenem IV; Hüllen, Passphrase und Notfallblatt sind unverändert gegenüber Version 2. Die Datei endet genau hinter dem letzten Anhangsteil, ohne Anhangsteile (Bit 2 nicht gesetzt) genau hinter dem Hauptteil; angehängte Bytes machen sie ungültig, und ein Längenfeld, das über das Dateiende zeigt, ergibt eine Fehlermeldung (PB 2026-10-24 NP N10).
- Beim Einlesen prüft WATT jeden Anhangsteil (Prüfsumme, Nummer, Größe nach `groesse` des Anhangs); ein Teil, den kein Anhang nennt, und ein genannter Teil, der fehlt, machen die Datei ungültig (PB 2026-10-24 EFG E6). Anhänge ohne `teil` liegen nicht in der Datei: Ihre Angaben bleiben beim Einspielen erhalten, der Inhalt fehlt („Inhalt nicht in dieser Sicherung enthalten“); liegt derselbe Anhang (Anzeige, Name, Art, Größe, Datum) schon auf dem Gerät, bleibt er mit Inhalt (PB 2026-10-24 EFG E4).
- Die Datei für die Android-Sicherung („Super bequem“) enthält die Anhänge nur, solange sie insgesamt unter 20 MB bleibt (Sicherheitsabstand zu den 25 MB, die Android in die Cloud nimmt); sonst liegt die vollständige Datei unter `files/sicherung-geraet/` nur für die Geräteübertragung, und `files/sicherung/` erhält die Fassung ohne Anhangsteile (Bit 2 nicht gesetzt, Anhänge ohne `teil`).
- Dokumente der Nachweise (PDFs von WATT) stehen weiter als Base64 im JSON-Text.

### Container Version 2 (03.10.2026 bis 23.10.2026, nur noch lesbar)

| Teil | Länge | Inhalt |
| --- | --- | --- |
| Kennung | 8 Byte | `WATTBKUP` in ASCII |
| Version | 1 Byte | 2 |
| Merkmale | 1 Byte | Bit 0: Inhalt mit Deflate komprimiert (immer gesetzt), Bit 1: Notfallhülle vorhanden |
| Erstellt | 8 Byte, Big Endian | Zeitpunkt der Sicherung in Millisekunden seit 1970 (UTC) |
| Passphrase-Hülle | 4 + 16 + 12 + 48 Byte | Iterationen (WATT schreibt 600.000, liest 1.000 bis 10.000.000), Salz, IV, umhüllter Sicherungsschlüssel mit GCM-Prüfsumme |
| Notfallhülle | 16 + 12 + 48 Byte | nur bei gesetztem Bit 1: Salz, IV, umhüllter Sicherungsschlüssel mit GCM-Prüfsumme |
| IV des Inhalts | 12 Byte | zufällig |
| Chiffrat | Rest | AES-256-GCM über den mit Deflate komprimierten JSON-Text, mit 16 Byte Prüfsumme am Ende |

- Sicherungsschlüssel: 256 Bit Zufall. Er verschlüsselt den Inhalt. Auf dem Gerät liegt er nur mit einem Schlüssel aus dem Android Keystore umhüllt (`no_backup/sicherung_schluessel.bin`); diese Hülle steht nie in einer Sicherungsdatei. Jede Änderung von Passphrase oder Notfallblatt erzeugt einen neuen Sicherungsschlüssel (A-2); ältere Dateien öffnet nur das Geheimnis, das zu ihrer Zeit galt.
- Passphrase-Hülle: AES-256-GCM mit einem Schlüssel aus PBKDF2-HMAC-SHA256 (Passphrase in UTF-8, eigenes Salz; WATT schreibt 600.000 Iterationen und liest 1.000 bis 10.000.000; eine schwächere Hülle aus einer eingespielten Datei bildet WATT beim Übernehmen mit der Passphrase neu oder übernimmt sie nicht), zusätzliche Daten `WATT-Huelle-Passphrase`.
- Notfallhülle: AES-256-GCM mit dem Schlüssel HMAC-SHA256(Schlüssel = Wiederherstellungsschlüssel in ASCII, Nachricht = `WATT-Huelle-Notfallblatt` ‖ Salz), zusätzliche Daten `WATT-Huelle-Notfallblatt`. Der Wiederherstellungsschlüssel besteht aus 30 Zeichen Base32 (RFC 4648, `A`–`Z`, `2`–`7`), also 150 Bit Zufall; das Notfallblatt zeigt ihn in sechs Fünfergruppen und als QR-Code. Beim Eingeben zählen Leerzeichen, Bindestriche sowie Groß- und Kleinschreibung nicht; `0`, `1` und `8` gelten als `O`, `I` und `B`.
- Jede der beiden Hüllen öffnet die Datei allein. Der ganze Kopf bis einschließlich IV des Inhalts geht als zusätzliche Daten in die Prüfsumme des Inhalts ein; jede Änderung an der Datei lässt das Öffnen scheitern.
- Komprimiert wird vor dem Verschlüsseln (Deflate, höchste Stufe, `java.util.zip`). Dokumente der Nachweise stehen als Base64 im JSON-Text und kommen byte-identisch zurück. Anhänge an Anzeigen (Version 19 des JSON-Textes) standen in diesem Container als Base64 im Feld `inhalt`; WATT liest das weiterhin (die Größe muss zu `groesse` passen, sonst ist die Datei ungültig) und spielt den Inhalt ein (PB 2026-10-24 NP N4).
- Der Erstellungszeitpunkt ist ohne Geheimnis lesbar, damit WATT „Sicherung vom … gefunden“ anzeigen kann; verändert lässt er die Datei nicht mehr öffnen.
- Passphrase und Wiederherstellungsschlüssel werden nirgends gespeichert.

#### Messwerte (F-107)

Gemessen am 03.10.2026 mit dem Gerätetest `AutoSicherungGeraetTest` (#44): drei Schuljahre, 3.920 Einträge, 20 Anzeigen mit je drei PDFs (PDFs zusammen 3.198.136 Byte).

| Größe | Byte |
| --- | --- |
| JSON-Text roh | 9.041.854 |
| Container 1 ohne Kompression | 9.041.911 |
| Container 2 mit Kompression | 2.839.184 (31 % der Rohgröße) |
| Detailnachweis (PDF) | 86.126, mit Deflate 63.838 |

`PdfDocument` nutzt bereits FlateDecode; Base64 im JSON erhöht die Größe der Dokumente um etwa ein Drittel.

### Container Version 1 (29.09.2026 bis 02.10.2026, nur noch lesbar)

| Teil | Länge | Inhalt |
| --- | --- | --- |
| Kennung | 8 Byte | `WATTBKUP` in ASCII |
| Version | 1 Byte | 1 |
| Iterationen | 4 Byte, Big Endian | Iterationen für PBKDF2 (WATT schreibt 600.000, liest 1.000 bis 10.000.000) |
| Salz | 16 Byte | zufällig |
| IV | 12 Byte | zufällig |
| Chiffrat | Rest | AES-256-GCM über den unkomprimierten JSON-Text, mit 16 Byte Prüfsumme am Ende |

- Schlüssel: PBKDF2-HMAC-SHA256 aus der Passphrase (UTF-8) und dem Salz, 256 Bit; die ersten 41 Byte gehen als zusätzliche Daten in die Prüfsumme ein.
- Version 1 öffnet nur die Passphrase; ein Notfallblatt gibt es dafür nicht.

## CSV-Export

Der CSV-Export im Überblick („Exportieren“, F-234) ist für Tabellenkalkulationen gedacht und nicht zum Wiedereinlesen. Kodierung UTF-8 mit BOM, Trennzeichen Semikolon, Zeilenende CRLF, Felder mit Semikolon, Anführungszeichen oder Zeilenumbruch stehen in Anführungszeichen.

Spalten: Datum, Wochentag, Beginn, Ende, Art („Zeitraum“ oder „Tagesblock“, seit dem 10.10.2026, F-179; die Herkunft „Kalenderlink“ steht in der Spalte „Quelle“, F-212), Minuten, Stunden, Oberkategorie, Unterkategorie, Fach, Klasse (beide seit dem 06.10.2026, F-06, leer ohne Schlagwort), Notiz, Quelle, Erfasst am, Zuletzt geändert, Kennzeichen. Nach den Einträgen folgt je Abwesenheitstag eine Zeile mit „Abwesenheit“ als Oberkategorie und der Art als Unterkategorie. Nur mit „Angaben zu Ermäßigungen und Deputat einbeziehen“ (F-178) folgt danach ein Abschnitt mit Bezeichnung und Wert je Zeile (Geburtszeitraum, GdB-Stufe, Urlaub, Altersteilzeit, Pflichtstunden, Teilzeit, Ermäßigungen, Anrechnungsstunden, Deputat bzw. Unterricht je Woche, Deputatskonto).

## Kalender-Export (ICS)

Seit dem 12.10.2026 (F-216) exportiert der Export-Dialog neben CSV und Excel einen Kalender nach RFC 5545, Dateiname `WATT_<Zeitraum>.ics`, UTF-8 ohne BOM, Zeilenende CRLF, Zeilenfaltung bei 75 Oktetten (ohne ein UTF-8-Zeichen zu teilen), Escaping von ``, `;`, `,` und Zeilenumbruch in Textfeldern. Er ist für Kalender-Apps gedacht und nicht zum Wiedereinlesen in WATT.

- Kopf: `BEGIN:VCALENDAR`, `VERSION:2.0`, `PRODID:-//WATT//WATT <Version>//DE`, `CALSCALE:GREGORIAN`, `X-WR-CALNAME:WATT <Zeitraum>`, `X-WR-CALDESC` mit dem Prüfvermerk (Text wie am Fuß der Dokumente, F-150) und bei aktivem Kategorienfilter dem Vermerk „Gefiltert: nur …“ (F-217).
- Je Eintrag mit Uhrzeit ein `VEVENT`: `UID:<Eintrags-ID>@watt`, `DTSTAMP` (Exportzeitpunkt, UTC), `DTSTART` und `DTEND` in UTC (`Z`), `SUMMARY` „Oberkategorie · Unterkategorie“, `CATEGORIES` Oberkategorie, `DESCRIPTION` mit Fach, Klasse, Notiz, Art (Herkunft) und Nachtrag-Kennzeichnung.
- Tagesblöcke ohne Uhrzeit: ganztägig (`DTSTART;VALUE=DATE`, `DTEND` am Folgetag), `SUMMARY` mit Dauer, z. B. „Unterricht · Regelunterricht (3:30 h)“.
- Abwesenheiten: ganztägig, zusammenhängende Tage als ein Termin (ein Arztbesuch mit Uhrzeit), `UID:<UUID>@watt` mit einer aus „abwesenheit-<erster Tag>“ abgeleiteten UUID (Version 3), die die Art nicht nennt (seit 19.10.2026, C-20), `SUMMARY` „Abwesenheit: <Art>“.
- Der Export verankert den Kopf der Arbeitszeitkette wie jeder Export (F-116). Der Kategorienfilter (F-217) wirkt nur auf die Termine; Soll, Ist und Saldo stehen nicht im Kalender.

`IcsExportTest` liest den Export mit einem eigenen Parser wieder ein (Zeiten über die Zeitumstellung und Mitternacht, Faltung, Escaping, Filter).

## QR-Code des Prüfvermerks

Seit dem 07.10.2026 (F-151) steht auf der letzten Seite jedes Dokuments (Anschreiben, Zusammenfassung, Einzelnachweis, Bericht, Aktenauszug) neben dem Prüfvermerk ein QR-Code mit dem Stand des Arbeitszeitprotokolls, erzeugt mit ZXing Core (Fehlerkorrektur M, UTF-8). Inhalt als eine Zeile, Felder durch Semikolon getrennt, keine persönlichen Daten:

`watt-pruef:1;v=<Version>;n=<Anzahl>;t=<JJJJ-MM-TTTHH:MM>;sha256=<64 Zeichen>` und nur bei erkannten Rücksprüngen und Abweichungen der Geräteuhr zusätzlich `;r=<Anzahl>`, nur nach Wiederherstellungen `;w=<Anzahl>` und `;wa=<Anzahl>` (seit 18.10.2026, F-238), nur bei eingelesenen Buchungen `;i=<Anzahl>` (seit 20.10.2026, F-235), nur bei signiertem Kopf `;s=1` (seit 24.10.2026, F-243).

- `watt-pruef:1`: Schema und Version des Formats; die Prüf-App WATTest (https://aves83.github.io/wattest/) liest den Inhalt.
- `v`: Version von WATT, wie sie der Prüfvermerk nennt (F-132), z. B. `0.8.123`.
- `n`: Anzahl der Einträge des Arbeitszeitprotokolls, zugleich die laufende Nummer des Kopfs.
- `t`: Zeitpunkt des Kopfs (Zeitpunkt des jüngsten Glieds) in Berliner Zeit, Minuten genau, z. B. `2026-10-06T14:32`.
- `sha256`: Prüfsumme des Kopfs der Arbeitszeitkette, 64 Hexadezimalzeichen klein, dieselbe wie im Prüfvermerk.
- `r`: Anzahl der Vermerke zur Geräteuhr, `uhr_ruecksprung` (F-133), `uhr_abweichung` (F-239) und `uhr_verstellt` (F-242); fehlt, wenn es keine gibt.
- `w`: Anzahl der Wiederherstellungen seit Beginn der Kette (F-238); fehlt ohne Wiederherstellung.
- `wa`: Anzahl der Wiederherstellungen ohne übereinstimmende Vorgeschichte (abweichend, nicht abgleichbar oder vor dem Abgleich); fehlt, wenn jede übereinstimmt.
- `i`: Anzahl der aus einer Datei eingelesenen Einträge und Abwesenheiten mit Gutschrift (Anlage-Glieder mit `importDatei`, F-235); fehlt ohne Einlesen.
- `s`: `1`, wenn der festgehaltene Kopf eine Signatur des Schlüssels der Kette trägt (F-243); fehlt sonst. Kein Fingerabdruck des Schlüssels.

Ohne Einträge (leeres Protokoll) gibt es keinen Code. Beispiel: `watt-pruef:1;v=0.8.123;n=1234;t=2026-10-06T14:32;sha256=f85dd10a5ff51917ff6cd40e1263806151e5e6b7ab07191c23d2df2810253ea2;r=3`.

## Nicht enthalten

Einstellungen, die nichts über die Arbeitszeit aussagen und nur für das Gerät gelten: Stand des Einrichtungsassistenten, ausgeblendete Hinweise, Lastanzeige des App-Symbols, Backup-Erinnerung und der Zeitpunkt des letzten Backups, die Stufe der automatischen Sicherung mit Ordner und Aufbewahrung, Datum des Notfallblatts und der letzten Passphrase-Prüfung (F-105, F-106). Die mit dem Keystore umhüllte Fassung des Sicherungsschlüssels bleibt immer auf dem Gerät. Ebenso die Einwilligung in den automatischen Abruf der Ferien mit Stichtag und Stand der KMK-Datei sowie die Erinnerungen an fehlende Ferien (F-95, F-99): Eine Einwilligung gilt nur für das Gerät, auf dem sie erteilt wurde. Ebenfalls nur auf dem Gerät: die tägliche Erinnerung mit Wochenrückblick und Ferienhinweisen (F-130, F-166, F-167), die Widget-Belegungen (F-14a), „Details anzeigen“ (F-177), die Sperrzeit (F-114), „Bildschirmfotos erlauben“ (F-210) sowie Einwilligung und Stand des Kalenderlink-Abrufs (der Link selbst steht in der Sicherung, F-211); allgemein alle Bedienzustände und technischen Werte, die `UnverschluesselteWerteTest` und `PersoenlicheWerte` führen und die oben nicht beschrieben sind. Der Aktenauszug (PDF mit ZIP) bleibt der Weg, die Nachweise ohne WATT lesbar weiterzugeben. Nur auf dem Gerät außerdem der Uhrstand (Wanduhr, Monotonuhr, Neustartzähler und Summen des letzten Glieds, Tabelle `uhrstand`, F-242), der Vorratsschlüssel einer leeren Kette, der Stand der letzten Signaturprüfung und der private Schlüssel der Kette im Android Keystore (F-243).
