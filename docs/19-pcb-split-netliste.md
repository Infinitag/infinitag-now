# 19 – Station V2 PCB‑Split: Bauteil‑ und Netz‑Aufteilung

> **Status: lebendes Arbeitsdokument.** Grundlage für die Aufteilung des
> Single‑Board‑Schaltplans (EasyEDA, Rev 1.0 vom 2026‑05‑14) in **zwei
> getrennte Schaltpläne/PCBs** (Logic/Data + Audio). **Stand 2026‑06‑08:
> beide Station‑PCBs + die Wand‑PCB sind geroutet und DRC‑sauber (V1) –
> finaler Layout‑Stand + Pre‑Order‑Checkliste in § 8.** Begründung und
> Mechanik des Stacks: [`12-refactor-station-v2.md`](12-refactor-station-v2.md)
> § 8.14. Connector‑Wechsel: § 8.11.

Begonnen: 2026‑06‑08.

---

## 1. Ziel

Das bisherige eine Station‑PCB wird in **zwei physisch getrennte Boards**
zerlegt, die im Gehäuse Rücken‑an‑Rücken gestapelt und über ein kurzes
JST‑XH‑Kabel verbunden werden:

- **Logic‑PCB** – ESP32‑S3 + alle Signal‑/Stab‑/Konfig‑Anschlüsse
- **Audio‑PCB** – Power‑Entry (12 V), Buck (5 V), DAC, Verstärker, Speaker

Es bleiben **genau 6 Netze** zwischen den Boards (8‑pol Stecker, 5 V + GND
gedoppelt). Alles andere ist board‑lokal.

---

## 2. Bauteil → Board

### 2.1 Logic‑PCB

| Designator | Bauteil | Aufgabe |
|---|---|---|
| BB1 | ESP32‑S3‑DevKitC‑1U N16R8 | Steuerung, ESP‑NOW, I²S‑Quelle |
| U1 | SN74AHCT125D | NeoPixel‑Levelshifter 3,3 → 5 V |
| C3 | 100 nF | U1 Bypass |
| R5 | 470 Ω | Serien‑R Status‑LED‑Daten |
| H1 | Station‑LED (SK6812RGBW) | über 3‑pol JST an Außenwand |
| R1 | 100 Ω | Serien‑R NeoPixel‑Daten |
| R2 | 100 Ω | Serien‑R IR_TX |
| R3 | 100 Ω | Serien‑R Laser |
| R4 | 100 Ω | Serien‑R Trigger |
| C1, C2 | 100 nF | Filter Trigger / Laser |
| **D1, D2** | **USBLC6‑2SC6** | ESD Stab‑Signale – D1: NeoPixel + Trigger, D2: IR + Laser; Pass‑Through inline |
| **CN1** | **6‑pol Schraubklemmstecker KF2EDGRC‑5.08‑6P** | Stab‑Anschluss (ersetzt USBC1) |
| CN2 | OLED Service‑Display 8‑pol *(optional)* | Konfig am Gerät, nur falls bestückt |
| ~~H2, H3~~ | ~~Reserve‑/Testpin‑Header~~ | **entfällt 2026‑06‑08** – Debug/Flash läuft über die DevKitC‑USB‑C |
| **CN3** | **JST‑XH 8‑pol (S8B‑XH‑A‑1)** | Inter‑Board‑Kabel zum Audio‑PCB |

> **ESD auf der Logic‑PCB:** Das alte 4‑Kanal `USBLC6‑4SC6Y` (war „D2") wird durch
> **2× `USBLC6‑2SC6` (D1 + D2)** ersetzt – bleibt **auf dem Data‑Board am
> Stab‑Connector CN1**, Pass‑Through inline (`_F` → Pin 1/6 → `_FO`),
> VBUS = Pin 5 → +5 V. D1 schützt NeoPixel + Trigger, D2 schützt IR + Laser.
> Die Serien‑R **R1–R4 bleiben**. Die Wand‑PCB braucht kein eigenes
> ESD‑Array (s. § 8.11).

### 2.2 Audio‑PCB

| Designator | Bauteil | Aufgabe |
|---|---|---|
| J1 | DC‑Hohlbuchse 12 V 5,5/2,5 | Power‑Entry |
| F1 | Polyfuse 1,5 A | Überstromschutz (→ Nebenbefund § 7) |
| **D3** | SS34 Schottky 40 V/3 A | Verpolschutz |
| MP2307 | DC‑DC‑Step‑Down‑Modul | 12 V → 5 V |
| C9 | 100 nF | Buck In Bypass |
| C10 | 220 µF | Buck Input |
| C11 | 100 nF | Buck Out Bypass |
| C8 | 470 µF | Buck Output |
| U2 | GY‑PCM5102 DAC‑Modul | I²S → Line‑Level |
| C6 | 100 nF | DAC Bypass |
| C7 | 10 µF | DAC |
| U3 | TPA3110 XH‑A232 Verstärker‑Modul | Line → Speaker, 12 V |
| C5 | 100 nF | Amp Bypass |
| C4 | 1000 µF 16 V | Amp Bulk‑Elko |
| **CN4** | Speaker‑Klemme (2601‑1102) | → Visaton FR 10/4 |
| **CN5** | **JST‑XH 8‑pol (S8B‑XH‑A‑1)** | Inter‑Board‑Kabel zum Logic‑PCB |

---

## 3. Inter‑Board‑Interface: `CN3` (Logic) ↔ `CN5` (Audio), JST‑XH 8‑pol, 26 AWG

| Pin | Netz | Richtung | Hinweis |
|---|---|---|---|
| 1 | +5V | Audio → Logic | vom Buck |
| 2 | +5V | Audio → Logic | gedoppelt (Drop/Reserve) |
| 3 | GND | gemeinsam | |
| 4 | GND | gemeinsam | gedoppelt |
| 5 | I2S_BCLK | Logic → Audio | ESP → DAC |
| 6 | I2S_LRC | Logic → Audio | ESP → DAC |
| 7 | I2S_DOUT | Logic → Audio | ESP → DAC |
| 8 | XSMT | Logic → Audio | ESP (XSMT‑Pin lt. Schaltplan) → PCM5102 XSMT (Mute, **wie im funktionierenden Single‑Board**) |

Auf **beiden** Schaltplänen identisch benennen – es gibt keine
board‑übergreifende ERC, die Netznamen sind der einzige Abgleich.

> **Kein Redesign – 1:1‑Abbild des funktionierenden Boards.** Der Split bildet
> exakt das bestehende, im Betrieb bewährte Single‑Board ab; es wandern nur
> die Schnitt‑Netze aufs Kabel. Daher kreuzt **`XSMT` genau wie gehabt**
> (ESP GPIO 18 → PCM5102). Die Mute‑Strategie bleibt unverändert, `STBY` des
> TPA3110 wird **nicht** angefasst.

---

## 4. Stab‑Connector `CN1` (6‑pol Schraubklemmstecker KF2EDGRC‑5.08‑6P)

| Pin | Netz | Funktion |
|---|---|---|
| 1 | +5V | Versorgung Stab (IR‑LED, Laser, 6× NeoPixel) |
| 2 | GND | Masse (puffert 5 V gegen die Signal‑Pins) |
| 3 | NEO_DATA_5V | NeoPixel‑Daten (5 V, nach U1 + R1, ESD über D1) – **kein Cap** (800 kHz) |
| 4 | IR_TX | IR‑Steuerung 3,3 V (nach R2, ESD über D2) → BSS138‑Gate im Stab – **kein Cap** (38 kHz) |
| 5 | LASER_TX | Laser‑Steuerung 3,3 V (nach R3 + Filter‑Cap, ESD über D2) → BSS138‑Gate |
| 6 | TRIG_IN | Trigger active‑LOW (nach R4 + Filter‑Cap, ESD über D1), ESP `INPUT_PULLUP` |

Belegung im Schaltplan verifiziert 2026‑06‑08 (CN1 = KF2EDGRC‑5.08‑6P).
Reihenfolge‑Begründung + Wand‑Seite: § 8.11 / Doc 13. Kabel wandseitig
direkt angelötet. Silkscreen: `1:5V 2:GND 3:NEO 4:IR 5:LAS 6:TRIG`.

---

## 5. Netz‑Aufteilung (lokal vs. kreuzend)

| Netz | Logic | Audio | kreuzt (CN3↔CN5) |
|---|:--:|:--:|:--:|
| +12V / +12V_RAW | – | ✅ | – |
| +5V | ✅ (Verbrauch) | ✅ (Erzeugung) | **✅** |
| +3V3 | ✅ (ESP‑LDO) | ✅ (DAC‑LDO) | – (je lokal) |
| GND | ✅ | ✅ | **✅** (gemeinsam) |
| I2S_BCLK / I2S_LRC / I2S_DOUT | Quelle (ESP) | Senke (DAC) | **✅** |
| XSMT | Quelle (ESP, XSMT‑Pin) | Senke (PCM5102) | **✅** |
| AUDIO_L | – | ✅ (DAC → Amp) | – |
| SPK_L+ / SPK_L‑ | – | ✅ | – |
| STBY (TPA3110) | – | wie im Single‑Board (nicht ESP‑gesteuert) | – |
| NEO_DATA_33V / _5V_AHCT / _TO_LED / _5V | ✅ | – | – |
| IR_TX(_F) / LASER_TX(_F) / TRIG_IN(_F) | ✅ | – | – |
| I2C_SDA / I2C_SCL / K1–K4 | ✅ | – | – |
| TSOP_IN / RSV_* / U0RXD/TXD / UART_* | ✅ | – | – |

**Merksatz:** Nur **+5 V, GND, I²S×3, XSMT** verlassen je ein Board. 3,3 V
und 12 V bleiben strikt lokal.

---

## 6. Vorgehen in EasyEDA

1. **Ein Projekt, zwei Schaltpläne** anlegen: `Station_Logic.sch` und
   `Station_Audio.sch` (zwei eigenständige Schaltpläne, **nicht** zwei Sheets
   eines Schaltplans).
2. Im bestehenden Single‑Schaltplan den **Audio‑Block** (J1, F1, D3, MP2307
   + C8–C11, U2 + C6/C7, U3 + C4/C5, CN4) auswählen und nach
   `Station_Audio.sch` kopieren. Den **Rest** nach `Station_Logic.sch`.
3. Auf Logic: `USBC1` durch das **6‑pol Klemmstecker‑Symbol** (`CN1`) ersetzen
   (Pinbelegung nach § 4) und das alte 4‑Kanal‑`USBLC6‑4SC6Y` durch
   **D1 + D2 (2× USBLC6‑2SC6)** ersetzen (Pass‑Through inline, s. § 2.1).
4. Auf beiden Schaltplänen ein **JST‑XH‑8‑Symbol** platzieren – `CN3` (Logic)
   bzw. `CN5` (Audio) – Netze nach § 3 **identisch** benennen (inkl. `XSMT`
   auf Pin 8 – die durchtrennte Mute‑Leitung vom ESP zum DAC).
5. Pro Schaltplan **„Convert to PCB"** → zwei PCB‑Dokumente.
6. Beide PCBs: Befestigungsbohrungen **Ø 3,2 mm (M3)** an deckungsgleichen
   Positionen (Standoff‑Stack), hohe Module nach außen, Anschlüsse an die
   zur Klappe zeigende Kante (s. § 8.14).

---

## 7. Nebenbefund / Hinweise

> **Grundsatz:** Es wird **nichts an der bewährten Schaltung geändert** –
> reiner Split + Connector‑Tausch. Die Punkte hier sind nur Hinweise, keine
> Pflicht‑Änderungen.

- **F1 = 1,5 A Polyfuse:** lief im bisherigen Aufbau problemlos → **bleibt**.
  (Nur falls beim Neuaufbau die Sicherung bei sehr lautem Bass mal auslösen
  sollte, könnte man auf ~2,5–3 A gehen – aktuell kein Handlungsbedarf.)
- **Status‑LED `H1`** bleibt an der Logic‑PCB‑NeoPixel‑Kette (5 V +
  NEO_DATA_TO_LED), wie gehabt.
- **OLED `CN2`** als optionaler Bestückungs‑Footprint auf Logic (s. § 8.14).

---

## 8. PCB‑Layout – Stand V1 (2026‑06‑08)

Alle drei Boards sind in EasyEDA geroutet, **DRC‑sauber** und bereit zur
JLCPCB‑Bestellung (ggf. zusammen panelisieren). 2‑Layer, 1 oz Kupfer,
hand‑bestückbar. Silkscreen‑Titel: „WandStation V4 – Data/Audio PCB V1 06/26",
Wand „INFINITAG WAND V4", alle `www.hallow-tech.de`.

**Design‑Rules (in EasyEDA als Net‑Class‑Rules gesetzt):**

| Rule | Track min | Clearance min | Via Dia | Via Drill | Netze |
|---|---|---|---|---|---|
| Default (Signal) | 0,25 mm | 0,2 mm | 0,6 mm | 0,3 mm | alle Signale |
| Power | 0,5 mm | 0,2 mm | 0,8 mm | 0,4 mm | +5V, +12V, +12V_RAW, D3_2, GND |
| Speaker (nur Audio) | 0,8 mm | 0,2 mm | 0,8 mm | 0,4 mm | SPK_L+, SPK_L− |

**GND:** auf allen Boards **Copper‑Pour beidlagig + Stitching‑Vias** (Top‑ und
Bottom‑GND verbunden, niederohmiger Rückpfad, Schirmung).

**SMD‑Elko‑Umbau (Audio‑Board, THT → SMD‑Alu‑Elko aus Sortiment‑Kit):**

| Ref | Wert | Gehäuse | Footprint |
|---|---|---|---|
| C4 | 1000 µF 16 V (Amp‑Bulk, +12 V) | 10 × 10,5 mm | CAP‑SMD_BD10.0 |
| C10 | 220 µF 35 V (Buck‑Eingang) | 8 × 10,5 mm | CAP‑SMD_BD8.0 |
| C8 | 470 µF 6.3 V (Buck‑Ausgang, 5 V) | 6,3 × 7,7 mm | CAP‑SMD_BD6.3 |

**ESP‑Footprint:** Reihenabstand des DevKitC am realen DiYmore‑Board
nachgemessen und Footprint getauscht – der ursprüngliche
`CONN-TH_ESP32-S3-DEVKITC-1U-N8` hatte die Pin‑Reihen zu eng. Vor Serie
1:1‑Druck‑Gegencheck.

**Inter‑Board‑Kabel:** `CN3` (Data) ↔ `CN5` (Audio), beide **S8B‑XH‑A‑1
(LCSC C5306549)**, JST‑XH 8‑pol gewinkelt, 26 AWG. **Identische Belegung →
Kabel 1:1 gerade crimpen** (Pos 1↔1 … 8↔8).

**Wand‑Kabel:** 6‑adriges Mantelkabel an **2×3‑Lötheader `J1` (HDR‑2X3‑2.54)**
auf der Wand‑PCB angelötet, Belegung identisch zur Station‑Klemme `CN1`
(1:5V 2:GND 3:NEO 4:IR 5:LAS 6:TRIG) → ebenfalls gerades 1:1‑Kabel.

### 8.1 Pre‑Order‑Checkliste (pro Board)

- [ ] **DRC grün** (nach dem Setzen der Stitching‑Vias erneut laufen lassen).
- [ ] **„Unrouted nets" = 0** (DRC‑OK ≠ fertig geroutet – explizit prüfen).
- [ ] **Keine isolierten Kupfer‑Inseln** im Pour („Remove Dead Copper").
- [ ] **Footprints per 1:1‑PDF‑Druck + 3D verifizieren** – kritisch:
  - **ESP‑DevKitC** (Reihenabstand),
  - **Cherry‑MX 5‑Pin PCB‑Mount** (Wand, SW1) – echten Switch auflegen,
  - **WS2812B‑4020** (Wand, LED1–4) – Pad‑Geometrie liegend/stehend,
  - **6‑pol Schraubklemmstecker CN1** (Station) – Plug‑Raster 5,08 mm,
  - JST‑XH‑Header (CN3/CN5), 2×3‑Header (J1).
- [ ] **Stack‑Bohrungen Ø 3,2 mm (M3)** Data ↔ Audio deckungsgleich, CN3/CN5
  an der zueinander zeigenden Kante (kurze Kabelschleife).
- [ ] **Silkscreen‑Legenden** an den feldverdrahteten Steckern (CN1‑Stab,
  J1‑Wand: `1:5V 2:GND 3:NEO 4:IR 5:LAS 6:TRIG`; CN3/CN5: `1/2:5V 3/4:GND
  5:BCLK 6:LRC 7:DOUT 8:XSMT`).

### 8.2 Finale Designatoren (Quick‑Reference)

| | Data‑PCB | Audio‑PCB | Wand‑PCB |
|---|---|---|---|
| MCU/Module | BB1 (ESP32‑S3) | U2 (PCM5102), U3 (TPA3110), MP2307 | — |
| Connector Kabel↔Station | CN3 (Inter‑Board) | CN5 (Inter‑Board) | J1 (2×3 zum Kabel) |
| Stab/Speaker | CN1 (Stab‑Klemme) | CN4 (Speaker) | CN1/CN2 (IR/Laser) |
| OLED | CN2 | — | — |
| Levelshifter | U1 (74AHCT125) | — | — |
| ESD | D1, D2 (USBLC6‑2SC6) | — | — (ESD auf Station) |
| Schottky | — | D3 (SS34) | — |
| Trigger/LED | — | — | SW1 (MX), LED1–4 (WS2812B), Q1/Q2 (BSS138) |

---

## 9. Verweise

- Stack‑Mechanik, Strom‑Architektur, EasyEDA‑Grundsatz:
  [`12-refactor-station-v2.md`](12-refactor-station-v2.md) § 8.14
- Stab‑Connector (6‑pol Klemmstecker, ESD): § 8.11
- Wand‑Seite (angelötetes Kabel, **kein** eigenes ESD‑Array – ESD sitzt auf
  der Station): [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md) § 4
- Entscheidung + Cross‑Doc‑Sync: [`11-offene-punkte.md`](11-offene-punkte.md)
  Punkt 40
- GPIO‑Plan v7: [`12-refactor-station-v2.md`](12-refactor-station-v2.md) § 7
