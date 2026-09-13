# Glide-Logik – Vorstandstest Premium Platzmanager

Stand: 13.09.2026

Dieses Dokument hält bewusst auch Logiken fest, die aus Glide-Screenshots rekonstruiert wurden. Damit sind die Konfigurationen nicht nur im Chat sichtbar, sondern dauerhaft im Repository nachvollziehbar.

## Grundprinzip

Die produktive Live-Ansicht bleibt unangetastet. Premium-spezifische Darstellung wird über zusätzliche Anzeige-/Clean-Spalten und separate Premium-HTML-Spalten realisiert.

## Tabelle `Plaetze`

### `Q1 Query`
- Typ: Query
- Source: `Zonen`
- Filter 1: `PlatzID` is `This row -> PlatzID`
- Filter 2: `Zone` is `Q1`
- Sortierung: `Table order`
- `Match multiple`: aktiv
- gezeigtes Limit im Screenshot: 1000

Q2–Q4 folgen demselben Muster mit der jeweiligen Zone.

### `Q1 Team`
- Typ: Single Value
- Get: `First`
- From: `Q1 Query -> Anzeige Team`

### `Q1 Zeit`
- Typ: Single Value
- Get: `First`
- From: `Q1 Query -> Anzeige Zeit`

### `Q1 NaechstesTeam`
- Typ: Single Value
- Get: `First`
- From: `Q1 Query -> NächsteTeamAnzeige Clean`

Q2–Q4 wurden analog auf `NächsteTeamAnzeige Clean` umgestellt.

### `Q1 Icon`
- Typ: Single Value
- Get: `First`
- From: `Q1 Query -> AnzeigeIcon`

### `Kabinen HTML Liste`
- Typ: Joined List
- Join the texts in: `Kabinen Anzeige`
- Separator: leer

### `Kabinen HTML Liste Clean`
Neue Premium-/Clean-Ausgabe:
- IF `Kabinen HTML Liste` is empty
- THEN: dezentes HTML `Keine Belegung`
- ELSE: `Kabinen HTML Liste`

Verwendetes Fallback-HTML siehe `kabinen-fallback.html`.

### `Sperr Overlay Anzeige Premium`
- Typ: If → Then → Else
- IF `Heutige Sperre Grund` is not empty
- THEN `Sperr Overlay HTML Premium`
- ELSE leer

Das Premium-Platz-HTML referenziert ausschließlich `{Sperr Overlay Anzeige Premium}`. Die produktiven Spalten `Sperr Overlay HTML` / `Sperr Overlay Anzeige` bleiben erhalten.

## Tabelle `Zonen`

### `Anzeige Team`
- Typ: If → Then → Else
- IF `Aktives Team` is empty
- THEN `FREI`
- ELSE `TeamName`

Vorher wurde `---` ausgegeben. Die Änderung ist reine Darstellung und greift auch in der bestehenden Ansicht sauber.

### `Aktive Zeit`
- Typ: Template
- Template: `{Aktive Von} - {Aktive Bis}`
- `{Aktive Von}` kommt aus Lookup `AktivVon`
- `{Aktive Bis}` analog aus `AktivBis`

Wichtig: Sind beide Lookups leer, bleibt durch das feste Trennzeichen trotzdem ein `-` übrig. Deshalb darf `Anzeige Zeit` nicht nur `Aktive Zeit is empty` prüfen.

### `AktivVon`
- Typ: Lookup
- Relation column: `Aktive Belegung Query -> VonZeit`

### `Anzeige Zeit`
Bereinigte Darstellung:
- IF `AktivFlag = 0`
- THEN leer
- ELSE `Aktive Zeit`

Grund: Lookup-/Template-Werte verhalten sich in Glide nicht zuverlässig wie echter leerer Text.

### `Nächste Belegung Query`
- Typ: Query
- Source: `Platzbelegungen`
- Filter:
  - `ZoneID` is aktuelle Zonen-ID
  - `WochentagNr` equals `Heute Woch...`
  - `VonMinuten` > `Jetzt Minute`
  - `Heute KW` is `true`
- Sortierung: `VonMinuten` aufsteigend
- `Match multiple`: aktiv
- Limit: `1`

Zweck: pro Zone genau die nächste zukünftige Belegung des heutigen Tages ermitteln.

### `NaechstesTeam`
- Typ: Single Value
- Get: `First`
- From: `Nächste Belegung Query -> TeamID`

### `NächsteTeamAnzeige`
- Typ: Template
- Template: `Nächste: Team Zeit`
- Team-Replacement: `NaechstesTeam`
- Zeit-Replacement: `NaechstesTeamZeit`

Problem: ohne nächste Belegung bleibt der feste Text `Nächste:` stehen.

### `NächsteTeamAnzeige Clean`
Neue If → Then → Else-Spalte:
- IF `NaechstesTeam` is empty
- THEN leer
- ELSE `NächsteTeamAnzeige`

Ergebnis:
- ohne Folgebelegung: komplett leer
- mit Folgebelegung: z. B. `Nächste: G1 09:00`

### `AnzeigeIcon`
Aktueller Stand:
- Frei liefert noch einen weißen/grauen Punkt
- Aktiv liefert gelben Punkt
- Konflikt/Gesperrt haben eigene Statusdarstellung

Bewusster offener Punkt: Frei später optional auf leer setzen, ohne die Warnzustände zu beschädigen.

## Kabinenanzeige

Die Kabinenzeit wurde in der Premium-Ansicht bewusst entfernt. Kerninformation ist die Zuordnung, z. B.:
- `K1  G1-Jugend`
- `K4  G1-Jugend`

Kabinen-ID wird dezent dargestellt, Teamname stärker. Wenn die komplette Joined List leer ist, zeigt `Kabinen HTML Liste Clean` einmalig `Keine Belegung`.

## Premium-Sperr-Overlay

### `Sperr Overlay HTML Premium`
Replacements wie in der bestehenden Sperrlogik:
- `{Heutige Sperre Grund}`
- `{Heutige Sperre VonZeit}`
- `{Heutige Sperre BisZeit}`

Optik: dunkles Glas, rote Akzentkante links, klare Überschrift `PLATZ GESPERRT`, Grund neutral, Zeit als dezente Pill.

## Screenshot-Dokumentationsregel für dieses Projekt

Wenn zukünftig Glide-Screenshots mit Tabellen-/Spaltenkonfigurationen geteilt werden, sollen relevante Konfigurationen in diesem Dokument textlich nachgezogen werden. Wichtige Referenz-Screenshots können zusätzlich unter `screenshots/glide-logic/` gespeichert werden. So bleibt die Logik auch nach dem Chat dauerhaft in GitHub verfügbar und kann bei späteren Änderungen als Referenz verwendet werden.
