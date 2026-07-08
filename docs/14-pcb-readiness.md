# 14 – PCB‑Readiness‑Checkliste Station V2

> **Status: PCB‑Freigabe für Prototyp‑Bestellung erteilt am 2026‑05‑14.**
> Sieben kritische Punkte (2.1 Rauschen, 2.2 Buck, 2.3 Trigger, 2.4 IR,
> 2.5 Laser, 2.6 NeoPixel‑Levelshifter, 2.8 Mute) sind durchgemessen und
> dokumentiert. Drei verbleibende Punkte (2.7 ESP‑NOW, 2.9 Kombinations‑
> test, 2.10 USB‑C 6‑Adern) sind als **Verifikationspunkte am bestückten
> Prototyp** abgeschoben – sie betreffen keine layoutkritische Bauteil‑
> Entscheidung, sondern System‑Verhalten / Antennen‑Reichweite, was am
> Lochraster ohnehin nur eingeschränkt simulierbar wäre.
>
> Konsequenz: Schaltplan und PCB‑Layout dürfen mit dem in Doc 12 + diesem
> Doc dokumentierten Stand entworfen werden. Empfehlung: **erste Charge
> klein halten (5–10 Boards von JLCPCB)**, davon nur 1–2 bestücken für
> den Bring‑up.

Begonnen 2026‑05‑14. Lebende Datei – wird beim Abarbeiten direkt
fortgeschrieben (Datum + Ergebnis pro Punkt).

---

## 1. Zweck

Ein PCB‑Re‑Spin kostet 2–4 Wochen Wartezeit und 30–60 € pro Iteration.
Diese Datei hält fest, **welche Schaltungs‑ und Verkabelungs‑Entscheidungen
auf dem Lochraster final getestet sein müssen**, bevor das Station‑V2‑PCB
in Auftrag geht. Jeder Punkt ist so formuliert, dass der Test ein
eindeutiges *Bestanden / Nicht bestanden* liefert.

Reihenfolge spiegelt das Risiko fürs PCB:

| Kategorie | Risiko ohne Test | Aktion bei nicht‑bestanden |
|---|---|---|
| **KRITISCH** | falsche Bauteile / Pins / Filter im Layout | Layout anpassen |
| **vor PCB sinnvoll** | kleine Re‑Spins (Stecker‑Footprint, Pullup‑Werte) | nachziehen oder als Verifikation am bestückten PCB einplanen |
| **später** | hat keinen Einfluss auf das Kupfer | im laufenden Code/Mechanik weiter‑machen |

---

## 2. KRITISCH (vor Bestellung)

Stand pro Punkt: `OFFEN` / `IN ARBEIT` / `OK YYYY‑MM‑DD (Notiz)`.

### 2.1 Rauschen‑Quelle isolieren

**Test:** Mit dem aktuellen Audio‑Aufbau das hörbare Rauschen
schrittweise eingrenzen. Reihenfolge bewusst von „verändert nichts an
Hardware" zu „braucht Bauteile":

1. **XSMT‑Test.** XSMT auf LOW lassen (DAC stumm) → Rauschen weg?
   - **Ja** → Quelle liegt **vor oder im DAC** (I²S‑Signal, Versorgungs‑
     Welligkeit am DAC, Pin‑Quersprechen).
   - **Nein** → Quelle liegt **nach dem DAC** (TPA3110‑Eingang, Speaker‑
     Leitung, Speaker‑GND‑Schleife).
2. **Versorgungs‑Test.** ESP nicht mehr über USB‑C, sondern den gesamten
   Aufbau nur über 12 V → Buck → 5 V speisen.
   - **Schlimmer** → Buck‑Down ist die Quelle (typisch billiger LM2596:
     150 kHz Schaltbrumm hörbar als Pfeifen/Sirren).
   - **Besser** → vorher gab es eine Masseschleife zwischen Laptop‑USB
     und Speaker‑GND.
3. **I²S‑Leitungen** auf ≤ 10 cm kürzen, BCLK/LRC/DOUT verdrillt mit
   einer GND‑Litze gemeinsam führen. Speaker‑Kabel nicht parallel zu
   I²S verlegen.
4. **Audio‑Masse einsternen.** DAC‑GND und TPA3110‑GND treffen sich an
   **einem** Punkt (am Buck‑Output‑Kondensator‑Minus), nicht über das
   Breadboard verteilt.
5. **Bypass‑Kondensatoren** aus Doc 12 Abschnitt 5.6 dranlöten:
   100 nF + 10 µF nahe an DAC‑VIN und TPA‑VIN.
6. Wenn Rauschen jetzt immer noch hörbar:
   **LC‑Filter** (10 µH + 100 µF) zwischen Buck‑Out und TPA‑VIN.

**Konsequenz für das PCB:**

| Test‑Ergebnis | PCB‑Layout‑Folge |
|---|---|
| Rauschen bei Schritt 1 weg (XSMT) | DAC‑VIN bekommt eigenen LDO oder Pi‑Filter im Layout |
| Rauschen bei Schritt 2 weg (Buck) | Buck und Audio‑VIN trennen; ggf. zweiter Buck oder LDO nur für DAC |
| Rauschen bei Schritt 5/6 weg | Bypass‑/LC‑Komponenten gehören fest aufs Layout |
| Rauschen bleibt | TPA3110‑Modul wechseln (XH‑A232 hat schlechten Ruf bei manchen Chargen) oder MAX98357 als Fallback |

**Status: OK 2026‑05‑14** – Quelle ist *nach* dem DAC, dominanter Faktor
ist die mechanische Breadboard‑Verkabelung. Kein extra Analog‑LDO, kein
Pi‑Filter nötig; Standard‑Bypass aus Doc 12 Abschnitt 5.6 genügt.

**Messung (Sound‑Meter‑App, Mikro ~10 cm vor Speaker, USB‑C vom Mac
abgezogen, Versorgung nur 12 V):**

| Zustand | MIN | AVG | MAX | Δ zu Raumgrund |
|---|---|---|---|---|
| Raum, System aus | 34 | 37 | 40 | — |
| **Mute** (XSMT LOW, DAC stumm) | 55 | 61 | 63 | +24 dB |
| **DAC idle** (XSMT HIGH, I²S Stille) | 60 | 60 | 61 | +24 dB |
| Nach Wackeln an Kabeln/Display | 46 | 46 | 47 | **+7 dB** |

**Auswertung:**

- Mute = Idle (Δ 0 dB innerhalb Messtoleranz) → der **DAC selbst ist
  nicht die Quelle**. Wäre er es, müsste Idle deutlich lauter sein.
- 14 dB Reduktion durch Wackeln (60 → 46 dB, gefühlt Faktor ≈ 5) ist
  *typisch* für lose Breadboard‑Kontakte/Federn und parallele Jumper‑
  Wires, *untypisch* für eine PCB‑Schaltung. Auf dem fertigen PCB mit
  Lötverbindungen entfällt dieser Anteil komplett.
- Restrauschen auch im Idle vorhanden → TPA3110‑Eingang ist hochohmig
  und fängt EM‑Einstreuung von I²S‑Takt + WLAN + Buck‑Schaltflanke,
  sobald die Eingangs‑GND‑Federn nicht fest sitzen.

**PCB‑Layout‑Konsequenz:**

| Maßnahme | Status |
|---|---|
| Bypass 100 nF + 10 µF nahe DAC‑VIN | aus Doc 12 5.6 übernehmen ✓ |
| Bypass 100 nF + 470 µF nahe TPA‑VIN | aus Doc 12 5.6 übernehmen ✓ |
| Stern‑GND am Buck‑Output | aus Doc 12 5.6 übernehmen ✓ |
| I²S‑Tracks kurz, Massefläche darunter | Routing‑Hinweis aufs Layout |
| Speaker‑Pads ≤ 30 mm vom TPA entfernt | Mechanik‑/Layout‑Hinweis |
| Separater Analog‑LDO für DAC | **NICHT NÖTIG** |
| LC‑Filter Buck → TPA | **NICHT NÖTIG** |

---

### 2.2 12 V → 5 V Buck unter Last

**Test:** Aufbau ausschließlich über 12‑V‑Hohlbuchse versorgen, Buck‑Down
auf 5 V dran, ESP + DAC + TPA + OLED + Speaker dran. Vier Messungen
parallel:

- 5‑V‑Rail mit Multimeter (im DC‑Modus) → soll zwischen 4,9 V und 5,1 V stehen, auch beim Sound‑Peak.
- 5‑V‑Rail mit Oszi (AC‑Kopplung, 20 mV/div) → Ripple < 50 mVpp, kein „Sirren"‑Muster.
- 12‑V‑Rail beim Sound‑Peak → kein Einbruch (TPA zieht da kurz Ampere).
- Buck‑Spule und Buck‑Chip nach 10 min Sound → handwarm, nicht heiß.

**Konsequenz:** Wenn der Buck einbricht oder rauscht, muss das PCB‑Layout
**zwei** Wege fahren: 12 V direkt zum TPA, 5 V vom Buck nur für die
3,3‑V‑Logik und das Display. DAC ggf. an einen kleinen LDO (z. B.
AMS1117‑3.3 oder LP5907 für sauberen Analog‑3,3 V).

**Status: OK 2026‑05‑14** – Buck‑Output ist exzellent stabil, der einzige
größere 12 V‑Drop sitzt in der Test‑Verkabelung (Hohlbuchsen‑Litze) und
ist auf dem PCB hinfällig. Ripple‑Messung am Oszi als optionaler
Verifikationspunkt zurückgestellt (am bestückten PCB nachholen).

**Messung (Fluke 115, MIN/MAX über mehrere Sound‑Peaks, FR 10/4 @ Vol 100 %):**

| Messpunkt | MAX | MIN | Drop | Bewertung |
|---|---|---|---|---|
| 5 V Buck‑OUT (direkt am Modul) | 5,026 V | 5,007 V | **19 mV** | exzellent ✓ |
| 12 V am Buck‑IN | 12,17 V | 11,21 V | **960 mV** | Borderline |
| 12 V an der Wago vor dem Buck | identisch zum Buck‑IN | — | 960 mV | Wago nicht ursächlich |

**Auswertung:**

- **5 V‑Rail steht wie eine Wand.** 19 mV Drop bei 1‑A‑Lastvariation ist
  im Messrauschen verschwindend – das MP2307‑Buck‑Modul mit 4R7‑Spule
  liefert genau das was die Spec verspricht. ESP, DAC und OLED haben
  keine Brown‑Out‑Gefahr, der Mute‑Pop‑Schutz funktioniert
  unabhängig davon.
- **12 V‑Drop von 960 mV bei 1,2 A** entspricht ca. 0,8 Ω Widerstand im
  Pfad – sitzt nachweislich **nach der Wago** in der gelöteten
  Hohlbuchsen‑Litze. Typisches Problem dieser DC‑Pigtail‑Kabel: 0,14 mm²
  oder dünner, dazu der Stecker‑/Innenkontakt der Hohlbuchse. Auf dem
  PCB entfällt das vollständig (Hohlbuchse direkt verlötet, kupferdicke
  Trace zum TPA).

**PCB‑Layout‑Konsequenzen:**

| Maßnahme | Status |
|---|---|
| DC‑Hohlbuchse direkt aufs PCB (kein Pigtail‑Kabel) | im Layout zwingend |
| 12 V‑Trace zum TPA breit (≥ 1 mm bei 1 oz Cu) oder als Polygon | Routing‑Vorgabe |
| 1000 µF Low‑ESR Elko nahe TPA‑VIN als lokaler Lastpuffer | aus Doc 12 5.6 ✓ |
| Buck‑Modul (MP2307) auf dem PCB behalten | bestätigt |
| Separater Analog‑LDO oder zweiter Buck | **NICHT NÖTIG** |

**Halloween‑Realität (Folge‑Punkt):** Für den Vorgarten‑Aufbau braucht
die Station eine ordentliche externe 12 V‑Zuleitung vom Netzteil – nicht
das mitgelieferte Pigtail‑Kabel verlängern, sondern **≥ 0,75 mm²
Installations‑/Lautsprecherlitze** vom Netzteil zur Station‑Hohlbuchse.
Dieses Item ist als Doku‑Hinweis für die Aufbau‑Beschreibung gemerkt
(gehört dann in `09-halloween-setup.md`, sobald 2.* alle durch sind).

**Optionaler Verifikationspunkt am bestückten PCB:** Ripple‑Messung
5 V‑Rail Idle vs. Sound‑Peak mit Rigol DS1202. Schwelle: < 100 mVpp.
Bei den 19 mV DC‑Drop heute ist die Wahrscheinlichkeit sehr hoch, dass
das Bild ohnehin sauber ist – wird beim ersten PCB‑Bring‑up nachgeholt.

---

### 2.3 Trigger‑Eingang (GPIO 4)

**Test:** Drahtbrücke gegen GND legen → ESP soll im `loop()` das
`digitalRead(4) == LOW` sehen. INPUT_PULLUP eingestellt, Debouncing
identisch zum OLED‑Button‑Code.

**Konsequenz:** Wenn das hier nicht stabil ist, braucht das PCB einen
zusätzlichen externen Pullup (10 kΩ gegen 3,3 V) + 100 nF gegen GND
nahe am Stecker, um lange USB‑C‑Kabel‑Kapazitäten zu kompensieren.

**Status: OK 2026‑05‑14** – Mit 6×6 mm Tactile Switch zwischen GPIO 4
und GND validiert. Idle TRIG=1 stabil, Tastendruck zieht sauber auf
LOW, Edge‑Detection zählt korrekt, Debouncing (20 ms) prellt nicht
sichtbar. Sound wird ueber den Trigger‑Pfad mit eigenem `[TRIG]`‑Tag
geloggt – Counter (T) und Sound‑Counter (P) zaehlen synchron hoch.

**PCB‑Layout‑Konsequenz:**

| Maßnahme | Status |
|---|---|
| GPIO 4 als Input, intern Pullup im Code | ✓ |
| ESD‑/Filter‑Beschaltung an der USB‑C‑Buchse (10 kΩ + 100 nF) | aufs Layout, weil USB‑C‑Stab‑Kabel 1–2 m lang ist und Kabelkapazitaet entstoeren muss |
| 100 nF Stuetzkondensator nahe GPIO 4 | aufs Layout |

Verifikation am laengeren USB‑C‑Kabel (echte Stab‑Verkabelung) bleibt
Aufgabe von Punkt 2.10.

---

### 2.4 IR‑LED‑Treiber (GPIO 5 + MOSFET)

**Test:** Lochraster mit AO3400 (oder IRLML2502/IRF7341) als
Low‑Side‑Switch, Drain → IR‑LED Kathode, Source → GND, Gate → GPIO 5,
Gate‑Pulldown 10 kΩ gegen GND. RMT‑Signal mit 38 kHz Träger senden,
Empfang gegen TSOP4838 oder einen vorhandenen Target prüfen.

**Konsequenz:** MOSFET‑Auswahl + Gate‑Pulldown sind PCB‑Bauteile, die
**jetzt** festgelegt werden. Auch der LED‑Vorwiderstand muss zur
Spannungsebene (5 V vs. 3,3 V) und LED‑Spec (TSAL6200, ca. 1,5 V, 100 mA)
passen.

**Status: OK 2026‑05‑14** – Send‑ und Empfangs‑Pfad sauber verifiziert.

**Aufbau (Lochraster):**

| Pin / Anschluss | Verbindung |
|---|---|
| +5 V Buck‑Out | 100 Ω → TSAL6200 Anode |
| TSAL6200 Kathode | BC546C #2 Collector |
| BC546C Base | 820 Ω → GPIO 5 |
| BC546C Emitter | GND |
| TSOP4838 VS | 3,3 V |
| TSOP4838 GND | GND |
| TSOP4838 OUT | GPIO 10 (mit internem Pullup) |

**Frequenzverifikation:** `ledcSetup(CH0, 38000, 8)` erzeugt Ist‑Frequenz
**37986 Hz** – Abweichung 0,04 %, mitten im TSOP4838‑Sweet‑Spot
(Bandbreite ±3 kHz). Frequenz‑Drift unkritisch, auch bei
Resolution‑Wechsel auf 10/12 bit nicht nötig.

**Tests bestanden:**

- Idle: `TSOP=1` stabil, kein Fremdlicht im 38 kHz‑Spektrum erkennbar.
- 1 ms Burst auf K4 oder Trigger:
  ```
  [IR]  Burst #1, 1 ms gesendet. TSOP: empfangen ✓
  ```
- Sound nach IR‑Burst spielt sauber, kein Audio‑Stall.
- Lokaler Test mit ~30 cm Abstand zwischen LED und TSOP. **Reichweiten‑
  Test mit echtem Halloween‑Target steht noch aus** (Tobias hatte beim
  Lochraster‑Test keines vor Ort).

**PCB‑Layout‑Konsequenz (aktualisiert 2026‑05‑14):**

Die MOSFET‑Schaltung wandert **vom Station‑PCB ins Stab‑PCB** (Doc 13
§ 4.1.1). Die Station gibt nur das **3,3 V‑Logiksignal `IR_TX`** über
die USB‑C‑Stab‑Buchse weiter.

| Maßnahme | Wo | Status |
|---|---|---|
| BSS138 (SOT‑23) Low‑Side‑Switch | **Stab‑PCB** | dort aufs Layout |
| Gate‑Pulldown 10 kΩ | Stab‑PCB | dort |
| Series‑Gate‑Widerstand 820 Ω | Stab‑PCB | dort |
| Vorwiderstand TSAL 33 Ω 0,5 W | Stab‑PCB | dort |
| TSAL6100 mit OM3‑Optik | Stab (Optik‑Block) | Doc 13 § 3.2 |
| 100 Ω Series‑R am USB‑C‑Pin | Station‑PCB | ESD‑Schutz |
| TVS‑Diode‑Array USB‑C | Station‑PCB | für Signaladern |

**Offener Verifikationspunkt:** Reichweiten‑Test mit dem echten
Halloween‑Target (V3.2). Tobias hat es bei der Lochraster‑Session nicht
vor Ort gehabt → bei nächster Gelegenheit nachholen, idealerweise auch
gleich Validierung dass das Infinitag‑24‑Bit‑Telegramm mit dem alten
Target‑Sketch noch entschluesselt wird. Aktion: bei Tobias zu Hause.

---

### 2.5 Laser‑Treiber (GPIO 7 + MOSFET)

**Test:** Gleicher Aufbau wie 2.4, aber mit einem Halloween‑Laser‑Modul
(5 V, ca. 30 mA). Mit GPIO 7 schalten, Strom messen, kein Flackern
zwischen Sound‑Wiedergabe und Schaltvorgang.

**Konsequenz:** Identisch zu 2.4 – MOSFET + Pulldown auf das Layout.
Tobias möchte „Zielen ohne Schuss" → Laser und IR‑Treiber müssen
unabhängig schaltbar sein (im v7‑Plan getrennte Pins, das passt).

**Status: OK 2026‑05‑14** – Laser‑Modul (5 V, 650 nm, 5 mW, 2‑pin
Halloween‑Style) über BC546C an GPIO 7 sauber schaltbar.

**Aufbau (Lochraster):**

| Pin / Anschluss | Verbindung |
|---|---|
| Laser‑Draht rot (+) | +5 V Buck‑Out |
| Laser‑Draht blau (−) | BC546C Collector |
| BC546C Base | 820 Ω → GPIO 7 |
| BC546C Emitter | GND |
| Vorwiderstand Laser | keiner – im Modul integriert |

**Tests bestanden:**

- Boot mit GPIO 7 = LOW → Laser sicher aus.
- K3 toggle schaltet zuverlässig (Display + Serial `[K3] Laser ON/OFF`).
- Sound‑Peak bei gleichzeitig leuchtendem Laser: **kein Flackern** →
  Buck‑5 V‑Rail trägt beide Lasten parallel ohne Brown‑Out.
- Trigger‑Pfad bleibt unverändert nutzbar.

**PCB‑Layout‑Konsequenz (aktualisiert 2026‑05‑14):**

Identisch zu 2.4 – der MOSFET‑Switch sitzt **im Stab‑PCB** (Doc 13
§ 4.1.1). Station gibt nur das **3,3 V‑Logiksignal `LASER_TX`** auf die
USB‑C‑Stab‑Buchse aus.

| Maßnahme | Wo | Status |
|---|---|---|
| BSS138 (SOT‑23) Low‑Side‑Switch | **Stab‑PCB** | dort aufs Layout |
| Gate‑Pulldown 10 kΩ + Series 820 Ω | Stab‑PCB | dort |
| Laser‑Vorwiderstand entfaellt (Modul integriert) | Stab‑PCB | dort |
| 100 Ω Series‑R am USB‑C‑Pin | Station‑PCB | ESD‑Schutz |

**Bedienung Sketch nach Diagnose‑Phase:** K3 ist aktuell Laser‑Toggle.
Sobald 2.4 (IR) erledigt ist, wird der Trigger‑Pfad final umgebaut auf
„Schuss = Laser + IR + Sound", K3 bleibt als manueller Laser‑Toggle für
das „Zielen ohne Schuss".

---

### 2.6 NeoPixel SK6812RGBW (GPIO 6)

**Test:** Eine SK6812RGBW direkt am GPIO 6 mit FastLED, einmal mit
3,3‑V‑Datenleitung direkt vom ESP, einmal über 74AHCT125 als
Level‑Shifter auf 5 V. Vergleichen: flackert die LED bei langer
Daisy‑Chain (Status‑LED + 2 × Stab‑LED)?

**Konsequenz für PCB:** Wenn ohne Level‑Shifter stabil → einfaches
Layout, ein Daisy‑Chain‑Pad reicht. Wenn flackert → 74AHCT125‑Footprint
auf das PCB. Außerdem: 470 Ω Reihen­widerstand in der Datenleitung
direkt nach dem Shifter, 1000 µF Pufferkondensator zwischen LED‑+5 V
und GND nahe den LEDs.

**Status: OK 2026‑05‑14** – Level‑Shifter zwingend nötig.

**Test‑Aufbau:** 8er SK6812RGBW‑Stripe direkt am GPIO 6 ohne
Level‑Shifter, 5 V vom Buck, GND gemeinsam. Boot‑Test (alle LEDs durch
R/G/B/W) + kontinuierliche Rainbow‑Animation.

**Befund:**

| Versorgung | Verhalten |
|---|---|
| **USB‑VBUS am DevKitC** | LEDs zeigen gar nichts. Vermutlich Kombination aus USB‑VBUS‑Drop (~4,5 V Stripe‑VDD) und ESP‑Datenflanke an der Erkennungsschwelle. |
| **12 V → Buck → 5 V** | Erste 1–3 LEDs leuchten rot, danach flackern die folgenden LEDs wild in zufaelligen Farben. Lehrbuch‑Signatur fuer **degradierte Daten‑Flanke** durch Cascading: jede Stage puffert das Signal an der Schwelle, nach wenigen Stages liest die naechste LED Schrott. |

→ SK6812 erwartet Daten‑High = 0,7 × VDD = **3,5 V** bei 5 V‑Versorgung.
Der ESP32‑S3 liefert nur 3,3 V (0,2 V drunter). Pin‑zu‑LED‑Direkt reicht
nicht.

**PCB‑Layout‑Konsequenz – 74AHCT125 mit Bypass‑Option:**

| Bauteil / Maßnahme | Status |
|---|---|
| **74AHCT125** (SOIC‑14) als Level‑Shifter 3,3 V → 5 V | aufs Layout, Pflicht |
| Versorgung des AHCT125: 5 V (gleiche Rail wie SK6812) | aufs Layout |
| Nur **ein Gate** aktiv nutzen; OE der anderen drei Gates auf VCC ziehen (deaktiviert) | aufs Layout |
| 100 nF Bypass‑C am VCC des AHCT125 nahe dem Pin | aufs Layout |
| 470 Ω Reihenwiderstand in der Datenleitung nach dem Shifter | aufs Layout |
| 1000 µF Low‑ESR Elko zwischen SK6812‑+5 V und GND nahe der ersten LED | aufs Layout (Lastpuffer fuer LED‑Spitzenstroeme) |
| **Bypass‑Möglichkeit** für den Fall „neue Charge geht doch ohne Shifter" | aufs Layout, siehe unten |

**Bypass‑Lösung:** Direkt vor der SK6812‑Daten‑Trace einen
**3‑Pad‑Lötjumper** plazieren:

```
   ESP GPIO6 ──┐
               ●─ Pad A
               │
               ●─ Pad B  ◄── SK6812 DIN (Daten zum Stripe)
               │
   AHCT125-OUT ●─ Pad C
```

- Pad A ↔ B löten = **Bypass aktiv** (3,3 V direkt zum Stripe)
- Pad B ↔ C löten = **Level‑Shifter aktiv** (Default für Halloween 2026)

Pads in 1206‑Geometrie, ein 0 Ω SMD‑Widerstand passt in eine der beiden
Positionen. So bleibt die Option offen, falls in einer späteren Charge
SK6812 mit niedrigerer Daten‑Schwelle verbaut sind — dann kann ein 0 Ω
umlegen die Schaltung vereinfachen.

**Default bei der Erstbestueckung:** Level‑Shifter aktiv (Pad B↔C).

---

### 2.7 ESP‑NOW Sender + Empfänger

**Test:** Zwei S3‑DevKitCs (das aktuelle Station‑Board + zweites Board
oder vorhandenes Target‑Modul). Test‑Sketch sendet alle 500 ms ein
Paket mit `station_id`, `sound_id`, Counter. Empfänger loggt RSSI und
verlorene Pakete.

Auch im **negativen** Fall messen: Pigtail‑Antenne lose / vertauscht
ohne Antenne → was passiert? Aus dem Vorgarten gegen die Hauswand mit
20 m Distanz – RSSI > −80 dBm und < 1 % Paketverlust = OK.

**Konsequenz für PCB:** Wenn die Pigtail‑Antennenführung nicht reicht,
muss der PCB‑Slot für die Antennen­fixierung anders sitzen, evtl. doch
SMA‑Buchse außen statt interne Whip.

**Status: VERSCHOBEN auf Prototyp‑Bring‑up (2026‑05‑14).** Risiko‑
Abwägung: ESP‑NOW läuft auf der Standard‑WiFi‑MAC‑Schicht des S3, das
WROOM‑1U‑Modul ist bereits HF‑getestet ausgeliefert. Die einzige
PCB‑Layout‑Frage ist die Antennen‑Positionierung – und dort haben wir
flexibel: das Pigtail ist verschieb­bar, die Whip‑Antenne wird mit
Heißkleber/Halterung an der Service‑Rückwand fixiert. Layout selbst
ist dadurch unkritisch.

**Nachzuholen am bestueckten Prototyp:**
- Sender/Empfaenger‑Sketch‑Paar zwischen Station‑PCB und entweder
  zweitem S3 oder zukuenftigem Target‑Refactor laufen lassen.
- RSSI bei Vorgartendistanz (~10 m) > −80 dBm, Paketverlust < 1 %.
- Falls Reichweite nicht reicht → Antennen‑Position aendern (Pigtail
  ermoeglicht das) oder doch externe SMA‑Buchse nachruesten.

---

### 2.8 TPA3110 Mute‑Strategie (Plan A oder B)

**Status:** **OK 2026‑05‑14** – Plan A (XSMT am PCM5102A, GPIO 12) ist
auf dem Lochraster live verifiziert. Kein Power‑On‑Pop, kein Hiss
zwischen Sounds, sauberer Fade‑In am Sound‑Start. STBY‑Pin am
XH‑A232‑Modul wird **nicht** herausgeführt (Modul‑Eigenschaft, Doc 12
Abschnitt 2.3 + 11‑offene‑punkte Punkt 28), Plan B entfällt damit.

**Konsequenz für PCB:** GPIO 11 (war für STBY reserviert) bleibt frei
oder bekommt einen optionalen Footprint, falls Plan B später doch noch
mit anderem Amp realisiert werden soll.

---

### 2.9 Kombinationstest

**Test:** Alle Subsysteme aus 2.3 – 2.7 gleichzeitig aktiv:

- Audio spielt einen 4‑Sek.‑WAV
- NeoPixel‑Status‑LED fadet währenddessen RGB‑Effekt
- ESP‑NOW sendet alle 500 ms ein Paket an ein zweites Board
- OLED rendert weiterhin den Status (Vol, Plays, „Playing…")
- Trigger wird einmal kurz gegen GND gezogen → ein Test‑Sound startet
- Direkt danach Laser an, IR‑Burst senden

**Erwartet:** Sound ohne Aussetzer, NeoPixel ohne Flackern, ESP‑NOW
ohne Paketverlust > 1 %, OLED rendert weiter, keine Boot‑Loops oder
Watchdogs.

**Konsequenz für PCB:** Wenn hier Aussetzer auftreten, ist es entweder
Software (Task‑Priorisierung, Core‑Affinität) oder Strom (Spitzen‑Last).
Beide Fälle ändern das Layout nicht, müssen aber **vor** PCB nochmal
adressiert werden, weil sie das Bauteil‑Sizing beeinflussen (Buck‑Strom,
Pufferkondensator‑Wert).

**Status: VERSCHOBEN auf Prototyp‑Bring‑up (2026‑05‑14).** Risiko‑
Abwägung: alle Subsysteme einzeln validiert (Audio, Trigger, IR, Laser,
OLED, NeoPixel‑Pfad ueber Level‑Shifter, Buck unter Audio‑Last).
Buck‑Strom (~500 mA bei Vollast inkl. NeoPixels) liegt deutlich unter
den 3 A des MP2307. Pufferkondensator‑Werte aus Doc 12 Abschnitt 5.6
sind grosszuegig dimensioniert.

**Nachzuholen am bestueckten Prototyp:**
- Alle Subsysteme parallel hochfahren: Sound spielt + IR sendet +
  NeoPixel‑Animation laeuft + Laser an + OLED rendert + ESP‑NOW
  empfaengt.
- Beobachten: keine Audio‑Aussetzer, kein NeoPixel‑Flackern, OLED ohne
  Tearing, kein Watchdog/Brown‑Out.
- 12 V‑Stromaufnahme messen, mit Buck‑Spec abgleichen.
- Buck‑Spule und ‑Chip Temperatur nach 10 min Dauerlast: handwarm.

---

### 2.10 USB‑C Stab‑Connector 6 Adern

**Test:** Siehe 11‑offene‑punkte.md Punkt 24 → bestelltes Anker‑240 W‑Kabel +
2 × USB‑C‑Breakout‑Board. Multimeter‑Durchgangstest Pin‑für‑Pin. Ergebnis
fließt zurück in Doc 12 Abschnitt 8.11.

**Konsequenz für PCB:** Pinbelegung der USB‑C‑Buchse auf der Station
wird **erst nach** diesem Test final. Bis dahin Footprint mit
Vollbeschaltung (alle 24 Pin) vorhalten, damit beide Optionen
(6‑Adern‑Plan A bzw. 4‑Adern‑Fallback Plan B) bestückbar bleiben.

**Status: VERSCHOBEN auf Prototyp‑Bring‑up (2026‑05‑14).** Risiko‑
Abwägung: USB‑C‑Buchsen‑Footprint ist Standard, beide Pin‑Layouts
(Plan A + Plan B) sind in Doc 12 Abschnitt 8.11 dokumentiert. Solange
**alle 24 Pin** der Buchse auf‑PCB durchgeroutet sind (Default bei
USB‑C‑Connectors fuer Vollbeschaltung), kann je nach Kabel‑Testergebnis
die Adern‑Zuordnung erst nachtraeglich per kleinen Patch‑Draehten
auf dem Bring‑up‑Board festgelegt werden.

**Nachzuholen vor finaler Stab‑Verbindung:**
- Anker‑240W‑Kabel + 2 × USB‑C Breakout‑Board (siehe 11‑offene‑punkte
  Punkt 24) bestellen und Durchgangstest Pin‑fuer‑Pin.
- SBU1/SBU2 ja/nein klaeren → entscheidet zwischen 6‑Adern‑Plan A
  (IR + Laser separat) und 4‑Adern‑Plan B (kombinierter Schalt‑Draht).

---

## 3. Vor PCB sinnvoll

### 3.1 OLED über JST‑XH 8‑pol statt Breadboard

Aktuell hängt das OLED‑Modul mit Jumper‑Drähten direkt. Auf dem PCB
sitzt es an einem Steckverbinder mit 8‑adrigem Kabel (~20–30 cm). Test:
gleiches OLED über ein gecrimptes JST‑XH‑Kabel betreiben, schauen ob
Bus weiterhin sauber bei 100 kHz läuft. Falls Probleme → ggf. doch auf
400 kHz nicht mehr aufdrehen oder ein I²C‑Bus‑Repeater.

### 3.2 I²C‑Pullups auf dem OLED‑Modul prüfen

Multimeter zwischen SDA↔3V3 und SCL↔3V3 (Stromlos!) auf Durchgang
prüfen. Wenn das DST‑015‑0 keine onboard Pullups hat, müssen 4,7 kΩ
zwischen SDA/SCL und 3,3 V aufs Layout (Doc 12 Abschnitt 7,
Beschaltungs‑Hinweis I²C).

### 3.3 SH1106‑Variantenoffset dokumentieren

**Status:** **OK 2026‑05‑14** – auf dem DST‑015‑0‑Modul ist
`u8x8.x_offset = 0` korrekt (statt der U8g2‑NONAME‑Default 2). In
Doc 12 Abschnitt 2.5 als Modul‑Hinweis hinterlegt.

### 3.4 Stromaufnahme bei Sound‑Peak messen

Damit der Buck‑Down nicht knapp dimensioniert ist. Erwartet bei Sound
mit voller Lautstärke: ESP 100–200 mA, OLED 20 mA, NeoPixels 50 mA pro
LED weiß, TPA 12 W ≈ 1 A am 12‑V‑Eingang. Buck‑5 V‑Last bleibt damit
unter 500 mA, ein LM2596 (3 A) wäre ausreichend.

### 3.5 Bypass‑/Stützkondensatoren testweise einlöten

Direkt aus Doc 12 Abschnitt 5.6: 100 nF + 10 µF an DAC‑VIN, 100 nF +
470 µF an TPA‑VIN. Wenn das Rauschen schon auf dem Lochraster
zurückgeht, ist klar dass diese Bauteile auf dem Layout zwingend
einzuplanen sind.

---

## 4. Später (kein Layout‑Impact)

Diese Themen können parallel laufen und brauchen nur „Pins stehen fest" –
das ist der Fall:

- ESP‑NOW‑Paket‑Format ausformulieren (Doc 12 Abschnitt 11)
- Sound‑ID‑Mapping endgültig festlegen (Punkt 1a in 11‑offene‑punkte)
- OLED‑Menü‑Struktur und Bildschirm‑Skizzen
- OTA‑Update‑Pfad testen
- Vollständige Sound‑Bibliothek anlegen + Audacity‑Batch‑Skript
- A/B‑Hörtest FR 7 vs. FR 10
- Gehäuse‑Druck (einteilig, Boden‑Klappe, Pfostenhalter)
- Antennen‑Fixierung mit Heißkleber‑Auflage in der Rückwand

---

## 5. Abnahme‑Definition

Das PCB darf bestellt werden, wenn:

- Alle Punkte aus Kapitel 2 entweder **OK** mit Datum oder **VERSCHOBEN
  auf Prototyp‑Bring‑up** mit dokumentiertem Risiko stehen.
- Die Konsequenzen für das Layout aus Kapitel 2 in Doc 12 Abschnitt 7
  (GPIO‑Plan) bzw. Abschnitt 2 (Audio‑Kette) eingepflegt sind.
- Die offenen Punkte aus Kapitel 3 mindestens als Footprint im Layout
  reserviert sind (auch wenn der Bauteil‑Wert noch nicht final ist).

### Stand 2026‑05‑14

| Punkt | Status | Detail |
|---|---|---|
| 2.1 Rauschen | OK | verkabelungs‑dominiert, Standard‑Bypass reicht |
| 2.2 Buck unter Last | OK | 5 V steht, 12 V Drop liegt an Test‑Verkabelung |
| 2.3 Trigger | OK | Debouncing 20 ms reicht; ESD/Filter aufs PCB |
| 2.4 IR + 38 kHz | OK | TSAL6200 + BC546C; PCB‑MOSFET = AO3400 |
| 2.5 Laser | OK | Halloween‑Modul direkt, kein Vorwiderstand extra |
| 2.6 NeoPixel | OK mit Konsequenz | **74AHCT125 Pflicht** + Bypass‑Lötjumper |
| 2.7 ESP‑NOW | verschoben | Antennen‑Pigtail erlaubt nachträgliche Korrektur |
| 2.8 Mute‑Strategie | OK | XSMT auf GPIO 12, STBY entfällt |
| 2.9 Kombinationstest | verschoben | nach Bestueckung am echten Board |
| 2.10 USB‑C 6 Adern | verschoben | USB‑C Vollbeschaltung im Footprint |

### Layout‑Hinweise zur Bestellung

Aus den OK‑Punkten ergeben sich folgende verbindliche Layout‑Vorgaben,
die im KiCad/EasyEDA‑Entwurf umgesetzt werden müssen:

| Bereich | Vorgabe |
|---|---|
| **Buck‑Topologie** | Ein MP2307‑Modul (oder kompatibel), 12 V in, 5 V out. **Kein** separater LDO/Pi‑Filter fuer DAC noetig. |
| **Audio Bypass** | 100 nF + 10 µF nahe DAC‑VIN; 100 nF + 470 µF nahe TPA‑VIN (Doc 12 Abschnitt 5.6). |
| **Stern‑GND** am Buck‑Output‑Minus | Audio‑GND, NeoPixel‑GND, Laser‑GND treffen sich dort. |
| **DC‑Hohlbuchse** | direkt aufs PCB, kein Pigtail; 12 V‑Trace ≥ 1 mm bei 1 oz Cu. |
| **MOSFETs IR + Laser** | 2 × AO3400 (SOT‑23) oder IRLML2502; Gate‑Pulldown 10 kΩ + Series 100 Ω; IR‑Vorwiderstand 33 Ω 0,5 W. |
| **NeoPixel‑Pfad** | 74AHCT125 SOIC‑14, 100 nF Bypass, 470 Ω Series in Datenleitung; **3‑Pad‑Lötjumper** (Bypass A↔B oder Shifter B↔C); 1000 µF Low‑ESR Elko an Stripe‑+5 V. |
| **I²C OLED** | 4,7 kΩ Pullups extern auf SDA/SCL **als DNP** vorsehen — DST‑015‑0 hat onboard Pullups (am Lochraster 2026‑05‑14 verifiziert: Display lief ohne externe Pullups stabil). Footprints bleiben für späteren Modul‑Wechsel als Reserve. 8‑pol JST‑XH zur Service‑Klappe. |
| **GPIO 4 Trigger** | 10 kΩ Pullup + 100 nF gegen GND nahe USB‑C‑Stab‑Buchse als ESD/Filter. |
| **USB‑C Stab‑Buchse** | Vollbeschaltung (24 Pin), Pinbelegung Plan A (6 Adern) als Default, Plan B (4 Adern) als Patch‑Möglichkeit. |
| **Antenne** | IPEX‑Pigtail‑Steckplatz am S3‑Modul‑Footprint, Antennen‑Position am Boden‑Klappen‑Rand vorbereiten. |

Diese Liste ist die Grundlage für den Schaltplan‑Entwurf. Sobald der
Entwurf steht, wird er gegen diese Tabelle geprüft, dann freigegeben.
