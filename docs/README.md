# Infinitag – Wissensbasis

Zentrale Sammlung aller Informationen zum Infinitag‑Lasertag‑System, das ursprünglich
zusammen mit Jani Taxidis und Florian Kleene als offenes System entwickelt wurde
(siehe `Infinitag_Core` Lib, Copyright 2017) und heute als Halloween‑Schießbude für Kinder
weiter eingesetzt wird.

Diese Wissensbasis liegt unter `/Volumes/basteln/Infinitag/wissensbasis/`
auf dem DS920‑NAS. Der Projekt‑Stamm ist `/Volumes/basteln/Infinitag/`,
dort liegt auch die [`CLAUDE.md`](../CLAUDE.md) mit den Arbeitsanweisungen
für Claude.

Die Doku liegt absichtlich verteilt auf mehrere Markdown‑Dateien, damit man sich
gezielt einlesen kann ohne ein 1000‑Zeilen‑Dokument zu durchforsten. Diese
README ist Index und Schnelleinstieg.

> **Status (Mai 2026):** Lebende Doku. Diskrepanzen zwischen Code und Schaltplänen
> sind in [`11-offene-punkte.md`](11-offene-punkte.md) gesammelt – bitte zuerst
> dort hinein schauen, bevor man Änderungen plant.

## Schnellüberblick

Drei Hardware‑Bausteine, alle ESP32‑basiert:

1. **Wand Station** – Mittelpunkt jedes Spiels / Halloween‑Aufbaus. Ein PCB mit
   zwei separaten ESPs (Sound‑ESP + Logic‑ESP), MAX98357 I2S‑DAC und
   Wand‑Anschluss. Verteilt Sounds, vermittelt Treffer‑Meldungen, hostet das
   WiFi.
2. **Wand (Tagger)** – Reine „Frontend“-Hardware ohne eigenen Mikrocontroller:
   IR‑LED + Linse, Laserpointer, Trigger‑Button, Status‑LEDs. Hängt per
   7‑poligem Kabel an der Station.
3. **Target** – Eigenständiger ESP32‑S3 mit IR‑Empfänger, RGBW‑LED‑Ring und drei
   Schaltausgängen (potentialfrei + 5 V + 3,3 V) für Halloween‑Props. Hängt
   per WLAN an der Station.

```
                       ┌────────────────────────────────────┐
                       │         WAND STATION (PCB)         │
   Wand‑Kabel  ──────► │ Logic‑ESP  ◄─── I²C ─►  Sound‑ESP  │
   (S7B 7‑pin)         │  (IR/WiFi)            (MAX98357)   │
                       └────────────┬───────────────────────┘
                                    │ WiFi (HTTP)
                                    │ /trigger_effect?sound=N
                       ┌────────────┴─────────┐
                       │       Target(s)      │
                       │ ESP32‑S3 + IR‑Recv   │
                       │ + LED‑Ring + Switches│
                       └──────────────────────┘
```

## Inhaltsverzeichnis

| Datei | Inhalt |
| --- | --- |
| [`01-system-uebersicht.md`](01-system-uebersicht.md) | Architektur, Datenflüsse, Lebensgeschichte des Projekts |
| [`02-hardware-wand.md`](02-hardware-wand.md) | Wand V2.1 – Schaltplan & Pinbelegung des Tagger‑Stabs |
| [`03-hardware-wand-station.md`](03-hardware-wand-station.md) | Wand Station Dev Board V1 – Schaltplan, beide ESPs, Sound‑Stage |
| [`04-hardware-target.md`](04-hardware-target.md) | InfinitagTarget V3.2 – ESP32‑S3 + Switches + IR‑Receiver |
| [`05-protokoll-ir-i2c.md`](05-protokoll-ir-i2c.md) | 24‑Bit Infinitag‑IR‑Protokoll + I²C‑Befehlssatz aus `Infinitag_Core` |
| [`06-software-wand-logic.md`](06-software-wand-logic.md) | `InfinitagWand.ino` – Logic‑ESP der Station |
| [`07-software-wand-sound.md`](07-software-wand-sound.md) | `InfinitagWandStation.ino` – Sound‑ESP der Station |
| [`08-software-target.md`](08-software-target.md) | `Infinitag Target` (PlatformIO) – Target‑Firmware |
| [`09-halloween-setup.md`](09-halloween-setup.md) | Wie der Halloween‑Aufbau im Vorgarten konkret läuft |
| [`10-bill-of-materials.md`](10-bill-of-materials.md) | Verbaute Bauteile, ICs, Stecker, 3D‑Druck‑Teile, Sound‑Files |
| [`11-offene-punkte.md`](11-offene-punkte.md) | Diskrepanzen Code↔Schaltplan, TODOs, Ideen |
| [`12-refactor-station-v2.md`](12-refactor-station-v2.md) | **Lebendes Konzept‑Dokument** zum laufenden Station‑V2‑Refactor (ESP32‑S3‑DevKitC‑1U N16R8, ESP‑NOW, LittleFS‑Audio, OLED‑Konfig) |
| [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md) | **Lebendes Konzept‑Dokument** zum Wand‑V3‑Refactor (Optik‑Block mit OM3, Halbschalen‑Gehäuse, angelötetes Kabel, neues PCB) |
| [`14-pcb-readiness.md`](14-pcb-readiness.md) | **Pre‑Flight‑Checkliste** vor der Bestellung des Station‑V2‑PCBs (Audio‑Rauschen, GPIO‑Validierung, ESP‑NOW, Kombinationstest) |
| [`15-bestellliste-station-v2.md`](15-bestellliste-station-v2.md) | **Bestellliste** für das Station‑V2‑PCB (Inventar‑Check, LCSC‑Teile, AliExpress‑Module, JLCPCB‑Optionen) |
| [`16-pcb-layout-hinweise.md`](16-pcb-layout-hinweise.md) | **Layout‑Hinweise** für das PCB‑Routing (Bypass‑Platzierung, Stern‑GND, I²S‑Routing, Trace‑Breiten) |
| [`17-bring-up-prototyp-1.md`](17-bring-up-prototyp-1.md) | **Bring‑Up Plan** für den ersten PCB‑Prototyp (Bestueckungsreihenfolge, Tests, verschobene Verifikationspunkte) |
| [`18-config-tool.md`](18-config-tool.md) | **Lebendes Konzept‑Dokument** zur zentralen Config‑Box (ESP32‑C3 Super Mini + OLED + Rotary Encoder, stateless, ESP‑NOW‑Discovery + Identify‑Blink, Web‑UI im SoftAP) |
| [`19-pcb-split-netliste.md`](19-pcb-split-netliste.md) | **Arbeitsdokument** zum Station‑V2 PCB‑Split: Bauteil‑/Netz‑Aufteilung Logic‑Board ↔ Audio‑Board, Inter‑Board‑Pinout, EasyEDA‑Vorgehen |
| [`20-configbox-steckbrett.md`](20-configbox-steckbrett.md) | **Aufbau‑Anleitung** für den Config‑Box‑Steckbrett‑Prototyp: Verdrahtung nach GPIO‑Plan v3, Flash‑Anleitung, Bring‑up in 6 Stufen, Troubleshooting |

## Quellverweise

Alle Quellen liegen aktuell außerhalb dieses Doku‑Ordners. Die Doku verlinkt
sie über deren tatsächliche Pfade auf der Maschine:

- Arduino‑Sketches: `/Volumes/basteln/Arduino/`
  - `InfinitagWand/InfinitagWand.ino` – Logic‑ESP der Station
  - `InfinitagWandStation/InfinitagWandStation.ino` – Sound‑ESP der Station
  - `Infinitag_Core/` – Core‑Library (lokal vorgehalten, identisch zur Lib in `libraries/`)
  - `InfinitagTargetTest/`, `InfinitagTargetResetter/`, `InfinitagTargetAdminTool/` – Hilfssketche
- PlatformIO‑Projekt: `/Volumes/basteln/PlatformIo/Projects/Infinitag Target/`
- Halloween‑Assets: `/Volumes/basteln/Halloween/Lasertag/`
  - `StationSounds/sound-effects/*.mp3` – Sound‑Effekte (durchnummeriert 0‑13)
  - `Spell/*.mp3` – Sound‑Effekte für den Wand
  - `Target/*.3mf` – 3D‑Druck‑Modelle für Targets

## Lizenz

Alle ursprünglichen Infinitag‑Quellen stehen unter
[Creative Commons BY‑NC‑SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).
Diese Doku übernimmt diese Lizenz.
