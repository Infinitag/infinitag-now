# 04 – Hardware: Infinitag Target V3.2

PCB‑Bezeichnung: **Infinitag Target V3.2** (Stand 2023‑08‑30).
Quelle: Schaltplan & Layout (Easyeda).

Anders als die Wand ist das Target ein **eigenständiger ESP32‑S3‑Computer**:
es bringt einen ESP32‑S3‑WROOM‑1 (`N8R8`), USB‑UART‑Bridge und alle Peripherie
auf eine Platine. Programmierung über USB‑C, Konfiguration per WiFiManager.

## Funktionsblöcke

### ESP Logic

- `U1 = ESP32‑S3‑WROOM‑1 (N8R8)` – 8 MB Flash, 8 MB PSRAM.
- `R1 = 0 Ω` Reihenwiderstand am 3V3‑Eingang (Filterung).
- `C1 = 100 nF`, `C2 = 10 µF` als VCC‑Stütz.
- `R2 = 10 kΩ` Pullup auf `CHIP_PU` (EN‑Pin), `C3 = 1 µF` auf GND als
  Reset‑Verzögerung.
- Alle relevanten GPIOs werden direkt zu Funktionsblöcken geroutet.

### USB‑Stage

- `USB2 = 105164‑0001` – USB‑Buchse (USB‑C oder Mini, je nach Footprint).
- `D2/D3/D4 = LESD5D5.0CT1G` – ESD‑Schutzdioden auf D+/D−/VBUS.
- `D1 = 1N5819HW‑7‑F` Schottky → schützt VBUS gegen Rückstrom auf 5 V‑Schiene.
- `C4 = 10 µF` Stützkondensator an VCC_5V.

### USB‑zu‑UART (CP2102N)

- `U3 = CP2102N‑A02‑GQFN28R` – wandelt USB ↔ UART.
- Wired up to `U0RXD`/`U0TXD` des ESP32‑S3 (Standard‑Programmier‑UART).
- `R8/R9/R10/R11 = 1k/47k/22k/10k` typische Beschaltung gemäß Datasheet.
- `C11 = 10 µF`, `C12 = 100 nF` Stütz.
- `LED1 = KT‑0603R` – Power/USB‑Status‑LED über `R7 = 5,1 kΩ`.

### Auto‑Programmierung

- `Q1, Q2 = L8050QLT1G` (NPN), `R3, R4 = 0 Ω`, `R5, R6 = 10 kΩ`.
- Standard‑Schaltung CP2102 → DTR/RTS → Reset/IO0 für automatischen
  Flashvorgang. Die Wahrheitstabelle ist im Schaltplan abgebildet:
  `DTR=1, RTS=1` → Boot, `DTR=0, RTS=0` → Run, etc.

### Reset / Boot

- `RESET = TS‑1187A‑B‑A‑B`, an `CHIP_PU` mit `C10 = 100 nF` Entprellung.
- `BOOT = TS‑1187A‑B‑A‑B`, an `GPIO0` mit `C9 = 100 nF`.
- Beide Taster ziehen GND.

### 5 V → 3,3 V

- `U2 = AMS1117‑3.3` LDO.
- `C5 = 10 µF`, `C6 = 100 nF` am Eingang; `C7 = 100 nF`, `C8 = 10 µF` am Ausgang.
- `R7 = 5,1 kΩ` + `LED1` als Power‑LED an 3,3 V.

### Switches (3 Schaltausgänge)

| Block | Bauteile | GPIO | Funktion |
| --- | --- | --- | --- |
| `SW1` (potentialfrei) | Optokoppler `U16 = BPC‑817S/A`, `R21 = 180 Ω`, `R15 = 5,1 kΩ`, Status‑LED `L‑SW1` | GPIO21 | Galvanisch getrennter Schaltausgang. **Steckplatz `SW1 = S2B‑PH‑K‑(LF)(SN)`** – an die beiden Pins kann eine Last bis 80 mA / 35 V DC gehängt werden (BPC‑817S/A Datenblatt). |
| `SW‑5V` | NPN `Q4 = SS8050`, `R17 = 1 kΩ`, `R19 = 5,1 kΩ`, Status‑LED `L‑5V` | GPIO46 | Schaltet 5 V auf den Stecker `SW‑5V = S2B‑PH‑K‑(LF)(SN)`. Geeignet für 5 V‑Halloween‑Props (LEDs, kleine Pumpen). |
| `SW‑3.3V` | NPN `Q5 = SS8050`, `R18 = 1 kΩ`, `R20 = 5,1 kΩ`, Status‑LED `L‑3.3V` | GPIO48 | Schaltet 3,3 V auf den Stecker `SW‑3.3V = S2B‑PH‑K‑(LF)(SN)`. |

Alle drei Steckplätze sind 2‑pol JST‑PH. Pinbelegung jeweils Pin 1 = +,
Pin 2 = − (laut Schaltplan‑Konvention).

### IR‑Receiver

- `LED2 = TSOP4138` – 38 kHz IR‑Demodulator, OUT‑Pin → `IR_RCV` Net.
- VS an 3,3 V, GND an Masse.
- `IR_RCV` geht zum ESP32 → im Code als `IR_PIN = 15` definiert.

> ⚠️ **Verifikationspunkt:** Im Schaltplan ist der genaue GPIO‑Pin nicht
> eindeutig auf dem Bild lesbar (nur Net‑Name `IR_RCV`). Der Code geht von
> GPIO15 aus. Bitte beim nächsten Flash kurz prüfen, indem man am IR‑Empfänger
> einen RC5‑Code sendet und im seriellen Monitor das `IrReceiver.decode()`
> beobachtet.

### LED Ring

- 12 × `SK6812RGBW‑5050` (`U4‑U15`), in Reihe verkettet (DOUT → DIN).
- Jede LED hat einen eigenen 100 nF Abblockkondensator (`C13–C24`) – sehr
  saubere Auslegung.
- Versorgung 5 V, Daten kommen vom ESP32 (`LED_ESP` Net) – im Code
  `LED_PIN = 38`, `LED_COUNT = 12`.

### External LEDs / External Power Supply

- `LED‑Connector = S3B‑PH‑K(LF)(SN)` – zusätzlicher 3‑pol Stecker für externe
  WS281x‑Stränge: 5 V, GPIO47 als Datenleitung, GND.
- `PWR = S2B‑PH‑K(LF)(SN)` – externer 5‑V‑Eingang, parallel zur USB‑VBUS‑Schiene.

## GPIO‑Belegung im Überblick

| GPIO | Funktion | Quelle |
| --- | --- | --- |
| 0 | BOOT‑Taster | Schaltplan |
| 15 | IR_RCV (TSOP4138) | Code (`IR_PIN`), Schaltplan‑Net `IR_RCV` |
| 21 | SW1 (Optokoppler) | Code (`SW_PIN`), Schaltplan |
| 38 | LED‑Ring (SK6812) | Code (`LED_PIN`), Schaltplan |
| 46 | SW‑5V (NPN) | Code (`SW_5V_PIN`), Schaltplan |
| 47 | LED‑Ring extern (LED‑Connector) | Schaltplan |
| 48 | SW‑3.3V (NPN) | Code (`SW_33V_PIN`), Schaltplan |
| EN | RESET‑Taster + CHIP_PU | Schaltplan |
| U0RXD/U0TXD | UART zu CP2102N | Schaltplan |

## Layout‑Hinweise

- ESP32‑S3 mit Antenne nach oben aus dem Board heraus orientiert (typische
  Platzierung für freie Funkkeule).
- AMS1117 mittig rechts, viele Vias unter dem Pad.
- Massive GND‑Plane vorhanden.
- Status‑LEDs (`L‑5V`, `L‑3.3V`, `L‑SW1`, `LED1`) am unteren Rand für gute
  Sichtbarkeit beim Debuggen.
- USB‑Buchse mittig unten.
