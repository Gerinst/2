# Avis-Flow Version 2 Schritt für Schritt

Diese Anleitung baut den Flow für ein **freigegebenes Postfach** (Exchange Shared Mailbox). Jede neue Mail geht als Ganzes an das Skript: PDF-Anlagen, Excel- und CSV-Dateien, SAP-Exporte und der Mailtext. Ist die Mail ein Zahlungsavis, bekommt die **Original-Mail** die Kategorie „Avise" und den Betrag hinten an den Betreff. Andere Mails bleiben unberührt. Es kommen keine zusätzlichen Mails.

Alle Klickwege in dieser Anleitung sind am 30.10.2026 in einem Microsoft-365-Testzugang mit genau diesem Skript und diesem Flow nachgegangen worden (12 Test-Mails, 12 richtige Ergebnisse). Einen klickbaren Nachbau des Flows mit allen Feldern, Einstellungen und Formeln zum Kopieren gibt es in `dist/avis-flow.html` (herunterladen und im Browser öffnen).

## Überblick

| Ergebnis des Skripts | Kategorie | Betreff danach |
| --- | --- | --- |
| Ein Avis, Betrag sicher | Avise | `Zahlungsavis - 2.601,35 EUR` |
| Ein Avis, Betrag ziemlich sicher | Avise | `Zahlungsavis - 2.601,35 EUR (prüfen)` |
| Ein Avis, Betrag unsicher | Avis prüfen | `Zahlungsavis - 2.601,35 EUR ?` |
| Mehrere Avise in einer Mail | mehrere Avise | unverändert |
| Unklar: Avis-Hinweis, aber nicht lesbar (Scan, verschlüsselt, altes .xls) oder nicht sicher erkennbar | Avis prüfen | unverändert |
| Skript-Fehler oder Zeitüberschreitung | Avis prüfen | unverändert |
| Kein Avis (Rechnung, Reklamation, Bestellung, Info, Mahnung …) | keine | unverändert |

„Zahlungsavis" steht hier für den Betreff, den der Absender geschrieben hat. Er bleibt vorne stehen, der Betrag kommt immer im deutschen Format mit Währung dahinter.

Der Flow in Kurzform:

```
Wenn eine neue E-Mail in einem freigegebenen Postfach eingeht (V2)
└─ Keine_Automatik (Abwesenheitsnotizen und Zustellmeldungen raus)
   └─ Wahr: Export_email → Run_script → Kategorie_noetig
            └─ Wahr: Kategorie_setzen → Betreff_noetig
                     └─ Wahr: Betreff_setzen
```

**Ab 31.12.2026** will Microsoft das Ändern des Betreffs empfangener Mails sperren. Dann schlägt nur `Betreff_setzen` fehl, die Kategorie wird weiter gesetzt. Im Januar testen; was dann zu tun ist, steht unter „Ab Januar 2027: Betreff-Sperre prüfen".

## Was du brauchst

- Ein **freigegebenes Postfach** in Exchange (z. B. `avis@firma.de`). Ein Microsoft-365-Gruppenpostfach geht nicht: Dort kann der Flow weder Kategorie noch Betreff setzen.
- Ein Konto mit Microsoft-365-Business-Lizenz (Excel im Browser und Power Automate), unter dem der Flow läuft. Dieses Konto braucht **Vollzugriff** auf das Postfach.
- Die Datei `dist/AvisV2.ts` (das Skript) und die Formeln aus dem Ordner `formeln/`.
- Etwa eine Stunde Zeit.

Überall, wo `avis@firma.de` steht, die Adresse eures freigegebenen Postfachs einsetzen.

## Schritt 1: Rechte auf das freigegebene Postfach (IT, einmalig)

1. Microsoft 365 Admin Center → **Teams und Gruppen** → **Freigegebene Postfächer** → das Postfach anklicken → im Bereich **Mitglieder** auf **Bearbeiten** → **Mitglieder hinzufügen** → das Konto des Flows anhaken → **Hinzufügen**. Damit hat das Konto Vollzugriff und „Senden als". (Beim Anlegen eines neuen Postfachs bietet das Admin Center den Link „Hinzufügen von Mitgliedern zu Ihrem freigegebenen Postfach" direkt an.)
2. Prüfen (optional): Exchange Admin Center → **Empfänger** → **Postfächer** → das Postfach → Reiter **Delegierung**: Unter „Lesen und Verwalten (Vollzugriff)" steht das Konto.
3. Microsoft nennt bis zu 2 Stunden, bis neue Rechte wirken. Im Test ging es nach wenigen Minuten. Woran man es merkt: In Outlook im Web öffnet sich das Postfach unter `https://outlook.office.com/mail/avis@firma.de/`.

## Schritt 2: Kategorien im Postfach anlegen (einmalig)

1. Outlook im Web öffnen und das Postfach aufrufen: `https://outlook.office.com/mail/avis@firma.de/`.
2. Eine beliebige Mail anklicken, oben auf das **Etikett-Symbol** (Kategorisieren) → **Kategorien verwalten**.
3. **Neu erstellen** → Name `Avise` → eine Farbe (z. B. Grün) → **Speichern**.
4. Genauso `Avis prüfen` (z. B. Orange) und `mehrere Avise` (z. B. Lila) anlegen. Die Namen müssen genau so geschrieben sein, mit Leerzeichen und „ü".

Ohne diesen Schritt erscheinen die Kategorien trotzdem an den Mails, nur grau. Die Farben gelten für alle, die das Postfach öffnen.

## Schritt 3: Skript AvisV2 in Excel anlegen (einmalig)

1. Die Arbeitsmappe `AvisParser.xlsx` aus Version 1 in OneDrive öffnen. Gibt es sie noch nicht: OneDrive im Browser → **Erstellen oder hochladen** → **Excel-Arbeitsmappe** → oben auf den Namen „Mappe" klicken → `AvisParser` eintippen → Enter. Die Mappe bleibt leer.
2. Reiter **Automatisieren** → **Neues Skript** → **Im Code-Editor erstellen**. Erscheint rechts der Hinweis „Aktionen in Skripts umwandeln", auf **Skript schreiben** klicken.
3. Auf GitHub die Datei `dist/AvisV2.ts` öffnen und oben rechts auf **Copy raw file** klicken (Symbol mit den zwei Rechtecken).
4. In den Editor klicken, **Strg+A** (Mac: **⌘+A**), dann **Strg+V** (Mac: **⌘+V**). Das Skript hat etwa 9.700 Zeilen; das Einfügen dauert ein paar Sekunden. Excel rückt dabei manche Zeilen anders ein. Das ist normal und ändert nichts.
5. Auf das **Speichern-Symbol** (Diskette) klicken. Kommt der Hinweis „Zugeordnetes Skript", mit **OK** bestätigen.
6. Umbenennen: links oben auf **← Alle Skripts** → bei der Karte des neuen Skripts auf **Umbenennen** → im Feld **Name** `AvisV2` eintragen → Tab-Taste. Oben erscheint „Gespeichert ✓".

Das alte Skript `AvisPdfAmount` bleibt, wie es ist. Excel speichert Skripte in OneDrive unter **Dokumente → Office-Skripts** (`AvisV2.osts`).

**Später ein neues Skript einspielen:** Excel → Automatisieren → **Alle Skripts** → **AvisV2** → **Bearbeiten** → Strg+A, Strg+V mit dem neuen Inhalt → Speichern. Der Flow nimmt ab dem nächsten Lauf automatisch das neue Skript.

## Schritt 4: 5-Minuten-Test (empfohlen, bevor der echte Flow gebaut wird)

Damit prüfst du vorab, ob Vollzugriff und Betreffänderung im Postfach funktionieren.

1. Power Automate → **Erstellen** → **Automatisierter Cloud-Flow** → Name `PATCHTEST` → Trigger suchen: `freigegebenen Postfach` → **Wenn eine neue E-Mail in einem freigegebenen Postfach eingeht (V2)** → **Erstellen**.
2. Trigger anklicken: **Ursprüngliche Postfachadresse** = `avis@firma.de` eintippen und in der Liste **„avis@firma.de" verwenden** anklicken. **Erweiterte Parameter** → **Alle anzeigen** → **Filter für Betreff** = `PATCHTEST`.
3. **+** unter dem Trigger → Suche `HTTP-Anforderung senden Office 365 Outlook` → **HTTP-Anforderung senden** (Office 365 Outlook):
   - **URI**: ins Feld klicken → **fx** → Formel aus `formeln/URI_Kategorie_und_Betreff.txt` (mit eurer Adresse) → **Hinzufügen**
   - **Methode**: `PATCH`
   - **Erweiterte Parameter** → **Alle anzeigen** → **Text**: `{"categories":["Avise"]}`
4. Zweite gleiche Aktion, nur **Text**: `{"subject":"PATCHTEST Zahlungsavis - 2.601,35 EUR"}`.
5. **Speichern**. Aus dem eigenen Postfach eine Mail mit Betreff `PATCHTEST Zahlungsavis` an `avis@firma.de` schicken und 1 bis 5 Minuten warten.
6. Richtig: In Outlook hat die Mail den neuen Betreff und die Kategorie „Avise", im Ausführungsverlauf sind beide Aktionen grün. Fehler: siehe Fehlertabelle (403, 404, 400).
7. Danach den Testflow ausschalten: Meine Flows → `PATCHTEST` → **⋯** → **Deaktivieren**.

## Schritt 5: Flow „Avis V2" bauen

Allgemeine Hinweise zum Designer:

- **Nachbau daneben öffnen:** `dist/avis-flow.html` herunterladen, im Browser öffnen und oben rechts bei „Postfach" die Adresse eintragen. Jede Karte dort zeigt die Felder genau wie im Designer, mit den fertigen Formeln zum Kopieren. Nach dem Speichern kann man im Reiter **Codeansicht** die Codeansicht des eigenen Flows einfügen und vergleichen lassen.
- **Umbenennen:** Aktion anklicken, oben im Bereich auf den Titel klicken, den neuen Namen eintippen, Enter. Auf der Fläche zeigt Power Automate statt `_` ein Leerzeichen („Run script"). Das ist normal; die Formeln brauchen den Namen mit `_`.
- **Formeln** immer über **fx** einfügen: ins Feld klicken → rechts neben dem Feld auf **fx** → Formel aus der passenden Datei in `formeln/` einfügen → **Hinzufügen**. Meldet der Editor „Der Ausdruck ist ungültig", die Formel prüfen; ist sie genau kopiert, hilft oft ein zweiter Klick auf **Hinzufügen**.
- **Werte aus dem Trigger** (Betreff, Nachrichten-ID): ins Feld klicken → **Blitz-Symbol** → im Suchfeld den Namen eintippen → anklicken.
- Tippst du ein „/" in ein Feld, öffnet sich ein kleines Menü. Einfach weitertippen; wählst du dort etwas aus, verschwindet das „/".

### 5.1 Neuer Flow und Trigger

1. Power Automate → **Erstellen** → **Automatisierter Cloud-Flow** → Name `Avis V2`.
2. Trigger suchen: `freigegebenen Postfach` → **Wenn eine neue E-Mail in einem freigegebenen Postfach eingeht (V2)** → **Erstellen**.
3. Trigger anklicken:
   - **Ursprüngliche Postfachadresse**: `avis@firma.de` eintippen → **„avis@firma.de" verwenden** anklicken.
   - **Erweiterte Parameter** → **Alle anzeigen**:
     - **Nur mit Anlagen**: **Nein** (auch Avise nur im Mailtext sollen ankommen)
     - **Anlagen einschließen**: **Nein** (die Anlagen holt Export_email)
     - **Ordner**: Ordnersymbol rechts im Feld → **Posteingang**
     - alle anderen Felder leer lassen
4. Reiter **Einstellungen** des Triggers:
   - **Parallelitätssteuerung**: **Ein**
   - **Grad der Parallelität**: `1`
   - **Maximale Anzahl wartender Ausführungen**: `100`

   Grund: Excel lässt das Skript höchstens 3-mal in 10 Sekunden laufen. Mit Grad 1 arbeitet der Flow die Mails nacheinander ab. Die Flowprüfung zeigt dazu den Hinweis „Drosselung der Trigger-Parallelität erkannt". Den kann man ignorieren.

### 5.2 Bedingung Keine_Automatik

1. **+** unter dem Trigger → Suche `Bedingung` → **Bedingung** (Steuerung) → umbenennen in `Keine_Automatik`.
2. Verknüpfung oben links: **Und**.
3. Fünf Zeilen, jede so: links Blitz → **Betreff** (aus dem Trigger), Mitte **enthält nicht**, rechts der Text:
   - `Automatische Antwort`
   - `Abwesend`
   - `Unzustellbar`
   - `Automatic reply`
   - `Undeliverable`
4. Der Designer hängt nach dem Ausfüllen oft eine leere Zeile an: rechts **⋯** → **Löschen**. Nach dem Speichern und Neuladen zeigt er trotzdem wieder eine leere Zeile an; die wird nicht gespeichert und stört nicht.

Alle folgenden Aktionen kommen in den Zweig **Wahr** von Keine_Automatik. Der Zweig **Falsch** bleibt leer.

### 5.3 Export_email

1. Im Zweig **Wahr**: **+** → Suche `E-Mail exportieren` → **E-Mail exportieren (V2)** (Office 365 Outlook) → umbenennen in `Export_email`.
2. **Nachrichten-ID**: Blitz → `Nachrichten-ID` → den Eintrag **Nachrichten-ID** des Triggers anklicken.
3. **Erweiterte Parameter** → **Alle anzeigen** → **Ursprüngliche Postfachadresse**: `avis@firma.de` eintippen → **„avis@firma.de" verwenden**.

Die Aktion gibt die ganze Mail als Datei (.eml) weiter: Text, alle Anlagen, auch weitergeleitete Mails.

### 5.4 Run_script

1. **+** unter Export_email → Suche `Skript ausführen` → **Skript ausführen** (Excel Online (Business)) → umbenennen in `Run_script`.
2. Fragt Power Automate nach einer Anmeldung für Excel: **Anmelden** und im Microsoft-Fenster das Konto des Flows anklicken. (Öffnet sich kein Fenster, hat der Browser es blockiert: rechts in der Adressleiste Pop-ups für make.powerautomate.com erlauben.)
3. Felder:
   - **Speicherort**: **OneDrive for Business**
   - **Dokumentbibliothek**: **Dokumente** (englische Oberfläche: OneDrive)
   - **Datei**: Ordnersymbol → **AvisParser.xlsx**
   - **Skript**: **AvisV2**
   - **pdfBase64**: **fx** → Formel aus `formeln/Run_script_pdfBase64.txt`: `base64(body('Export_email'))`
   - **Erweiterte Parameter** → **Alle anzeigen** → **mailSubject**: Blitz → **Betreff** (Trigger)
   - **extraKeywords**: leer lassen
4. Reiter **Einstellungen** → **Wiederholungsrichtlinie**: **Festes Intervall**, **Anzahl** `3`, **Intervall** `PT1M`.

Im Lasttest mit 40 Mails brauchte ein Lauf meist 8 bis 20 Sekunden, auch für ein Avis mit 72 Seiten oder eine Mail mit 20 PDF-Anlagen; das meiste davon ist der Start von Excel.

### 5.5 Bedingung Kategorie_noetig

1. **+** unter Run_script → **Bedingung** → umbenennen in `Kategorie_noetig`.
2. Eine Zeile:
   - links: **fx** → Formel aus `formeln/Kategorie_noetig_links.txt`
   - Mitte: **ist gleich**
   - rechts: **fx** → `true` → **Hinzufügen** (nicht als Text eintippen, sonst ist es das Wort „true")
3. Eine angehängte leere Zeile löschen (⋯ → Löschen).
4. Reiter **Einstellungen** → **Ausführen nach** → **Run script** aufklappen → **Ist erfolgreich**, **Timeout** und **Ist fehlgeschlagen** anhaken. So bekommt auch eine Mail, bei der das Skript nicht laufen konnte, die Kategorie „Avis prüfen".

### 5.6 Kategorie_setzen

1. Im Zweig **Wahr** von Kategorie_noetig: **+** → Suche `HTTP-Anforderung senden Office 365 Outlook` → **HTTP-Anforderung senden** (Office 365 Outlook) → umbenennen in `Kategorie_setzen`.
2. Felder:
   - **URI**: **fx** → Formel aus `formeln/URI_Kategorie_und_Betreff.txt`, mit eurer Postfachadresse statt `avis@firma.de`
   - **Methode**: **PATCH**
   - **Erweiterte Parameter** → **Alle anzeigen** → **Text**: **fx** → Formel aus `formeln/Kategorie_setzen_Text.txt`
   - **Content-Type**: `application/json` (steht schon da)

Die Aktion setzt die Kategorie, die das Skript liefert, z. B. `{"categories":["Avise"]}`. Kategorien, die schon an der Mail hingen, bleiben erhalten. Ist das Skript gescheitert, setzt sie „Avis prüfen".

### 5.7 Bedingung Betreff_noetig

1. **+** unter Kategorie_setzen (noch im Zweig Wahr von Kategorie_noetig) → **Bedingung** → umbenennen in `Betreff_noetig`.
2. Eine Zeile: links **fx** → Formel aus `formeln/Betreff_noetig_links.txt`, Mitte **ist gleich**, rechts **fx** → `true`. Leere Zeile löschen.

Bei „mehrere Avise", unklaren Fällen und schon bearbeiteten Betreffs liefert das Skript keinen neuen Betreff. Dann bleibt es bei der Kategorie.

### 5.8 Betreff_setzen

1. Im Zweig **Wahr** von Betreff_noetig: **+** → **HTTP-Anforderung senden** (Office 365 Outlook) → umbenennen in `Betreff_setzen`.
2. Felder:
   - **URI**: **fx** → dieselbe Formel wie bei Kategorie_setzen
   - **Methode**: **PATCH**
   - **Text**: **fx** → Formel aus `formeln/Betreff_setzen_Text.txt`
   - **Content-Type**: `application/json`

### 5.9 Speichern

Oben **Speichern**. Die Meldung „Ihr Flow ist fertiggestellt" erscheint. Der Flow ist sofort eingeschaltet und bearbeitet jede neue Mail im Posteingang des Postfachs (ältere Mails nicht).

## Schritt 6: Testen

1. Aus dem eigenen Postfach eine Mail mit einem Zahlungsavis-PDF an `avis@firma.de` schicken, eine zweite ohne Avis (z. B. eine Rechnung).
2. 1 bis 5 Minuten warten. Power Automate fragt das Postfach nur alle paar Minuten ab.
3. In Outlook im Web das Postfach öffnen: Die Avis-Mail hat die Kategorie „Avise" und den Betrag im Betreff, die andere Mail ist unverändert.
4. Ausführungsverlauf: Power Automate → **Meine Flows** → **Avis V2** → unten die Läufe. Einen Lauf anklicken → **Keine Automatik** aufklappen → **Run script** anklicken → unter **Ausgaben** stehen `result.verdict`, `result.category`, `result.newSubject`, `result.verdictWhy` und je Dokument `result.documents`.

Was das Skript zurückgibt:

| Feld | Inhalt |
| --- | --- |
| `verdict` | `advice` (ein Avis), `several` (mehrere), `unclear`, `other` (kein Avis) |
| `verdictWhy` | Kurzer Grund auf Englisch, z. B. `advice title 'Zahlungsavis' in Zahlung_4711.pdf` (Anhang B) |
| `category` | `Avise`, `Avis prüfen`, `mehrere Avise` oder leer |
| `subjectSuffix` / `newSubject` | Anhang an den Betreff / der fertige neue Betreff (leer = keine Änderung) |
| `patchCategory` / `patchSubject` | Fertiger Text für Kategorie_setzen und Betreff_setzen (leer = nichts tun) |
| `documents` | Je geprüftem Dokument eine Zeile: Name, Art, Urteil, Betrag, Sicherheit, Grund |
| `amount`, `amountNumber`, `currency`, `confidence`, `line`, `method`, `candidates`, `pages`, `text`, `textLength`, `warnings`, `error` | Wie in Version 1, zur Fehlersuche (Gründe und Hinweise: Anhang B) |

## Schritt 7: Vom alten Flow umstellen

- Nie zwei Flows gleichzeitig auf dasselbe Postfach loslassen. Lief der alte Flow (Version 1) auf demselben Postfach: dort **⋯** → **Deaktivieren**, dann den neuen einschalten.
- Lief der alte Flow auf einem persönlichen Postfach, kann er weiterlaufen, bis der neue sich bewährt hat.
- Mails, die vor dem Einschalten eingegangen sind, bearbeitet der neue Flow nicht.

## Im Alltag

- **Avise:** Betrag im Betreff. Endet er auf „(prüfen)", ist sich das Skript nur ziemlich sicher (z. B. Avis nur im Mailtext). In den ersten Wochen jede Avis-Mail kurz gegen das Dokument halten.
- **Avis prüfen:** Steht ein Betrag mit „?" im Betreff, ist er ein Vorschlag. Ohne Betrag: Mail öffnen und von Hand buchen. Den Grund zeigt der Ausführungsverlauf bei Run script (`result.error`, `result.verdictWhy`, Anhang B).
- **mehrere Avise:** Die Mail enthält zwei oder mehr Avise mit verschiedenen Beträgen. Beträge von Hand buchen; der Betreff bleibt unverändert.
- **Keine Kategorie:** kein Avis, oder der Flow ist an dieser Mail nicht gelaufen (Ausführungsverlauf prüfen).
- **Suchen:** Den Betrag ins Outlook-Suchfeld eintippen, z. B. `2.601,35`.
- **Antworten:** Wer auf eine bearbeitete Mail antwortet, schickt den Betrag im Betreff mit („AW: Zahlungsavis - 2.601,35 EUR").
- Ein Betreff, der schon mit dem Betrag endet, wird kein zweites Mal ergänzt.
- Eine Mail ohne Betreff bekommt „(Kein Betreff) - 2.601,35 EUR" (so zeigt Outlook leere Betreffzeilen an). Sehr lange Betreffzeilen werden ebenfalls ergänzt (live getestet mit 250 Zeichen).
- **Zwei Läufe für eine Mail:** Bei vielen gleichzeitig eintreffenden Mails startet Power Automate manchmal zwei Läufe für dieselbe Mail (im Lasttest bei 20 von 40 Mails). Das schadet nicht: Der zweite Lauf setzt dieselbe Kategorie und denselben Betreff, der Betrag steht nie doppelt im Betreff.

## Fehler finden

| Was du siehst | Ursache | Was tun |
| --- | --- | --- |
| Flow startet nicht | Flow aus, falsche Postfachadresse oder falscher Ordner im Trigger, Mail ging in einen anderen Ordner | Trigger prüfen; Ausführungsverlauf ansehen |
| Kategorie_setzen oder Betreff_setzen: **403** | Kein Vollzugriff, falsches Konto in der Verbindung oder neue Rechte noch nicht aktiv | Schritt 1; bis zu 2 Stunden warten; Verbindung unten in der Aktion prüfen |
| **404** | URI falsch (Postfachadresse) oder die Mail wurde verschoben oder gelöscht, bevor der Flow lief | Formel in URI prüfen; Kolleginnen und Kollegen bitten, neue Mails ein paar Minuten liegen zu lassen |
| **400** | Text (Body) falsch | Formel aus `formeln/` neu einfügen, über **fx** |
| Run_script: „We couldn't find the script" oder „BadGateway" | Skript nicht gespeichert, anders benannt oder andere Datei gewählt | In Excel prüfen, dass das Skript **AvisV2** heißt und gespeichert ist; in Run_script Datei und Skript neu wählen |
| Run_script: Zeitüberschreitung | Sehr große Mail | Die Mail bekommt „Avis prüfen"; von Hand bearbeiten |
| Kategorie grau statt farbig | Kategorie nicht im Postfach angelegt | Schritt 2 |
| Betreff_setzen rot, Kategorie gesetzt, ab Januar 2027 | Microsofts Betreff-Sperre ist aktiv | Abschnitt „Ab Januar 2027" |
| Abwesenheitsnotiz bekommt eine Kategorie | Text der Notiz nicht in Keine_Automatik | Zeile in Keine_Automatik ergänzen; das Skript erkennt die meisten automatischen Antworten auch selbst |

## Ab Januar 2027: Betreff-Sperre prüfen

Microsoft hat angekündigt, ab dem 31.12.2026 Änderungen an Betreff und Text empfangener Mails über die Graph-Schnittstelle zu sperren. Kategorien sind nicht betroffen.

1. In der ersten Januarwoche den 5-Minuten-Test (Schritt 4) wiederholen.
2. Sind beide Aktionen grün: nichts tun.
3. Ist die Kategorie-Aktion grün und die Betreff-Aktion rot (meist 403): Die Sperre greift. Dann auf Plan B umstellen. Plan B legt statt der Betreffänderung eine Weiterleitung als Entwurf an, setzt dort den Betreff, schickt sie an das Postfach und verschiebt das Original in einen Ordner „Avis-Original". Alle Aktionen und Formeln dazu stehen in `dist/avis-flow.html` (oben auf „Plan B" umschalten). Plan B braucht zusätzlich „Senden als" (Schritt 1 gibt das schon).

## Grenzen

- Etwa 150 Avise und 150 andere Mails am Tag ergeben rund 300 Skript-Aufrufe (Limit 1.600 pro Nutzer und Tag) und rund 1.500 Power-Automate-Anfragen (Limit bei Microsoft-365-Lizenz offiziell 6.000 in 24 Stunden).
- Das Skript darf höchstens 3-mal in 10 Sekunden laufen (daher Grad 1 und Wiederholungsrichtlinie) und höchstens 120 Sekunden. Es hört nach 90 Sekunden selbst auf und meldet, was es nicht mehr geschafft hat.
- Sehr große Mails (über etwa 4 MB) können am Größenlimit scheitern und bekommen dann „Avis prüfen".
- Nicht lesbar: reine Scans ohne Text (keine Texterkennung), verschlüsselte Mails und PDFs mit Kennwort, altes Excel-Format .xls, Word-Dateien, mit Microsoft Purview geschützte Mails. Sie bekommen „Avis prüfen", wenn ein Hinweis auf ein Avis da ist.
- Bilder als Anlage (Foto, Fax als TIFF/JPG) werden nicht gelesen.
- Je Mail liest das Skript höchstens 30 Anlagen (Dateien mit „Avis" oder „Zahlungsavis" im Namen zuerst). Hat eine Mail mehr, bekommt sie „Avis prüfen", wenn unter den gelesenen kein Avis war.

## Anhang A: Alle Formeln zum Kopieren

Die Formeln liegen auch einzeln in `formeln/` (Datei öffnen, **Copy raw file**, im Feld über **fx** einfügen).

| Wo | Formel | Datei |
| --- | --- | --- |
| Run_script, pdfBase64 | `base64(body('Export_email'))` | `Run_script_pdfBase64.txt` |
| Kategorie_noetig, links (rechts: fx `true`) | `or(not(equals(actions('Run_script')?['status'], 'Succeeded')), not(empty(outputs('Run_script')?['body/result/patchCategory'])))` | `Kategorie_noetig_links.txt` |
| Kategorie_setzen und Betreff_setzen, URI | `concat('https://graph.microsoft.com/v1.0/users/avis@firma.de/messages/', triggerOutputs()?['body/id'])` | `URI_Kategorie_und_Betreff.txt` |
| Kategorie_setzen, Text | `if(equals(actions('Run_script')?['status'], 'Succeeded'), outputs('Run_script')?['body/result/patchCategory'], '{"categories":["Avis prüfen"]}')` | `Kategorie_setzen_Text.txt` |
| Betreff_noetig, links (rechts: fx `true`) | `and(equals(actions('Run_script')?['status'], 'Succeeded'), not(empty(outputs('Run_script')?['body/result/patchSubject'])))` | `Betreff_noetig_links.txt` |
| Betreff_setzen, Text | `outputs('Run_script')?['body/result/patchSubject']` | `Betreff_setzen_Text.txt` |

## Anhang B: Gründe und Hinweise auf Deutsch

Das Skript schreibt Gründe und Hinweise auf Englisch, damit sie überall gleich lesbar sind. Du findest sie im Ausführungsverlauf: Lauf anklicken → **Keine Automatik** aufklappen → **Run script** → **Ausgaben**: `result.error` (Grund), `result.warnings` (Hinweise), `result.verdictWhy` (Urteil über die Mail) und `result.documents` (je Dokument). In der Tabelle steht der Anfang des englischen Textes. Wo `…` steht, folgen Zahlen oder Details aus dem Dokument.

**Kein Betrag gefunden (ok = false)**

| Grund (englisch) | Bedeutung | Was tun |
| --- | --- | --- |
| `no PDF content passed` | Das Skript hat keine Datei bekommen | In Run_script das Feld pdfBase64 prüfen: fx `base64(body('Export_email'))` (Schritt 5.4) |
| `not a PDF (header missing, …)` | Die Anlage ist keine PDF oder beschädigt, auch wenn sie so heißt. In der Klammer stehen die Größe und die ersten Zeichen dessen, was beim Skript ankam: `starts 'JVBER…'` ist eine Base64-PDF, `starts '%PDF…'` eine PDF als Text | Anlage von Hand öffnen; Betrag von Hand. Kommt das bei jeder Mail: Feld pdfBase64 in Run_script prüfen (Schritt 5.4) |
| `PDF arrived as text instead of Base64 - content damaged (…)` | Power Automate hat die PDF als Text statt als Base64 übergeben, dabei sind Teile verloren gegangen. In der Klammer steht, woran das Lesen gescheitert ist | Betrag von Hand. In Run_script das Feld pdfBase64 auf fx `base64(body('Export_email'))` umstellen (Schritt 5.4) |
| `PDF is password protected - cannot be read` | Die PDF hat ein Kennwort zum Öffnen | Betrag von Hand (Kennwort beim Absender erfragen). PDFs, die nur gegen Kopieren oder Drucken geschützt sind, liest das Skript dagegen normal |
| `PDF is encrypted with a certificate - cannot be read` | Die PDF ist mit einem Zertifikat verschlüsselt | Betrag von Hand |
| `PDF is encrypted with an unsupported method (…) - cannot be read` | Die PDF ist mit einem Verfahren verschlüsselt, das das Skript nicht kennt | Betrag von Hand |
| `no pages found` | In der Datei wurden keine Seiten gefunden (Datei kaputt oder abgeschnitten) | PDF von Hand öffnen; ggf. beim Absender neu anfordern |
| `no text layer found (scanned PDF?)` | Reiner Scan: Die PDF ist nur ein Bild, ohne Text | Betrag von Hand |
| `PDF read incompletely (time budget exhausted)` | Die PDF war so groß oder aufwendig, dass das Zeitlimit von 90 Sekunden erreicht wurde, bevor Text gelesen war | Betrag von Hand |
| `no amount pattern found` | Text gelesen, aber kein Betrag erkannt (z. B. Beträge ohne Nachkommastellen wie „3.454 EUR") | Betrag von Hand |
| `exception: …` | Interner Fehler im Skript | Betrag von Hand; PDF aufheben und an die Person geben, die das Skript pflegt |

**Signierte, verschlüsselte und geschützte Mails und PDFs**

| Grund (englisch) | Bedeutung | Was tun |
| --- | --- | --- |
| `S/MIME encrypted mail - cannot be read` | Die Mail ist mit S/MIME verschlüsselt. Nur dein Outlook mit deinem Zertifikat kann sie öffnen | Original-Mail in Outlook öffnen, Betrag von Hand |
| `protected mail (Microsoft Purview) - cannot be read` | Die Mail ist mit Microsoft Purview geschützt (Anlage `message.rpmsg`, z. B. „Verschlüsseln" oder „Nicht weiterleiten" in Outlook) | Original-Mail in Outlook öffnen, Betrag von Hand |
| `signed mail contains no PDF` | Die signierte Mail enthält keine PDF (das Avis steht vielleicht im Mailtext oder ist eine andere Datei) | Original-Mail öffnen, Betrag von Hand |
| `protected PDF - cannot be read` | Geschützte PDF (z. B. mit Microsoft Purview / Vertraulichkeitsbezeichnung), nur mit Berechtigung zu öffnen | In Outlook oder Acrobat öffnen, Betrag von Hand |

**Unsicher (ok = true, Grund beginnt mit `uncertain:`)**

Nach `uncertain:` folgen ein oder mehrere dieser Gründe, getrennt durch `;`. Den Vorschlag des Skripts siehst du im Betreff (mit „?") und im Ausführungsverlauf unter `result.candidates`.

| Grund (englisch) | Bedeutung |
| --- | --- |
| `OCR text layer` | Der Text stammt aus einer Texterkennung (OCR) über einem Scan. Ziffern können falsch erkannt sein |
| `PDF arrived as text instead of Base64 - parts may be damaged` | Power Automate hat die PDF als Text statt als Base64 übergeben. Ein Teil ließ sich lesen, andere Teile sind verloren. Den Vorschlag mit der PDF vergleichen und das Feld pdfBase64 in Run_script auf fx `base64(body('Export_email'))` umstellen (Schritt 5.4) |
| `PDF read incompletely (…)` | Die PDF konnte nur teilweise gelesen werden (beschädigt, abgeschnitten, Zeichen ohne Zuordnung). Der richtige Betrag kann fehlen |
| `PDF read incompletely (time budget exhausted)` | Die PDF war so groß oder aufwendig, dass das Zeitlimit von 90 Sekunden erreicht wurde |
| `PDF changed after signing` | Die PDF selbst ist digital signiert und wurde nach der letzten Signatur noch verändert. Der gelesene Betrag ist vielleicht nicht der signierte. Im Zweifel beim Absender nachfragen |
| `several PDFs in the mail with different amounts (…)` | Die signierte Mail enthält mehrere PDFs mit verschiedenen Beträgen (in der Klammer: Dateiname und Betrag der anderen PDF) |
| `'…' - not a payment advice?` | Im Dokument steht ein Wort wie „Mahnung", „Lieferschein", „Lastschrift", „Angebot", „Auftragsbestätigung" oder „Abschlagsrechnung" (oder ein fremdsprachiger Titel wie „Estratto conto", „Bulletin de paie"), aber keine Überschrift wie „Zahlungsavis". Auch eine Zahlungsbitte („Wir bitten Sie, den Restbetrag … zu überweisen") oder ein Bankeinzug („… wird von Ihrem Konto abgebucht") steht hier. Wahrscheinlich kein Zahlungsavis. Wörter in Bezugsfeldern („Ihr Zeichen: Mahnung vom …"), in der Empfängeradresse („Personalabteilung / Entgeltabrechnung") und in einer reinen Abteilungszeile („z. Hd. Lohn- und Gehaltsabrechnung") zählen nicht als Titel. „Ihre Mahnung" oder „bezugnehmend auf Ihre Zahlungserinnerung" im Text zählt nur dann nicht, wenn das Schreiben eine erfolgte Zahlung nennt („haben wir heute überwiesen", „dass wir heute … an Sie angewiesen haben", „heute überweisen wir"); ein Schreiben, das die Zahlung verweigert oder erst ankündigt, geht in die Prüfung. Das gilt genauso für einen genannten Lieferschein („Ihr Lieferschein LS-55871", „Bezug: Lieferschein 80012345"). Ein genannter Lieferschein zählt aber nur dann nicht, wenn der gewählte Betrag als Zahlung beschriftet ist; sonst bleibt der Grund. Ein verneinter Satz („haben wir noch nicht überwiesen") ist keine erfolgte Zahlung. Auch eine Gutschriftsanzeige der Bank über eine Rücküberweisung, eine Rückbuchung, eine Wiedergutschrift oder Rückgabe einer Lastschrift, eine Dividende oder Ertragsgutschrift, erstattete Bankentgelte oder eigene Zinsen der Bank („Zinsgutschrift"), eine Gutschriftsanzeige, deren Auftraggeber die Bank selbst ist, und eine „Ratenzahlungsvereinbarung" stehen hier. Der Hinweis „Eine Rückbuchung bleibt vorbehalten" und „Rechnungsabschluss" in den Bankbedingungen zählen nicht. Ebenso: eine fehlgeschlagene Auszahlung („Payout failed", „Status: Reversed by your bank"), eine „Belastungsanzeige" oder „Debit advice" ohne Empfänger, eine „Umsatzübersicht", „Auszahlungsvorschau", „Vorläufige Abrechnung" oder ein „Zahlungsvorschlag", Zahlungsbedingungen für den Leser, wenn das Schreiben keine eigene Zahlung nennt („Wenn Sie bis zum … überweisen, können Sie 2 % Skonto abziehen", „Bei Überweisung bis … gewähren wir 2 % Skonto", „Zahlungsbedingungen", „is still outstanding", „… ist noch offen", „die Skontofrist war bereits abgelaufen") und „Den offenen Restbetrag von … erwarten wir" (nicht, wenn eine Gutschrift erwartet wird) |
| `asks for a payment advice - not an advice itself` | Das Schreiben bittet um ein Zahlungsavis (z. B. eine Mahnung mit „senden Sie uns ein Zahlungsavis" oder ein Schreiben mit dem Titel „Bitte um Zahlungsavis", „Anforderung Zahlungsavis", „Zahlungsavis fehlt", „Zahlungsavis angefordert", „Request for remittance advice", „… but no remittance advice"). Es ist selbst kein Avis. Antwortet der Zahler auf so eine Bitte („Ihrer Bitte um Zusendung des Zahlungsavis kommen wir nach", Betreff „AW:", „Antwort:" oder „WG: Zahlungsavis fehlt"), bleibt es ein Avis. Das gilt nur, wenn zwischen „Ihre Bitte / Anfrage / Anforderung" und dem Wort „Zahlungsavis" nichts anderes steht als Wörter wie „um Zusendung des". „Ihre Anfrage haben wir erhalten, das Zahlungsavis fehlt uns aber noch" oder „… liegt uns aber nicht vor" bleibt eine Bitte |
| `payee's letter about a received payment - not a payment advice?` | Der Absender hat das Geld selbst bekommen (z. B. „auf unserem Konto ist ein Betrag von … eingegangen", „können wir keiner Rechnung zuordnen", „we are unable to match it"). Das ist ein Schreiben des Zahlungsempfängers, kein Avis. Auch „Ihre Teilzahlung über 2.500,00 € ist heute eingegangen" ist die Sicht des Empfängers. Nennt der Zahler etwas, das bei ihm eingegangen ist (die Rechnung, eine Rückzahlung), oder schreibt er „If you cannot allocate this payment, please contact us", bleibt es ein Avis. Auch diese Sätze sind die Sicht des Empfängers: „Sie haben uns … überwiesen", „Sie haben unsere Rechnung … mit 1.960,00 EUR beglichen", „dass Sie die Rechnung … beglichen haben", „wir bestätigen den Eingang Ihrer Zahlung", „Ihre Überweisung … haben wir dankend erhalten", „… wurde unserem Konto … gutgeschrieben", „vielen Dank für Ihre Teilzahlung", „thank you for your payment", „we confirm receipt of your payment", „you paid EUR …". „Your payment reminder … was received. We have today paid …" und „thank you for your payment reminder" bleiben ein Avis des Zahlers |
| `credit note title - not a payment advice?` | Das Dokument ist als Gutschrift überschrieben (z. B. Gutschrift eines Lieferanten), nicht als Zahlungsavis |
| `amount due with VAT - invoice?` | Ein fälliger Betrag zusammen mit einer Mehrwertsteuerzeile. Sieht nach einer Rechnung aus |
| `payroll words - not a payment advice?` | Wörter einer Lohn- oder Gehaltsabrechnung. Wahrscheinlich kein Zahlungsavis |
| `travel expense words - not a payment advice?` | Wörter einer Reisekostenabrechnung. Wahrscheinlich kein Zahlungsavis |
| `shipping words - not a payment advice?` | Wörter einer Versand- oder Lieferbestätigung. Wahrscheinlich kein Zahlungsavis |
| `no payment wording - not a payment advice?` | Im Dokument kommt kein Wort wie Zahlung, überwiesen, Gutschrift oder (englisch) payout, paid out, amount transferred vor. Bei fremdsprachigen Dokumenten fehlt ein Satz wie „nous avons payé" / „wij hebben betaald" / „przelaliśmy", oder es steht ein offener Betrag oder eine Lastschrift darin. Ein offener Rest nach einer Teilzahlung („Pozostało do zapłaty", „Do zapłaty pozostało", „reste à payer") zählt nicht als offener Betrag, wenn der Zahler die Zahlung nennt. „Next payout", „since last payout", „expected payout" oder „available for payout" zählen nicht als Zahlungswort. Wahrscheinlich kein Zahlungsavis |
| `amount from the quoted e-mail` | Der Betrag steht nur im zitierten Teil einer beantworteten oder weitergeleiteten Mail, nicht im eigenen Text |
| `credit note not netted in the total` | Eine Gutschrift steht in einem eigenen Block und ist in der Summe nicht verrechnet |
| `total … below includes the payment` | Weiter unten steht eine Summe, die den gewählten Betrag und weitere Beträge enthält |
| `payment amount without decimals` | In einem Satz zur Zahlung steht ein Betrag ohne Nachkommastellen (z. B. „3.454 EUR"), den das Skript nicht liest |
| `deduction near the total` | Neben der Summe steht ein Abzug (Skonto, Gebühr, Gutschrift). Unklar, ob vor oder nach Abzug gezahlt wurde |
| `deduction below the total` | Unter der Summe steht noch ein Abzug oder eine Gutschrift, die nicht verrechnet ist |
| `amount looks like a deduction` | Der gewählte Betrag sieht selbst wie ein Abzug aus |
| `close runner-up …` | Ein zweiter Betrag ist fast genauso wahrscheinlich (steht dahinter) |
| `second payment amount …` | Es gibt zwei Zahlbeträge (der zweite steht dahinter). Das kommt auch, wenn unter einer Summe Abzüge und dann eine kleinere Zahlzeile stehen, die nicht aufgeht (z. B. „Zahlbetrag 10.000,00", „abzüglich Delkredere 150,00", „Gutschrift auf Ihr Konto 9.800,00"). Geht die Rechnung auf, nimmt das Skript die untere Zeile und dieser Grund kommt nicht |
| `transfer amount … differs from …` | Ein ausdrücklich genannter Überweisungsbetrag (z. B. „Überweisungsbetrag", „Auszahlungsbetrag", „Gutschriftsbetrag", oft im Kopf oder auf der letzten Seite) weicht vom gewählten Betrag ab, meist von der Tabellensumme. Die Differenz ergibt sich nicht aus den Abzugszeilen im Dokument (Gebühr, USt auf die Gebühr, Delkredere, Bonus, Skonto). Welcher Betrag überwiesen wurde, ist unklar. Steht der genannte Betrag als Zwischenschritt über weiteren Abzügen („Zahlbetrag lt. Ihrer Rechnung", darunter „abzüglich Skonto" und „Überweisung"), erklären die Abzüge dazwischen die Differenz, und dieser Grund kommt nicht |
| `payment sentences with different amounts` | Im Text stehen mehrere Sätze wie „wir haben … überwiesen" mit verschiedenen Beträgen. Keiner ist als frühere Zahlung („bereits", „schon", „zuvor") oder als heutige Zahlung („heute", „Restbetrag") erkennbar, und keiner ist die Summe der anderen. Eine erst angekündigte Zahlung („überweisen wir nach Klärung", „nächste Woche", „im November", „am 15.12.2026") oder eine noch nicht geleistete („haben wir noch nicht überwiesen") zählt dabei nicht mit |
| `several amounts in the payment sentence` | Ein Satz zur Zahlung nennt einen Abzug (Skonto, Rabatt, Gutschrift) und mehrere Beträge, und die Rechnung geht nicht auf. Beispiel: „über 1.190,00 EUR abzüglich 2 % Skonto, also 1.160,00 EUR" (2 % von 1.190,00 ergeben 1.166,20) oder „abzüglich Skonto, also 1.166,20 EUR" ohne Prozentsatz und ohne Skontobetrag. Auch eine Summe, die nicht aufgeht („… zusammen 1.450,00 EUR"), oder eine Summe, von der nur ein Teil bezahlt ist („insgesamt 3.000,00 EUR, haben wir heute nur die Rechnung … bezahlt"), steht hier. Vorgeschlagen wird der Betrag nach „also" / „=". Ein Betrag nach „statt" / „instead of" / „nicht" zählt in einem Zahlungssatz nicht als Zahlbetrag („den vollen Betrag von 2.000,00 EUR statt 1.960,00 EUR"). Eine Rate von einer errechneten offenen Summe („ergibt 4.000,00 EUR. Hiervon haben wir heute eine erste Rate von 2.000,00 EUR überwiesen") ist kein Widerspruch |
| `payment announced for later` | Der gewählte Betrag wird nur angekündigt: „überweisen wir im Dezember", „werden wir am 15.11.2026 überweisen", „sobald die Mängel behoben sind", „haben wir noch nicht überwiesen", oder er heißt „Pending payout", „Estimated payout", „Expected payout", „Voraussichtlicher Auszahlungsbetrag", „Auszahlungsbetrag (vorläufig)", „Payout on hold". Auch: „Die Überweisung wird am 20.11.2026 ausgeführt", „Die Auszahlung erfolgt am …", „ist für den 30.11.2026 vorgesehen", „haben die Zahlung zur Ausführung am 20.11.2026 freigegeben", „überweisen wir am 15. Dezember", „in der 46. KW", „nach Abnahme / Wareneingang", „hiermit teilen wir Ihnen mit, dass wir … überweisen werden", „… werden wir erst nach Erhalt … begleichen. Der Betrag wird Ihnen dann überwiesen". Nennt das Schreiben daneben eine heutige oder erfolgte Zahlung, nimmt das Skript diese, und der Grund kommt nicht. „Der Betrag wird Ihrem Konto mit Valuta … gutgeschrieben" ist keine Ankündigung, ebenso wenig der Monat der Lieferung oder Leistung („für die im Oktober gelieferte Ware"), ein Fälligkeitsdatum („fällig am …") oder ein Zeitraum („Leistungszeitraum 01.11.2026 bis 30.11.2026") |
| `negative amount - not a received payment?` | Der gewählte Betrag ist negativ (z. B. „-120,00" in einer Umsatzliste), und das Dokument hat keine Überschrift wie „Zahlungsavis". Das ist eher Geld, das abgeflossen ist, kein Zahlungseingang. Ein Avis mit Überschrift, das seine Summe mit nachgestelltem Minus druckt („Gesamt 3.454,00- EUR"), bekommt diesen Grund nicht |
| `payment line says …, total says …` / `payment line says …` | Die Zahlbetragszeile und die Summe (oder der gewählte Betrag) widersprechen sich |
| `several unlabelled sums` | Mehrere Summen ohne Beschriftung, z. B. „bezahlt" und „offen" |
| `no label, amount chosen by size` | Kein Wort wie „Zahlbetrag" gefunden, nur der größte Betrag |
| `amount only backed by a sum check` | Nur über eine Rechenprobe gefunden (Netto + Steuer = Brutto) |
| `only a weak label (Betrag/Zahlung)` | Nur ein schwaches Wort wie „Betrag" vor dem Betrag |
| `amount from an attached invoice copy` | Der Betrag stammt aus einer angehängten Rechnungskopie, nicht aus dem Avis |
| `total follows a VAT line - invoice?` | Die Summe steht direkt nach einer Mehrwertsteuerzeile. Das sieht eher nach einer Rechnung aus |
| `Skonto/Rest column on the right of the table` | Die Tabelle hat ganz rechts eine Skonto- oder Restbetragsspalte |
| `payment label without readable amount` | „Zahlbetrag" o. ä. steht da, aber der Betrag daneben ist nicht lesbar |
| `gross column chosen, payment column empty` | Betrag aus der Bruttospalte, die Zahlbetragsspalte ist leer oder unlesbar |
| `amount is zero` | Der gefundene Betrag ist 0,00 |
| `several currencies (…)` | Im Dokument kommen mehrere Währungen vor |
| `weak signals` | Allgemein zu wenige Hinweise für einen sicheren Betrag |

**Version 2: ganze Mail (.eml), Mailtext, Tabellen und andere Anlagen**

Mit der ganzen Mail prüft das Skript jede PDF, jede andere lesbare Anlage und den Mailtext. Die Felder `verdict` (advice / several / unclear / other) und `verdictWhy` sagen, wie die Mail eingeordnet wurde; `documents` hat je geprüftem Dokument eine Zeile `Name | Typ | Urteil | Betrag | Konfidenz | Grund`.

| Grund (englisch) | Bedeutung | Was tun |
| --- | --- | --- |
| `several PDFs in the mail with different amounts (…)` / `several advices in the mail with different amounts (…)` | Die Mail enthält zwei oder mehr Avise (PDF, Tabelle oder Mailtext) mit verschiedenen Beträgen. In der Klammer stehen Name und Betrag je Avis. Kategorie „mehrere Avise", der Betreff bleibt | Beträge von Hand buchen |
| `uncertain: the mail text names another payment amount (…) than the attached advice` | Im Mailtext steht „wir haben … überwiesen" mit einem anderen Betrag als im angehängten Avis, und dieser Betrag kommt in keiner Anlage vor (Tippfehler, Teilzahlung, falsches Avis). In der Klammer steht der Betrag aus dem Mailtext. Der Betreff bekommt den Betrag aus dem Avis mit „?", Kategorie „Avis prüfen" | Avis und Zahlungseingang vergleichen, Betrag von Hand prüfen |
| `mail text with another amount than the attached advice (…)` (in `documents`) | Der Mailtext nennt einen anderen Betrag als das angehängte Avis; gezählt wird das Avis. Steht im Mailtext „zusätzlich", „separat", „gesondert", „außerdem" oder „weitere Zahlung", ist der Mailtext eine eigene zweite Zahlung: Kategorie „mehrere Avise" | – |
| `several payments in one document (…)` | Eine Tabelle enthält mehrere getrennte Zahlungen (z. B. zwei Zahlungsblöcke) | Beträge von Hand buchen |
| `mail contains no PDF` | Die Mail hat keine PDF, und weder Mailtext noch andere Anlagen enthalten einen Betrag | Nur bei Kategorie „Avis prüfen" etwas tun: Mail öffnen |
| `file named PDF without a PDF header - cannot be read` | Eine Anlage heißt `.pdf` oder ist als PDF gekennzeichnet, ist aber keine PDF (beschädigt oder etwas anderes) | Anlage von Hand öffnen |
| `Office file (old .xls/.doc, encrypted or .msg) - cannot be read` | Altes Excel- oder Word-Format, eine verschlüsselte Office-Datei oder eine angehängte Outlook-Nachricht (.msg) | Anlage von Hand öffnen, Betrag von Hand |
| `Word document - cannot be read` | Word-Dokument (.docx); das Skript liest keine Word-Dateien | Anlage von Hand öffnen |
| `ZIP file - cannot be read` | ZIP-Archiv oder unbekannte Office-Datei | Anlage von Hand öffnen |
| `spreadsheet could not be read` (auch `spreadsheet could not be read (…)`) | Tabellendatei (z. B. .xlsx), die das Skript nicht lesen konnte | Anlage von Hand öffnen |
| `not checked (limit of 30 documents or time budget)` | Die Mail hat mehr als 30 Anlagen, oder die 90 Sekunden waren um; die übrigen Anlagen wurden nicht geprüft | Mail von Hand prüfen, wenn kein Avis erkannt wurde |
| `not a PDF (header missing, …) - read as …: …` | Statt einer PDF kam eine Tabelle, eine HTML- oder Textdatei; nach dem Doppelpunkt steht, warum darin kein Betrag gefunden wurde | Datei von Hand öffnen |
| `no text` | Ein Mailtext oder eine Textdatei ist leer | – |
| `uncertain: total computed from … rows of column '…'` | Eine Zahlungsliste ohne Summenzeile: Das Skript hat die Zahlbetrag-Spalte selbst addiert (Zahl der Zeilen und Spaltenname in der Meldung). Das kommt auch, wenn das Skript sonst nur eine einzelne Zeile der Liste gefunden hätte | Summe mit der Liste vergleichen |
| `uncertain: list read in part - total not computed` | Eine Zahlungsliste ohne Summenzeile ist länger als 5000 Zeilen (oder das Zeitlimit war erreicht). Eine selbst addierte Summe wäre unvollständig, daher kein Betrag im Betreff; die Mail wird „Avis prüfen" | Summe in der Liste von Hand bilden |
| `uncertain: total row differs from the sum of column '…' (…, … rows)` | Die Summenzeile einer Tabelle passt nicht zur Summe der Zahlbetrag-Spalte (in der Klammer: die nachgerechnete Summe und die Zahl der Zeilen) | Summe mit der Liste vergleichen |
| `old Excel .xls format (…) cannot be read - save it as .xlsx` | Altes Excel-Format (Excel 97-2003, 5.0/95 oder 2-4). Die Mail wird „Avis prüfen" | Anlage von Hand öffnen, Betrag von Hand. Absender bitten, künftig .xlsx, CSV oder PDF zu schicken |
| `password-protected old Excel .xls file cannot be read` / `password-protected (encrypted) Office file cannot be read` / `password-protected ZIP archive cannot be read` | Die Tabelle hat ein Kennwort zum Öffnen | Anlage von Hand öffnen (Kennwort beim Absender erfragen) |
| `rights-protected Office file (IRM) cannot be read` | Die Tabelle ist mit Microsoft Purview / IRM geschützt | Anlage in Excel öffnen |
| `Excel binary workbook (.xlsb) cannot be read - save it as .xlsx` / `OpenDocument spreadsheet (.ods) cannot be read - save it as .xlsx` | Excel-Binärformat oder LibreOffice-Tabelle; das Skript liest nur .xlsx/.xlsm, CSV und die Text-/HTML-Exporte | Anlage von Hand öffnen |
| `Outlook message (.msg) cannot be read` / `old Word .doc format cannot be read` / `old PowerPoint .ppt format cannot be read` | Angehängte Outlook-Nachricht, altes Word- oder PowerPoint-Format | Anlage von Hand öffnen |
| `Office binary file of unknown kind cannot be read` / `damaged Office binary file cannot be read` | Unbekannte oder beschädigte Office-Datei | Anlage von Hand öffnen |
| `damaged Excel file: …` / `damaged spreadsheet cannot be read (…)` | Die Excel-Datei ist beschädigt oder unvollständig (z. B. abgeschnitten) | Anlage von Hand öffnen; ggf. beim Absender neu anfordern |
| `uncertain: total row '…' is not the payment; sum of column '…' (… rows) used` | Die Summenzeile einer Tabelle (in einer Anlage oder im Mailtext) addiert die Rechnungsbeträge („Summe Rechnungen", „brutto"), nicht die Zahlungen. Das Skript nimmt die Summe der Zahlbetrag-Spalte | Summe mit der Liste vergleichen |
| `uncertain: no payment list (…) - total not computed` | Die Tabelle hat eine Zahlbetrag-Spalte, ist aber ein Zahlungsplan, eine Offene-Posten- oder Abstimmungsliste (in der Klammer: Fälligkeitsspalte, Ratenzeilen oder der Datei-/Blattname). Kein Betrag im Betreff, die Mail wird „Avis prüfen" | Liste von Hand ansehen |
| `uncertain: column sum out of range - total not computed` | Die Summe der Zahlbetrag-Spalte ist unsinnig groß (Überlauf in der Datei) | Liste von Hand ansehen |
| `amount out of range (…)` | Der gefundene Betrag ist unsinnig groß (eine Billion oder mehr, z. B. eine falsch gelesene Zahl). Er kommt nie in den Betreff, die Mail wird „Avis prüfen" | Dokument von Hand öffnen |
| `uncertain: sum of … amounts in the payer's sentence (…), no total stated` | Im Mailtext nennt der Zahler mehrere Beträge je Rechnung („980,00 EUR für RE-1 und 2.474,00 EUR für RE-2 überwiesen"), aber keine Summe. Das Skript addiert sie | Summe nachrechnen |
| `uncertain: the mail text refers to an attachment that cannot be read (…)` | Der Betrag stammt nur aus dem Begleittext („anbei unser Zahlungsavis über …"), die Anlage selbst ist unlesbar (Scan, Kennwort, altes .xls). In der Klammer: Name der Anlage und Grund | Anlage von Hand öffnen und Betrag vergleichen |
| `uncertain: … attachments not checked, one named like an advice ('…')` | Die Mail hat mehr als 30 Anlagen; unter den nicht geprüften ist eine, die wie ein Avis heißt | Übrige Anlagen von Hand prüfen |
| `empty attachment (0 bytes) - cannot be read` | Eine Anlage (PDF oder mit Avis-Wort im Namen) ist leer; vermutlich ist beim Absender das Anhängen schiefgegangen | Avis beim Absender neu anfordern |
| `mail parts not read (nested too deep, too many parts or a missing boundary)` | Teile der Mail konnten nicht gelesen werden: sehr oft als Anlage weitergeleitet (über etwa 14 Ebenen), mehr als 5000 Teile oder eine kaputte MIME-Struktur. Die Mail wird „Avis prüfen" | Mail von Hand öffnen |
| `mail could not be parsed` | Die Eingabe beginnt wie eine Mail, ihr Kopf ließ sich aber nicht lesen | Mail von Hand öffnen |
| `e-invoice XML` | Eine XRechnung oder ZUGFeRD-/Factur-X-Datei (XML). Sie ist eine Rechnung, kein Avis, und zählt nicht | – |

| Hinweis (englisch) | Bedeutung |
| --- | --- |
| `from mail: …` / `from signed mail: …` | Aus welchem Dokument der Mail das Ergebnis stammt (`body` = Mailtext) |
| `… PDFs checked (…)` | Welche PDFs der Mail geprüft wurden |
| `not read: …` | Anlagen, die nie gelesen werden: Bilder, Signaturen (`smime.p7s`), Visitenkarten (.vcf), Termine (.ics), `winmail.dat`. Steht ein Avis-Wort im Betreff oder Dateinamen und gibt es sonst kein Avis, wird die Mail „Avis prüfen" |
| `sheet '…': … hidden row(s) skipped` / `… hidden column(s) skipped` / `… cell(s) in hidden columns skipped` | Ausgeblendete Zeilen und Spalten einer Tabelle wurden nicht gelesen (sie zählen auch nicht in die Summe) |
| `sheet '…' is hidden` | Ein ausgeblendetes Tabellenblatt. Es wird nur gelesen, wenn die Datei kein sichtbares Blatt hat |
| `… formula cell(s) without a saved value (file not recalculated)` | Die Datei wurde von einem Programm erzeugt, das Formeln nicht ausrechnet; eine Summenzeile kann leer sein. Das Skript addiert dann selbst (`total computed from …`) |
| `only the first 5000 rows were read` / `only the first 5000 rows of a table were read` / `only the first 64 sheets were read` | Sehr große Tabelle, nur der Anfang wurde gelesen. Eine selbst addierte Summe ist dann unvollständig |
| `time budget exhausted …` (`sheet read in part`, `while reading the workbook`, `while reading the HTML tables`, `while reading the XML spreadsheet`) | Das Zeitlimit von 90 Sekunden war erreicht, die Tabelle wurde nur teilweise gelesen |
| `sheet '…': part not found or damaged` / `zip entry '…' is damaged` (auch `is encrypted`, `is too large (…)`, `uses the unsupported compression method …`, `is truncated or damaged (read in part)`, `could not be decompressed: …`) | Ein Teil der Excel-Datei fehlt oder ist beschädigt; die übrigen Blätter wurden gelesen |
| `spreadsheet reader stopped: …` | Interner Fehler beim Lesen der Tabelle, für die Person, die das Skript pflegt |
| `automatic reply` / `read receipt` / `delivery report` | Die Mail ist eine automatische Antwort (Abwesenheitsnotiz, „Automatische Antwort:", Kopf `Auto-Submitted: auto-replied`), eine Lesebestätigung („Gelesen:") oder eine Unzustellbarkeitsmeldung. Sie ist kein Avis und bekommt keine Kategorie, auch wenn „Zahlungsavis" im Betreff steht |
| `mailing list` | Newsletter oder Verteiler (Kopf `List-Unsubscribe`, `List-Id` oder `Precedence: bulk`). Nur ein Dokument mit Avis-Titel oder Bankformular zählt dann noch als Avis |
| `… mail parts not read` | Teile der Mail wurden nicht gelesen (siehe `mail parts not read` oben) |

| Urteil (`verdictWhy`, englisch) | Bedeutung |
| --- | --- |
| `invoice features (…) beside payment label '…' - not a payment advice?` / `payment label '…', but invoice features (…)` | Ein Dokument ohne Avis-Titel endet mit „Zahlbetrag", hat aber Merkmale einer Rechnung, Quittung oder eines Leistungsnachweises (Positionstabelle, Umsatzsteuerzeile, „bezahlt mit girocard", Titel „Leistungsnachweis"). Zwei Merkmale: kein Avis; ein Merkmal: „Avis prüfen" |
| `…, but the mail text speaks against an advice (…)` | Der Mailtext ist eine Reklamation, Mahnung, Bestellung, ein Zahlungsplan oder das Begleitschreiben einer Rechnung („anbei erhalten Sie unsere Rechnung"). Eine Anlage, die nur eine Zahlbetrag-Spalte oder -Zeile hat, gilt dann nicht als Avis („Avis prüfen") |
| `column '…' with its total, but …` | Tabelle mit Zahlbetrag-Spalte und Summe, aber mit Fälligkeitsspalte, Ratenzeilen oder einem Namen wie Zahlungsplan, OP-Liste, Abstimmung: kein sicheres Avis („Avis prüfen") |
| `payment announced for later ('…')` | Der Schreiber kündigt eine Zahlung nur an („wird nach Prüfung überwiesen", „sobald", „nächste Woche"), und der Betrag ist unsicher: „Avis prüfen" ohne Betrag |
| `payer statement …, but …` (z. B. `holiday notice`) | Eine Zahlungserklärung neben einem Hinweis wie Betriebsferien, neuer Bankverbindung oder einer Reklamation: „Avis prüfen", nie stillschweigend „kein Avis" |
| `bank transfer confirmation? ('Auftragsbestätigung' …)` | Eine Bank-„Auftragsbestätigung" mit Auftraggeber, Empfänger, IBAN und Betrag: wahrscheinlich eine Überweisung, „Avis prüfen" |
| `payment word '…' in the subject, but no advice found` | „Zahlung" oder „Überweisung" im Betreff, dazu ein Betrag im Mailtext oder ein Foto als Anlage, aber kein erkennbares Avis: „Avis prüfen" |
| `automatic reply (…)` / `read receipt (…)` / `mailing list (…)` | Automatische Antwort, Lesebestätigung oder Newsletter: keine Kategorie |

**Aus dem Flow selbst (nicht vom Skript)**

| Was du siehst | Bedeutung |
| --- | --- |
| Kategorie „Avis prüfen", im Ausführungsverlauf ist **Run script** rot | Das Skript ist auch nach 3 Wiederholungen gescheitert; die Meldung von Microsoft steht im Verlauf bei Run script. Mail von Hand bearbeiten |
| Kategorie „Avis prüfen", Run script mit **Zeitüberschreitung** | Run script hat länger als 120 Sekunden gebraucht (sehr große Mail). Mail von Hand bearbeiten |
| Keine Kategorie, Lauf endet bei **Keine Automatik** | Der Betreff enthielt einen der Texte aus Keine_Automatik (Abwesenheitsnotiz, Zustellmeldung) |

**Weitere Hinweise (Feld `warnings`)**

| Hinweis (englisch) | Bedeutung |
| --- | --- |
| leer | Alles normal |
| `from signed mail: …` | Der Betrag stammt aus dieser PDF in der signierten Mail (`from mail: …` bei einer weitergeleiteten Mail-Datei) |
| `… PDFs checked (…)` / `only … of … PDFs checked` | Die Mail enthielt mehrere PDFs, das Skript hat die genannten geprüft und die beste genommen. Hat eine PDF „Zahlungsavis" oder „Avis" als eigenes Wort im Namen (nicht „Lieferavis"), prüft es nur diese |
| `only the first 50 of … PDFs read` | Die Mail enthielt mehr als 50 PDFs. Das Skript hat nur die ersten 50 angesehen |
| `PDF arrived as text instead of Base64` | Power Automate hat die PDF als Text übergeben, sie ließ sich aber vollständig zurückwandeln. Der Betrag stimmt; trotzdem das Feld pdfBase64 in Run_script auf fx `base64(body('Export_email'))` umstellen (Schritt 5.4), sonst gehen bei anderen PDFs Teile verloren |
| `decrypted (permissions-only encryption)` | Die PDF war nur gegen Kopieren oder Drucken geschützt und wurde normal gelesen |
| `OCR text layer (… % of the text)` | Text aus Texterkennung. Solche Avise bekommen nie „Avise" ohne Prüfung |
| `incomplete: …` | PDF teilweise gelesen. Kommt bei „Avise" ebenfalls nicht vor |
| andere Texte (z. B. `page 2: …`) | Technische Details zum Lesen der PDF, für die Person, die das Skript pflegt |
