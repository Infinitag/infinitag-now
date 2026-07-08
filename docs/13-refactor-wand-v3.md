# 13 – Refactor: Wand V3 (Optik‑Block, angelötetes Kabel, Halbschalen‑Gehäuse)

> **Status: lebendes Konzept‑Dokument** – wird laufend aktualisiert,
> bis eine getestete und für Halloween 2026 produktiv eingesetzte
> Wand‑Version steht. Dann werden die finalen Inhalte in
> [`02-hardware-wand.md`](02-hardware-wand.md) migriert und die V2.1
> als „historisch" markiert.

Beginn der Diskussion: 2026‑05‑08, parallel zum Station‑V2‑Refactor
([`12-refactor-station-v2.md`](12-refactor-station-v2.md)). Vorgänger:
das aktuelle Wand‑PCB V2.1 (siehe [`02-hardware-wand.md`](02-hardware-wand.md)).

> **Stoßrichtung 2026‑05‑08 (Tobias):** Die Wand bleibt „dumm" und
> kabelgebunden, Sound bleibt in der Station, der Laser bleibt als
> Zielvisier (wegen unsichtbarer IR‑Keule bei 2–10 m). Reines
> Hardware‑Refactor; keine MCU, kein Akku, kein ESP‑NOW im Stab.

---

## 1. Status‑Übersicht

| Bereich | Entscheidung | Status |
|---|---|---|
| **MCU im Stab** | keine – Wand bleibt dumm, alle Signale per Kabel zur Station | ✅ gesetzt |
| **Anbindung** | **kabelgebunden; 6 Adern an einen 2×3‑2,54‑mm‑Lötheader `J1` (HDR‑2X3) angelötet** (kein steckbarer Connector am Stab); 6‑adriges Mantelkabel, Zugentlastung im Knauf; Station‑Seite = grüner 6‑pol Schraubklemmstecker `CN1` | ✅ J1 gesetzt 2026‑06‑08 (vorher USB‑C‑Buchse → kurz Lötpads → finale Lösung 2×3‑Header) |
| **Sound im Stab** | nein – Sound bleibt in der Station | ✅ gesetzt |
| **Laser** | bleibt drin, als Zielvisier bei 2–10 m | ✅ gesetzt |
| **IR‑Linse** | **AstroMedia 304.OM3** (f = 30 mm, Ø 16,5 mm, plankonvex) – 10 Stück bei AstroMedia bestellt 2026‑05‑09 (siehe Abschnitt 3.1.1 zur Linsen‑Auswahl OM3 vs. OM2/OM4) | ✅ gesetzt 2026‑05‑09 |
| **IR‑LED** | **Vishay TSAL6100** (940 nm, ±10°, drop‑in‑Ersatz für TSAL6200) – siehe Abschnitt 3.2 | ✅ gesetzt 2026‑05‑08 |
| **Optik‑Block** | **gemeinsamer Druckteil** für IR‑LED + Linse + Laser, alleiniges Referenzbauteil | ✅ gesetzt |
| **IR‑LED‑Position** | 28 mm hinter Linsen‑Hauptebene (2 mm Defokus → ±1,5° Beam mit TSAL6100, ~13 cm Cone bei 5 m) | ✅ gesetzt, im Test verifizieren |
| **Linsen‑Fixierung** | **4 × M2‑Halte‑Pins** mit Schrauben (kein gedrucktes Feingewinde) | ✅ gesetzt 2026‑05‑08 |
| **Laser‑Justage** | 3 × M2‑Madenschrauben um 120° versetzt am Lasermodul, Endfixierung mit Loctite 243 | ✅ gesetzt |
| **PCB** | komplett neu, V3‑Layout passend zum Stab | ✅ gesetzt |
| **Gehäuse** | komplett neu, **Zauberstab‑Form**, aber für Kinder‑ und Erwachsenen‑Hand griffig und stoßfest | ✅ gesetzt |
| **Gehäuse‑Aufbau** | **Halbschalen längs**, einteilig pro Halbschale (290 mm am Stück, dank Bambu H2D 350×320 mm Bauraum) | ✅ gesetzt 2026‑05‑08 |
| **Material** | PETG (UV‑stabil, zäh, im Bestand) + **PLA‑Stützstruktur (Dual‑Extruder)** + **transparentes PETG** für Edelstein‑Keycap | ✅ gesetzt 2026‑05‑17 |
| **Trigger‑Hardware** | **Gateron Green (Cherry MX Green Klon), 5‑Pin PCB‑Mount, direkt aufgelötet** auf Haupt‑PCB. 36er‑Pack bestellt 2026‑05‑17 (DRAOZA via Amazon, 10,99 €). 80 ± 15 gf Aktivierung, 2,3 mm Pretravel, 50 M Cycles, Click‑Jacket. | ✅ bestellt 2026‑05‑17 |
| **Trigger‑Position** | **Daumen‑Trigger oben am Griff**, MX‑Stem mit gestecktem Edelstein‑Keycap (transparent PETG, hexagonal‑gefacettet) | ✅ gesetzt 2026‑05‑17 |
| **Trigger‑Beleuchtung** | **4× SK6812 5050 Halo** direkt um den MX‑Switch auf dem Haupt‑PCB (statt 1× zentraler NeoPixel) | ✅ gesetzt 2026‑05‑17 |
| **NeoPixel‑Anzahl** | **6 SK6812RGBW** (1 Schaft + 4 Halo um Trigger + 1 Knauf), Daisy‑Chain in der Wand | ✅ gesetzt 2026‑05‑17 |
| **PCBs im Stab** | **2 PCBs:** Haupt‑PCB (Trigger + Halo + 2×3‑Kabel‑Lötheader `J1` + Treiber) + Optik‑PCB (IR‑LED‑Tochterplatine im Optik‑Block). **ESD‑Schutz sitzt auf der Station** (D1/D2 an CN1), nicht im Stab. | ✅ gesetzt 2026‑05‑17, Connector geändert 2026‑06‑08 |
| **Linsen‑Versatz hinter Stab‑Front** | **15 mm** (vorher 5 mm) – konischer Schutz‑Trichter, Front‑Öffnung Ø 14 mm | ✅ gesetzt 2026‑05‑17 |
| **Front‑Stoßlastpfad** | über Halbschalen‑Vorderkante in die Halbschalen; Linsen‑Frontplatte und M2‑Schrauben **nicht** im Lastpfad | ✅ gesetzt 2026‑05‑17 |
| **Front‑Optik (Magic‑Wand‑Look)** | versteckt – Front‑Öffnung Ø 14 mm, gleicher Farbton wie Halbschale, Linse sitzt ~15 mm dunkel zurückgesetzt | ✅ gesetzt 2026‑05‑17 |
| **Oberflächen‑Reliefs** | **Art Nouveau / Jugendstil‑Ranken**, 3 Slot‑Positionen (Spitze‑Schaft‑Übergang, Trigger‑Bereich, Knauf‑Übergang), Erhebung 1,0 mm, mono‑material in V1 (multi‑color erst V2) – siehe §5.8 | ✅ gesetzt 2026‑05‑17, CAD offen |
| **TPU‑Manschette am Schaft** | **optional**, separat gedruckt, bei Bedarf aufgezogen | ✅ optional |
| **TPU‑Schutzring an der Spitze** | **optional / nachrüstbar**, separat gedruckt, aufziehbar – jetzt nicht bestückt (1 Tag / 4 h Halloween‑Einsatz, V2.1 bislang ohne Spitzendefekt) | ✅ Slot offen |
| **Erster Schritt** | Optik‑Block isoliert drucken + IR‑Reichweiten‑Test mit OM3 + TSAL6100, vor PCB‑Design | ✅ gesetzt |

---

## 2. Anforderungen

### 2.1 Funktional

- IR‑Schuss in Reichweite **2–10 m**, sicheres Treffen einer ~80–100 mm
  Target‑Apertur.
- Laser‑Visier sichtbar im Halbdunkel und tageshell, parallel zur
  IR‑Achse mit definiertem, kompensierbarem Versatz.
- 2 SK6812RGBW als Stab‑Effekt‑LEDs.
- Trigger als deutlich spürbarer Mikrotaster.
- Anbindung über fest angelötetes 6‑adriges Kabel an die Station V2; dort
  endet es im 6‑pol Schraubklemmstecker (siehe
  [`12-refactor-station-v2.md`](12-refactor-station-v2.md), Abschnitt 8.11).

### 2.2 Mechanisch / Ergonomisch (Tobias‑Vorgaben 2026‑05‑08)

- **Form:** weiterhin Zauberstab, kein Pistolen‑/Lampen‑Formfaktor.
- **Griff:** für **jung und alt** angenehm zu halten; nicht zu dünn,
  nicht filigran.
- **Stoßfestigkeit:** der Stab landet in Kinderhänden, fällt herunter,
  wird gegen Pfosten/Boden geschlagen → die Konstruktion soll das
  überleben, ohne dass die Optik dejustiert wird.
- **Wartbar:** Linse, IR‑LED, Lasermodul und PCB einzeln tauschbar
  ohne den Stab zu zerstören.

### 2.3 Reproduzierbarkeit

- **3–4 Stäbe** sollen ohne handwerkliche Sonderkenntnisse aus
  Standard‑Komponenten + 3D‑Druck baubar sein.
- Optische Justage **nur einmal pro Stab** (Laser auf 5 m auf
  IR‑Spot eingenordet, dann fixiert) – kein laufendes Nachjustieren.
- Montagezeit pro Stab **< 30 min** (siehe Zielwert in Punkt 26 von
  [`11-offene-punkte.md`](11-offene-punkte.md)).

---

## 3. Optik‑Modul (das Herzstück des Refactors)

### 3.1 Grundprinzip: Optik‑Block als alleiniges Referenzbauteil

Das ursprüngliche Toleranz‑Problem (Punkt 24 in den offenen Punkten)
entsteht durch eine zu lange Toleranzkette:

```
Wand‑PCB → IR‑LED gelötet → Druckteil A (Linsenhalter) → Druckteil B (Stab)
            ±0,2 mm           ±0,3 mm                    ±0,3 mm
            = ±0,8 mm Summe → Linse läuft an LED vorbei
```

**Lösung:** IR‑LED, Linse und Lasermodul sitzen alle in **einem
einzigen Druckteil** (= Optik‑Block), das in einer einzigen
Druckoperation entsteht. Die Toleranz reduziert sich auf die
Punkt‑zu‑Punkt‑Genauigkeit innerhalb desselben Druckteils, typisch
**±0,05 mm** auf einem gut kalibrierten FDM‑Drucker. Der Optik‑Block
wird im Stab‑Gehäuse verschraubt, aber das Stab‑Gehäuse kann den
LED‑zu‑Linse‑Abstand nicht mehr beeinflussen – es fasst den
Optik‑Block nur ein.

```
   ┌──────── Optik‑Block (ein Druckteil) ────────┐
   │                                             │
   │  IR‑LED‑Sitz   Linsen‑Sitz    Laser‑Sitz    │
   │  (Reibsitz)   (OM3 Ø 16,5)    (3 Maden‑S.)  │
   │     ↓              ↓               ↓        │
   │  TSAL6200 ──30 mm──► OM3            Laser   │
   │                                             │
   └─── 4× M3‑Inserts → Stab‑Gehäuse ────────────┘
```

### 3.1.1 Linsen‑Auswahl: warum OM‑3 und nicht OM‑2 oder OM‑4

Tobias‑Klärung 2026‑05‑09: in seinem Bestand lagen tatsächlich **OM‑4‑Linsen**,
nicht OM‑3 (die ursprüngliche Annahme im Doku‑Entwurf war falsch). Bevor
„nimm einfach, was da ist" zur Default‑Antwort wird, haben wir die ganze
AstroMedia‑OM‑Serie sauber gegen unseren Anwendungsfall (TSAL6100 ±10°,
Target 80–100 mm bei 2–10 m, kompakter Stab‑Vorbau) abgeklopft:

| Linse | f | Ø außen | R_eff (eff. Apertur) | Lichterfassung TSAL6100 (±10°) | Block‑Länge | Beam bei 2 mm Defokus | Bewertung |
|---|---|---|---|---|---|---|---|
| OM‑1 | 10 mm | 6 mm | ~2,5 mm | knapp (95 %, Cone 1,8 mm < 2,5 mm) | ~15 mm | sehr breit | zu filigran, Toleranzen kritisch |
| OM‑2 | 15 mm | 8 mm | ~3,8 mm | komplett (95 %, Cone 2,6 mm < 3,8 mm) | ~20 mm | ~±4° (zu breit) | kompakt, aber Beam‑Tuning schwierig |
| **OM‑3** | **30 mm** | **16,5 mm** | **~6,1 mm** | **komplett (95 %, Cone 5,3 mm < 6,1 mm)** | **~35 mm** | **±2° (Soll‑Wert) ✅** | **optimal, gewählt** |
| OM‑4 | 49 mm | 16,5 mm | ~6,1 mm | beschnitten (~65 %, Cone 8,6 mm > 6,1 mm) | ~54 mm | ±1,2° (zu eng) | suboptimal trotz Toleranz |

**Warum OM‑3 alle Kriterien gleichzeitig trifft:**

- **Apertur passt exakt zur TSAL6100:** der Lichtkegel der LED bei 30 mm
  (Radius 5,3 mm) liegt sauber innerhalb der ~6,1 mm effektive Apertur.
  Wir verlieren nahezu kein Licht.
- **Natürliche Strahl‑Divergenz** aus der Chip‑Größe (≈ 1 mm) liegt bei
  ±1°, mit nur 2 mm Defokus erreichen wir die gewünschten ±2° – das
  ist ein Maß, das ein Bambu H2D mit 0,12 mm Layern reproduzierbar trifft.
- **Block‑Länge 35 mm** passt in 50 mm Spitze + 40 mm Schaft, ohne dass
  der Stab vorne wuchtig wirkt.
- **Mechanisch robust** mit 16,5 mm Außendurchmesser – die
  Halte‑Pin‑Lösung mit 4 × M2 funktioniert sauber.

**Warum OM‑4 trotz Bestand nicht eingesetzt wird:**

Die OM‑4 (f = 49 mm, gleicher Außendurchmesser 16,5 mm wie OM‑3) hätte
Vorteile: tolerantere Defokus‑Empfindlichkeit (~0,5°/mm statt 0,75°/mm)
und natürlich engerer Beam (±0,6° natürlich). Aber die effektive
Apertur (~6,1 mm Radius) ist **kleiner als der Lichtkegel der TSAL6100
bei 49 mm** (8,6 mm Radius) → ~35 % der Lichtleistung wird abgeschnitten.
Effekt: rechnerisch ~17 % Reichweiten‑Verlust gegenüber OM‑3 (sqrt(0,65/0,95)).
Das wäre vermeidbar, deshalb OM‑3.

**Warum nicht OM‑2 für ein noch kompakteres Design:**

Bei f = 15 mm wäre der Optik‑Block ~20 mm lang statt 35 mm – verlockend
kompakt. Aber:

- der 8 mm Außendurchmesser ist mechanisch grenzwertig für die
  4 × M2‑Halte‑Pin‑Frontplatte
- die Defokus‑Empfindlichkeit ist hoch (1°/0,15 mm) – jede
  Druck‑Toleranz schlägt auf den Beam durch
- die natürliche Divergenz aus der Chip‑Größe ist mit ±1,9° schon nahe
  am Soll‑Wert, mit Defokus wird der Beam schnell zu breit

Kurz: OM‑2 wäre zu „zappelig" beim Druck, der Komfortgewinn (15 mm
weniger Stab‑Spitze) ist es nicht wert.

**Premium‑Optiken (Edmund/Thorlabs/Glas‑Achromate)** wären bei 940 nm
nicht messbar besser als der AstroMedia‑Präzisionsguss‑Acryl, kosten
aber 30–80 € pro Stück. Kein Argument für unseren Halloween‑Use‑Case.

### 3.2 IR‑LED‑Auswahl: TSAL6200 → TSAL6100

Tobias‑Anstoß 2026‑05‑08: „Wir können auch eine andere IR‑LED nehmen,
die vom Winkel und der Leistung her besser passt." Sehr gute Idee – die
verbaute TSAL6200 ist Standard, aber für die OM3 nicht optimal.

**Problem mit der TSAL6200 + OM3:**

| Größe | Wert |
|---|---|
| TSAL6200 Halbwertswinkel | ±17° |
| Lichtkegel‑Radius bei 30 mm (= f der OM3) | 30 × tan(17°) = **9,2 mm** |
| OM3 Innen‑Ø (effektive Apertur) | 12,2 mm → **Radius 6,1 mm** |
| **Konsequenz** | Lichtkegel ist **größer als die Linsen‑Apertur** → ~50 % der IR‑Energie geht an der Linse vorbei |

Die OM3 schneidet bei der TSAL6200 die Hälfte des Lichts ab. Der Rest
wird zwar kollimiert, aber wir verschenken Reichweite und Treffsicherheit.

**Lösung: TSAL6100 als drop‑in‑Ersatz**

Vishay TSAL6100 ist mechanisch identisch zur TSAL6200 (gleiches 5 mm
T‑1¾ Gehäuse, gleiche Pin‑Belegung, gleiche elektrische Spec) – aber
mit deutlich engerem Abstrahlwinkel und höherer Achs‑Intensität:

| Parameter | TSAL6200 (alt) | **TSAL6100 (neu)** | Δ |
|---|---|---|---|
| Wellenlänge | 940 nm | 940 nm | gleich – TSOP‑Empfänger sind auf 940 nm optimiert |
| Halbwertswinkel | ±17° | **±10°** | engerer Beam |
| Lichtkegel‑Radius bei 30 mm | 9,2 mm | **5,3 mm** | **passt komplett in OM3‑Apertur** |
| Achs‑Intensität I_e (typ., 100 mA DC) | 80 mW/sr | **170 mW/sr** | **2,1× heller auf der Achse** |
| Forward‑Spannung U_F | 1,35 V | 1,35 V | gleich – R2 = 33 Ω bleibt |
| Maximaler Pulsstrom | 1 A | 1 A | gleich |
| Gehäuse | 5 mm T‑1¾ klar | 5 mm T‑1¾ klar | gleich – Optik‑Block unverändert |
| Preis Reichelt | ~0,40 € | ~0,50 € | minimal |

**Effektive Reichweiten‑Verbesserung mit TSAL6100 + OM3:**

- Gesamte Lichtenergie der TSAL6100 (±10°) trifft die OM3 → **2× mehr
  Strahlleistung am Linsen‑Ausgang** als mit TSAL6200.
- Höhere axiale Intensität konzentriert das Licht **enger** im
  parallelen Strahl nach der Linse.
- Theoretische Reichweiten‑Steigerung: bei gleicher Empfindlichkeit
  des TSOP‑Empfängers √2 ≈ 1,4× → aus 10 m Sicherheits‑Reichweite
  werden ~14 m. Praktisch eher 2× (12–20 m), weil auch der parallele
  Strahl länger eng bleibt.
- **Praxis‑Effekt:** Wir hatten in der Defokus‑Tabelle (Abschnitt 3.3)
  ±2° / 17 cm Cone bei 5 m. Mit TSAL6100 wird daraus eher ±1,5° /
  ~13 cm Cone bei 5 m. **Heller, fokussierter, treffsicherer.**

**Alternativen (falls TSAL6100 nicht lieferbar):**

| Alternative | Halbwerts‑Winkel | I_e (mW/sr) | Hinweis |
|---|---|---|---|
| OSRAM SFH 4544 | ±10° | ~135 | gleiches Package, 940 nm |
| OSRAM SFH 4547 | ±10° | ~150 | etwas teurer, sonst gleichwertig |
| TSAL6400 (Vishay) | ±25° | ~70 | **schlechter als TSAL6200**, NICHT verwenden |
| Vishay VSLY5940 | ±20° | ~90 | breit, NICHT empfehlen |
| Power‑IR‑LED (z. B. SFH 4715AS) | ±45° (3W) | hoch absolut, niedrig pro sr | für **Kamera‑Beleuchtung** gedacht, nicht für Visier‑IR |

**Entscheidung 2026‑05‑08: TSAL6100.** Bestellung in Sammelposition mit
dem nächsten Reichelt‑Auftrag (gemeinsam mit TVS‑Diode und Mikrotaster).
Bestellmenge: 8 Stück (4 Stäbe + 4 Reserve), ~4 €.

> **Sicherheit:** TSAL6100 emittiert wie TSAL6200 nur sichtbar **un**sichtbares
> 940 nm IR. Augensicherheit: gleich, weiterhin LED‑Klasse 1 ohne
> spezielle Schutzmaßnahmen. Trotzdem: nicht gezielt in offene Augen,
> insbesondere von Kindern, halten – ist eh durch das diffuse Linsen‑Bild
> kein Punktstrahl wie der Laser.

> **Hinweis zur Linsen‑Wahl bei TSAL6100:** Mit dem engeren Beam wäre
> sogar die OM2 (f = 15 mm, kompakter) eine Option, weil deren kleine
> Apertur (innen Ø 7,6 mm, Radius 3,8 mm) bei 15 mm × tan(10°) = 2,6 mm
> Lichtkegel‑Radius gut harmoniert. Optik‑Block würde 15 mm kürzer.
> **Bleibe bei OM3, weil im Bestand** – aber gut zu wissen, falls wir
> mal kompakter bauen wollen.

### 3.2.1 Justage‑LED: Vishay VLHW5100 (weiß) als sichtbarer Zwilling

**Problem:** TSAL6100 emittiert bei 940 nm → für das menschliche Auge
**unsichtbar**. Der Optik‑Block muss aber auf den richtigen Defokus
(28 mm hinter Linsen‑Hauptebene, ~2 mm vom Brennpunkt entfernt, siehe
§ 3.3) eingestellt werden. Ohne sichtbares Licht heißt das: blind
einstellen, dann mit Handy‑Kamera (IR‑Filter‑Pixel sehen 940 nm noch
schwach lila) prüfen, dann nachjustieren – aufwendig und ungenau.

**Lösung:** Eine weiße sichtbare LED, die mechanisch und optisch
möglichst identisch zur TSAL6100 ist, in den Optik‑Block einsetzen,
Defokus per Auge an einer Wand einstellen, dann wieder TSAL6100
einsetzen.

**Auswahl 2026‑05‑23: Vishay VLHW5100** (Farnell Best.‑Nr. 3777973,
0,21 € / Stück bei 100 Stück, ca. 21 €). Begründung:

| Kriterium | VLHW5100 | TSAL6100 | Bewertung |
|---|---|---|---|
| Hersteller | Vishay | Vishay | **gleiche Fertigung → konsistente Chip‑Z‑Position im Gehäuse** |
| Gehäuse | 5 mm T‑1¾ klar, untinted | 5 mm T‑1¾ klar | identisch → Press‑Fit Ø 4,9 mm passt ohne Anpassung |
| Halbwerts‑Winkel 2θ½ | **±10°** datenblatt‑spezifiziert | ±10° | **identische Strahlcharakteristik** |
| Farbe | weiß (Cool White, sichtbar) | 940 nm IR (unsichtbar) | sichtbar = Zweck der Justage |
| I_V (typ.) | 5.600–11.200 mcd | n/a | hell genug für Wand‑Projektion |
| U_F | 2,8–3,6 V | 1,35 V | **wichtig: separat versorgen, NICHT an 33 Ω/5 V‑Strang** |

**Geometrie‑Vorteil:** Bei Vishay sitzt der Die innerhalb des 5‑mm‑Bodies
nach Datenblatt an etwa der gleichen Z‑Position wie bei der TSAL‑Serie
(Tolerierung ±0,2 mm laut Vishay‑Standard). Das heißt: Wenn der Brennpunkt
mit der VLHW5100 visuell scharf ist, ist er es auch mit der TSAL6100
in derselben Bohrung – die Justage überträgt sich 1:1.

**Caveat – muss noch verifiziert werden:** Die Z‑Position des Dies
*kann* zwischen den Vishay‑Familien minimal abweichen. Vor dem
endgültigen Verkleben des Optik‑Blocks: Defokus mit VLHW5100 einstellen,
TSAL6100 einsetzen, mit Handy‑Kamera (Frontkamera ohne IR‑Filter
funktioniert oft besser als Rück) am 5‑m‑Ziel querchecken. Siehe offener
Punkt 39 in [`11-offene-punkte.md`](11-offene-punkte.md).

**Anschluss zur Justage:**

- Nicht an den 33 Ω/5 V‑Strang der TSAL6100 hängen! Bei U_F = 3,2 V
  fließen statt 100 mA nur ~55 mA – LED leuchtet, aber Vorwiderstand
  ist falsch dimensioniert.
- Für die Justage: separates Steckbrett, 5 V Labornetzteil, 100 Ω
  → ca. 18 mA, LED hellt sich gut, kein thermisches Problem.
- LED **nicht in den fertigen Stab einbauen**, nur als Werkstück‑LED
  zum Justieren des Optik‑Blocks vor dem Endeinbau der TSAL6100.

**Workflow:**

1. Optik‑Block gedruckt, OM3 eingelegt.
2. VLHW5100 in den IR‑LED‑Sitz pressen (gleiche Press‑Fit‑Bohrung).
3. An 5 V/100 Ω einschalten, Block auf weiße Wand bei ~5 m richten.
4. Defokus durch Verschieben oder Beilegen einer 0,5‑mm‑Distanzscheibe
   so einstellen, dass die Wand‑Projektion einen sauberen kreisförmigen
   Spot von ca. 13 cm Durchmesser zeigt (= ±1,5° Cone bei 5 m, siehe
   § 3.3).
5. Distanzscheibe / Block‑Geometrie als finale Z‑Position übernehmen
   (3D‑Druck‑STL anpassen, falls Maß nicht exakt 28 mm trifft).
6. VLHW5100 herausziehen, **TSAL6100 + Tochter‑PCB einsetzen**,
   verkleben.
7. IR‑Reichweiten‑Test mit TSOP‑Receiver gegenchecken (Punkt 39).

**Bestand:** 100 × VLHW5100 bei Farnell bestellt 2026‑05‑23 (BestNr.
3777973). Davon werden für die 4 produktiven Stäbe je 1 Stück zur
Justage gebraucht (= 4), Rest bleibt im Bauteilbestand für künftige
Optik‑Block‑Iterationen (OM2, OM11, Target‑Optik, …).

### 3.3 IR‑LED‑Sitz: dreifach redundante Zentrierung

Drei mechanische Maßnahmen, kombinierbar – jede einzelne reicht meist
schon, alle zusammen ergeben ±0,05 mm Reproduzierbarkeit:

| Mechanismus | Maß | Wirkung |
|---|---|---|
| **Reibsitz auf der LED‑Außenkontur** | Bohrung Ø 4,9 mm × 5 mm tief (TSAL6200‑Body Ø 5,0 mm) | LED wird per Hand reingepresst, PETG gibt minimal nach, sitzt fest und zentriert |
| **Konische Vor‑Centerung** | Bohrungs‑Mund auf Ø 5,3 mm aufgeweitet, läuft auf Ø 4,9 mm zu | Erleichtert Einsetzen, führt automatisch zur Mitte |
| **Beine als Selbstführung** | 2 × Bohrungen Ø 0,8 mm im 2,54 mm Pitch hinter dem LED‑Sitz, ~3 mm tief | LED‑Beine zwingen die LED in definierte Drehlage, blockieren axiale Verschiebung |

**Anschluss:** Die LED‑Beine kommen auf der Rückseite des Optik‑Blocks
heraus und werden auf einer **kleinen Tochter‑PCB** (10 × 14 mm,
2‑Layer, JST‑PH 2‑pol Stecker) verlötet. Die Tochter‑PCB enthält keine
Treiberlogik – nur die Lötpads für die LED‑Beine, einen optionalen
SMD‑Widerstand‑Footprint (R1 = 0 Ω Brücke, falls die Wand‑PCB‑R2
(33 Ω) als Vorwiderstand nicht ausreicht, kann hier ein zusätzlicher
Widerstand bestückt werden) und den 2‑pol JST‑Stecker. Treiber‑MOSFET
und Vorwiderstand bleiben auf dem Wand‑Haupt‑PCB.

> **Vorteil dieser Trennung:** Die LED ist nicht direkt am Haupt‑PCB
> verlötet. Wenn die LED stirbt: Tochter‑PCB nachbestellen (~0,30 €
> bei JLCPCB im Sammelversand) oder LED nachlöten, ohne den ganzen
> Stab zerlegen zu müssen. Die Optik‑Block‑Justage bleibt erhalten.

#### 3.3.1 Mechanische Fixierung: Klemmblock statt Schrauben (Entscheidung 2026‑05‑17)

Die ursprünglich angedachte Variante „Optik‑PCB mit 2× M2 am Optik‑Block
verschrauben" wurde 2026‑05‑17 von Tobias verworfen. Begründung: M2‑
Inserts in der ~3 mm dünnen Optik‑Block‑Wand sind grenzwertig (Wand
kann reißen, Insert wackelt). Stattdessen:

**Klemmblock‑Konstruktion:**

```
Querschnitt Optik‑Block bei der LED‑Aufnahme:

   Linse vorne (in Linsen‑Sitz, §3.4)
       │
       ▼
   ┌──────────────┐
   │              │   ◀── runde Aufnahme Ø 4,9 mm × 3 mm
   │  LED         │     (vorne, Press‑Fit für Linsenkuppel)
   │  Kuppel      │
   │  (rund)      │
   └─────┬────────┘
         │
   ┌─────┴────────┐
   │ □ Sockel  □ │   ◀── eckige Aufnahme 5 × 5 mm × 2 mm
   │  TSAL6100   │     (Verdrehsicherung, Press‑Fit für
   │             │      eckigen LED‑Body)
   └───╥─────╥───┘
       ║     ║         ◀── LED‑Beine durch Optik‑Block
       ║     ║              (2 × Ø 0,8 mm Bohrungen, 2,54 mm Pitch)
   ════╬═════╬══════════
       ║     ║         ◀── Optik‑PCB (10 × 14 mm)
       ║     ║              flach auf einer Auflage liegend,
       ║     ║              LED‑Beine durchgesteckt und gelötet
   ════╬═════╬══════════
       │     │
   ┌───┴─────┴────┐    ◀── 3D‑gedruckter Klemmblock
   │  Klemmblock  │       (10 × 14 × 6 mm PETG)
   │  drückt PCB  │       drückt das PCB von hinten an die
   │  an LED‑     │       Auflage, hält den ganzen Stack zusammen
   │  Aufnahme    │
   └──────╥───────┘
          ║
       M3 Schraube     ◀── M3‑Schraube von hinten durch den
          ║                Klemmblock und den Optik‑Block‑Boden
          ▼                (M3‑Insert sitzt im BODEN, nicht in der
   ══════════════════         dünnen Wand → robust)
   Optik‑Block‑Boden
```

**Vorteile gegenüber direkter Verschraubung des PCBs:**

| Aspekt | Klemmblock (gewählt) | M2‑Schrauben am PCB (verworfen) |
|---|---|---|
| M‑Inserts in dünner Wand | nicht nötig (M3 sitzt im Boden) | nötig (M2 in 3 mm Wand, ausreiß‑gefährdet) |
| LED‑Position‑Toleranz | ±0,05 mm (durch Press‑Fit der LED im Druckteil definiert) | ±0,2 mm (durch Schrauben‑Passung addiert) |
| PCB‑Fläche | komplett für Bauteile nutzbar | 2× M2‑Bohrungen verlieren Bauteil‑Platz |
| LED‑Tausch | Klemmblock‑M3 lösen → PCB samt LED rausnehmen → neue LED einlöten → wieder reinklemmen | Schrauben lösen, gleicher Aufwand |
| Druckteil‑Aufwand | 1 zusätzliches kleines Druckteil (~6 g PETG) | 0 zusätzliche Druckteile |

**Klemmblock‑Maße:**

| Element | Wert |
|---|---|
| Außenmaß | 10 × 14 × 6 mm |
| Material | PETG opak |
| Vertiefung für PCB‑Auflage | 0,3 mm an einer Seite (PCB‑Sitz mit leichtem Spiel) |
| M3‑Durchgangsloch mittig | Ø 3,2 mm |
| Optional: Aussparung für JST‑Stecker | 5 × 4 mm Tasche, damit der JST nicht den Klemm‑Druck stört |
| Druckzeit | ~10 min auf Bambu H2D |

**Stand 2026‑05‑17:** Konzept festgeschrieben, CAD‑Modellierung mit Optik‑
Block zusammen offen. Siehe `11-offene-punkte.md` Punkt 35.

### 3.4 Linsen‑Sitz: OM3 mit 4 M2‑Halte‑Pins (robust statt Feingewinde)

Die OM3 hat eine umlaufende Acrylglas‑Fassung mit Steg, Außen‑Ø
**16,5 mm**. Sie wird nicht in den Optik‑Block eingeklebt, sondern
**eingelegt und mit 4 M2‑Halte‑Pins gehalten**, die von vorn das
Steg‑Element der Fassung am Anschlag fixieren. So bleibt die Linse
tauschbar, falls eine OM3 verkratzt oder zerbricht.

> **Entscheidung 2026‑05‑08:** robustere Lösung statt gedrucktem
> M20×0,5‑Feingewinde. Feingewinde am FDM‑Druck ist auf Hobby‑Druckern
> oft toleranzkritisch; 4 Schrauben sind reproduzierbarer und nehmen
> auch Stoßlasten zuverlässig auf.

| Maß | Wert | Begründung |
|---|---|---|
| Linsen‑Aufnahme‑Bohrung | Ø 16,7 mm × 4 mm tief | OM3 außen 16,5 mm + 0,2 mm Spiel für Einlege‑Sitz |
| Anschlag‑Innenradius | 3,5 mm | drückt auf den Steg der Fassung, hält Linse axial |
| Halte‑Pins | 4 × Ø 2,5 mm Bohrung in einer Frontplatte, 90° versetzt um die Linse | M2‑Schrauben mit Senkkopf, kommen von vorn durch eine kleine Frontplatte |
| Frontplatte | 2 mm dick, gedruckt, gleicher Material‑Farbe wie Optik‑Block, hält die Halte‑Pins und drückt auf den Linsen‑Steg | abnehmbar zum Linsen‑Tausch |
| M2‑Inserts im Optik‑Block | 4 × Ruthex M2 Short, in Boss‑Auswüchsen um den Linsen‑Sitz | aus Bestand |
| Schrauben | 4 × M2 × 6 mm Senkkopf | Sammelposition mit den Laser‑Madenschrauben |

**LED‑zu‑Linse‑Abstand:** Die OM3 hat f = 30 mm. Im Brennpunkt liegt
die LED → quasi paralleler Strahl mit ±0,5° Beam, das ergäbe bei 5 m
nur ±4 cm Cone – sehr eng, für zitternde Kinderhände zu eng.
**Konstruktiver Defokus von 2 mm** (LED bei 28 mm hinter Linse statt
30 mm) erzeugt ~±2° Beam → bei 5 m ~17 cm Cone. Das passt zur
Target‑Apertur (~80–100 mm) plus Zielfehler‑Reserve.

### 3.4.1 Linsen‑Versatz hinter der Stab‑Front (Stoßschutz)

**Entscheidung 2026‑05‑17 (Tobias):** Die OM3 ist aus Präzisionsguss‑Acryl
und kratzempfindlich (Mohshärte ~2,5). Bei nur 5 mm Versatz zur
Stab‑Frontkante kommen Finger, Sand und Pfostenkanten zu leicht ran.
Linsen‑Frontkante rutscht deshalb auf **15 mm hinter die
Stab‑Vorderkante**. Die Stab‑Spitze bekommt einen **konischen
Schutz‑Trichter** mit Front‑Öffnung Ø 14 mm.

**Strahl‑Mathematik – darf die Front‑Öffnung kleiner sein als die Linse?**

Mit TSAL6100 + OM3 + 2 mm Defokus tritt der Strahl an der
Linsen‑Frontseite mit ~12,2 mm Durchmesser aus und divergiert mit ±2°.
Mit zunehmender Distanz wird der Strahl breiter:

| Distanz Linse → Front | Strahl‑Ø an der Front | Front‑Loch Ø 14 mm | Lichtverlust |
|---|---|---|---|
| 5 mm (Doku‑Stand alt) | 12,55 mm | passt | 0 % |
| 10 mm | 12,90 mm | passt | 0 % |
| **15 mm (neu, Soll)** | **13,25 mm** | **passt, 0,75 mm Reserve** | **0 %** |
| 20 mm | 13,60 mm | passt knapp (0,4 mm) | 0 % |
| 25 mm | 13,95 mm | passt nicht mehr | erste Beschneidung |

→ **15 mm ist der Sweet‑Spot:** maximaler Schutz‑Abstand, ohne den
parallelen Strahl zu beschneiden. Bei 20 mm wird's grenzwertig (Fertigungs‑
toleranzen schlagen durch), 25 mm beschneidet bereits.

**Konstruktive Konsequenzen** (Details in §5.1 und §5.7):

- Optik‑Block sitzt 15 mm hinter der Halbschalen‑Vorderkante (statt
  bündig). Die Spitzen‑Länge in §5.1 bleibt 50 mm – passt mit
  15 mm Schutzdistanz + 35 mm Optik‑Block.
- Halbschalen‑Vorderkante bildet einen konischen Trichter, der von
  Ø 14 mm an der Front nach hinten auf Ø 26 mm aufweitet
  (Linsen‑Frontplatten‑Sitz).
- Die **Linsen‑Frontplatte (mit 4 M2‑Halteschrauben) bleibt
  konstruktiv erhalten**, sitzt aber jetzt **mechanisch entkoppelt
  vom Außen‑Stoß**: ein Front‑Aufprall geht in die Halbschalen, nicht
  in die Frontplatte. M2‑Schrauben sehen damit nie eine Schlaglast.
- **Positiver Nebeneffekt:** der konische Trichter wirkt als
  Streulicht‑Blende. Schräg von vorne einfallendes Umgebungs‑IR (Sonne,
  Halogen‑Strahler) wird vor der OM3 abgeschattet, was im Halbdunkel
  die TSOP‑Empfänger der Targets weniger gestört arbeiten lässt.

> **Front‑Stil (Tobias‑Vorgabe 2026‑05‑17):** „Magic‑Wand‑Look", nicht
> „Geräte‑Look". Der Trichter ist innen dunkel (gleicher PETG‑Ton wie
> Halbschale), die Linse glimmert tief im Stab. Keine glänzende
> Schutzscheibe und kein metallischer Bezel.

> **TPU‑Schutzring an der Spitze:** als zusätzliche Stoßdämpfung außen
> aufgezogen (Innen‑Ø 30 mm, Außen‑Ø 34 mm, Länge 15 mm, TPU 95A) ist
> möglich, **wird für die Erstauflage 2026 aber nicht bestückt** – die
> V2.1 hat bislang keinen Spitzendefekt gezeigt, bei 1 Tag und ~4 h
> Peak ist die Belastung überschaubar. Die Geometrie ist so ausgelegt,
> dass der Ring nachgerüstet werden kann, falls nach Halloween 2026
> sichtbare Schäden an der Halbschalen‑Front entstanden sind.

| Position | Beam | Cone bei 5 m | Cone bei 10 m | Bewertung |
|---|---|---|---|---|
| LED bei 30 mm (Brennpunkt) | ±0,5° | ±4 cm | ±9 cm | zu eng, jeder Zielfehler verfehlt |
| **LED bei 28 mm (Defokus 2 mm)** | **±2°** | **±17 cm** | **±35 cm** | **Soll‑Wert; Treffen sicher, kein „Streulicht‑Spam"** |
| LED bei 25 mm (Defokus 5 mm) | ±5° | ±43 cm | ±87 cm | zu breit, Targets in der Nachbarschaft erfasst |

> **Verifikation im Test:** Nach dem Druck des Optik‑Blocks mit einer
> Test‑Wand auf 5 m und 10 m messen, ob der Beam tatsächlich die
> erwartete Cone‑Breite hat. Optimum ist erreicht, wenn ein Target
> bei ±10 cm Zielfehler bei 5 m noch sicher angesprochen wird.

### 3.5 Lasermodul: parallel zur IR‑Achse mit definiertem Versatz

**Verbaute Lasermodule:** KY‑008‑artige 5 V Dot‑Laser, Modul‑Ø ~6 mm,
Länge ~10 mm, axiale Linse vorne. Fertigungstoleranz des Strahl‑Austritts
relativ zur Modul‑Achse: ±2°. Das heißt: selbst eine perfekte
Optik‑Block‑Geometrie reicht **nicht** – jeder Stab braucht **eine**
einmalige Justage.

**Konstruktion:**

```
         ┌── Optik‑Block (Querschnitt vorn) ──┐
         │                                    │
         │   ┌─ OM3 ─┐                        │ ← Stab‑Vorderseite
         │   │       │                        │
         │   └───────┘                        │
         │      ↑                             │
         │      IR‑LED 28 mm hinter Linse     │
         │                                    │
         │   ●── 12 mm vertikaler Versatz ──● │
         │                                    │
         │  Lasermodul Ø 6 mm                 │
         │   in Bohrung Ø 6,5 mm              │
         │   mit 3 Madenschrauben M2          │
         │   um 120° versetzt                 │
         └────────────────────────────────────┘
```

| Element | Maß | Hinweis |
|---|---|---|
| Lasermodul‑Bohrung | Ø 6,5 mm × 12 mm tief | KY‑008 Ø 6 mm + 0,5 mm Spiel für Justierbarkeit |
| Madenschraube M2 × 6 mm | 3 Stück, 120° versetzt | drücken auf Modulgehäuse, ±2° Justage |
| M2‑Inserts | Ruthex M2 Short, in 3 Boss‑Auswüchsen am Optik‑Block | Standard‑Bestand bei Tobias |
| Vertikaler Versatz IR↔Laser | **12 mm** (Lasermodul über IR‑Achse) | bekannt und reproduzierbar; Spieler kompensiert mental |
| Laser‑Endfixierung | Loctite 243 (mittelfest, lösbar) auf Madenschrauben‑Gewinden | nach erfolgreicher Justage, vor Halloween |
| Laser‑Anschluss | 2‑pol JST‑PH am Modulkabel, am Optik‑Block‑Außen | wechselbar bei Defekt, Modul kommt fertig mit Kabel |

**Justage‑Vorgehen pro Stab (einmalig):**

1. Stab auf Stativ in 5 m Entfernung zu einer Test‑Wand.
2. Test‑Wand mit 30 × 30 cm Karton, Mittelpunkt markiert.
3. IR‑Schuss → IR‑Detektor (z. B. Smartphone‑Kamera, IR ist sichtbar)
   zeigt den IR‑Spot. Position auf der Wand markieren.
4. Laser einschalten (Wand auf Setup‑Modus, Laser permanent an –
   siehe Software‑Hinweis unten).
5. Drei Madenschrauben so verstellen, dass der Laser‑Punkt 12 mm
   **über** dem IR‑Spot liegt (= konstruktiver Versatz).
6. Madenschrauben mit Loctite 243 fixieren.
7. Test mit Schuss auf Target bei 5 m und 10 m – treffen muss
   reproduzierbar sein, wenn der Spieler 12 mm unter dem Laser‑Spot
   zielt.

> **Software‑Hinweis (Station V2):** Für die Justage‑Phase braucht
> der Logic‑ESP der Station einen „Setup‑Modus", der den Laser
> permanent ansteuert. Im OLED‑Menü als Punkt „Stab‑Setup → Laser
> permanent" einbauen. Siehe Punkt 11 unten.

### 3.6 Optik‑Block: Außenkontur und Befestigung

**Außenmaße:**

| Maß | Wert | Begründung |
|---|---|---|
| Block‑Außenmaß B × H × T | 28 × 28 × 35 mm | Linsen‑Ø 16,5 + 4 mm Wand + Lasermodul‑Versatz; LED‑zu‑Linse 30 mm + 5 mm Block‑Material vor LED |
| Material | PETG, 6 Perimeter, 100 % Infill | maximale Steifigkeit, Toleranzen halten |
| Druck‑Orientierung | **Front (Linsen‑Seite) auf Bauplatte** | Linsen‑Anschlag wird sauber, LED‑Bohrung wächst exakt rechtwinklig zur Linsen‑Ebene |
| Druck‑Genauigkeit | Layer 0,12 mm, 0,4 mm Düse | feiner als Standard, weil die LED‑Sitz‑Toleranz kritisch ist |
| Einbau im Stab | 2 × M3‑Inserts in Boss‑Erweiterungen seitlich, wird im Stab‑Vorderteil eingeschraubt | Optik‑Block trägt sich selbst, Stab‑Gehäuse fasst nur ein |

**Druck‑Strategie:** Front auf Bauplatte ist entscheidend. Die LED‑Bohrung
wächst dann exakt senkrecht zur Linsen‑Auflage (Layer‑Ebene), die OM3
liegt sauber auf, und die LED zeigt exakt zur Linsenmitte. Kein
Rasten, kein Schiefstehen.

---

## 4. PCB V3 (Wand‑Haupt‑PCB)

> **⚠️ Update 2026‑06‑08 – Connector‑Wechsel USB‑C → angelötetes Kabel.**
> Der Stab‑Connector wurde stationsseitig auf einen **grünen 6‑pol
> Schraubklemmstecker** umgestellt; auf der **Wand wird das Kabel direkt an
> die Haupt‑PCB angelötet** (keine USB‑C‑Buchse mehr). Begründung in
> [`12-refactor-station-v2.md`](12-refactor-station-v2.md) § 8.11.
> Die folgenden Abschnitte 4.1 (USB‑C‑Buchse/ESD), 4.2 und 4.3 sowie die
> Mechanik‑Prosa in **§ 5.5** und die ASCII‑Pinout‑Skizzen enthalten teils
> noch die alte USB‑C‑Formulierung. **Decision‑/Bestell‑relevante Stellen
> sind unten korrigiert; die rein mechanische Kabel‑Beschreibung in § 5.5
> ist als Verifikationspunkt zu überarbeiten** (Zugentlastung gilt sinngemäß
> weiter, nur „Buchse" → „Lötpads", „kein Stecker" gilt erst recht).

> **⚠️ Update 2026‑06‑21 – Bring‑up neues Stab‑PCB: NEO_DATA‑Buffer am Stab.**
> Beim Inbetriebnehmen des V3‑PCB (direktes Kabel, kein USB‑C mehr) zeigte
> sich die Datenstrecke Station → Stab als **signaltechnisch grenzwertig**:
> Grün‑Aussetzer und – bei voller Helligkeit – Magenta‑Fehler (Rot‑Byte
> „verschmiert" ins Blau‑Byte). Vollständige Diagnose in
> [`11-offene-punkte.md`](11-offene-punkte.md) § 3a.
> **Verbindliche Konsequenzen fürs Stab‑PCB:**
> - **Re‑Driver `74AHCT1G125` (1G Single‑Gate) am Stab‑Eingang**, direkt vor
>   der ersten LED: /OE (Pin 1) → GND, A (Pin 2) ← NEO‑Daten vom Kabel,
>   GND (Pin 3) → GND, Y (Pin 4) → 0‑Ω‑Platzhalter → erste LED‑DIN,
>   VCC (Pin 5) → +5 V, **100 nF an VCC/GND**. Frischt das Signal lokal auf
>   saubere 5‑V‑Flanken → Helligkeit/Bitfolge/Kabellänge werden unkritisch.
>   (Das ist **kein** ESD‑Ersatz – die ESD‑Dioden bleiben auf der Station.)
> - Serienwiderstand am Buffer‑**Ausgang** als **0‑Ω‑Platzhalter** (Tuning
>   ~50–100 Ω nur falls Klingeln). Am Eingang kein Serien‑R nötig.
> - **LED‑Typ bestätigt: SK6812 5050 RGBW**; die zuvor getesteten WS2812B‑4020
>   waren beim Handlöten zu unzuverlässig. Firmware‑Datenformat **`NEO_GRBW`**.
> - Entkopplung: **100 nF je LED** + **ein 10–22 µF Bulk** am LED‑Cluster.
>
> **⚠️ Verifikationspunkt LED‑Anzahl:** Dieses Dok plant **6 SK6812RGBW**
> (1 Schaft + 4 Halo + 1 Knauf, s. § 4.1‑Tabelle / Steckbrief). Der aktuelle
> Bring‑up‑Aufbau am Stab läuft mit **2 LEDs** (Tobias‑Aussage 2026‑06‑21:
> „2 oder 4 LEDs"). Vor PCB‑Finalisierung klären, ob die 6‑LED‑Architektur
> (Schaft/Halo/Knauf) so bleibt oder auf 2–4 reduziert wird.

### 4.1 Was alles drauf muss

| Block | Bauteile | Aufgabe |
|---|---|---|
| **2×3‑Lötheader `J1` (statt USB‑C‑Buchse)** | 2×3‑Stiftleiste 2,54 mm (HDR‑2X3), 6 Pins; die 6 Kabeladern werden an die Pins gelötet | Belegung 1:5V 2:GND 3:NEO 4:IR 5:LAS 6:TRIG (identisch zur Station‑CN1 → gerades 1:1‑Kabel). C5/C6 als lokale 5‑V‑Stützung an J1‑Pin 1/2 |
| **Trigger** | **Gateron Green (Cherry MX Green Klon), 5‑Pin PCB‑Mount, direkt aufgelötet** | Schaltet Trigger‑Pin gegen GND (active‑LOW). Stem (MX‑Plus) trägt Edelstein‑Keycap. Bestellt 2026‑05‑17 (36er‑Pack DRAOZA) |
| **Trigger‑Halo** | **4× SK6812 5050 RGBW SMD onboard**, diagonal um den MX‑Switch (~7 mm vom Stem‑Zentrum) | beleuchtet Edelstein‑Keycap von unten und Halbschalen‑Lichtfuge um den Switch |
| **IR‑Treiber** | Q1 = **BSS138** Logic-Level N-MOSFET, R3 = 820 Ω Series‑R am Gate, R5 = 10 kΩ Gate‑Pulldown, R2 = 33 Ω Vorwiderstand für IR‑LED | Schaltet IR‑LED gegen GND, LED auf Optik‑PCB im Optik‑Block. Steuersignal `IR_TX` (3,3 V Logik) kommt direkt vom Station‑ESP über USB‑C |
| **Laser‑Treiber** | Q2 = **BSS138** Logic-Level N-MOSFET, R4 = 820 Ω Series‑R, R6 = 10 kΩ Gate‑Pulldown | Schaltet Laser-Modul gegen GND, kein Vorwiderstand (Modul hat eigenen internen Treiber). Steuersignal `LASER_TX` (3,3 V Logik) vom Station‑ESP über USB‑C |
| **NeoPixel‑Anschluss vorn** | JST‑PH 3‑pol (5 V, GND, Daten‑in) | NeoPixel #1 (Schaft) per Kabelpeitsche – speist die Daisy‑Chain |
| **NeoPixel‑Anschluss hinten** | JST‑PH 3‑pol (5 V, GND, Daten‑out) | NeoPixel #3 (Knauf) per Kabelpeitsche – schließt die Daisy‑Chain |
| **IR‑LED‑Anschluss** | JST‑PH 2‑pol (Anode → R2 → 5 V, Kathode → Q1 Drain) | Geht zur Optik‑PCB im Optik‑Block |
| **Laser‑Anschluss** | JST‑PH 2‑pol (5 V via Q2, GND) | Geht direkt zum Lasermodul |
| **NEO_DATA‑Buffer** | **`74AHCT1G125` (1G Single‑Gate, SOT‑23‑5) + 100 nF** am Stab‑Eingang | Re‑Driver direkt vor der ersten LED (/OE→GND, A←NEO‑Daten, Y→0‑Ω‑Platzhalter→LED‑DIN). Regeneriert das über Kabel/ESD leicht verschliffene Signal auf saubere 5‑V‑Flanken. Begründung [`11-offene-punkte.md`](11-offene-punkte.md) § 3a. **Kein ESD‑Ersatz** |
| **ESD‑Schutz** | **kein eigenes Array auf der Wand** | Der ESD‑Schutz der Signal‑Adern sitzt auf der **Station** (D2/D3 = 2× USBLC6‑2SC6 am Stab‑Connector CN1, s. [`12-refactor-station-v2.md`](12-refactor-station-v2.md) § 8.11) – wie im funktionierenden Aufbau. Verwechslungsschutz entfällt (keine USB‑Buchse) |

> **Trigger‑Auswahl Gateron Green (Entscheidung 2026‑05‑17, Bestellung
> 2026‑05‑17):** Der ursprünglich vorgesehene 6×6 mm Mikrotaster
> (TS‑1187A/PB‑22E85L) ist die mechanisch schwächste Variante – 100 k–
> 500 k Klicks, Stoßlasten vom Daumen landen direkt auf den SMD‑Pads.
> Gateron Green (Cherry MX Green Klon) dagegen: 50 M Klicks, Stem
> läuft im Switch‑Body geführt (laterale Lasten bleiben im Body), klar
> definiertes „Klick‑Jacket"-Klick mit 80 ± 15 gf Aktivierung, Pretravel
> 2,3 mm / Total Travel 4,0 mm. Der genormte MX‑Plus‑Stem erlaubt jeden
> beliebig geformten Keycap (siehe Edelstein in §4.4). PCB‑Mount‑
> Variante mit 5 Pins (3 elektrisch + 2 mechanische Plugs) ersetzt eine
> separate Plate – die untere Halbschale plus Stand‑offs trägt das PCB
> direkt, der Switch sitzt spielfrei verkrampft auf dem PCB.
>
> **Gateron statt Cherry MX Original:** Cherry MX Green Original kostet
> ~1,20 € pro Stück + Versand (~25 € für 8 Stück bei Reichelt).
> Gateron Green ist mechanisch und elektrisch ein 1:1‑Klon, identische
> Pin‑Belegung und Stem‑Geometrie, identische Click‑Jacket‑Mechanik.
> Im Set bei DRAOZA via Amazon: **10,99 € für 36 Stück = 0,31 €/Stück**
> (Faktor 2,5 günstiger als Original). 50 M Cycles statt 100 M ist für
> unseren Halloween‑Use‑Case (~6 000 Klicks/Stab/Halloween über 10 Jahre
> = 60 000 Klicks insgesamt) ~830× überdimensioniert. Lebensdauer‑
> Argument ist hier irrelevant. QC‑Schwankungen im Sammelbeutel werden
> dadurch ausgeglichen, dass 36 Stück vorhanden sind und nur 4 + 4
> Reserve gebraucht werden – die besten 8 werden ausgewählt.

> **Halo statt Single‑NeoPixel (Entscheidung 2026‑05‑17):** Ein einzelner
> „in‑switch"-NeoPixel unter dem MX‑Stem ginge mit Cherry MX Green RGB
> (transparenter Body, ~3× teurer), würde aber nur eine punktförmige
> Lichtquelle bieten – der Edelstein‑Keycap zeigt dann einen Hotspot
> statt eines flächigen Glow‑Effekts. Stattdessen sitzen **4 SK6812 5050
> SMD‑Pixel** im Quadrat um den Switch herum (DI → 4 in Reihe → DO).
> Vorteile: 4× mehr Lichtleistung, flächige Ausleuchtung über
> Diffusion im Keycap, freie Switch‑Auswahl (Standard‑Green statt
> Green RGB), zusätzlich glüht eine umlaufende Lichtfuge in der
> oberen Halbschale um den Keycap herum. Strom: vollweiß ~80 mA für
> alle 4, im Halloween‑Animationsbetrieb deutlich darunter.

### 4.1.1 Schalter‑Topologie: warum MOSFETs im Stab, nicht in der Station

**Architektur‑Entscheidung 2026‑05‑14** nach Diskussion mit Tobias:

Die zwei N-MOSFETs für IR-LED und Laser sitzen **im Stab-PCB**, nicht im
Station-PCB. Der Station‑ESP gibt nur die 3,3 V-Steuersignale `IR_TX`
(GPIO 5, mit 38 kHz Träger‑Modulation) und `LASER_TX` (GPIO 7, DC an/aus)
über zwei Adern des USB‑C‑Kabels zum Stab. Im Stab schalten die BSS138
dann den lokal verfügbaren 5 V‑Strom auf die IR‑LED bzw. das Lasermodul.

| Aspekt | MOSFET in Station (verworfen) | MOSFET im Stab (gewählt) |
|---|---|---|
| Strom durchs USB‑C‑Kabel | 100 mA mit 38 kHz Modulation | < 1 mA Logiksignal |
| Spannungsabfall auf 1,5 m | ~30 mV → reduzierte IR‑Reichweite | irrelevant |
| EMV‑Abstrahlung Kabel | 38 kHz‑Träger + Oberschwingungen | wenig HF‑Strom |
| Schaltflanken | leicht verschmiert durch Kabel‑C (~75 pF) | sauber, direkt am MOSFET |
| Bauteilanzahl Station‑PCB | 2× MOSFET + Pulldown + Series‑R + Vorwiderstand | gespart |
| Bauteilanzahl Stab‑PCB | nur passive Komponenten | +2 MOSFETs (s. Tabelle 4.1) |

Die zwei MOSFETs im Stab werden mit den am Lochraster für die Station
verifizierten Werten beschaltet (Doc 14 § 2.4 + 2.5):

- **820 Ω Series‑R** am Gate (Strombegrenzung beim Schalten, dämpft Ringing)
- **10 kΩ Pulldown** zwischen Gate und GND (sichert „aus" beim Boot)
- IR‑Vorwiderstand **33 Ω 0,5 W** (~100 mA durch TSAL6100 @ 5 V)

### 4.2 ESD‑Schutz der Signal‑Adern (Verwechslungsschutz entfällt)

> **Geändert 2026‑06‑08:** Mit dem Wegfall der USB‑C‑Buchse gibt es **keinen
> Verwechslungsschutz‑Fall mehr** – in eine grüne Schraubklemme bzw. ein
> angelötetes Kabel kann niemand ein PD‑Ladegerät stecken. Die alte
> PD‑Klemmung und die „INFINITAG STAB ONLY"‑Markierung sind hinfällig.

Was bleibt, ist reiner **ESD‑Schutz**: Über das 1,5‑m‑Kabel zur
kinderhand‑gehaltenen Wand kann statische Entladung in die Signal‑Adern
(und damit in die Station‑ESP‑GPIOs) einkoppeln.

**Maßnahme – auf der Station, nicht auf der Wand:** Der ESD‑Schutz sitzt
(wie im funktionierenden Aufbau, dort das alte `USBLC6‑4SC6Y`) auf dem
**Station‑Data‑Board am Stab‑Connector CN1**: **2× `USBLC6‑2SC6`** (D1 + D2),
Pass‑Through inline, VBUS an +5 V, GND an GND; zusätzlich begrenzen die
Serien‑Widerstände R1–R4 (100 Ω) dort den ESD‑Strom in die GPIOs. Voll
dokumentiert in [`12-refactor-station-v2.md`](12-refactor-station-v2.md)
§ 8.11. **Die Wand‑PCB bekommt kein eigenes ESD‑Array.**

> **Verfügbarkeit / Geschwindigkeit:** Der `USBLC6‑2SC6` wurde wegen guter
> Verfügbarkeit in DE / auf AliExpress gewählt (das ursprünglich geplante
> `SP3012‑04UTG` war schlecht zu bekommen). Die ultraniedrige USB‑Kapazität
> ist für unsere langsamen Signale (NeoPixel ~800 kHz, IR 38 kHz, Rest DC)
> irrelevant – jedes 5‑V‑TVS‑Array täte es.

### 4.3 PCB‑Form: kompaktes Rechteck 62 × 24 mm (Entscheidung 2026‑05‑17)

**Architektur‑Entscheidung 2026‑05‑17 (Tobias):** Das Haupt‑PCB wird
auf eine **kompakte rechteckige Bauform reduziert** (62 × 24 mm),
zentriert unter dem Trigger‑Bereich. Vorgängerplan war eine T‑Form
über die ganze Griff‑Länge (130 × 16/22 mm), aber die ist mit der
4‑LED‑Halo‑Anordnung um den MX‑Switch und der festen USB‑C‑Buchse
nicht nötig – alle aktiven Bauteile passen in 62 × 24 mm.

```
   Vorderseite (Optik)                  Hinterseite (Knauf)
   ┌────────────────────────────────────────────────────────┐
   │ ⊙M2                                              ⊙M2  │
   │                                                        │
   │ [CN1 IR]   [Q1, R2, R3, R4]                    [USB‑C │
   │ [CN2 Laser][Q2, R5, R6]    [LED1‑4 Halo + SW1]  Buchse│
   │                            [C1‑C4 Bypass-Caps]  + C5,C6│
   │                                                        │
   │ ⊙M2                                              ⊙M2  │
   └────────────────────────────────────────────────────────┘
   ◄──────────────── 62 mm ────────────────────────────►
                                                       ▲
                                                       │ 24 mm
                                                       ▼
```

**Layout‑Eckdaten:**

| Maß | Wert | Begründung |
|---|---|---|
| PCB‑Außenmaß | **62 × 24 mm rechteckig** | passt locker in den 32 mm Innen‑Ø des Griffs |
| PCB‑Position im Stab | **unter dem Trigger zentriert**, mittig im Griff | Stab‑Position ~150 mm vom Vorderende |
| PCB‑Dicke | 1,6 mm | Standard JLCPCB |
| Befestigung | **4 × M2‑Bohrungen** in den 4 Ecken | M3 wäre zu groß bei 24 mm Breite; M2 reicht mechanisch völlig |
| Layer | 2‑Layer, FR4, HASL Verzinnung | günstigste Standard‑Bestellung bei JLCPCB |
| BOM‑Werteinheit | **PCB‑Mount‑MX (THT mit 5 Pins)** + THT für JST/USB‑C + **SMD für 4× SK6812 SIDE‑A 4020 Halo** (stehend montiert) | MX wird durchgesteckt und von unten gelötet, Halo‑Pixel stehen senkrecht für direktes Anleuchten des Edelstein‑Keycaps |
| Kosten‑Abschätzung | ~3 € pro Stück bei 5 Stück MOQ (kompaktere Form, weniger Material) | JLCPCB Sammelposition |

**Vorteile der kompakten Bauform:**

1. **Weniger Material**, niedrigere PCB‑Kosten
2. **Mechanisch stabiler** im Trigger‑Bereich – kleinere PCB‑Fläche ist
   weniger biege‑empfindlich bei Daumendruck
3. **Mehr freier Stab‑Innenraum** – Optik‑Kabel und USB4‑Kabel können
   ungehindert durch den vorderen bzw. hinteren Griff‑Bereich laufen
4. **Bauteil‑Anordnung dichter**, kürzere Tracks im Layout
5. **Halbschalen‑Stand‑offs nur an einer Position nötig** (mittig im
   Griff statt verteilt)

**PCB‑Farben (Tobias‑Entscheidung 2026‑05‑17 bei der Bestellung):**

| PCB | Lötmaske‑Farbe | Begründung |
|---|---|---|
| **Wand‑Haupt‑PCB** | **Weiß** | Weiße Lötmaske **reflektiert das Halo‑NeoPixel‑Licht** zurück nach oben Richtung Edelstein‑Keycap → verstärkt den Glow‑Effekt um Faktor 1,3–1,5×. Schwarze Lötmaske würde ~30 % des Halo‑Lichts absorbieren. |
| **Optik‑PCB** | **Schwarz** | Schwarze Lötmaske **absorbiert Streulicht** der IR‑LED → kein reflektiertes Licht innerhalb des Optik‑Blocks, cleaner Strahl durch die Linse. Weniger Stör‑Reflexion zum TSOP‑Empfänger der Targets. |

**Footprint‑Strategie für SK6812 SIDE‑A 4020 (Tobias‑Entscheidung 2026‑05‑17):**

Statt eines Custom‑Footprints für stehende Montage wurde der
**Standard‑4020‑Footprint (liegend, Pads an Längsseite) verwendet,
aber mit erweitertem Pad‑Abstand**, sodass die LEDs sowohl liegend
als auch stehend montierbar sind. Final‑Entscheidung wird nach der
LED‑Lieferung getroffen:

| Plan A: liegend | Plan B: stehend |
|---|---|
| LED flach auf die Pads gelegt, Standard‑SMD‑Handlöt | LED senkrecht gestellt, castellated Pads über kurze Drahtbrücken (~0,3 mm Cu‑Draht) zu den breiten PCB‑Pads gebrückt |
| Halo strahlt seitlich nach außen → Glow über Halbschalen‑Lichtfuge | Halo strahlt direkt nach oben → Edelstein‑Direkt‑Beleuchtung + Halo |
| Wenig direkter Edelstein‑Glow | Voller Edelstein‑Glow + Halo |

Bei der LED‑Lieferung (DRAOZA / Wushi Optoelectronics 100er‑Pack,
bestellt 2026‑05‑17) wird unter Lupe die castellated‑Pad‑Geometrie
verifiziert → entweder liegend (einfach) oder stehend (besser für
Edelstein‑Glow) bestückt.

**Bestellung 2026‑05‑17 (JLCPCB):**

| Posten | Spec | Stück | Preis |
|---|---|---|---|
| Wand‑Haupt‑PCB | 62 × 24 mm, **weiß**, 1,6 mm, HASL with Lead | 5 | $2,10 (Special Offer) |
| Optik‑PCB | 10 × 14 mm, **schwarz**, 1,6 mm, HASL with Lead | 5 | $4,20 |
| **Subtotal PCB** | | | **$6,30** |
| Versand | DHL Express | – | ~$15 |
| **Total** | | | **~$21** |

**HASL with Lead statt bleifrei (Tobias‑Entscheidung):** Verbleites Lot
spart $13/Board × 2 PCBs = $26 gegenüber lead‑free. Für ein privates
Halloween‑Projekt im Vorgarten (kein Verkauf an Dritte) ist RoHS
irrelevant; verbleites Lot lötet sich zudem leichter bei Hand‑
Bestückung. Build‑Time bei JLCPCB: 3 Tage. Erwartete Lieferung mit
DHL Express: 5–7 Tage gesamt.

**Daisy‑Chain im Stab:**

```
USB‑C(NEO_DATA) ──► R1 (470 Ω) ──► NEO_DATA_5V_F
                                       │
                                       ▼
                              LED1 → LED2 → LED3 → LED4 (alle onboard, 4× SK6812 SIDE‑A)
                                       │
                                       ▼
                              LED4 DOUT → offenes Ende (NC)

(Externe NeoPixel #1 + #3 sind in V1 weggelassen, siehe §1 Status‑Tabelle und §11 Punkt 32 – nachrüstbar in V2)
```

**NeoPixel‑Daisy‑Chain im Stab (6 LEDs gesamt, Halo zählt als 4 in Reihe):**

```
NeoPixel‑Pin von Station ──► NeoPixel #1 (Schaft, JST‑PH 3‑pol)
                                     │
                                     ▼
                             NeoPixel #2a (onboard SMD, Halo NW)
                                     │
                                     ▼
                             NeoPixel #2b (onboard SMD, Halo NE)
                                     │
                                     ▼
                             NeoPixel #2c (onboard SMD, Halo SW)
                                     │
                                     ▼
                             NeoPixel #2d (onboard SMD, Halo SE)
                                     │
                                     ▼
                             NeoPixel #3 (Knauf, JST‑PH 3‑pol) ──► (offenes Ende)
```

Die 4 Halo‑NeoPixel sind **direkt auf dem Wand‑Haupt‑PCB bestückt**
(SMD‑SK6812 5050), diagonal um den Cherry MX im Quadrat (~7 mm vom
Stem‑Zentrum). Kein eigenes Sub‑PCB nötig – Tobias‑Entscheidung
2026‑05‑17: maximal 2 PCBs im Stab (Haupt + Optik), kein Trigger‑Sub.

NeoPixel #1 und #3 sind über JST‑PH 3‑pol angeschlossen, sitzen
positionsgenau im Stab‑Gehäuse an Schaft‑Übergang bzw. kurz vor dem
Knauf.

| Maß | Wert | Begründung |
|---|---|---|
| PCB‑Außenmaß | T‑Form, 130 mm Länge, 16 mm Streifenbreite, 22 mm Trigger‑Verbreiterung über ~25 mm Länge | passt durch Innen‑Ø ~32 mm im Griff, Trigger‑Zone lokal aufgeweitet |
| PCB‑Dicke | 1,6 mm | Standard JLCPCB |
| Befestigung | 3 × M3‑Bohrungen (vorn, mitte hinter Trigger, hinten) + 2 × M2‑Bohrungen links/rechts vom Switch | nimmt Daumen‑Druck (70 cN Aktivierung) ohne PCB‑Biegung auf |
| Layer | 2‑Layer, FR4, HASL Verzinnung | günstigste Standard‑Bestellung bei JLCPCB |
| BOM‑Werteinheit | **PCB‑Mount‑MX (THT mit 5 Pins)** + THT für Treiber + **SMD für 4× SK6812 5050 Halo** | MX wird durchgesteckt und von unten gelötet, Halo‑Pixel präzise positioniert |
| Kosten‑Abschätzung | ~6 € pro Stück bei 5 Stück MOQ (mehr Fläche durch Verbreiterung als 16 mm‑Streifen, < 8 €) | JLCPCB Sammelposition |

### 4.4 Trigger: Gateron Green mit Edelstein‑Keycap

**Position:** Daumen‑Trigger oben am Griff, ~Position 75 mm vom Griff‑
Vorderende (= ~Stab‑Position 165 mm vom Vorderende, mittig im Griff).
Der Spieler hält den Stab in der Faust, der Daumen liegt natürlich
auf der Oberseite. Gateron Green Switch (Cherry MX Green Klon) sitzt
PCB‑Mount aufgelötet auf dem Haupt‑PCB, der Stem ragt durch eine
14 × 14 mm Aussparung in der oberen Halbschale, darauf gesteckt ein
**Edelstein‑förmiger Keycap** in transparentem PETG.

**Haptik (Gateron‑Green‑Specs laut Listing, bestellt 2026‑05‑17):**

| Parameter | Gateron Green | Bedeutung im Spiel |
|---|---|---|
| Aktivierungskraft | 80 ± 15 gf (~ 80 cN) | bewusster Druck nötig, nicht versehentlich auslösbar |
| Tactile Force (Bump) | ~ 85 cN (typ., baugleich Cherry‑MX‑Green) | klar fühlbarer Druckpunkt |
| Klick‑Mechanik | Click‑Jacket („knackiger Klang") | akustisch + taktil, „satter" Klick |
| Pretravel (Hub bis Aktivierung) | 2,3 ± 0,6 mm | merklicher Druckweg vor Auslösen |
| Total Travel (Hub bis Anschlag) | 4,0 mm | spürbarer Anschlag am Switch‑Boden |
| Bounce Time | 5 ms max (bei 16 in/sek) | Software‑Debounce auf 10–15 ms im Station‑ESP eingeplant |
| Lebensdauer | 50 M Klicks | bei ~6 000 Klicks/Stab/Halloween × 10 Jahre ~830× überdimensioniert |
| Pin‑Layout | 5‑Pin PCB‑Mount, 100 % MX‑kompatibel | gleicher Footprint wie Cherry MX Original |
| Stem‑Profil | MX‑Plus (4 × 1,2 mm Pluskreuz) | beliebiger Keycap (siehe Edelstein) passt drauf |

**„Kinder‑Variante" Fallback:** ursprünglich war Cherry MX Blue als
Fallback (50 cN Aktivierung) für Kinder eingeplant. Tobias‑
Entscheidung 2026‑05‑17: erst mit Gateron Green probetragen und nach
realer Reaktion entscheiden, ob Blue gebraucht wird. Bei 80 gf
Aktivierung ist Green eher zu‑genau‑richtig als zu‑schwer für einen
„Schuss"-Trigger. Falls ein Kinder‑Test zeigt, dass die 80 gf zu viel
sind, wird ein 4er‑Set Cherry MX Blue oder Gateron Blue (~3 €) als
Folge‑Bestellung beschafft – mit derselben PCB‑Footprint kompatibel.

**Edelstein‑Keycap (transparenter PETG‑Druck):**

Der Cherry‑MX‑Standard‑Plus‑Stem (4 × 1,2 mm Pluskreuz) erlaubt einen
völlig frei geformten Keycap. Wir nutzen das für einen **Magic‑Wand‑
Look** statt eines Tastatur‑Keycaps – ein hexagonal‑gefacetteter
Edelstein, der von der Halo‑LED durchleuchtet glüht wie ein Saphir.

| Element | Maß / Material | Hinweis |
|---|---|---|
| Außenform | hexagonal‑gefacettet, 6 Seiten | klassische „Saphir/Quarz"-Anmutung |
| Breite über Ecken | 14 mm | bündig zum MX‑Switch‑Footprint |
| Breite über Flächen | 12 mm | gefacettete Verjüngung nach oben |
| Höhe gesamt (über Stem‑Boden) | 8 mm | hält Stack‑Höhe unter Halbschalen‑Innenmaß |
| Höhe „Spitze" über Top‑Plateau | 1,5 mm | abgerundet, daumen‑sicher |
| Wandstärke | 1,2 mm umlaufend | gleichmäßige Lichtdiffusion, keine Hotspots |
| Stem‑Aussparung unten | 4 × 4 mm Plus‑Profil, 4 mm tief, +0,1 mm Spiel | Standard MX, Bambu druckt das bei 0,12 mm Layer sauber |
| Material | transparentes PETG (Bambu Basic Clear oder Polymaker PolyLite Clear) | UV‑stabil, formstabil, klar |
| Optional Multi‑Material | TPU 95A Kappe oben (0,5 mm) | griffiger Daumenkontakt, Kratzschutz |
| Tönung optional | leichte Färbung möglich (Filament‑Mix im AMS oder After‑Print IPA + Acryl‑Tinte) | „Hexen‑Stab"-Look mit blass‑violett |

**Mounting‑Konzept (Tastatur‑Stil, Tobias‑Entscheidung 2026‑05‑17):**

- Haupt‑PCB sitzt auf 3 × M3 + 2 × M2 Stand‑offs in der **unteren
  Halbschale**. Die Stand‑offs links und rechts vom Switch sind
  entscheidend – sie nehmen den Daumen‑Druck (~80 cN) direkt auf, das
  PCB darf an der Switch‑Position nicht durchbiegen.
- Gateron Green wird mit 5 Pins (PCB‑Mount‑Variante) durch das PCB
  durchgesteckt und von unten gelötet (3 elektrisch + 2 mechanische
  Kunststoff‑Plugs).
- Die **obere Halbschale** hat eine 14 × 14 mm Aussparung, durch die
  der Switch‑Stem nach oben ragt. Die Halbschale übernimmt **keine
  tragende Funktion** für den Switch – kein Plate‑Druck nötig.
- Darauf der Edelstein‑Keycap, gesteckt auf den MX‑Stem.

**Bauhöhen‑Stack:**

| Schicht | Höhe |
|---|---|
| Stand‑off‑Höhe (Sockel auf unterer Halbschalen‑Innenwand) | 3 mm |
| Haupt‑PCB | 1,6 mm |
| Gateron Green Body über PCB‑Oberkante (MX‑Standard) | 11,6 mm |
| Stem‑Auszug (volle Auslenkung, unbedrückt) | 4 mm |
| Edelstein‑Keycap über Stem‑Boden | 8 mm |
| **Σ bis Keycap‑Oberkante (unbedrückt)** | **28,2 mm** |
| **Σ bis Switch‑Top (= Innenseite obere Halbschale)** | **16,2 mm** |
| Verfügbare Innen‑Höhe (Griff Ø 38 mm, Wandstärke 3 mm) | 19 mm |
| **Reserve** | **2,8 mm** |
| Keycap‑Auszug aus Halbschale (unbedrückt) | ~10 mm |
| Keycap‑Auszug bei vollem Druck (Stem 4 mm + 8 mm Keycap − Halbschale) | ~6 mm |

→ Passt. Optional kann der Griff im Trigger‑Bereich lokal auf
Ø 42–44 mm aufgedickt werden („Daumen‑Mulde"), das macht die
Reserve größer und gibt eine spürbare Auflage für den Daumen.

**Halo‑Beleuchtung (Lichtkonzept):**

Direkt um den Gateron‑Green‑Switch herum sitzen **4 SK6812 5050 NeoPixel**
im Quadrat (~7 mm vom Stem‑Zentrum), auf der Haupt‑PCB. Die obere
Halbschale hat um die 14 × 14 mm Switch‑Aussparung herum eine **~3 mm
breite, 1 mm dünne umlaufende Lichtfuge** (Höhenabsatz in der
Wandung) → wirkt als Lichtleiter und glüht bei aktivierter Halo als
quadratischer Ring um den Keycap.

Effekt im Spiel:

- Im Standby pulst der Edelstein in Stations‑Farbe (z. B. Cyan), die
  Lichtfuge spiegelt die gleiche Farbe → der Stab signalisiert
  „bereit".
- Beim Schuss blitzt Edelstein + Lichtfuge gleichzeitig weiß auf.
- Im Setup‑Modus beide lila.
- Bei Niedrig‑Spannung beide rot pulsierend.

> **Warum nicht „in‑switch"-NeoPixel:** MX‑Switches mit transparentem
> Body (RGB‑Varianten) gäbe es zwar (Cherry MX Green RGB, Gateron RGB)
> und sie erlauben eine einzelne SMD‑LED unter dem Stem. Der Edelstein
> würde dann aber mit einem zentralen Hotspot leuchten statt flächig
> zu glühen. Die Halo‑Lösung mit 4 LEDs außen herum ist deutlich
> heller und gleichmäßiger.

> **Warum kein Sub‑PCB für den Trigger:** in einer früheren
> Diskussionsrunde war eine eigene Trigger‑Sub‑PCB mit Kailh
> Hot‑Swap‑Sockel + 4 LEDs angedacht. Tobias‑Entscheidung 2026‑05‑17:
> **2 PCBs reichen** (Haupt + Optik). Der Gateron‑Green wird direkt
> aufgelötet, kein Hot‑Swap. Ein zusätzliches Sub‑PCB würde
> JST‑Stecker und Kabel in den ohnehin kompakten Griff einbringen,
> ohne nennenswerten Wartbarkeits‑Gewinn (Switch wird selten
> getauscht, Keycap ist separat tauschbar).

### 4.5 Active‑LOW‑Verdrahtung

Trigger wird zwischen GND und dem Trigger‑Pin der USB‑C‑Buchse
geschaltet. Kein Pull‑down‑Widerstand auf der Wand‑PCB – die Station
zieht ihren Eingang per `INPUT_PULLUP` auf 3,3 V hoch und liest den
Trigger als invertiertes Signal aus.

```
USB‑C Trigger‑Pin ──────── Trigger‑Schalter ──── GND
                              ↓
                        gedrückt = LOW
                        offen   = HIGH (Pullup in Station)
```

R1 (10 kΩ Pull‑down von V2.1) entfällt vollständig. Die 3,3‑V‑Ader
im Kabel entfällt vollständig. Kabel hat 6 Adern statt vorher 7.

---

## 5. Mechanik: Stab‑Gehäuse V3

### 5.1 Form‑Konzept: Zauberstab in 3 Zonen

```
   ┌─ Spitze ─┐    ┌────── Schaft ──────┐    ┌─── Griff ───┐    ┌─ Knauf ─┐
   │          │    │                    │    │             │    │         │
   │   Optik  │    │  NeoPixel Zone     │    │  PCB +      │    │ Kabel‑  │
   │   Block  │    │                    │    │  Trigger    │    │ Austritt│
   │          │    │                    │    │             │    │         │
   ├──Ø30────►├────►Konus 30→38────────►├────►Ø38─────────►├────►Ø35─────►│
   │          │    │                    │    │             │    │         │
   │   50 mm  │    │      40 mm         │    │   150 mm    │    │  50 mm  │
   └──────────┘    └────────────────────┘    └─────────────┘    └─────────┘

           Gesamtlänge ~290 mm
```

| Zone | Außen‑Ø | Länge | Inhalt |
|---|---|---|---|
| **Spitze** (vorn) | 30 mm rund außen, **innen konischer Schutz‑Trichter** (Ø 14 mm Front → Ø 26 mm Linsen‑Sitz) | 50 mm | 15 mm Schutz‑Trichter + Optik‑Block (28 × 28 × 35 mm), Linse **15 mm hinter Stab‑Frontkante** versetzt (siehe §3.4.1) |
| **Schaft** | 30 mm → 38 mm konisch | 40 mm | Übergang, freier Innenraum (JST‑Kabel zur Optik laufen hier durch); externe NeoPixel #1 in V1 nicht bestückt |
| **Griff** | 38 mm zentral, 35 mm Taille mittig, optional lokal 42–44 mm im Trigger‑Bereich („Daumen‑Mulde") | 150 mm | **kompaktes Haupt‑PCB 62 × 24 mm zentriert unter dem Trigger** (siehe §4.3), Cherry MX + Edelstein‑Keycap oben, 4× Halo‑NeoPixel onboard. Restlicher Griff‑Innenraum frei – Kabel laufen ungehindert |
| **Knauf** | 35 mm | 50 mm | **Kabel‑Durchführung mit Klemmbacken‑Zugentlastung** (siehe §5.5); kein externer USB‑C‑Stecker mehr, das Kabel ist intern am PCB eingesteckt |

**Detail Spitzen‑Geometrie (Entscheidung 2026‑05‑17):**

```
   Querschnitt durch die Spitze (vereinfacht):

   Stab‑Front (außen Ø 30 mm)          Linse OM3 (Ø 16,5 mm)
        │                                │
        ▼                                ▼
   ┌────┐                              ┌─┐
   │    │\                             │ │
   │    │ \                            │ │   Innen‑Trichter:
   │    │  \   konisch                 │ │   konisch aufweitend
   │    │   \  aufweitend              │ │
   │    │    \                         │ │
   │    │     \                     ┌──┤ │
   │    │ Ø14  \                    │  │ │   Optik‑Block
   │    │ Front \                   │  │ │   (Druckteil B)
   │    │ Öffng. \                  │  │ │
   │    │         \                 │  │ │
   │    │          ╲                │  ▼ │
   │    │ Halbschale╲────────────►──┤    │
   │    │ (Druckteil A)╲            │ Linsen‑
   │    │ 4 mm Wand    ╲            │ aufnahme +
   │    │ im Trichter   ╲           │ Frontplatte
   │    │                ╲          │    │
   │    │                 ╲────────┤     │
   │    │                          │     │
   │    │ ◄── 15 mm Schutz‑       │     │
   │    │     Distanz ──►          │     │
   └────┘                          └─────┘
```

- **Konischer Innen‑Trichter** mit Front‑Öffnung Ø 14 mm, aufweitend
  bis Ø 26 mm an der Linsen‑Frontplatten‑Position. Der Strahl wird
  nicht beschnitten (siehe Strahl‑Mathematik in §3.4.1).
- **Wandstärke im Trichter‑Bereich: 4 mm** (statt 3 mm regulär in der
  Halbschale). Bewusste Knautschzone für Front‑Stoß.
- **Lastpfad bei Front‑Stoß:** vom Trichter‑Wulst in die
  Halbschalen‑Vorderkante → über die Halbschalen‑Verschraubung in den
  Stab‑Körper. **Nicht** in die Linsen‑Frontplatte und ihre
  4 M2‑Schrauben.
- Außen abgerundete Kontur (Radius ~3 mm an der Frontkante) → keine
  scharfe Spitze, die beim Tragen verletzen oder hängenbleiben könnte.
- **Optionaler TPU‑Schutzring** (Innen‑Ø 30 mm, Außen‑Ø 34 mm,
  Länge 15 mm) kann nachträglich aufgezogen werden – für die
  Erstauflage 2026 nicht bestückt (siehe §3.4.1).

**Begründung der Maße:**

- **Ø 30–38 mm Griff** liegt im Komfortbereich für Kinder ab 6 Jahren
  (Hand‑Spannweite ~60 mm) **und** Erwachsene (passt locker in eine
  geschlossene Faust). Filigrane Stäbe wie aus Harry‑Potter (Ø 15–20 mm)
  sind für Kinderhände schwer zu greifen und brechen leichter.
- **Taille auf 35 mm in der Mitte des Griffs:** ergonomisch wahrnehmbar
  („das ist die Mitte"), gibt der Hand Halt und verhindert, dass der
  Stab durch die Faust rutscht.
- **Spitze nicht spitz:** flach abgerundet, Linse Ø 16,5 mm sitzt
  zentral. Stoßfestigkeit > Optik – wenn der Stab fällt, soll die
  Wand‑Außenkontur den Aufprall fangen, nicht die Linse.
- **50 mm Knauf** mit **fester Kabel‑Durchführung axial nach hinten**
  (Entscheidung 2026‑05‑17). Das USB4‑Kabel ist intern an der USB‑C‑
  Buchse auf dem PCB eingesteckt; nur das Kabel selbst tritt aus dem
  Knauf. Eine 3D‑gedruckte Klemmbacken‑Zugentlastung im Knauf
  überträgt Zugkräfte aufs Gehäuse, nicht auf den Stecker (siehe §5.5).
  Kein externer USB‑C‑Stecker mehr → manipulationssicher gegenüber
  Kindern, die das Kabel nicht versehentlich abziehen können.

### 5.2 Aufbau: Halbschalen längs

**Das Gehäuse besteht aus zwei Längs‑Halbschalen, geschraubt mit
M3‑Inserts.** Begründung:

1. **Druckbarkeit:** Halbschalen liegen auf der Bauplatte, Layer‑Linien
   parallel zur Stab‑Achse. Stoßlast längs zur Achse trifft nicht die
   schwächste Z‑Anhaftungs‑Richtung.
2. **Wartbarkeit:** Halbschale auf, alles drin sichtbar, jedes Modul
   tauschbar. Keine Hot‑Swaps, aber definitiv „Lötkolben + 4 Schrauben"‑
   tauschbar.
3. **Stoßfestigkeit:** PETG‑Halbschalen mit 4 mm Wand sind zäh. Bei
   Aufprall geben sie elastisch nach, die innenliegenden Module
   bekommen den Schock nicht voll ab.

**Halbschalen‑Trennung:**

- **Trennebene:** durch die Längsachse, leicht versetzt damit die
  Trennfuge nicht direkt auf dem Trigger oder der USB‑C‑Buchse liegt.
- **Schraubenanzahl:** 6 × M3 × 12 mm Schrauben in M3‑Short‑Inserts
  über die Länge (vorne, mittig, hinten und drei zwischendurch).
- **Dichtung:** umlaufender 1 mm Schaumstoff‑Streifen oder 3D‑gedruckte
  Lippe in der Trennfuge → Spritzwasser‑Schutz für Outdoor.
- **Inserts:** Ruthex M3 Short (4 mm) in einer Halbschale, Schrauben
  von der anderen.

### 5.3 Druck‑Strategie auf Bambu H2D (einteilig pro Halbschale)

**Tobias‑Drucker:** Bambu Lab H2D (Bauraum 350 × 320 × 325 mm,
Dual‑Extruder mit AMS, voll PLA‑Stützmaterial‑fähig).

→ **Stab wird einteilig pro Halbschale gedruckt** – keine Mitten‑
Trennstelle. Maximaler mechanischer Halt, weniger Toleranzen,
schneller zu montieren.

| Druckteil | Maße | Druck‑Orientierung |
|---|---|---|
| Halbschale Oberseite | 290 × 38 × 19 mm (halbe Stab‑Höhe) + Art‑Nouveau‑Reliefs an 3 Slot‑Positionen (siehe §5.8) | flach auf der Bauplatte, Innenseite nach oben (Reliefs werden mit gedruckt, da auf der Außenseite) |
| Halbschale Unterseite | 290 × 38 × 19 mm + spiegelbildliche Reliefs an denselben 3 Slot‑Positionen | flach auf der Bauplatte, Innenseite nach oben |
| Optik‑Block | 28 × 28 × 35 mm | Front (Linsen‑Seite) auf Bauplatte |
| Linsen‑Frontplatte | Ø 28 × 2 mm | flach auf Bauplatte |
| **Edelstein‑Keycap** (transparent) | hexagonal Ø 14 mm × 8 mm hoch | flach auf Bauplatte, Stem‑Öffnung nach oben (sauberere Facetten) |
| Knickschutz‑Tülle (TPU, optional) | Ø 8 × 30 mm | aufrecht stehend, mit PLA‑Innenstütze (Dual‑Extruder) |
| TPU‑Manschette Schaft (optional) | Ø 30 → 35 × 60 mm konisch | aufrecht stehend, PLA‑Innenstütze |
| TPU‑Schutzring Spitze (optional, nicht in V1) | Ø 30 innen / 34 außen × 15 mm | aufrecht stehend, PLA‑Innenstütze |

**Multi‑Material‑Verwendung (Bambu H2D):**

| Material | Verwendung |
|---|---|
| **PETG opak** (Halloween‑Farbe) | Halbschalen, Optik‑Block, Linsen‑Frontplatte |
| **PETG transparent** | **Edelstein‑Keycap** (durchleuchteter Trigger) |
| **PLA** (zweiter Extruder) | **Stützstruktur für Innenkanäle** – z. B. die Kabelkanäle und der 90°‑Knauf‑Bogen brauchen Überhang‑Stützen, die nach dem Druck herausgebrochen werden. PLA löst sich besser von PETG als PETG‑Stütz‑PETG |
| **TPU 95A** (optional) | Knickschutz‑Tülle, Schaft‑Manschette, Spitzen‑Schutzring; optionale 0,5 mm‑Kappe auf dem Edelstein‑Keycap für griffigen Daumen |

> **Vorteil PLA als Stützstruktur:** PETG haftet auf PETG sehr stark
> (chemische Verbindung beim Schmelzen) → Stütz‑Bruchstellen sind
> schwer zu entfernen. PLA hat eine niedrigere Schmelztemperatur und
> eine andere Polymer‑Familie, **klebt nicht an PETG fest** und löst
> sich beim Bruch sauber. Standardpraxis für hochwertige Drucke auf
> Dual‑Extruder‑Maschinen.

### 5.4 Inneres Layout

```
   Längsschnitt durch eine Halbschale, von vorn nach hinten:
   (Stand 2026‑05‑17 – kompakte PCB, festes USB4‑Kabel)

   ┌─ Spitze ─┬─ Schaft ──┬───── Griff ──────┬──── Knauf ──────────────┐
   │ ┌──Optik──┐          │                  │                          │
   │ │ Block + │  freier  │  ┌──Haupt‑PCB──┐ │                          │
   │ │ Linse + │  Innen‑  │  │  62 × 24 mm │ │  Kabel‑Klemmung          │
   │ │ Laser + │  raum,   │  │ Trigger‑zen │ │  (3D‑Druck +             │
   │ │ Optik‑  │  IR/Lsr‑ │  │  triert     │ │   M3‑Schraube)           │
   │ │ PCB     │  Kabel   │  │  4× M2 Stand│ │     │                    │
   │ │         │  laufen  │  │  ‑offs      │ │     │                    │
   │ └─────────┘  hier    │  └─────────────┘ │     │                    │
   │     │        durch   │       ↑          │     │                    │
   │     │                │     SW1 + Halo   │     ▼                    │
   │     │                │     + Edelstein  │  ════════════►  Kabel    │
   │     │                │     Keycap       │   (USB4 ummantelt,       │
   │     │  JST‑Kabel ────┼────►USB‑C‑Buchse │    fest installiert)     │
   │     │  (IR + Laser)  │     auf PCB      │                          │
   │     │                │                  │                          │
   │  Optik‑Block fixiert │  Halbschalen     │  Knauf‑Innen:            │
   │  mit M3‑Inserts in   │  M3‑Inserts (6×) │  - M3‑Insert für         │
   │  beiden Halbschalen, │  in Trennfuge    │    Klemmbacken           │
   │  15 mm hinter        │                  │  - Kabel‑Durchführungs‑  │
   │  Stab‑Front          │                  │    loch ~6 mm            │
   └──────────────────────┴──────────────────┴──────────────────────────┘
        50 mm                                            50 mm
                  40 mm Schaft     150 mm Griff
   ◄────────────────── Gesamtlänge ~290 mm ───────────────────────────►
```

**NeoPixel‑Anordnung (6 LEDs in Daisy‑Chain, Stand 2026‑05‑17):**

- **NeoPixel #1** sitzt am Übergang Schaft → Griff (~Position 90 mm
  vom Vorderende). Sichtbar durch ein Klarsicht‑Inlay (gedrucktes PETG
  transparent) in der **oberen** Halbschale. Verbindung über JST‑PH
  3‑pol Kabelpeitsche zum Haupt‑PCB.
- **NeoPixel #2a–d** sitzen **direkt auf dem Wand‑Haupt‑PCB als 4× SMD‑
  SK6812 5050** im Quadrat um den Cherry MX (~Position 165 mm vom
  Vorderende, ~7 mm vom Stem‑Zentrum). Beleuchten den **Edelstein‑
  Keycap** flächig und glühen über die Lichtfuge in der oberen
  Halbschale als quadratischer Halo um den Switch (siehe §4.4).
- **NeoPixel #3** sitzt am hinteren Ende des Griffs (~Position 240 mm
  vom Vorderende), kurz vor dem Knauf. Sichtbar durch Klarsicht‑Inlay
  in der **oberen** Halbschale. Verbindung über JST‑PH 3‑pol Kabelpeitsche.

```
Stab‑Längsschnitt (vereinfacht, nur NeoPixel‑Positionen):

                   NeoPixel #1   NeoPixel #2a–d        NeoPixel #3
                   (Klarsicht)   (Halo um MX,          (Klarsicht)
                                  Edelstein‑Keycap)
                       ▼              ▼▼▼▼               ▼
[Spitze] [Schaft ──────●───── Griff ──●●●●─── Griff ─────●── Knauf]
                     90 mm           165 mm            240 mm
                     (JST)         (PCB onboard SMD)    (JST)
```

**Klarsicht‑Inlays an Position #1 und #3:**

- Bohrung Ø 8 mm in der oberen Halbschale, mit innen umlaufender
  Anschlag‑Lippe.
- Inlay aus transparentem PETG (Multi‑Material‑Druck oder separat
  einlegen), Ø 8,2 mm × 4 mm dick, leicht aufgeraut für Diffusor‑Effekt.
- Inlay sitzt fest in der Bohrung durch leichten Press‑fit oder mit
  einem Tropfen Sekundenkleber von innen fixiert.

**Optionale Erweiterung:** ein **NeoPixel‑Ring** (8 oder 12 LEDs)
direkt vorn um die Linse herum als „Magie‑Effekt". Würde den
Optik‑Block aufwendiger machen und die Daisy‑Chain auf 11–15 LEDs
verlängern (für die Station‑V2 trivial machbar). Erst V4‑Thema – auf
der Liste in Abschnitt 11.

### 5.5 Knauf: Kabel‑Durchführung mit Klemmbacken‑Zugentlastung

Der Knauf ist die Stelle, wo das **fest installierte ummantelte
USB4‑Kabel** den Stab verlässt. Hier treten die größten mechanischen
Lasten auf (Kabel wird gezogen, verbogen, am Boden geschleift).

**Architektur‑Entscheidung 2026‑05‑17:** Statt eines externen USB‑C‑
Steckers im Knauf hat der Stab ein **fest installiertes Kabel**, das
intern an die USB‑C‑Buchse auf der Haupt‑PCB eingesteckt ist. Vorteile:

- Kein Kind kann den Stecker versehentlich abziehen
- Robust ummanteltes USB4‑Kabel ist eigener Knickschutz – kein extra TPU‑Tülle nötig
- Optisch unauffälliger – sieht aus wie ein „normales" Headset‑Kabel statt mit prominent abstehender Spiral‑Anschluss‑Buchse
- Beim Stecker‑Tausch (Schaden, Verschleiß): Halbschalen aufschrauben → Stecker entfernen → neues Kabel einstecken → wieder zuschrauben

**Konstruktive Maßnahmen im Knauf:**

1. **Kabel‑Durchführungs‑Loch** Ø 6 mm axial am Knauf‑Ende. Das Kabel
   (Ø ~4–5 mm bei USB4 ummantelt) tritt gerade nach hinten aus, kein
   90°‑Bogen mehr nötig. Lochkante mit 1 mm Fasen‑Radius, damit das
   Kabel beim Verbiegen nicht an einer scharfen Druck‑Kante reibt.

2. **Klemmbacken‑Zugentlastung als Druckteil** (Variante A,
   Tobias‑Entscheidung 2026‑05‑17):

   ```
   Querschnitt durch den Knauf:
   
   ┌─ obere Halbschale ──────────────────────┐
   │   ┌── Klemmbacke oben ──┐               │
   │   │  ◯ M3‑Schraube      │               │
   │   │  ╲ klemmt           │               │
   │   │   ╲ gegen           │      Kabel    │
   │   │    ════════════════════════════►  raus
   │   │   ╱ untere Backe    │               │
   │   │  ╱                  │               │
   │   └── Klemmbacke unten ─┘               │
   │                                          │
   └─ untere Halbschale ─────────────────────┘
   ```

   - **Zwei spiegelbildliche gedruckte Klemmbacken** (~15 × 8 × 5 mm
     pro Backe), in den jeweiligen Halbschalen integriert
   - Innen‑Profil halbkreisförmig (Radius an Kabel‑Außen‑Ø
     angepasst, typ. 2,5 mm), längs gerillt für besseren Reibungs‑Halt
     auf der Kabelummantelung
   - **1 × M3‑Insert (Ruthex Short) und 1 × M3‑Schraube** klemmen die
     beiden Halbschalen über die Klemmbacken zusammen
   - Position der Klemmbacken: ~15 mm hinter der USB‑C‑Buchse auf
     dem PCB → Stecker selbst sieht keine Zugkraft mehr
   - Beim Anziehen der Schraube wird das Kabel komprimiert
     festgehalten; bei Zug am Kabel überträgt sich die Kraft über die
     Klemmbacken in die Halbschale und über die Halbschalen‑Verschraubung
     in den Stab‑Körper

3. **Kein externer USB‑C‑Stecker mehr** an der Knauf‑Außenseite. Das
   Kabel ist intern auf der PCB eingesteckt und durch die Klemmbacken
   gegen Zug gesichert. Wer das Kabel tauschen will, muss die
   Halbschalen aufschrauben (4 × M3 Halbschalen‑Schraube + 1 × M3
   Klemmbacken‑Schraube) → kindersicher.

4. **Kabel ist ummanteltes USB4‑Kabel** (statt vorherigem Spiralkabel):
   - Standard USB‑C‑male auf USB‑C‑male, beide Seiten gleich
   - Innen am Stab eingesteckt in der USB‑C‑Buchse auf dem PCB
   - Außen geht es zur Station (dort auch USB‑C‑Buchse, vergleichbar)
   - Länge: 1,5 m, gleich wie vorher
   - Ummantelung: Stoffgewebe oder TPE, robust gegen Kratzer und
     Zug, eigene Biege‑Wiederherstellung (= Knickschutz integriert)
   - **Kein TPU‑Tülle/Knickschutz mehr nötig** – die Kabel‑Ummantelung
     selbst übernimmt die Funktion

5. **Knickschutz an der Knauf‑Austrittsstelle:** durch den 1‑mm‑
   Fasen‑Radius am Loch + die robuste Kabel‑Ummantelung erübrigt sich
   ein separates Knickschutz‑Bauteil. Falls das Kabel doch sichtbar
   am Austritt verbiegt: kleines 3D‑gedrucktes Kunststoff‑Hütchen
   (~10 mm lang, konisch) kann nachträglich aufgeschoben werden.

### 5.6 Material und Druck

| Eigenschaft | Wert | Begründung |
|---|---|---|
| Material | PETG | UV‑stabil (Halloween outdoor), zäh, im Bestand |
| Farbe | beliebig (Tobias‑Wahl) | passend zum Halloween‑Look – matt schwarz, dunkles Lila, oder anthrazit |
| Wandstärke Halbschalen | 4 mm | guter Kompromiss zwischen Stoßfestigkeit und Druckzeit |
| Wandstärke Spitze | 4 mm im Trichter‑Bereich (statt 3 mm regulär in der Halbschalen‑Wand) + 4 mm hohe Versteifungsrippen innen | bewusste Knautschzone für Front‑Stoß (siehe §3.4.1 + §5.7) |
| Perimeter | 5 (statt 3) | mehr Material in der Außenhaut für Schlag‑Resistenz |
| Infill | 25 % Gyroid | dynamisch belastbar, leichter |
| Layer | 0,2 mm Standard, 0,15 mm an der Spitze | feinere Layers wegen runder Außenkontur |
| Druck‑Orientierung Halbschalen | flach auf Bauplatte (Halbschalen‑Innenseite nach oben) | Layer‑Linien längs zur Stab‑Achse, schräge Wände selbsttragend |

### 5.7 Stoßfestigkeit: konstruktive Details

Tobias‑Anforderung „stoßfest". Konkret:

| Stelle | Belastung | Maßnahme |
|---|---|---|
| **Spitze auf Boden / gegen Pfosten** | Senkrechter Aufprall vorn, Stab kippt nach vorne | 4 mm Trichter‑Wand, 15 mm Schutz‑Distanz zur Linse (§3.4.1), Halbschalen‑Vorderkante trägt den Stoß. Linsen‑Frontplatte + M2‑Schrauben **nicht** im Lastpfad. Optional TPU‑Schutzring außen nachrüstbar. |
| **Stab fällt seitlich** | Lateraler Aufprall, Halbschalen‑Trennfuge belastet | 6 × M3 statt 4 × M3, Trennfuge versetzt zur Hauptlast‑Richtung |
| **Stab wird geschwungen, schlägt gegen Pfosten** | Lateraler Aufprall + Drehmoment | TPU‑Manschette außen am Schaft (optional, separat gedruckt, übergezogen) |
| **Trigger gedrückt mit Vollkraft** | Druck auf Gateron Green und Haupt‑PCB | PCB sitzt auf 3 × M3 + 2 × M2 Stand‑offs, die zusätzlichen M2‑Stand‑offs links/rechts vom Switch leiten die ~80 cN‑Daumenkraft direkt in die untere Halbschale ab. MX‑Stem läuft im Switch‑Body geführt – laterale Kräfte bleiben im Body, nicht auf den 5 Lötpins. |
| **Trigger‑Keycap bricht/verschwindet** | Keycap fällt ab oder zerbricht | Edelstein‑Keycap ist gesteckt, nicht geklebt → Ersatz‑Keycap aufstecken, Stab bleibt funktionsfähig auch ohne Keycap (man kann notfalls den MX‑Stem mit dem Fingernagel drücken) |
| **Zug am Kabel** | bei festem USB4‑Kabel: Last würde sonst direkt auf die USB‑C‑Buchse der Haupt‑PCB wirken | **Klemmbacken‑Zugentlastung im Knauf** (§5.5 Punkt 2) – das Kabel ist 15 mm hinter dem USB‑C‑Stecker zwischen 2 gedruckten Backen mit M3‑Schraube geklemmt, Zug überträgt sich auf die Halbschalen, nicht auf den Stecker. Externer Steckerabzug ist nicht mehr möglich, da das Kabel fest installiert ist (Halbschalen müssen aufgeschraubt werden, um es zu lösen) |

**Weicher Bonus: TPU‑Manschette.** Eine über den Schaft gezogene
TPU‑Manschette (separat gedruckt, Shore‑Härte 95A) bietet:

- griffigeren Halt (Reibung höher als PETG)
- visuellen Akzent (z. B. Halloween‑orange)
- zusätzlichen Stoßdämpfer um die anfälligste Zone (Schaft)

Maße: 60 mm lang, Innen‑Ø 30 mm (eng aufgezogen), Außen‑Ø 35 mm.
Sitzt am Schaft‑Konus, wird vor dem Schließen der Halbschalen aufgezogen.

### 5.8 Oberflächen‑Reliefs (Art Nouveau, Tobias‑Entscheidung 2026‑05‑17)

Im Zuge der ersten KI‑Render‑Iteration (siehe `21-render-konzept.md` falls
später angelegt) wurden vom Image‑Modell Art‑Nouveau‑Ranken‑Reliefs an
den Übergangszonen vorgeschlagen, die das Stab‑Bild von „Tech‑Stab"
zu „Magic Wand" heben. Tobias‑Entscheidung 2026‑05‑17: **Reliefs ins
Design aufnehmen**.

**Stil:** Art Nouveau / Jugendstil‑Ranken – geschwungene organische
Linien, asymmetrische Anordnung, leichte Stilisierung in Richtung
„keltisch‑mystisch". Kein Skull / Spider / Bat‑Motiv, kein Runen‑
Schriftzeichen, kein hartes Geometrie‑Muster.

**Slot‑Positionen (3 Stellen pro Halbschale):**

| Position | Beschreibung | Maße der Relief‑Fläche |
|---|---|---|
| **Spitze‑Schaft‑Übergang** | Manschette um den 30 → 38 mm Konus, betont den Übergang von Spitze zu Schaft | umlaufend ~360°, Länge entlang Stab‑Achse 35–40 mm |
| **Trigger‑Bereich** | links und rechts vom Edelstein‑Keycap, betont die Bedien‑Stelle | je Seite ~30 × 60 mm, am 38 mm‑Griff |
| **Knauf‑Übergang** | Manschette am Schaft‑Knauf‑Übergang, spiegelt optisch die Spitzen‑Manschette | umlaufend ~360°, Länge entlang Stab‑Achse 25–30 mm |

**Relief‑Geometrie:**

| Parameter | Wert | Begründung |
|---|---|---|
| Erhebung über Halbschalen‑Außenfläche | **1,0 mm** (Range 0,8–1,2 mm) | bei 0,2 mm Druck‑Layer = 5 Layer Aufbau, gut sichtbar, drucktechnisch stabil |
| Kantenwinkel | 60–80° (leicht angeschrägt, nicht senkrecht) | bessere Druckbarkeit ohne Stützmaterial, Licht‑Schatten‑Spiel im fertigen Druck |
| Mindest‑Strichbreite | 0,8 mm | unter 0,6 mm wird's bei 0,4 mm Nozzle unschön |
| Mindest‑Abstand zwischen Ranken | 1,5 mm | sonst „verschmieren" benachbarte Linien beim Druck |
| Anzahl Ranken pro Slot | 3–5 Hauptlinien + kleinere Verästelungen | dichter wird unleserlich, weniger wirkt zu sparsam |

**Multi‑Material‑Optionen (Bambu H2D AMS):**

Drei Varianten, von einfach nach aufwendig:

| Variante | Druck‑Aufwand | Optik |
|---|---|---|
| **Mono (gleiche Farbe)** | nur 1 Filament | Reliefs als Höhenprägung, sichtbar durch Licht‑Schatten. Subtil, „edler Hexen‑Stab"-Look. Empfehlung für Erstauflage. |
| **Akzent‑Farbe** | 2 Filamente (Body + Relief) | Relief in dunkler/heller Tonung als Body. Z. B. anthrazit Body + bronze‑grau Relief, oder schwarz Body + dunkles Lila Relief. Sehr stimmig, aber AMS‑Lernkurve. |
| **Patina‑Effekt** | 2–3 Filamente + Slicer‑Trick | Body anthrazit, Relief‑Höhen + Vertiefungen in unterschiedlichen Farben gestaffelt (Color Painting im Slicer). Sehr aufwendig. Erst für V4 / Halloween 2027 sinnvoll. |

**Entscheidung Erstauflage 2026:** **Mono** – nur 1 Filament. Reliefs
nur als geometrische Höhenprägung. Begründung: AMS‑Multi‑Material‑Druck
verlängert die Druckzeit pro Halbschale ~30 % und verbraucht durch
Tool‑Change‑Purge‑Türmchen zusätzliches Filament. Für eine reine
Optik‑Verbesserung am Anfang nicht nötig. Wenn der Mono‑Druck steht
und gefällt, kann V2 der Halbschalen als Multi‑Material‑Druck
nachfolgen.

**CAD‑Umsetzung (Fusion 360):**

Reliefs auf einer gekrümmten Zylinder‑Außenfläche sind in Fusion einer
der trickreicheren Workflows. Zwei Methoden funktionieren:

| Methode | Wofür | Schritte |
|---|---|---|
| **SVG → Project to Surface → Emboss** | für moderate Krümmung (Griff Ø 38 mm = relativ flach gesehen) | 1. SVG mit Ranken in Inkscape/Illustrator als geschlossene Pfade vorbereiten. 2. In Fusion: `Insert > Insert SVG` auf eine Hilfs‑Ebene tangential zur Halbschale. 3. `Sketch > Project / Include > Project to Surface` (Projection Type: „Closest Point" bei Zylindern, Verzerrung bei <30 mm Breite vernachlässigbar). 4. `Create > Emboss`, Profile = projizierter Sketch, Face = Halbschalen‑Außenfläche, Effect = Raised, Depth = 1,0 mm. |
| **Form Workspace + Wrap to Surface** | für starke Krümmung oder umlaufende Reliefs | 1. SVG → Sketch auf flache Hilfs‑Ebene. 2. `Form` Workspace öffnen, T‑Spline‑Plane einfügen. 3. Sketch auf die T‑Spline mit `Pull/Press` formen. 4. `Modify > Wrap to Surface` projiziert die T‑Spline auf den Zylinder. 5. T‑Spline finalisieren (`Finish Form`), Solid in Hauptbody kombinieren. |

Für den **Erstdruck** ist die SVG‑Project‑Methode der schnelle Weg.
Wrap‑to‑Surface lohnt sich erst, wenn umlaufende Reliefs gewünscht
sind, die sich nicht mehr planar projizieren lassen ohne sichtbare
Verzerrung.

**SVG‑Quellen für Art‑Nouveau‑Ranken:**

| Quelle | Lizenz | Tipp |
|---|---|---|
| Freepik.com | meist kostenlos mit Attribution | Suchbegriff: „art nouveau border ornament svg" oder „filigree vector black" |
| The Noun Project | CC BY 4.0 oder Pro | Suchbegriff: „nouveau scroll", „filigree" |
| Vecteezy.com | kostenlos mit Attribution | „art nouveau divider" |
| Eigene Erstellung in Inkscape | frei | aus den KI‑Render‑Bildern (siehe Render‑Konzept) Konturen nachvektorisieren |
| KI‑Generation | frei | Claude/ChatGPT um SVG‑Code bitten („generate a black-on-white art nouveau scroll vine SVG, 80 × 30 mm, suitable for embossing on a 3D printed wand grip") |

**Test‑Workflow vor dem Halbschalen‑Erstdruck:**

1. Ein flaches Test‑Stück ~40 × 60 mm in der Hauptfarbe drucken mit
   einem Relief‑Probestück bei 1,0 mm Erhebung.
2. Sicht‑Check: Linien sauber, Layer‑Linien stören das Muster nicht?
3. Falls ja: Reliefs in die Halbschalen‑STL einbauen und drucken.
4. Falls nein: Höhe auf 1,2 mm erhöhen oder Linienbreite ergrößern.

> **Stand 2026‑05‑17:** Reliefs sind im Design eingeplant, CAD‑Umsetzung
> noch offen (siehe `11-offene-punkte.md` Punkt 33). Falls die
> Reliefs‑Modellierung nicht rechtzeitig vor Halloween 2026 fertig ist,
> wird die Erstauflage **ohne Reliefs** gebaut – das Design funktioniert
> auch ohne, die Reliefs sind ein Optik‑Bonus, kein Funktions‑Bauteil.

---

## 6. BOM V3 (Diff zur V2.1)

| Bauteil | V2.1 | V3 | Hinweis |
|---|---|---|---|
| Connector | JST‑PH 7‑pol (S7B‑PH‑K‑S) | **2×3‑Lötheader `J1` (HDR‑2X3‑2.54)**, Kabel angelötet; Station‑Seite = 6‑pol Schraubklemmstecker | geändert 2026‑06‑08, ersetzt USB‑C 24‑Pin |
| Trigger‑Pull‑down R1 | 10 kΩ vorhanden | **entfällt** | active‑LOW |
| **Trigger‑Schalter U3** | PB‑22E85L 6‑Pin Mikrotaster | **Gateron Green (Cherry MX Green Klon), 5‑Pin PCB‑Mount** | 50 M Klicks, Click‑Jacket, 80 ± 15 gf Aktivierung, 2,3 mm Pretravel. Direkt aufgelötet. 36er‑Pack bestellt 2026‑05‑17. Fallback (Cherry MX Blue / Gateron Blue) wird nur beschafft, falls Kinder‑Test die 80 gf als zu hart bewertet. |
| **IR‑LED IR1** | TSAL6200 auf Haupt‑PCB | **TSAL6100** auf Optik‑PCB im Optik‑Block | drop‑in, ±10° statt ±17°, 2× heller, passt komplett in OM3‑Apertur |
| IR‑Vorwiderstand R2 | 33 Ω auf Haupt‑PCB | **33 Ω auf Haupt‑PCB** | unverändert (TSAL6100 hat gleiche U_F wie TSAL6200) |
| IR‑Treiber Q1 + R3 | SS8050 + 1 kΩ | **BSS138 + 820 Ω + 10 kΩ Pulldown** | Logic‑Level MOSFET statt NPN (siehe §4.1.1) |
| Laser | KY‑008 mit Kabel | **KY‑008 mit Kabel + Madenschrauben‑Justage** | Modul gleich, Halterung neu |
| Laser‑Treiber Q2 + R4 | SS8050 + 1 kΩ | **BSS138 + 820 Ω + 10 kΩ Pulldown** | Logic‑Level MOSFET statt NPN (siehe §4.1.1) |
| **NeoPixel‑Anzahl** | 2 × SK6812RGBW | **6 × SK6812RGBW** (Schaft + 4 Halo um Trigger + Knauf) | Halo statt Single‑NeoPixel: 4× mehr Lichtleistung, flächige Ausleuchtung |
| NeoPixel‑Bestückung | beide auf Haupt‑PCB | **#1 + #3 per Kabel, #2a–d (4× SMD) onboard um Cherry MX** | Position‑genau, Halo‑LEDs fest am PCB |
| **ESD‑Array** | – | **kein Teil auf der Wand** | ESD sitzt auf der Station (D2/D3 = 2× USBLC6‑2SC6 an CN1, Doc 12 § 8.11). SP3012‑04UTG verworfen; Verwechslungsschutz entfällt (keine USB‑Buchse) |
| Linse | OM3 (Bestand Tobias) | **OM3** | unverändert, **15 mm hinter Stab‑Front** versetzt (statt 5 mm) für Stoßschutz |
| **Linsen‑Frontplatte (neu)** | – | **3D‑Druck PETG, Ø 28 × 2 mm + 4 × M2** | Linse austauschbar (Halte‑Pin‑Lösung statt Feingewinde), mechanisch entkoppelt vom Front‑Stoßpfad |
| **Optik‑Block (neu)** | – | **3D‑Druck PETG, 28×28×35 mm** | Herzstück V3, sitzt 15 mm hinter Stab‑Front |
| **Edelstein‑Keycap (neu)** | – | **3D‑Druck transparentes PETG, hexagonal‑gefacettet Ø 14 × 8 mm, Plus‑Stem 4 × 4 mm** | gesteckt auf Cherry MX Stem, beleuchtet durch Halo |
| Druckteile | Linsenhalter + Stab‑Gehäuse | **Optik‑Block + 2 Halbschalen mit Schutz‑Trichter, integrierten Klemmbacken im Knauf und Art‑Nouveau‑Reliefs (§5.8) + Linsen‑Frontplatte + Edelstein‑Keycap + (opt. TPU‑Manschette + TPU‑Spitzen‑Schutzring)** | einteilige Halbschalen mit integralem Trichter und Knauf‑Klemmbacken; Reliefs als Höhenprägung mono‑material. TPU‑Knickschutz‑Tülle entfällt (Kabel‑Ummantelung übernimmt das) |
| Inserts | nicht spezifiziert | **7 × M3 Short** (6× Halbschalen‑Verschraubung + 1× Klemmbacken im Knauf) + **2 × M3 Standard** + **8 × M2 Short** (3 Laser + 4 Linsen‑Frontplatte + 1 unused) + **4 × M2‑Stand‑off‑Inserts** für PCB‑Befestigung im Griff | alles im Tobias‑Bestand |
| **3D‑Druck‑Zugentlastung** | – | **Klemmbacken im Knauf, integraler Teil der Halbschalen** (§5.5 Punkt 2) | Halten das fest installierte, angelötete Kabel; Lötpads werden nicht belastet |
| **Stab‑Kabel** | USB‑C‑Spiralkabel (1,5 m) | **ummanteltes 6‑adriges Kabel, fest installiert, 1,5 m, an die Wand‑PCB angelötet** (5 V/GND je ~0,34 mm², Signale ~0,25 mm²) | extern nicht abziehbar (kindersicher); robuste Ummantelung als Knickschutz; geändert 2026‑06‑08 |
| **Filament‑Bedarf** | – | **PETG opak (Halloween‑Farbe) + PETG transparent + PLA Stütze + opt. TPU 95A** | dank Bambu H2D Multi‑Material in einem Druck |

### 6.1 Bestell‑Liste (was wirklich neu zu kaufen ist)

| Position | Menge (für 4 Stäbe) | Preis pro Stab | Quelle |
|---|---|---|---|
| **2×3‑Stiftleiste `J1`** (HDR‑2X3‑2.54, Kabel angelötet) | 1 pro Stab | ~0,10 € | beliebig / Bestand |
| ~~2× USBLC6‑2SC6 (ESD)~~ – **gehört zur Station‑BOM**, nicht zur Wand | – | – | s. Doc 12 § 8.11 / Doc 15 |
| **6‑adriges Mantelkabel, 1,5 m** (5 V/GND ~0,34 mm², Signale ~0,25 mm²) | 4 + 1 Reserve | ~2,00 € | beliebig (Steuerleitung / Litzenbündel mit Geflecht‑Schlauch) |
| **6‑pol Schraubklemmstecker grün** (Station‑Seite, z. B. DIFLAX) | 1 pro Station | ~1,10 € | AliExpress / Amazon |
| **Gateron Green, 5‑Pin PCB‑Mount** (36er‑Pack inkl. 28 Stück Reserve für andere Projekte) | 1 × 36er‑Pack | ~2,75 € (10,99 € / 4 Stäbe) | DRAOZA via Amazon, bestellt 2026‑05‑17 |
| KY‑008 Lasermodul mit Kabel | 4 + 2 Reserve | ~1,50 € | AliExpress |
| **4 × SK6812 SIDE‑A 4020 RGB** (Halo onboard, stehend montiert) | 1 × 100er‑Pack (DRAOZA‑Modell) bestellt 2026‑05‑17 | ~0,30 € (11,59 € / 4 Stäbe, Pessimisten‑Ansatz) | AliExpress (Wushi Optoelectronics) |
| Wand‑V3‑PCB JLCPCB‑Order (**62 × 24 mm rechteckig, weiß, HASL with Lead**) ✅ **bestellt 2026‑05‑17** | 5 Stück (5er Min.) | ~0,40 € ($2,10 Special Offer) | JLCPCB |
| Optik‑PCB JLCPCB (IR‑LED‑Tochter, **10 × 14 mm, schwarz, HASL with Lead**) ✅ **bestellt 2026‑05‑17** | 5 Stück (5er Min.) | ~0,85 € ($4,20) | JLCPCB |
| **TSAL6100 IR‑LED** (drop‑in für TSAL6200, ±10° statt ±17°, 2× heller) | 4 + 4 Reserve | ~0,50 € | Reichelt |
| 4 × M2 Senkkopf für Linsen‑Frontplatte | 16 + 8 Reserve | ~0,20 € | Reichelt (Sammelposition) |
| **Summe Material pro Stab** | | **~11,65 €** | |
| **Plus Filament + Bestand‑Anteile** | | ~3 € | |
| **Endpreis pro Stab** | | **~14,65 €** | |

> **Trigger‑Posten in Perspektive:** Wenn man die 36 Gateron Green ehrlich
> nur auf die 4 Stäbe umlegt (4 verbraucht, 32 Reserve), liegt der reine
> Trigger‑Posten bei 0,31 €/Stück × 4 = **1,24 € insgesamt** = 0,31 €/Stab.
> Der oben aufgeführte Wert von 2,75 €/Stab rechnet das ganze 10,99 €‑Pack
> kostenseitig dem Stab‑Projekt zu (Pessimisten‑Ansatz). Realistisch
> wandern die 32 Reserve‑Switches in andere Maker‑Projekte (Macropads,
> Halloween‑Props, V4‑Wand), so dass der echte Stab‑Trigger‑Anteil eher
> bei 0,31 €/Stab liegt → **Endpreis pro Stab dann ~9,80 €**.

(Vergleich: aktuelle V2.1 ist nicht systematisch erfasst, aber der
GX16‑Stecker allein war ~2 € + Custom‑Kabel ~3 € = 5 € nur Anschluss.)

---

## 7. Konzeptioneller Datenfluss

Die Wand selbst hat keinen Datenfluss „in der Wand" – alle Signale
sind direkt vom Trigger / zur LED und gehen über das Kabel zur
Station. Damit ist die Wand reine analoge/digitale Signal‑Vermittlung.

```
            ┌─────────── Wand V3 ──────────────────────────────────┐
            │                                                       │
USB‑C Pin 1 │ 5 V ───┬─── R2 ─── IR‑LED‑Anode (Tochter‑PCB)         │
USB‑C Pin 2 │ GND ───┼──────────── IR‑LED‑Kathode ── Q1 ── GND      │
USB‑C Pin 3 │ Trig ──┼──── Trigger‑Schalter ──── GND                │
USB‑C Pin 4 │ NPx ───┼─── NeoPixel #1 ── #2 (PCB) ── #3 ── GND       │
USB‑C Pin 5 │ IR ────┼─── R3 ── Q1 Basis                            │
USB‑C Pin 6 │ Laser ─┴─── R4 ── Q2 Basis ── Laser‑Modul ── GND      │
            │                                                       │
            │  Tochter‑PCB im Optik‑Block: nur LED + JST            │
            │  Lasermodul fest verbunden, JST‑PH abgehend           │
            │                                                       │
            └───────────────────────────────────────────────────────┘
                                ↑
                              Kabel zur Station
                              (USB‑C‑Spiralkabel, 6 Adern)
```

**Logik‑Verhalten (in der Station, zur Erinnerung):**

- Trigger gedrückt → Station löst Schuss aus
- Schuss = Station treibt RMT‑Signal auf IR‑Pin (24‑Bit Infinitag‑Protokoll)
- Schuss + 1 s = Station treibt Laser‑Pin auf HIGH (Visier‑Hilfe)
- Setup‑Modus = Station treibt Laser‑Pin permanent auf HIGH
- NeoPixel = Station treibt Daisy‑Chain (Status‑LED → Stab‑#1 → Stab‑#2)

---

## 8. Iterationsplan

### Phase 1 – Optik‑Block isoliert

Bevor irgendein Stab gebaut wird, muss die Optik überzeugen:

1. Optik‑Block als Single‑Print (CAD → STL → PETG, Front auf Bauplatte).
2. TSAL6200 einsetzen, OM3 einlegen, Tochter‑PCB anlöten.
3. Test‑Aufbau auf Steckbrett: 5 V, 33 Ω, NPN, vorhandene Wand‑PCB
   als Treiber, Schuss auf Test‑Target bei 5 m und 10 m.
4. **Erfolgskriterium:** sicheres Treffen einer 80‑mm‑Apertur bei
   ±10 cm Zielfehler bei 5 m, ±20 cm bei 10 m.
5. **Falls nicht:** LED‑Position um ±1 mm variieren (mehrere Block‑Drucke
   mit unterschiedlicher Defokus‑Tiefe), bis Beam‑Geometrie stimmt.

### Phase 2 – Lasermodul integriert

1. Lasermodul mit Madenschrauben‑Halterung im Optik‑Block.
2. Justage‑Vorgang trocken durchspielen (5 m Test‑Wand,
   IR‑Spot mit Smartphone‑Kamera markieren, Laser nachstellen).
3. Loctite‑Fixierung testen, Stabilität nach 24 h prüfen.

### Phase 3 – PCB V3

1. KiCad/EasyEDA‑Design des Wand‑Haupt‑PCB.
2. Tochter‑PCB für IR‑LED im selben Auftrag.
3. JLCPCB 5er‑Order (~10 € inkl. Versand).
4. SMD‑Bestückung von Hand oder JLC‑Assembly (für 5 Stück
   wahrscheinlich von Hand günstiger).
5. Bench‑Test: Trigger, IR‑LED, Laser, NeoPixel an Test‑Station.

### Phase 4 – Stab‑Gehäuse

1. CAD der vier Halbschalen (Vorderteil oben/unten, Hinterteil oben/unten).
2. Erstdruck eines Vorderteils + eines Hinterteils, ohne Elektronik.
3. Ergonomie‑Test: in der Hand halten, Trigger erreichen,
   Stabilität bei Aufprall‑Test (auf Holzboden fallen lassen aus 1 m).
4. Falls Schwachstellen: Wandstärke / Verstärkung anpassen, neu drucken.
5. Vollmontage eines Test‑Stabs mit allen Modulen.

### Phase 5 – Halloween 2026

1. 4 Stäbe in Serie bauen (jeder ~30 min Montagezeit).
2. Anbindung an die Station V2 via USB‑C‑Kabel.
3. End‑zu‑End‑Test mit Targets im Vorgarten‑Setup.

---

## 9. Beantwortete Detailfragen

### Warum OM3 und nicht OM11/OM12?

OM11/OM12 wären optisch besser (bikonvex, größerer Ø → mehr
Lichtausbeute), aber OM3 ist bei dir auf Lager. Der Aufwand für eine
zweite Linsen‑Bestellung lohnt sich nur, wenn der V3‑Optik‑Block mit
OM3 nicht ausreichend Reichweite zeigt. Erst‑Bau mit OM3, ggf. später
auf OM11 wechseln (Optik‑Block neu drucken, Linsen‑Bohrung Ø 16,7 →
Ø 25,2, LED‑Position 28 mm → 40 mm).

### Warum nicht weiße LED statt Laser?

Eine weiße High‑Power‑LED hat einen Lichtkegel von typ. ±15–30°. Selbst
mit Sammellinse bekommt man nicht den klaren Punkt, den ein Laser
liefert. Bei 5 m Distanz ist der LED‑Lichtkreis ~50 cm groß – kein
Visierpunkt mehr. Tobias‑Analyse bestätigt: Laser bleibt für Visierung
konkurrenzlos.

### Wieso 12 mm Versatz IR↔Laser, nicht 0?

Echte Koaxialität braucht einen dichroitischen Strahlteiler (~30 €,
Justier‑Albtraum). Bei 2–10 m und 80‑mm‑Targets reicht ein
**konstruktiv festgelegter Versatz** völlig aus, weil der Spieler
mental kompensiert (= „ich ziele 1 cm unter den roten Punkt"). 12 mm
sind so klein, dass sie nicht stören, aber so groß, dass IR‑LED und
Lasermodul mechanisch nebeneinander passen, ohne sich zu behindern.

### Bleibt die Wand bei ihrem 7‑pol JST‑Connector?

Nein. Am Stab gibt es **keinen steckbaren Connector mehr** – das 6‑adrige
Kabel wird an einen **2×3‑2,54‑mm‑Lötheader `J1` (HDR‑2X3)** auf der Wand‑PCB
angelötet (Entscheidung 2026‑06‑08, vorher USB‑C). Belegung
1:5V 2:GND 3:NEO 4:IR 5:LAS 6:TRIG. Auf der Station endet es im **6‑pol
Schraubklemmstecker** (Station‑V2‑Doku § 8.11). 6 Adern statt 7. Trigger
active‑LOW. R1 entfällt.

### Daumen‑Trigger oder Spitze‑Drücken?

**Entschieden 2026‑05‑17:** Daumen‑Trigger oben am Griff. Cherry MX
Green PCB‑Mount mit Edelstein‑Keycap (siehe §4.4). Begründung: für
Kinderhand intuitiver (Stab in der Faust, Daumen oben), Erwachsene
greifen genauso. „Spitze drücken" wäre wie ein Kugelschreiber – aber
dafür muss der Stab am Knauf gedrückt werden, was mit der gleichen
Hand, die ihn hält, schwierig ist.

### Warum Cherry MX / Gateron statt Mikrotaster?

Der ursprüngliche Doku‑Stand hatte einen 6×6 mm Mikrotaster
(TS‑1187A/PB‑22E85L) als Trigger. Beim Re‑Review der Trigger‑
Konstruktion am 2026‑05‑17 hat Tobias Cherry MX vorgeschlagen –
und die Analyse zeigt klare Vorteile:

- **Lebensdauer:** 50–100 M Klicks (MX/Gateron) statt 100 k–500 k
  (Mikrotaster). Bei 200 Schüssen pro Halloween × 10 Jahre × 4 Stäbe
  ~ 24 000 Klicks – selbst Mikrotaster würden das überleben, aber
  MX hat keinen Verschleißpfad.
- **Haptik:** Click‑Jacket mit 80 gf Aktivierung und 4 mm Hub ist für
  einen „Schuss" das richtige Feedback. Mikrotaster mit 0,3 mm Hub
  fühlt sich „zufällig ausgelöst" an.
- **Mechanische Stoßlasten:** MX‑Stem läuft im Switch‑Body geführt;
  Daumendruck landet nicht auf SMD‑Lötpads, sondern im Body. PCB‑Mount
  5‑Pin gibt zwei zusätzliche Kunststoff‑Plugs durchs PCB – kein
  Wackeln.
- **Standard‑Stem:** Genormter Plus‑Stem erlaubt beliebige
  Keycap‑Geometrien (siehe Edelstein), keine proprietäre Knopf‑
  Lochsystem‑Mechanik mehr nötig.

### Warum Gateron Green und nicht Cherry MX Original?

Gateron Green ist ein **mechanisch und elektrisch identischer Klon**
des Cherry MX Green: gleiche Maße, gleicher 5‑Pin‑PCB‑Mount‑Footprint,
gleiche Plus‑Stem‑Geometrie, gleiche Click‑Jacket‑Mechanik, gleiche
Aktivierungs‑Charakteristik (80 gf statt Cherry's 70 gf – minimal
schwerer, für unseren Use‑Case eher noch passender). Der einzige
nennenswerte Unterschied: 50 M Cycles statt 100 M – für unseren
~6 000‑Klicks‑pro‑Halloween‑pro‑Stab‑Bedarf irrelevant.

Preis‑Realität:

- **Cherry MX Green Original:** ~1,20 € pro Stück bei Reichelt, mit
  Versand ~25 € für 8 Stück.
- **Gateron Green via DRAOZA (Amazon):** 10,99 € für **36 Stück** =
  **0,31 €/Stück**, Prime‑Versand inklusive.

Faktor 2,5 günstiger pro Stück und 4,5× mehr Switches im Set. Die
28 Reserve‑Switches landen in zukünftigen Maker‑Projekten
(Macropads, Halloween‑Props, V4‑Wand). Bestellt **2026‑05‑17**.

> **Wichtige Bestell‑Falle bei diesem Listing:** Der Produkttitel
> spricht von „Gateron Cremiger Gelber Schalter" (= Yellow, linear).
> In der Variantenauswahl „Grün" werden aber Gateron Green (clicky,
> heavy) geliefert – Amazon‑Sammel‑Listing mit gemischten Typen unter
> derselben SKU. Im Bestellvorgang **unbedingt die Variante „Grün"
> auswählen**, nicht den Default‑Yellow.

### Warum nur 2 PCBs im Stab und nicht 3?

In einer Zwischendiskussion war eine dritte „Trigger‑Sub‑PCB" mit
Kailh Hot‑Swap‑Sockel + Halo‑LEDs angedacht. Tobias‑Entscheidung
2026‑05‑17: bei dem kompakten Bauraum im Griff bringt ein drittes PCB
mehr Stecker‑/Kabel‑Komplexität als Wartbarkeits‑Gewinn. Der Cherry MX
wird direkt aufgelötet, Halo‑LEDs sitzen daneben auf demselben Haupt‑
PCB. Bleibt: 1× Haupt‑PCB (T‑Form, mit Trigger‑Zone) + 1× Optik‑PCB
im Optik‑Block.

### Warum ist die Linse 15 mm tief im Stab und nicht direkt vorn?

Schutz vor Kratzern und Stoßschäden. OM3 ist aus Acrylglas
(Mohshärte ~2,5) und kratzempfindlich. Bei 5 mm Versatz wäre die
Linse mit jedem Pfostenkontakt fällig. Bei 15 mm passt der austretende
IR‑Strahl noch durch die Ø 14 mm Front‑Öffnung des Schutz‑Trichters,
und die Linse ist mechanisch unzugänglich. Details + Strahl‑Mathematik
in §3.4.1, Geometrie in §5.1.

### Warum zwei Halbschalen längs, nicht zwei Schalen quer?

Längs‑Halbschalen erlauben den Druck mit Layer‑Linien längs zur
Stab‑Achse (= stoßfest in Aufprall‑Richtung). Quer‑Halbschalen würden
zwei Drücke mit Layer‑Linien quer zur Stab‑Achse erzeugen → schwächste
Z‑Anhaftung läuft genau in die Bruch‑Richtung. Längs ist mechanisch
deutlich besser.

---

## 10. Beziehung zur Station V2 und zu den offenen Punkten

| Thema | Wand V3 trägt bei zu | Status |
|---|---|---|
| Punkt 23 (Kabel‑Connector) | 6 Adern an 2×3‑Lötheader J1 angelötet; Station‑Seite 6‑pol Schraubklemmstecker (geändert 2026‑06‑08, vorher USB‑C) | ✅ in V3 gelöst |
| Punkt 24 (Linsen‑Toleranz) | Optik‑Block als Referenzbauteil | ✅ in V3 gelöst |
| Punkt 25 (Laser‑Justage) | Madenschrauben + Loctite | ✅ in V3 gelöst |
| Punkt 26 (Montage‑Aufwand) | Modulares Subassembly + Halbschalen | ✅ in V3 gelöst |
| Stations‑Anbindung | Wand‑V3 + Station‑V2 = neues Standard‑Paar | ✅ Station‑Doku V2.11 verweist auf V3 |

---

## 11. Lebende Diskussionspunkte / nächste Themen

- [x] **Drucker‑Bett‑Größe bestätigen** ✅ 2026‑05‑08 – Bambu H2D
      (350 × 320 mm), Halbschalen einteilig 290 mm möglich, kein
      Mitten‑Splitting nötig.
- [x] **Trigger‑Position** ✅ 2026‑05‑08, präzisiert 2026‑05‑17 –
      Daumen oben am Griff, **Gateron Green (Cherry MX Green Klon),
      5‑Pin PCB‑Mount** mit hexagonal‑gefacettetem Edelstein‑Keycap,
      beleuchtet durch 4× SK6812 5050 Halo onboard auf dem
      Haupt‑PCB (siehe §4.4).
- [x] **Trigger‑Hardware MX vs. Mikrotaster** ✅ 2026‑05‑17 –
      MX‑Switch wegen Haptik, Lebensdauer, MX‑Standard‑Stem für
      freie Keycap‑Geometrie. Direkt aufgelötet (5‑Pin PCB‑Mount),
      kein Hot‑Swap.
- [x] **Trigger‑Switch‑Bestellung** ✅ 2026‑05‑17 – 36er‑Pack Gateron
      Green via DRAOZA/Amazon (10,99 €). Cherry‑MX‑Blue‑Fallback aus
      Bestand entfällt – wird nur nachgekauft, falls Kinder‑Test die
      80 gf Aktivierung als zu hart bewertet.
- [x] **PCB‑Maße kompakt** ✅ 2026‑05‑17 – Haupt‑PCB auf **62 × 24 mm
      rechteckig** verkleinert (statt T‑Form 130 × 16/22 mm),
      Position: unter dem Trigger zentriert im Griff. Restlicher
      Stab‑Innenraum frei für Kabel‑Durchführungen.
- [x] **Kabel‑Architektur** ✅ 2026‑05‑17, **geändert 2026‑06‑08** –
      **festes, ummanteltes 6‑adriges Kabel, direkt an die Wand‑PCB
      angelötet** (vorher USB4‑Kabel auf USB‑C‑Buchse). Kein Stecker am
      Stab → erst recht kindersicher. Klemmbacken‑Zugentlastung im Knauf
      (Variante A, §5.5) gilt sinngemäß weiter, hält jetzt das angelötete
      Kabel. TPU‑Knickschutz‑Tülle entfällt – Kabel‑Ummantelung übernimmt.
- [ ] **§5.5‑Mechanik‑Prosa + ASCII‑Pinouts (§4.3, §6.2) auf „angelötetes
      Kabel" überarbeiten** – noch USB‑C‑Formulierung (Verifikationspunkt,
      s. Banner Abschnitt 4). Funktional unkritisch, nur Wording/Skizzen.
- [x] **Oberflächen‑Reliefs‑Stil** ✅ 2026‑05‑17 – Art Nouveau /
      Jugendstil‑Ranken, mono‑material in V1, drei Slot‑Positionen
      (Spitze‑Schaft, Trigger‑Bereich, Knauf‑Übergang). Geometrie
      und Multi‑Material‑Optionen in §5.8 spezifiziert. CAD‑Umsetzung
      in Fusion 360 offen (siehe `11-offene-punkte.md` Punkt 33).
- [ ] **Reliefs‑CAD‑Umsetzung in Fusion 360:** SVG‑Quelle für
      Art‑Nouveau‑Ranken finden / generieren, in Fusion auf Halbschalen‑
      Außenfläche projizieren (Project to Surface), als Emboss mit
      1,0 mm Erhebung anbringen. Test‑Druck auf einem 40 × 60 mm
      Probestück vor Halbschalen‑Erstdruck.
- [x] **PCB‑Anzahl im Stab** ✅ 2026‑05‑17 – 2 PCBs: Haupt‑PCB
      (T‑Form, 16/22 mm) + Optik‑PCB im Optik‑Block. Kein Trigger‑
      Sub‑PCB.
- [x] **TPU‑Manschette** ✅ 2026‑05‑08 – als optionales Zubehör im
      Plan, separat gedruckt, bei Bedarf aufgezogen.
- [x] **TPU‑Spitzen‑Schutzring** ✅ 2026‑05‑17 – optional / nachrüstbar
      vorgesehen, für V1 (Halloween 2026) **nicht bestückt** wegen
      bisherigen V2.1‑Erfahrungswerten (keine Spitzendefekte).
- [x] **Linsen‑Fixierung** ✅ 2026‑05‑08 – 4 × M2‑Halte‑Pins via
      Linsen‑Frontplatte (robuster als Feingewinde M20×0,5).
- [x] **Linsen‑Schutzdistanz hinter Stab‑Front** ✅ 2026‑05‑17 –
      15 mm Versatz statt 5 mm + konischer Schutz‑Trichter mit
      Ø 14 mm Front‑Öffnung (siehe §3.4.1, §5.1). Magic‑Wand‑Look,
      Linse mechanisch entkoppelt vom Front‑Stoß.
- [x] **IR‑LED‑Auswahl** ✅ 2026‑05‑08 – TSAL6100 (drop‑in‑Ersatz
      mit ±10° und 2× Achs‑Intensität, optimal zur OM3‑Apertur).
- [ ] **IR‑Schutzscheibe vor der Linse:** **nicht bestückt** für V1,
      Slot im Optik‑Block aber so vorbereiten, dass eine 1 mm
      IR‑durchlässige Acryl‑Scheibe (z. B. Plexiglas IR Acryl GS 1146)
      nachgerüstet werden kann, falls Halloween 2026 sichtbare
      Linsen‑Kratzer ergibt.
- [ ] **NeoPixel‑Ring an der Spitze (V4‑Vorschau):** als Magie‑Effekt
      um die Linse herum. Würde den Optik‑Block aufwendiger machen,
      aber optisch beeindruckend. Erst nach erfolgreichem V3‑Bau.
- [ ] **Probedruck Spitze + Schutz‑Trichter:** ~80 mm langes
      Test‑Stück (Halbschalen‑Vorderkante mit konischem Innen‑Trichter
      + Optik‑Block‑Sitz) drucken, Linse einlegen und Front‑Aufprall
      auf Holzkante testen, bevor das fertige Halbschalen‑STL läuft.
- [ ] **Probedruck Edelstein‑Keycap:** hexagonal Ø 14 × 8 mm in
      transparentem PETG, Plus‑Stem‑Aussparung 4 × 4 mm. Sitz auf
      Cherry‑MX‑Stem prüfen (sollte mit +0,1 mm Spiel saugend
      passen). Lichtdurchlässigkeit gegen Halo‑LED testen.
- [ ] **Probebestückung MX‑Trigger‑Footprint:** kleines Test‑PCB mit
      MX‑Footprint + 4 NeoPixel + JST bei JLCPCB ordern (~5 €/5 Stück),
      Halo‑Look gegen Edelstein‑Keycap validieren, bevor das finale
      Haupt‑PCB‑Layout fertiggestellt wird.
- [x] **Wand‑Haupt‑PCB CAD‑Entwurf** ✅ 2026‑05‑17 – Schaltplan
      und PCB‑Layout in EasyEDA fertig (62 × 24 mm rechteckig statt
      ursprünglich T‑Form geplant). 5 Stück bei JLCPCB bestellt
      (weiß, HASL with Lead, $2,10 Special Offer).
- [x] **Optik‑PCB CAD‑Entwurf** ✅ 2026‑05‑17 – kleine Platine
      10 × 14 mm mit TSAL6100 + JST‑PH 2‑pol + 0 Ω SMD‑Brücke (R1).
      5 Stück bei JLCPCB bestellt (schwarz, HASL with Lead, $4,20).
- [x] **PCB‑Bestellung** ✅ 2026‑05‑17 – beide PCBs in einer JLCPCB‑
      Sammelbestellung (Subtotal $6,30 + DHL Express). Build‑Time
      3 Tage, erwartete Lieferung ~5–7 Tage. Wand‑PCB weiß für
      Halo‑Reflexion, Optik‑PCB schwarz für IR‑Streulicht‑Absorption.
- [x] **NeoPixel‑Footprint‑Strategie** ✅ 2026‑05‑17 – Standard 4020
      mit erweiterten Pads, sowohl liegende als auch stehende Montage
      möglich; Final‑Entscheidung nach LED‑Lieferung (DRAOZA 100er‑
      Pack ebenfalls 2026‑05‑17 bestellt).
- [x] **IR‑LED‑Fixierung mit Klemmblock** ✅ 2026‑05‑17 – statt
      M2‑Schrauben (würde Inserts in dünner Wand brauchen) wird ein
      3D‑gedruckter Klemmblock (10 × 14 × 6 mm) mit M3‑Schraube
      im Optik‑Block‑Boden verwendet. Siehe §3.3.1 für Konstruktions‑
      Details.
- [ ] **Software‑Anpassung Station V2 für Setup‑Modus:** im Logic‑ESP
      einen Menüpunkt „Stab‑Setup → Laser permanent" + IR‑Test
      ergänzen. In `06-software-wand-logic.md` aufnehmen, sobald
      Wand V3 elektrisch existiert.
- [ ] **Farbwahl PETG:** Halloween‑Look – mattschwarz, dunkles Lila,
      anthrazit oder gar zwei Halbschalen in unterschiedlichen Farben?
- [ ] **Laser‑Auto‑Off im Code:** Fail‑Safe‑Logik, dass der Laser nie
      länger als z. B. 60 s permanent läuft (Augenschutz, Hitze im Modul).
- [ ] **Optionale Vibration:** ein kleiner Vibrationsmotor am
      Haupt‑PCB für haptisches Schuss‑Feedback (~1 € extra). Cherry MX
      gibt taktilen Druckpunkt, Vibration wäre zusätzliches Schuss‑
      Feedback.

---

## 12. Verweise

- Stations‑Refactor (Pendant zur Wand V3):
  [`12-refactor-station-v2.md`](12-refactor-station-v2.md)
- Aktueller Wand‑Stand V2.1: [`02-hardware-wand.md`](02-hardware-wand.md)
- Linsen‑Katalog AstroMedia: [`10-bill-of-materials.md`](10-bill-of-materials.md),
  Abschnitt „Optik"
- Zusammenfassung der offenen Wand‑Punkte: [`11-offene-punkte.md`](11-offene-punkte.md),
  Punkte 23–26
- IR‑Protokoll, das die Wand emittiert: [`05-protokoll-ir-i2c.md`](05-protokoll-ir-i2c.md)
