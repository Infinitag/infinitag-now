# 15 – Bestellliste Station V2 (erstes PCB)

> **Status: lebende Liste**, basiert auf den Bauteil‑Verifikationen aus
> Doc 14 (PCB‑Readiness) und dem Tobias‑Inventar‑Check 2026‑05‑14.
>
> Bei Änderungen am Schaltplan oder neuen Verifikationspunkten direkt
> hier nachpflegen.

---

## 1. Bereits im Bestand vorhanden – nicht nachbestellen

| Bauteil | Verwendung | Anzahl/Station |
|---|---|---|
| 0805 100 nF Keramik X7R 25 V | Bypass‑Cs (DAC, TPA, AHCT, MOSFET, alle ICs) | ~10 |
| 0805 10 µF Keramik X7R | DAC‑Bypass, Output‑Filter | ~4 |
| 220 µF 16 V Elko TH | Buck‑IN (C2) | 1 |
| 470 µF 16 V Elko TH (Standard 8×12 mm, Aukenien‑Kit) | Buck‑OUT (C4) | 1 |
| 1000 µF 16 V Elko TH (Standard 10×16 mm, Aukenien‑Kit) | optional als Reserve falls Low‑ESR knapp wird | 1 |
| SS34 SMA Schottky 40 V / 3 A | D1 Verpolungsschutz (12 V) | 1 |
| 1N4148 SOD‑123 | ESD‑Schutz Trigger‑Eingang | 1 |
| 0805 33 Ω 0,5 W | IR‑LED Vorwiderstand (R_IR) — **gehört aufs Stab‑PCB** (Doc 13) | – |
| 0805 4× 10 kΩ Widerstandsarray | I²C‑Pullups | 1 |
| 0805 4× 470 Ω Widerstandsarray | Series‑R AHCT125 → SK6812 | 1 |
| 0805 1 kΩ | (Reserve, falls Pullup‑Wert anders) | – |
| ~~BSS138 SOT‑23~~ | ~~IR + Laser MOSFET‑Switch~~ — **gehört aufs Stab‑PCB**, nicht Station (Entscheidung 2026‑05‑14, siehe Doc 13 § 4.1.1) | 0 |
| SK6812 5050 | Außenwand‑Status‑LED | 1 |
| 1,3" OLED 128×64 SH1106 (Modul, DST‑015‑0) | Service‑Display + 4 Tasten | 1 |
| Pushbutton 6×6 mm | (für externe Test‑Trigger) | bei Bedarf |
| USB‑C Buchse 6P (Bestand) | ~~nicht für Plan A geeignet (nur Power+CC, keine Daten‑Pins)~~ – bleibt für Charging‑Anwendungen | – |
| TSAL6200 IR‑LED | nicht aufs PCB – fürs nachgelagerte Wand‑Refactor (TSAL6100) | – |
| TSOP36238TR | nicht aufs PCB – fürs Target | – |
| Pin‑Header 1×4 Stiftleiste 2,54 mm | für MP2307‑Modul | 1 |

---

## 2. LCSC – Komplettbestellung (Empfehlung für SMD + Connectoren)

> Sammelbestellung 2026‑05‑14 zusammengestellt nach dem fertigen Schaltplan.
> Alle Teile in einer LCSC‑Bestellung bestellen.

### 2.1 SMD‑ICs (Original‑Hersteller garantiert)

| Bauteil | LCSC‑Nr | Anzahl | ca. Preis |
|---|---|---|---|
| **SN74AHCT125D** SOIC‑14 (TI Original) | **C7869** | 10 | ~0,40 €/Stk |
| **USBLC6‑4SC6Y** SOT‑23‑6 (ST Original) | **C7415** | 5 | ~0,25 €/Stk |

### 2.2 Schutz + Polyfuse

| Bauteil | LCSC‑Nr | Anzahl | ca. Preis |
|---|---|---|---|
| **Polyfuse 1812 SMD 1,5 A** | **C18198333** (LUTE 1812L150/33GR, 33 V) oder C70967 (RUILON SMD1812P150TF, 24 V) | 5 | ~0,06 €/Stk |

### 2.3 0805 Widerstände (nicht im Bestand)

| Bauteil | LCSC‑Nr | Anzahl | ca. Preis |
|---|---|---|---|
| **0805 100 Ω 1 %** | **C17475** | 100 | ~1 €/Beutel |
| **0805 4,7 kΩ 1 %** (DNP für I²C‑Pullups) | **C17714** | 100 | ~1 €/Beutel |
| **0805 0 Ω** (Bypass‑Jumper) | **C17477** (UNI‑ROYAL 0805W8F0000T5E) | 100 | ~1 €/Beutel |

### 2.4 Kondensatoren (optional Low‑ESR statt Bestand)

**Empfehlung 2026‑05‑14:** Standard‑Werte aus dem Aukenien‑Kit sind für unsere
Anwendung absolut ausreichend. Low‑ESR ist nur „nice to have" — überspring
diesen Punkt wenn Budget knapp oder Bestellzeit kurz.

**Such‑Strategie bei LCSC** (statt fester C‑Nummern):

| Bauteil | LCSC‑Suchbegriff | Anmerkung |
|---|---|---|
| 470 µF 16 V Low‑ESR TH | `Nichicon UPW 470uF 16V` oder `Panasonic FR 470uF 16V` | Tatsächlich Low‑ESR; UHW Serie ist nur „High Reliability" — auch OK aber nicht Low‑ESR |
| 1000 µF 16 V Low‑ESR TH | `Panasonic FR 1000uF 16V` oder `Rubycon YXG 1000uF 16V` | C10 TPA‑VIN Lastpuffer, hier bringt Low‑ESR wirklich was |

**Worauf achten:**
- **Body Diameter (BD)** muss zum EasyEDA‑Footprint passen — Standard 470 µF/16 V = D8 × 12 mm, 1000 µF/16 V = D10 × 16 mm
- **Spannungsfestigkeit**: 16 V minimum, 25 V/35 V geht auch (gibt Reserve)
- **Specs‑Tag „Low ESR"** oder „Low Impedance" in der Beschreibung suchen
- ESR‑Wert (falls angegeben): < 0,1 Ω ist Low‑ESR

### 2.5 Connectors

| Bauteil | LCSC‑Nr | Anzahl | ca. Preis |
|---|---|---|---|
| **DC‑022B Hohlbuchse 5,5/2,5 mm THT** | **C40128** (Klassiker) oder C295468 / C2837419 | 5 | ~0,30 €/Stk |
| **USB‑C 12P TYPE‑C‑31‑M‑12 SMT+THT** | **C165948** | 10 | ~0,20 €/Stk |
| **JST‑XH 8‑pol** Gehaeuse + Kabel‑Buchse Pärchen | **C158015** / `JST XH 8P` | 5 Sets | ~0,50 €/Set |
| **JST‑XH 4‑pol** (für Station‑LED H1) | `JST XH 4P` | 5 Sets | ~0,30 €/Set |

### 2.6 Pin‑Header & Jumper

| Bauteil | LCSC‑Nr | Anzahl | ca. Preis |
|---|---|---|---|
| **Pin‑Header Male 1×40 2,54 mm** (zum Kürzen für H1/H2/H3/Buck/PCM/TPA) | **C124379** | 10 | ~0,10 €/Stk |
| **Pin‑Header Female 1×40 2,54 mm** (DevKitC‑Sockel zum Kürzen) | **C124382** | 10 | ~0,20 €/Stk |
| **Jumper Cap 2,54 mm** (für H1 Bypass) | **C42037** | 50 | ~0,05 €/Stk |

### 2.7 LCSC‑Bestellzusammenfassung

| Position | Anzahl | Summe |
|---|---|---|
| 2 SMD‑ICs | 15 | ~5 € |
| Polyfuse | 5 | ~1 € |
| 0805 Widerstände | 3 × 100 | ~3 € |
| Low‑ESR Elkos (optional) | 8 | ~6 € |
| DC‑Buchse + USB‑C | 15 | ~3 € |
| JST‑XH Connectoren | 10 sets | ~4 € |
| Pin‑Header + Jumper | 50 | ~5 € |
| **Bauteile gesamt** | | **~25–30 €** |
| **LCSC Standard Direct Line Versand** | | **~10–15 € nach DE** |
| **TOTAL LCSC** | | **~40–45 €** |

**LCSC‑Hinweise:**

- **Mindestbestellwert keine festen Schwelle**, aber unter 30 € ist Versand teuer
- **Bestellzeit** typisch 1–2 Wochen nach DE
- **Direkter Pfad:** Suche bei LCSC mit den `C‑Nummern` (z. B. „C7869") für genaue Treffer
- **JLCPCB‑Konto = LCSC‑Konto** — wenn du das PCB bestellt hast, kannst du dich mit demselben Login einloggen
- **Versand mit JLCPCB‑PCB‑Order bündeln war billiger** — aber du hast schon bestellt, also separater Versand

### 2.8 Was bei LCSC NICHT bestellen

| Position | Wohin stattdessen |
|---|---|
| Module (ESP DevKitC, MP2307, GY‑PCM5102A, TPA3110) | AliExpress – besseres Sortiment + günstiger |
| Antenne (IPEX + Pigtail + Whip) | AliExpress |
| WAGO 2601‑1102 | Reichelt (deutschsprachig, schneller) |
| Visaton FR 10/4 Speaker | Reichelt / Bürklin |
| Anker USB‑C 240W Kabel | Amazon / Anker direkt |

---

## 2.9 Alte/dedizierte LCSC‑Bestellliste (Detail‑Übersicht)

Spalte „Suche" sind Stichwoerter fuer die LCSC‑Bauteilsuche. Vor Auftrag
LCSC‑Stock und Footprint final pruefen.

| Bauteil | LCSC‑Suche | Anzahl | Verwendung |
|---|---|---|---|
| **SN74AHCT125D** SOIC‑14 | „SN74AHCT125D" – C7869 oder C9015 | 10 | Level‑Shifter 3,3 V → 5 V fuer SK6812 |
| 100 Ω 0805 | „100R 0805 1%" – `C17475` | 30 | Series‑R im ESD‑Filter pro Signal‑Ader (4× pro USB‑C); ggf. zusätzlich Series‑Gate fuer MOSFETs im Stab |
| **USBLC6‑4SC6 SOT‑23‑6** | „USBLC6 4-channel TVS" – `C7415` | 5 | 4‑Kanal TVS‑Diode‑Array, ESD‑Schutz für USB‑C‑Signaladern (Trigger, NeoPixel, IR, Laser) |
| 4,7 kΩ 0805 | „4.7K 0805 1%" | 20 | I²C‑Pullups extern (falls OLED‑Modul keine onboard hat) |
| ~~470 µF 16 V THT~~ | nicht nötig – Bestand reicht (Standard 8×12 mm, 15 Stk im Aukenien‑Kit) | – | C4 am Buck‑Output |
| **1000 µF 16 V Low‑ESR** (Panasonic FR oder Rubycon YXG) | „1000uF 16V Low ESR Panasonic FR" – `C92841` / `C42018` | 4 | **Lastpuffer direkt am TPA‑VIN** – hier zählt Low‑ESR wirklich (Audio‑Peak‑Dynamik) |
| Polyfuse THT 1,5 A | „1812 polyfuse 1.5A" oder „RXEF150" | 5 | PF1 Sicherung 12 V‑Eingang |
| **DC‑022B 5,5/2,5 mm Through‑Hole** | „DC-022 5.5×2.5" | 5 | DC‑Hohlbuchse |
| **USB‑C Buchse 12P SMT+THT Hybrid** | „TYPE‑C‑31‑M‑12" – `C165948` | 10 | Stab‑Connector (Plan A 6 Adern). Eigener 6P‑Bestand reicht nicht (keine D+/D−/SBU). 10 Stück = 3–4 Stations‑Buchsen + 3–4 Stab‑Buchsen + Reserve |
| JST‑XH 8‑pol Stecker + Buchse Pärchen | „JST XH 8P" | 5 sets | Service‑Klappe ↔ Hauptboard |
| **Pin‑Header 1×4 Stiftleiste 2,54 mm** | „Header 1×4 Male 2.54" – `C124379` | 10 | Station‑LED‑Connector J5 (5V, DIN, DOUT, GND) – mit Standard‑Jumper als Bypass |
| **Jumper‑Cap 2,54 mm** (Standard‑Mainboard‑Jumper) | „Jumper Connector 2.54" / „2P Jumper Cap" – `C42037` | 20 | überbrückt DIN/DOUT von J5 wenn keine Station‑LEDs angeschlossen; auch generell für Konfig‑Optionen am PCB |
| **0 Ω 0805** Widerstand | „0R 0805" – `C17168` | 20 | optionaler Pad‑Connector / Bypass an anderen Stellen, Reserve |
| **WAGO 2601‑1102** Leiterplattenklemme 2‑pol, Push‑In, 3,5 mm Pitch | Reichelt Art‑Nr. `WAGO 2601-1102` | 8 | Speaker‑Anschluss J3 (1 pro Station + Reserve). **Reichelt 0,95 €/Stück**, werkzeuglos mit Hebel — ideal für Halloween‑Aufbau |
| Pin‑Header 1×22 Female 2,54 mm | „Header 1×22 Female 2.54" | 4 | DevKitC‑Steckverbinder (2 pro Board + Reserve) |
| Pin‑Header 1×12 Female 2,54 mm | „Header 1×12 Female 2.54" | 5 | Reserve‑Header `J_RSV` |
| Pin‑Header 1×7 Female 2,54 mm | „Header 1×7 Female 2.54" | 5 | PCM5102A‑Steckverbinder |
| Pin‑Header 1×10 Female 2,54 mm | „Header 1×10 Female 2.54" | 5 | TPA3110‑XH‑A232‑Steckverbinder |
| Pin‑Header 1×3 (Status‑LED extern) | „Header 1×3" | 5 | externer SK6812 |
| Pin‑Header 1×6 TP‑Reihe (optional) | „Header 1×6" | 5 | Test‑Pads TP1..TP6 als Stiftleiste |

**Sparpotenzial:** 220 µF aus Bestand kann an manchen Stellen 470 µF
ersetzen (z. B. Buck‑OUT). Dann nur 2× 470 µF statt 4× bestellen.

---

## 3. AliExpress – Sammel‑Bestellung fuer PCB‑Bestueckung

> **Status: noch zu bestellen**, vor dem 02.06.2026 (= geplanter PCB‑Liefertermin),
> damit beim Bring‑up alles griffbereit ist.

### 3.1 Module (steckbar aufs PCB)

| Bauteil | Suchbegriff bei AliExpress | Anzahl | ca. Preis |
|---|---|---|---|
| **ESP32‑S3‑DevKitC‑1U N16R8** | `ESP32-S3-DevKitC N16R8 IPEX` (DiYmore Store) | 3 | ~10 € / Stück |
| **MP2307 Mini Buck‑Down 5V 3A** (mit Lötpads, 4R7 Spule) | `MP2307 mini buck 5V 4R7` | 5 | ~1 € / Stück |
| **GY‑PCM5102A** I²S DAC Modul | `GY-PCM5102A` oder `PCM5102 I2S DAC` | 3 | ~3 € / Stück |
| **TPA3110 XH‑A232** Mono‑Verstaerker (V928) | `TPA3110 XH-A232 mono` | 3 | ~3 € / Stück |
| **IPEX‑zu‑SMA‑Female Pigtail 10cm** + 2,4 GHz Whip‑Antenne | `IPEX SMA pigtail + 2.4GHz antenna` | 3 Sets | ~3 € / Set |

### 3.2 Connectors

| Bauteil | Suchbegriff bei AliExpress | Anzahl | ca. Preis |
|---|---|---|---|
| **USB‑C 12P Buchse THT/SMT Hybrid** (TYPE‑C‑31‑M‑12) | `USB-C 12P SMT female TYPE-C-31-M-12` | 10 | ~0,30 € / Stück |
| **DC‑Hohlbuchse 5,5/2,5 mm THT** | `DC-022 5.5x2.5 PCB jack` | 5 | ~0,30 € / Stück |
| **JST‑XH 8‑pol Stecker + Buchse Sets** | `JST XH 2.54 8P male female kit` | 5 Sets | ~0,50 € / Set |
| **JST‑XH Sortiment** (2P/3P/4P/5P/6P/8P) | `JST XH connector kit 2.54` | 1 Sortiment | ~10 € |

### 3.3 Pin‑Header & Jumper

| Bauteil | Suchbegriff bei AliExpress | Anzahl | ca. Preis |
|---|---|---|---|
| **Pin‑Header Buchsenleiste 1×40 Female 2,54 mm** (zum Kürzen auf 1×22, 1×15, 1×10, 1×7) | `pin header female 2.54 1x40` | 10 Stück | ~0,30 € / Stück |
| **Pin‑Header Stiftleiste 1×40 Male 2.54 mm** (zum Kürzen auf 1×12, 1×8, 1×4 etc.) | `pin header male 2.54 1x40` | 10 Stück | ~0,20 € / Stück |
| **Jumper Caps 2.54 mm** Sortiment (gefärbt) | `jumper cap 2.54 100pcs` | 1 Beutel = 100 Stück | ~3 € |

### 3.3a SMD‑Bauteile (alternativ zu LCSC, wenn alles aus einer Quelle)

| Bauteil | Suchbegriff bei AliExpress | Anzahl | ca. Preis |
|---|---|---|---|
| **SN74AHCT125D** SOIC‑14 | `SN74AHCT125 SOIC-14` (Texas Instruments Original) | 10 | ~0,50 € / Stück |
| **USBLC6‑4SC6Y** SOT‑23‑6 | `USBLC6-4SC6` (ST Microelectronics) | 5 | ~0,30 € / Stück |
| **Polyfuse SMD 1812 1,5 A** | `SMD1812 polyfuse 1.5A 24V` | 5 | ~0,30 € / Stück |
| **0805 Widerstands‑Kit** (alle Werte) | `0805 resistor kit 170 values 50pcs` | 1 Kit | ~10 € |
| _oder einzeln:_ 0805 100 Ω | `0805 100R 1% 100pcs` | 50 | ~1 € |
| _einzeln:_ 0805 4,7 kΩ | `0805 4.7K 1% 100pcs` | 50 | ~1 € |
| _einzeln:_ 0805 0 Ω (Jumper) | `0805 0R 100pcs` | 50 | ~1 € |

**Wichtige Hinweise zur Klonproblematik bei ICs:**

- Bei SN74AHCT125D auf **„TI" oder „Texas Instruments"** im Listing achten
- Bei USBLC6 auf **„ST Microelectronics"** achten
- Store mit > 1000 Bewertungen, > 95 % positiv waehlen
- Bei Sicherheitsbedarf: LCSC fuer ICs ist 1–2 € teurer, aber garantiert Original

### 3.4 Audio + Halloween‑Zusatz

| Bauteil | Lieferant | Anzahl | Anmerkung |
|---|---|---|---|
| **Visaton FR 7/4 (Art. 2086)** 4 Ω 5 W | **Reichelt** oder Bürklin oder Visaton‑Shop | 1 + 1 Reserve | nicht AliExpress – Original. **Entscheidung 2026‑05‑14: FR 7 statt FR 10**, weil Sound/Gehäusegröße‑Faktor besser ist (3D‑gedruckte Vergleichsboxen). VOL_MAX im Sketch automatisch auf 0,65 durch SpeakerProfile FR7. |
| **DC‑Hohlbuchsen‑Stecker 5,5/2,5 mm zum Konfektionieren** | Reichelt / Bürklin / AliExpress | 3 | für 0,75 mm² Zuleitung zum Aufbau |

### 3.5 Werkzeug‑Doppelcheck (falls noch nicht vorhanden)

| Werkzeug | Anmerkung |
|---|---|
| Lötkolben mit feiner Spitze (≤ 1 mm) | für SMD‑0805 + SOIC‑14 + SOT‑23 |
| Lötzinn 0,5 mm bleifrei | RoHS‑konform, passt zu LeadFree HASL |
| Flussmittel‑Stift / Flüssig | sauberere SMD‑Lötung |
| SMD‑Pinzette (ESD‑sicher) | für 0805‑Bauteile pickup |
| Lupenleuchte oder USB‑Mikroskop | optisch Lötstellen prüfen |
| Entlöt‑Litze | falls was schiefgeht |

→ Vermutlich alles schon im Maker‑Setup. Nur als Sanity‑Check.

### 3.6 USB‑C 6‑Adern‑Test (fuer Doc 14 § 2.10)

| Bauteil | Suchbegriff | Anzahl |
|---|---|---|
| **Anker USB‑C 240W Kabel** (eMarker, 6+ Adern) | Anker offiziell oder Amazon | 1 |
| **USB‑C Breakout‑Board** (14×20 mm, 8 Pin‑Header) | `USB-C breakout board 8pin` | 2 |

→ Damit kannst du Pin‑für‑Pin durchpiepen welche Adern dein Kabel hat.

### 3.7 Geschätzte AliExpress‑Gesamtkosten

| Position | Summe |
|---|---|
| Module (Buck, DAC, TPA, ESP‑Boards, Antennen) | ~60 € |
| Connectors + Pin‑Header + Jumper | ~25 € |
| Werkzeug‑Nachkauf (falls nötig) | 0–30 € |
| USB‑C‑Test‑Kit | ~25 € |
| **Gesamt AliExpress** | **~110–140 €** |
| Plus Reichelt (Visaton + WAGO + ggf. LCSC‑Lücken) | ~30 € |

→ Insgesamt **ca. 140–170 €** für die komplette Bauteilversorgung einer Station.

---

## 3.1 Bauteile fürs Stab‑PCB (Doc 13 Wand‑V3)

Wenn das Stab‑PCB im selben JLCPCB‑Lauf bestellt wird, sind das die
direkt vom Station‑Refactor abhängigen Bauteile (durch Architektur‑
Entscheidung 2026‑05‑14, MOSFETs in den Stab verschoben). Vollständige
Stab‑BOM steht in Doc 13 § 6 — hier nur die Diff zum „MOSFETs in Station"‑Plan.

Bedarf: 3–4 Stäbe + Reserve = **8 Stück pro Bauteil**.

| Bauteil | Verwendung | Anzahl |
|---|---|---|
| BSS138 SOT‑23 | 2× pro Stab als IR + Laser Low‑Side‑Switch | 8 (4 Stäbe) |
| 0805 820 Ω | Series‑R am MOSFET‑Gate | 8 |
| 0805 10 kΩ | Gate‑Pulldown am MOSFET | 8 |
| 0805 33 Ω 0,5 W | IR‑LED Vorwiderstand (TSAL6100) | 4 |
| TSAL6100 IR‑LED | (mit OM3‑Linse, siehe Doc 13 § 3.2) | 4 |

## 4. Bestell‑Tracking

| Bestellung | Datum | Order‑Nr. | Betrag | Lieferung | Status |
|---|---|---|---|---|---|
| **JLCPCB** (PCB ×5) | 2026‑05‑14 | – | ~ 6 USD | 27.05.–02.06.2026 | ✅ bestellt |
| **LCSC** (SMD + Connectoren) | 2026‑05‑17 | **WM2605170016** | **$18.35 USD** ($7,28 Ware + $8,30 Versand + $3,00 Handling − $0,23 Rabatt) | ~1–2 Wochen DE | ✅ bestellt (Zahlung PayPal, 3 Tage Frist) |
| **AliExpress** (Module + DC‑Buchsen) | 2026‑05‑17 | – | **28,01 €** kostenfreier DHL/Hermes‑Versand | **17.05.–29.05.2026** | ✅ bestellt |
| **Farnell** (VLHW5100 Justage‑LED ×100) | 2026‑05‑23 | – (Bestätigung steht aus) | **20,90 €** + Express 14,99 € | erwartet 27.05.2026 (UK‑Lager) | ⏳ bestellt, Bestätigung ausstehend |
| **Reichelt** (Visaton FR 7/4 + ggf. WAGO) | offen | – | – | – | ⏳ noch zu bestellen |
| **USB‑C‑Kabel** (TB4/USB4 vollverdrahtet, Doc 14 § 2.10) | offen | – | – | – | ⏳ Recherche läuft (§ 6.2) |

**AliExpress‑Cart‑Inhalt 2026‑05‑17** (5 Positionen, 28,01 €):

| Position | Anzahl | Stk‑Preis |
|---|---|---|
| GY‑PCM5102A | 2 | 3,25 € |
| ESP32‑S3 N16R8 mit IPEX‑Socket + Antenne | 2 | 6,69 € |
| DC‑Buchse Variante A | 1 | 1,79 € |
| DC‑Buchse Variante B | 1 | 1,65 € |
| TPA3110 XH‑A232 (Stereo 2×30 W) | 1 | 4,69 € |

**Aus Bestand verfügbar** (deshalb nicht in AliExpress‑Cart):
ESP32‑S3 ×1, MP2307 Mini Buck 5 V (mehrere), 0805 100 Ω, Pin‑Header,
Jumper, WAGO‑Klemme, OLED‑Modul, SK6812, Aukenien‑Elko‑Kit.

**Geplanter Stations‑Ausbau:** 3 Stationen + Reserve
(ESPs total: 2 bestellt + 1 Bestand = 3 ✓).

## 5. JLCPCB‑Bestellung (Detail)

**Status: bestellt am 2026‑05‑14**, Lieferung ~27.05.–02.06.2026 erwartet.

| Position | Bestellung |
|---|---|
| **Stueckzahl** | 5 |
| **Layer** | 2‑Layer |
| **Kupfer** | 1 oz |
| **Maße** | **100 × 76 mm** (final aus Layout) |
| **Thickness** | 1,6 mm |
| **Farbe** | Grün |
| **Silkscreen** | Weiß |
| **Material** | FR4 TG135 |
| **Surface‑Finish** | LeadFree HASL |
| **Via Covering** | Tented |
| **Min Via Drill** | 0,3 mm |
| **Min Trace** | 0,25 mm (DRC‑Default) |
| **Outline Tolerance** | ±0,2 mm Regular |
| **Mark on PCB** | 2D Barcode (kostenlos, JLCPCB‑Standard) |
| **Stencil** | nicht bestellt (Hand‑Bestueckung) |
| **Assembly** | nein – wir bestuecken selbst |
| **Versand** | Global Standard Direct Line (9–13 Werktage) |
| **Kosten gesamt** | ~ 6 USD (inkl. PCB + Versand + Zoll + Payment‑Fee) |

---

## 5. „Do Not Place" (DNP) – Footprint vorhanden, Bestueckung optional

Diese Bauteile bekommen einen Footprint und sind im Schaltplan, werden
aber bei der ersten Bestueckung **nicht** verloetet. So bleibt die Option
offen, ohne dass das PCB‑Layout sich aendert.

| Ref | Bauteil | Begruendung DNP |
|---|---|---|
| PF1 | Polyfuse | falls erstes Board ohne Sicherung getestet werden soll |
| Bypass‑Pads NeoPixel | 0 Ω | nur bestueken wenn Level‑Shifter umgangen werden soll |
| TP1..TP6 | Test‑Pads | unbestueckt = Loetpunkte; mit Stiftleiste = Steckverbinder |

---

## 6. Verbleibende Verifikationspunkte vor finaler Bestellung

Diese Punkte aus Doc 14 sind „verschoben auf Prototyp‑Bring‑up", brauchen
aber Bauteile aus dieser Liste:

- 2.7 ESP‑NOW: benoetigt zweites ESP32‑S3‑DevKitC + Antennen‑Pigtail.
  → in Pos. 3 oben enthalten.
- 2.10 USB‑C 6 Adern: benoetigt Anker‑240 W‑Kabel + USB‑C Breakout‑Board
  fuer Durchgangstest. → siehe 11‑offene‑punkte Punkt 24.

### 6.1 Bestell‑Verifikationen (vor Versand pruefen)

| # | Punkt | Status 2026‑05‑16 |
|---|---|---|
| V1 | **Aukenien‑Bestand „0805 100"** ist 100 Ω | ✅ **Erledigt** — bestätigt 100 Ω; trotzdem zur Sicherheit bei LCSC nachbestellt |
| V2 | **TPA3110 XH‑A232 „60 W Stereo"** vs. Mono‑Variante | ✅ **OK** — bewusst die Stereo‑Variante (2× 30 W), identisch zum Test‑Setup. Schaltplan nutzt nur einen Kanal, der zweite bleibt unbestückt |
| V3 | **IPEX‑Antenne** beim ESP32‑S3‑DevKitC‑1U inkludiert | ✅ **Inkludiert** — DiYmore‑Listing zeigt explizit „ESP‑S3 N16R8 with IPEX socket + antenna". Pigtail+Whip aus Pos. 3.1 entfällt |
| V4 | **MP2307 Mini Buck 5 V Modul** | ✅ **Bestand** — schon im Lager, nicht nachbestellen |
| V5 | **Modul‑Footprints** stimmen mit Lieferung ueberein | ⏳ **Verschoben auf Bring‑up** — Doc 17 § 6 |
| V6 | **USB‑C‑Kabel mit allen 6 Adern (Plan A)** für Doc 14 § 2.10 | ❌ **Offen** — Anker‑Kabel im Bestand hatten zu wenig Pins. Neue Recherche nötig, siehe § 6.2 |

→ V1–V4 abgeschlossen, V5 beim Bring‑up, V6 als eigene Recherche (siehe § 6.2).

### 6.2 USB‑C‑Kabel für 6‑Adern‑Plan‑A (offen)

**Hintergrund Plan A** (Doc 12, Doc 13): Wir nutzen sechs Adern eines
„vollverdrahteten" USB‑C‑Kabels als Stab‑Verbindung:

| Ader | Belegung |
|---|---|
| VBUS | +5 V vom Buck |
| GND | Masse |
| D+ | NeoPixel‑Daten 3,3 V → 5 V (über 74AHCT125) |
| D− | Trigger‑Input |
| SBU1 | IR‑Modulation 38 kHz |
| SBU2 | Laser EIN/AUS |

→ Daraus folgt: **Es muss ein Kabel mit SBU1/SBU2 sein**, sonst fehlen
uns IR + Laser. Reine USB 2.0 240W „Power"‑Kabel haben **kein** SBU.

**Kabel‑Klassen nach Belegung:**

| Klasse | VBUS+GND | D+/D− | CC1/CC2 | SBU1/SBU2 | TX/RX‑Paare | Plan A tauglich? |
|---|---|---|---|---|---|---|
| USB 2.0 Charging (z. B. viele „Anker 240W") | ✅ | ✅ | ✅ | ❌ | ❌ | **Nein** |
| USB 3.x (USB 3.2 Gen 1/2) | ✅ | ✅ | ✅ | ✅ | ✅ | **Ja** |
| USB4 / Thunderbolt 3/4 (40 Gbps) | ✅ | ✅ | ✅ | ✅ | ✅✅ | **Ja** (Overkill) |

**Empfehlung beim Einkauf:**

- Sicherheits‑Wahl: **Thunderbolt 4 oder USB4 Kabel** — per Standard
  vollverdrahtet (alle 24 Pins). Anker, Cable Matters, Belkin haben
  passende ~1 m / ~2 m Varianten ab 20 €
- Günstiger: **USB 3.2 Kabel** (10 Gbps) — vollverdrahtet, aber nicht
  jeder Hersteller hält sich dran. Vor dem Kauf SBU‑Pinout im Listing
  oder Bewertungen suchen
- Beim **Anker** explizit nach Thunderbolt‑Modellen schauen (z. B. „Anker
  765 USB4" oder „Anker Thunderbolt 4"), **nicht** 240W‑Power‑only
- Länge **1 m oder kürzer** — wir wollen den Stab tragbar halten;
  Verluste in dünnen SBU‑Adern bleiben klein

**Aktion offen:** Konkretes Modell pruefen (Pinout im Datenblatt / Review)
und beschaffen, bevor der USB‑C 6‑Adern‑Test (Doc 14 § 2.10) gemacht
werden kann. Bis dahin bleibt Doc 14 § 2.10 „verschoben".

**Test‑Kandidat 2026‑05‑16:** „FDBOY 240W E‑Marker Chip USB4 Datenkabel
40 Gbps 8K 60 Hz" (AliExpress, ~4,99 € / 1 m). Indizien fuer volle Verdrahtung:

- Beigelegtes mehrsprachiges PDF‑Datenblatt mit korrekten Specs
  (USB4 + TB4 Dual Protocol, 40 Gbps, 8K@60 Hz DP Alt Mode, PCIe Tunneling,
  PD 3.1 EPR 240 W, USB 3.2 Gen 2x2 Backwards)
- Specs sind physikalisch nur mit voller 24‑Pin‑Verdrahtung inkl. SBU1/SBU2
  realisierbar
- 1000+ Verkaeufe, 71 Bewertungen (4,9 ★), Marke+ zertifizierter Shop
  (94,2 % positiv)
- Limit 20 Stueck/Kunde — bestelle 1 zum Verifizieren

**Verifikations‑Protokoll nach Lieferung:**

1. USB‑C‑Breakout‑Board (Doc 15 § 3.6) an beide Enden
2. Multimeter‑Durchgang auf alle 24 Pins, dokumentieren in Doc 14 § 2.10
3. Pflicht: VBUS, GND, D+, D−, CC1, CC2, **SBU1, SBU2** durchverdrahtet
4. Bonus: TX1±, RX1±, TX2±, RX2± durchverdrahtet (bestaetigt TB4‑Spec)
5. Wenn OK: in BOM Doc 15 § 3.4 als Standard‑Stab‑Kabel aufnehmen
6. Wenn nicht: Eintrag in Doc 11 (offene Punkte), Kandidat 2 suchen

---

## Aenderungshistorie

- **2026‑05‑14**: Datei angelegt nach Inventar‑Check und PCB‑Freigabe in
  Doc 14. Erste Version mit allen Bauteilen aus den 2.1–2.6 Tests.
- **2026‑05‑16**: Bestell‑Verifikationspunkte V1–V6 in § 6.1 ergaenzt.
  V1–V4 abgehakt (100 Ω OK; TPA3110 Stereo 2×30 W ist Absicht; IPEX‑Antenne
  ist in der DiYmore‑Lieferung enthalten; MP2307 schon im Bestand).
  V6 neu: USB‑C‑Kabel mit vollen 6 Adern (SBU‑Pins) wird gesondert
  recherchiert (§ 6.2). FR 7/4 wird noch bei Reichelt bestellt.
