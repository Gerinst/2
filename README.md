# Avis-Flow Version 2

Erkennt in einem freigegebenen Postfach jede Zahlungsavis-Mail am Inhalt: PDF, Excel- und CSV-Dateien, SAP-Exporte oder ein Avis direkt im Mailtext. Die **Original-Mail** bekommt die Kategorie „Avise" und den Zahlbetrag hinten an den Betreff (`Zahlungsavis - 2.601,35 EUR`). Rechnungen, Reklamationen, Bestellungen und alle anderen Mails bleiben unberührt. Es kommen keine zusätzlichen Mails. Das Skript läuft als Office Script über Excel Online (Business) → „Skript ausführen": Standard-Connector, keine AI-Builder-Credits, keine Drittanbieter.

## Diese Dateien brauchst du

| Datei | Wofür |
|---|---|
| [docs/ANLEITUNG.md](docs/ANLEITUNG.md) | **Hier anfangen.** Schritt für Schritt: Rechte auf das Postfach, Kategorien, Skript in Excel, 5-Minuten-Test, Flow, Testen, Fehlertabelle und alle Gründe des Skripts auf Deutsch (Anhang B). |
| [dist/AvisV2.ts](dist/AvisV2.ts) | Das Skript für Excel. Datei öffnen, oben rechts auf **Copy raw file** klicken (Symbol mit den zwei Rechtecken) und in Excel einfügen (Anleitung, Schritt 3). |
| [formeln/](formeln) | Alle Formeln des Flows als TXT zum Kopieren: [URI_Kategorie_und_Betreff.txt](formeln/URI_Kategorie_und_Betreff.txt), [Run_script_pdfBase64.txt](formeln/Run_script_pdfBase64.txt), [Kategorie_noetig_links.txt](formeln/Kategorie_noetig_links.txt), [Kategorie_setzen_Text.txt](formeln/Kategorie_setzen_Text.txt), [Betreff_noetig_links.txt](formeln/Betreff_noetig_links.txt), [Betreff_setzen_Text.txt](formeln/Betreff_setzen_Text.txt). Datei öffnen, **Copy raw file**, im Feld über **fx** einfügen. In der URI `avis@firma.de` durch eure Postfachadresse ersetzen. |
| [dist/avis-flow.html](dist/avis-flow.html) | Übersicht aller Flow-Aktionen und Felder auf einer Seite, mit Plan B. GitHub zeigt die Datei nur als Code: oben rechts auf **Download raw file** klicken und die heruntergeladene Datei per Doppelklick im Browser öffnen. |

## Gut zu wissen

- **Live getestet** am 30.10.2026 in einem Microsoft-365-Testzugang mit freigegebenem Postfach, mit erfundenen Test-Mails:
  - **Lasttest:** 40 Mails in unter 2 Minuten, davon 15 Avise (eines mit 72 Seiten und 2,7 Mio. EUR, eines mit 27 Seiten und Zwischensummen je Filiale, Excel, Mailtext, Gutschriftsanzeige der Bank, Englisch, Französisch, Schweizer Franken, Weiterleitung, Scan, zwei Avise in einer Mail, ein anderer Betrag im Mailtext) und 25 Fallen (Rechnungen, Gutschriften, Mahnungen, Kontoauszug, Saldenbestätigung, Lastschrift-Ankündigung, Lieferavis, Bestellung, Reklamation …). Jede Mail wurde verarbeitet, 37 genau wie erwartet, 3 vertretbar (eine Quittung und eine englische Rechnung bekamen „Avis prüfen", eine Antwort auf ein Avis blieb unberührt), keine Falle bekam „Avise", kein Avis ging verloren.
  - **Viele Anlagen:** 20 Rechnungen ohne Avis bleiben unberührt; ein Avis als 20. Anlage wird gefunden.
  - Dazu zwei frühere Runden mit 32 Mails.
- **Kategorien:** „Avise" = Betrag steht im Betreff. Endet er auf „(prüfen)", ist sich das Skript nur ziemlich sicher. „Avis prüfen" = unsicherer Betrag mit „?" oder ein Avis, das sich nicht lesen ließ. „mehrere Avise" = zwei oder mehr Avise mit verschiedenen Beträgen; der Betreff bleibt dann unverändert.
- **Nicht lesbar** (bekommen „Avis prüfen", wenn ein Avis-Hinweis da ist): reine Scans ohne Text, PDFs mit Kennwort, verschlüsselte oder mit Microsoft Purview geschützte Mails, altes Excel-Format .xls, Word-Dateien. Bilder als Anlage werden nicht gelesen.
- **Ab 31.12.2026** will Microsoft das Ändern des Betreffs empfangener Mails sperren. Kategorien sind nicht betroffen. Im Januar den 5-Minuten-Test wiederholen; greift die Sperre, auf Plan B umstellen (Anleitung, Abschnitt „Ab Januar 2027").
- **Voraussetzung:** ein Exchange-Postfach, das als freigegebenes Postfach eingerichtet ist (kein Microsoft-365-Gruppenpostfach), und Vollzugriff für das Konto, unter dem der Flow läuft.
- **Limits von Microsoft:** „Skript ausführen" höchstens 3-mal in 10 Sekunden und 1.600-mal pro Tag und Nutzer; der Flow arbeitet die Mails deshalb nacheinander ab (im Lasttest 8 bis 20 Sekunden je Mail, auch bei 72 Seiten). Je Mail werden höchstens 30 Anlagen gelesen.
- Version 1 (Ergebnis-Mails für PDFs mit „avis" im Namen oder Betreff): [Gerinst/1](https://github.com/Gerinst/1).
