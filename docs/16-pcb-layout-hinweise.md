# 16 – PCB‑Layout‑Hinweise Station V2

> **Status: lebende Sammlung**, wird beim Schaltplan‑Bauen mitgeführt.
> Sobald wir ins PCB‑Layout wechseln, ist das hier die Checkliste, gegen
> die geroutet wird. Nicht im Schaltplan sichtbare Regeln (Trace‑Breite,
> Platzierung, Massefläche) landen hier — Schaltplan‑Logik bleibt in
> Doc 12 + Doc 14.
>
> Begonnen: 2026‑05‑14.

---

## 1. Versorgung & Bypass

### 1.1 Eingangs‑Bypass am Buck (C1 + C2)

- **C1 (100 nF Keramik 0805)**: so nah wie irgend möglich am MP2307‑VIN+‑Pin
  platzieren, **vor** dem Elko. HF‑Bypass funktioniert nur bei kurzer
  Schleife → ≤ 5 mm Trace‑Länge zum VIN+‑Pin.
- **C2 (220 µF Elko TH)**: maximal 10 mm vom VIN+‑Pin entfernt. Längere
  Strecken sind tolerierbar, weil der Elko niederfrequenten Lastpuffer
  liefert, aber kürzer ist besser.
- **GND‑Anschluss** beider Cs direkt auf die Massefläche, **nicht** über
  eine schmale Trace.
- Ordnung von Buck‑Pin nach außen: VIN+ → C1 → C2 → DC‑Buchse.

### 1.2 Ausgangs‑Bypass am Buck (C3 + C4)

- **C3 (100 nF Keramik 0805)**: ≤ 5 mm vom MP2307‑VOUT+‑Pin entfernt.
- **C4 (470 µF Low‑ESR Elko TH)** *oder* 2 × 220 µF parallel:
  unmittelbar nach C3, möglichst nah am TPA3110‑VIN.
- Diese Cs bilden den **lokalen Lastpuffer** für die Sound‑Peaks; je
  länger die 5 V‑Trace zwischen Buck‑Output und TPA, desto wichtiger
  ist ein zusätzlicher 1000 µF nahe am TPA selbst.

### 1.3 12 V‑Trace dick auslegen

- Trace‑Breite von DC‑Buchse über F1 + D1 zum Buck‑VIN+: **≥ 1 mm**
  bei 1 oz Kupfer. Berechnung: 1,2 A bei ΔT 10 K → 0,8 mm reicht
  rechnerisch, 1 mm gibt Reserve.
- Bei doppelseitigem PCB die 12 V‑Versorgung **nicht** ins
  Top‑Polygon‑Pour aufnehmen — eigene Polyline/Track, damit beim
  Routing keine Schleife durch hochfrequente Logik entsteht.

---

## 2. Audio‑Pfad

### 2.1 Stern‑GND am Buck‑Output

- **Stern‑Punkt** ist der GND‑Pin des Buck‑Output (= MP2307‑VOUT‑).
- DAC‑GND, TPA‑GND und Speaker‑GND treffen sich an diesem Punkt.
- Logik‑GND (ESP, OLED, Tasten, NeoPixel‑Logik) liegt auf der allgemeinen
  Massefläche und trifft den Stern‑Punkt über die Fläche — nicht über
  separate Tracks.
- Stern‑Punkt im Layout markieren (Test‑Pad oder Beschriftung), damit
  später nachvollziehbar.

### 2.2 I²S‑Tracks

- BCLK, LRC, DOUT zum PCM5102A: kurz halten, **≤ 30 mm**.
- Direkt unter den I²S‑Tracks **Massefläche** im darunterliegenden Layer
  führen → wirkt als Mikrostrip‑Rückleiter, reduziert EMV.
- I²S‑Tracks **nicht** parallel zu Lautsprecher‑Leitungen routen.

### 2.3 Speaker‑Pads

- Speaker‑Anschluss (2‑pol Schraubklemme oder JST‑XH) **≤ 30 mm** vom
  TPA3110‑Output entfernt.
- Track‑Breite ≥ 0,8 mm (bis zu 1 A Peak‑Strom an 4 Ω bei Vollast).

---

## 3. Schalter‑MOSFETs (IR + Laser) — sitzen im Stab, nicht in der Station

**Architektur-Entscheidung 2026-05-14:** Die BSS138-MOSFETs für IR-LED und
Laser sitzen auf dem **Stab-PCB** (Doc 13 § 4.1 + 4.1.1), nicht im
Station-PCB. Im Station-PCB wird `IR_TX` (GPIO 5) und `LASER_TX` (GPIO 7)
direkt als 3,3 V-Logiksignal auf die USB-C-Stab-Buchse geroutet.

### 3.1 Was die Station noch tut

- `IR_TX` (GPIO 5) und `LASER_TX` (GPIO 7) als kurze 3,3 V-Logik-Trace
  zum USB-C-Connector führen.
- **Series-Widerstand 100 Ω** zwischen ESP-GPIO und USB-C-Pin nahe am
  Stecker → schützt gegen ESD-Transienten beim Stecken/Ziehen.
- **TVS-Diode-Array** an den USB-C-Signaladern (siehe Punkt 5 unten).

### 3.2 Was aufs Stab-PCB gehört (Doc 13 — hier nur Referenz)

- Gate-Pulldown 10 kΩ am BSS138, Series-R 820 Ω am Gate.
- IR-LED-Vorwiderstand 33 Ω 0,5 W in Serie zur TSAL6100.
- Laser-Modul direkt am MOSFET-Drain, kein zusätzlicher Vorwiderstand.
- Trace-Breite zum IR-LED-Pad ≥ 0,3 mm, zum Laser ≥ 0,2 mm.

---

## 4. NeoPixel‑Pfad

### 4.1 Level‑Shifter 74AHCT125

- Eingangsseite (3,3 V vom ESP) und Ausgangsseite (5 V zum SK6812)
  räumlich trennen — Signal‑Routing nicht über die VCC‑Trace des
  Shifters führen.
- **Bypass 100 nF** direkt an VCC des AHCT125 (Pin 14).
- Unbenutzte Gate‑Eingänge (Pin 4, 7, 9 = 3OE/4OE bzw. 3A, 4A) auf VCC
  oder GND legen — nicht floatend.

### 4.2 Daten‑Trace zum ersten SK6812

- **470 Ω Series‑R** in der Datenleitung direkt am Ausgang des AHCT125,
  vor der Verlegung zum Pad → schützt vor Ringing über das längere Kabel.
- 1000 µF Low‑ESR Elko zwischen SK6812‑+5 V und GND nahe der ersten
  LED.

### 4.3 Station‑LED‑Connector J5 mit Plastik‑Jumper

**Architektur (aktualisiert 2026‑05‑14):** Die Station hat einen
**4‑pol Pin‑Header Stiftleiste (J5)** für optionale externe Station‑LEDs.
Default kein LED‑Strang angeschlossen → ein **Standard‑Mainboard‑Jumper**
über Pin 2/3 überbrückt die Daten‑Leitung (DIN↔DOUT).

```
AHCT.1Y → R7 (470Ω) → NEO_DATA_TO_LED ── J5.Pin 2 (DIN)
                                          │
                                       [Jumper Pin 2↔3, default gesteckt]
                                          │
                  NEO_DATA_5V_F ──────── J5.Pin 3 (DOUT)
                                          │
                                          ↓
                                   → Filter R2 → USB-C

J5.Pin 1 = +5V (für externe LEDs)
J5.Pin 4 = GND (für externe LEDs)
```

**Modi:**

- **Default (keine Station‑LEDs):** Jumper auf Pin 2/3 → Daten gehen direkt zum USB‑C
- **Mit Station‑LEDs:** Jumper raus, externe LED‑Kette an alle 4 Pins anschließen — Pin 1=+5V, Pin 2→LED.DIN, letzte LED.DOUT→Pin 3, Pin 4=GND. Im Sketch dann `NEOPIXEL_COUNT = STATION_LEDS + STAB_LEDS`

**Layout‑Hinweis:** J5 nahe an R7 platzieren (Trace ≤ 30 mm zum Stab‑
USB‑C‑Anschluss), Pin 2/3 nicht durch andere Bauteile blockiert (Jumper
braucht Greif‑Zugang). Mechanik‑Höhe ~10 mm beachten beim Gehäuse‑Design.

### 4.4 Vor‑Version (verworfen)

Der ursprünglich vorgeschlagene 0 Ω SMD‑SJ1‑Lötjumper entfällt — durch
den Plastik‑Jumper auf der Stiftleiste werkzeuglos umsteckbar, kein
Lötkolben nötig.

### 4.4 NeoPixel‑Bypass‑Lötjumper früherer Vorschlag (verworfen)

Der ursprünglich in dieser Doku vorgeschlagene 3‑Pad‑Lötjumper
„AHCT‑Level‑Shifter umgehbar" entfaellt — der 74AHCT125 ist nach Doc 14
§ 2.6 zwingend bestueckt, kein Umgehen sinnvoll.

---

## 5. Stab‑Connector (USB‑C)

- Footprint mit **allen 24 Pads** routen, auch wenn wir nur 6 Adern
  nutzen — erlaubt späteren Wechsel zwischen Plan A (6 Adern) und Plan
  B (4 Adern) ohne PCB‑Re‑Spin.
- ESD‑/Filter‑Schutz am Trigger‑Eingang (GPIO 4):
  - 10 kΩ Pullup auf 3,3 V nahe am Connector
  - 100 nF Bypass gegen GND nahe am Connector
  - Optional: 1N4148 von Pin zu 3V3 (ESD‑Klemmschutz)

---

## 6. Antenne & WLAN

- **Pigtail‑Verlegung**: Antennenkabel nicht durch das Speaker‑Volumen
  führen, nicht parallel zu I²S‑Tracks. Eigener Kanal durch das
  Druckteil.
- Auf dem PCB den IPEX‑Anschluss am ESP‑Modul **frei** halten — keine
  hohen Bauteile in 10 mm Umkreis.

---

## 6.5 Footprint‑Verifikation für aufgelötete Module

Diese Module werden direkt mit ihren seitlichen Lötpads aufs PCB
gelötet (kein Pin‑Header):

- MP2307‑Mini Buck‑Modul (U2)
- Eventuell zukünftig: PCM5102A, TPA3110‑XH‑A232, falls sie als
  Solder‑Pad‑Variante verfügbar sind

**Vor JLCPCB‑Bestellung** für jeden dieser Module:

1. **Schieblehre nehmen**, Pad‑Abstände in beiden Richtungen am echten
   Modul messen.
2. EasyEDA → PCB‑Editor → Footprint öffnen → Pad‑Maße ablesen.
3. **Toleranz < 0,2 mm** zwischen beidem.
4. Wenn der EasyEDA‑Footprint zu kleine Pads hat: im PCB‑Editor die
   Pads auf **mindestens 1,5 mm × 2,0 mm** vergrößern → bessere
   mechanische Festigkeit beim Lötring.
5. Modul‑Pin‑Reihenfolge prüfen (IN+/IN−/OUT+/OUT− vs.
   IN+/OUT+/IN−/OUT−), Symbol‑Pin‑Namen entsprechend anpassen.

User‑Contributed‑Symbole und ‑Footprints aus EasyEDA Library sind
**nicht garantiert korrekt** — diese Verifikation ist Pflicht vor
Bestellung.

---

## 7. Mechanische Layouts

### 7.1 DC‑Hohlbuchse

- Direkt an die Boden‑Klappe‑Außenwand des Gehäuses platzieren
  (siehe Doc 12 § 8).
- Hohlbuchsen‑Schraubmutter‑Spielraum: 12 mm Durchmesser frei lassen
  rund um den Buchsenkörper.

### 7.2 OLED‑Stecker

- JST‑XH 8‑pol an der Kante, die zur Klappenscharnier‑Seite zeigt,
  damit das Kabel beim Auf‑/Zuklappen nicht spannt.

### 7.3 Reserve‑Header J_RSV

- An der Kante platzieren, die nach Service‑Öffnung schaut → mit
  USB‑zu‑UART‑Adapter ansteckbar ohne PCB ausbauen.

---

## 7.3 Finale Board‑Maße + JLCPCB‑Konfiguration (Bestellung 2026‑05‑14)

| Parameter | Wert |
|---|---|
| Board‑Maße | **100 × 76 mm** |
| Layer | 2 |
| Kupfer | 1 oz |
| Thickness | 1,6 mm |
| Farbe | Grün |
| Silkscreen | Weiß |
| Surface Finish | LeadFree HASL |
| Via Covering | Tented |
| Min Via Drill | 0,3 mm |

Stueckzahl 5, Lieferung erwartet 27.05.–02.06.2026.

## 7.4 Design Rules für JLCPCB (2026‑05‑14)

**Default Rule (Signale):**

| Parameter | Wert |
|---|---|
| Track Width min | 0,25 mm |
| Clearance min | 0,2 mm |
| Via Diameter min | 0,6 mm |
| Via Drill min | 0,3 mm |

**Power Rule (High Voltage)** für `+12V`, `+12V_RAW`, `F1_2`, `GND`:

| Parameter | Wert |
|---|---|
| Track Width min | 1,0 mm |
| Clearance min | 0,2 mm |

**Power_LV Rule** für `+5V`, `+3V3` (eigene Rule, weil 1,0 mm zu breit für USB‑C‑VBUS‑Pads):

| Parameter | Wert |
|---|---|
| Track Width min | 0,5 mm |
| Clearance min | 0,2 mm |

**Audio-Tracks** (optional als eigene Rule):
- AUDIO_L: 0,4 mm mit Massefläche darunter
- SPK_L+, SPK_L-: 0,8 mm

**GND:** als **Polygon-Pour 100 %** auf beiden Layern, nicht als Track.

**Routing-Reihenfolge:**
1. Power-Nets zuerst
2. Kritische Signale (I²S, I²C, USB-C-Signale)
3. Restliche Signale
4. GND-Polygon-Pour zuletzt

---

## 8. Test‑Pads + Designators

- Stern‑GND‑Punkt: kleines Pad mit Beschriftung `★GND`
- 5 V‑Rail: Test‑Pad `TP_5V`
- 12 V‑Rail (nach D1): Test‑Pad `TP_12V`
- XSMT: Test‑Pad `TP_XSMT`
- I²S BCLK: Test‑Pad `TP_BCLK`

→ jeweils 1 mm Lötpunkt, kein Pin‑Header nötig.

---

## Aenderungshistorie

- **2026‑05‑14**: Datei angelegt, erste Hinweise für Buck‑Bypass C1/C2,
  Audio‑GND, MOSFETs, NeoPixel‑Pfad, USB‑C, Antenne, Test‑Pads.
