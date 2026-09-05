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

## Sichtbarkeitslogik

Wichtig für den Zwei-Tab-Kiosk:

- Der Sponsor-Tab darf im Hintergrund vollständig laden.
- Der eigentliche Sponsor-Block startet erst, wenn der Tab sichtbar wird.
- Beim Wechsel zurück zum Platzmanager werden laufende Timer gestoppt.
- Beim nächsten Sichtbarwerden beginnt der durch `start`/`count` definierte Block sauber neu.

Dadurch werden keine 8-Sekunden-Intervalle im Hintergrund verbraucht.

## Raspberry-Integration

Produktiver Ablauf:

1. Platzmanager-Tab ist sichtbar.
2. Sponsor-Tab wird im Hintergrund für den nächsten Block vorbereitet.
3. Nach der zentral konfigurierten Platzmanager-Dauer wird der Sponsor-Tab aktiviert.
4. Sponsor 1, 2 und 3 laufen gemäß `AnzeigedauerSek` (standardmäßig 8 s).
5. Nach Ablauf des Blocks aktiviert der Raspberry wieder den Platzmanager-Tab.
6. Der nächste Sponsor-Block verwendet den fortgeschriebenen `start`-Index.

Die zentrale Kiosk-Konfiguration (`SponsorRondellAktiv`, `PlatzmanagerDauerSek`, `SponsorenProBlock`, `AktualisierungSek`, `RondellURL`, `PlatzmanagerURL`, `NurPlatzmanager`) kommt weiterhin aus Glide/Google Sheets.

## Technische Entscheidung 05.09.2026

Verworfen wurden:

- iframe-Einbettung des Glide-Platzmanagers (durch Glide/CSP blockiert)
- Tab-Wechsel per Chromium Manifest-V3-Extension (Race Conditions / Service-Worker-Verhalten)
- `xdotool` (unter Wayland/labwc nicht zuverlässig)
- Navigation eines einzelnen Tabs (zu lange Ladezeit des Platzmanagers)

Produktiv verwendet werden deshalb **zwei vorgeladene Tabs + DevTools-Target-Aktivierung**.

## Status

Stand 05.09.2026: Raspberry-Test erfolgreich. Sponsorblöcke und Rücksprung zum Platzmanager laufen stabil; Sichtbarkeitsstart ist im Rondell implementiert.