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
2. OLED 0,96" SSD1315 mit 4-Tasten-Board. Tobias' Exemplar ist die
   **Zweifarb-Variante** (Zeilen 0–15 gelb, 16–63 blau) – das UI-Layout
   nutzt das seit 2026-07-09: Titelbalken füllt exakt die gelbe Zone,
   Inhalt beginnt erst in der blauen.
3. KY-040 Rotary Encoder.
4. Steckbrett + Female-Female-Jumper (das OLED- und Encoder-Modul haben
   Stiftleisten – je nach Aufbau gehen auch Female-Jumper direkt, ganz
   ohne Steckbrett).

Versorgung in dieser Phase: **nur USB-C**. Batteriefach, Schiebeschalter,
1N5817 und der 100k/22k-Teiler kommen erst beim Gehäuse-Einbau dazu
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
                                    22 kΩ (10k + 12k
                                        in Serie)
                                           │
4×AA (−) ─────────────────────────────────●──► GND
```

Ursprünglich waren 47 kΩ unten geplant; gebaut wurde mit 10k + 12k in
Serie (= 22 kΩ, war im Sortiment vorhanden). Teilerfaktor 122/22 ≈ 5,55,
`VBAT_DIVIDER` in `Config.h` ist entsprechend gesetzt – 6,4 V ergeben
~1,15 V am Pin, weiterhin sicher unter den ~2,5 V des ADC-Bereichs.

Als Diode ist statt der geplanten 1N5817 eine **SS34** verbaut
(Schottky 40 V/3 A, SMA-SMD – elektrisch gleichwertig, ~0,3 V Drop).
SMD-Bauform: Anschlussdrähte anlöten; **Kathodenstrich Richtung
5V-Pin**. Alle 1N5817-Erwähnungen in diesem Doc gelten sinngemäß.

Der Teiler hängt **vor** der Diode (misst echte Batteriespannung) und
**hinter** dem Schalter (zieht bei „Aus" keinen Strom; Dauerlast bei „An"
ist 6 V/122 kΩ ≈ 49 µA, egal). Ohne Batterien liest GPIO3 über die 22k
sauber 0 V → Anzeige „USB".

> ⚠️ **USB + Alkalines nicht gleichzeitig:** Der Super Mini verbindet
> USB-VBUS direkt mit dem 5V-Pin (keine Board-Diode). Frische 4×AA
> drücken ~6 V hinter der 1N5817 zurück in den USB-Port des Hosts.
> Regel: **erst Schiebeschalter aus, dann USB rein.** Mit NiMH-Akkus
> (4×1,2 V → ~4,5 V nach Diode) ist Gleichzeitbetrieb dagegen
> unkritisch, weil die Diode dann von selbst sperrt.

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
| VBAT-Anzeige zu niedrig und springend (z. B. 4,2–5,0 V statt 6,4 V) | Teiler zu hochohmig für den ADC-Sample-Kondensator → **100 nF von GPIO3 nach GND** direkt am Teiler-Mittelpunkt. Zusätzlich mittelt die FW seit 2026-07-10 über 8 Samples. Beobachtet beim Bring-up 2026-07-10 |
| ESP-NOW-Init-Fehler im Log | Sollte nicht passieren; falls doch: Arduino-Core-Version prüfen (espressif32@^6.x, Core 2.x – Core 3.x hat andere recv-Callback-Signatur) |

## 6. Offene Verifikationspunkte (aus Doc 18 § 4.3)

- [ ] Encoder-Interrupt-Decoding stabil bei schnellem Drehen
- [x] SSD1315 läuft mit SSD1306-Konstruktor (2026-07-09; Panel ist die
      Zweifarb-Variante gelb/blau, UI-Layout angepasst)
- [ ] K-Tasten active-low bestätigt (Pullups intern reichen)
- [x] 6-V-Kette: 5,996 V vor / 5,6 V hinter der SS34 (~0,4 V Drop) bei
      Zellen mit 6,38 V Leerlauf → unter 6,3 V ✓ (2026-07-10)
- [ ] Super Mini: VBUS↔5V-Pin wirklich diodenlos? (Multimeter-Diodentest
      USB-Stecker-VBUS gegen 5V-Pin, beide Richtungen – bestimmt die
      Schärfe der USB+Batterie-Warnung in § 2)
- [x] VBAT-Anzeige plausibel: OLED 5,91–5,97 V vs. ~5,99 V real (~1 %
      Abweichung, keine Kalibrierung nötig; mit 8-fach-Mittelung, noch
      ohne 100-nF-Kondensator) (2026-07-10)

Stolperfalle beim Aufbau 2026-07-10 (gelöst): Teiler-Mittelabgriff
steckte auf **GPIO9 (BOOT, Onboard-Pullup)** statt GPIO3 → Messung
zeigte die 3,3-V-Schiene, Display Zufallswerte (GPIO3 floatete). Bei
Fehlersuche also zuerst prüfen, ob der Jumper wirklich auf dem Pin mit
Silk „3" sitzt.

Ergebnisse bitte hier eintragen und Häkchen setzen – das Dokument ist
Teil der Wissensbasis und wandert mit ins `infinitag-now`-Repo (docs/).
