# Raspberry Kabinengang – Produktivstand 07.09.2026

## Status

Die automatische Kabinengang-Anzeige funktioniert mit der Architektur:

`Glide / Google Sheets -> Python Switch auf Raspberry -> Chromium DevTools Port 9222 -> Platzmanager / Sponsor-Rondell`

Der Wechsel über die Chromium-Extension wurde verworfen. Die Steuerung erfolgt über `/home/kiosk/kabinengang-switch.py` und den systemd-User-Service `kabinengang.service`.

Erfolgreich getestet am 07.09.2026:

1. Platzmanager wird im Kiosk angezeigt.
2. Sponsorblock wird im bestehenden Sponsor-Tab vorbereitet.
3. Nach `PlatzmanagerDauerSek` wird der Sponsor-Tab aktiviert.
4. Individuelle `AnzeigedauerSek` je Sponsor wird aus dem Sponsor-CSV gelesen und für den Block aufsummiert.
5. Nach dem Sponsorblock wird wieder der Platzmanager aktiviert.
6. Der nächste Block beginnt beim nächsten Sponsor (`start=0`, danach `start=3`, usw.).
7. Sponsor-Fortschritt wird lokal persistent gespeichert.
8. `Aktiv`, `GueltigVon` und `GueltigBis` werden bei der Sponsor-Auswahl berücksichtigt.
9. Bei Fehlern versucht die Steuerung als Fallback den Platzmanager anzuzeigen.

## Raspberry / Kiosk

- Hostname: `kabinengang`
- SSH-User: `kiosk`
- zuletzt verwendete IP: `192.168.178.95` (kann sich ändern)
- SSH: `ssh kiosk@192.168.178.95`
- Chromium: `/usr/bin/chromium`
- Python: Python 3.13.x
- lokaler Chromium DevTools-Port: `127.0.0.1:9222`

DevTools-Test:

```bash
curl -s http://127.0.0.1:9222/json/version
```

Python-WebSocket-Unterstützung ist installiert (`python3-websocket`) und wurde erfolgreich gegen Chromium getestet.

## Systemd-Autostart

Service:

`/home/kiosk/.config/systemd/user/kabinengang.service`

```ini
[Unit]
Description=Kabinengang Platzmanager und Sponsoren
After=graphical-session.target

[Service]
ExecStart=/home/kiosk/kabinengang-start.sh
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Der Service ist aktiviert und startet automatisch.

Status prüfen:

```bash
systemctl --user --no-pager --full status kabinengang.service
```

Neustart:

```bash
systemctl --user restart kabinengang.service
```

## Startskript

Datei:

`/home/kiosk/kabinengang-start.sh`

Ablauf:

1. wartet 30 Sekunden auf den Desktop,
2. beendet ggf. altes Chromium,
3. startet Chromium im Kiosk-Modus mit DevTools-Port 9222,
4. öffnet Platzmanager und Sponsor-Rondell als Tabs,
5. wartet 10 Sekunden,
6. startet `/home/kiosk/kabinengang-switch.py`.

Chromium wird u. a. mit folgenden Parametern gestartet:

```text
--remote-debugging-port=9222
--remote-debugging-address=127.0.0.1
--kiosk
--password-store=basic
--noerrdialogs
--disable-infobars
--no-first-run
--disable-session-crashed-bubble
--start-maximized
```

## Python Switch

Produktive Steuerdatei:

`/home/kiosk/kabinengang-switch.py`

Backup vor der Überarbeitung vom 07.09.2026:

`/home/kiosk/kabinengang-switch.py.backup-2026-09-07`

Wichtig: Es soll nur **eine** Switch-Steuerung laufen. Keinen zusätzlichen zweiten Controller parallel starten.

Prozessprüfung:

```bash
ps aux | grep '[k]abinengang-switch.py'
```

### Tab-Steuerung

Die stabile Version behält Platzmanager- und Sponsor-Tab bestehen. Sponsor-Tabs werden nicht mehr bei jedem Zyklus geschlossen und neu erzeugt.

Die Steuerung nutzt Chromium DevTools/WebSocket für:

- `Page.bringToFront` zum sichtbaren Umschalten,
- `Page.navigate` zum Vorbereiten des nächsten Sponsorblocks.

Diese Änderung wurde vorgenommen, nachdem die alte `close -> /json/new -> activate`-Logik zu instabilem bzw. schwarzem Bildschirm geführt hatte.

## Platzmanager

URL:

`https://fcg09-platzmanager.glide.page/dl/9cec69`

Der Platzmanager wird für den Wert `PlatzmanagerDauerSek` angezeigt. Im Test am 07.09.2026 waren dies 60 Sekunden.

## Sponsor-Rondell

GitHub Pages:

`https://steffeng7-maker.github.io/FCG09-Sponsor-Assets/`

Branch für die aktuelle Rondell-Version:

`sponsor-rondell-v1e`

Unterstützte URL-Parameter:

- `start` = Index des ersten Sponsors im Block
- `count` = Anzahl Sponsoren im Block

Beispiel:

`https://steffeng7-maker.github.io/FCG09-Sponsor-Assets/?start=0&count=3`

Danach wird bei drei Sponsoren der nächste Block mit `start=3&count=3` vorbereitet.

## Individuelle Sponsor-Anzeigedauer

Die frühere feste Berechnung `SponsorenProBlock * 8 Sekunden` ist entfernt.

Der Switch liest `AnzeigedauerSek` für jeden tatsächlich ausgewählten Sponsor aus dem Sponsor-CSV. Mindestdauer ist 3 Sekunden, Standardwert bei fehlendem/ungültigem Wert ist 8 Sekunden.

Erfolgreicher Test am 07.09.2026:

```text
Güven Pflegedienst GmbH => 30.0 Sekunden
Selgros Cash & Carry => 30.0 Sekunden
ATZ GmbH => 30.0 Sekunden
Reine Sponsorzeit: 90.0 Sekunden
Gesamter Sponsorblock: 91.2 Sekunden
```

Die zusätzliche Blockzeit berücksichtigt die Übergänge des Rondells (ca. 0,35 Sekunden zwischen Sponsoren) und eine kleine Sicherheitsreserve.

Damit gilt z. B. bei 30 + 8 + 8 Sekunden eine reine Sponsorzeit von 46 Sekunden.

## Sponsor-Auswahl: Aktiv und Gültigkeit

Ein Sponsor wird vom Raspberry nur berücksichtigt, wenn:

- `Aktiv = TRUE`,
- der Sponsorname nicht leer ist,
- `GueltigVon` leer ist oder das Datum erreicht wurde,
- `GueltigBis` leer ist oder das Datum noch nicht überschritten wurde.

`GueltigBis` gilt bis einschließlich 23:59:59 des angegebenen Tages.

Glide/Google Sheets liefert Datumswerte z. B. als:

```text
2026-09-07T00:00:00.000Z
```

Die Funktion `parse_date()` im Switch unterstützt dieses ISO-Format sowie `YYYY-MM-DD`, `DD.MM.YYYY` und `DD/MM/YYYY`.

Erfolgreicher End-to-End-Test am 07.09.2026:

```text
Güven Pflegedienst GmbH
Aktiv: TRUE
GueltigVon: 2026-09-07T00:00:00.000Z
GueltigBis: 2026-09-08T00:00:00.000Z
```

Der Sponsor wurde für den 07.09.2026 korrekt als aktuell gültig erkannt.

## Sponsor-Fortschritt

Der aktuelle Startindex wird persistent gespeichert in:

`/home/kiosk/.kabinengang-state.json`

Nach einem tatsächlich ausgespielten Sponsorblock wird der Index um `SponsorenProBlock` erhöht und modulo der aktuell gültigen Sponsorenanzahl weitergeführt.

Damit ergibt sich bei 3 Sponsoren pro Block:

`1–3 -> 4–6 -> 7–9 -> ... -> nach dem letzten Sponsor wieder von vorne`

## Google-Sheets-Fernsteuerung

Konfigurations-CSV:

`https://docs.google.com/spreadsheets/d/e/2PACX-1vTYX-jozloAfaj-tZ11ZOAIglmyjME58WRa2H52XHjBm0ZpHx78JEsL08091BMRxrSgTHm_BDleAzk0/pub?gid=1713471992&single=true&output=csv`

Sponsor-CSV:

`https://docs.google.com/spreadsheets/d/e/2PACX-1vTYX-jozloAfaj-tZ11ZOAIglmyjME58WRa2H52XHjBm0ZpHx78JEsL08091BMRxrSgTHm_BDleAzk0/pub?gid=2113186516&single=true&output=csv`

Verwendete Fernsteuerungsfelder:

- `SponsorRondellAktiv`
- `PlatzmanagerDauerSek`
- `SponsorenProBlock`
- `AktualisierungSek`
- `RondellURL`
- `PlatzmanagerURL`
- `NurPlatzmanager`

Am 07.09.2026 wurde vom laufenden Raspberry u. a. folgende Konfiguration gelesen:

```text
SponsorRondellAktiv: TRUE
PlatzmanagerDauerSek: 60
SponsorenProBlock: 3
AktualisierungSek: 300
NurPlatzmanager: FALSE
```

Hinweis: `AktualisierungSek` stand beim Produktivtest auf 300 Sekunden. Wenn 60 Sekunden gewünscht sind, muss der Wert in der Fernsteuerung entsprechend geändert werden.

## Refresh-Extension

Ordner:

`/home/kiosk/kabinengang-refresh`

Die ursprüngliche Refresh-Extension wurde nach früheren Tests wiederhergestellt. Die Platzmanager/Sponsor-Umschaltung wird **nicht** über diese Extension umgesetzt.

Vorhandene Backups:

- `background.js.backup`
- `background.js.v2-backup`
- `manifest.json.backup`

Wichtig: Nicht erneut versuchen, die Sponsor-Umschaltung über den Manifest-V3-Service-Worker der Extension umzusetzen. Der funktionierende Steuerweg ist Python + DevTools 9222.

## Fehler-/Rollback-Verhalten

Bei Fehlern in der Hauptschleife versucht der Python-Switch den Platzmanager wieder nach vorne zu bringen und wartet anschließend 10 Sekunden vor dem nächsten Versuch.

Lokales Rollback des Switches ist über das Backup möglich:

```bash
cp ~/kabinengang-switch.py.backup-2026-09-07 ~/kabinengang-switch.py
systemctl --user restart kabinengang.service
```

## Produktivtest 07.09.2026

Bestätigter Ablauf:

```text
>>> PLATZMANAGER
Sponsor vorbereitet: ...?start=0&count=3
Platzmanager für 60.0 Sekunden
>>> SPONSOREN
Güven Pflegedienst GmbH => 30.0 Sekunden
Selgros Cash & Carry => 30.0 Sekunden
ATZ GmbH => 30.0 Sekunden
Reine Sponsorzeit: 90.0 Sekunden
Gesamter Sponsorblock: 91.2 Sekunden
>>> ZURÜCK ZUM PLATZMANAGER
Sponsor vorbereitet: ...?start=3&count=3
```

Nach der Erweiterung der ISO-Datumsverarbeitung wurde der Service erneut gestartet; der Platzmanager erschien erfolgreich.

## Noch beobachten

Für den endgültigen Dauerbetrieb sollten noch mehrere aufeinanderfolgende Blöcke beobachtet werden, insbesondere:

- kein erneuter schwarzer Bildschirm,
- korrekter Wechsel über das Ende der Sponsorenliste zurück zum Anfang,
- Verhalten bei temporärem Netzwerkausfall,
- Verhalten bei `NurPlatzmanager = TRUE`,
- Verhalten bei `SponsorRondellAktiv = FALSE`,
- Änderung der Konfiguration während des laufenden Betriebs.

Stand: **07.09.2026 – neue Python/DevTools-Steuerung erfolgreich in Betrieb und Kernfunktionen getestet.**
