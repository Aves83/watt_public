Stand: 2026-10-25 (YYYY-MM-DD)

# Technische Erläuterung: Nachvollziehbarkeit und Fälschungssicherheit in WATT

WATT-Version: Vorabfassung · Stand: 25.10.2026

## 1. Zweck

Diese Erläuterung richtet sich an Gutachter:innen, Rechtsbeistände und andere Fachleute, die Aufzeichnungen aus WATT bewerten. Sie beschreibt, mit welchen technischen Mitteln WATT nachträgliche Änderungen erkennbar macht, wie sich das prüfen lässt und wo die Fälschungssicherheit endet.

## 2. Kurzfassung

WATT stellt sicher:

- Nachträgliche Änderungen, Löschungen und Einfügungen innerhalb des Änderungsprotokolls sind erkennbar.
- Löschungen bleiben mit Inhalt nachvollziehbar, außer im eng begrenzten Fall des sofortigen Rückgängigmachens oder nach einem ausdrücklichen Neubeginn des Protokolls, der selbst sichtbar ist.
- Nachträge (Erfassung mehr als 24 Stunden nach der Tätigkeit) sind gekennzeichnet.
- Verstellungen der Geräteuhr um mehr als 2 Minuten in beide Richtungen sind im Protokoll vermerkt (uhr_verstellt, seit 24.10.2026): WATT vergleicht die Wanduhr mit der Monotonuhr des Geräts, auch im Hintergrund, wenn Android eine Änderung der Uhrzeit meldet. Jeder Eintrag nennt außerdem die Neustarts des Geräts seit dem vorigen und ob die Uhrzeit automatisch eingestellt war (4.7). Ohne diesen Bezug, etwa nach einem Neustart, gilt weiter: Rücksprünge um mehr als 2 Minuten zwischen zwei Einträgen sind vermerkt (uhr_ruecksprung).
- Die Arbeitszeitkette ist seit dem 24.10.2026 mit einem Schlüssel signiert, den Android in geschützter Hardware des Geräts für WATT erzeugt und bestätigt (Key Attestation, 4.7). Eine gültige Signatur belegt, dass ein Stand zu diesem Zeitpunkt im echten WATT auf einem Gerät mit diesem Schlüssel lag.
- Ein in einem versandten Dokument festgehaltener Protokollstand (Prüfvermerk) lässt sich später gegen das Protokoll prüfen. Das ist der Kern des Beweiswerts: Seit einem herausgegebenen Prüfvermerk wurde nichts bis zu diesem Stand unbemerkt geändert.
- Eingespielte Sicherungen und Klartexte passen zu ihrem eigenen Änderungsprotokoll (7.); jede Wiederherstellung steht mit dem Abgleich ihrer Vorgeschichte im Protokoll und in jedem Prüfvermerk.

WATT stellt nicht sicher:

- dass die erfassten Zeiten der Wirklichkeit entsprechen; es handelt sich um eine Selbstaufzeichnung
- dass niemand eine vollständig neue, in sich stimmige Aufzeichnung erzeugt hat, sofern kein früherer Anker vorliegt: Das Verfahren ist offen beschrieben, eine neue Kette lässt sich mit einfachen Werkzeugen berechnen
- dass die Uhr des Geräts richtig ging
- eine Urheberschaft im Sinne einer elektronischen Signatur: Die Signatur der Kette (4.7) belegt die Herkunft aus dem echten WATT auf einem bestätigten Gerät, nicht die Person, die es bediente (10.)

## 3. Datenhaltung und Verschlüsselung

- Alle Daten liegen ausschließlich auf dem Gerät, in einer SQLite-Datenbank im geschützten Speicherbereich der App. Es gibt keinen Server des Anbieters.
- Die Datenbank ist mit SQLCipher verschlüsselt (AES-256). Der Datenbankschlüssel ist zufällig erzeugt und mit einem Schlüssel aus dem Android-Schlüsselspeicher (Keystore) umhüllt. Dieser Hüllschlüssel kann das Gerät nicht verlassen.
- Der Hüllschlüssel ist nicht an eine Anmeldung gebunden, damit Hintergrundfunktionen (Timer, Stundenplan-Übernahme, Sicherung) arbeiten können. Die App-Sperre schützt die Oberfläche, nicht kryptografisch die Daten.
- Die Verschlüsselung schützt vor dem Auslesen der Datenbankdatei außerhalb der App. Sie ist kein Mittel gegen Manipulation durch eine Person, die die App auf dem Gerät bedienen oder das Gerät technisch kontrollieren kann.
- Der Schlüssel zum Signieren der Arbeitszeitkette (4.7) liegt ebenfalls im Android-Schlüsselspeicher, ist nicht exportierbar und nicht an eine Anmeldung gebunden, damit auch die Hintergrundarbeit signieren kann.
- Sicherungsdateien verschlüsselt ein eigener zufälliger Sicherungsschlüssel (256 Bit). Auf dem Gerät liegt er nur mit einem eigenen Keystore-Schlüssel umhüllt, in jeder Datei zweifach umhüllt: mit der Passphrase (PBKDF2-HMAC-SHA256, 600.000 Iterationen) und, wenn vorhanden, mit dem Wiederherstellungsschlüssel des Notfallblatts. Jede Änderung von Passphrase oder Notfallblatt verlangt vorher die bisherige Passphrase oder die Gerätesperre und erzeugt einen neuen Sicherungsschlüssel; neue Dateien öffnen danach nur die neuen Geheimnisse. Ältere Dateien bleiben mit dem alten Geheimnis lesbar: Wer ein altes Notfallblatt oder eine alte Passphrase und eine ältere Datei besitzt, kann deren Inhalt lesen, aber keine spätere Datei.

## 4. Das Änderungsprotokoll (Arbeitszeitkette)

### 4.1 Aufbau

Das Änderungsprotokoll schützt das Ist, also das, was nur die Lehrkraft verbindlich kennt: Jede Änderung an Arbeitszeiten und an Abwesenheiten mit Gutschrift erzeugt einen Protokolleintrag. Protokolliert werden Arbeitszeit-Einträge jeder Herkunft (Timer, von Hand, Stundenplan, Kalenderlink, Tagesblock) mit allen Änderungen und Löschungen (eintrag) und Abwesenheiten mit Gutschrift, also Krankheit, Sonderurlaub, Dienstbefreiung und Arztbesuch (abwesenheit; beim stundenweisen Arztbesuch zusätzlich mit Beginn und Ende in UTC), dazu Vorgänge am Protokoll selbst: eine Wiederherstellung aus einer Sicherung (wiederherstellung), der Neubeginn des Protokolls (neubeginn), ein erkannter Rücksprung der Geräteuhr (uhr_ruecksprung), eine beim Timer erkannte Abweichung der Wanduhr von der Monotonuhr des Geräts bzw. ein Neustart des Geräts während des Laufs (uhr_abweichung, seit 20.10.2026), eine erkannte Verstellung der Geräteuhr (uhr_verstellt) und der Schlüssel zum Signieren der Kette (schluessel), beide seit 24.10.2026 (4.7). Einstellungen stehen nicht im Protokoll: Beschäftigungsumfang mit Pflichtstunden und Teilzeit, Geburtszeitraum, GdB, Deputatskonto mit Startwert und Ausgleichen, Unterricht je Woche im Vorbereitungsdienst, Land, Schulart, Amt, Funktionen, Feiertagsoptionen, Ferien, Stundenplan, Kategorien, Tagesmarkierungen, dienstfreie Tage sowie die Zuordnung der Fächer und die Entscheidungen zu Vorschlägen des Kalenderlinks sind Einstellungen und Grundlagen des Solls, nicht das Ist; der Dienstherr kennt sie aus eigenen Akten, und sie wirken nur auf künftige Übernahmen; Berichte und Überlastungsanzeigen legen offen, mit welchen Werten gerechnet wurde, und die Nachweiskette (5.) bewahrt die Zahlen jeder gesendeten Anzeige. Ebenfalls nicht im Protokoll stehen der Vermerk „Plausibilität bestätigt“ eines Eintrags, mit dem die Lehrkraft eine Rückfrage zu ungewöhnlichen Werten bestätigt hat, und der Verweis „Ähnlich wie …“ einer eigenen Kategorie: Beide beschreiben nicht das Ist; der Vermerk ändert weder den Änderungszeitpunkt noch die Kette. Protokolle bis zum 08.10.2026 können außerdem Einträge der Arten profil, funktion, ferien, sondertag, kategorie, unterkategorie und stundenplan enthalten; sie werden wie alle anderen geprüft. Ein Protokolleintrag hat diese Felder:

- seq: laufende Nummer, beginnt bei 1 und steigt ohne Lücke um 1
- zeitpunkt: Zeitpunkt des Eintrags nach der Uhr des Geräts, in UTC, auf Millisekunden gekürzt
- aktion: ANGELEGT, GEAENDERT oder GELOESCHT
- objektArt: Art des Datensatzes, einer der Schlüssel oben
- objektId: Kennung des Datensatzes, z. B. die eines Eintrags oder das Datum einer Abwesenheit; beim Neubeginn leer
- daten: Stand des Datensatzes als JSON-Text, bei Löschungen der letzte Stand und gegebenenfalls die Begründung, bei Änderungen gegebenenfalls der Kommentar zur Änderung (Feld begruendung; beim Arbeitszeit-Eintrag steht das Feld immer, ohne Kommentar als null, bei Abwesenheiten nur mit Kommentar; seit 23.10.2026 auch der automatische Kommentar, wenn WATT den Vorgänger eines vorverlegten Timer-Beginns anpasst; entstehen bei einer Änderung mit Kommentar weitere Teilstücke eines Eintrags oder neue Tage einer Abwesenheit, tragen auch deren Anlage-Einträge den Kommentar); beim Arbeitszeit-Eintrag mit Tag, Kennung der Kategorie und Unterkategorie, Beginn, Ende, Dauer, Notiz, Quelle (MANUELL, TIMER, PLAN für den Stundenplan oder KALENDERLINK für einen Termin aus dem Kalenderlink; bei PLAN und KALENDERLINK dazu die Referenz planRef auf die Stunde bzw. den Termin), Erfassungs- und Änderungszeitpunkt; trägt der Eintrag die Schlagworte Fach oder Klasse, stehen sie als fach bzw. klasse dahinter, sonst fehlen diese Felder; wandelt die Lehrkraft einen Eintrag vom Timer, aus dem Stundenplan oder aus dem Kalenderlink in einen Tagesblock um, gilt er danach als von Hand erfasst (Quelle MANUELL, ohne Bezug zum Stundenplan oder Kalender), und das Glied dieser Änderung nennt die frühere Quelle als umgewandeltAus (TIMER, PLAN oder KALENDERLINK), sonst fehlt das Feld; am Ende stehen Kategorie und Unterkategorie im Klartext zum Zeitpunkt der Buchung (kategorie, unterkategorie, ohne Unterkategorie null), damit jeder Eintrag auch ohne die Einstellungen eindeutig beschrieben bleibt und eine spätere Umbenennung kein geprüftes Glied verändert; ist der Eintrag aus einer Datei mit Buchungsdaten eingelesen (7.1), steht ganz am Ende seines Anlage-Eintrags importDatei mit der SHA-256 dieser Datei, sonst fehlt das Feld
- prev: Prüfsumme des vorherigen Eintrags
- hash: Prüfsumme dieses Eintrags nach 4.2
- neustarts und autoZeit: Uhrstand bei Einträgen seit dem 24.10.2026 (4.7); ältere Einträge haben die Felder nicht

Rücksprung der Geräteuhr: Liegt der Zeitpunkt eines neuen Eintrags mehr als 2 Minuten vor dem des vorherigen, legt WATT den Eintrag trotzdem an und hängt unmittelbar danach einen Eintrag uhr_ruecksprung an: aktion ANGELEGT, objektId die laufende Nummer (seq) des auslösenden Eintrags, zeitpunkt wie dieser, daten {"vorher":"…","nachher":"…"} mit dem Zeitpunkt des vorherigen und des neuen Eintrags in UTC. Verglichen wird in UTC; die Umstellung zwischen Sommer- und Winterzeit löst deshalb nichts aus. Der Vermerk ist ein gewöhnliches Glied der Kette und wird wie jedes andere geprüft. Seit dem 20.10.2026 prüft WATT die Uhr auch beim Start eines Timers: Steht sie mehr als 2 Minuten vor dem letzten Eintrag, folgt der Vermerk uhr_ruecksprung sofort, mit objektId timer und zeitpunkt der Uhr beim Start. Ebenso beim Festhalten eines Nachweises (5.): Steht die Uhr mehr als 2 Minuten vor dem letzten Eintrag der Arbeitszeitkette oder dem letzten Glied der Nachweiskette, folgt in der Arbeitszeitkette der Vermerk uhr_ruecksprung mit objektId nachweis; die Nachweiskette selbst bleibt unverändert. Seit dem 24.10.2026 entsteht uhr_ruecksprung nur noch, wenn kein gültiger Uhrstand vorliegt (nach einem Neustart, nach einer Wiederherstellung, beim ersten Eintrag nach dem Update); sonst erkennt WATT jede Verstellung als uhr_verstellt (4.7).

Abweichung der Geräteuhr beim Timer (seit 20.10.2026): Beim Start eines Timers merkt sich WATT neben der Wanduhr den Stand der Monotonuhr des Geräts (Millisekunden seit dem Start des Geräts, die ein Verstellen der Uhrzeit nicht verändert) und eine Kennung des Boots; beide Werte stehen nur in der Datenbank des Geräts, nie in Sicherung oder Export. Beim Stopp, bei der Höchstdauer, beim App-Start und wenn Android eine Änderung der Uhrzeit meldet, rechnet WATT den Beginn aus der Monotonuhr nach: Stopp minus verstrichene Zeit der Monotonuhr. Weicht er mehr als 2 Minuten vom gespeicherten Beginn ab, übernimmt WATT den Beginn nach der Monotonuhr und hängt vor dem Eintrag des Laufs einen Eintrag uhr_abweichung an: aktion ANGELEGT, objektId timer, daten {"art":"korrektur","beginnUhr":"…","beginnMonoton":"…","zeitpunkt":"…","abweichungSekunden":n} mit dem Beginn nach der Wanduhr, dem Beginn nach der Monotonuhr, dem Zeitpunkt der Prüfung (alle in UTC) und der Abweichung in Sekunden (positiv, wenn die Wanduhr beim Start nachging). Wurde das Gerät während des Laufs neu gestartet (andere Boot-Kennung), zählt die Monotonuhr von neuem; dann bleibt der Beginn nach der Wanduhr, und WATT vermerkt nur {"art":"neustart","beginnUhr":"…","beginnMonoton":null,"zeitpunkt":"…","abweichungSekunden":null}. Abweichungen bis 2 Minuten bleiben unvermerkt. Die Prüfvermerke zählen uhr_abweichung zusammen mit uhr_ruecksprung und uhr_verstellt. Seit dem 24.10.2026 entsteht art neustart nicht mehr (der Neustart steht in neustarts des nächsten Eintrags), und art korrektur nur, wenn der Timer abweicht, ohne dass eine Verstellung erkannt wurde; eine Verstellung während des Laufs korrigiert den Timer im Eintrag uhr_verstellt (4.7).

Wiederherstellung: Der Eintrag wiederherstellung (aktion ANGELEGT) nennt in daten Zeitpunkt des Exports, ob die Datei verschlüsselt war (verschluesselt: true für eine Sicherung, false für einen Klartext), Anzahl und letzte Prüfsumme des vorher vorhandenen Protokolls und seit dem 18.10.2026 vorgeschichte: UEBEREINSTIMMEND (eine Kette ist Glied für Glied der Anfang der anderen), ABWEICHEND (die Ketten gehen auseinander) oder NICHT_ABGLEICHBAR (auf dem Gerät gab es keine Kette). Ältere Einträge ohne vorgeschichte gelten als nicht abgeglichen. Seit dem 24.10.2026 nennt er den Befund der mitgelieferten Signaturen (signaturen: GEPRUEFT, UNGUELTIG mit signaturenUngueltigAb als erstem Eintrag mit ungültiger Signatur, oder KEINE, 4.7).

Eingelesene Buchungen: Liest die Lehrkraft Buchungsdaten eines anderen Geräts ein (7.1), erhält jeder übernommene Arbeitszeit-Eintrag einen Eintrag eintrag mit aktion ANGELEGT, zeitpunkt des Einlesens, einer neuen objektId und den daten wie oben, am Ende mit importDatei (SHA-256 der eingelesenen Datei, hexadezimal); eine übernommene Abwesenheit mit Gutschrift erhält einen Eintrag abwesenheit, ebenfalls mit importDatei. Erfassungs- und Änderungszeitpunkt in daten stammen aus der Datei, damit die Nachtrag-Kennzeichnung stimmt; der Eintrag selbst trägt zusätzlich „Importiert am …“. Diese Einträge sind gewöhnliche Glieder der Kette; die Kette des anderen Geräts wird nicht übernommen.

### 4.2 Prüfsumme: Berechnungsvorschrift

Die Prüfsumme jedes Protokolleintrags lässt sich mit Standardwerkzeugen nachrechnen, z. B. sha256sum (Linux, macOS), Get-FileHash bzw. .NET-Klassen in der Windows-PowerShell, Python hashlib oder Online-Rechnern. Tabellenkalkulationen wie Excel, LibreOffice oder Google Tabellen haben keine eingebaute SHA-256-Funktion und brauchen dafür ein Makro oder Skript.

- Hash-Verfahren: SHA-256 (FIPS 180-4), Ausgabe als 64 Zeichen Hexadezimal in Kleinbuchstaben
- Geheime Werte: keine; kein Seed, kein Salt
- Verfahrensversion: Die Arbeitszeitkette hat ein Verfahren in zwei Formen, die sich an der Zahl der Felder unterscheiden: Einträge bis zum 24.10.2026 haben sieben Felder, Einträge seitdem zusätzlich den Uhrstand (neustarts, autoZeit, 4.7). Ältere Einträge werden unverändert nach der Form mit sieben Feldern geprüft; ein Eintrag mit nur einem der beiden Felder des Uhrstands gilt als Bruch.
- Felder in dieser Reihenfolge: seq, zeitpunkt, aktion, objektArt, objektId, daten, prev, bei Einträgen mit Uhrstand dahinter neustarts (Dezimalzahl, -1 ohne Bezug) und autoZeit (true oder false). Jedes Feld steht als \<Länge>:\<Wert>; die Länge ist die Zahl der Bytes des Werts in UTF-8 als Dezimalzahl, danach folgen ein Doppelpunkt und der Wert in UTF-8. Die Felder folgen ohne Trennzeichen aufeinander. Die Längenangaben machen die Feldgrenzen eindeutig; Werte werden nicht maskiert. Umlaute zählen zwei Bytes, „Überlastungsanzeige“ ist 20 Bytes lang. Ein leerer Wert steht als 0:.
- Darstellung der Werte: Zeitpunkte nach ISO 8601 in UTC mit Z, immer mit Sekunden, Sekundenbruchteile nur, wenn sie nicht null sind, dann mit drei Ziffern (z. B. 2026-09-28T16:05:00Z oder 2026-09-28T16:05:00.123Z). Zahlen dezimal ohne führende Nullen und ohne Tausendertrennzeichen. Aktion als Name (ANGELEGT, GEAENDERT, GELOESCHT). daten ist der gespeicherte JSON-Text ohne Leerzeichen zwischen den Elementen; darin stehen Wahrheitswerte als true und false, fehlende Werte als null, Texte in Anführungszeichen, maskiert nach der Regel in 4.6 (Abschnitt Maskierung). Das Ergebnis hängt nicht von Sprache, Zeitzone oder Plattform des Geräts ab.
- Vorgänger: prev ist die Prüfsumme (hash) des vorherigen Eintrags als 64 Zeichen Hex; beim ersten Eintrag einer Kette ist prev der Text 0.

Formel: hash(n) = SHA-256( L(seq) ‖ L(zeitpunkt) ‖ L(aktion) ‖ L(objektArt) ‖ L(objektId) ‖ L(daten) ‖ L(prev) ) mit L(x) = Länge von x in UTF-8-Bytes, ein Doppelpunkt und x; ‖ heißt aneinanderhängen ohne Trennzeichen; prev = hash(n − 1), beim ersten Eintrag prev = 0. Bei Einträgen mit Uhrstand (4.7): hash(n) = SHA-256( … ‖ L(prev) ‖ L(neustarts) ‖ L(autoZeit) ).

Rechenbeispiel: Eintrag 1 legt am 28.09.2026 um 16:05 UTC eine Korrektur von 14:00 bis 15:30 UTC an (seq 1, aktion ANGELEGT, objektArt eintrag, objektId e1, prev 0). Eintrag 2 ändert um 16:20 UTC das Ende auf 16:00 UTC (seq 2, aktion GEAENDERT, prev ist die Prüfsumme von Eintrag 1). Die Eingabe ist die aufbereitete Bytefolge, die Befehle rechnen sie nach. Die Python-Ausdrücke setzen import hashlib voraus.

- Eingabe (B1):

  ```
  1:120:2026-09-28T16:05:00Z8:ANGELEGT7:eintrag2:e1343:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null}1:0
  ```

- SHA-256 (B1):

  ```
  6b8dfd63e7541957a8503fbc90d380e69f5c3e94532dda5b8f17e10913b6d013
  ```

- bash (B1):

  ```
  printf '%s' '1:120:2026-09-28T16:05:00Z8:ANGELEGT7:eintrag2:e1343:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null}1:0' | sha256sum
  ```

- PowerShell (B1):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes('1:120:2026-09-28T16:05:00Z8:ANGELEGT7:eintrag2:e1343:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null}1:0'))).Replace('-','').ToLower()
  ```

- Python (B1):

  ```
  hashlib.sha256('1:120:2026-09-28T16:05:00Z8:ANGELEGT7:eintrag2:e1343:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null}1:0'.encode('utf-8')).hexdigest()
  ```

- Eingabe (B2):

  ```
  1:220:2026-09-28T16:20:00Z9:GEAENDERT7:eintrag2:e1361:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T16:00:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":"2026-09-28T16:20:00Z","begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null}64:6b8dfd63e7541957a8503fbc90d380e69f5c3e94532dda5b8f17e10913b6d013
  ```

- SHA-256 (B2):

  ```
  759a07d61bb0d53f334cf8f48b3937df7d945405ea55ffbb78f40a6bb99f19da
  ```

- bash (B2):

  ```
  printf '%s' '1:220:2026-09-28T16:20:00Z9:GEAENDERT7:eintrag2:e1361:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T16:00:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":"2026-09-28T16:20:00Z","begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null}64:6b8dfd63e7541957a8503fbc90d380e69f5c3e94532dda5b8f17e10913b6d013' | sha256sum
  ```

- PowerShell (B2):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes('1:220:2026-09-28T16:20:00Z9:GEAENDERT7:eintrag2:e1361:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T16:00:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":"2026-09-28T16:20:00Z","begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null}64:6b8dfd63e7541957a8503fbc90d380e69f5c3e94532dda5b8f17e10913b6d013'))).Replace('-','').ToLower()
  ```

- Python (B2):

  ```
  hashlib.sha256('1:220:2026-09-28T16:20:00Z9:GEAENDERT7:eintrag2:e1361:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T16:00:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":"2026-09-28T16:20:00Z","begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null}64:6b8dfd63e7541957a8503fbc90d380e69f5c3e94532dda5b8f17e10913b6d013'.encode('utf-8')).hexdigest()
  ```

Eintrag 3 legt um 16:40 UTC einen Timer-Lauf der Vor- und Nachbereitung von 16:00 bis 16:40 UTC an (seq 3, aktion ANGELEGT, objektId t1, Quelle TIMER, prev ist die Prüfsumme von Eintrag 2; B10): Die Lehrkraft hat um 16:40 UTC per Timer auf die Korrektur gewechselt. Danach verlegt sie den Beginn des laufenden Korrektur-Timers auf 16:35 UTC vor; Eintrag 4 ändert dazu um 16:45 UTC das Ende des Vorgängers auf 16:35 UTC mit dem automatischen Kommentar zur Änderung (seq 4, aktion GEAENDERT, prev ist die Prüfsumme von Eintrag 3; B9); das Feld begruendung trägt den Text und geht so in die Prüfsumme ein.

- Eingabe (B10):

  ```
  1:320:2026-09-28T16:40:00Z8:ANGELEGT7:eintrag2:t1327:{"tag":"2026-09-28","kategorieId":"vorbereitung","unterkategorieId":null,"beginn":"2026-09-28T16:00:00Z","ende":"2026-09-28T16:40:00Z","dauerMinuten":0,"notiz":"","quelle":"TIMER","planRef":null,"erfasstAm":"2026-09-28T16:40:00Z","geaendertAm":null,"begruendung":null,"kategorie":"Vor- und Nachbereitung","unterkategorie":null}64:759a07d61bb0d53f334cf8f48b3937df7d945405ea55ffbb78f40a6bb99f19da
  ```

- SHA-256 (B10):

  ```
  b2e4bd74aefe61ad9a82dfc13671731557206ada5c9d935718e93ee0989c77a3
  ```

- bash (B10):

  ```
  printf '%s' '1:320:2026-09-28T16:40:00Z8:ANGELEGT7:eintrag2:t1327:{"tag":"2026-09-28","kategorieId":"vorbereitung","unterkategorieId":null,"beginn":"2026-09-28T16:00:00Z","ende":"2026-09-28T16:40:00Z","dauerMinuten":0,"notiz":"","quelle":"TIMER","planRef":null,"erfasstAm":"2026-09-28T16:40:00Z","geaendertAm":null,"begruendung":null,"kategorie":"Vor- und Nachbereitung","unterkategorie":null}64:759a07d61bb0d53f334cf8f48b3937df7d945405ea55ffbb78f40a6bb99f19da' | sha256sum
  ```

- PowerShell (B10):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes('1:320:2026-09-28T16:40:00Z8:ANGELEGT7:eintrag2:t1327:{"tag":"2026-09-28","kategorieId":"vorbereitung","unterkategorieId":null,"beginn":"2026-09-28T16:00:00Z","ende":"2026-09-28T16:40:00Z","dauerMinuten":0,"notiz":"","quelle":"TIMER","planRef":null,"erfasstAm":"2026-09-28T16:40:00Z","geaendertAm":null,"begruendung":null,"kategorie":"Vor- und Nachbereitung","unterkategorie":null}64:759a07d61bb0d53f334cf8f48b3937df7d945405ea55ffbb78f40a6bb99f19da'))).Replace('-','').ToLower()
  ```

- Python (B10):

  ```
  hashlib.sha256('1:320:2026-09-28T16:40:00Z8:ANGELEGT7:eintrag2:t1327:{"tag":"2026-09-28","kategorieId":"vorbereitung","unterkategorieId":null,"beginn":"2026-09-28T16:00:00Z","ende":"2026-09-28T16:40:00Z","dauerMinuten":0,"notiz":"","quelle":"TIMER","planRef":null,"erfasstAm":"2026-09-28T16:40:00Z","geaendertAm":null,"begruendung":null,"kategorie":"Vor- und Nachbereitung","unterkategorie":null}64:759a07d61bb0d53f334cf8f48b3937df7d945405ea55ffbb78f40a6bb99f19da'.encode('utf-8')).hexdigest()
  ```

- Eingabe (B9):

  ```
  1:420:2026-09-28T16:45:00Z9:GEAENDERT7:eintrag2:t1410:{"tag":"2026-09-28","kategorieId":"vorbereitung","unterkategorieId":null,"beginn":"2026-09-28T16:00:00Z","ende":"2026-09-28T16:35:00Z","dauerMinuten":0,"notiz":"","quelle":"TIMER","planRef":null,"erfasstAm":"2026-09-28T16:40:00Z","geaendertAm":"2026-09-28T16:45:00Z","begruendung":"Automatische Anpassung wegen verspätet gestarteter Folgetätigkeit","kategorie":"Vor- und Nachbereitung","unterkategorie":null}64:b2e4bd74aefe61ad9a82dfc13671731557206ada5c9d935718e93ee0989c77a3
  ```

- SHA-256 (B9):

  ```
  88316b05dae509f5a0038826b8325eed8cfd201da60d9070e10e7239e7af7c45
  ```

- bash (B9):

  ```
  printf '%s' '1:420:2026-09-28T16:45:00Z9:GEAENDERT7:eintrag2:t1410:{"tag":"2026-09-28","kategorieId":"vorbereitung","unterkategorieId":null,"beginn":"2026-09-28T16:00:00Z","ende":"2026-09-28T16:35:00Z","dauerMinuten":0,"notiz":"","quelle":"TIMER","planRef":null,"erfasstAm":"2026-09-28T16:40:00Z","geaendertAm":"2026-09-28T16:45:00Z","begruendung":"Automatische Anpassung wegen verspätet gestarteter Folgetätigkeit","kategorie":"Vor- und Nachbereitung","unterkategorie":null}64:b2e4bd74aefe61ad9a82dfc13671731557206ada5c9d935718e93ee0989c77a3' | sha256sum
  ```

- PowerShell (B9):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes('1:420:2026-09-28T16:45:00Z9:GEAENDERT7:eintrag2:t1410:{"tag":"2026-09-28","kategorieId":"vorbereitung","unterkategorieId":null,"beginn":"2026-09-28T16:00:00Z","ende":"2026-09-28T16:35:00Z","dauerMinuten":0,"notiz":"","quelle":"TIMER","planRef":null,"erfasstAm":"2026-09-28T16:40:00Z","geaendertAm":"2026-09-28T16:45:00Z","begruendung":"Automatische Anpassung wegen verspätet gestarteter Folgetätigkeit","kategorie":"Vor- und Nachbereitung","unterkategorie":null}64:b2e4bd74aefe61ad9a82dfc13671731557206ada5c9d935718e93ee0989c77a3'))).Replace('-','').ToLower()
  ```

- Python (B9):

  ```
  hashlib.sha256('1:420:2026-09-28T16:45:00Z9:GEAENDERT7:eintrag2:t1410:{"tag":"2026-09-28","kategorieId":"vorbereitung","unterkategorieId":null,"beginn":"2026-09-28T16:00:00Z","ende":"2026-09-28T16:35:00Z","dauerMinuten":0,"notiz":"","quelle":"TIMER","planRef":null,"erfasstAm":"2026-09-28T16:40:00Z","geaendertAm":"2026-09-28T16:45:00Z","begruendung":"Automatische Anpassung wegen verspätet gestarteter Folgetätigkeit","kategorie":"Vor- und Nachbereitung","unterkategorie":null}64:b2e4bd74aefe61ad9a82dfc13671731557206ada5c9d935718e93ee0989c77a3'.encode('utf-8')).hexdigest()
  ```

### 4.3 Schutz in der Datenbank

Die Protokolltabelle wird nur ergänzt. Zwei Datenbank-Trigger verhindern das Ändern (protokoll_keine_aenderung) und das Löschen (protokoll_keine_loeschung) einzelner Einträge; WATT legt sie bei jedem Öffnen der Datenbank an. Die Sperre gegen Ändern wird nie aufgehoben. Die Sperre gegen Löschen heben nur drei Vorgänge innerhalb einer Transaktion kurz auf und setzen sie in derselben Transaktion wieder: das sofortige Rückgängigmachen (entfernt nur den jüngsten Eintrag), der Neubeginn (entfernt alle Einträge) und die Wiederherstellung aus einer Sicherung (ersetzt das Protokoll durch das der Sicherung). Für die Nachweiskette gilt dasselbe mit eigenen Triggern; dort hebt nur die Wiederherstellung die Löschsperre auf. Diese Sperre wirkt gegen Fehler der App und gewöhnliche Bedienung, nicht gegen jemanden mit vollem technischem Zugriff auf das Gerät (siehe Abschnitt 9).

### 4.4 Löschen

- **Normales Löschen:** Der Datensatz verschwindet aus Ansichten und Berechnungen. Im Protokoll entsteht ein Lösch-Eintrag mit dem vollständigen bisherigen Inhalt und einer optionalen Begründung.
- **Sofortiges Rückgängigmachen:** Nur zulässig, wenn der Anlage-Eintrag noch der jüngste Eintrag der Kette ist und seitdem weder Sicherung, Export, Bericht, Überlastungsanzeige noch Aktenauszug erstellt wurde. Bei einer Abwesenheit über mehrere Tage müssen alle ihre Anlage-Einträge die jüngsten sein; Einträge aus dem Stundenplan sind ausgenommen. Dann werden Datensatz und Anlage-Eintrag entfernt; die Kette endet wieder beim vorherigen Eintrag. Da der entfernte Eintrag nirgends außerhalb festgehalten war, entsteht keine prüfbare Lücke. Die Signatur des entfernten Kopfs entfällt mit ihm, die des vorherigen bleibt (4.7).
- **Neubeginn des Protokolls:** Ausdrückliche Funktion mit doppelter Bestätigung. Alle bisherigen Einträge werden gelöscht, die aktuellen Daten bleiben. Die neue Kette beginnt mit einem Start-Eintrag „Protokoll neu begonnen am …“, dessen Prüfsumme den Datenbestand zu diesem Zeitpunkt abdeckt (siehe 4.6). Berichte und Aktenauszüge weisen den Neubeginn im Prüfvermerk aus. Die Historie davor ist auf dem Gerät nicht mehr vorhanden.

### 4.5 Nachträge

Jeder Arbeitszeit-Eintrag speichert, wann er erfasst (erfasstAm) und zuletzt geändert (geaendertAm) wurde. Die Kennzeichnung folgt dieser Regel:

- Abstand: Erfassungs- bzw. Änderungszeitpunkt minus Ende der Tätigkeit. Ende ist das Ende des Zeitraums, bei Tagesblöcken (Einträge ohne Uhrzeit) das Ende des Tages, also 00:00 Uhr deutscher Zeit (Europe/Berlin) des folgenden Kalendertags.
- Erfassung und letzte Änderung werden getrennt geprüft und getrennt gekennzeichnet, z. B. „Nachgetragen ca. 31 Stunden nach Ende“ und „Geändert ca. 12 Tage nach Ende“.
- Gekennzeichnet wird erst ab einem Abstand von mehr als 24 Stunden; genau 24 Stunden bleiben ohne Kennzeichnung.
- Bis einschließlich 96 Stunden steht der Abstand in ganzen Stunden („ca. x Stunden“), darüber in ganzen Tagen zu je 24 Stunden („ca. x Tage“).
- Rundung: kaufmännisch auf ganze Sekunden des Abstands gerechnet; Sekundenbruchteile entfallen, ab einer halben Stunde bzw. einem halben Tag wird aufgerundet. Stunden = (Sekunden + 1800) ganzzahlig geteilt durch 3600, Tage = (Sekunden + 43200) ganzzahlig geteilt durch 86400. So ergeben 24 Stunden und 1 Sekunde „ca. 24 Stunden“, 96 Stunden und 1 Sekunde „ca. 4 Tage“.
- Die Regel gilt für jede Herkunft, auch für Einträge aus dem Stundenplan oder dem Kalenderlink, die erst später übernommen werden.
- Eingelesene Einträge (7.1): Ihre Erfassungs- und Änderungszeitpunkte stammen aus der Datei; die Kennzeichnung danach trägt den Zusatz „(laut Datei)“. Zusätzlich steht „Importiert am …“ mit dem Zeitpunkt ihres Anlage-Eintrags im Protokoll, und der Abstand zwischen Ende der Tätigkeit und Einlesen wird nach derselben Regel gekennzeichnet („Eingelesen ca. 36 Tage nach Ende“).

Die Kennzeichnung erscheint in Ansichten und vollständigen Exporten; im Detailnachweis einer Überlastungsanzeige ist sie bewusst nicht enthalten.

### 4.6 Prüfsumme des Datenbestands beim Neubeginn

Der Start-Eintrag nach einem Neubeginn hat aktion ANGELEGT, objektArt neubeginn, eine leere objektId und als daten {"bestand":"\<Prüfsumme>", …}. Seit dem 19.10.2026 folgen auf bestand: vorherAnzahl, vorherLetzterHash und vorherZeitpunkt (Anzahl, Prüfsumme und Zeitpunkt des letzten Eintrags der gelöschten Kette; ohne Kette 0, 0 und null), eintraege (je Eintrag Kennung und Bestandsprüfsumme) und abwesenheiten (je Abwesenheit mit Gutschrift Tag und Bestandsprüfsumme), beide nach Kennung bzw. Tag sortiert. So lässt sich ein früherer Anker mit dem Neubeginn verknüpfen, und der Abgleich (7.) prüft auch Einträge, die seit dem Neubeginn unverändert sind. Steht die Geräteuhr mehr als 2 Minuten vor dem letzten Eintrag der alten Kette, lehnt WATT den Neubeginn ab und löscht nichts. Eingabe der Prüfsumme ist der Klartext-Export aller Daten zum Zeitpunkt des Neubeginns im offenen Datenformat watt-daten, ohne Änderungsprotokoll: JSON in UTF-8 ohne Byte Order Mark, je Ebene um vier Leerzeichen eingerückt, Zeilenende LF, ohne Zeilenumbruch am Ende. Die Felder stehen in der Reihenfolge des Datenformats, auch leere; Listen sind sortiert wie im Export (Einträge nach Tag, Beginn, Erfassungszeitpunkt und Kennung, Anzeigen und Gesprächsnotizen nach Kennung). Darin ist exportiertAm 1970-01-01T00:00:00Z, appVersion leer und protokoll die leere Liste []; die Versionsnummer des Datenformats (version) geht mit ein, ebenso Nachweise, Einstellungen und ein laufender Timer.

Formel: bestand = SHA-256(Klartext-Export als JSON ohne Protokoll und ohne Schlüssel und Signaturen der Kette, mit exportiertAm 1970-01-01T00:00:00Z und leerer appVersion). Klartext-Export und Prüfsumme entstehen aus derselben Funktion. Der Kalenderlink fließt nie in die Prüfsumme ein: Der Klartext enthält ihn nie (kalenderlink ist null), nur die verschlüsselte Sicherung fügt ihn ausdrücklich hinzu. Hinweis: Ein Neubeginn zwischen dem 13.10.2026 und dem 18.10.2026 bei gesetztem Kalenderlink hat den Link in die Prüfsumme einbezogen; seine Prüfsumme lässt sich aus dem Klartext-Export nicht nachrechnen.

Nachrechnen lässt sich bestand nur mit einem Klartext-Export als JSON (nicht XML), der unmittelbar nach dem Neubeginn entstand, bevor sich irgendetwas an Daten, Einstellungen oder Nachweisen änderte: darin exportiertAm und appVersion wie oben setzen und die Liste protokoll durch [] ersetzen. Frühere Stände hält WATT nicht vor; jeder spätere Export ergibt eine andere Prüfsumme, ohne dass das auf eine Manipulation hindeutet. Ohne einen solchen Export bleibt bestand für Prüfende ein Wert, den sie nur festhalten und mit einer anderen Quelle desselben Zeitpunkts vergleichen können (etwa einer Sicherung, die die Lehrkraft unmittelbar nach dem Neubeginn angelegt hat). Nachprüfbar bleiben auch ohne ihn: die Bestandsprüfsummen je Eintrag und Abwesenheit (eintraege, abwesenheiten, unten) für jeden Eintrag eines späteren Exports, der seit dem Neubeginn keinen Protokolleintrag hat (so prüft WATT selbst, 7.), und vorherAnzahl, vorherLetzterHash und vorherZeitpunkt gegen Prüfvermerke, Exporte und Sicherungen der gelöschten Kette.

Bestandsprüfsumme eines Eintrags: die ersten 32 Hex-Zeichen (128 Bit) der SHA-256 über die kanonische Form als JSON ohne Leerzeichen mit den Feldern id, tag, kategorieId, unterkategorieId, beginn, ende, dauerMinuten, notiz, quelle, planRef, erfasstAm, geaendertAm, fach und klasse in dieser Reihenfolge; Zeitpunkte auf Millisekunden in UTC (ohne Nachkommastellen, wenn sie null sind), fehlende Werte null, fehlendes Fach und fehlende Klasse als leerer Text. Bei Abwesenheiten die Felder tag, art, beginn und ende. Gekürzt wird, damit der Start-Eintrag auch bei vielen Einträgen klein bleibt. Rechenbeispiel für den Eintrag aus 4.2 in der ersten Fassung (Kennung e1):

- Eingabe (B8):

  ```
  {"id":"e1","tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"fach":"","klasse":""}
  ```

- SHA-256 (B8):

  ```
  11b5450f7c375bc79b62cc52ba607207f06ecc7b3a4986152bb03938f430b5c7
  ```

- Bestandsprüfsumme (B8):

  ```
  11b5450f7c375bc79b62cc52ba607207
  ```

- bash (B8):

  ```
  printf '%s' '{"id":"e1","tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"fach":"","klasse":""}' | sha256sum
  ```

- PowerShell (B8):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes('{"id":"e1","tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"fach":"","klasse":""}'))).Replace('-','').ToLower()
  ```

- Python (B8):

  ```
  hashlib.sha256('{"id":"e1","tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"fach":"","klasse":""}'.encode('utf-8')).hexdigest()
  ```

Maskierung: Dieselbe Regel gilt für die kanonische Form der Bestandsprüfsumme, für den Klartext-Export, über den bestand gerechnet wird, und für daten der Einträge (4.2); WATT schreibt alle drei mit der Bibliothek kotlinx.serialization. Ein Text steht in doppelten Anführungszeichen. Maskiert werden genau das Anführungszeichen, der umgekehrte Schrägstrich und die Steuerzeichen U+0000 bis U+001F; fünf Steuerzeichen haben eine Kurzform, die übrigen stehen als \u mit vier Hexadezimalziffern in Kleinbuchstaben:

```
Zeichen                      maskiert
"  (U+0022)                  \"
\  (U+005C)                  \\
U+0008 Rückschritt           \b
U+0009 Tabulator             \t
U+000A Zeilenvorschub        \n
U+000C Seitenvorschub        \f
U+000D Wagenrücklauf         \r
übrige U+0000 bis U+001F     \u0000 bis \u001f, z. B. U+001B als \u001b
```

Alle anderen Zeichen stehen unverändert in UTF-8, nie als \u-Folge: Umlaute und ß (je zwei Bytes), das Eurozeichen (drei Bytes), Zeichen außerhalb der Basisebene wie Emoji (vier Bytes, kein Ersatzpaar), der Schrägstrich /, U+007F sowie U+2028 und U+2029. Ein einzelnes Ersatzzeichen (ungültiges UTF-16, über die Oberfläche praktisch nicht einzugeben) wird bei der Umwandlung in UTF-8 zu einem Fragezeichen (Byte 0x3F). In Python entsteht dieselbe Form mit json.dumps(…, ensure_ascii=False, separators=(',', ':')); mit ensure_ascii=True (Standard in Python) oder einer Bibliothek, die Nicht-ASCII-Zeichen oder den Schrägstrich maskiert, ergibt sich eine andere Prüfsumme.

Rechenbeispiel für einen Eintrag mit Umlauten, Anführungszeichen und Tabulator (Kennung e3, Notiz Aufsatz "Märchen", ein Tabulator und Größe, Fach Französisch, Klasse 7c). Die Befehle setzen voraus, dass die Konsole die Umlaute als Unicode bzw. UTF-8 übergibt; Python braucht r'…', damit die umgekehrten Schrägstriche erhalten bleiben:

- Eingabe (B13):

  ```
  {"id":"e3","tag":"2026-09-29","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-29T14:00:00Z","ende":"2026-09-29T15:00:00Z","dauerMinuten":0,"notiz":"Aufsatz \"Märchen\"\tGröße","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-29T15:10:00Z","geaendertAm":null,"fach":"Französisch","klasse":"7c"}
  ```

- SHA-256 (B13):

  ```
  be0f6e05b619d12dadb72adddb87c1c0d1c6a6ddf4c32ddb3c0fdd844e43db2e
  ```

- Bestandsprüfsumme (B13):

  ```
  be0f6e05b619d12dadb72adddb87c1c0
  ```

- bash (B13):

  ```
  printf '%s' '{"id":"e3","tag":"2026-09-29","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-29T14:00:00Z","ende":"2026-09-29T15:00:00Z","dauerMinuten":0,"notiz":"Aufsatz \"Märchen\"\tGröße","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-29T15:10:00Z","geaendertAm":null,"fach":"Französisch","klasse":"7c"}' | sha256sum
  ```

- PowerShell (B13):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes('{"id":"e3","tag":"2026-09-29","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-29T14:00:00Z","ende":"2026-09-29T15:00:00Z","dauerMinuten":0,"notiz":"Aufsatz \"Märchen\"\tGröße","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-29T15:10:00Z","geaendertAm":null,"fach":"Französisch","klasse":"7c"}'))).Replace('-','').ToLower()
  ```

- Python (B13):

  ```
  hashlib.sha256(r'{"id":"e3","tag":"2026-09-29","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-29T14:00:00Z","ende":"2026-09-29T15:00:00Z","dauerMinuten":0,"notiz":"Aufsatz \"Märchen\"\tGröße","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-29T15:10:00Z","geaendertAm":null,"fach":"Französisch","klasse":"7c"}'.encode('utf-8')).hexdigest()
  ```

Rechenbeispiel für einen kleinsten Datenbestand (Standardprofil, keine Kategorien, kein Zeitraster, keine Einträge, leere Nachweise); es gilt unverändert, wenn ein Kalenderlink gesetzt ist, denn Eingabe und Prüfsumme bleiben gleich (kalenderlink null). In der Eingabe ist jeder Zeilenumbruch als \n geschrieben; die Befehle setzen ihn wieder ein.

- Eingabe (B3):

  ```
  {\n    "format": "watt-daten",\n    "version": 19,\n    "exportiertAm": "1970-01-01T00:00:00Z",\n    "appVersion": "",\n    "umfang": "VOLLSTAENDIG",\n    "profil": {\n        "beschaeftigung": [\n            {\n                "ab": "2000-01-01",\n                "pflichtstunden": null,\n                "pflichtstundenMinuten": null,\n                "teilzeitStunden": null,\n                "altersteilzeit": false,\n                "pflichtstundenAnderer": false\n            }\n        ],\n        "regionaleFeiertage": [\n            "MARIAE_HIMMELFAHRT"\n        ],\n        "bundesland": "BAYERN",\n        "schulart": "BERUFLICHE_SCHULEN",\n        "unterart": null,\n        "amt": null,\n        "amtsform": null,\n        "amtBezeichnung": "",\n        "funktionenBestaetigt": false,\n        "vorbereitungsdienstBeginn": null,\n        "vorbereitungsdienstEnde": null,\n        "geburtHalbjahr": null,\n        "geburtMonat": null,\n        "geburtTag": null,\n        "gdbAnfang": null,\n        "gdbAenderungen": [],\n        "antworten": {},\n        "unterrichtVd": [],\n        "deputatStartwert": 0,\n        "deputatAusgleiche": []\n    },\n    "kategorien": [],\n    "ferien": [],\n    "sonderTage": [],\n    "stundenplan": {\n        "aktiv": true,\n        "abWochenAktiv": false,\n        "zykluslaenge": 1,\n        "abBezugstag": null,\n        "abFerienwochenUeberspringen": true,\n        "uebernommenBis": null,\n        "raster": [],\n        "eintraege": [],\n        "bloecke": [],\n        "ausgesetzteTage": [],\n        "verworfen": [],\n        "quelle": "HAND"\n    },\n    "eintraege": [],\n    "abwesenheiten": [],\n    "protokoll": [],\n    "laufenderTimer": null,\n    "einstellungen": {\n        "unterkategorienEingeblendet": [],\n        "schlagworteEingeblendet": []\n    },\n    "nachweise": {\n        "meineDaten": {\n            "name": "",\n            "schule": "",\n            "adressat": "",\n            "empfaengerEmail": "",\n            "kopien": [\n                {\n                    "stelle": "PERSONALRAT",\n                    "aktiv": false,\n                    "email": ""\n                },\n                {\n                    "stelle": "SCHWERBEHINDERTENVERTRETUNG",\n                    "aktiv": false,\n                    "email": ""\n                },\n                {\n                    "stelle": "GLEICHSTELLUNGSBEAUFTRAGTE",\n                    "aktiv": false,\n                    "email": ""\n                }\n            ]\n        },\n        "anzeigen": [],\n        "gespraeche": [],\n        "kette": []\n    },\n    "funktionen": [],\n    "kalenderlink": null,\n    "kalenderZuordnungen": [],\n    "kalenderEntscheidungen": []\n}
  ```

- SHA-256 (B3):

  ```
  afa3d7d0361a798add4613b553d328e63c81c007f5e3ebbe72359e48ef2a3c32
  ```

- bash (B3):

  ```
  printf '%b' '{\n    "format": "watt-daten",\n    "version": 19,\n    "exportiertAm": "1970-01-01T00:00:00Z",\n    "appVersion": "",\n    "umfang": "VOLLSTAENDIG",\n    "profil": {\n        "beschaeftigung": [\n            {\n                "ab": "2000-01-01",\n                "pflichtstunden": null,\n                "pflichtstundenMinuten": null,\n                "teilzeitStunden": null,\n                "altersteilzeit": false,\n                "pflichtstundenAnderer": false\n            }\n        ],\n        "regionaleFeiertage": [\n            "MARIAE_HIMMELFAHRT"\n        ],\n        "bundesland": "BAYERN",\n        "schulart": "BERUFLICHE_SCHULEN",\n        "unterart": null,\n        "amt": null,\n        "amtsform": null,\n        "amtBezeichnung": "",\n        "funktionenBestaetigt": false,\n        "vorbereitungsdienstBeginn": null,\n        "vorbereitungsdienstEnde": null,\n        "geburtHalbjahr": null,\n        "geburtMonat": null,\n        "geburtTag": null,\n        "gdbAnfang": null,\n        "gdbAenderungen": [],\n        "antworten": {},\n        "unterrichtVd": [],\n        "deputatStartwert": 0,\n        "deputatAusgleiche": []\n    },\n    "kategorien": [],\n    "ferien": [],\n    "sonderTage": [],\n    "stundenplan": {\n        "aktiv": true,\n        "abWochenAktiv": false,\n        "zykluslaenge": 1,\n        "abBezugstag": null,\n        "abFerienwochenUeberspringen": true,\n        "uebernommenBis": null,\n        "raster": [],\n        "eintraege": [],\n        "bloecke": [],\n        "ausgesetzteTage": [],\n        "verworfen": [],\n        "quelle": "HAND"\n    },\n    "eintraege": [],\n    "abwesenheiten": [],\n    "protokoll": [],\n    "laufenderTimer": null,\n    "einstellungen": {\n        "unterkategorienEingeblendet": [],\n        "schlagworteEingeblendet": []\n    },\n    "nachweise": {\n        "meineDaten": {\n            "name": "",\n            "schule": "",\n            "adressat": "",\n            "empfaengerEmail": "",\n            "kopien": [\n                {\n                    "stelle": "PERSONALRAT",\n                    "aktiv": false,\n                    "email": ""\n                },\n                {\n                    "stelle": "SCHWERBEHINDERTENVERTRETUNG",\n                    "aktiv": false,\n                    "email": ""\n                },\n                {\n                    "stelle": "GLEICHSTELLUNGSBEAUFTRAGTE",\n                    "aktiv": false,\n                    "email": ""\n                }\n            ]\n        },\n        "anzeigen": [],\n        "gespraeche": [],\n        "kette": []\n    },\n    "funktionen": [],\n    "kalenderlink": null,\n    "kalenderZuordnungen": [],\n    "kalenderEntscheidungen": []\n}' | sha256sum
  ```

- PowerShell (B3):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes(('{\n    "format": "watt-daten",\n    "version": 19,\n    "exportiertAm": "1970-01-01T00:00:00Z",\n    "appVersion": "",\n    "umfang": "VOLLSTAENDIG",\n    "profil": {\n        "beschaeftigung": [\n            {\n                "ab": "2000-01-01",\n                "pflichtstunden": null,\n                "pflichtstundenMinuten": null,\n                "teilzeitStunden": null,\n                "altersteilzeit": false,\n                "pflichtstundenAnderer": false\n            }\n        ],\n        "regionaleFeiertage": [\n            "MARIAE_HIMMELFAHRT"\n        ],\n        "bundesland": "BAYERN",\n        "schulart": "BERUFLICHE_SCHULEN",\n        "unterart": null,\n        "amt": null,\n        "amtsform": null,\n        "amtBezeichnung": "",\n        "funktionenBestaetigt": false,\n        "vorbereitungsdienstBeginn": null,\n        "vorbereitungsdienstEnde": null,\n        "geburtHalbjahr": null,\n        "geburtMonat": null,\n        "geburtTag": null,\n        "gdbAnfang": null,\n        "gdbAenderungen": [],\n        "antworten": {},\n        "unterrichtVd": [],\n        "deputatStartwert": 0,\n        "deputatAusgleiche": []\n    },\n    "kategorien": [],\n    "ferien": [],\n    "sonderTage": [],\n    "stundenplan": {\n        "aktiv": true,\n        "abWochenAktiv": false,\n        "zykluslaenge": 1,\n        "abBezugstag": null,\n        "abFerienwochenUeberspringen": true,\n        "uebernommenBis": null,\n        "raster": [],\n        "eintraege": [],\n        "bloecke": [],\n        "ausgesetzteTage": [],\n        "verworfen": [],\n        "quelle": "HAND"\n    },\n    "eintraege": [],\n    "abwesenheiten": [],\n    "protokoll": [],\n    "laufenderTimer": null,\n    "einstellungen": {\n        "unterkategorienEingeblendet": [],\n        "schlagworteEingeblendet": []\n    },\n    "nachweise": {\n        "meineDaten": {\n            "name": "",\n            "schule": "",\n            "adressat": "",\n            "empfaengerEmail": "",\n            "kopien": [\n                {\n                    "stelle": "PERSONALRAT",\n                    "aktiv": false,\n                    "email": ""\n                },\n                {\n                    "stelle": "SCHWERBEHINDERTENVERTRETUNG",\n                    "aktiv": false,\n                    "email": ""\n                },\n                {\n                    "stelle": "GLEICHSTELLUNGSBEAUFTRAGTE",\n                    "aktiv": false,\n                    "email": ""\n                }\n            ]\n        },\n        "anzeigen": [],\n        "gespraeche": [],\n        "kette": []\n    },\n    "funktionen": [],\n    "kalenderlink": null,\n    "kalenderZuordnungen": [],\n    "kalenderEntscheidungen": []\n}' -replace '\\n', "`n")))).Replace('-','').ToLower()
  ```

- Python (B3):

  ```
  hashlib.sha256('{\n    "format": "watt-daten",\n    "version": 19,\n    "exportiertAm": "1970-01-01T00:00:00Z",\n    "appVersion": "",\n    "umfang": "VOLLSTAENDIG",\n    "profil": {\n        "beschaeftigung": [\n            {\n                "ab": "2000-01-01",\n                "pflichtstunden": null,\n                "pflichtstundenMinuten": null,\n                "teilzeitStunden": null,\n                "altersteilzeit": false,\n                "pflichtstundenAnderer": false\n            }\n        ],\n        "regionaleFeiertage": [\n            "MARIAE_HIMMELFAHRT"\n        ],\n        "bundesland": "BAYERN",\n        "schulart": "BERUFLICHE_SCHULEN",\n        "unterart": null,\n        "amt": null,\n        "amtsform": null,\n        "amtBezeichnung": "",\n        "funktionenBestaetigt": false,\n        "vorbereitungsdienstBeginn": null,\n        "vorbereitungsdienstEnde": null,\n        "geburtHalbjahr": null,\n        "geburtMonat": null,\n        "geburtTag": null,\n        "gdbAnfang": null,\n        "gdbAenderungen": [],\n        "antworten": {},\n        "unterrichtVd": [],\n        "deputatStartwert": 0,\n        "deputatAusgleiche": []\n    },\n    "kategorien": [],\n    "ferien": [],\n    "sonderTage": [],\n    "stundenplan": {\n        "aktiv": true,\n        "abWochenAktiv": false,\n        "zykluslaenge": 1,\n        "abBezugstag": null,\n        "abFerienwochenUeberspringen": true,\n        "uebernommenBis": null,\n        "raster": [],\n        "eintraege": [],\n        "bloecke": [],\n        "ausgesetzteTage": [],\n        "verworfen": [],\n        "quelle": "HAND"\n    },\n    "eintraege": [],\n    "abwesenheiten": [],\n    "protokoll": [],\n    "laufenderTimer": null,\n    "einstellungen": {\n        "unterkategorienEingeblendet": [],\n        "schlagworteEingeblendet": []\n    },\n    "nachweise": {\n        "meineDaten": {\n            "name": "",\n            "schule": "",\n            "adressat": "",\n            "empfaengerEmail": "",\n            "kopien": [\n                {\n                    "stelle": "PERSONALRAT",\n                    "aktiv": false,\n                    "email": ""\n                },\n                {\n                    "stelle": "SCHWERBEHINDERTENVERTRETUNG",\n                    "aktiv": false,\n                    "email": ""\n                },\n                {\n                    "stelle": "GLEICHSTELLUNGSBEAUFTRAGTE",\n                    "aktiv": false,\n                    "email": ""\n                }\n            ]\n        },\n        "anzeigen": [],\n        "gespraeche": [],\n        "kette": []\n    },\n    "funktionen": [],\n    "kalenderlink": null,\n    "kalenderZuordnungen": [],\n    "kalenderEntscheidungen": []\n}'.encode('utf-8')).hexdigest()
  ```

### 4.7 Uhrstand, Schlüssel und Signatur (seit 24.10.2026)

Uhrstand je Eintrag: Jeder seit dem 24.10.2026 angelegte Eintrag trägt zwei weitere Felder, die in seine Prüfsumme eingehen (4.2): neustarts, die Anzahl der Neustarts des Geräts seit dem vorigen Eintrag nach dem Zähler BOOT_COUNT von Android (beim ersten Eintrag einer Kette 0; -1, wenn es keinen Bezug gibt, etwa beim ersten Eintrag nach dem Update, nach einer Wiederherstellung oder ohne Zähler), und autoZeit, ob „Datum und Uhrzeit automatisch“ eingeschaltet war. Schreibt ein Vorgang mehrere Einträge, nennt der erste die Neustarts, die folgenden 0. Ältere Einträge haben die Felder nicht und rechnen unverändert mit sieben Feldern.

Verstellung der Geräteuhr (uhr_verstellt): Zum jüngsten Eintrag merkt sich WATT auf dem Gerät Wanduhr, Monotonuhr (Millisekunden seit dem Start des Geräts, die ein Verstellen der Uhrzeit nicht verändert) und Neustartzähler; diese Werte stehen nie in Sicherung, Export oder Prüfsumme. Ohne Neustart seitdem muss die verstrichene Wanduhrzeit der verstrichenen Monotonuhr entsprechen. Weicht sie um mehr als 2 Minuten ab, in eine der beiden Richtungen, entsteht ein Eintrag uhr_verstellt: aktion ANGELEGT, daten {"vorher":"…","nachher":"…","abweichungSekunden":n,"richtung":"vor"} bzw. "zurueck", mit der Zeit, die die Uhr ohne Verstellung zeigen müsste, und der Zeit, die sie zeigte (UTC). abweichungSekunden trägt ein Vorzeichen: Es ist nachher − vorher in ganzen Sekunden, zur Null hin gekürzt, also positiv bei richtung vor und negativ bei richtung zurueck (eine um eine Stunde zurückgestellte Uhr ergibt etwa -3600); der Betrag ist immer mindestens 120. richtung ist genau dann vor, wenn abweichungSekunden positiv ist. Bei summierten kleinen Schritten (schritte, unten) nennt abweichungSekunden die Summe ihrer Beträge mit dem Vorzeichen der Gesamtrichtung (Summe der Schritte mit Vorzeichen); heben sich die Schritte genau auf, gilt vor. objektId nennt den Anlass: glied (vor einem neuen Eintrag, der unmittelbar folgt), zeitsignal (Android meldet eine Änderung der Uhrzeit; geprüft und vermerkt auch im Hintergrund ohne Entsperren), timer, nachweis oder start (Start eines Timers, Festhalten eines Nachweises, Start der App). Zeitzone und Sommerzeit verschieben die Zeitpunkte in UTC nicht und lösen nichts aus. Abweichungen bis 2 Minuten, etwa Korrekturen der Netzzeit, setzen nur den Bezug neu; ist die automatische Uhrzeit aus, summiert WATT ihre Beträge seit dem letzten Eintrag, und liegt die Summe über 2 Minuten, entsteht uhr_verstellt mit der Summe und dem Feld schritte, damit sich die Uhr nicht in vielen kleinen Schritten unbemerkt verstellen lässt. Läuft dabei ein Timer, korrigiert WATT seinen Beginn nach seiner eigenen Monotonuhr im selben Eintrag (timerBeginnVorher, timerBeginnNachher); ein Ereignis ergibt genau einen Eintrag. Nach einem Neustart gibt es keinen Bezug; dann gilt für Rücksprünge weiter uhr_ruecksprung (4.1).

Zeitpunkt des Eintrags uhr_verstellt: Er entsteht erst, wenn WATT die Verstellung erkennt, also nach ihr, und trägt die Zeit der verstellten Uhr in diesem Moment; zeitpunkt gleicht nachher. Bei objektId glied ist das zugleich der Zeitpunkt des unmittelbar folgenden Eintrags, bei zeitsignal liegt er in der Regel wenige Sekunden nach der Verstellung, bei den übrigen Anlässen beim Start des Timers, beim Festhalten des Nachweises bzw. beim Start der App. Alle Einträge davor tragen die Zeit der Uhr vor der Verstellung, alle danach die der verstellten Uhr. Einen Zeitsprung zwischen dem vorigen Eintrag und dem Eintrag uhr_verstellt erklären Prüfende so:

zeitpunkt(uhr_verstellt) − zeitpunkt(voriger Eintrag) = (vorher − zeitpunkt(voriger Eintrag)) + (nachher − vorher).

Der erste Teil ist die verstrichene Zeit nach der Monotonuhr plus die Abweichungen, die WATT seit dem vorigen Eintrag bei Prüfungen ohne Befund unvermerkt in den Bezug übernommen hat (jede höchstens 2 Minuten). Bei ausgeschalteter automatischer Uhrzeit summiert WATT diese Abweichungen (oben); solange ihre Beträge zusammen höchstens 2 Minuten ergeben, bleiben sie unvermerkt, darüber entsteht uhr_verstellt mit schritte. Daraus folgt die Prüfregel: Der erste Teil ist nicht kleiner als −2 Minuten (−120 Sekunden, Grenze eingeschlossen); bei summierten kleinen Schritten (schritte) stehen die Schritte vollständig im zweiten Teil, und der erste Teil ist die verstrichene Zeit nach der Monotonuhr, also nicht negativ. Ausgenommen sind Korrekturen der Uhr zurück bei eingeschalteter automatischer Uhrzeit (Netzzeit): WATT summiert sie nicht, jede beträgt höchstens 2 Minuten, in der Praxis Sekunden. Liegt der erste Teil unter der Grenze, ist das ein Hinweis zur Klärung, kein Beleg einer Manipulation; autoZeit des Eintrags uhr_verstellt zeigt, ob die automatische Uhrzeit bei der Erkennung eingeschaltet war. Der zweite Teil ist die Verstellung: Er gleicht abweichungSekunden (in ganzen Sekunden, zur Null hin gekürzt), bei summierten kleinen Schritten (schritte) der Summe mit Vorzeichen, während abweichungSekunden dann die Summe der Beträge nennt. Ein Rücksprung von mehr als 2 Minuten bis zum Eintrag uhr_verstellt mit richtung zurueck ist damit erklärt; ein uhr_ruecksprung folgt nicht.

Schlüssel: Für jede Kette lässt WATT Android auf dem Gerät einen Schlüssel erzeugen (EC P-256, nur zum Signieren, SHA256withECDSA) im Android-Schlüsselspeicher in geschützter Hardware (TEE, bewusst nicht StrongBox), ohne Bindung an die Anmeldung. Der private Schlüssel verlässt das Gerät nie und steht in keiner Datei. Android bestätigt den Schlüssel mit einer Zertifikatskette (Key Attestation): Das Blattzertifikat enthält den öffentlichen Schlüssel und die Erweiterung 1.3.6.1.4.1.11129.2.1.17 mit Sicherheitsstufe, Challenge, der App (Paketname und SHA-256 ihres Signaturzertifikats), Android-Version, Stand der Sicherheitsupdates und RootOfTrust (verifizierter Start, Bootloader gesperrt); Gerätekennungen fordert WATT nicht an. Die Challenge (attestationChallenge) ist immer genau 32 Bytes lang:

- Kette mit Einträgen: der hash des Kopfs der Kette beim Erzeugen des Schlüssels, als Hexadezimaltext zu 32 Bytes dekodiert (nicht die 64 Zeichen als Text). Das ist der Eintrag unmittelbar vor dem Eintrag schluessel (seq um 1 kleiner). Einzige Ausnahme: Erkennt WATT beim Anhängen des Eintrags schluessel eine Verstellung der Uhr, steht dazwischen ein Eintrag uhr_verstellt mit objektId glied; dann ist die Challenge der hash des Eintrags vor diesem. Nach einem Neubeginn gilt das ebenso; der Eintrag schluessel folgt dem Start-Eintrag neubeginn bzw. späteren Einträgen.
- Leere Kette: WATT erzeugt den Schlüssel im Voraus mit 32 Zufallsbytes als Challenge. Sein Eintrag schluessel ist dann Eintrag 1 der Kette (seq 1, prev 0) mit demselben Zeitpunkt wie der erste eigentliche Eintrag, der ihm folgt, und trägt die Zufallsbytes in daten als challenge, 64 Zeichen Hexadezimal in Kleinbuchstaben. Nur dieser Eintrag schluessel hat das Feld challenge.

Der Eintrag schluessel (aktion ANGELEGT, objektId der Fingerabdruck) trägt in daten status verfuegbar, fingerabdruck (SHA-256 über die SubjectPublicKeyInfo des öffentlichen Schlüssels in DER), zertifikateSha256 (SHA-256 über die Zertifikate in DER hintereinander, Blatt zuerst), beide als 64 Zeichen Hexadezimal in Kleinbuchstaben, sicherheitsstufe (SOFTWARE, TEE oder STRONGBOX) und gegebenenfalls challenge. Die Zertifikate selbst stehen in der Liste kettenschluessel von Sicherung und Klartext (Base64, DER, Blatt zuerst). Gelingt die Bestätigung nicht, steht ein Eintrag schluessel mit leerer objektId und status nicht_verfuegbar mit grund (nur beim ersten Mal und bei neuem Grund), und die Kette läuft unsigniert weiter; einen neuen Versuch gibt es höchstens einmal am Tag beim Start. Ein Eintrag nicht_verfuegbar beendet den Abschnitt des vorigen Schlüssels: Dieser reicht nur bis zum Eintrag davor. Von dem Eintrag nicht_verfuegbar bis vor den nächsten Eintrag schluessel mit status verfuegbar gibt es keine gültige Signatur; eine Signatur über einen Kopf in diesem Bereich gilt als ungültig, auch wenn sie mit dem früheren Schlüssel rechnerisch stimmt. Einen neuen Schlüssel gibt es beim ersten Start nach dem Update, nach einem Neubeginn (die Schlüssel der alten Kette werden entfernt), nach einer Wiederherstellung ohne Schlüssel dieses Geräts in der Kette und nach einer Neuinstallation.

Signatur und Deutung: Nach jedem Vorgang, der Einträge anhängt, signiert WATT den neuen Kopf mit SHA256withECDSA über die ASCII-Nachricht watt-kopf-1:\<hash des Kopfs>. Die Signaturen stehen getrennt von der Kette (Liste kettensignaturen mit seq, fingerabdruck und signatur in Base64, DER) und gehen in keine Prüfsumme ein. Eine gültige Signatur bestätigt: Dieser Stand lag zu diesem Zeitpunkt im echten WATT auf einem Gerät mit diesem Schlüssel; über die Hash-Kette deckt sie alle Einträge davor mit. Schlägt das Signieren fehl, deckt die nächste Signatur die Einträge mit. Eingespielte Einträge signiert WATT nie nach; die Herkunft eingespielter Einträge zeigen deren eigene Signaturen. Beim Einspielen prüft WATT sie gegen die öffentlichen Schlüssel aus den Zertifikaten und vermerkt den Befund dauerhaft im Eintrag wiederherstellung (4.1); ungültige Signaturen verhindern das Einspielen nicht. Vor jedem Dokument signiert WATT einen noch unsignierten Kopf, sofern möglich; der QR-Code trägt dann ;s=1 (6.2).

Prüfverfahren für die Signaturen (WATTest, 8., oder eigenes Werkzeug):

1. Zertifikatskette jedes Schlüssels bis zu einer Wurzel von Google prüfen: der RSA-Wurzel für Geräte mit Werksschlüsseln oder der Wurzel „Key Attestation CA1“ (ECDSA P-384), die Google seit 2026 für neu bereitgestellte Schlüssel nutzt; jedes Zertifikat muss vom nächsten signiert sein. Die Gültigkeitszeiträume (notBefore bis notAfter, Grenzen eingeschlossen) aller Zertifikate der Kette zu einem einzigen Bezugszeitpunkt prüfen, dem Zeitpunkt der Erzeugung nach der Geräteuhr. Maßgeblich ist der zeitpunkt des Eintrags schluessel; WATT erzeugt den Schlüssel unmittelbar vor diesem Eintrag. Einzige Ausnahme ist der im Voraus erzeugte Schlüssel einer leeren Kette (der Eintrag schluessel mit challenge): Sein Eintrag entsteht erst mit dem ersten eigentlichen Eintrag, unter Umständen Tage nach der Erzeugung; dort ist creationDateTime aus der Attestierung maßgeblich (Tag 701 der Erweiterung, Millisekunden seit 1970 in UTC), und nur wenn es fehlt, der zeitpunkt des Eintrags. Sonst dient creationDateTime nur der Ergänzung: Eine Abweichung vom zeitpunkt des Eintrags ist kein Befund, beide stammen von der Geräteuhr. Für das Blattzertifikat setzt WATT keine eigenen Grenzen; Android trägt dann seine Standardwerte ein (01.01.1970 bis 01.01.2048), entscheidend sind also die Zwischenzertifikate.
2. Sperrliste https://android.googleapis.com/attestation/status: Kein Zertifikat der Kette darf darin stehen.
3. attestationApplicationId: Paketname de.achim.watt und SHA-256 des App-Signaturzertifikats 4accbc1ff7bb15f57a2364e39906568e01c4e4b53e4e5d63e77148eb3bfa99d9 (dasselbe Zertifikat signiert die Installationen aus Google Play).
4. RootOfTrust: verifiedBootState Verified und deviceLocked true.
5. Sicherheitsstufe: attestationSecurityLevel TrustedEnvironment oder StrongBox.
6. Challenge: gleich dem hash des Eintrags vor dem Eintrag schluessel (mit der Ausnahme uhr_verstellt oben) bzw. den Zufallsbytes in challenge, jeweils als 32 Bytes; fingerabdruck und zertifikateSha256 im Eintrag passen zu den Zertifikaten.
7. Signaturen: Jede Signatur mit dem Schlüssel ihres Abschnitts über watt-kopf-1:\<hash des Eintrags seq> prüfen. Ein Abschnitt reicht vom Eintrag schluessel mit status verfuegbar bis vor den nächsten Eintrag schluessel, gleich mit welchem Status, sonst bis zum Kopf. Eine Signatur ist nur gültig, wenn ihr fingerabdruck der des Abschnitts ist, in dem ihr Eintrag seq liegt.

Befund in drei Stufen:

- bestätigt: alle Schritte erfüllt.
- nicht bestätigbar (keine Fälschung belegt, aber auch keine Herkunft): keine Signaturen; Einträge vor dem ersten Schlüssel oder nach einem Eintrag nicht_verfuegbar; ein Zertifikat war zum Bezugszeitpunkt aus Schritt 1 (Zeitpunkt der Erzeugung nach der Geräteuhr) nicht gültig (die Uhr des Geräts kann falsch gegangen sein); verifiedBootState nicht Verified oder deviceLocked false (entsperrter Bootloader oder verändertes System: Android kann dann nicht für die App bürgen); Sicherheitsstufe Software (Emulator, sehr alte Geräte); Sperrliste nicht erreichbar.
- widerlegt: falsche Signatur oder Signatur mit dem falschen Schlüssel; falsche Challenge; anderer Paketname oder anderes Signaturzertifikat der App; fingerabdruck oder zertifikateSha256 passen nicht zu den Zertifikaten; ein Zertifikat ist nicht vom nächsten signiert oder steht auf der Sperrliste; die Kette führt nicht zu einer Wurzel von Google.

WATT prüft unter „Mehr → Integrität“ die Signaturen und zeigt Sicherheitsstufe, Bootloader, Android-Version und Sicherheitsupdate aus der Bestätigung; die Kette bis zur Wurzel und die Sperrliste prüft erst WATTest (8.).

Grenzen der Signatur:

- Über einen Neustart hinweg prüft WATT die Uhr nicht gegen die Monotonuhr; dann bleibt nur uhr_ruecksprung, und wer die Uhr bei ausgeschaltetem Gerät verstellt, fällt nur durch einen Rücksprung auf.
- Die Bestätigung beruht auf dem Vertrauen in Google und die Gerätehersteller, deren Schlüssel die Zertifikate ausstellen.
- Software-Attestierung (Emulator, sehr alte Geräte) ist nicht bestätigt.
- Eine Signatur trägt keinen Zeitstempel eines Dritten; zeitlich grenzen sie nur die kurzlebigen Zwischenzertifikate von Google grob nach unten ein.
- Eine Kette vor dem ersten Anker (6.) bleibt ohne äußeren Vergleich: Die Signatur zeigt, dass die Einträge in WATT entstanden, nicht, dass dort kein früherer Stand verworfen wurde.

Rechenbeispiel Uhrstand (B11): der Eintrag 1 aus 4.2 mit zwei Neustarts seit dem vorigen Eintrag und automatischer Uhrzeit.

- Eingabe (B11):

  ```
  1:120:2026-09-28T16:05:00Z8:ANGELEGT7:eintrag2:e1343:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null}1:01:24:true
  ```

- SHA-256 (B11):

  ```
  95146a2017725f7122a29f497d6693e66bb89956eb060dab075cf3d4526a68db
  ```

- bash (B11):

  ```
  printf '%s' '1:120:2026-09-28T16:05:00Z8:ANGELEGT7:eintrag2:e1343:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null}1:01:24:true' | sha256sum
  ```

- PowerShell (B11):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes('1:120:2026-09-28T16:05:00Z8:ANGELEGT7:eintrag2:e1343:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null}1:01:24:true'))).Replace('-','').ToLower()
  ```

- Python (B11):

  ```
  hashlib.sha256('1:120:2026-09-28T16:05:00Z8:ANGELEGT7:eintrag2:e1343:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null}1:01:24:true'.encode('utf-8')).hexdigest()
  ```

Rechenbeispiel Signatur (B12): Signatur des Kopfs aus B11 mit einem festen Testschlüssel (nicht der Schlüssel eines Geräts). Die Eingabe ist die signierte Nachricht, die SHA-256 ihr Digest; die Signaturprüfung braucht OpenSSL, PowerShell 7 bzw. das Python-Paket cryptography.

- Eingabe (B12):

  ```
  watt-kopf-1:95146a2017725f7122a29f497d6693e66bb89956eb060dab075cf3d4526a68db
  ```

- SHA-256 (B12):

  ```
  53078a7535d394176afad0d51f56c9284ae81ceec01a0f4489b51141c319e27f
  ```

- bash (B12):

  ```
  printf '%s' 'watt-kopf-1:95146a2017725f7122a29f497d6693e66bb89956eb060dab075cf3d4526a68db' | sha256sum
  ```

- PowerShell (B12):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes('watt-kopf-1:95146a2017725f7122a29f497d6693e66bb89956eb060dab075cf3d4526a68db'))).Replace('-','').ToLower()
  ```

- Python (B12):

  ```
  hashlib.sha256('watt-kopf-1:95146a2017725f7122a29f497d6693e66bb89956eb060dab075cf3d4526a68db'.encode('utf-8')).hexdigest()
  ```

- Öffentlicher Schlüssel (B12, SubjectPublicKeyInfo, Base64):

  ```
  MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEhxtk9HRQgKVnxYjQycLUeu8HXymjAGpwX0qjkPSoQk8DGhJBcDJ7JzkyGhoqLKbJ530pkqlgV1MAq+WeuAfrjg==
  ```

- Fingerabdruck (B12):

  ```
  f3bb2eab722f4dcb14beb4d98d68c33630966b7edea8f967fe9e3997fc229445
  ```

- Signatur (B12, DER, Base64):

  ```
  MEUCIQCKS2bSW695v1wZ6feklKXmJ5BGV6FC6twMs6zdIyg1GgIgSGlYfxermK6cfq1Xq5ywcMoaNxr4cr6a0kGg2UZWjHA=
  ```

- Signaturprüfung bash (B12, Ausgabe Verified OK):

  ```
  printf '%s' 'watt-kopf-1:95146a2017725f7122a29f497d6693e66bb89956eb060dab075cf3d4526a68db' > nachricht.txt; printf '%s' 'MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEhxtk9HRQgKVnxYjQycLUeu8HXymjAGpwX0qjkPSoQk8DGhJBcDJ7JzkyGhoqLKbJ530pkqlgV1MAq+WeuAfrjg==' | base64 -d > schluessel.der; printf '%s' 'MEUCIQCKS2bSW695v1wZ6feklKXmJ5BGV6FC6twMs6zdIyg1GgIgSGlYfxermK6cfq1Xq5ywcMoaNxr4cr6a0kGg2UZWjHA=' | base64 -d > signatur.der; openssl dgst -sha256 -verify schluessel.der -keyform DER -signature signatur.der nachricht.txt
  ```

- Signaturprüfung PowerShell 7 (B12, Ausgabe True):

  ```
  $k = [Security.Cryptography.ECDsa]::Create(); $null = $k.ImportSubjectPublicKeyInfo([Convert]::FromBase64String('MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEhxtk9HRQgKVnxYjQycLUeu8HXymjAGpwX0qjkPSoQk8DGhJBcDJ7JzkyGhoqLKbJ530pkqlgV1MAq+WeuAfrjg=='), [ref]0); $k.VerifyData([Text.Encoding]::ASCII.GetBytes('watt-kopf-1:95146a2017725f7122a29f497d6693e66bb89956eb060dab075cf3d4526a68db'), [Convert]::FromBase64String('MEUCIQCKS2bSW695v1wZ6feklKXmJ5BGV6FC6twMs6zdIyg1GgIgSGlYfxermK6cfq1Xq5ywcMoaNxr4cr6a0kGg2UZWjHA='), [Security.Cryptography.HashAlgorithmName]::SHA256, [Security.Cryptography.DSASignatureFormat]::Rfc3279DerSequence)
  ```

- Signaturprüfung Python (B12, ohne Ausnahme gültig):

  ```
  from cryptography.hazmat.primitives import hashes, serialization; from cryptography.hazmat.primitives.asymmetric import ec; import base64; serialization.load_der_public_key(base64.b64decode('MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEhxtk9HRQgKVnxYjQycLUeu8HXymjAGpwX0qjkPSoQk8DGhJBcDJ7JzkyGhoqLKbJ530pkqlgV1MAq+WeuAfrjg==')).verify(base64.b64decode('MEUCIQCKS2bSW695v1wZ6feklKXmJ5BGV6FC6twMs6zdIyg1GgIgSGlYfxermK6cfq1Xq5ywcMoaNxr4cr6a0kGg2UZWjHA='), b'watt-kopf-1:95146a2017725f7122a29f497d6693e66bb89956eb060dab075cf3d4526a68db', ec.ECDSA(hashes.SHA256()))
  ```

## 5. Die Nachweiskette

Überlastungsanzeigen und Gesprächsnotizen werden in einer eigenen Kette geführt, getrennt von der Arbeitszeitkette.

- Einträge entstehen, wenn eine Anzeige als gesendet markiert, eine Gesprächsnotiz gespeichert, ein Nachtrag dazu erstellt, ein Nachweis gelöscht oder eine Sicherung wiederhergestellt wird.
- Ein Eintrag enthält nur Art, Zeitpunkt, Referenz und die Prüfsumme des Inhalts, nicht den Inhalt selbst.
- Nicht gesendete Entwürfe erzeugen keinen Eintrag und werden beim Löschen vollständig entfernt.
- Gesendete Anzeigen und Gesprächsnotizen lassen sich löschen; ihr Inhalt verschwindet dabei, der Vermerk ihrer Existenz bleibt in der Kette.
- Gesprächsnotizen sind nach dem Speichern unveränderlich; Korrekturen erfolgen nur als Nachtrag mit eigenem Eintrag.
- Die Nachweiskette trägt keine Vermerke zur Geräteuhr. Ein Rücksprung der Uhr beim Festhalten eines Nachweises wird in der Arbeitszeitkette vermerkt (uhr_ruecksprung mit objektId nachweis, 4.1) und zählt in den Prüfvermerken mit.
- Eine Wiederherstellung ersetzt die Nachweiskette des Geräts durch die der Datei. Anzahl und letzte Prüfsumme der ersetzten Kette stehen seit dem 19.10.2026 im Eintrag wiederherstellung der Arbeitszeitkette (nachweisketteVorherAnzahl, nachweisketteVorherLetzterHash); die Nachweiskette selbst enthält weiter nur Art, Zeitpunkt, Referenz und Prüfsumme, und Arbeitszeitberichte nennen die Angaben nicht. Eine vollständige Datei ab Formatversion 4 ohne Nachweise spielt WATT nicht ein.

### 5.1 Prüfsummen der Nachweiskette und der Dokumente

- Inhaltsprüfsumme einer Anzeige bzw. Gesprächsnotiz: SHA-256 über die Felder unten, jedes als \<Länge>:\<Wert> wie in 4.2 und ohne Trennzeichen aneinandergehängt; Zeitpunkte und Darstellung wie in 4.2, Tage als JJJJ-MM-TT, leere oder fehlende Werte als 0:. Formel: SHA-256( L(Feld 1) ‖ L(Feld 2) ‖ … ).
- Prüfsumme eines Dokuments: SHA-256 über die unveränderten Bytes der PDF-Datei, wie in der Datei SHA256SUMS.txt im ZIP des Aktenauszugs (je Zeile Prüfsumme, zwei Leerzeichen, Pfad). Nachrechnen mit sha256sum datei.pdf, Get-FileHash datei.pdf -Algorithm SHA256 oder hashlib.sha256(open('datei.pdf','rb').read()).hexdigest().
- Anhänge an einer Anzeige (fremde PDFs und Fotos, die die Lehrkraft zur Anzeige ablegt, seit 23.10.2026) sind kein Teil des Nachweises: Sie gehen weder in die Inhaltsprüfsumme der Anzeige noch in die Nachweiskette ein, ändern also keinen Ketteneintrag. In die Prüfsumme des Datenbestands beim Neubeginn (4.6) gehen von jedem Anhang Name, Art, Größe und Datum ein, nicht der Inhalt. Die verschlüsselte Sicherung enthält die Inhalte vollständig als eigene verschlüsselte Teile hinter dem Hauptteil (Container der Version 3, seit 24.10.2026; Aufbau in der Beschreibung des Datenformats, doc/datenformat.md in watt_public): jeder Teil mit eigenem IV unter demselben Sicherungsschlüssel, als zusätzliche Daten der ganze Kopf, der Text WATT-Teil, die Nummer und die Länge des Teils, sodass ein vertauschter, fremder oder veränderter Teil am Prüfwert scheitert; die Datei endet genau hinter dem letzten Teil, der Klartext-Export nur Name, Art, Größe und Datum (mit „Externe Anhänge mitspeichern“ als unverschlüsselte Kopien in einer ZIP-Datei). Fehlt der Inhalt eines Anhangs in einer Sicherung (Fassung für die Cloud über 25 MB), bleiben seine Angaben beim Einspielen erhalten; ein gleicher Anhang, der auf dem Gerät schon liegt, bleibt mit Inhalt.
- Ketteneintrag der Nachweiskette: hash(n) = SHA-256( L(seq) ‖ L(zeitpunkt) ‖ L(ereignis) ‖ L(referenzId) ‖ L(pruefsumme) ‖ L(prev) ), Darstellung, prev und Startwert 0 wie in 4.2. ereignis ist ANZEIGE_GESENDET, ANZEIGE_GELOESCHT, GESPRAECHSNOTIZ_GESPEICHERT, NACHTRAG, GESPRAECHSNOTIZ_GELOESCHT, WIEDERHERSTELLUNG oder WIEDERHERSTELLUNG_UNTERBROCHEN; referenzId ist die Kennung der Anzeige, der Gesprächsnotiz oder des Nachtrags (\<Kennung der Notiz>:\<Kennung>), bei einer Wiederherstellung der Zeitpunkt der Sicherung; pruefsumme ist die Inhaltsprüfsumme, beim Löschvermerk die beim Eintragen, bei einer Wiederherstellung die SHA-256 der eingespielten Datei.

Felder einer Anzeige (Verfahren 2, für Anzeigen ab dem 04.10.2026), in dieser Reihenfolge:

- die Ziffer 2 (Verfahren)
- Kennung, Inhaltsstufe (AUSSAGE, ZUSAMMENFASSUNG, NACHWEIS), erster und letzter Tag des Zeitraums (leer ohne Zeitraum), Herkunft der Angaben (PLATZHALTER, EINMALIG, MEINE_DATEN), „nur für dieses Schreiben“ (true oder false), vorgesehene Empfänger-E-Mail
- Anzahl der Kopie-Empfänger:innen, danach je Empfänger:in die Stelle (PERSONALRAT, SCHWERBEHINDERTENVERTRETUNG, GLEICHSTELLUNGSBEAUFTRAGTE) und die E-Mail
- Betreff, gespeicherte Fassung des Anschreibens einschließlich Prüfvermerk, Datum des Schreibens
- Empfänger:in beim Versand und Sendezeitpunkt (leer, wenn nicht angegeben)
- Anzahl der Dokumente, danach je Dokument Art (ANSCHREIBEN, ZUSAMMENFASSUNG, DETAILNACHWEIS), Dateiname und SHA-256 der Datei
- Anzahl der Notizen mit fest true, danach je solche Notiz Zeitpunkt (erstelltAm) und Text, sortiert nach erstelltAm und Kennung wie im Datenformat. fest true tragen genau die Notizen, die beim Markieren als gesendet vorhanden waren; Notizen mit fest false (nach dem Markieren angelegt, nachVersand true) gehen nicht ein und zählen nicht mit
- Kopf der Arbeitszeitkette beim Erzeugen: Anzahl der Einträge, Prüfsumme und Zeitpunkt (alle drei leer, wenn er fehlt)

Anzeigen bis zum 04.10.2026 tragen Verfahren 1: ohne die Ziffer am Anfang und ohne Kettenkopf; die Kopie-Empfänger:innen stehen dort in einem einzigen Feld, je Empfänger:in \<STELLE>=\<E-Mail>, mit LF verbunden. Das Verfahren steht je Anzeige gespeichert (pruefverfahren im Datenformat). Der Erzeugungszeitpunkt und die Wiedervorlage gehören in keinem Verfahren zur Prüfsumme.

Felder einer Gesprächsnotiz: Kennung, Art (TELEFONAT, PERSOENLICH, VIDEOKONFERENZ), Zeitpunkt des Gesprächs, Dauer in Minuten (leer ohne), Gesprächspartner:innen, Anwesende, Anlass, Inhalt, Zusagen, Frist (Tag, leer ohne), Zeitpunkt der Erfassung.

Felder eines Nachtrags: Kennung, Kennung der Notiz, Text, Zeitpunkt der Erfassung.

Rechenbeispiel einer gesendeten Anzeige (B4) mit einem Dokument, dessen Inhalt die vier Bytes %PDF sind (B6); der Kettenkopf ist Eintrag 2 aus 4.2. Dazu eine Gesprächsnotiz (B5) und der Ketteneintrag zum Versand der Anzeige (B7).

- Eingabe (B4):

  ```
  1:22:a115:ZUSAMMENFASSUNG10:2026-09-0110:2026-09-3011:PLATZHALTER5:false17:sl@schule.example1:111:PERSONALRAT17:pr@schule.example20:Ueberlastungsanzeige29:Sehr geehrte Damen und Herren10:2026-10-0117:sl@schule.example20:2026-10-01T08:00:00Z1:111:ANSCHREIBEN15:Anschreiben.pdf64:315d429b7714cedb6ad04ac31240145257692630457f3c88253c5beceac760271:01:264:759a07d61bb0d53f334cf8f48b3937df7d945405ea55ffbb78f40a6bb99f19da20:2026-09-28T16:20:00Z
  ```

- SHA-256 (B4):

  ```
  71106548e39f6879dab3b120034a07da4382b67ce3d2cc3e91c21ff6613912b7
  ```

- bash (B4):

  ```
  printf '%s' '1:22:a115:ZUSAMMENFASSUNG10:2026-09-0110:2026-09-3011:PLATZHALTER5:false17:sl@schule.example1:111:PERSONALRAT17:pr@schule.example20:Ueberlastungsanzeige29:Sehr geehrte Damen und Herren10:2026-10-0117:sl@schule.example20:2026-10-01T08:00:00Z1:111:ANSCHREIBEN15:Anschreiben.pdf64:315d429b7714cedb6ad04ac31240145257692630457f3c88253c5beceac760271:01:264:759a07d61bb0d53f334cf8f48b3937df7d945405ea55ffbb78f40a6bb99f19da20:2026-09-28T16:20:00Z' | sha256sum
  ```

- PowerShell (B4):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes('1:22:a115:ZUSAMMENFASSUNG10:2026-09-0110:2026-09-3011:PLATZHALTER5:false17:sl@schule.example1:111:PERSONALRAT17:pr@schule.example20:Ueberlastungsanzeige29:Sehr geehrte Damen und Herren10:2026-10-0117:sl@schule.example20:2026-10-01T08:00:00Z1:111:ANSCHREIBEN15:Anschreiben.pdf64:315d429b7714cedb6ad04ac31240145257692630457f3c88253c5beceac760271:01:264:759a07d61bb0d53f334cf8f48b3937df7d945405ea55ffbb78f40a6bb99f19da20:2026-09-28T16:20:00Z'))).Replace('-','').ToLower()
  ```

- Python (B4):

  ```
  hashlib.sha256('1:22:a115:ZUSAMMENFASSUNG10:2026-09-0110:2026-09-3011:PLATZHALTER5:false17:sl@schule.example1:111:PERSONALRAT17:pr@schule.example20:Ueberlastungsanzeige29:Sehr geehrte Damen und Herren10:2026-10-0117:sl@schule.example20:2026-10-01T08:00:00Z1:111:ANSCHREIBEN15:Anschreiben.pdf64:315d429b7714cedb6ad04ac31240145257692630457f3c88253c5beceac760271:01:264:759a07d61bb0d53f334cf8f48b3937df7d945405ea55ffbb78f40a6bb99f19da20:2026-09-28T16:20:00Z'.encode('utf-8')).hexdigest()
  ```

- Eingabe (B5):

  ```
  2:g111:PERSOENLICH20:2026-10-02T12:00:00Z2:2012:Schulleitung0:20:Ueberlastungsanzeige36:Entscheidung bis Monatsende zugesagt12:Rueckmeldung10:2026-10-3020:2026-10-02T13:10:00Z
  ```

- SHA-256 (B5):

  ```
  91f081c27971d7668642eb508fc7ee4549cdb67b7f2284c3502dc9893917250b
  ```

- bash (B5):

  ```
  printf '%s' '2:g111:PERSOENLICH20:2026-10-02T12:00:00Z2:2012:Schulleitung0:20:Ueberlastungsanzeige36:Entscheidung bis Monatsende zugesagt12:Rueckmeldung10:2026-10-3020:2026-10-02T13:10:00Z' | sha256sum
  ```

- PowerShell (B5):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes('2:g111:PERSOENLICH20:2026-10-02T12:00:00Z2:2012:Schulleitung0:20:Ueberlastungsanzeige36:Entscheidung bis Monatsende zugesagt12:Rueckmeldung10:2026-10-3020:2026-10-02T13:10:00Z'))).Replace('-','').ToLower()
  ```

- Python (B5):

  ```
  hashlib.sha256('2:g111:PERSOENLICH20:2026-10-02T12:00:00Z2:2012:Schulleitung0:20:Ueberlastungsanzeige36:Entscheidung bis Monatsende zugesagt12:Rueckmeldung10:2026-10-3020:2026-10-02T13:10:00Z'.encode('utf-8')).hexdigest()
  ```

- Eingabe (B6):

  ```
  %PDF
  ```

- SHA-256 (B6):

  ```
  315d429b7714cedb6ad04ac31240145257692630457f3c88253c5beceac76027
  ```

- bash (B6):

  ```
  printf '%s' '%PDF' | sha256sum
  ```

- PowerShell (B6):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes('%PDF'))).Replace('-','').ToLower()
  ```

- Python (B6):

  ```
  hashlib.sha256('%PDF'.encode('utf-8')).hexdigest()
  ```

- Eingabe (B7):

  ```
  1:120:2026-10-01T08:30:00Z16:ANZEIGE_GESENDET2:a164:71106548e39f6879dab3b120034a07da4382b67ce3d2cc3e91c21ff6613912b71:0
  ```

- SHA-256 (B7):

  ```
  65abd5b0c8432113e3236d16a8764f4023a16a569ad0dd609f814e63a3a08762
  ```

- bash (B7):

  ```
  printf '%s' '1:120:2026-10-01T08:30:00Z16:ANZEIGE_GESENDET2:a164:71106548e39f6879dab3b120034a07da4382b67ce3d2cc3e91c21ff6613912b71:0' | sha256sum
  ```

- PowerShell (B7):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes('1:120:2026-10-01T08:30:00Z16:ANZEIGE_GESENDET2:a164:71106548e39f6879dab3b120034a07da4382b67ce3d2cc3e91c21ff6613912b71:0'))).Replace('-','').ToLower()
  ```

- Python (B7):

  ```
  hashlib.sha256('1:120:2026-10-01T08:30:00Z16:ANZEIGE_GESENDET2:a164:71106548e39f6879dab3b120034a07da4382b67ce3d2cc3e91c21ff6613912b71:0'.encode('utf-8')).hexdigest()
  ```

## 6. Externe Verankerung

Eine Hash-Kette auf einem einzelnen Gerät beweist nur ihre innere Stimmigkeit. Aussagekraft gegenüber Dritten gewinnt sie durch Anker außerhalb des Geräts:

- **Überlastungsanzeige:** Prüfvermerk mit WATT-Version, Anzahl der Protokolleinträge, Zeitpunkt und vollständiger Prüfsumme des Kettenkopfs zum Zeitpunkt der Erzeugung: im Mailtext und in der gespeicherten Fassung des Anschreibens als letzte Zeile, davor seit dem 08.10.2026 die Signatur-Trennzeile „-- “ (zwei Bindestriche und ein Leerzeichen) auf eigener Zeile; sie gehört wie der übrige Text zur Prüfsumme der Anzeige, früher gespeicherte Anzeigen ohne sie werden gegen ihren gespeicherten Text geprüft; in den PDFs von Anschreiben und Anhängen klein und grau am Fuß jeder Seite (6.2).
- **Berichte und Exporte (PDF, Excel):** Prüfvermerk zum Protokollstand (6.2) und Kettenvermerk mit der Anzahl der Einträge und der Anzahl erkannter Verstellungen, Rücksprünge und Abweichungen der Geräteuhr (uhr_verstellt, uhr_ruecksprung und uhr_abweichung zusammen), bei eingelesenen Buchungen mit der Anzahl der aus einer Datei eingelesenen Einträge und Abwesenheiten, nach einem Neubeginn mit dessen Zeitpunkt; im PDF am Fuß jeder Seite, in Excel als eigene Zeile.
- **Aktenauszug:** Prüfvermerke je Eintrag und zur Nachweiskette mit der vollständigen Prüfsumme ihres letzten Glieds, dazu am Fuß jeder Seite der Prüfvermerk des Änderungsprotokolls der Arbeitszeiten (6.2), beide Prüfsummen in fester Zeichenbreite; optional ZIP-Archiv mit den unveränderten Originaldokumenten und einer Liste ihrer SHA-256-Prüfsummen.

Liegt ein solcher Anker bei einer dritten Stelle, z. B. eine Überlastungsanzeige beim Dienstherrn, lässt sich später prüfen: Enthält das Protokoll an der angegebenen Position genau diesen Kopf, sind alle Einträge bis dahin seither unverändert. Nach allem, was den Kopf so festhält (Bericht, Exporte, Klartext, Aktenauszug, Überlastungsanzeige), legt WATT bei automatischer Sicherung sofort eine Sicherung an (seit 24.10.2026), damit ein herausgegebener Anker nach einem Geräteverlust nicht auf einen Eintrag zeigt, den keine Sicherung enthält.

### 6.1 Zusammenhang Prüfvermerk und Protokoll

Die Anzahl im Prüfvermerk einer Überlastungsanzeige ist die laufende Nummer (seq) des jüngsten Protokolleintrags beim Erzeugen; weil die Kette lückenlos bei 1 beginnt, ist sie zugleich die Zahl der Einträge. Die Prüfsumme ist dessen hash nach 4.2, der Stand sein Zeitpunkt in deutscher Zeit (Europe/Berlin), auf Minuten. Bei leerem Protokoll nennt der Vermerk 0 Einträge, die Prüfsumme 0 und als Stand den Zeitpunkt der Erzeugung. Zum Prüfen im Klartext-Export den Eintrag mit seq gleich der Anzahl suchen: Sein hash muss der Prüfsumme gleichen und sein zeitpunkt dem Stand. Nach einem Neubeginn bezieht sich der Vermerk auf die neue Kette. Der Kopf steht zusätzlich mit der Anzeige gespeichert (kettenkopf im Datenformat) und geht in deren Prüfsumme ein (5.1).

### 6.2 Wortlaut, Fußzeile und QR-Code des Prüfvermerks

Der Prüfvermerk lautet: „Prüfvermerk WATT \<Version>: Arbeitszeitprotokoll mit \<Anzahl> Einträgen, Stand \<TT.MM.JJJJ>, \<HH:MM> Uhr, SHA-256:“, darunter die Prüfsumme (64 Zeichen, feste Zeichenbreite) als eigene Zeile, darunter: „Mit dieser Prüfsumme lässt sich später prüfen, ob das Arbeitszeitprotokoll bis zu diesem Stand unverändert geblieben ist. Nachträge und Änderungen sind im Protokoll selbst vermerkt.“ Darauf folgt der Kettenvermerk („Änderungsprotokoll mit SHA-256-Hash-Kette: geprüft und intakt. Anzahl der Glieder: \<Anzahl>.“ bzw. „unterbrochen bei Glied Nr. \<Nummer>“, die Anzahl erkannter Verstellungen, Rücksprünge und Abweichungen der Geräteuhr, gegebenenfalls „Aus einer Datei eingelesene Einträge und Abwesenheiten: \<Anzahl>.“ – gezählt werden die Anlage-Einträge (aktion ANGELEGT) der Arten eintrag und abwesenheit, deren daten importDatei enthalten (4.1); in Bericht und Anlagen folgt „Davon im Zeitraum dieses Dokuments: \<Anzahl>.“ –, gegebenenfalls der Neubeginn des Protokolls und jede Wiederherstellung seit Beginn der Kette: „Wiederherstellungen seit Beginn der Kette: \<TT.MM.JJJJ> aus einer Sicherung (Vorgeschichte übereinstimmend); \<TT.MM.JJJJ> aus einem Klartext-Export (abweichende Vorgeschichte).“ mit „Vorgeschichte nicht abgleichbar“ bzw. „Vorgeschichte nicht abgeglichen“ für Wiederherstellungen ohne Abgleich). Im Text des Anschreibens und der Mail folgt derselbe Satz zu den Wiederherstellungen auf den Prüfvermerk. In PDFs stehen beide nur in der Fußzeile, einmal je Seite, klein und grau; im Text der Dokumente erscheinen sie nicht.

Auf der letzten Seite jedes PDFs (Anschreiben, Zusammenfassung, Einzelnachweis, Bericht, Aktenauszug) steht neben dem Vermerk ein QR-Code mit etwa 2 cm Kantenlänge (Fehlerkorrektur M, Ruhezone 4 Module). Sein Inhalt ist eine Zeile ohne persönliche Daten: watt-pruef:1;v=\<Version>;n=\<Anzahl>;t=\<JJJJ-MM-TT>T\<HH:MM>;sha256=\<64 Zeichen> und bei erkannten Verstellungen, Rücksprüngen oder Abweichungen der Geräteuhr zusätzlich ;r=\<Anzahl>, nach Wiederherstellungen ;w=\<Anzahl> und, wenn nicht jede eine übereinstimmende Vorgeschichte hat, ;wa=\<Anzahl ohne übereinstimmende Vorgeschichte>, bei eingelesenen Buchungen ;i=\<Anzahl> und, wenn der festgehaltene Kopf signiert ist, ;s=1 (seit 24.10.2026, 4.7; ohne Fingerabdruck des Schlüssels). Version, Anzahl, Zeitpunkt (deutsche Zeit, auf Minuten) und Prüfsumme sind dieselben wie im Prüfvermerk; der Code lässt sich mit jedem QR-Leser lesen und wie der Vermerk nach 6.1 gegen das Protokoll prüfen. Bei leerem Protokoll gibt es keinen Code. Dasselbe Format beschreibt das Datenformat (doc/datenformat.md in watt_public).

## 7. Sicherung und Wiederherstellung

- Sicherungsdateien sind verschlüsselt (AES-GCM, Schlüssel über Passphrase oder Notfallblatt) und enthalten beide Ketten vollständig.
- Beim Wiederherstellen prüft WATT beide Ketten und zeigt das Ergebnis an („intakt“ bzw. „unterbrochen bei …“). Ein Bruch wird dauerhaft vermerkt und in Prüfvermerken ausgewiesen.
- Abgleich mit dem Protokoll der Datei (Sicherung und vollständiger Klartext): WATT spielt die Arbeitszeitkette der Datei Eintrag für Eintrag nach (Anlage, Änderung samt Umwandlung, Löschung, eingelesene Buchungen) und vergleicht den entstehenden Stand aller Arbeitszeit-Einträge und Abwesenheiten mit Gutschrift mit den Daten der Datei, Feld für Feld, soweit das Protokoll sie trägt (Tag, Tätigkeit, Beginn, Ende, Dauer, Notiz, Quelle, Bezug zum Stundenplan, Erfassungs- und Änderungszeitpunkt, Fach, Klasse bzw. Art, Beginn und Ende). Weicht etwas ab, lehnt die Vorschau die Datei ab („Die Daten passen nicht zum Änderungsprotokoll.“) und nennt die abweichenden Einträge; ein Einspielen trotzdem gibt es nicht. Nach einem Neubeginn ist der Bestand im Start-Eintrag (4.6) der Ausgangsstand: Ein Eintrag ohne Protokolleintrag seither muss mit Kennung und Bestandsprüfsumme darin stehen, und kein unveränderter Eintrag des Bestands darf fehlen; ebenso Abwesenheiten mit Tag und Bestandsprüfsumme. Bei Start-Einträgen vor dem 19.10.2026 ohne Bestand gilt ein Eintrag ohne Protokolleintrag seither als Bestand zum Neubeginn, wenn er davor erfasst und zuletzt geändert wurde; sein Inhalt ist dann nur über die Prüfsumme des Neubeginns belegt. Dateien der Formatversionen vor 16 lassen sich nicht abgleichen (ihre Einträge tragen nicht sicher den vollen Stand, keine ausgelieferte Fassung schrieb sie) und werden nicht mehr eingespielt.
- Signaturen der Datei (seit 24.10.2026): Vor dem Einspielen prüft WATT im Hintergrund jede mitgelieferte Signatur gegen den öffentlichen Schlüssel aus den Zertifikaten der Datei und zeigt den Befund (alle gültig, ungültig ab Eintrag n, keine). Ungültige Signaturen verhindern das Einspielen nicht; der Befund steht dauerhaft im Eintrag wiederherstellung (4.1, 4.7). Schlüssel und Signaturen bleiben erhalten, WATT signiert eingespielte Einträge nicht nach.
- Derselbe Abgleich läuft seit dem 19.10.2026 auch auf dem Gerät: unter „Mehr → Integrität → Protokolle prüfen“ (eigene Zeile im Befund) und für den Kettenvermerk jedes Prüfvermerks. Passen die gespeicherten Daten nicht zum Protokoll, etwa nach einem Eingriff in die Datenbank, steht dort statt „intakt“, dass sie nicht passen, mit der Zahl der Abweichungen.
- Abgleich der Vorgeschichte: Liegt auf dem Gerät eine Kette, muss sie Glied für Glied der Anfang der eingelesenen sein oder umgekehrt. Sonst wird die Datei trotzdem eingespielt, der Eintrag wiederherstellung trägt aber dauerhaft „abweichende Vorgeschichte“; ohne Kette auf dem Gerät „Vorgeschichte nicht abgleichbar“ (4.1). Beides steht in jedem späteren Prüfvermerk (6.2).
- Die Wiederherstellung selbst wird in der Nachweiskette mit der Prüfsumme der Sicherungsdatei vermerkt und im Änderungsprotokoll mit einem Eintrag wiederherstellung, der Anzahl und letzte Prüfsumme des vorher vorhandenen Protokolls und der vorher vorhandenen Nachweiskette nennt (5.); betroffene Anzeigen und Gesprächsnotizen tragen den Hinweis „Wiederhergestellt am … aus Backup vom …“.

### 7.1 Buchungsdaten eines anderen Geräts

Eine Datei mit Buchungsdaten (Klartext mit umfang BUCHUNGEN, nur Einträge, Abwesenheiten, Tagesmarkierungen und Ferien eines Zeitraums, ohne Protokoll) ersetzt nichts, sondern ergänzt, alles oder nichts: Übernommen werden nur Einträge und Abwesenheiten. Tätigkeiten ordnet WATT ausschließlich über den angezeigten Namen von Ober- und Unterkategorie zu; fehlt eine, wird nichts übernommen. Gleiche Einträge (gleicher Beginn, gleiches Ende, gleiche Tätigkeit) und gleiche Abwesenheiten werden übersprungen; Überschneidungen und mehr Zeit, als ein Tag lang ist, verhindern die Übernahme ganz. Jede Übernahme erzeugt die Einträge nach 4.1 mit der Prüfsumme der Datei; mit ihr lässt sich die eingelesene Datei später eindeutig zuordnen. Seit dem 20.10.2026 prüft WATT vor der Übernahme auch die Zeitpunkte: Einträge in der Zukunft (Abwesenheiten dürfen vorausgeplant sein), ein Erfassungszeitpunkt vor dem Beginn eines Timer-Laufs oder nach dem Export der Datei, ein Tag, der nicht zum Beginn passt, und ein Export mehr als eine Stunde in der Zukunft verhindern die Übernahme. Einträge aus Stundenplan oder Kalenderlink des anderen Geräts behalten ihre Herkunft, der Bezug auf dessen Stundenplan entfällt. Erfassungs- und Änderungszeitpunkt eingelesener Einträge stammen aus der Datei und lassen sich nicht prüfen; WATT weist sie als „laut Datei“ aus und kennzeichnet zusätzlich den Abstand zwischen Ende der Tätigkeit und Einlesen nach der Nachtrag-Regel („Eingelesen ca. 36 Tage nach Ende“). „Importiert am“ ist der Zeitpunkt des Anlage-Eintrags im Protokoll; beim Einspielen eines Klartexts vergleicht der Abgleich (7.) ihn mit, sodass sich die Markierung nicht entfernen lässt. Der Einzelnachweis einer Überlastungsanzeige nennt eingelesene Einträge an der Herkunft („Timer (eingelesen)“), der QR-Code des Prüfvermerks ihre Anzahl (i=…), der Kettenvermerk zusätzlich die Anzahl im Zeitraum des Dokuments.

## 8. Prüfverfahren

1. Klartext-Export bzw. Aktenauszug mit ZIP von der betroffenen Person anfordern, dazu vorhandene Anker (Überlastungsanzeigen, Berichte). Den Klartext-Export gibt es als JSON (JSON Schema ab Formatversion 16) und als XML: Ausgelieferte Fassungen exportieren XML ab Formatversion 18 (WATT 0.8.109 „Cherimoya“), das XML Schema ist ab Version 18 veröffentlicht. Beide enthalten dieselben Angaben; die Prüfsumme des Bestands (4.6) lässt sich nur aus JSON nachrechnen. Datenformat und Schemas sind veröffentlicht: https://github.com/Aves83/watt_public (doc/datenformat.md, Ordner schema).
2. Das eigene Werkzeug zuerst mit den Rechenbeispielen aus 4.2, 4.6 und 5.1 abgleichen.
3. Für jeden Eintrag die Prüfsumme nach 4.2 neu berechnen und mit der gespeicherten vergleichen; die Verkettung über die Vorgänger-Prüfsumme prüfen. Die Prüfung ist auch in WATT selbst möglich (unter „Mehr → Integrität“: beide Ketten mit Befund, Anzahl, Stand, Prüfsumme des Kopfs und erkannten Rücksprüngen der Geräteuhr sowie die letzten Änderungen). Unabhängig von WATT prüft die quelloffene Prüf-App WATTest Klartext-Exporte und Aktenauszüge nach dieser Erläuterung: https://aves83.github.io/wattest/ (Quelltext https://github.com/Aves83/wattest, Lizenz EUPL-1.2). Sie läuft im Browser; die geprüften Dateien verlassen den Rechner nicht.
4. Laufende Nummern auf Lücken prüfen; bei Einträgen mit Uhrstand Neustarts und automatische Uhrzeit auswerten (4.7).
5. Zeitstempel: Steigen sie mit der laufenden Nummer monoton an, und sind sie plausibel? Liegt der Zeitpunkt eines Eintrags mehr als 2 Minuten vor dem seines Vorgängers, muss das erklärt sein: Entweder folgt unmittelbar danach ein Eintrag uhr_ruecksprung mit beiden Zeitpunkten (4.1), oder der Eintrag ist selbst ein Vermerk zur Uhr, nämlich uhr_ruecksprung mit objektId timer (Start eines Timers mit zurückgestellter Uhr) oder nachweis (Festhalten eines Nachweises mit zurückgestellter Uhr) oder uhr_verstellt mit richtung zurueck (4.7). Einen Sprung vor oder zurück bis zu einem Eintrag uhr_verstellt erklärt die Rechnung in 4.7 aus vorher und nachher; der Zeitpunkt von uhr_verstellt liegt nach der Verstellung. Einträge uhr_abweichung zeigen, dass der Beginn eines Timer-Laufs nach der Monotonuhr korrigiert wurde oder das Gerät während des Laufs neu gestartet wurde (4.1); die Prüfvermerke von Berichten und Aktenauszug nennen die Anzahl beider Arten zusammen. Rücksprünge bis 2 Minuten, vorgestellte Uhren und eine durchgehend falsch gehende Uhr vermerkt WATT nicht; die Zeitstempel stammen unverändert von der Uhr des Geräts. Seit dem 24.10.2026 (4.7) steht bei Einträgen mit Uhrstand eine Verstellung in beide Richtungen als uhr_verstellt vor dem auslösenden Eintrag bzw. an ihrem eigenen Zeitpunkt; uhr_ruecksprung folgt dann nur noch, wenn kein Bezug bestand (neustarts größer als 0 oder -1). Die Prüfvermerke zählen alle drei Arten zusammen.
6. Kopf, Anzahl und Zeitpunkt aus jedem Prüfvermerk mit dem Eintrag an derselben Position im Protokoll abgleichen.
7. Besondere Einträge auswerten: Neubeginne, Wiederherstellungen mit ihrer Vorgeschichte (4.1, 7), Löschungen mit Begründung, gekennzeichnete Nachträge und eingelesene Buchungen (importDatei, 7.1).
8. Die SHA-256-Prüfsummen aus dem ZIP mit den Originaldokumenten vergleichen.
9. Schlüssel und Signaturen nach dem Prüfverfahren in 4.7 prüfen und den Befund (bestätigt, nicht bestätigbar, widerlegt) je Abschnitt festhalten.

## 9. Grenzen der Fälschungssicherheit

- **Ein früherer Eintrag wird verändert, gelöscht oder eingefügt:** erkennbar, die Kette bricht ab dieser Stelle.
- **Die letzten Einträge werden entfernt (Kette am Ende gekürzt):** nur mit Anker erkennbar; ohne einen späteren Anker ist ein kürzeres Protokoll in sich stimmig.
- **Eine vollständig neue, stimmige Kette wird erzeugt:** ohne früheren Anker nicht erkennbar. Dafür braucht es keinen besonderen Zugriff auf das Gerät: Das Verfahren ist offen beschrieben, ein Klartext-Export lässt sich bearbeiten und seine Kette mit einfachen Werkzeugen neu berechnen. Auf einem Gerät mit eigener Kette fällt das Einspielen einer solchen Datei als „abweichende Vorgeschichte“ auf; ein herausgegebener Prüfvermerk passt nicht zur neuen Kette. Erst ein Anker bei Dritten widerlegt sie sicher.
- **Daten einer Datei werden bearbeitet, die Kette bleibt:** erkennbar; WATT spielt eine solche Datei nicht ein (7.).
- **Daten in der Datenbank des Geräts werden verändert, die Kette bleibt (z. B. mit Root-Zugriff):** erkennbar; „Integrität“ und jeder Prüfvermerk gleichen die Daten mit dem Protokoll ab und nennen die Abweichung (7.).
- **Die Geräteuhr wird verstellt:** eingeschränkt erkennbar. Zeitstempel stammen vom Gerät. Ein Rücksprung um mehr als 2 Minuten fällt auf, wenn der nächste Protokolleintrag, der Start eines Timers oder das Festhalten eines Nachweises noch mit der zurückgestellten Uhr entsteht; WATT vermerkt ihn als eigenen Eintrag uhr_ruecksprung mit beiden Zeitpunkten in der Kette, weist in der App darauf hin und nennt die Anzahl in den Prüfvermerken. Wird die Uhr zurückgestellt und vor dem nächsten Eintrag wieder vorgestellt, fällt das nicht auf. Beim Timer prüft WATT die Wanduhr zusätzlich gegen die Monotonuhr des Geräts: Ein während des Laufs verstelltes Datum oder eine verstellte Uhrzeit ändern die Dauer des Laufs nicht, der Beginn wird nach der Monotonuhr korrigiert und der Vorgang als uhr_abweichung vermerkt (4.1); ein Neustart des Geräts während des Laufs wird vermerkt, danach gilt wieder die Wanduhr. Kleinere Rücksprünge, eine vorgestellte Uhr außerhalb eines Timer-Laufs und eine durchgehend falsch gestellte Uhr fallen nicht auf. Es gibt keinen unabhängigen Zeitstempeldienst. Seit dem 24.10.2026 (4.7) gilt ohne Neustart dazwischen mehr: Jede Verstellung um mehr als 2 Minuten in beide Richtungen fällt gegen die Monotonuhr auf, beim nächsten Eintrag oder sofort, wenn Android die Änderung meldet, und steht als uhr_verstellt in der Kette, auch wenn die Uhr vor dem nächsten Eintrag wieder zurückgestellt wird; viele kleine Schritte bei ausgeschalteter automatischer Uhrzeit werden summiert. Über einen Neustart hinweg bleibt nur der Rückfall auf uhr_ruecksprung; die Neustarts selbst stehen in jedem Eintrag, ebenso ob die Uhrzeit automatisch eingestellt war. Eine von Anfang an falsch gehende Uhr fällt weiterhin nicht auf.
- **Erfundene Zeiten werden zeitnah erfasst:** nicht erkennbar. Die Kette belegt, wann etwas erfasst wurde, nicht, ob es stimmt.
- **Eine veränderte App-Version erzeugt die Daten:** für signierte Abschnitte erkennbar: Die Bestätigung von Android nennt Paketname und Signaturzertifikat der App, eine veränderte App signiert mit einem anderen Zertifikat (4.7). Einträge ohne gültige Signatur (vor dem 24.10.2026, auf Geräten ohne Bestätigung, nach einem Fehlschlag) bleiben ohne diesen Nachweis.
- **Die Debug-Variante der App erzeugt die Daten:** erkennbar am Zusatz „-debug“ in der Version jedes Prüfvermerks und in signierten Abschnitten zusätzlich an der Bestätigung von Android: Sie nennt den Paketnamen de.achim.watt.debug und das Debug-Signaturzertifikat statt des Release-Zertifikats (4.7, Schritt 3). In ihr lassen sich beide Ketten über das Debug-Menü löschen und Beispieldaten anlegen; Bildschirmfotos sind dort immer erlaubt.
- **Ein älterer Stand wird aus einer Sicherung eingespielt:** auf dem Gerät erkennbar; die Wiederherstellung wird in der Nachweiskette vermerkt, Anzahl und Kopf der ersetzten Nachweiskette stehen im Eintrag wiederherstellung (5.), und vorhandene Anker späteren Datums passen nicht zum zurückgesetzten Stand.
- **Das Protokoll wird neu begonnen:** erkennbar am Start-Eintrag; die frühere Historie ist auf dem Gerät nicht mehr prüfbar, wohl aber über früher erzeugte Anker und Sicherungen: Der Start-Eintrag nennt Anzahl, letzte Prüfsumme und Zeitpunkt der gelöschten Kette (4.6). Mit zurückgestellter Uhr lässt sich nicht neu beginnen; WATT lehnt den Neubeginn ab, wenn die Uhr mehr als 2 Minuten vor dem letzten Eintrag steht.
- **Buchungen werden aus einer Datei eingelesen:** erkennbar am Feld importDatei im Anlage-Eintrag und am Vermerk „Importiert am …“; Erfassungs- und Änderungszeitpunkt stammen aus der Datei und lassen sich nur mit deren Prüfsumme und den Ankern des anderen Geräts prüfen, nicht mit der Kette dieses Geräts.

## 10. Was WATT ausdrücklich nicht leistet

- keine elektronische Signatur der Person: Die Prüfsummen belegen Unverändertheit. Die Signatur des Geräts (4.7) belegt, dass die Einträge im echten WATT entstanden sind, also in der mit dem Release-Schlüssel signierten App und nicht in einem Nachbau oder einer veränderten App, und zwar auf einem von Android bestätigten Gerät. Sie ist keine elektronische Signatur der Person im Sinne der eIDAS-Verordnung und belegt nicht, wer die Einträge vorgenommen hat
- kein qualifizierter Zeitstempel und keine Bestätigung der Uhrzeit durch Dritte
- keine Bestätigung von Gerät oder App durch eine unabhängige Stelle: Die Bestätigung des Signaturschlüssels (4.7) stammt von Android, also von Google und dem Gerätehersteller
- keine Aussage über die Richtigkeit der Inhalte; es handelt sich um eine Selbstaufzeichnung der Lehrkraft

Die Beweiskraft der Aufzeichnungen ergibt sich aus dem Zusammenspiel von zeitnaher Erfassung, lückenloser Kette, gekennzeichneten Nachträgen und externen Ankern, nicht aus der Technik allein.

## 11. Zugang zum Quellcode

Zum Abgleich der Aussagen dieser Erläuterung mit der tatsächlichen Umsetzung wird Gutachter:innen Zugang zum Quellcode von WATT ermöglicht. Voraussetzungen sind die Unterzeichnung einer Vertraulichkeitsvereinbarung und der Nachweis eines entsprechenden Prüfauftrags. Anfragen an: hans-joachim.steudel@tum.de.
