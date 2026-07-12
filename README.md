# Infinitag Now

**Ein Zauberstab-Spiel zum Selberbauen:** Kinder wirken mit einem
Zauberstab Zaubersprüche auf Ziele – trifft der Zauber, spukt es: Sound-
Effekte, Lichter, bewegliche Props. Alles funkt direkt von Gerät zu
Gerät über ESP-NOW, ganz ohne WLAN-Router, Server oder App.
Einschalten → läuft. Entstanden für einen Halloween-Zauberstand im
Vorgarten, einsetzbar überall, wo gespielt wird.

![Funk](https://img.shields.io/badge/Funk-ESP--NOW-purple)
![Plattform](https://img.shields.io/badge/Plattform-ESP32-blue)
![Lizenz](https://img.shields.io/badge/Lizenz-CC%20BY--NC--SA%204.0-lightgrey)

<!-- TODO: Hero-Foto des Aufbaus einfügen:
     ![Infinitag Now](docs/assets/hero.jpg) -->

## So funktioniert es

1. Der **Zauberstab** (an der Station) löst per Trigger einen
   IR-Zauber aus – mit Laser-Zielhilfe und Statusfarben im Stab.
2. Das **Target** am Ziel empfängt den IR-Impuls und meldet den
   Treffer per Funk.
3. Die **Station** spielt den passenden Sound-Effekt, das Target
   schaltet Licht oder Props (z. B. eine sich öffnende Geistertür).
4. Die **Config-Box** ist die Fernbedienung für alles: Geräte finden,
   einstellen, testen und über die Luft aktualisieren.

Geräte identifizieren sich allein über ihre MAC-Adresse: kein Pairing,
keine ID-Vergabe, kein Setup-Ritual. Firmware-Updates laufen kabellos
über einen SoftAP-Update-Modus; jede Version liegt als fertige `.bin`
in den GitHub-Releases der Geräte-Repos.

## Repos der Familie

| Repo | Inhalt |
|---|---|
| [`infinitag-now`](https://github.com/Infinitag/infinitag-now) | Dieses Repo: Doku (Wissensbasis), Hardware, Übersicht |
| [`infinitag-now-core`](https://github.com/Infinitag/infinitag-now-core) | Gemeinsame Bibliothek: Funkprotokoll (`PROTOCOL.md`), ESP-NOW-Service, SoftAP-Updater |
| [`infinitag-now-station`](https://github.com/Infinitag/infinitag-now-station) | Station: Sound + Zauberstab (ESP32-S3, I²S-Audio, 12 V) |
| [`infinitag-now-config`](https://github.com/Infinitag/infinitag-now-config) | Config-Box: Handheld-Konfigurator (ESP32-C3, OLED, Encoder) |
| `infinitag-now-target` | Target-Firmware *(in Arbeit)* |

## Doku

Die komplette Wissensbasis liegt in [`docs/`](docs/) – Einstieg über
[`docs/README.md`](docs/README.md). Wichtigste Dokumente:

- `docs/12-refactor-station-v2.md` – Station V2 inkl. ESP-NOW-Protokoll (§ 3)
- `docs/13-refactor-wand-v3.md` – Zauberstab V3 (Optik, Mechanik)
- `docs/18-config-tool.md` – Config-Box (Konzept, Bedienung, Updates)
- `docs/09-halloween-setup.md` – der ursprüngliche Einsatz als Beispiel

Hinweis: Die Wissensbasis dokumentiert historisch auch das frühere
HTTP/WiFiManager-Setup (Doc 01–08); verbindlich für Neues sind die
Refactor-Dokumente 12/13/18 und `PROTOCOL.md` im Core-Repo.

## Abgrenzung zum klassischen Infinitag

Das klassische [Infinitag](https://github.com/Infinitag) (Arena-Lasertag
mit Tagger, Sensoren und Spieler-Management, 2017) ist ein eigenes
System. Infinitag Now ist kein Nachfolger und kein Fork, sondern ein
Geschwister mit anderem Einsatzzweck: wenige Geräte, null Infrastruktur,
Peer-to-Peer-Funk – und ein Zauberstab statt eines Taggers.

## Lizenz

[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) – Tobias Stewen.

Infinitag Now ist eine **komplette Neuentwicklung** – eigenes Protokoll,
eigene Hardware, eigene Firmware. Mit dem klassischen Infinitag teilt es
nur den Namenskosmos und die Grundidee.
