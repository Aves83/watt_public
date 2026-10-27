Stand: 2026-10-27 (YYYY-MM-DD)

# Technische Erläuterung: Nachvollziehbarkeit und Fälschungssicherheit in WATT

WATT-Version: Vorabfassung · Stand: 27.10.2026

## 1. Zweck

Diese Erläuterung richtet sich an Gutachter:innen, Rechtsbeistände und andere Fachleute, die Aufzeichnungen aus WATT bewerten. Sie beschreibt, mit welchen technischen Mitteln WATT nachträgliche Änderungen erkennbar macht, wie sich das prüfen lässt und wo die Fälschungssicherheit endet. Neuerungen sind an die Formatversion gebunden, mit der sie zuerst ausgeliefert wurden; den Verlauf zeigt 12.

## 2. Kurzfassung

WATT stellt sicher:

- Nachträgliche Änderungen, Löschungen und Einfügungen innerhalb des Änderungsprotokolls sind erkennbar.
- Löschungen bleiben mit Inhalt nachvollziehbar, außer im eng begrenzten Fall des sofortigen Rückgängigmachens oder nach einem ausdrücklichen Neubeginn des Protokolls, der selbst sichtbar ist.
- Nachträge (Erfassung mehr als 24 Stunden nach der Tätigkeit) sind gekennzeichnet.
- Verstellungen der Geräteuhr um mehr als 2 Minuten in beide Richtungen sind im Protokoll vermerkt (uhr_verstellt, ab Formatversion 19): WATT vergleicht die Wanduhr mit der Monotonuhr des Geräts an einem festen Anker je Gerätestart, sodass sich auch viele kleine Schritte summieren, auch im Hintergrund, wenn Android eine Änderung der Uhrzeit meldet. Jeder Eintrag nennt außerdem die Neustarts des Geräts seit dem vorigen und ob die Uhrzeit automatisch eingestellt war (4.7). Ohne Anker, etwa beim ersten Eintrag nach einem Neustart, gilt zusätzlich: Rücksprünge um mehr als 2 Minuten zwischen zwei Einträgen sind vermerkt (uhr_ruecksprung).
- Die Arbeitszeitkette ist ab Formatversion 19 mit einem Schlüssel signiert, den Android in geschützter Hardware des Geräts für WATT erzeugt und bestätigt (Key Attestation, 4.7). Eine gültige Signatur belegt, dass der signierte Stand in der echten App auf einem bestätigten Gerät lag; welche Einträge sie hinsichtlich ihrer Herkunft deckt, regelt 4.7 (nur den eigenen Abschnitt seit der letzten Wiederherstellung bzw. dem Neubeginn, frühestens ab dem ersten Schlüssel oder einem Neubeginn bzw. einer Wiederherstellung unmittelbar davor).
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

Das Änderungsprotokoll schützt das Ist, also das, was nur die Lehrkraft verbindlich kennt: Jede Änderung an Arbeitszeiten und an Abwesenheiten mit Gutschrift erzeugt einen Protokolleintrag. Protokolliert werden Arbeitszeit-Einträge jeder Herkunft (Timer, von Hand, Stundenplan, Kalenderlink, Tagesblock) mit allen Änderungen und Löschungen (eintrag) und Abwesenheiten mit Gutschrift, also Krankheit, Sonderurlaub, Dienstbefreiung und Arztbesuch (abwesenheit; beim stundenweisen Arztbesuch zusätzlich mit Beginn und Ende in UTC), dazu Vorgänge am Protokoll selbst: eine Wiederherstellung aus einer Sicherung (wiederherstellung), der Neubeginn des Protokolls (neubeginn), ein erkannter Rücksprung der Geräteuhr (uhr_ruecksprung), eine beim Timer erkannte Abweichung der Wanduhr von der Monotonuhr des Geräts bzw. ein Neustart des Geräts während des Laufs (uhr_abweichung, ab Formatversion 18), eine erkannte Verstellung der Geräteuhr (uhr_verstellt) und der Schlüssel zum Signieren der Kette (schluessel), beide ab Formatversion 19 (4.7). Einstellungen stehen nicht im Protokoll: Beschäftigungsumfang mit Pflichtstunden und Teilzeit, Geburtszeitraum, GdB, Deputatskonto mit Startwert und Ausgleichen, Unterricht je Woche im Vorbereitungsdienst, Land, Schulart, Amt, Funktionen, Feiertagsoptionen, Ferien, Stundenplan, Kategorien, Tagesmarkierungen, dienstfreie Tage sowie die Zuordnung der Fächer und die Entscheidungen zu Vorschlägen des Kalenderlinks sind Einstellungen und Grundlagen des Solls, nicht das Ist; der Dienstherr kennt sie aus eigenen Akten, und sie wirken nur auf künftige Übernahmen; Berichte und Überlastungsanzeigen legen offen, mit welchen Werten gerechnet wurde, und die Nachweiskette (5.) bewahrt die Zahlen jeder gesendeten Anzeige. Ebenfalls nicht im Protokoll stehen der Vermerk „Plausibilität bestätigt“ eines Eintrags, mit dem die Lehrkraft eine Rückfrage zu ungewöhnlichen Werten bestätigt hat, und der Verweis „Ähnlich wie …“ einer eigenen Kategorie: Beide beschreiben nicht das Ist; der Vermerk ändert weder den Änderungszeitpunkt noch die Kette. Protokolle vor Formatversion 13 (Vorabversionen vor der ersten ausgelieferten Fassung) können außerdem Einträge der Arten profil, funktion, ferien, sondertag, kategorie, unterkategorie und stundenplan enthalten; sie werden wie alle anderen geprüft. Begriffe: Ein Element der Ketten heißt Glied (Protokolleintrag); App, Prüfvermerk und Kettenvermerk sagen „Glied“. In dieser Erläuterung steht „Eintrag“ in den Abschnitten zur Kette (4. bis 8.) für ein Glied; die erfassten Arbeitszeiten heißen Arbeitszeit-Eintrag. Ein Glied hat diese Felder:

- seq: laufende Nummer, beginnt bei 1 und steigt ohne Lücke um 1
- zeitpunkt: Zeitpunkt des Eintrags nach der Uhr des Geräts, in UTC, auf Millisekunden gekürzt
- aktion: ANGELEGT, GEAENDERT oder GELOESCHT
- objektArt: Art des Datensatzes, einer der Schlüssel oben
- objektId: Kennung des Datensatzes, z. B. die eines Eintrags oder das Datum einer Abwesenheit; beim Neubeginn leer
- daten: Stand des Datensatzes als JSON-Text, bei Löschungen der letzte Stand und gegebenenfalls die Begründung, bei Änderungen gegebenenfalls der Kommentar zur Änderung (Feld begruendung; beim Arbeitszeit-Eintrag steht das Feld immer, ohne Kommentar als null, bei Abwesenheiten nur mit Kommentar; ab Formatversion 19 auch der automatische Kommentar, wenn WATT den Vorgänger eines vorverlegten Timer-Beginns anpasst; entstehen bei einer Änderung mit Kommentar weitere Teilstücke eines Eintrags oder neue Tage einer Abwesenheit, tragen auch deren Anlage-Einträge den Kommentar); beim Arbeitszeit-Eintrag mit Tag, Kennung der Kategorie und Unterkategorie, Beginn, Ende, Dauer, Notiz, Quelle (MANUELL, TIMER, PLAN für den Stundenplan oder KALENDERLINK für einen Termin aus dem Kalenderlink; bei PLAN und KALENDERLINK dazu die Referenz planRef auf die Stunde bzw. den Termin), Erfassungs- und Änderungszeitpunkt; trägt der Eintrag die Schlagworte Fach oder Klasse, stehen sie als fach bzw. klasse dahinter, sonst fehlen diese Felder; wandelt die Lehrkraft einen Eintrag vom Timer, aus dem Stundenplan oder aus dem Kalenderlink in einen Tagesblock um, gilt er danach als von Hand erfasst (Quelle MANUELL, ohne Bezug zum Stundenplan oder Kalender), und das Glied dieser Änderung nennt die frühere Quelle als umgewandeltAus (TIMER, PLAN oder KALENDERLINK), sonst fehlt das Feld; am Ende stehen Kategorie und Unterkategorie im Klartext zum Zeitpunkt der Buchung (kategorie, unterkategorie, ohne Unterkategorie null), damit jeder Eintrag auch ohne die Einstellungen eindeutig beschrieben bleibt und eine spätere Umbenennung kein geprüftes Glied verändert; ist der Eintrag aus einer Datei mit Buchungsdaten eingelesen (7.1), steht ganz am Ende seines Anlage-Eintrags importDatei mit der SHA-256 dieser Datei, sonst fehlt das Feld; ab Formatversion 19 steht in Änderungs- und Lösch-Einträgen eines eingelesenen Arbeitszeit-Eintrags bzw. einer eingelesenen Abwesenheit (auch beim Arztbesuch) ganz am Ende von daten importiertAm mit dem Zeitpunkt des Einlesens in UTC (7.1, Rechenbeispiel B15), sonst fehlt das Feld; Anlage-Einträge tragen es nicht, dort ist „Importiert am“ ihr eigener Zeitpunkt
- prev: Prüfsumme des vorherigen Eintrags
- hash: Prüfsumme dieses Eintrags nach 4.2
- neustarts und autoZeit: Uhrstand bei Einträgen ab Formatversion 19 (4.7); ältere Einträge haben die Felder nicht

Rücksprung der Geräteuhr: Liegt der Zeitpunkt eines neuen Eintrags mehr als 2 Minuten vor dem des vorherigen, legt WATT den Eintrag trotzdem an und hängt unmittelbar danach einen Eintrag uhr_ruecksprung an: aktion ANGELEGT, objektId die laufende Nummer (seq) des auslösenden Eintrags, zeitpunkt wie dieser, daten {"vorher":"…","nachher":"…"} mit dem Zeitpunkt des vorherigen und des neuen Eintrags in UTC. Verglichen wird in UTC; die Umstellung zwischen Sommer- und Winterzeit löst deshalb nichts aus. Der Vermerk ist ein gewöhnliches Glied der Kette und wird wie jedes andere geprüft. Ab Formatversion 18 prüft WATT die Uhr auch beim Start eines Timers: Steht sie mehr als 2 Minuten vor dem letzten Eintrag, folgt der Vermerk uhr_ruecksprung sofort, mit objektId timer und zeitpunkt der Uhr beim Start. Ebenso beim Festhalten eines Nachweises (5.): Steht die Uhr mehr als 2 Minuten vor dem letzten Eintrag der Arbeitszeitkette oder dem letzten Glied der Nachweiskette, folgt in der Arbeitszeitkette der Vermerk uhr_ruecksprung mit objektId nachweis; die Nachweiskette selbst bleibt unverändert. Ab Formatversion 19 entsteht uhr_ruecksprung nur noch ohne Anker vor der Messung (4.7: erste Messung nach einem Neustart oder nach dem Update, Geräte ohne Monotonuhr oder Neustartzähler), dann auch mit objektId start (Öffnen der App oder einer Seite) oder zeitsignal (Android meldet eine Änderung der Uhrzeit); mit Anker erkennt WATT jede Verstellung als uhr_verstellt (4.7). Ein Vermerk zur Uhr (uhr_ruecksprung, gleich mit welcher objektId, oder uhr_verstellt mit richtung zurueck) erklärt seinen eigenen Rücksprung gegenüber dem vorigen Eintrag. Ab Formatversion 19 schreibt WATT bei leerer Kette keinen Vermerk zur Uhr.

Abweichung der Geräteuhr beim Timer (seit 20.10.2026): Beim Start eines Timers merkt sich WATT neben der Wanduhr den Stand der Monotonuhr des Geräts (Millisekunden seit dem Start des Geräts, die ein Verstellen der Uhrzeit nicht verändert) und eine Kennung des Boots; beide Werte stehen nur in der Datenbank des Geräts, nie in Sicherung oder Export. Beim Stopp, bei der Höchstdauer, beim App-Start und wenn Android eine Änderung der Uhrzeit meldet, rechnet WATT den Beginn aus der Monotonuhr nach: Stopp minus verstrichene Zeit der Monotonuhr. Weicht er mehr als 2 Minuten vom gespeicherten Beginn ab, übernimmt WATT den Beginn nach der Monotonuhr und hängt vor dem Eintrag des Laufs einen Eintrag uhr_abweichung an: aktion ANGELEGT, objektId timer, daten {"art":"korrektur","beginnUhr":"…","beginnMonoton":"…","zeitpunkt":"…","abweichungSekunden":n} mit dem Beginn nach der Wanduhr, dem Beginn nach der Monotonuhr, dem Zeitpunkt der Prüfung (alle in UTC) und der Abweichung in Sekunden (positiv, wenn die Wanduhr beim Start nachging). Wurde das Gerät während des Laufs neu gestartet (andere Boot-Kennung), zählt die Monotonuhr von neuem; dann bleibt der Beginn nach der Wanduhr, und WATT vermerkt nur {"art":"neustart","beginnUhr":"…","beginnMonoton":null,"zeitpunkt":"…","abweichungSekunden":null}. Abweichungen bis 2 Minuten bleiben unvermerkt. Die Prüfvermerke zählen uhr_abweichung zusammen mit uhr_ruecksprung und uhr_verstellt. Ab Formatversion 19 entsteht art neustart nicht mehr (der Neustart steht in neustarts des nächsten Eintrags), und art korrektur nur, wenn der Timer abweicht, ohne dass eine Verstellung erkannt wurde; eine Verstellung während des Laufs korrigiert den Timer im Eintrag uhr_verstellt (4.7).

Wiederherstellung: Der Eintrag wiederherstellung (aktion ANGELEGT) nennt in daten Zeitpunkt des Exports, ob die Datei verschlüsselt war (verschluesselt: true für eine Sicherung, false für einen Klartext), Anzahl und letzte Prüfsumme des vorher vorhandenen Protokolls und ab Formatversion 18 vorgeschichte: UEBEREINSTIMMEND (eine Kette ist Glied für Glied der Anfang der anderen), ABWEICHEND (die Ketten gehen auseinander) oder NICHT_ABGLEICHBAR (auf dem Gerät gab es keine Kette). Ältere Einträge ohne vorgeschichte gelten als nicht abgeglichen. Ab Formatversion 19 nennt er den Befund der mitgelieferten Signaturen (signaturen: GEPRUEFT, UNGUELTIG mit signaturenUngueltigAb als erstem Eintrag mit ungültiger Signatur, oder KEINE, 4.7), dazu WIDERLEGT (ein Schlüssel besteht die Prüfungen ohne Netz nicht) und NICHT_PRUEFBAR (die Prüfung war nicht möglich). GEPRUEFT heißt: alle Signaturen rechnerisch gültig und die Prüfungen ohne Netz bestanden (4.7); die Kette bis zur Wurzel von Google und die Sperrliste sind damit nicht geprüft. Nach jeder Wiederherstellung folgt ein neuer Schlüssel (4.7).

Eingelesene Buchungen: Liest die Lehrkraft Buchungsdaten eines anderen Geräts ein (7.1), erhält jeder übernommene Arbeitszeit-Eintrag ein Glied eintrag mit aktion ANGELEGT, zeitpunkt des Einlesens, einer neuen objektId und den daten wie oben, am Ende mit importDatei (SHA-256 der eingelesenen Datei, hexadezimal); eine übernommene Abwesenheit mit Gutschrift erhält einen Eintrag abwesenheit, ebenfalls mit importDatei. Erfassungs- und Änderungszeitpunkt in daten stammen aus der Datei, damit die Nachtrag-Kennzeichnung stimmt; der Eintrag selbst trägt zusätzlich „Importiert am …“. Ab Formatversion 19 tragen spätere Änderungs- und Lösch-Einträge dieses Datensatzes den Zeitpunkt des Einlesens als importiertAm (oben). Diese Einträge sind gewöhnliche Glieder der Kette; die Kette des anderen Geräts wird nicht übernommen.

### 4.2 Prüfsumme: Berechnungsvorschrift

Die Prüfsumme jedes Protokolleintrags lässt sich mit Standardwerkzeugen nachrechnen, z. B. sha256sum (Linux, macOS), Get-FileHash bzw. .NET-Klassen in der Windows-PowerShell oder Python hashlib. Online-Rechner taugen nur für die Rechenbeispiele dieser Erläuterung; echte Daten enthalten personenbezogene Angaben und gehören nur in lokale Werkzeuge (etwa WATTest, das die Datei nicht hochlädt). Tabellenkalkulationen wie Excel, LibreOffice oder Google Tabellen haben keine eingebaute SHA-256-Funktion und brauchen dafür ein Makro oder Skript.

- Hash-Verfahren: SHA-256 (FIPS 180-4), Ausgabe als 64 Zeichen Hexadezimal in Kleinbuchstaben
- Geheime Werte: keine; kein Seed, kein Salt
- Verfahrensversion: Die Arbeitszeitkette hat ein Verfahren in zwei Formen, die sich an der Zahl der Felder unterscheiden: Einträge bis Formatversion 18 haben sieben Felder, ab Formatversion 19 zusätzlich den Uhrstand (neustarts, autoZeit, 4.7). Ältere Einträge werden unverändert nach der Form mit sieben Feldern geprüft; ein Eintrag mit nur einem der beiden Felder des Uhrstands gilt als Bruch. Ein Eintrag ohne Uhrstand nach dem ersten Eintrag mit Uhrstand bricht die Kette nicht, wenn seine Prüfsumme in der Form mit sieben Feldern stimmt; WATT schreibt seitdem aber nur noch die Form mit neun Feldern. Er ist deshalb „nicht bestätigbar“ (4.8): WATT nennt solche Einträge unter „Mehr → Integrität“, in der Vorschau vor dem Einspielen und im Kettenvermerk mit ihrer Anzahl und der Nummer des ersten, die Kette heißt dort weiter intakt.
- Kodierung: Alle Eingaben aller Prüfsummen dieser Erläuterung (Felder der Ketten, daten, Bestand, Bestandsprüfsummen, Prüfsummen der Nachweise und Dokumente) wandelt WATT mit derselben Funktion in UTF-8 um. Ein einzelnes Ersatzzeichen (ungültiges UTF-16 ohne Partner, über die Oberfläche praktisch nicht einzugeben) wird dabei überall zu einem Fragezeichen (Byte 0x3F); Prüfende rechnen mit diesem Byte.
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
- **Sofortiges Rückgängigmachen:** Nur zulässig, wenn der Anlage-Eintrag noch der jüngste Eintrag der Kette ist und seitdem weder Sicherung, Export, Bericht, Überlastungsanzeige noch Aktenauszug erstellt wurde. Bei einer Abwesenheit über mehrere Tage müssen alle ihre Anlage-Einträge die jüngsten sein; Einträge aus dem Stundenplan sind ausgenommen. Dann werden Datensatz und Anlage-Eintrag entfernt; die Kette endet wieder beim vorherigen Eintrag. Da der entfernte Eintrag nirgends außerhalb festgehalten war, entsteht keine prüfbare Lücke. Die Signatur des entfernten Kopfs entfällt mit ihm, die des vorherigen bleibt (4.7). Ging dem entfernten Eintrag ein Vermerk uhr_verstellt mit objektId glied oder ein Eintrag schluessel voraus, bleibt dieser stehen; der „unmittelbar folgende Eintrag“ aus 4.7 fehlt dann, und der nächste Eintrag kann einen anderen Zeitpunkt haben. Der Anker der Uhrprüfung bleibt unberührt (4.7); die Neustarts des entfernten Eintrags zählen beim nächsten wieder mit.
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

Die Kennzeichnung erscheint in Ansichten und vollständigen Exporten; im Einzelnachweis einer Überlastungsanzeige ist sie bewusst nicht enthalten.

### 4.6 Prüfsumme des Datenbestands beim Neubeginn

Der Start-Eintrag nach einem Neubeginn hat aktion ANGELEGT, objektArt neubeginn, eine leere objektId und als daten {"bestand":"\<Prüfsumme>", …}. Ab Formatversion 18 folgen auf bestand: vorherAnzahl, vorherLetzterHash und vorherZeitpunkt (Anzahl, Prüfsumme und Zeitpunkt des letzten Eintrags der gelöschten Kette; ohne Kette 0, 0 und null), eintraege (je Eintrag Kennung und Bestandsprüfsumme) und abwesenheiten (je Abwesenheit mit Gutschrift Tag und Bestandsprüfsumme), beide nach Kennung bzw. Tag sortiert, ab Formatversion 19 am Ende bestandsform mit dem Wert 2 (Form der Bestandsprüfsummen, unten; ohne das Feld Form 1). So lässt sich ein früherer Anker mit dem Neubeginn verknüpfen, und der Abgleich (7.) prüft auch Einträge, die seit dem Neubeginn unverändert sind. Steht die Geräteuhr mehr als 2 Minuten vor dem letzten Eintrag der alten Kette, lehnt WATT den Neubeginn ab und löscht nichts. Eingabe der Prüfsumme ist der Klartext-Export aller Daten zum Zeitpunkt des Neubeginns im offenen Datenformat watt-daten, ohne Änderungsprotokoll: JSON in UTF-8 ohne Byte Order Mark, je Ebene um vier Leerzeichen eingerückt, Zeilenende LF, ohne Zeilenumbruch am Ende. Die Felder stehen in der Reihenfolge des Datenformats, auch leere; Listen sind sortiert wie im Export (Einträge nach Tag, Beginn, Erfassungszeitpunkt und Kennung, Anzeigen und Gesprächsnotizen nach Kennung). Darin ist exportiertAm 1970-01-01T00:00:00Z, appVersion leer und protokoll die leere Liste []; die Versionsnummer des Datenformats (version) geht mit ein, ebenso Nachweise, Einstellungen und ein laufender Timer.

Formel: bestand = SHA-256(Klartext-Export als JSON ohne Protokoll und ohne Schlüssel und Signaturen der Kette, mit exportiertAm 1970-01-01T00:00:00Z und leerer appVersion). Klartext-Export und Prüfsumme entstehen aus derselben Funktion. Der Kalenderlink fließt nie in die Prüfsumme ein: Der Klartext enthält ihn nie (kalenderlink ist null), nur die verschlüsselte Sicherung fügt ihn ausdrücklich hinzu. Hinweis: Ein Neubeginn mit WATT „Bergamotte“ (Formatversion 17) bei gesetztem Kalenderlink hat den Link in die Prüfsumme einbezogen; seine Prüfsumme lässt sich aus dem Klartext-Export nicht nachrechnen.

Nachrechnen lässt sich bestand nur mit einem Klartext-Export als JSON (nicht XML), der unmittelbar nach dem Neubeginn entstand, bevor sich irgendetwas an Daten, Einstellungen oder Nachweisen änderte: darin exportiertAm und appVersion wie oben setzen, die Liste protokoll durch [] ersetzen und die Felder kettenschluessel und kettensignaturen ganz entfernen (nicht durch [] ersetzen; nach einem Neubeginn signiert WATT sofort, ein Export danach enthält beide Felder). Frühere Stände hält WATT nicht vor; jeder spätere Export ergibt eine andere Prüfsumme, ohne dass das auf eine Manipulation hindeutet. Ohne einen solchen Export bleibt bestand für Prüfende ein Wert, den sie nur festhalten und mit einer anderen Quelle desselben Zeitpunkts vergleichen können (etwa einer Sicherung, die die Lehrkraft unmittelbar nach dem Neubeginn angelegt hat). Nachprüfbar bleiben auch ohne ihn: die Bestandsprüfsummen je Eintrag und Abwesenheit (eintraege, abwesenheiten, unten) für jeden Eintrag eines späteren Exports, der seit dem Neubeginn keinen Protokolleintrag hat (so prüft WATT selbst, 7.), und vorherAnzahl, vorherLetzterHash und vorherZeitpunkt gegen Prüfvermerke, Exporte und Sicherungen der gelöschten Kette.

Bestandsprüfsumme eines Eintrags: die ersten 32 Hex-Zeichen (128 Bit) der SHA-256 über die kanonische Form als JSON ohne Leerzeichen mit den Feldern id, tag, kategorieId, unterkategorieId, beginn, ende, dauerMinuten, notiz, quelle, planRef, erfasstAm, geaendertAm, fach und klasse in dieser Reihenfolge; Zeitpunkte auf Millisekunden in UTC (ohne Nachkommastellen, wenn sie null sind), fehlende Werte null, fehlendes Fach und fehlende Klasse als leerer Text. Bei Abwesenheiten die Felder tag, art, beginn und ende. Gekürzt wird, damit der Start-Eintrag auch bei vielen Einträgen klein bleibt. Das ist die Form 1 der Start-Einträge bis Formatversion 18. Ab Formatversion 19 (Form 2, bestandsform 2 im Start-Eintrag) folgt bei Einträgen und Abwesenheiten am Ende importiertAm, der Zeitpunkt des Einlesens aus einer Datei mit Buchungsdaten (7.1) in derselben Darstellung, bei eigenen Einträgen null. So lässt sich die Markierung „Importiert am“ auch nach einem Neubeginn nicht unbemerkt entfernen. Ältere Start-Einträge ohne bestandsform bleiben nach Form 1 prüfbar. Rechenbeispiel für den Eintrag aus 4.2 in der ersten Fassung (Kennung e1) in Form 1:

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

Rechenbeispiel für einen Eintrag mit Umlauten, Anführungszeichen und Tabulator in Form 1 (Kennung e3, Notiz Aufsatz "Märchen", ein Tabulator und Größe, Fach Französisch, Klasse 7c). Die Befehle setzen voraus, dass die Konsole die Umlaute als Unicode bzw. UTF-8 übergibt; Python braucht r'…', damit die umgekehrten Schrägstriche erhalten bleiben:

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

Rechenbeispiel für die Form 2 (Start-Einträge ab Formatversion 19, bestandsform 2): derselbe Eintrag e1, aus einer Datei mit Buchungsdaten am 20.10.2026 um 07:30 UTC eingelesen. Am Ende steht importiertAm; bei eigenen Einträgen ist es null.

- Eingabe (B14):

  ```
  {"id":"e1","tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"fach":"","klasse":"","importiertAm":"2026-10-20T07:30:00Z"}
  ```

- SHA-256 (B14):

  ```
  20f130875bfd1eb384eda67610412631acea29a6bc3c58bf94f7ff6f2f06c385
  ```

- Bestandsprüfsumme (B14):

  ```
  20f130875bfd1eb384eda67610412631
  ```

- bash (B14):

  ```
  printf '%s' '{"id":"e1","tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"fach":"","klasse":"","importiertAm":"2026-10-20T07:30:00Z"}' | sha256sum
  ```

- PowerShell (B14):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes('{"id":"e1","tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"fach":"","klasse":"","importiertAm":"2026-10-20T07:30:00Z"}'))).Replace('-','').ToLower()
  ```

- Python (B14):

  ```
  hashlib.sha256('{"id":"e1","tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T15:30:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":null,"fach":"","klasse":"","importiertAm":"2026-10-20T07:30:00Z"}'.encode('utf-8')).hexdigest()
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

### 4.7 Uhrstand, Schlüssel und Signatur (ab Formatversion 19)

Uhrstand je Eintrag: Jeder ab Formatversion 19 angelegte Eintrag trägt zwei weitere Felder, die in seine Prüfsumme eingehen (4.2): neustarts, die Anzahl der Neustarts des Geräts seit dem vorigen Eintrag nach dem Zähler BOOT_COUNT von Android (beim ersten Eintrag einer Kette 0; -1, wenn kein Bezug zum vorigen Eintrag besteht, etwa beim ersten Eintrag nach dem Update, beim Eintrag wiederherstellung, nach einem Zähler, der zurückging, oder ohne Zähler), und autoZeit, ob „Datum und Uhrzeit automatisch“ eingeschaltet war. Schreibt ein Vorgang mehrere Einträge, nennt der erste die Neustarts, die folgenden 0. Ältere Einträge haben die Felder nicht und rechnen unverändert mit sieben Feldern.

Anker (ab Formatversion 19): Bei der ersten Messung nach einem Gerätestart merkt sich WATT auf dem Gerät einen festen Anker aus Wanduhr, Monotonuhr (Millisekunden seit dem Start des Geräts, die ein Verstellen der Uhrzeit nicht verändert) und Neustartzähler; diese Werte stehen nie in Sicherung, Export oder Prüfsumme. Wanduhr und Monotonuhr misst WATT bei jeder Prüfung selbst und unmittelbar nacheinander, nie aus dem Zeitpunkt eines Eintrags, sodass auch eine lange Transaktion (Einspielen, Neubeginn, großer Import) keine Verstellung vortäuscht. Der Versatz einer Messung ist (Wanduhr − Wanduhr des Ankers) − (Monotonuhr − Monotonuhr des Ankers); ohne Stellvorgang bleibt er im Millisekundenbereich gleich. WATT vergleicht ihn mit dem zuletzt vermerkten Versatz (beim Anker 0). Der Anker hängt an keinem Eintrag: Einträge, „Sofort wieder löschen“ (4.4), Wiederherstellung, Neubeginn und die automatische Uhrzeit ändern ihn nicht. Einzige Ausnahme: Bei eingeschalteter automatischer Uhrzeit führt eine Änderung des Versatzes um weniger als 5 Sekunden je Messung den Anker nach (Korrektur der Netzzeit); sie zählt nicht. Zeitzone und Sommerzeit ändern die Zeitpunkte in UTC nicht und lösen nichts aus, gleich ob Android die Zeitzone setzt oder die Lehrkraft; wer nach einer Reise die Uhrzeit statt der Zeitzone von Hand stellt, verstellt die Uhr wirklich.

Verstellung der Geräteuhr (uhr_verstellt): Weicht der Versatz einer Messung um mehr als 2 Minuten vom zuletzt vermerkten ab, in eine der beiden Richtungen, entsteht ein Eintrag uhr_verstellt, und der neue Versatz gilt als vermerkt. Viele kleine Schritte summieren sich so von selbst: Sobald sie zusammen mehr als 2 Minuten seit dem letzten Vermerk ergeben, entsteht der Eintrag, mit und ohne automatische Uhrzeit. aktion ANGELEGT, daten {"vorher":"…","nachher":"…","abweichungSekunden":n,"richtung":"vor"} bzw. "zurueck", mit der Zeit, die die Uhr ohne Verstellung seit dem letzten Vermerk zeigen müsste (vorher), und der Zeit, die sie zeigte (nachher, UTC). abweichungSekunden trägt ein Vorzeichen: Es ist nachher − vorher in ganzen Sekunden, zur Null hin gekürzt, also die Änderung des Versatzes seit dem letzten Vermerk, positiv bei richtung vor und negativ bei richtung zurueck (eine um eine Stunde zurückgestellte Uhr ergibt etwa -3600); der Betrag ist immer mindestens 120. richtung ist genau dann vor, wenn abweichungSekunden positiv ist. schritte steht nur, wenn sich die Verstellung aus mehreren Messungen ergibt: die Zahl der Messungen seit dem letzten Vermerk (bzw. seit dem Anker), bei denen sich der Versatz um mindestens eine Sekunde geändert hat, ohne die nachgeführten Korrekturen der Netzzeit, einschließlich der auslösenden Messung, eine ganze Zahl größer als 1; ein einzelner Sprung hat kein Feld schritte. Läuft dabei ein Timer, korrigiert WATT seinen Beginn nach seiner eigenen Monotonuhr im selben Eintrag (timerBeginnVorher, timerBeginnNachher); ein Ereignis ergibt genau einen Eintrag. Bei leerer Kette entsteht kein Vermerk; der Versatz gilt dann als vermerkt.

Gewöhnliche Ursachen von uhr_verstellt, die keine Manipulation sind: die Uhrzeit von Hand gestellt statt der Zeitzone (oben), eine falsche Netzzeit des Netzbetreibers, die nach langer Zeit ohne Netz korrigierte Uhr und das Fortsetzen eines Emulator-Abbilds; autoZeit des Vermerks zeigt, ob die automatische Uhrzeit eingeschaltet war.

objektId nennt den Anlass: glied (vor einem neuen Eintrag, der ihm unmittelbar folgt; wird dieser sofort wieder gelöscht, fehlt er, 4.4), die laufende Nummer eines Eintrags neubeginn oder wiederherstellung (der Vermerk folgt diesem Eintrag, damit der Start-Eintrag des Neubeginns der erste bleibt und die Verstellung hinter der Grenze der eingespielten Einträge steht), zeitsignal (Android meldet eine Änderung der Uhrzeit; geprüft und vermerkt auch im Hintergrund ohne Entsperren), timer (Start, Stopp, Wechsel, Vorverlegen des Beginns oder Höchstdauer eines Timers, auch bei einem Stopp über die Benachrichtigung), nachweis (Festhalten eines Nachweises) oder start (Öffnen der App, Prüfung der Höchstdauer im Vordergrund und Öffnen einzelner Seiten, etwa der Benachrichtigungen).

Neustart: Die erste Messung in einem neuen Boot setzt den neuen Anker, zählt die Neustarts für den nächsten Eintrag weiter und verankert einen laufenden Timer neu (Beginn nach der Monotonuhr = Monotonuhr jetzt − (jetzt − Beginn)). Dabei prüft WATT den Beginn des Boots (Wanduhr − Monotonuhr): Bei einem echten Neustart mit richtiger Uhr beginnt der neue Boot nach dem alten. Liegt sein Beginn mehr als 2 Minuten vor dem des alten (nach dem zuletzt vermerkten Versatz), entsteht uhr_verstellt mit richtung zurueck, auch wenn sich der Neustartzähler geändert hat; so fällt auch ein ohne Neustart veränderter Zähler auf, wenn die Uhr danach zurückgestellt ist. vorher ist dann die Zeit, die die Uhr nach dem alten Boot mindestens zeigen müsste.

Bezug für neustarts und uhr_ruecksprung: Prüfende erkennen an neustarts des ersten Eintrags, den ein Vorgang schreibt, ob ein Bezug zum vorigen Eintrag bestand:

- 0: Bezug bestand, oder es ist der erste Eintrag einer Kette (seq 1, auch der Start-Eintrag neubeginn). Die weiteren Einträge desselben Vorgangs haben ebenfalls 0, darunter ein uhr_ruecksprung, der seinem auslösenden Eintrag folgt.
- größer als 0: Das Gerät wurde seitdem so oft neu gestartet.
- -1: kein Bezug: erster Eintrag nach dem Update auf eine Fassung mit Uhrstand, der Eintrag wiederherstellung (nur wenn die eingespielte Kette mit demselben Kopf endet wie die des Geräts zuvor, zählt WATT weiter), Gerät ohne Monotonuhr oder Neustartzähler (dann haben alle ersten Einträge eines Vorgangs -1) oder ein Zähler, der zurückging (etwa nach dem Zurücksetzen des Geräts).

uhr_ruecksprung bei Einträgen mit Uhrstand: WATT schreibt ihn nur, wenn vor der Messung kein Anker bestand (erste Messung nach einem Neustart oder nach dem Update, Geräte ohne Monotonuhr oder Neustartzähler); sonst steht eine Verstellung als uhr_verstellt in der Kette. Prüfende erkennen das so:

- objektId ist eine laufende Nummer: Der auslösende Eintrag (seq gleich objektId, unmittelbar davor) hat neustarts größer als 0 oder -1; der Vermerk selbst hat neustarts 0. vorher ist der Zeitpunkt des Eintrags vor dem auslösenden.
- objektId timer, nachweis, start oder zeitsignal (Prüfung ohne eigenen Eintrag, Anlässe wie bei uhr_verstellt): Der Vermerk selbst hat neustarts größer als 0 oder -1. Bei nachweis ist vorher der spätere Zeitpunkt aus letztem Eintrag der Arbeitszeitkette und letztem Glied der Nachweiskette (5.), muss also nicht dem vorigen Eintrag entsprechen. Bei leerer Arbeitszeitkette entsteht kein Vermerk; Ketten von Vorabversionen können einen uhr_ruecksprung mit objektId nachweis als ersten Eintrag haben (seq 1, prev 0, neustarts 0, vorher aus der Nachweiskette).
- In eingespielten Einträgen (vor dem Eintrag wiederherstellung) beziehen sich neustarts und uhr_ruecksprung auf das Gerät, auf dem sie entstanden; ein uhr_ruecksprung mit der seq des Eintrags wiederherstellung folgt, wenn auf dem Gerät noch kein Anker bestand und die Uhr beim Einspielen mehr als 2 Minuten vor dem letzten eingespielten Eintrag stand.
- Einträge ohne Uhrstand (vor Formatversion 19, sieben Felder) folgen der Regel aus 4.1 ohne Bezug.

Hat bei einem uhr_ruecksprung der auslösende Eintrag (objektId mit laufender Nummer) bzw. der Vermerk selbst (übrige objektId) neustarts 0, außer im Fall seq 1 oben, passt er nicht zu dieser Regel; das ist ein Befund der Stufe „nicht bestätigbar“ (4.8), kein Beleg einer Manipulation. Umgekehrt ist ein Rücksprung von mehr als 2 Minuten zwischen zwei Einträgen mit Bezug ohne uhr_verstellt ebenfalls „nicht bestätigbar“ (4.8).

Zeitpunkt des Eintrags uhr_verstellt: Er entsteht erst, wenn WATT die Verstellung erkennt, also nach ihr, und trägt die Zeit der verstellten Uhr in diesem Moment; zeitpunkt ist der Zeitpunkt des Vorgangs und liegt höchstens um dessen Dauer vor nachher (in der Regel Millisekunden, beim Einspielen oder Neubeginn auch länger). Bei objektId glied ist das zugleich der Zeitpunkt des unmittelbar folgenden Eintrags (sofern er nicht sofort wieder gelöscht wurde), bei einer laufenden Nummer der des Eintrags neubeginn bzw. wiederherstellung, bei zeitsignal liegt er in der Regel wenige Sekunden nach der Verstellung, bei den übrigen Anlässen beim jeweiligen Vorgang. Alle Einträge davor tragen die Zeit der Uhr vor der Verstellung, alle danach die der verstellten Uhr. Einen Zeitsprung zwischen dem vorigen Eintrag und dem Eintrag uhr_verstellt erklären Prüfende so:

zeitpunkt(uhr_verstellt) − zeitpunkt(voriger Eintrag) = (vorher − zeitpunkt(voriger Eintrag)) + (nachher − vorher).

Der zweite Teil ist die Verstellung seit dem letzten Vermerk; er gleicht abweichungSekunden (in ganzen Sekunden, zur Null hin gekürzt), mit und ohne schritte. Der erste Teil ist die verstrichene Zeit nach der Monotonuhr plus die Änderung des Versatzes, die vor dem vorigen Eintrag schon bestand und noch nicht vermerkt war, und plus die nachgeführten Korrekturen der Netzzeit. Die unvermerkte Änderung liegt nach der Regel oben immer zwischen −2 und +2 Minuten. Daraus folgt die Prüfregel: Der erste Teil ist nicht kleiner als −2 Minuten (−120 Sekunden, Grenze eingeschlossen), mit und ohne schritte; ausgenommen sind nur nachgeführte Korrekturen der Netzzeit zurück bei eingeschalteter automatischer Uhrzeit, jede unter 5 Sekunden. Nach einer Wiederherstellung oder einem Neubeginn gilt die Regel erst ab dem ersten Eintrag des Geräts danach, denn der vorige Eintrag stammt dann aus der Datei bzw. aus der gelöschten Kette. Liegt der erste Teil unter der Grenze, ist das ein Hinweis (4.8), kein Beleg einer Manipulation. Ein Rücksprung von mehr als 2 Minuten bis zum Eintrag uhr_verstellt mit richtung zurueck ist damit erklärt; ein uhr_ruecksprung folgt nicht.

Schlüssel: Für jede Kette lässt WATT Android auf dem Gerät einen Schlüssel erzeugen (EC P-256, nur zum Signieren, SHA256withECDSA) im Android-Schlüsselspeicher in geschützter Hardware (TEE, bewusst nicht StrongBox), ohne Bindung an die Anmeldung. Der private Schlüssel verlässt das Gerät nie und steht in keiner Datei. Android bestätigt den Schlüssel mit einer Zertifikatskette (Key Attestation): Das Blattzertifikat enthält den öffentlichen Schlüssel und die Erweiterung 1.3.6.1.4.1.11129.2.1.17 mit Sicherheitsstufe, Challenge, der App (Paketname und SHA-256 ihres Signaturzertifikats), Android-Version, Stand der Sicherheitsupdates und RootOfTrust (verifizierter Start, Bootloader gesperrt); Gerätekennungen fordert WATT nicht an. Die Challenge (attestationChallenge) ist immer genau 32 Bytes lang:

- Kette mit Einträgen: der hash des Kopfs der Kette beim Erzeugen des Schlüssels, als Hexadezimaltext zu 32 Bytes dekodiert (nicht die 64 Zeichen als Text). Das ist der Eintrag unmittelbar vor dem Eintrag schluessel (seq um 1 kleiner). Einzige Ausnahme: Erkennt WATT beim Anhängen des Eintrags schluessel eine Verstellung der Uhr, steht dazwischen ein Eintrag uhr_verstellt mit objektId glied; dann ist die Challenge der hash des Eintrags vor diesem. Nach einem Neubeginn und nach einer Wiederherstellung gilt das ebenso: Der Eintrag schluessel folgt dem Eintrag neubeginn bzw. wiederherstellung oder einem Vermerk zur Uhr dahinter.
- Leere Kette: WATT erzeugt den Schlüssel im Voraus mit 32 Zufallsbytes als Challenge. Sein Eintrag schluessel ist dann Eintrag 1 der Kette (seq 1, prev 0) mit demselben Zeitpunkt wie der erste eigentliche Eintrag, der ihm folgt (wird dieser sofort wieder gelöscht, 4.4, steht der Eintrag schluessel allein, und der nächste Eintrag hat einen anderen Zeitpunkt), und trägt die Zufallsbytes in daten als challenge, 64 Zeichen Hexadezimal in Kleinbuchstaben. Nur dieser Eintrag schluessel hat das Feld challenge.

Der Eintrag schluessel (aktion ANGELEGT, objektId der Fingerabdruck) trägt in daten status verfuegbar, fingerabdruck (SHA-256 über die SubjectPublicKeyInfo des öffentlichen Schlüssels in DER), zertifikateSha256 (SHA-256 über die Zertifikate in DER hintereinander, Blatt zuerst), beide als 64 Zeichen Hexadezimal in Kleinbuchstaben, sicherheitsstufe (SOFTWARE, TEE oder STRONGBOX) und gegebenenfalls challenge. Die Zertifikate selbst stehen in der Liste kettenschluessel von Sicherung und Klartext (Base64, DER, Blatt zuerst). Zu jedem Eintrag schluessel mit status verfuegbar steht sein Schlüssel in kettenschluessel, auch nach Wiederherstellung und Neubeginn: WATT speichert ihn im selben Vorgang wie den Eintrag, eine Wiederherstellung übernimmt die Schlüssel der Datei, und ein Neubeginn entfernt die alten Schlüssel zusammen mit ihren Einträgen. Fehlt der Schlüssel zu einem solchen Eintrag, schreibt WATT das so nie; die Signaturen seines Abschnitts lassen sich ohne ihn weder bestätigen noch widerlegen. Das ist „nicht bestätigbar“ (4.8), nicht „widerlegt“. Gelingt die Bestätigung nicht, steht ein Eintrag schluessel mit leerer objektId und status nicht_verfuegbar mit grund (nur beim ersten Mal und bei neuem Grund), und die Kette läuft unsigniert weiter; einen neuen Versuch gibt es höchstens einmal am Tag beim Start. Ein Eintrag nicht_verfuegbar beendet den Abschnitt des vorigen Schlüssels: Dieser reicht nur bis zum Eintrag davor. Einen neuen Schlüssel gibt es beim ersten Start nach dem Update, nach einem Neubeginn, nach jeder Wiederherstellung (auch wenn der Schlüssel der eingespielten Kette noch auf dem Gerät liegt) und nach einer Neuinstallation; dabei entfernt WATT alle übrigen Schlüssel aus dem Schlüsselspeicher des Geräts, beim Start alle außer dem aktiven und dem im Voraus erzeugten.

Signatur: Nach jedem Vorgang, der Einträge anhängt, und sofort nach einem neuen Eintrag schluessel signiert WATT den neuen Kopf mit SHA256withECDSA über die ASCII-Nachricht watt-kopf-1:\<hash des Kopfs>, mit dem Schlüssel des letzten Eintrags schluessel, sofern nach ihm weder ein Eintrag wiederherstellung noch ein Eintrag neubeginn liegt. Die Signaturen stehen getrennt von der Kette (Liste kettensignaturen mit seq, fingerabdruck und signatur in Base64, DER) und gehen in keine Prüfsumme ein. Je Eintrag gibt es höchstens eine Signatur, und nur zu Einträgen der Kette (seq von 1 bis zum Kopf). Mehrere Signaturen zu einem Eintrag schreibt WATT nie; eine Datei mit mehreren Signaturen zu einem Eintrag oder mit einer Signatur außerhalb der Kette ist für WATT ungültig und wird nicht eingespielt. Ein Prüfwerkzeug wertet mehrere Signaturen zu einem Eintrag als „nicht bestätigbar“ (4.8). Schlägt das Signieren fehl, deckt die nächste Signatur die Einträge mit. Eingespielte Einträge signiert WATT nie nach, und kein Schlüssel aus einer Datei signiert nach dem Einspielen weiter: Den Kopf wiederherstellung deckt erst die erste Signatur des neuen Schlüssels. Beim Einspielen prüft WATT die Signaturen der Datei und vermerkt den Befund dauerhaft im Eintrag wiederherstellung (4.1); ungültige oder widerlegte Signaturen verhindern das Einspielen nicht. Vor jedem Dokument signiert WATT einen noch unsignierten Kopf, sofern möglich; der QR-Code trägt ;s=1 (6.2), wenn der festgehaltene Kopf eine rechnerisch gültige Signatur des aktiven Schlüssels trägt.

Was eine gültige Signatur belegt (Herkunft): Eine gültige Signatur bei Eintrag j belegt, dass der signierte Stand in der echten App auf einem bestätigten Gerät lag. Hinsichtlich der Herkunft deckt sie über die Hash-Kette nur die Einträge ihres Abschnitts, vom Beginn des Abschnitts (einschließlich) bis j. Der Abschnitt beginnt beim letzten Eintrag wiederherstellung bzw. neubeginn vor j, wenn zwischen ihm und dem nächsten Eintrag schluessel mit status verfuegbar nur Vermerke zur Uhr stehen (uhr_verstellt, uhr_ruecksprung, uhr_abweichung) oder wenn er nach dem ersten Eintrag schluessel mit status verfuegbar der Kette liegt; sonst beim ersten Eintrag schluessel mit status verfuegbar der Kette. Ohne Eintrag wiederherstellung bzw. neubeginn vor j beginnt er ebenfalls beim ersten Eintrag schluessel mit status verfuegbar. Ein Neubeginn und eine Wiederherstellung, auf die unmittelbar (bis auf Vermerke zur Uhr) ein neuer Schlüssel folgt, gehören also immer zum Abschnitt dieses Schlüssels, gleich ob er der erste der Kette ist; der Start-Eintrag eines Neubeginns (Eintrag 1) wird so gedeckt. Dasselbe gilt für eine wiederherstellung als Eintrag 1 (Einspielen einer Sicherung in eine leere Kette): Folgt ihr bis zum ersten Eintrag schluessel mit status verfuegbar nichts als Vermerke zur Uhr, beginnt der Abschnitt dieses Schlüssels bei Eintrag 1. Bei Eintrag 1 beginnt ein Abschnitt also nur, wenn dort neubeginn, wiederherstellung oder dieser erste Eintrag schluessel steht (leere Kette, oben). Daraus folgt:

- Einträge vor dem Beginn des ersten Abschnitts deckt keine Signatur, also die Einträge vor dem ersten Eintrag schluessel mit status verfuegbar außer einem Eintrag neubeginn bzw. wiederherstellung unmittelbar davor (oben), etwa die Einträge vor dem Update auf eine Fassung mit Signatur, Einträge auf einem Gerät, auf dem zunächst kein bestätigter Schlüssel verfügbar war, und ein Eintrag neubeginn bzw. wiederherstellung, auf den der erste Schlüssel der Kette erst nach weiteren Einträgen folgt (etwa weil zunächst kein bestätigter Schlüssel verfügbar war). Unter „Mehr → Integrität“ und in der Vorschau vor dem Einspielen nennt WATT diese Einträge („Glieder Nr. 1 bis n stammen aus der Zeit vor dem ersten Schlüssel und sind nicht signiert“) und sagt dann „Alle vorhandenen Signaturen sind gültig“ statt „Alle Signaturen sind gültig“; den Befund zeigt es erst nach Abschluss der Prüfung. Ihre Herkunft ist nicht bestätigbar; belegt ist nur, dass sie beim ersten Schlüssel in WATT lagen (wie beim Bestand eines Neubeginns, unten). Bis dahin lagen sie unsigniert auf dem Gerät.
- Einträge vor einer Wiederherstellung sind eingespielt. Ihre Herkunft belegen nur Signaturen mit seq kleiner als die Wiederherstellung, also die, die mit der Datei kamen; eine Signatur danach deckt sie nicht. Ohne eine solche Signatur ist ihre Herkunft nicht bestätigbar.
- Einträge nach der letzten gültigen Signatur eines Abschnitts (bis zur nächsten Grenze) deckt keine Signatur; ihre Herkunft ist nicht bestätigbar.
- Bei Einträgen mit importDatei (7.1) bestätigt die Signatur das Einlesen in WATT, nicht Erfassungs- und Änderungszeitpunkt aus der Datei.
- Der Bestand eines Neubeginns (eintraege, abwesenheiten in neubeginn, 4.6) besteht nicht aus Einträgen der Kette; seine Herkunft ist nicht bestätigbar, belegt ist nur, dass dieser Bestand beim Neubeginn in WATT lag.

Prüfungen in WATT ohne Netz (ab Formatversion 19): Bei der Vorschau vor dem Einspielen und unter „Mehr → Integrität“ prüft WATT je Eintrag schluessel: Paketname de.achim.watt und SHA-256 des App-Signaturzertifikats (Schritt 3 unten; die Debug-Variante erkennt zusätzlich ihr eigenes Paket und Zertifikat und zeigt „Debug-Variante“), die Challenge (Schritt 6), zertifikateSha256 gegen die mitgelieferten Zertifikate und die Signatur jedes Zertifikats durch das nächste. Weicht etwas ab, gilt der Schlüssel als widerlegt. Fehlt der Schlüssel in kettenschluessel, ist sein Abschnitt nicht bestätigbar (oben); seine Signaturen zählen dann weder als gültig noch als ungültig. Die Kette bis zur Wurzel von Google und die Sperrliste prüft WATT nicht; Vorschau und Integrität sagen das in einem Satz.

Prüfverfahren für die Signaturen (WATTest, 8., oder eigenes Werkzeug):

1. Zertifikatskette jedes Schlüssels bis zu einer Wurzel von Google prüfen: der RSA-Wurzel für Geräte mit Werksschlüsseln oder der Wurzel „Key Attestation CA1“ (ECDSA P-384), die Google seit 2026 für neu bereitgestellte Schlüssel nutzt; jedes Zertifikat muss vom nächsten signiert sein. Die Gültigkeitszeiträume (notBefore bis notAfter, Grenzen eingeschlossen) aller Zertifikate der Kette zu einem einzigen Bezugszeitpunkt prüfen, dem Zeitpunkt der Erzeugung nach der Geräteuhr. Maßgeblich ist der zeitpunkt des Eintrags schluessel; WATT erzeugt den Schlüssel unmittelbar vor diesem Eintrag. Einzige Ausnahme ist der im Voraus erzeugte Schlüssel einer leeren Kette (der Eintrag schluessel mit challenge): Sein Eintrag entsteht erst mit dem ersten eigentlichen Eintrag, unter Umständen Tage nach der Erzeugung; dort ist creationDateTime aus der Attestierung maßgeblich (Tag 701 der Erweiterung, Millisekunden seit 1970 in UTC), und nur wenn es fehlt, der zeitpunkt des Eintrags. Sonst dient creationDateTime nur der Ergänzung: Eine Abweichung vom zeitpunkt des Eintrags ist kein Befund, beide stammen von der Geräteuhr. Für das Blattzertifikat setzt WATT keine eigenen Grenzen; Android trägt dann seine Standardwerte ein (01.01.1970 bis 01.01.2048), entscheidend sind also die Zwischenzertifikate.
2. Sperrliste https://android.googleapis.com/attestation/status: Steht ein Zertifikat der Kette darin, ist der Abschnitt dieses Schlüssels nicht bestätigbar, nicht widerlegt (Befund unten).
3. attestationApplicationId: Paketname de.achim.watt und SHA-256 des App-Signaturzertifikats 4accbc1ff7bb15f57a2364e39906568e01c4e4b53e4e5d63e77148eb3bfa99d9 (dasselbe Zertifikat signiert die Installationen aus Google Play).
4. RootOfTrust: verifiedBootState Verified und deviceLocked true.
5. Sicherheitsstufe: attestationSecurityLevel TrustedEnvironment oder StrongBox.
6. Challenge: gleich dem hash des Eintrags vor dem Eintrag schluessel (mit der Ausnahme uhr_verstellt oben) bzw. den Zufallsbytes in challenge, jeweils als 32 Bytes; fingerabdruck und zertifikateSha256 im Eintrag passen zu den Zertifikaten.
7. Signaturen: Jede Signatur mit dem Schlüssel ihres Abschnitts über watt-kopf-1:\<hash des Eintrags seq> prüfen. Ein Abschnitt reicht vom Eintrag schluessel mit status verfuegbar bis vor den nächsten Eintrag schluessel, gleich mit welchem Status, und bis vor den nächsten Eintrag wiederherstellung, sonst bis zum Kopf. Eine Signatur ist nur gültig, wenn ihr fingerabdruck der des Abschnitts ist, in dem ihr Eintrag seq liegt; eine Signatur ab einer Wiederherstellung mit einem Schlüssel, dessen Eintrag schluessel davor liegt, ist ungültig. Danach die Herkunft je Eintrag nach der Regel oben bestimmen.

Befund in drei Stufen:

- bestätigt: alle Schritte erfüllt, für die Einträge, die eine gültige Signatur nach der Regel oben deckt.
- nicht bestätigbar (keine Fälschung belegt, aber auch keine Herkunft): keine Signaturen; mehrere Signaturen zu einem Eintrag; ein Eintrag schluessel mit status verfuegbar, dessen Schlüssel in kettenschluessel fehlt; Einträge vor dem Beginn des ersten Abschnitts oder nach einem Eintrag nicht_verfuegbar; Einträge nach der letzten gültigen Signatur ihres Abschnitts; eingespielte Einträge ohne Signatur vor der Wiederherstellung; der Bestand eines Neubeginns; ein Zertifikat war zum Bezugszeitpunkt aus Schritt 1 (Zeitpunkt der Erzeugung nach der Geräteuhr) nicht gültig (die Uhr des Geräts kann falsch gegangen sein); verifiedBootState nicht Verified oder deviceLocked false (entsperrter Bootloader oder verändertes System: Android kann dann nicht für die App bürgen); Sicherheitsstufe Software (Emulator, sehr alte Geräte); ein Zertifikat steht auf Googles Sperrliste (die Liste nennt keinen Zeitpunkt der Sperre; ob das Zertifikat beim Signieren schon gesperrt war, lässt sich nicht feststellen); Sperrliste nicht erreichbar.
- widerlegt: falsche Signatur oder Signatur mit dem falschen Schlüssel; falsche Challenge; anderer Paketname oder anderes Signaturzertifikat der App; fingerabdruck oder zertifikateSha256 passen nicht zu den Zertifikaten; ein Zertifikat ist nicht vom nächsten signiert; die Kette führt nicht zu einer Wurzel von Google.

WATT zeigt unter „Mehr → Integrität“ je Schlüssel, von welchem bis zu welchem Eintrag er signiert hat und wie viele Einträge danach ohne seine Signatur folgen, markiert die eingespielten Einträge (bis vor die letzte Wiederherstellung) und nennt Sicherheitsstufe, Bootloader, Android-Version und Sicherheitsupdate aus der Bestätigung; die Kette bis zur Wurzel und die Sperrliste prüft erst WATTest (8.).

Grenzen der Signatur:

- Über einen Neustart hinweg prüft WATT die Uhr nur mit dem Beginn des Boots (oben); wer die Uhr bei ausgeschaltetem Gerät um weniger als die Laufzeit des vorigen Boots zurückstellt, fällt nur durch einen Rücksprung hinter den letzten Eintrag auf.
- Die Bestätigung beruht auf dem Vertrauen in Google und die Gerätehersteller, deren Schlüssel die Zertifikate ausstellen.
- Software-Attestierung (Emulator, sehr alte Geräte) ist nicht bestätigt.
- Eine Signatur trägt keinen Zeitstempel eines Dritten; zeitlich grenzen sie nur die kurzlebigen Zwischenzertifikate von Google grob nach unten ein.
- Eine Kette vor dem ersten Anker (6.) bleibt ohne äußeren Vergleich: Die Signatur zeigt, dass die Einträge ihres Abschnitts in WATT lagen, nicht, dass dort kein früherer Stand verworfen wurde.

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

Rechenbeispiel Änderung eines eingelesenen Eintrags (B15): Der Eintrag e1, wie in B14 am 20.10.2026 um 07:30 UTC aus einer Datei mit Buchungsdaten eingelesen (7.1), wird am 21.10.2026 um 09:00 UTC geändert: Ende 16:00 UTC, ohne Kommentar, ohne Neustart seit dem vorigen Eintrag und mit automatischer Uhrzeit (seq 2, aktion GEAENDERT; prev ist hier zur Vereinfachung die Prüfsumme von B11). Ab Formatversion 19 steht am Ende von daten importiertAm mit dem Zeitpunkt des Einlesens (4.1); der Abgleich (7.) nimmt „Importiert am“ aus diesem Feld.

- Eingabe (B15):

  ```
  1:220:2026-10-21T09:00:00Z9:GEAENDERT7:eintrag2:e1399:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T16:00:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":"2026-10-21T09:00:00Z","begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null,"importiertAm":"2026-10-20T07:30:00Z"}64:95146a2017725f7122a29f497d6693e66bb89956eb060dab075cf3d4526a68db1:04:true
  ```

- SHA-256 (B15):

  ```
  618e031247f649b8ddfa7929bb036558f157cae6af5d56651eddff34e1a471e6
  ```

- bash (B15):

  ```
  printf '%s' '1:220:2026-10-21T09:00:00Z9:GEAENDERT7:eintrag2:e1399:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T16:00:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":"2026-10-21T09:00:00Z","begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null,"importiertAm":"2026-10-20T07:30:00Z"}64:95146a2017725f7122a29f497d6693e66bb89956eb060dab075cf3d4526a68db1:04:true' | sha256sum
  ```

- PowerShell (B15):

  ```
  [BitConverter]::ToString([System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes('1:220:2026-10-21T09:00:00Z9:GEAENDERT7:eintrag2:e1399:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T16:00:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":"2026-10-21T09:00:00Z","begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null,"importiertAm":"2026-10-20T07:30:00Z"}64:95146a2017725f7122a29f497d6693e66bb89956eb060dab075cf3d4526a68db1:04:true'))).Replace('-','').ToLower()
  ```

- Python (B15):

  ```
  hashlib.sha256('1:220:2026-10-21T09:00:00Z9:GEAENDERT7:eintrag2:e1399:{"tag":"2026-09-28","kategorieId":"korrektur","unterkategorieId":null,"beginn":"2026-09-28T14:00:00Z","ende":"2026-09-28T16:00:00Z","dauerMinuten":0,"notiz":"Schulaufgabe 10b","quelle":"MANUELL","planRef":null,"erfasstAm":"2026-09-28T16:05:00Z","geaendertAm":"2026-10-21T09:00:00Z","begruendung":null,"kategorie":"Korrektur und Bewertung","unterkategorie":null,"importiertAm":"2026-10-20T07:30:00Z"}64:95146a2017725f7122a29f497d6693e66bb89956eb060dab075cf3d4526a68db1:04:true'.encode('utf-8')).hexdigest()
  ```

### 4.8 Wertung von Unregelmäßigkeiten

Unregelmäßigkeiten, die diese Erläuterung beschreibt, gehören zu einer von zwei Stufen. Beide sind ausdrücklich kein Beleg einer Manipulation; sie zeigen, wie viel die Kette an dieser Stelle belegt.

- **Hinweis:** Abweichungen, die bei gewöhnlichem Gebrauch von WATT entstehen können. Dazu gehören ein erster Teil der Zerlegung eines Zeitsprungs außerhalb der Grenze (4.7), Vermerke uhr_verstellt durch eine falsche Netzzeit, eine nach langer Zeit ohne Netz korrigierte Uhr, eine von Hand gestellte Uhrzeit nach einer Reise oder das Fortsetzen eines Emulator-Abbilds (4.7), Zeitsprünge über einen Neustart hinweg sowie Rücksprünge bis 2 Minuten und eine vorgestellte Uhr ohne Vermerk bei Gliedern ohne Uhrstand (9.), außerdem eine Datei, in der „Importiert am“ fehlt, obwohl das Protokoll das Einlesen belegt (bis WATT 0.8 Cherimoya verlor eine bearbeitete eingelesene Abwesenheit den Vermerk, 7.1); WATT ergänzt ihn beim Einspielen. Ein abweichender Zeitpunkt bleibt eine Datenabweichung (7.).
- **Nicht bestätigbar:** Stellen, die WATT so nie schreibt. Dazu gehören ein uhr_ruecksprung, der nicht zur Regel zum Bezug passt (4.7), ein Eintrag ohne Uhrstand nach einem Eintrag mit Uhrstand (4.2; WATT nennt ihn mit seiner Nummer, die Kette bleibt intakt), mehrere Signaturen zu einem Eintrag und ein fehlender Schlüssel in kettenschluessel (4.7), ein Zeitpunkt mehr als 2 Minuten vor dem des Vorgängers, den kein Vermerk zur Uhr erklärt (8., Schritt 5), und eine Verstellung über 2 Minuten zwischen zwei Gliedern mit Bezug ohne uhr_verstellt. Die Herkunft solcher Stellen lässt sich aus der Kette nicht bestätigen; sie brauchen eine Erklärung von außen.

Davon getrennt sind ein Bruch der Hash-Kette (4.2), Daten, die nicht zum Protokoll passen (7.), und widerlegte Signaturen (4.7): Sie zeigen, dass Glieder oder Daten nachträglich verändert wurden, beschädigt sind oder nicht aus WATT stammen. Ein Zertifikat auf Googles Sperrliste gehört nicht dazu: Die Liste nennt keinen Zeitpunkt der Sperre; ob das Zertifikat beim Signieren schon gesperrt war, lässt sich nicht feststellen. Der Abschnitt dieses Schlüssels ist deshalb nicht bestätigbar (Befund nach 4.7), nicht widerlegt.

## 5. Die Nachweiskette

Überlastungsanzeigen und Gesprächsnotizen werden in einer eigenen Kette geführt, getrennt von der Arbeitszeitkette.

- Einträge entstehen, wenn eine Anzeige als gesendet markiert, eine Gesprächsnotiz gespeichert, ein Nachtrag dazu erstellt, ein Nachweis gelöscht oder eine Sicherung wiederhergestellt wird.
- Ein Eintrag enthält nur Art, Zeitpunkt, Referenz und die Prüfsumme des Inhalts, nicht den Inhalt selbst.
- Nicht gesendete Entwürfe erzeugen keinen Eintrag und werden beim Löschen vollständig entfernt.
- Gesendete Anzeigen und Gesprächsnotizen lassen sich löschen; ihr Inhalt verschwindet dabei, der Vermerk ihrer Existenz bleibt in der Kette.
- Gesprächsnotizen sind nach dem Speichern unveränderlich; Korrekturen erfolgen nur als Nachtrag mit eigenem Eintrag.
- Die Nachweiskette trägt keine Vermerke zur Geräteuhr. Ein Rücksprung der Uhr beim Festhalten eines Nachweises wird in der Arbeitszeitkette vermerkt und zählt in den Prüfvermerken mit: mit Anker (4.7) als uhr_verstellt mit objektId nachweis, nur ohne Anker als uhr_ruecksprung mit objektId nachweis (4.1). Mit Anker vergleicht WATT dabei nicht mit dem letzten Glied der Nachweiskette; ein Zeitpunkt der Nachweiskette vor ihrem Vorgänger ist dann nur über uhr_verstellt in der Arbeitszeitkette erklärt.
- Eine Wiederherstellung ersetzt die Nachweiskette des Geräts durch die der Datei. Anzahl und letzte Prüfsumme der ersetzten Kette stehen ab Formatversion 18 im Eintrag wiederherstellung der Arbeitszeitkette (nachweisketteVorherAnzahl, nachweisketteVorherLetzterHash); die Nachweiskette selbst enthält weiter nur Art, Zeitpunkt, Referenz und Prüfsumme, und Arbeitszeitberichte nennen die Angaben nicht. Eine vollständige Datei ab Formatversion 4 ohne Nachweise spielt WATT nicht ein.

### 5.1 Prüfsummen der Nachweiskette und der Dokumente

- Inhaltsprüfsumme einer Anzeige bzw. Gesprächsnotiz: SHA-256 über die Felder unten, jedes als \<Länge>:\<Wert> wie in 4.2 und ohne Trennzeichen aneinandergehängt; Zeitpunkte und Darstellung wie in 4.2, Tage als JJJJ-MM-TT, leere oder fehlende Werte als 0:. Formel: SHA-256( L(Feld 1) ‖ L(Feld 2) ‖ … ).
- Prüfsumme eines Dokuments: SHA-256 über die unveränderten Bytes der PDF-Datei, wie in der Datei SHA256SUMS.txt im ZIP des Aktenauszugs (je Zeile Prüfsumme, zwei Leerzeichen, Pfad). Nachrechnen mit sha256sum datei.pdf, Get-FileHash datei.pdf -Algorithm SHA256 oder hashlib.sha256(open('datei.pdf','rb').read()).hexdigest().
- Anhänge an einer Anzeige (fremde PDFs und Fotos, die die Lehrkraft zur Anzeige ablegt, ab Formatversion 19) sind kein Teil des Nachweises: Sie gehen weder in die Inhaltsprüfsumme der Anzeige noch in die Nachweiskette ein, ändern also keinen Ketteneintrag. In die Prüfsumme des Datenbestands beim Neubeginn (4.6) gehen von jedem Anhang Name, Art, Größe und Datum ein, nicht der Inhalt. Die verschlüsselte Sicherung enthält die Inhalte vollständig als eigene verschlüsselte Teile hinter dem Hauptteil (Container der Version 3, ab Formatversion 19; Aufbau in der Beschreibung des Datenformats, doc/datenformat.md in watt_public): jeder Teil mit eigenem IV unter demselben Sicherungsschlüssel, als zusätzliche Daten der ganze Kopf, der Text WATT-Teil, die Nummer und die Länge des Teils, sodass ein vertauschter, fremder oder veränderter Teil am Prüfwert scheitert; die Datei endet genau hinter dem letzten Teil, der Klartext-Export nur Name, Art, Größe und Datum (mit „Externe Anhänge mitspeichern“ als unverschlüsselte Kopien in einer ZIP-Datei). Fehlt der Inhalt eines Anhangs in einer Sicherung (Fassung für die Cloud, wenn die vollständige Sicherung 20 MB oder mehr hätte), bleiben seine Angaben beim Einspielen erhalten; ein gleicher Anhang, der auf dem Gerät schon liegt, bleibt mit Inhalt.
- Ketteneintrag der Nachweiskette: hash(n) = SHA-256( L(seq) ‖ L(zeitpunkt) ‖ L(ereignis) ‖ L(referenzId) ‖ L(pruefsumme) ‖ L(prev) ), Darstellung, prev und Startwert 0 wie in 4.2. ereignis ist ANZEIGE_GESENDET, ANZEIGE_GELOESCHT, GESPRAECHSNOTIZ_GESPEICHERT, NACHTRAG, GESPRAECHSNOTIZ_GELOESCHT, WIEDERHERSTELLUNG oder WIEDERHERSTELLUNG_UNTERBROCHEN; referenzId ist die Kennung der Anzeige, der Gesprächsnotiz oder des Nachtrags (\<Kennung der Notiz>:\<Kennung>), bei einer Wiederherstellung der Zeitpunkt der Sicherung; pruefsumme ist die Inhaltsprüfsumme, beim Löschvermerk die beim Eintragen, bei einer Wiederherstellung die SHA-256 der eingespielten Datei.

Felder einer Anzeige (Verfahren 2, in allen ausgelieferten Fassungen ab Formatversion 16), in dieser Reihenfolge:

- die Ziffer 2 (Verfahren)
- Kennung, Inhaltsstufe (AUSSAGE, ZUSAMMENFASSUNG, NACHWEIS), erster und letzter Tag des Zeitraums (leer ohne Zeitraum), Herkunft der Angaben (PLATZHALTER, EINMALIG, MEINE_DATEN), „nur für dieses Schreiben“ (true oder false), vorgesehene Empfänger-E-Mail
- Anzahl der Kopie-Empfänger:innen, danach je Empfänger:in die Stelle (PERSONALRAT, SCHWERBEHINDERTENVERTRETUNG, GLEICHSTELLUNGSBEAUFTRAGTE) und die E-Mail
- Betreff, gespeicherte Fassung des Anschreibens einschließlich Prüfvermerk, Datum des Schreibens
- Empfänger:in beim Versand und Sendezeitpunkt (leer, wenn nicht angegeben)
- Anzahl der Dokumente, danach je Dokument Art (ANSCHREIBEN, ZUSAMMENFASSUNG, DETAILNACHWEIS), Dateiname und SHA-256 der Datei
- Anzahl der Notizen mit fest true, danach je solche Notiz Zeitpunkt (erstelltAm) und Text, sortiert nach erstelltAm und Kennung wie im Datenformat. fest true tragen genau die Notizen, die beim Markieren als gesendet vorhanden waren; Notizen mit fest false (nach dem Markieren angelegt, nachVersand true) gehen nicht ein und zählen nicht mit
- Kopf der Arbeitszeitkette beim Erzeugen: Anzahl der Einträge, Prüfsumme und Zeitpunkt (alle drei leer, wenn er fehlt)

Anzeigen aus Vorabversionen vor Formatversion 16 tragen Verfahren 1: ohne die Ziffer am Anfang und ohne Kettenkopf; die Kopie-Empfänger:innen stehen dort in einem einzigen Feld, je Empfänger:in \<STELLE>=\<E-Mail>, mit LF verbunden. Das Verfahren steht je Anzeige gespeichert (pruefverfahren im Datenformat). Der Erzeugungszeitpunkt und die Wiedervorlage gehören in keinem Verfahren zur Prüfsumme.

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

- **Überlastungsanzeige:** Prüfvermerk mit WATT-Version, Anzahl der Protokolleinträge, Zeitpunkt und vollständiger Prüfsumme des Kettenkopfs zum Zeitpunkt der Erzeugung: im Mailtext und in der gespeicherten Fassung des Anschreibens als letzte Zeile, davor die übliche Trennzeile vor einer E-Mail-Signatur, „-- “ (zwei Bindestriche und ein Leerzeichen), auf eigener Zeile; sie gehört wie der übrige Text zur Prüfsumme der Anzeige, früher gespeicherte Anzeigen ohne sie werden gegen ihren gespeicherten Text geprüft; in den PDFs von Anschreiben und Anhängen klein und grau am Fuß jeder Seite (6.2).
- **Berichte und Exporte (PDF, Excel, Kalender):** Prüfvermerk zum Protokollstand (6.2) und Kettenvermerk mit der Anzahl der Einträge und der Anzahl erkannter Verstellungen, Rücksprünge und Abweichungen der Geräteuhr (uhr_verstellt, uhr_ruecksprung und uhr_abweichung zusammen), bei eingelesenen Buchungen mit der Anzahl der aus einer Datei eingelesenen Einträge und Abwesenheiten, nach einem Neubeginn mit dessen Zeitpunkt; im PDF am Fuß jeder Seite, in Excel als eigene Zeile, im Kalender (.ics) im Feld X-WR-CALDESC; CSV-Dateien tragen keinen Prüfvermerk.
- **Aktenauszug:** Prüfvermerke je Eintrag und zur Nachweiskette mit der vollständigen Prüfsumme ihres letzten Glieds, dazu am Fuß jeder Seite der Prüfvermerk des Änderungsprotokolls der Arbeitszeiten (6.2), beide Prüfsummen in fester Zeichenbreite; optional ZIP-Archiv mit den unveränderten Originaldokumenten (Ordner Eintrag_NN je Eintrag), der Technischen Erläuterung als PDF, der Anleitung PRUEFEN.txt zur unabhängigen Prüfung mit WATTest (ab Formatversion 19; Text in UTF-8 mit Windows-Zeilenenden) und der Liste SHA256SUMS.txt mit den SHA-256-Prüfsummen all dieser Dateien, also auch der von PRUEFEN.txt; Aufbau in 6.3.

Liegt ein solcher Anker bei einer dritten Stelle, z. B. eine Überlastungsanzeige beim Dienstherrn, lässt sich später prüfen: Enthält das Protokoll an der angegebenen Position genau diesen Kopf, sind alle Einträge bis dahin seither unverändert. Nach allem, was den Kopf so festhält (Bericht, Exporte, Klartext, Aktenauszug, Überlastungsanzeige), legt WATT bei automatischer Sicherung sofort eine Sicherung an (ab Formatversion 19), damit ein herausgegebener Anker nach einem Geräteverlust nicht auf einen Eintrag zeigt, den keine Sicherung enthält.

### 6.1 Zusammenhang Prüfvermerk und Protokoll

Die Anzahl im Prüfvermerk einer Überlastungsanzeige ist die laufende Nummer (seq) des jüngsten Protokolleintrags beim Erzeugen; weil die Kette lückenlos bei 1 beginnt, ist sie zugleich die Zahl der Einträge. Die Prüfsumme ist dessen hash nach 4.2, der Stand sein Zeitpunkt in deutscher Zeit (Europe/Berlin), auf Minuten. Bei leerem Protokoll nennt der Vermerk 0 Einträge, die Prüfsumme 0 und als Stand den Zeitpunkt der Erzeugung. Zum Prüfen im Klartext-Export den Eintrag mit seq gleich der Anzahl suchen: Sein hash muss der Prüfsumme gleichen und sein zeitpunkt dem Stand. Nach einem Neubeginn bezieht sich der Vermerk auf die neue Kette. Der Kopf steht zusätzlich mit der Anzeige gespeichert (kettenkopf im Datenformat) und geht in deren Prüfsumme ein (5.1).

### 6.2 Wortlaut, Fußzeile und QR-Code des Prüfvermerks

Der Prüfvermerk lautet: „Prüfvermerk WATT \<Version>: Arbeitszeitprotokoll mit \<Anzahl> Einträgen, Stand \<TT.MM.JJJJ>, \<HH:MM> Uhr, SHA-256:“, darunter die Prüfsumme (64 Zeichen, feste Zeichenbreite) als eigene Zeile, darunter: „Mit dieser Prüfsumme lässt sich später prüfen, ob das Arbeitszeitprotokoll bis zu diesem Stand unverändert geblieben ist. Nachträge und Änderungen sind im Protokoll selbst vermerkt.“ Darauf folgt der Kettenvermerk („Änderungsprotokoll mit SHA-256-Hash-Kette: geprüft und intakt. Anzahl der Glieder: \<Anzahl>.“ bzw. „Änderungsprotokoll mit SHA-256-Hash-Kette: unterbrochen bei Glied Nr. \<Nummer>. Die Daten wurden außerhalb von WATT verändert oder sind beschädigt.“ bzw., wenn die gespeicherten Daten nicht zum Protokoll passen, „Änderungsprotokoll mit SHA-256-Hash-Kette: geprüft, Anzahl der Glieder: \<Anzahl>. Die gespeicherten Daten passen nicht zum Protokoll, sie wurden außerhalb von WATT verändert oder sind beschädigt. Anzahl der Abweichungen: \<Anzahl>.“, die Anzahl erkannter Verstellungen, Rücksprünge und Abweichungen der Geräteuhr, gegebenenfalls „Aus einer Datei eingelesene Einträge und Abwesenheiten: \<Anzahl>.“ – gezählt werden die Anlage-Einträge (aktion ANGELEGT) der Arten eintrag und abwesenheit, deren daten importDatei enthalten (4.1); in Bericht und Anlagen folgt „Davon im Zeitraum dieses Dokuments: \<Anzahl>.“ –, gegebenenfalls der Neubeginn des Protokolls und jede Wiederherstellung seit Beginn der Kette: „Wiederherstellungen seit Beginn der Kette: \<TT.MM.JJJJ> aus einer Sicherung (Vorgeschichte übereinstimmend); \<TT.MM.JJJJ> aus einem Klartext-Export (abweichende Vorgeschichte).“ mit „Vorgeschichte nicht abgleichbar“ bzw. „Vorgeschichte nicht abgeglichen“ für Wiederherstellungen ohne Abgleich). Im Text des Anschreibens und der Mail folgt derselbe Satz zu den Wiederherstellungen auf den Prüfvermerk. In PDFs stehen beide nur in der Fußzeile, einmal je Seite, klein und grau; im Text der Dokumente erscheinen sie nicht.

Auf der letzten Seite jedes PDFs (Anschreiben, Zusammenfassung, Einzelnachweis, Bericht, Aktenauszug) steht neben dem Vermerk ein QR-Code mit etwa 2 cm Kantenlänge (Fehlerkorrektur M, Ruhezone 4 Module). Sein Inhalt ist eine Zeile ohne persönliche Daten: watt-pruef:1;v=\<Version>;n=\<Anzahl>;t=\<JJJJ-MM-TT>T\<HH:MM>;sha256=\<64 Zeichen> und bei erkannten Verstellungen, Rücksprüngen oder Abweichungen der Geräteuhr zusätzlich ;r=\<Anzahl>, nach Wiederherstellungen ;w=\<Anzahl> und, wenn nicht jede eine übereinstimmende Vorgeschichte hat, ;wa=\<Anzahl ohne übereinstimmende Vorgeschichte>, bei eingelesenen Buchungen ;i=\<Anzahl> und, wenn der festgehaltene Kopf signiert ist, ;s=1 (ab Formatversion 19, 4.7; ohne Fingerabdruck des Schlüssels). Version, Anzahl, Zeitpunkt (deutsche Zeit, auf Minuten) und Prüfsumme sind dieselben wie im Prüfvermerk; der Code lässt sich mit jedem QR-Leser lesen und wie der Vermerk nach 6.1 gegen das Protokoll prüfen. Bei leerem Protokoll gibt es keinen Code. Dasselbe Format beschreibt das Datenformat (doc/datenformat.md in watt_public).

### 6.3 Aufbau des ZIP-Archivs zum Aktenauszug

Das ZIP-Archiv entsteht auf Wunsch zusammen mit dem Aktenauszug; der Aktenauszug selbst (PDF) liegt nicht darin. Alle Einträge des Archivs tragen als Änderungszeit den Zeitpunkt der Erstellung nach der Geräteuhr. Das Archiv enthält in dieser Reihenfolge:

1. Originaldokumente: Die Einträge des Aktenauszugs stehen in derselben Reihenfolge wie im Auszug, chronologisch nach ihrem Zeitpunkt, bei gleichem Zeitpunkt nach ihrer Kennung (Zeitpunkt einer Überlastungsanzeige: ihre Erzeugung, ohne diese der Versand, ohne beides 0:00 Uhr deutscher Zeit an ihrem Datum; Zeitpunkt einer Gesprächsnotiz: der des Gesprächs). Sie sind ab 1 durchnummeriert wie im Auszug. Die Dokumente eines Eintrags liegen im Ordner Eintrag_NN, NN die Nummer mit mindestens zwei Ziffern (Eintrag_01, Eintrag_02, …, ab 100 dreistellig). Dokumente haben nur Überlastungsanzeigen: ihre gespeicherten PDFs (Anschreiben, Zusammenfassung, Einzelnachweis, soweit erzeugt) Byte für Byte unverändert, in der Reihenfolge, in der WATT sie gespeichert hat. Gesprächsnotizen und Anzeigen ohne Dokument erhalten keinen Ordner, behalten aber ihre Nummer; Nummern können im Archiv also fehlen. Fremde Anhänge an Anzeigen (PDFs und Fotos) liegen nicht im Archiv. Der Aktenauszug nennt bei jedem Dokument denselben Pfad und dieselbe Prüfsumme.
2. Technische-Erlaeuterung.pdf: diese Erläuterung als PDF, in der Wurzel des Archivs.
3. PRUEFEN.txt: die Anleitung zur unabhängigen Prüfung mit WATTest (ab Formatversion 19), in der Wurzel, UTF-8 ohne BOM, Zeilenende CRLF (jedes Zeilenende der Vorlage, LF oder CRLF, wird zu CRLF).
4. SHA256SUMS.txt: als letzte Datei in der Wurzel, UTF-8 ohne BOM. Je Datei der Punkte 1 bis 3 eine Zeile in derselben Reihenfolge wie im Archiv: SHA-256 über die Bytes der Datei im Archiv als 64 Zeichen Hexadezimal in Kleinbuchstaben, zwei Leerzeichen, der Pfad im Archiv mit / als Trenner (z. B. Eintrag_03/Anschreiben.pdf), Zeilenende LF, auch nach der letzten Zeile. Das ist das Format von sha256sum; nach dem Entpacken prüft sha256sum -c SHA256SUMS.txt alle Dateien. SHA256SUMS.txt selbst steht nicht darin.

Dateiname eines Dokuments: Grundlage ist der gespeicherte Dateiname (dateiname im Datenformat). WATT bereinigt ihn in dieser Reihenfolge, damit ein Name aus einer eingespielten Datei beim Entpacken nie außerhalb seines Ordners schreibt und auch unter Windows entpackbar ist (kein Leerraum und kein Punkt am Anfang oder am Ende des Namens, die Endung bleibt erhalten):

1. Jedes der Zeichen \\ / : * ? " < > | und jedes Steuerzeichen (U+0000 bis U+001F und U+007F) wird zu _.
2. Jedes „..“ wird zu _, von links nach rechts ohne Überlappung (aus „...“ wird zunächst „_.“ und nach Schritt 6 „_“, aus „..\\x.pdf“ wird „__x.pdf“).
3. Am Anfang entfallen Punkte und Leerraum, bis dort keines von beiden mehr steht (jedes Zeichen, das Unicode als Leerraum führt, auch das geschützte Leerzeichen; aus „. x.pdf“ wird „x.pdf“).
4. Die Endung ist der Teil ab dem letzten Punkt, wenn dieser Punkt nicht am Anfang steht, nach ihm mindestens ein Zeichen folgt, die Endung mit Punkt höchstens 10 Zeichen hat und keinen Leerraum enthält; sonst hat der Name keine Endung („Bericht v1.2 final“ hat keine, weil nach dem letzten Punkt ein Leerzeichen steht).
5. Der Name ohne Endung wird so gekürzt, dass der ganze Name höchstens 120 Zeichen hat, gezählt in UTF-16-Codeeinheiten (ein Zeichen außerhalb der Basisebene zählt doppelt; bleibt am Ende nur die erste seiner beiden Codeeinheiten, entfällt es ganz). Ein Name von 150 Zeichen mit Endung .pdf besteht danach aus 116 Zeichen und .pdf.
6. Am Ende des Namens ohne Endung entfallen Leerraum und Punkte, bis dort keines von beiden mehr steht; endet der gekürzte Name also auf ein Leerzeichen oder einen Punkt, entfällt dieses Zeichen, die Endung folgt unmittelbar.
7. Ist der Name ohne Endung danach leer, heißt er dokument, mit der Endung, falls es eine gibt.

Gleiche Namen in einem Ordner: Ergibt die Bereinigung für ein Dokument einen Namen, den ein früheres Dokument desselben Eintrags (in der Reihenfolge oben) schon trägt, setzt WATT vor die Endung ein Leerzeichen und „(n)“ mit der kleinsten Zahl n ab 2, die einen im Ordner noch freien Namen ergibt; Groß- und Kleinschreibung zählen. Die Endung ist dieselbe wie in Schritt 4; ohne Endung steht „ (n)“ am Ende. Aus zwei Dokumenten Anschreiben.pdf werden so Eintrag_03/Anschreiben.pdf und Eintrag_03/Anschreiben (2).pdf; ein Name, der schon „ (2)“ trägt und vergeben ist, erhält ein weiteres „ (2)“ bzw. die nächste freie Zahl. Die Kürzung auf 120 Zeichen gilt vor diesem Zusatz. Der Aktenauszug nennt denselben Pfad.

## 7. Sicherung und Wiederherstellung

- Sicherungsdateien sind verschlüsselt (AES-GCM, Schlüssel über Passphrase oder Notfallblatt) und enthalten beide Ketten vollständig.
- Beim Wiederherstellen prüft WATT beide Ketten und zeigt das Ergebnis an („intakt“ bzw. „unterbrochen bei …“). Ein Bruch wird dauerhaft vermerkt und in Prüfvermerken ausgewiesen.
- Abgleich mit dem Protokoll der Datei (Sicherung und vollständiger Klartext): WATT spielt die Arbeitszeitkette der Datei Eintrag für Eintrag nach (Anlage, Änderung samt Umwandlung, Löschung, eingelesene Buchungen) und vergleicht den entstehenden Stand aller Arbeitszeit-Einträge und Abwesenheiten mit Gutschrift mit den Daten der Datei, Feld für Feld, soweit das Protokoll sie trägt (Tag, Tätigkeit, Beginn, Ende, Dauer, Notiz, Quelle, Bezug zum Stundenplan, Erfassungs- und Änderungszeitpunkt, Fach, Klasse bzw. Art, Beginn und Ende, dazu „Importiert am“ nach 7.1, ab Formatversion 19 aus dem Feld importiertAm der Änderungs-Einträge). Weicht etwas ab, lehnt die Vorschau die Datei ab („Die Daten passen nicht zum Änderungsprotokoll.“) und nennt die abweichenden Einträge; ein Einspielen trotzdem gibt es nicht. Nach einem Neubeginn ist der Bestand im Start-Eintrag (4.6) der Ausgangsstand: Ein Eintrag ohne Protokolleintrag seither muss mit Kennung und Bestandsprüfsumme darin stehen, und kein unveränderter Eintrag des Bestands darf fehlen; ebenso Abwesenheiten mit Tag und Bestandsprüfsumme. Bei Start-Einträgen vor Formatversion 18 ohne Bestand gilt ein Eintrag ohne Protokolleintrag seither als Bestand zum Neubeginn, wenn er davor erfasst und zuletzt geändert wurde; sein Inhalt ist dann nur über die Prüfsumme des Neubeginns belegt. Dateien der Formatversionen vor 16 lassen sich nicht abgleichen (ihre Einträge tragen nicht sicher den vollen Stand, keine ausgelieferte Fassung schrieb sie) und werden nicht mehr eingespielt.
- Signaturen der Datei (ab Formatversion 19): Vor dem Einspielen prüft WATT im Hintergrund jede mitgelieferte Signatur gegen den öffentlichen Schlüssel aus den Zertifikaten der Datei und die Prüfungen ohne Netz je Schlüssel (4.7) und zeigt den Befund (alle gültig, ungültig ab Eintrag n, widerlegt, keine, Prüfung nicht möglich) mit dem Satz, dass die Zertifikate bis zu Google erst eine unabhängige Prüfung prüft. Ungültige oder widerlegte Signaturen verhindern das Einspielen nicht; der Befund steht dauerhaft im Eintrag wiederherstellung (4.1, 4.7). Schlüssel und Signaturen bleiben erhalten, WATT signiert eingespielte Einträge nicht nach und legt danach immer einen neuen Schlüssel an (4.7). Ab Formatversion 19 ist eine Datei ungültig, wenn ein Zertifikat nicht genau ein DER-Element ist, eine Grenze für Schlüssel, Zertifikate oder Signaturen überschritten ist oder eine Signatur zu einem Eintrag außerhalb ihrer Kette gehört (Datenformat). Nach dem Einspielen beginnt die Prüfung unter „Integrität“ von vorn.
- Derselbe Abgleich läuft seit dem 19.10.2026 auch auf dem Gerät: unter „Mehr → Integrität → Protokolle prüfen“ (eigene Zeile im Befund) und für den Kettenvermerk jedes Prüfvermerks. Passen die gespeicherten Daten nicht zum Protokoll, etwa nach einem Eingriff in die Datenbank, steht dort statt „intakt“, dass sie nicht passen, mit der Zahl der Abweichungen.
- Abgleich der Vorgeschichte: Liegt auf dem Gerät eine Kette, muss sie Glied für Glied der Anfang der eingelesenen sein oder umgekehrt. Sonst wird die Datei trotzdem eingespielt, der Eintrag wiederherstellung trägt aber dauerhaft „abweichende Vorgeschichte“; ohne Kette auf dem Gerät „Vorgeschichte nicht abgleichbar“ (4.1). Beides steht in jedem späteren Prüfvermerk (6.2).
- Die Wiederherstellung selbst wird in der Nachweiskette mit der Prüfsumme der Sicherungsdatei vermerkt und im Änderungsprotokoll mit einem Eintrag wiederherstellung, der Anzahl und letzte Prüfsumme des vorher vorhandenen Protokolls und der vorher vorhandenen Nachweiskette nennt (5.); betroffene Anzeigen und Gesprächsnotizen tragen den Hinweis „Wiederhergestellt am … aus Backup vom …“.

### 7.1 Buchungsdaten eines anderen Geräts

Eine Datei mit Buchungsdaten (Klartext mit umfang BUCHUNGEN, nur Einträge, Abwesenheiten, Tagesmarkierungen und Ferien eines Zeitraums, ohne Protokoll) ersetzt nichts, sondern ergänzt, alles oder nichts: Übernommen werden nur Einträge und Abwesenheiten. Tätigkeiten ordnet WATT ausschließlich über den angezeigten Namen von Ober- und Unterkategorie zu; fehlt eine, wird nichts übernommen. Gleiche Einträge (gleicher Beginn, gleiches Ende, gleiche Tätigkeit) und gleiche Abwesenheiten werden übersprungen; Überschneidungen und mehr Zeit, als ein Tag lang ist, verhindern die Übernahme ganz. Jede Übernahme erzeugt die Einträge nach 4.1 mit der Prüfsumme der Datei; mit ihr lässt sich die eingelesene Datei später eindeutig zuordnen. Ab Formatversion 18 prüft WATT vor der Übernahme auch die Zeitpunkte: Einträge in der Zukunft (Abwesenheiten dürfen vorausgeplant sein), ein Erfassungszeitpunkt vor dem Beginn eines Timer-Laufs oder nach dem Export der Datei, ein Tag, der nicht zum Beginn passt, und ein Export mehr als eine Stunde in der Zukunft verhindern die Übernahme. Einträge aus Stundenplan oder Kalenderlink des anderen Geräts behalten ihre Herkunft, der Bezug auf dessen Stundenplan entfällt. Erfassungs- und Änderungszeitpunkt eingelesener Einträge stammen aus der Datei und lassen sich nicht prüfen; WATT weist sie als „laut Datei“ aus und kennzeichnet zusätzlich den Abstand zwischen Ende der Tätigkeit und Einlesen nach der Nachtrag-Regel („Eingelesen ca. 36 Tage nach Ende“). „Importiert am“ ist der Zeitpunkt des Anlage-Eintrags im Protokoll; beim Einspielen eines Klartexts vergleicht der Abgleich (7.) ihn mit, sodass sich die Markierung nicht entfernen lässt, ab Formatversion 19 auch nach einem Neubeginn: Die Bestandsprüfsummen des Start-Eintrags enthalten ihn (Form 2, 4.6). Er bleibt beim Ändern erhalten, auch beim Wechsel der Art einer Abwesenheit; eine Anlage ohne importDatei beginnt ohne, auch für den Tag einer Abwesenheit nach einem Lösch-Eintrag oder nach einem dienstfreien Tag, der keinen Eintrag hat. Ab Formatversion 19 tragen Änderungs- und Lösch-Einträge mit Uhrstand (aktion GEAENDERT, GELOESCHT) den Wert selbst als importiertAm am Ende von daten (4.1, Rechenbeispiel B15): Der Abgleich nimmt ihn aus dem letzten Änderungs-Eintrag, auch nach einem Neubeginn, und fehlt das Feld, gilt kein Vermerk; so lässt sich die Markierung auch an einem nach dem Neubeginn geänderten Datensatz weder unbemerkt entfernen noch hinzufügen. Änderungs-Einträge ohne Uhrstand (bis Formatversion 18) erben ihn vom vorigen Stand; ändert ein solcher Eintrag nach einem Neubeginn einen Datensatz des Bestands, belegt das Protokoll den geerbten Wert nicht, weil der Bestand nur Prüfsummen trägt; dann gilt der Wert der Datei (9.). Fehlt „Importiert am“ in der Datei, obwohl das Protokoll das Einlesen belegt (bis zum 26.10.2026 verlor eine bearbeitete eingelesene Abwesenheit den Vermerk), ergänzt WATT ihn beim Einspielen aus dem Protokoll, ebenso einmalig in den Daten des Geräts, ohne Eintrag im Protokoll; ein abweichender Zeitpunkt bleibt eine Abweichung. Bei Start-Einträgen in Form 1 ist er für Einträge ohne Protokolleintrag seit dem Neubeginn nicht belegt. Der Einzelnachweis einer Überlastungsanzeige nennt eingelesene Einträge an der Herkunft („Timer (eingelesen)“), der QR-Code des Prüfvermerks ihre Anzahl (i=…), der Kettenvermerk zusätzlich die Anzahl im Zeitraum des Dokuments.

## 8. Prüfverfahren

1. Klartext-Export bzw. Aktenauszug mit ZIP von der betroffenen Person anfordern, dazu vorhandene Anker (Überlastungsanzeigen, Berichte). Den Klartext-Export gibt es als JSON (JSON Schema ab Formatversion 16) und als XML: Ausgelieferte Fassungen exportieren XML ab Formatversion 18 (WATT 0.8.109 „Cherimoya“), das XML Schema ist ab Version 18 veröffentlicht. Beide enthalten dieselben Angaben; die Prüfsumme des Bestands (4.6) lässt sich nur aus JSON nachrechnen. Datenformat und Schemas sind veröffentlicht: https://github.com/Aves83/watt_public (doc/datenformat.md, Ordner schema).
2. Das eigene Werkzeug zuerst mit den Rechenbeispielen aus 4.2, 4.6, 4.7 und 5.1 abgleichen.
3. Für jeden Eintrag die Prüfsumme nach 4.2 neu berechnen und mit der gespeicherten vergleichen; die Verkettung über die Vorgänger-Prüfsumme prüfen. Die Prüfung ist auch in WATT selbst möglich (unter „Mehr → Integrität“: beide Ketten mit Befund, Anzahl der Glieder, Stand, Prüfsumme des letzten Glieds und erkannten Auffälligkeiten der Geräteuhr, die Signaturen je Schlüsselabschnitt sowie die letzten Änderungen). Klartext-Exporte und Aktenauszüge prüft außerdem die quelloffene Prüf-App WATTest, ohne Code von WATT allein nach den veröffentlichten Unterlagen entwickelt; Prüfende können ihren Quelltext einsehen oder ein eigenes Werkzeug verwenden: https://aves83.github.io/wattest/ (Quelltext https://github.com/Aves83/wattest, Lizenz EUPL-1.2). Sie läuft im Browser; die geprüften Dateien verlassen den Rechner nicht. Ab Formatversion 19 verweist WATT darauf unter „Mehr → Integrität → Unabhängig prüfen“ (WATTest im Browser und Einzeldatei mit Prüfsumme unter https://github.com/Aves83/wattest/releases; die Links öffnen den Browser, WATT selbst greift dafür nicht auf das Netz zu) und in PRUEFEN.txt im ZIP des Aktenauszugs (6.3).
4. Laufende Nummern auf Lücken prüfen; bei Einträgen mit Uhrstand Neustarts und automatische Uhrzeit auswerten (4.7).
5. Zeitstempel: Steigen sie mit der laufenden Nummer monoton an, und sind sie plausibel? Liegt der Zeitpunkt eines Eintrags mehr als 2 Minuten vor dem seines Vorgängers, muss das erklärt sein, sonst ist die Stelle „nicht bestätigbar“ (4.8): Entweder folgt unmittelbar danach ein Eintrag uhr_ruecksprung mit beiden Zeitpunkten (4.1), oder der Eintrag ist selbst ein Vermerk zur Uhr, der seinen eigenen Rücksprung erklärt, gleich mit welcher objektId: uhr_ruecksprung mit objektId timer (Timer mit zurückgestellter Uhr), nachweis (Festhalten eines Nachweises mit zurückgestellter Uhr), start (Öffnen der App oder einer Seite) oder zeitsignal (Android meldet eine Änderung der Uhrzeit) oder einer laufenden Nummer, oder uhr_verstellt mit richtung zurueck (4.7). Einen Sprung vor oder zurück bis zu einem Eintrag uhr_verstellt erklärt die Rechnung in 4.7 aus vorher und nachher; der Zeitpunkt von uhr_verstellt liegt nach der Verstellung. Einträge uhr_abweichung zeigen, dass der Beginn eines Timer-Laufs nach der Monotonuhr korrigiert wurde oder das Gerät während des Laufs neu gestartet wurde (4.1); die Prüfvermerke von Berichten und Aktenauszug nennen die Anzahl aller drei Arten zusammen. Rücksprünge bis 2 Minuten, vorgestellte Uhren und eine durchgehend falsch gehende Uhr vermerkt WATT nicht; die Zeitstempel stammen unverändert von der Uhr des Geräts. Ab Formatversion 19 (4.7) steht bei Einträgen mit Uhrstand eine Verstellung in beide Richtungen als uhr_verstellt vor dem auslösenden Eintrag, hinter einem Eintrag neubeginn bzw. wiederherstellung oder an ihrem eigenen Zeitpunkt; uhr_ruecksprung folgt dann nur noch, wenn vor der Messung kein Anker bestand: Bei objektId mit laufender Nummer hat der auslösende Eintrag davor, bei timer, nachweis, start und zeitsignal der Vermerk selbst neustarts größer als 0 oder -1; einzige Ausnahme ist ein uhr_ruecksprung mit objektId nachweis als erster Eintrag einer Kette (neustarts 0, nur in Ketten von Vorabversionen). Was Anker und Bezug heißen und welche Fälle es gibt, steht in 4.7. Die Prüfvermerke zählen alle drei Arten zusammen.
6. Kopf, Anzahl und Zeitpunkt aus jedem Prüfvermerk mit dem Eintrag an derselben Position im Protokoll abgleichen.
7. Besondere Einträge auswerten: Neubeginne, Wiederherstellungen mit ihrer Vorgeschichte (4.1, 7), Löschungen mit Begründung, gekennzeichnete Nachträge und eingelesene Buchungen (importDatei, 7.1).
8. Die SHA-256-Prüfsummen aus dem ZIP mit den Originaldokumenten vergleichen (SHA256SUMS.txt, Aufbau in 6.3).
9. Schlüssel und Signaturen nach dem Prüfverfahren in 4.7 prüfen und den Befund (bestätigt, nicht bestätigbar, widerlegt) je Abschnitt festhalten; welche Einträge eine Signatur hinsichtlich der Herkunft deckt, regelt 4.7 (Abschnitt ab der letzten Wiederherstellung bzw. dem Neubeginn, frühestens ab dem ersten Schlüssel oder einem Neubeginn bzw. einer Wiederherstellung unmittelbar davor, Einträge davor nicht bestätigbar, eingespielte Einträge nur über die Signaturen der Datei, Einträge nach der letzten Signatur nicht bestätigbar).

## 9. Grenzen der Fälschungssicherheit

- **Ein früherer Eintrag wird verändert, gelöscht oder eingefügt:** erkennbar, die Kette bricht ab dieser Stelle.
- **Die letzten Einträge werden entfernt (Kette am Ende gekürzt):** nur mit Anker erkennbar; ohne einen späteren Anker ist ein kürzeres Protokoll in sich stimmig.
- **Eine vollständig neue, stimmige Kette wird erzeugt:** ohne früheren Anker nicht erkennbar. Dafür braucht es keinen besonderen Zugriff auf das Gerät: Das Verfahren ist offen beschrieben, ein Klartext-Export lässt sich bearbeiten und seine Kette mit einfachen Werkzeugen neu berechnen. Auf einem Gerät mit eigener Kette fällt das Einspielen einer solchen Datei als „abweichende Vorgeschichte“ auf; ein herausgegebener Prüfvermerk passt nicht zur neuen Kette. Erst ein Anker bei Dritten widerlegt sie sicher.
- **Daten einer Datei werden bearbeitet, die Kette bleibt:** erkennbar; WATT spielt eine solche Datei nicht ein (7.).
- **Daten in der Datenbank des Geräts werden verändert, die Kette bleibt (z. B. mit Root-Zugriff):** erkennbar; „Integrität“ und jeder Prüfvermerk gleichen die Daten mit dem Protokoll ab und nennen die Abweichung (7.).
- **Glieder werden mit Laufzeitzugriff direkt in die Datenbank geschrieben (Exploit ohne entsperrten Bootloader):** nicht durch die Signatur erkennbar. WATT signiert nach jedem Vorgang den Kopf, der gerade in der Datenbank liegt; die nächste Signatur deckt solche Glieder mit. Erkennbar bleiben sie nur über Anker (6.) und den Abgleich der Daten (7.).
- **Die Geräteuhr wird verstellt:** eingeschränkt erkennbar. Zeitstempel stammen vom Gerät; es gibt keinen unabhängigen Zeitstempeldienst, und eine von Anfang an falsch gehende Uhr fällt nie auf.
- **Geräteuhr bei Gliedern mit Uhrstand (ab Formatversion 19, 4.7):** Im selben Gerätestart fällt jede Verstellung um mehr als 2 Minuten in beide Richtungen gegen die Monotonuhr auf, beim nächsten Glied oder sofort, wenn Android die Änderung meldet, und steht als uhr_verstellt in der Kette, auch wenn die Uhr vor dem nächsten Glied wieder richtig gestellt wird. Viele kleine Schritte summieren sich am festen Anker je Gerätestart, auch mit automatischer Uhrzeit und auch, wenn Einträge dazwischen sofort wieder gelöscht werden. Ein laufender Timer wird dabei nach seiner Monotonuhr korrigiert; nach einem Neustart während des Laufs verankert WATT ihn im neuen Gerätestart neu. Über einen Neustart hinweg prüft WATT nur, ob der neue Gerätestart nach dem alten beginnt, sonst bleibt der Rückfall auf uhr_ruecksprung; die Neustarts selbst stehen in jedem Glied, ebenso ob die Uhrzeit automatisch eingestellt war. Der Neustartzähler ist ein Wert aus den Systemeinstellungen von Android, den auch die Debug-Schnittstelle (adb) setzen kann; ein so veränderter Zähler fällt auf, wenn die Uhr danach zurückgestellt ist (4.7).
- **Geräteuhr bei Gliedern ohne Uhrstand (Formatversion 18 und früher):** Ein Rücksprung um mehr als 2 Minuten fällt auf, wenn das nächste Glied, der Start eines Timers oder das Festhalten eines Nachweises noch mit der zurückgestellten Uhr entsteht; WATT vermerkt ihn als eigenes Glied uhr_ruecksprung mit beiden Zeitpunkten, weist in der App darauf hin und nennt die Anzahl in den Prüfvermerken. Wird die Uhr zurückgestellt und vor dem nächsten Glied wieder vorgestellt, fällt das nicht auf. Beim Timer prüft WATT die Wanduhr gegen die Monotonuhr des Geräts: Ein während des Laufs verstelltes Datum oder eine verstellte Uhrzeit ändern die Dauer des Laufs nicht, der Beginn wird nach der Monotonuhr korrigiert und als uhr_abweichung vermerkt (4.1); ein Neustart während des Laufs steht dort als uhr_abweichung mit art neustart. Kleinere Rücksprünge und eine vorgestellte Uhr außerhalb eines Timer-Laufs fallen nicht auf.
- **Erfundene Zeiten werden zeitnah erfasst:** nicht erkennbar. Die Kette belegt, wann etwas erfasst wurde, nicht, ob es stimmt.
- **Eine veränderte App-Version erzeugt die Daten:** für signierte Abschnitte erkennbar: Die Bestätigung von Android nennt Paketname und Signaturzertifikat der App, eine veränderte App signiert mit einem anderen Zertifikat (4.7). Einträge ohne gültige Signatur (vor Formatversion 19, auf Geräten ohne Bestätigung, nach einem Fehlschlag) bleiben ohne diesen Nachweis.
- **Die Debug-Variante der App erzeugt die Daten:** erkennbar am Zusatz „-debug“ in der Version jedes Prüfvermerks und in signierten Abschnitten zusätzlich an der Bestätigung von Android: Sie nennt den Paketnamen de.achim.watt.debug und das Debug-Signaturzertifikat statt des Release-Zertifikats (4.7, Schritt 3). In ihr lassen sich beide Ketten über das Debug-Menü löschen und Beispieldaten anlegen; Bildschirmfotos sind dort immer erlaubt.
- **Ein älterer Stand wird aus einer Sicherung eingespielt:** auf dem Gerät erkennbar; die Wiederherstellung wird in der Nachweiskette vermerkt, Anzahl und Kopf der ersetzten Nachweiskette stehen im Eintrag wiederherstellung (5.), und vorhandene Anker späteren Datums passen nicht zum zurückgesetzten Stand.
- **Das Protokoll wird neu begonnen:** erkennbar am Start-Eintrag; die frühere Historie ist auf dem Gerät nicht mehr prüfbar, wohl aber über früher erzeugte Anker und Sicherungen: Der Start-Eintrag nennt Anzahl, letzte Prüfsumme und Zeitpunkt der gelöschten Kette (4.6). Mit zurückgestellter Uhr lässt sich nicht neu beginnen; WATT lehnt den Neubeginn ab, wenn die Uhr mehr als 2 Minuten vor dem letzten Eintrag steht.
- **Buchungen werden aus einer Datei eingelesen:** erkennbar am Feld importDatei im Anlage-Eintrag und am Vermerk „Importiert am …“; Erfassungs- und Änderungszeitpunkt stammen aus der Datei und lassen sich nur mit deren Prüfsumme und den Ankern des anderen Geräts prüfen, nicht mit der Kette dieses Geräts.
- **„Importiert am“ wird in einer Datei entfernt oder hinzugefügt, die Kette bleibt:** erkennbar (7., 7.1), ab Formatversion 19 auch nach einem Neubeginn über die Bestandsprüfsummen in Form 2 (4.6) und das Feld importiertAm der Änderungs-Einträge (4.1). Nicht erkennbar ist es nur im Altfall: an einem Datensatz des Bestands, den nach einem Neubeginn ein Änderungs-Eintrag ohne Uhrstand (bis Formatversion 18) geändert hat, und bei Start-Einträgen in Form 1 an einem seither unveränderten Datensatz; dort gilt der Wert der Datei.

## 10. Was WATT ausdrücklich nicht leistet

- keine elektronische Signatur der Person: Die Prüfsummen belegen Unverändertheit. Die Signatur des Geräts (4.7) belegt, dass der signierte Stand in der echten App auf einem bestätigten Gerät lag, also in der mit dem Release-Schlüssel signierten App und nicht in einem Nachbau oder einer veränderten App; „entstanden“ sind dort nur die Einträge, die sie nach der Regel in 4.7 hinsichtlich der Herkunft deckt (ihr Abschnitt ab der letzten Wiederherstellung bzw. dem Neubeginn, frühestens ab dem ersten Schlüssel, ohne eingelesene Zeitpunkte und ohne den Bestand eines Neubeginns). ;s=1 im QR-Code (6.2) sagt nur, dass der festgehaltene Kopf eine rechnerisch gültige Signatur des aktiven Schlüssels trägt. Sie ist keine elektronische Signatur der Person im Sinne der eIDAS-Verordnung und belegt nicht, wer die Einträge vorgenommen hat
- kein qualifizierter Zeitstempel und keine Bestätigung der Uhrzeit durch Dritte
- keine Bestätigung von Gerät oder App durch eine unabhängige Stelle: Die Bestätigung des Signaturschlüssels (4.7) stammt von Android, also von Google und dem Gerätehersteller
- keine Aussage über die Richtigkeit der Inhalte; es handelt sich um eine Selbstaufzeichnung der Lehrkraft

Die Beweiskraft der Aufzeichnungen ergibt sich aus dem Zusammenspiel von zeitnaher Erfassung, lückenloser Kette, gekennzeichneten Nachträgen und externen Ankern, nicht aus der Technik allein.

## 11. Zugang zum Quellcode

Zum Abgleich der Aussagen dieser Erläuterung mit der tatsächlichen Umsetzung wird Gutachter:innen Zugang zum Quellcode von WATT ermöglicht. Voraussetzungen sind die Unterzeichnung einer Vertraulichkeitsvereinbarung und der Nachweis eines entsprechenden Prüfauftrags. Anfragen an: hans-joachim.steudel@tum.de.

## 12. Änderungen je Formatversion

Neuerungen sind in dieser Erläuterung an die Formatversion gebunden, mit der sie zuerst in einer ausgelieferten Fassung von WATT erschienen; die Formatversion steht in jedem Klartext-Export (version). Glieder älterer Formatversionen werden unverändert nach ihren eigenen Regeln geprüft.

```
Version  Fassung                     Neu für die Prüfung
16       WATT „Apfel“, 11.10.2026    erste ausgelieferte Fassung; Kette nur mit
                                     dem Ist (seit Version 13), Verfahren 2 der
                                     Prüfsumme von Anzeigen (5.1)
17       WATT „Bergamotte“,          Kalenderlink, Herkunft KALENDERLINK; die
         14.10.2026                  Prüfsumme des Neubeginns enthielt bei
                                     gesetztem Link den Link (4.6)
18       WATT „Cherimoya“,           XML-Export; vorgeschichte und Nachweiskette
         20.10.2026                  vorher im Glied wiederherstellung; Bestand
                                     im Glied neubeginn (4.6); Buchungsdaten mit
                                     importDatei (7.1); Uhrprüfung beim Start
                                     eines Timers und beim Festhalten eines
                                     Nachweises; uhr_abweichung (4.1)
19       WATT „D“, noch nicht        Uhrstand je Glied, uhr_verstellt mit festem
         ausgeliefert                Anker (4.7); schluessel, Signaturen,
                                     kettenschluessel und kettensignaturen; neuer
                                     Schlüssel nach jeder Wiederherstellung;
                                     Anhänge an Anzeigen, Container 3; Kommentar
                                     zur Änderung (begruendung); bestandsform 2
                                     (4.6); Grenzen beim Einlesen; kein Vermerk
                                     zur Uhr bei leerer Kette; Eintrag ohne
                                     Uhrstand nach einem mit Uhrstand nicht
                                     bestätigbar statt Bruch (4.2); fehlender
                                     Schlüssel nicht bestätigbar (4.7);
                                     importiertAm in Änderungs- und
                                     Lösch-Einträgen (4.1, 7.1); PRUEFEN.txt
                                     im ZIP des Aktenauszugs (6.3); Zertifikat
                                     auf der Sperrliste nicht bestätigbar statt
                                     widerlegt (4.7, 4.8); Herkunft erst ab dem
                                     ersten Schlüssel, Einträge davor nicht
                                     bestätigbar, Neubeginn bzw.
                                     Wiederherstellung direkt vor einem
                                     Schlüssel im Abschnitt (4.7); Aufbau des
                                     ZIPs mit Ordnern, Dateinamen, „ (n)“ bei
                                     gleichen Namen, SHA256SUMS.txt (6.3)
```
