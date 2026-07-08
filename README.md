# Infinitag Now

**Infrastrukturloses ESP-NOW-Setup der Infinitag-Familie** – gedacht für
kleine Aufbauten wie die Halloween-Schießbude im Vorgarten: Zauberstab-
Stationen, IR-Targets und Halloween-Props, die ohne Router, Server oder
App auskommen. Einschalten → läuft.

**Abgrenzung:** Das klassische Infinitag (Arena-Lasertag mit Tagger,
Sensoren und Spieler-Management, siehe die übrigen Repos dieser Org) ist
ein eigenes System. Infinitag Now ist kein Nachfolger und kein Fork,
sondern ein Geschwister mit anderem Einsatzzweck: wenige Geräte, null
Infrastruktur, Peer-to-Peer-Funk via ESP-NOW.

## Repos der Familie

| Repo | Inhalt |
|---|---|
| [`infinitag-now`](https://github.com/Infinitag/infinitag-now) | Dieses Repo: Doku (Wissensbasis), BOM, Hardware, Übersicht |
| [`infinitag-now-core`](https://github.com/Infinitag/infinitag-now-core) | Gemeinsame ESP-NOW-Protokoll-Lib `InfinitagNow` + `PROTOCOL.md` |
| [`infinitag-now-config`](https://github.com/Infinitag/infinitag-now-config) | Config-Box: Handheld-Konfigurator (ESP32-C3 + OLED + Encoder) |
| `infinitag-now-target` | Target-Firmware (geplant, beim ESP-NOW-Umbau) |
| `infinitag-now-station` | Station-V2-Firmware (geplant) |

## Doku

Die komplette Wissensbasis liegt in [`docs/`](docs/) – Einstieg über
[`docs/README.md`](docs/README.md). Wichtigste Dokumente:

- `docs/12-refactor-station-v2.md` – Station V2 inkl. ESP-NOW-Protokoll (§ 3)
- `docs/13-refactor-wand-v3.md` – Wand V3
- `docs/18-config-tool.md` – Config-Box (Konzept, Bedienung, GPIO-Plan)
- `docs/09-halloween-setup.md` – der eigentliche Einsatzzweck

Hinweis: Die Wissensbasis dokumentiert historisch auch das heutige
HTTP/WiFiManager-Setup (Doc 01–08); verbindlich für Neues sind die
Refactor-Dokumente 12/13/18.

## Lizenz

CC BY-NC-SA 4.0 – Tobias Stewen.

Infinitag Now ist eine **komplette Neuentwicklung** von Tobias Stewen –
eigenes Protokoll, eigene Hardware, eigene Firmware. Mit dem klassischen
Infinitag (2017) teilt es nur den Namenskosmos und die Grundidee.
