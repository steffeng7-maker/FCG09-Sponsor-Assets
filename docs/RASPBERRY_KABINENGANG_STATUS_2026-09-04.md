# Raspberry Kabinengang – Arbeitsstand 04.09.2026

## Ziel
Der große Raspberry-Pi-Screen im Kabinengang soll automatisch zwischen dem FCG 09 Platzmanager und dem Sponsor-Rondell wechseln. Die Steuerparameter sollen von zuhause über die in Glide eingebundene Google-Sheets-Tabelle `Fernsteuerung` gepflegt werden können.

Zielablauf für den Test:

1. Platzmanager für 60 Sekunden anzeigen.
2. Sponsor-Rondell mit 3 Sponsoren anzeigen.
3. Zurück zum Platzmanager für 60 Sekunden.
4. Danach die nächsten 3 aktiven Sponsoren anzeigen (nicht wieder bei Sponsor 1 beginnen).
5. Nach dem letzten aktiven Sponsor zyklisch wieder vorne beginnen.

## Raspberry / Kiosk

- Hostname: `kabinengang`
- SSH-User: `kiosk`
- IP am 04.09.2026: `192.168.178.95` (kann sich künftig ändern)
- SSH: `ssh kiosk@192.168.178.95`
- Chromium: `/usr/bin/chromium`
- Python: `Python 3.13.5`
- Raspberry/Debian: Linux `kabinengang` 6.18.39+rpt-rpi-v8, aarch64

## Bestehender Chromium-Autostart

Datei:

`/home/kiosk/.config/autostart/kabinengang.desktop`

Die funktionierende Kiosk-Konfiguration wurde gesichert als:

`/home/kiosk/.config/autostart/kabinengang.desktop.backup`

Chromium startet weiterhin im Kiosk-Modus mit dem Platzmanager und der bestehenden Refresh-Extension. Zusätzlich wurde für die neue externe Steuerung der lokale Chrome-DevTools-Port ergänzt:

`--remote-debugging-port=9222 --remote-debugging-address=127.0.0.1`

Der DevTools-Zugriff wurde nach Reboot erfolgreich geprüft mit:

`curl -s http://127.0.0.1:9222/json/version`

Ergebnis: Chromium antwortet korrekt und liefert u. a. eine `webSocketDebuggerUrl`. Damit ist der lokale Steuerkanal über Port 9222 bestätigt.

## Platzmanager

Großer Kabinengang-Screen:

`https://fcg09-platzmanager.glide.page/dl/9cec69`

## Sponsor-Rondell

GitHub Pages:

`https://steffeng7-maker.github.io/FCG09-Sponsor-Assets/`

Das Rondell liest die Sponsor-Daten aus dem veröffentlichten Google-Sheets-CSV und berücksichtigt nur aktive bzw. aktuell gültige Sponsoren.

Die Rondell-Testversion auf Branch `sponsor-rondell-v1e` unterstützt inzwischen URL-Parameter:

- `start` = Index des ersten Sponsors
- `count` = Anzahl Sponsoren für diesen Block

Beispiel:

`https://steffeng7-maker.github.io/FCG09-Sponsor-Assets/?start=0&count=3`

Damit kann ein externer Controller Sponsorblöcke 1–3, danach 4–6, danach 7–9 usw. gezielt anfordern.

## Google-Sheets-Fernsteuerung

Veröffentlichtes CSV:

`https://docs.google.com/spreadsheets/d/e/2PACX-1vTYX-jozloAfaj-tZ11ZOAIglmyjME58WRa2H52XHjBm0ZpHx78JEsL08091BMRxrSgTHm_BDleAzk0/pub?gid=1713471992&single=true&output=csv`

Aktuell bestätigte Konfiguration am 04.09.2026:

- `ConfigID`: CFG001
- `SponsorRondellAktiv`: TRUE
- `PlatzmanagerDauerSek`: 60
- `SponsorenProBlock`: 3
- `AktualisierungSek`: 60
- `RondellURL`: GitHub-Pages Sponsor-Rondell
- `PlatzmanagerURL`: Kabinengang Glide URL
- `NurPlatzmanager`: FALSE

Der Raspberry konnte dieses CSV erfolgreich per `curl` abrufen. Google Sheets / Glide / Internetzugriff sind damit nicht Ursache des bisherigen Switch-Problems.

## Bestehende Refresh-Extension

Ordner:

`/home/kiosk/kabinengang-refresh`

Die ursprüngliche funktionierende Extension lädt die Platzmanager-Seite alle 15 Minuten neu.

Backups vorhanden:

- `background.js.backup` – ursprüngliche Refresh-Version
- `background.js.v2-backup` – zwischenzeitliche Fernsteuerungs-Version
- `manifest.json.backup` – ursprüngliches Manifest

Nach den Tests wurden `background.js` und `manifest.json` wieder auf die ursprüngliche Refresh-Version zurückgestellt.

### Erkenntnis aus den Extension-Tests

Der Chromium-Prozess wurde korrekt mit

`--load-extension=/home/kiosk/kabinengang-refresh`

gestartet. Trotzdem wurde weder die Alarm-basierte Switch-Logik noch ein Minimaltest ausgeführt, der beim Start unmittelbar einen Sponsor-Tab öffnen sollte. Deshalb wird der Platzmanager/Sponsor-Wechsel **nicht weiter über die Chromium-Extension umgesetzt**. Die bestehende Refresh-Extension bleibt ausschließlich für ihren bisherigen Reload-Zweck erhalten.

## Neue Architektur (aktueller Stand)

Die geplante robuste Architektur lautet:

`Google Sheets Fernsteuerung -> Python Controller auf Raspberry -> lokaler Chromium DevTools Port 9222 -> Platzmanager/Sponsor-Tabs`

Vorteile:

- unabhängig von Manifest-V3-Service-Worker-Verhalten
- bestehende Kiosk- und Refresh-Lösung bleibt erhalten
- Fernsteuerung weiterhin von zuhause über Glide / Google Sheets
- Sponsor-Fortschritt kann persistent auf dem Pi gespeichert werden
- kontrollierter Wechsel zwischen Platzmanager und Sponsor-Rondell

## Nächster Schritt

Noch **nicht ausgeführt**:

Prüfen, ob das Python-Modul `websocket` bereits installiert ist:

```bash
python3 -c "import websocket; print('WebSocket OK')"
```

Wenn vorhanden, wird anschließend der Python-Controller gebaut. Falls nicht vorhanden, zuerst geeignete Installation festlegen.

Der Controller soll anschließend:

1. Konfiguration regelmäßig aus Google Sheets laden.
2. `NurPlatzmanager` und `SponsorRondellAktiv` berücksichtigen.
3. Platzmanager für `PlatzmanagerDauerSek` anzeigen.
4. Sponsor-Rondell mit `start=<gespeicherter Index>&count=<SponsorenProBlock>` öffnen/aktivieren.
5. Nach jedem Sponsorblock den Startindex persistent erhöhen.
6. Nach dem letzten aktiven Sponsor sauber zyklisch weiterlaufen.
7. Bei Netzwerk-/CSV-Fehlern sicher auf den Platzmanager bzw. zuletzt gültige Konfiguration zurückfallen.
8. Nach Raspberry-Neustart automatisch wieder starten.

## Sicherheits-/Rollback-Stand

Vor weiteren Änderungen sind die wesentlichen funktionierenden Dateien lokal gesichert. Falls die neue Steuerung Probleme verursacht, kann der ursprüngliche Kiosk-/Refresh-Zustand wiederhergestellt werden.

Wichtig für die Fortsetzung: **Nicht erneut versuchen, den Sponsor-Switch über `background.js` der Chromium-Extension zu lösen. Weiter ab dem funktionierenden DevTools-Port 9222 und dem Python-Controller.**
