# 02 – Hardware: Wand (Tagger / „Zauberstab“)

PCB‑Bezeichnung: **Infinitag Wand V2.1** (Stand 2023‑06‑06).
Quelle: Schaltplan & Layout aus dem Easyeda‑Bestand
(siehe [Bilder im Projekt‑Anhang]).

Die Wand ist absichtlich „dumm“ – kein eigener Mikrocontroller. Alle Signale
und Stromversorgung kommen über den 7‑poligen Connector von der Station.

## Funktionsblöcke

| Block | Bauteile | Aufgabe |
| --- | --- | --- |
| IR‑Logik | `R2 = 33 Ω`, `IR1 = TSAL6200`, `Q1 = SS8050`, `R3 = 1 kΩ` | IR‑LED über NPN schalten, mit Vorwiderstand 33 Ω an 5 V |
| Laser‑Logik | `LASER` (MTF185‑102SY1, 2‑pol Stecker), `Q2 = SS8050`, `R4 = 1 kΩ` | Laserdiode (5 V) über NPN schalten |
| Trigger | `R1 = 10 kΩ`, `U3 = PB‑22E85L` (Kurzhub‑Taster, 6‑pol) | Taster zieht `Trigger_ESP` auf 3,3 V; sonst Pull‑down auf GND |
| LEDs | `U1, U2 = SK6812RGBW‑5050` (zwei in Reihe) | Trigger / Stab wird beleuchtet, durch Logic‑ESP angesteuert (NeoPixel GRBW) |
| Connector | `S7B‑PH‑K‑S‑(LF)(SN)` 7‑pol JST‑PH | Versorgung + alle Signale zur Station |

> **Geplanter Connector‑Wechsel (Station V2 Refactor 2026‑05‑08):**
> Der bisherige 7‑pol JST‑PH (in der Station als 7‑pol JST‑XH realisiert)
> wird durch eine **USB‑C Buchse mit 6 Adern** ersetzt. Vorteile: Standard‑
> Spiralkabel statt Custom‑Konfektion, robustere Knickentlastung, einfach
> auszutauschen. Adern‑Belegung neu: 5 V, GND, Trigger (active‑LOW),
> NeoPixel, IR, Laser – die 3,3‑V‑Ader entfällt. Details in
> [`12-refactor-station-v2.md`](12-refactor-station-v2.md), Abschnitt 8.11.

> **Wand V3 Refactor 2026‑05‑08:** Parallel zur Station V2 wird auch
> die Wand komplett neu konzipiert (kabelgebunden bleibt sie). Neuer
> Optik‑Block, der IR‑LED + OM3 + Laser in einem Druckteil vereinigt
> (löst Punkte 24–26 aus [`11-offene-punkte.md`](11-offene-punkte.md)),
> neues Halbschalen‑Gehäuse mit ergonomischem Griff für jung+alt und
> stoßfester Konstruktion, neues Wand‑PCB mit USB‑C statt JST‑PH.
> Vollständiges Konzept in [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md).

## Pinbelegung Connector (Wand‑Seite)

Reihenfolge gemäß Schaltplan‑Block „Connector“ in `02_Wand_Schaltplan.png`:

| Pin | Net‑Name | Bedeutung |
| --- | --- | --- |
| 1 | `VCC_5V` | +5 V Versorgung (Laser, IR‑LED, NeoPixel) |
| 2 | `VCC_3.3V` | +3,3 V (Trigger‑Pullup‑Logik) |
| 3 | `Trigger_ESP` | Aktiv‑High‑Signal vom Taster zur Station |
| 4 | `IR_ESP` | Steuersignal für IR‑LED‑Treiber (vom Logic‑ESP) |
| 5 | `LED_ESP` | Datenleitung zur ersten SK6812 (NeoPixel) |
| 6 | `Laser_ESP` | Steuersignal für Laser‑Treiber (vom Logic‑ESP) |
| 7 | `GND` | Masse |

> ⚠️ **Diskrepanz zur Wand‑Station‑Belegung:** Das Wand Station Dev Board V1
> belegt Pin 1 mit 3,3 V und Pin 2 mit 5 V – also **vertauscht** zur Wand.
> Bevor man ein neues Kabel oder einen anderen Wand crimped: unbedingt
> [`11-offene-punkte.md`](11-offene-punkte.md) lesen. Aktuell wird das
> vermutlich entweder über ein „Cross“-Kabel oder einen geänderten PCB‑Revisionsstand
> ausgeglichen.

## IR‑LED Auslegung

- LED: **TSAL6200** (940 nm, IF = 100 mA dauerhaft, IFSM = 1 A in 100 µs Pulsen)
- Vorwiderstand: 33 Ω an 5 V → bei UF ≈ 1,35 V ergibt sich ≈ 110 mA Dauerstrom.
- Treiberweg: `IR_ESP` (Logikpegel 3,3 V) → `R3 = 1 kΩ` → Basis von Q1
  (SS8050, NPN, BVCEO ≈ 25 V, IC ≈ 1,5 A) → Kollektor an LED‑Kathode.
- Beim Schießen wird die LED nur kurz (≤ 1 s, getaktet im RC5‑Protokoll) angesteuert,
  daher ist die Auslegung mit 110 mA dauerhaft entspannt.

## Laser

- Laserdiode‑Modul mit 5 V Versorgung (KY‑008‑artig). Steuerung identisch
  zur IR‑LED über Q2/SS8050 + R4=1 kΩ.
- Laser bleibt im Code maximal `laserDelay = 1000 ms` an
  (siehe [`06-software-wand-logic.md`](06-software-wand-logic.md)).

## Trigger

- Schalter `U3 = PB‑22E85L` ist ein 6‑Pin‑Mikrotaster. Pin‑Belegung im
  Schaltplan: Pins 1‑3 GND‑Seite, 4‑6 mit Pull‑down/Pull‑up, das Signal
  `Trigger_ESP` wird beim Drücken auf VCC_3.3V gezogen.
- `R1 = 10 kΩ` als Pull‑down → Default‑Pegel ist 0.

> **Geplante Modifikation für Station V2 Refactor (2026‑05‑08):** Trigger
> wird auf **active‑LOW** umverdrahtet – Schalter zwischen Pin und GND,
> R1 entfällt, ESP nutzt internen Pullup auf dem Trigger‑GPIO. Damit
> entfällt die 3,3‑V‑Ader im Stab‑Kabel komplett, und der neue
> 6‑adrige USB‑C‑Connector reicht. Siehe Abschnitt 8.11 in
> [`12-refactor-station-v2.md`](12-refactor-station-v2.md).

## NeoPixels

- Zwei `SK6812RGBW‑5050`, in Reihe verkettet (Daisy‑Chain DOUT → DIN).
- Versorgung 5 V, Daten 3,3 V vom `LED_ESP` (NeoPixel toleriert 3,3 V Logik
  bei kurzer Leitung; ggf. Pegelwandler als Robustheits‑Optimierung – siehe
  [`11-offene-punkte.md`](11-offene-punkte.md)).
- Im Code (`InfinitagWand.ino`) als `Adafruit_NeoPixel(2, LED_PIN, NEO_GRBW + NEO_KHZ800)`.

## 3D‑Druck / Mechanik

Die Wand selbst (Stab‑Gehäuse mit IR‑Linse) liegt aktuell nicht als Druckdatei
in einem der gemounteten Ordner. Das Halloween‑Set verwendet ein bestehendes
„Zauberstab”‑Gehäuse. (Datei nachreichen → siehe `11-offene-punkte.md`).

### Bekannte Fertigungs- und Montage-Probleme (Stand 2026-05-07)

Vier offene Punkte sind in [`11-offene-punkte.md`](11-offene-punkte.md)
(Punkte 23–26) dokumentiert:

- **Punkt 23 – Kabel/Stecker:** GX16-7-Kabel knickt am Stecker ab, aufwändige
  Konfektionierung. Optionen: Silikonkabel + 3D-Knickschutz, RJ45 oder XLR.
- **Punkt 24 – IR-Linse:** 3D-Druck-Toleranzen verschieben die LED relativ zur
  Linsenachse. Lösung: LED vom Haupt-PCB trennen, direkt am Linsenhalter montieren.
- **Punkt 25 – Laser:** Modul durch Heißkleber-Hitze beschädigt; Laser und IR-Achse
  nicht koaxial. Optionen: UV-Kleber + Justierschrauben oder Laser ganz weglassen.
- **Punkt 26 – Gesamtmontage:** Mehrere Versuche pro Wand nötig. Ziel: optischer
  Subassembly als tauschbare Einheit, Standardkabel, Montageanleitung.
