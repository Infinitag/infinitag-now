# 17 – Bring‑Up Plan: Prototyp‑PCB Nr. 1 (Station V2)

> **Status: vorbereitend.** Die PCBs sind am 2026‑05‑14 bei JLCPCB
> bestellt und werden Ende Mai / Anfang Juni 2026 erwartet. Dieses Doc
> ist der Ablaufplan fuer den Tag, an dem die Boards ankommen.

## 1. Lieferung und erste Sichtpruefung

- **Lieferung erwartet:** 27.05.–02.06.2026 (Global Standard Direct Line, 9–13 Werktage)
- **Stueckzahl:** 5 PCBs
- **Bestelldetails:** siehe Doc 15 § 4

**Bei Ankunft:**

1. Verpackung pruefen – Boards in der Anti‑Statik‑Folie?
2. Mit Schieblehre Maße verifizieren: **100 × 76 mm**
3. Eine PCB ans Licht halten und gegen Schaltplan abgleichen:
   - Sind alle Bauteil‑Footprints sichtbar?
   - Silkscreen lesbar?
   - Keine Kratzer oder Fertigungs‑Fehler im Kupfer?
4. Eine PCB als „Backup" zur Seite legen (unbestueckt). 4 weitere fuer
   die 3–4 Stations‑Aufbauten + 1 Reserve.

## 2. Bestueckungsreihenfolge

**Wir bestuecken NICHT alle 5 sofort**, sondern nur **eine als Bring‑up‑Board**.
Wenn das laeuft, kommen die weiteren dran.

### Schritt 1: Passive Komponenten (10–15 Min)

Reihenfolge: kleinste zuerst, damit groessere nicht im Weg sind.

1. Alle **0805 Widerstaende**: R1–R8 (100 Ω, 470 Ω, 820 Ω, etc.)
2. Alle **0805 Keramik‑Caps**: C1, C3, C5, C6, C7, C8, C9, C11
3. **Polyfuse F1** (1812 SMD)
4. **Schottky D1** (SMA SS34) – Polung beachten (Strich = Kathode)
5. **TVS D2** (SOT‑23‑6 USBLC6) – Pin 1 mit Marker

### Schritt 2: SMD ICs (5 Min)

- **74AHCT125 (U4)**: SOIC‑14 – Pin 1 mit Marker, vorsichtig lo10ten

### Schritt 3: Through‑Hole Komponenten (15–20 Min)

1. **Elkos C2, C4, C10** – Polung extrem wichtig (Plus an + Markierung)
2. **DC‑Hohlbuchse J1** – mechanisch fest verloeten
3. **WAGO‑Klemme CN1** (Speaker)
4. **JST‑XH 8‑pol CN2** (OLED)
5. **Pin‑Header für DevKitC** (2× 1×22 Female) und Module:
   - U2 PCM5102A (1×15)
   - U3 TPA3110 (1×9)
   - MP2307 Buck (1×4 Stiftleiste)
6. **Pin‑Header H1** (Station‑LED 4‑pol Stiftleiste, default mit Jumper Pin 2/3)
7. **Pin‑Header H2** (Reserve 12‑pol) – nur falls geplant zu nutzen
8. **Pin‑Header H3** (Test/JTAG 8‑pol) – nur falls geplant zu nutzen

### Schritt 4: USB‑C Buchse

Das ist das **knifflichste** Bauteil – 12 SMD‑Pads + 4 mechanische
Befestigungs‑Tabs. Letzte Position waehlen, damit es Zeit zum Pruefen gibt.

## 3. Erste Tests OHNE ESP / Module

**Bevor irgendetwas eingesteckt wird:**

1. **Multimeter‑Durchgangstest:**
   - +12V_RAW → +12V (durch F1 + D1) ✓
   - +5V (am MP2307‑Sockel Pin 3) → +5V (an versorgten Pins, z. B. U4 Pin 14)
   - +3V3 → +3V3 (am DevKitC‑Sockel Pin 1) zu OLED‑Connector Pin 2
   - **Kein Kurzschluss zwischen +12V und GND**
   - **Kein Kurzschluss zwischen +5V und GND**
   - **Kein Kurzschluss zwischen +3V3 und GND**
2. **Power‑Up Test ohne Module:** 12 V anlegen, nur Multimeter
   - +12V steht stabil
   - +5V am Buck‑Output steht (12 V → 5 V Konvertierung läuft)
3. **Strom messen** (mit Labornetzteil im Strom‑Limit‑Modus): 12 V Eingang
   sollte nur ein paar mA ziehen (LED am Buck‑Modul, falls vorhanden)

## 4. Module einstecken & ESP flashen

1. **DevKitC einstecken** (Pin‑Header‑Sockel)
2. **Buck‑Modul einstecken** (auf MP2307 Pin‑Header)
3. **DAC Pin‑Header einsetzen** (steckbar) — wenn nicht aktiv
4. **TPA Pin‑Header einsetzen** — wenn nicht aktiv
5. 12 V anlegen, ESP USB‑C anschließen, **Test‑Sketch flashen** (aus `Station/src/main.cpp`)
6. **Serial‑Monitor** öffnen – sieht das Boot‑Log normal aus?
7. **OLED‑Display** über JST‑XH dranklemmen – zeigt es „Bereit"?
8. **Speaker** an WAGO‑Klemme anschließen – K4 drücken, Sound spielt?

## 5. Verschobene Verifikationspunkte aus Doc 14 nachholen

Drei Punkte waren auf den Prototyp‑Bring‑up verschoben:

### 5.1 Doc 14 § 2.7 ESP‑NOW Reichweite

- Zweites S3‑DevKitC besorgen oder bestehendes Target‑Modul mit ESP‑NOW
- Test‑Sketch: alle 500 ms ein Paket senden, RSSI loggen
- Ziel: > −80 dBm bei 10 m Vorgarten‑Distanz, < 1 % Paketverlust
- Bei zu schwach: Antennen‑Pigtail‑Position aendern oder SMA‑Buchse nachruesten

### 5.2 Doc 14 § 2.9 Kombinationstest

Alle Subsysteme **gleichzeitig** laufen lassen:

- Audio spielt
- NeoPixel macht Rainbow‑Effekt
- ESP‑NOW empfaengt Pakete
- IR sendet (sobald Wand‑V3‑Prototyp da ist)
- Laser an
- OLED rendert
- Trigger reagiert

Erwartet: keine Aussetzer, kein Watchdog‑Reset, alle Funktionen
parallel stabil ueber > 5 Minuten.

### 5.3 Doc 14 § 2.10 USB‑C 6‑Adern Durchgangstest

Mit Anker‑240W‑Kabel + 2× USB‑C Breakout‑Boards Pin‑fuer‑Pin
durchpiepen. Welche Adern sind durchgeschaltet? Pin‑Mapping pruefen:

- VBUS, GND: müssen durch (Power)
- D+/D−, CC1/CC2, SBU1/SBU2: pruefen

→ Ergebnis fließt in die finale Stab‑Verkabelung (Doc 13).

## 6. Bekannte offene Themen vom Schaltplan

Aus den Schaltplan‑Diskussionen 2026‑05‑14 noch zu pruefen:

| Thema | Verifikation am Prototyp |
|---|---|
| PCM5102A‑Modul: 15‑Pin‑Footprint passt zum Aliexpress‑Modul | Schieblehre Pad‑Abstand vergleichen |
| TPA3110 XH‑A232: Pinout 1–9 stimmt | dito |
| MP2307‑Modul: Solder‑Pad‑Abstand passt | dito |
| WAGO 2601‑1102: 3,5 mm Pitch im Footprint | dito |
| USB‑C 12P TYPE‑C‑31‑M‑12: Footprint passt zur Lieferung | dito |
| OLED DST‑015‑0: 8‑pol JST‑XH passt | dito |

**Wenn ein Modul/Connector nicht passt:** Re‑Spin der naechsten PCB‑Charge
mit korrigiertem Footprint.

## 7. Wenn alles funktioniert — Halloween‑Halbjahres‑Plan

Nach erfolgreichem Bring‑up:

1. **Restliche 4 PCBs bestuecken** (3–4 Stationen + 1 Reserve)
2. **3D‑Druck Gehaeuse** starten (Doc 12 § 8)
3. **Wand‑V3 PCB** designen + bestellen (Doc 13)
4. **Sound‑Bibliothek** komplettieren (alle 14 Halloween‑Sounds)
5. **ESP‑NOW‑Protokoll** definieren
6. **OLED‑Menü** implementieren

Halloween 2026: 31. Oktober → **fünf Monate** zum Fertigstellen.

## Aenderungshistorie

- **2026‑05‑14**: Datei angelegt direkt nach PCB‑Bestellung. Detaillierter
  Bring‑up‑Plan strukturiert.
