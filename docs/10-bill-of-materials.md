# 10 – Stückliste / Bill of Materials

Aus den drei vorhandenen Schaltplänen extrahiert (Stand Mai 2026). Mengen
sind pro Einzelplatine angegeben.

## Wand V2.1 (Tagger / Stab)

| Pos | Bauteil | Wert / Typ | Menge | Bemerkung |
| --- | --- | --- | --- | --- |
| R1 | Widerstand | 10 kΩ | 1 | Pull‑down Trigger (Net `Trigger_ESP`) |
| R2 | Widerstand | 33 Ω | 1 | Vorwiderstand IR‑LED |
| R3, R4 | Widerstand | 1 kΩ | 2 | Basiswiderstände Q1/Q2 |
| Q1, Q2 | NPN | SS8050_C2150 | 2 | Treiber für IR‑LED + Laser |
| IR1 | IR‑LED | TSAL6200 | 1 | 940 nm IR‑LED |
| LASER | Laser‑Modul | MTF185‑102SY1 (2‑pol Steckverbindung) | 1 | KY‑008‑artig, 5 V |
| U1, U2 | RGBW LED | SK6812RGBW‑5050 | 2 | Daisy‑Chain |
| U3 | Mikrotaster | PB‑22E85L | 1 | 6‑pol, taktiles Feedback |
| Connector | JST PH | S7B‑PH‑K‑S‑(LF)(SN) | 1 | 7‑pol zur Station |

## Wand Station Dev Board V1

| Pos | Bauteil | Wert / Typ | Menge | Bemerkung |
| --- | --- | --- | --- | --- |
| ESP_S3_LEFT/RIGHT | Pinheader | 22850120ANG1SYA01 (20‑Pin) | 2 | Slot für Sound‑ESP (ESP32‑S3 DevKit) |
| ESP_S_LEFT/RIGHT | Pinheader | 22850120ANG1SYA01 (20‑Pin) | 2 | Slot für Logic‑ESP |
| MAX98357 | I²S DAC + Class‑D | MTF185‑107SY1 (7‑Pin Steck) | 1 | Mono‑Audio‑Verstärker |
| GAIN | Pinheader | PZ254V‑11‑03P (3‑Pin) | 1 | Gain‑Konfigurations‑Header |
| R1 | Widerstand | 100 kΩ | 1 | Gain‑Pulldown |
| PWR | JST XH | S2B‑XH‑A‑(LF)(SN) (2‑Pin) | 1 | externe 5 V‑Eingang |
| CN1 | JST XH | S7B‑XH‑A‑(LF)(SN) (7‑Pin) | 1 | Connector zur Wand |
| Lautsprecher | Visaton **VS‑FR7/4** | 7 cm, 4 Ω, 5 W RMS, 130 Hz–20 kHz | 1 | Mono‑Breitbänder, direkt an `+`/`−` des MAX98357 |
| Sound‑ESP‑Modul | ESP32‑S3‑DevKit mit eingebautem MicroSD‑Slot | konkretes Modell noch zu ergänzen | 1 | wird auf `ESP_S3_LEFT/RIGHT` gesteckt |
| Logic‑ESP‑Modul | klassisches ESP32‑DevKit (z. B. NodeMCU‑32S / ESP32 DevKit V1) | konkretes Modell noch zu ergänzen | 1 | wird auf `ESP_S_LEFT/RIGHT` gesteckt |

> Hinweis: Die ESPs selbst sind keine SMD‑Bauteile auf der Platine, sondern
> werden als komplette DevKit‑Module auf die Pinheader gesteckt. Das macht
> Reparaturen / Tausch sehr einfach. Die genauen Modul‑Bezeichnungen sind
> noch nachzureichen – siehe [`11-offene-punkte.md`](11-offene-punkte.md).

## Wand Station V2 (Refactor – ESP32‑S3 + ESP‑NOW)

> Stand 2026‑05‑07: Konzept festgeschrieben, Bauteile für **Lochraster‑Phase**
> bestellbar. Detaildoku in
> [`12-refactor-station-v2.md`](12-refactor-station-v2.md).

### Kern‑Elektronik

| Pos | Bauteil | Wert / Typ | Menge pro Station | Bemerkung |
| --- | --- | --- | --- | --- |
| MCU | ESP32‑S3‑DevKitC‑1U N16R8 | 16 MB Flash, 8 MB Octal PSRAM, IPEX/U.FL | 1 | diymore‑Klon o. ä., Standard‑Espressif‑Layout |
| Antenne | 2,4 GHz SMA‑Stub‑Antenne, 3 dBi | Pigtail im DevKit‑Lieferumfang | 1 | externe SMA‑Buchse in Gehäusewand |
| DAC | PCM5102A‑Modul („GY‑PCM5102") | I²S 24‑Bit | 1 | ~3 € |
| Verstärker | TPA3116D2‑Modul, Mono‑Bridge oder 1× Stereo‑Kanal | 12 V Eingang, 50 W max. | 1 | ~6 € |
| Lautsprecher | Visaton FR 10/4 | 10 cm, 4 Ω, 30 W, f₀ ~85 Hz | 1 | ~16 €. Upgrade‑Pfad: BG 17 oder W3‑1364SA |
| Step‑Down | LM2596 oder MP2307 Modul, 12 V → 5 V, 3 A | einstellbar | 1 | ~2 € |
| 12‑V‑Eingang | DC‑Hohlbuchse 5,5/2,5 mm | Einbau Gehäusewand | 1 | ~1 € |
| Steckernetzteil | 12 V / 2 A, möglichst IP‑Klasse | Outdoor‑tauglich | 1 | ~10 € |

### Konfiguration & Anzeige

| Pos | Bauteil | Wert / Typ | Menge | Bemerkung |
| --- | --- | --- | --- | --- |
| OLED | 1,3" SH1106 OLED + 4‑Tasten‑Modul | 128 × 64, I²C, K1–K4 | 1 | ~5–7 €. Hinter Service‑Klappe |
| Status‑LED | SK6812RGBW, einzeln | 5 V, daisy‑chained mit Stab‑LEDs | 1 | < 1 €, auf Außenwand |
| Service‑Stecker | JST‑XH 8‑pol, S8B‑XH‑A | für OLED‑Modul abnehmbar | 1 | – |

### Wand‑/Stab‑Anschluss

| Pos | Bauteil | Wert / Typ | Menge | Bemerkung |
| --- | --- | --- | --- | --- |
| Stab‑Connector | GX16‑7 (klein) | 7‑polig, weiblich, Einbau | 1 | beibehalten von V1 |
| MOSFET IR‑LED | N‑Kanal, Logic‑Level (z. B. AO3400) | SOT‑23 oder TO‑92 | 1 | für 3,3 V → 5 V Last |
| MOSFET Laser | wie oben | | 1 | |
| Level‑Shifter NeoPixel | 74AHCT125 (oder „Sacrificial‑LED") | 3,3 V → 5 V Datenleitung | 1 | bei Bedarf, im Lochraster verifizieren |
| I²C‑Pullups | 4,7 kΩ | für SDA + SCL gegen 3,3 V | 2 | nur falls OLED‑Modul nicht eigene Pullups hat |

### Mechanik / Gehäuse

| Pos | Bauteil | Bemerkung |
| --- | --- | --- |
| 3D‑Druck Gehäuse | Speaker‑Kammer + Elektronik‑Kammer akustisch getrennt | PETG oder PLA+ |
| Service‑Klappe | 3D‑gedruckt, M3‑Gewindeeinsätze oder Magnetverschluss | – |
| Schaumdichtung 1 mm | selbstklebend, am Klappenrand | gegen Spritzwasser |
| Polyesterwatte / Schafwolle | für Speaker‑Kammer‑Dämpfung | ~50–60 % Volumen füllen |
| M3‑Gewindeeinsätze | Heat‑set, für Service‑Klappe | 2–4 Stück |
| Klarsicht‑Filament‑Inlay | für Status‑LED‑Diffusor | PETG transparent oder Heißkleber‑Tropfen |

### Lochraster‑Phase (zusätzlich, wird später durch eigenes PCB ersetzt)

| Pos | Bauteil | Bemerkung |
| --- | --- | --- |
| GPIO‑Extension‑Board | für ESP32‑S3‑DevKitC‑1U, mit 12 V Eingang + 5 V/3,3 V Ausgang | diymore „ESP32‑S3 GPIO Extension Board" o. ä. |
| Lochraster | 70 × 90 mm, doppelseitig | für Audio‑Stage und Verbinder‑Sammelpunkt |
| Stiftleisten + Pinheader 2,54 mm | für steckbare Module (DAC, Amp, Buck‑Down) | – |
| Schraubklemmen 2,54 mm | für Versorgung + Stab‑Anschluss | – |

### Audio‑Files (Vorbereitung)

| Quelle | Pfad heute | Zielformat |
| --- | --- | --- |
| Treffer‑Sounds (14× MP3) | `/Volumes/basteln/Halloween/Lasertag/StationSounds/sound-effects/` | 22 kHz Mono 16‑Bit WAV nach Audacity‑Bearbeitung (Bass‑EQ, Loudness) |
| Spell‑Sounds (5× MP3) | `/Volumes/basteln/Halloween/Lasertag/Spell/` | gleich |
| Speicherort | – | LittleFS‑Partition im 16 MB Flash des S3 |

### Geschätzte Kosten Lochraster‑Phase pro Station

| Komponente | € |
|---|---|
| ESP32‑S3‑DevKitC‑1U N16R8 | ~10 |
| GPIO‑Extension‑Board | ~6 |
| PCM5102A | 3 |
| TPA3110 XH‑A232 (statt TPA3116) | 6 |
| Buck‑Down 12 V → 5 V | 2 |
| OLED + 4‑Tasten‑Modul | 6 |
| Visaton FR 10/4 | 16 |
| 12 V Steckernetzteil | 10 |
| Stecker, Kabel, Kleinteile | 5 |
| **Gesamt** | **~64 €** |

(Bei 4 Stationen: ~250 € Materialkosten, ohne 3D‑Druck‑Filament.)

### Konkreter Bestellstand 2026‑05‑08 – Lochraster‑Phase

| Komponente | Konkretes Produkt | Lieferant | Preis | Status |
|---|---|---|---|---|
| ESP32‑S3‑DevKitC‑1U N16R8 + GPIO‑Extension‑Board | „diymore ESP32 Combo mit Antennenschnittstelle" | Amazon | 16,99 € | ✅ bestellt |
| PCM5102A DAC | „Aihasd GY‑PCM5102 I2S DAC" | Amazon | 8,93 € | ✅ bestellt |
| Visaton FR 10/4 | „Best Price Square Speaker FR 10 4 OHMS by VISATON" | Amazon | 17,98 € | ✅ bestellt |
| TPA3110 XH‑A232 Stereo (2 Stück) | „VGOL TPA3110 XH‑A232 2 STÜCKE" – 2 × 30 W (Peak), Stereo BTL, 4–8 Ω, 8–26 V, mit Kühlkörper | Amazon | 12,19 € | ✅ bestellt |
| Buck‑Down 12 V → 5 V (Mengenpack) | MP1584/MP2307‑basierter Mini‑Buck, fest 5 V, 3 A, 96 % Eff., <30 mV Welligkeit | – | – | ✅ Eigenbestand vorhanden |
| 12 V Steckernetzteil + DC‑Hohlbuchse | Standardteil 12 V/2 A, 5,5 × 2,5 mm | – | – | ✅ Eigenbestand vorhanden |
| OLED 1,3" SH1106 + 4‑Tasten‑Modul | 8‑pol (GND/VCC/SCL/SDA/K1–K4) | – | – | ✅ Eigenbestand vorhanden |
| NeoPixel SK6812RGBW | LED‑Streifen‑Reststück | – | – | ✅ Eigenbestand vorhanden |
| Lochraster, Stiftleisten, Schaltlitze | – | – | – | ✅ Eigenbestand vorhanden |
| **Summe Bestellungen** | | | **~56 €** | |

**Pro Station gerechnet** (ESP+Extension + DAC + Speaker + Verstärker = 49,99 €) – die Verstärker liefert 2 Stück, also ~44 € pro Station bei der nächsten Bestellung.

### Ausstehend / klärt sich später

| Komponente | Anlass / Bemerkung |
|---|---|
| 2,4 GHz SMA‑Antenne mit Pigtail | prüfen ob im diymore‑Combo dabei (sollte sein); falls nicht: ~5 € separat |
| MOSFETs (AO3400 SMD oder IRLZ44N TO‑220) für IR‑LED + Laser | erst nötig, wenn Stab‑Schnittstelle in Phase 2 angeschlossen wird |
| 74AHCT125 Level‑Shifter | nur falls NeoPixel‑Pegel im Lochraster‑Test instabil ist |
| NJM2761 Audio‑Mute‑IC | **Plan C** für Audio‑Mute, nur falls Variante B + ggf. Variante A nicht reichen. ~3 €. Wird zwischen DAC‑Output und Verstärker‑Input geschaltet. |
| JST‑XH 8‑pol Stecker für späteres OLED‑Service‑Kabel | für ersten Lochraster‑Test optional, direkte Verdrahtung reicht |
| Mechanik (Heat‑Set‑Inserts, Schaumdichtung, Polyesterwatte) | erst für Phase 2 nötig (Gehäusebau nach Funktions‑Proof) |

**Konsequenz Verstärker‑Wahl:** TPA3110 XH‑A232 statt TPA3116D2. Begründung:
30 W RMS Speaker (FR 10/4) braucht keine 50 W RMS Verstärker‑Reserve;
TPA3110 mit 2 × 15 W RMS bei 12 V/4 Ω ist exakt im Sweet‑Spot, kompakter,
gleicher Klang‑Charakter (TI‑Class‑D mit gleicher Filter‑Architektur).
Refactor‑Doku [`12-refactor-station-v2.md`](12-refactor-station-v2.md)
ist entsprechend aktualisiert.

---

## Infinitag Target V3.2

### Hauptkomponenten

| Pos | Bauteil | Wert / Typ | Menge | Bemerkung |
| --- | --- | --- | --- | --- |
| U1 | ESP32‑S3‑WROOM‑1 (N8R8) | 8 MB Flash, 8 MB PSRAM | 1 | Hauptcontroller |
| U2 | LDO | AMS1117‑3.3 | 1 | 5 V → 3,3 V |
| U3 | USB‑UART | CP2102N‑A02‑GQFN28R | 1 | Programmierbrücke |
| U16 | Optokoppler | BPC‑817S/A | 1 | potentialfreier Schalter |
| U4–U15 | RGBW LED | SK6812RGBW‑5050 | 12 | LED‑Ring |
| LED2 | IR‑Receiver | TSOP4138 | 1 | 38 kHz |
| USB2 | USB‑Buchse | 105164‑0001 | 1 | Programmier‑/Stromanschluss |

### Diskrete Bauteile

| Pos | Bauteil | Wert | Menge |
| --- | --- | --- | --- |
| R1 | Widerstand | 0 Ω | 1 |
| R3, R4, R12, R13 | Widerstand | 0 Ω | 4 |
| R5, R6 | Widerstand | 10 kΩ | 2 |
| R8, R17, R18 | Widerstand | 1 kΩ | 3 |
| R9 | Widerstand | 47 kΩ | 1 |
| R10 | Widerstand | 22 kΩ | 1 |
| R11 | Widerstand | 10 kΩ | 1 |
| R7, R15, R19, R20 | Widerstand | 5,1 kΩ | 4 |
| R14 | Widerstand | (im Schaltplan ohne Wert) | 1 | – |
| R21 | Widerstand | 180 Ω | 1 | Optokoppler‑Vorwiderstand |
| C1, C3, C7, C9, C10, C13–C24 | Kondensator | 100 nF | 18 |
| C2, C4, C5, C8, C11 | Kondensator | 10 µF | 5 |
| C12 | Kondensator | 100 nF | 1 |
| C6 | Kondensator | 10 µF | 1 |
| Q1, Q2 | NPN | L8050QLT1G | 2 |
| Q4, Q5 | NPN | SS8050_C2150 | 2 |
| D1 | Diode | 1N5819HW‑7‑F (Schottky) | 1 |
| D2, D3, D4 | TVS / ESD | LESD5D5.0CT1G | 3 |
| LED1, L‑SW1, L‑5V, L‑3.3V | Status‑LED | KT‑0603R | 4 |
| RESET, BOOT | Mikrotaster | TS‑1187A‑B‑A‑B | 2 |
| SW1, SW‑5V, SW‑3.3V, LED, PWR | JST PH | S2B‑PH‑K‑(LF)(SN) (2‑Pin) | 5 |
| LED‑Connector | JST PH | S3B‑PH‑K(LF)(SN) (3‑Pin) | 1 |

## Optik: Linsen für IR‑LED‑Kollimation (Zauberstab)

Quelle: **AstroMedia / OptiMedia Acrylglas‑Linsen** (https://astromedia.de)
Alle Linsen haben eine umlaufende Kunststoff‑Fassung mit Steg, die direkt
in einen 3D‑gedruckten Halter eingepresst werden kann.

**Aktuell verbaut:** `304.OM3` – f = 30 mm, Ø 16,5 mm (**→ markiert mit ★**)

### Physikalischer Hintergrund: Linse + TSAL6200

Die TSAL6200 (5 mm Gehäuse) hat einen Abstrahlwinkel von **±17°** (halber Winkel,
auf 50 % Intensität). Damit der Strahl kollimiert wird, muss die LED im
**Brennpunkt** der Linse sitzen (Abstand = Brennweite f).

Lichtkegelradius an der Linse = f × tan(17°):

| Brennweite | Kegelradius an Linse | Benötigter Ø |
|---|---|---|
| 15 mm | 4,6 mm | 9,2 mm |
| 30 mm | 9,1 mm | 18,3 mm |
| 49 mm | 15,0 mm | 30,0 mm |
| 65 mm | 19,9 mm | 39,8 mm |

→ Die OM3 (f = 30 mm, innenliegend Ø 12,2 mm) schneidet bereits einen Teil
des Lichtkegels ab. Linsen mit **kürzerer Brennweite** oder **größerem Ø**
würden mehr Licht einfangen. Bei den kleinen Wand‑Gehäusen ist die
mechanische Tiefe (= Brennweite = LED‑Abstand zur Linse) aber begrenzt.

### Vollständige Linsenübersicht AstroMedia

| Art.‑Nr. | f (mm) | Vergr. | Dioptrien | Form | Ø (mm) | innen Ø (mm) | Rand (mm) | IR‑Eignung | Bemerkung |
|---|---|---|---|---|---|---|---|---|---|
| 302.OM1A | −65 | −4,8 | −15,4 | plankonkav | 16,2 | – | 2,2 | ✗ | Zerstreuungslinse – weitet Strahl auf |
| 301.OM1 | −35 | −8,1 | −28,6 | plankonkav | 16,5 | – | 3,5 | ✗ | Zerstreuungslinse |
| 303.OM2 | 15 | 17,7 | 66,7 | plankonvex | 16,5 | 7,6 | 3,6 | ○ | Sehr kurze Brennweite; LED muss nur 15 mm hinter Linse; Lichtkegelradius 4,6 mm passt gut zu innen Ø 7,6 mm; aber sehr positionsempfindlich |
| **304.OM3** ★ | **30** | 9,3 | 33,3 | plankonvex | 16,5 | 12,2 | 3,6 | **✓** | **Aktuell verbaut.** Guter Kompromiss; Lichtkegel Ø 18 mm > innen Ø 12,2 mm → etwas Licht‑Verlust; solide Reichweite |
| 316.OM11 | 42 | 7,0 | 23,8 | bikonvex | 25,0 | – | 1,2 | ✓+ | Größerer Ø → fängt mehr Licht ein; bikonvex = bessere optische Qualität; Brennweite 42 mm baut länger im Gehäuse |
| 317.OM12 | 45 | 6,6 | 22,2 | bikonvex | 25,0 | – | 1,2 | ✓+ | Ähnlich OM11; etwas längere Brennweite; bikonvex |
| 305.OM4 | 49 | 6,1 | 20,4 | plankonvex | 16,5 | – | 3,6 | ○ | f = 49 mm baut sehr lang; Lichtkegel Ø 30 mm weit größer als Linsen‑Ø → großer Lichtverlust |
| 306.OM5 | 65 | 4,8 | 15,4 | plankonvex | 16,5 | – | 3,7 | ✗ | f zu lang für kompaktes Gehäuse |
| 308.OM7 | 106 | 3,4 | 9,4 | plankonvex | 34,5 | – | 5,2 | ✗ | Für IR‑Kollimation zu lange Brennweite |
| 307.OM6 | 120 | 3,1 | 8,3 | plankonvex | 16,5 | – | 3,5 | ✗ | Zu lang |
| 309.OM7A | 170 | 2,5 | 5,9 | plankonvex | 35,0 | – | 1,2 | ✗ | Zu lang |
| 311.OM8A | 180 | 2,4 | 5,6 | plankonvex | 40,0 | – | 1,6 | ✗ | Zu lang |
| 315.OM6A | 185 | 2,4 | 5,4 | plankonvex | 16,5 | – | 3,6 | ✗ | Zu lang |
| 318.OM13 | 198 | 2,3 | 5,1 | bikonvex | 59,0 | – | 2,2 | ✗ | Zu lang, zu groß |
| 310.OM8 | 225 | 2,1 | 4,4 | plankonvex | 34,5 | – | 5,0 | ✗ | Zu lang |
| 313.OM9A | 245 | 2,0 | 4,1 | plankonvex | 50,0 | – | 1,5 | ✗ | Zu lang |
| 314.OM10 | 275 | 1,9 | 3,6 | plankonvex | 62,0 | – | 2,3 | ✗ | Zu lang |
| 312.OM9 | 360 | 1,7 | 2,8 | plankonvex | 45,0 | – | 5,4 | ✗ | Zu lang |

**Legende Eignung:** ✓+ = besser als aktuell · ✓ = geeignet (aktuell) · ○ = bedingt · ✗ = nicht geeignet

### Empfehlungen für den nächsten Bestellversuch

1. **316.OM11 oder 317.OM12** (f ≈ 42–45 mm, Ø 25 mm, bikonvex):
   Größerer Linsendurchmesser fängt mehr des IR‑Lichtkegels ein (+50 % Fläche
   ggü. OM3). Bikonvex reduziert sphärische Aberration. Nachteil: Gehäuse
   muss ≈ 12–15 mm tiefer gebaut werden (Brennweite länger als OM3).
   Bestellseite prüfen – innenliegender Ø nicht angegeben, evtl. nicht mit
   Steg wie OM3.

2. **303.OM2** (f = 15 mm, Ø 16,5 mm, innen 7,6 mm):
   Sehr kompakt (LED nur 15 mm hinter Linse), Lichtkegel passt gut.
   Aber: mechanisch sehr positionsempfindlich (±0,5 mm = große Strahlablage).
   Nur sinnvoll wenn der Linsenhalter‑Entwurf aus Punkt 24 sehr präzise wird.

3. **304.OM3 beibehalten** als Fallback, wenn Gehäuse‑Tiefe das Limit ist –
   dann aber mit dem überarbeiteten Linsenhalter aus Punkt 24, damit
   Toleranzen ausgeschlossen sind.

> Verlinkt mit offenem Punkt 24 (Linsenhalter‑Redesign) in
> [`11-offene-punkte.md`](11-offene-punkte.md).

### IR‑LED‑Vergleich (Wand V3, 2026‑05‑08)

Im Zuge des Wand‑V3‑Refactors wurde die IR‑LED neu bewertet. Die
TSAL6200 (±17°) ist Standard, aber für die OM3 nicht optimal, weil
ihr Lichtkegel größer als die Linsen‑Apertur ist und ~50 % der Energie
außen vorbeigeht. Auswahl‑Übersicht:

| LED | λ (nm) | Halbwerts‑Winkel | I_e (mW/sr) | Package | Eignung für OM3 |
|---|---|---|---|---|---|
| Vishay TSAL6200 (alt verbaut) | 940 | ±17° | 80 | 5 mm T‑1¾ | ○ Lichtkegel zu breit, 50 % Verlust |
| **Vishay TSAL6100 ★** | 940 | **±10°** | **170** | 5 mm T‑1¾ (drop‑in zu 6200) | **✓+ optimal: voll in OM3‑Apertur, 2× heller** |
| OSRAM SFH 4544 | 940 | ±10° | ~135 | 5 mm | ✓ gleichwertige Alternative |
| OSRAM SFH 4547 | 940 | ±10° | ~150 | 5 mm | ✓ gleichwertige Alternative |
| Vishay TSAL6400 | 940 | ±25° | ~70 | 5 mm | ✗ schlechter als 6200 |
| Vishay VSLY5940 | 940 | ±20° | ~90 | 5 mm | ✗ zu breit |
| OSRAM SFH 4715AS (Power) | 850 | ±45° (3W) | – | SMD | ✗ falsche λ + zu breit, für Kamera |

**Entschieden für Wand V3:** TSAL6100 als drop‑in‑Ersatz für TSAL6200.
Gleicher Footprint, gleiche U_F (Vorwiderstand R2 = 33 Ω bleibt), nur
±10° Beam und 2× höhere Achs‑Intensität. Bestellung über Reichelt in
Sammelposition. Details in
[`13-refactor-wand-v3.md`](13-refactor-wand-v3.md), Abschnitt 3.2.

### Justage‑LED (sichtbar) für Optik‑Block

Zum Einstellen des Optik‑Block‑Defokus (28 mm hinter Linse, § 3.3 in
Doc 13) muss man den Brennpunkt sehen können – TSAL6100 ist mit
940 nm fürs Auge unsichtbar. Lösung: eine weiße LED mit identischem
Gehäuse und Abstrahlwinkel, die wie ein mechanischer Zwilling der
TSAL6100 in den gleichen Press‑Fit‑Sitz passt.

| LED | Hersteller | Halbwerts‑Winkel | Gehäuse | Bemerkung |
|---|---|---|---|---|
| **Vishay VLHW5100 ★** | Vishay | **±10°** | 5 mm T‑1¾ klar, untinted | **Gleiche Fertigung wie TSAL6100 → konsistente Chip‑Z‑Position. Bestellt 2026‑05‑23 (Farnell 3777973, 100 Stück, 20,90 €).** |
| Everlight 5 mm 22500 mcd 20° | Everlight | ±10° (2θ½ = 20°) | 5 mm T‑1¾ klar | Reichelt 164227, Alternative falls VLHW5100 nicht lieferbar |
| Lumetheus 5 mm 25000 mcd 20° | Noname | „ca. 20°" Marketing | 5 mm wasserklar | Amazon Prime, billig, Chip‑Position unspezifiziert – nur als Notbehelf |

> **Verwendung:** NICHT für den produktiven Stab. Nur am Werkstück
> einsetzen, Optik‑Block justieren, dann gegen TSAL6100 tauschen.
> Versorgung separat über 5 V + 100 Ω (U_F ~3,2 V ≠ TSAL‑Strang).
> Vor Endmontage: Querchecken mit TSAL6100 + Handykamera + TSOP‑Reichweite
> (offener Punkt 39 in
> [`11-offene-punkte.md`](11-offene-punkte.md)).

## Halloween‑Assets

| Asset | Pfad | Bemerkung |
| --- | --- | --- |
| Sound‑Effekte (14 MP3) | `/Volumes/basteln/Halloween/Lasertag/StationSounds/sound-effects/` | siehe [`09-halloween-setup.md`](09-halloween-setup.md) |
| Spell‑Sounds (5 MP3) | `/Volumes/basteln/Halloween/Lasertag/Spell/` | aktuell ungenutzt im Code |
| 3D‑Druck Target V3 | `/Volumes/basteln/Halloween/Lasertag/Target/Target_V3 v26.3mf` | Einzelmodell |
| 3D‑Druck Target 4‑fach | `/Volumes/basteln/Halloween/Lasertag/Target/4x_Target_V3 v26.gcode.3mf` | druckfertig |
| 3D‑Druck Slide | `/Volumes/basteln/Halloween/Lasertag/Target/4xSlide.3mf` | Slide‑Mechanik |
| Wand‑Gehäuse | – | nicht in den gemounteten Ordnern aufzufinden |

## Software‑Abhängigkeiten

### Logic‑ESP (`InfinitagWand.ino`)

- `arduino-timer` (contrem)
- `Adafruit_NeoPixel`
- `EasyButton`
- `Infinitag_Core` (lokal)
- `IRremote.hpp` (z3t0/IRremote)
- `WiFiManager` (tzapu / wnatth3)

### Sound‑ESP (`InfinitagWandStation.ino`)

- `audio-tools` (schreibfaul1)
- `Adafruit_NeoPixel` (Erbe, kann entfernt werden)

### Target (`Infinitag Target` PlatformIO)

- `z3t0/IRremote@^4.2.0`
- `adafruit/Adafruit NeoPixel@^1.12.0`
- `wnatth3/WiFiManager@^2.0.16-rc.2`
- `contrem/arduino-timer@^3.0.1`
- `Infinitag_Core` (lokal in `lib/`)
