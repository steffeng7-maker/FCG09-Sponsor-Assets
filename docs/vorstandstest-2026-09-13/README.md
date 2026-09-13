# Vorstandstest – Premium Platzmanager

Stand: 13.09.2026

## Ziel

Separater Vorstands-/Teststand für die Premium-Ansicht des FCG09 Platzmanagers. Die produktive Live-Ansicht und das bestehende Sponsor-Rondell werden **nicht** ersetzt oder verändert. Die Premium-Version wird in Glide über eine eigene Testspalte/-seite geführt.

## Designstand

- freundlicher Stadion-/Rasenhintergrund statt komplett schwarzer Fläche
- dunkle, transparente Glasflächen für gute TV-Lesbarkeit
- beide Plätze bleiben in der Kabinengang-Ansicht auf einem Bildschirm sichtbar
- Platz-Icons zur schnellen Unterscheidung: Hybridplatz / Rasenplatz
- Orientierung bleibt erhalten (`Spielplatz`, `Tribüne`, `Angler`, linke/rechte Orientierung)
- Q1–Q4 bleiben als Spielfeldstruktur mit Torlinien sichtbar
- reduzierte Farbigkeit; Statusfarbe nur noch dezent über `AnzeigeIcon`
- Teamname ca. 26 px, Team-Icon klein (ca. 18 px)
- Terminart, Gegner, Wettbewerb, Zeit und Torwartstatus bleiben erhalten
- nächste Belegung bleibt erhalten, wird aber nur gezeigt wenn tatsächlich vorhanden
- Statuslegende wurde aus der Premium-Ansicht entfernt
- Kabinenbereich rechts bleibt erhalten, Kabinenzeit wurde aus der Anzeige entfernt; Anzeige aktuell kompakt `K1  G1-Jugend`
- Sperr-Overlay Premium ist vollständig von der Live-Sperranzeige getrennt

## Glide-Logiken, die im Zuge des Premium-Tests ergänzt/angepasst wurden

### Zonen – `Anzeige Team`

Vorher bei leerem aktiven Team: `---`.

Neu:

- IF `Aktives Team` leer → `FREI`
- ELSE → `TeamName`

Ergebnis: freie Viertel werden bewusst als `FREI` dargestellt.

### Zonen – `Anzeige Zeit`

Problem: `Aktive Zeit` ist ein Template `{Aktive Von} - {Aktive Bis}` und erzeugt bei fehlenden Werten weiterhin einen Bindestrich. Lookup-Empty-Prüfungen waren nicht stabil genug.

Bereinigte Anzeige:

- bei keiner aktiven Belegung → leer
- sonst → `Aktive Zeit`

Im Test wurde dafür die bestehende Aktiv-Logik (`AktivFlag`) als stabiler Trigger verwendet.

### Zonen – `NächsteTeamAnzeige Clean`

Neue If-Then-Else-Spalte:

- IF `NaechstesTeam` leer → leer
- ELSE → `NächsteTeamAnzeige`

Danach wurden die Single-Value-Spalten `Q1 NaechstesTeam`, `Q2 NaechstesTeam`, `Q3 NaechstesTeam`, `Q4 NaechstesTeam` auf `NächsteTeamAnzeige Clean` umgestellt.

Ergebnis:

- Hybridplatz ohne Folgebelegung: keine leere Beschriftung `Nächste:` mehr
- Rasenplatz mit Folgebelegung: z. B. `Nächste: G1 09:00` bleibt sichtbar

### Kabinenanzeige

Die neue Kabinenlogik bleibt erhalten. Für die Premium-Anzeige wurde die Darstellung vereinfacht:

- Uhrzeit in der Kabinenübersicht entfernt
- Kabine dezent, Teamname stärker
- Beispiel: `K1  G1-Jugend`

Die Daten kommen weiterhin über `{Kabinen HTML Liste}`.

### Premium-Sperrlogik

Produktive Spalten bleiben unverändert:

- `Sperr Overlay HTML`
- `Sperr Overlay Anzeige`

Neu für Premium:

- `Sperr Overlay HTML Premium`
- `Sperr Overlay Anzeige Premium`

`Sperr Overlay Anzeige Premium`:

- IF `Heutige Sperre Grund` is not empty
- THEN `Sperr Overlay HTML Premium`
- ELSE leer

Das Premium-Platz-HTML verwendet ausschließlich `{Sperr Overlay Anzeige Premium}`.

## Wichtige Platzhalter im Premium-HTML

- `{Platz Icon}`
- `{Platzname}`
- `{Auslastung}` / `{AuslastungFarbe}`
- `{ObenOrientierung}` / `{UntenOrientierung}` / `{LinkeOrientierung}` / `{RechteOrientierung}`
- `{Sperr Overlay Anzeige Premium}`
- Q1–Q4: Team, Team Icon, Terminart, Gegner, Wettbewerb, Zeit, StatusTorwart, NaechstesTeam, AnzeigeIcon
- `{Kabinen HTML Liste}`

## Bekannter offener Punkt

`AnzeigeIcon` liefert bei `Frei` aktuell noch den weißen Statuspunkt. In der Premium-Ansicht ist das deutlich kleiner, wurde aber absichtlich nicht per HTML entfernt, damit Aktiv/Konflikt/Gesperrt nicht beschädigt werden. Falls gewünscht, später gezielt in der Glide-Logik lösen: Frei → leer, Aktiv → gelb, Konflikt → rot, Gesperrt → Warnsymbol/Overlay.

## Dateien dieser Version

- `premium-platz-html.html` – aktueller Premium-Polish Platz-HTML-Stand
- `premium-sperr-overlay.html` – separates Premium-Sperr-Overlay
- `kabinen-template.html` – kompakte Kabinenzeile
- `screenshots/01-premium-overview.jpg` – freundlicher Premium-Gesamtstand
- `screenshots/02-premium-sperre.jpg` – Premium-Sperrzustand

## Screenshots

### Premium Gesamtstand

![Premium Übersicht](screenshots/01-premium-overview.jpg)

### Premium Sperre

![Premium Sperre](screenshots/02-premium-sperre.jpg)

## Betriebsprinzip

Der Raspberry-/Kabinengang-Ablauf bleibt wie heute: Platzmanager auf einem Bildschirm → anschließend Fade/Wechsel zum Sponsor-Rondell → zurück zum Platzmanager. Diese Vorstands-Testversion dokumentiert nur die neue Premium-Darstellung und die dazugehörigen Glide-Anzeigelogiken.
