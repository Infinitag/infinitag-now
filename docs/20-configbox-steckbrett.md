# 20 – Config-Box: Steckbrett-Aufbau & Bring-up

> **Status:** Anleitung für den ersten Hardware-Aufbau (GPIO-Plan v3,
> 2026-07-09). Konzept und Begründungen: [`18-config-tool.md`](18-config-tool.md).
> Firmware: Repo [`infinitag-now-config`](https://github.com/Infinitag/infinitag-now-config)
> (lokale Arbeitskopie: `/Volumes/Basteln/Infinitag/repos/infinitag-now-config/`).

## 1. Teile für die Steckbrett-Phase

Nur diese vier Dinge, Rest kommt beim Gehäuse-Einbau:

1. ESP32-C3 Super Mini – **Antenne per IPEX-Pigtail anschrauben**, bevor
   gefunkt wird (Senden ohne Antenne ist schlecht für die PA und die
   Reichweite der Onboard-Keramik ist mies).
2. OLED 0,96" SSD1315 mit 4-Tasten-Board.
3. KY-040 Rotary Encoder.
4. Steckbrett + Female-Female-Jumper (das OLED- und Encoder-Modul haben
   Stiftleisten – je nach Aufbau gehen auch Female-Jumper direkt, ganz
   ohne Steckbrett).

Versorgung in dieser Phase: **nur USB-C**. Batteriefach, Schiebeschalter,
1N5817 und der 100k/47k-Teiler kommen erst beim Gehäuse-Einbau dazu
(siehe § 4).

## 2. Verdrahtung (GPIO-Plan v3)

**Vor dem Verdrahten: USB abziehen.** Erst stecken, dann Strom.

### OLED-Modul (I²C + 4 Tasten)

| OLED-Pin | ESP32-C3 | Ader-Farbe (Vorschlag) |
|---|---|---|
| GND | GND | schwarz |
| VCC | **3V3** | orange |
| SCL | GPIO7 | blau |
| SDA | GPIO6 | blau/weiß |
| K1 (Zurück) | GPIO4 | grün |
| K2 (∧ / ×10) | GPIO5 | grün |
| K3 (Blink an/aus) | GPIO10 | grün |
| K4 (OK) | GPIO20 | grün |

### KY-040 Encoder

| KY-040-Pin | ESP32-C3 | Anmerkung |
|---|---|---|
| GND | GND | |
| + | **3V3** | **nicht 5 V!** Die Onboard-Pullups würden sonst 5 V auf GPIO0/1 legen |
| CLK (A) | GPIO0 | |
| DT (B) | GPIO1 | |
| SW (Push) | GPIO21 | seit Plan v3 (vorher GPIO3) |

### Später im Gehäuse: Versorgung + Batteriemessung

```
4×AA (+) ── Schiebeschalter ──●── 1N5817 ──► 5V-Pin
                              │
                              └── 100 kΩ ──●──► GPIO3 (VBAT-Messung)
                                           │
                                          47 kΩ
                                           │
4×AA (−) ─────────────────────────────────●──► GND
```

Der Teiler hängt **vor** der Diode (misst echte Batteriespannung) und
**hinter** dem Schalter (zieht bei „Aus" keinen Strom; Dauerlast bei „An"
ist 6 V/147 kΩ ≈ 40 µA, egal). Ohne Batterien liest GPIO3 über den 47k
sauber 0 V → Anzeige „USB".

## 3. Flashen

```bash
cd /Volumes/Basteln/Infinitag/repos/infinitag-now-config
pio run -e configbox -t upload
pio device monitor
```

- Die Protokoll-Lib zieht PlatformIO automatisch vom privaten
  `infinitag-now-core`-Repo – das lokale Git braucht dafür GitHub-Zugriff
  (hat der Mac, sonst wäre der Push nicht durchgegangen).
- **Falls der Upload-Port nicht auftaucht** (jungfräuliches Board o.
  Absturz-Firmware): BOOT-Taste gedrückt halten, USB einstecken,
  loslassen → Download-Modus, Upload wiederholen, danach RESET.
- Erwartete Log-Zeile: `[OK] Config-Box up, MAC xx:xx:xx:xx:xx:xx`.

## 4. Bring-up in Stufen

Nach jeder Stufe erst weiter, wenn sie sauber läuft:

| Stufe | Aufbau | Erwartung |
|---|---|---|
| 1 | Nur C3 an USB, nichts verdrahtet | Log-Zeile mit MAC; kein Absturz/Bootloop |
| 2 | + OLED (4 Adern GND/VCC/SCL/SDA) | Hauptmenü „Infinitag Config" erscheint |
| 3 | + Encoder | Drehen bewegt Cursor, Push wählt aus. Läuft der Cursor „falsch herum": CLK/DT-Kabel tauschen |
| 4 | + K1–K4 | K1 = zurück, K4 = OK, K2 halten + drehen = ×10, K3 in Listen = Blink-Toggle |
| 5 | Tools → Firmware-Info | Version, MAC, Heap, Uptime. Batterie-Zeile zeigt ohne Teiler Zufallswerte (Pin floatet) – erst mit Teiler aussagekräftig |
| 6 | Funk | Listen bleiben leer („suche Geraete…"), K2 stößt neue Suche an – **normal**, es existiert noch keine Gegenstelle. Nächster Schritt: Stub-Firmware auf einem zweiten ESP |

## 5. Troubleshooting

| Symptom | Wahrscheinliche Ursache → Abhilfe |
|---|---|
| OLED bleibt schwarz | SDA/SCL vertauscht; oder I²C-Adresse 0x3D statt 0x3C (`u8g2.setI2CAddress(0x3D << 1)`); oder Modul ist doch SH1106 → Konstruktor `U8G2_SH1106_128X64_NONAME_F_HW_I2C` testen |
| OLED zeigt versetzten/verschobenen Inhalt | SH1106-Panel mit SSD1306-Konstruktor (2-Pixel-Offset) → SH1106-Konstruktor |
| Encoder zählt doppelt/gar nicht | Detent-Raster ≠ Full-Step → in `InputController.cpp` den Teiler `/4` auf `/2` ändern |
| Encoder springt/prellt | RC-Glied nachrüsten: 10 kΩ in Serie, 100 nF gegen GND an CLK und DT (Doc 18 § 4.3) |
| Tasten reagieren nicht | Beschaltung des Moduls prüfen: Taster müssen gegen GND schalten (active-low). Multimeter: K-Pin gegen GND beim Drücken ≈ 0 Ω |
| Taste „prellt" ins Menü beim Boot | Taster beim Einschalten gedrückt? GPIO9-Nachbarpin erwischt? Verkabelung prüfen |
| Bootloop nach Verdrahtung | Strapping-Pin belastet: GPIO2 muss frei bleiben, GPIO8/9 nicht extern beschalten |
| Kein Upload-Port | BOOT-Taste-Trick (§ 3); anderes USB-Kabel (Ladekabel ohne Datenadern sind häufig) |
| ESP-NOW-Init-Fehler im Log | Sollte nicht passieren; falls doch: Arduino-Core-Version prüfen (espressif32@^6.x, Core 2.x – Core 3.x hat andere recv-Callback-Signatur) |

## 6. Offene Verifikationspunkte (aus Doc 18 § 4.3)

- [ ] Encoder-Interrupt-Decoding stabil bei schnellem Drehen
- [ ] SSD1315 läuft mit SSD1306-Konstruktor
- [ ] K-Tasten active-low bestätigt (Pullups intern reichen)
- [ ] 6-V-Kette: Spannung hinter 1N5817 bei frischen Zellen < 6,3 V
- [ ] VBAT-Anzeige plausibel (Multimeter vs. OLED-Wert)

Ergebnisse bitte hier eintragen und Häkchen setzen – das Dokument ist
Teil der Wissensbasis und wandert mit ins `infinitag-now`-Repo (docs/).
