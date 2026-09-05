# FCG09 Sponsor-Rondell

Produktive Sponsor-Anzeige für den Kabinengang des **FC Germania 09 Großkrotzenburg**.

## Zielbild

Die Sponsor-Inhalte werden außerhalb des Raspberry gepflegt und automatisch veröffentlicht:

`Glide Sponsorverwaltung -> Sponsor-Sheet (Google Sheets) -> GitHub Pages -> Chromium Sponsor-Tab`

Der Raspberry zeigt parallel den Platzmanager und das Sponsor-Rondell in **zwei dauerhaft geöffneten Chromium-Tabs**. Der Tab-Wechsel erfolgt extern über die Chromium DevTools-Schnittstelle (`127.0.0.1:9222`), nicht mehr über eine Browser-Extension.

## Sponsor-Daten

Die Seite liest die veröffentlichte Sponsor-CSV aus Google Sheets (`gid=2113186516`). Berücksichtigt werden nur aktive und aktuell gültige Sponsoren.

Relevante Felder u. a.:

- `SponsorID`
- `Sponsor`
- `Aktiv`
- `GueltigVon` / `GültigVon`
- `GueltigBis` / `GültigBis`
- `Logo` / `LogoURL`
- `AnzeigeLeitsatz`
- `Kategorie`
- `AnzeigedauerSek`
- `Reihenfolge`

Standard-Anzeigedauer bei fehlendem Wert: **8 Sekunden**.

## Blocksteuerung

Die URL unterstützt:

- `start=<Index>`
- `count=<Anzahl>`

Beispiel:

`?start=3&count=3`

Damit kann der Raspberry je Werbeblock gezielt drei aufeinanderfolgende Sponsoren anzeigen und beim nächsten Block mit dem nächsten Index fortsetzen.

Gewünschte produktive Folge bei `count=3`:

`1-2-3 -> Platzmanager -> 4-5-6 -> Platzmanager -> 7-8-9 -> Platzmanager -> ...`

Der Startindex wird auf Raspberry-Seite erst **nach einem tatsächlich ausgespielten Block** erhöht. Dadurch wird kein Sponsorblock doppelt gezählt oder übersprungen.

## Sichtbarkeitslogik

Wichtig für den Zwei-Tab-Kiosk:

- Der Sponsor-Tab darf im Hintergrund vollständig laden.
- Der eigentliche Sponsor-Block startet erst, wenn der Tab wirklich sichtbar ist.
- Eine kurze Sichtbarkeitsprüfung verhindert, dass das automatische Aktivieren eines neu erzeugten Tabs bereits als Sponsorblock gewertet wird.
- Beim Wechsel zurück zum Platzmanager werden laufende Timer gestoppt.
- Beim nächsten Sichtbarwerden beginnt der durch `start`/`count` definierte Block sauber neu.

Dadurch werden keine 8-Sekunden-Intervalle im Hintergrund verbraucht.

## Timing

Zur reinen Sponsorzeit kommen kleine technische Übergänge hinzu. Der Raspberry berücksichtigt deshalb zusätzlich:

- ca. `0,5 s` Sichtbarkeits-Sicherheitszeit
- `0,35 s` Übergang zwischen zwei Sponsoren
- kleiner Abschluss-Puffer

Bei drei Sponsoren ergibt sich damit ein kompletter Block von ungefähr **25,4 Sekunden**. Dadurch erhält auch der letzte Sponsor seine volle Anzeigezeit und wird nicht 1–2 Sekunden zu früh abgeschnitten.

## Raspberry-Integration

Produktiver Ablauf:

1. Platzmanager-Tab ist sichtbar.
2. Sponsor-Tab wird im Hintergrund für den nächsten Block vorbereitet.
3. Nach der zentral konfigurierten Platzmanager-Dauer wird der Sponsor-Tab aktiviert.
4. Sponsor 1, 2 und 3 laufen gemäß `AnzeigedauerSek` (standardmäßig 8 s).
5. Nach Ablauf des Blocks aktiviert der Raspberry wieder den Platzmanager-Tab.
6. Erst jetzt wird der Startindex fortgeschrieben.
7. Der nächste Sponsor-Block verwendet dadurch den nächsten eindeutigen Block.

Die zentrale Kiosk-Konfiguration (`SponsorRondellAktiv`, `PlatzmanagerDauerSek`, `SponsorenProBlock`, `AktualisierungSek`, `RondellURL`, `PlatzmanagerURL`, `NurPlatzmanager`) kommt weiterhin aus Glide/Google Sheets.

## Technische Entscheidung 05.09.2026

Verworfen wurden:

- iframe-Einbettung des Glide-Platzmanagers (durch Glide/CSP blockiert)
- Tab-Wechsel per Chromium Manifest-V3-Extension (Race Conditions / Service-Worker-Verhalten)
- `xdotool` (unter Wayland/labwc nicht zuverlässig)
- Navigation eines einzelnen Tabs (zu lange Ladezeit des Platzmanagers)

Produktiv verwendet werden deshalb **zwei vorgeladene Tabs + DevTools-Target-Aktivierung**.

## Status

Stand 05.09.2026: produktiv getestet. Sponsorblöcke laufen fortlaufend ohne Wiederholung des vorherigen Blocks, der Rücksprung zum Platzmanager funktioniert stabil, und der Timing-Puffer stellt die volle Anzeigezeit des letzten Sponsors sicher.
