# 03 – Hardware: Wand Station Dev Board V1

PCB‑Bezeichnung: **Infinitag Wand Station | Dev Board V1** (Stand 2023‑09‑08,
bestätigt via Foto 2026‑05‑07; frühere Schaltplan‑Notiz sagte 2023‑06‑18 –
vermutlich Datum der ersten Schaltplan‑Version, das PCB selbst wurde später
gefertigt).
Quelle: Schaltplan & Layout (Easyeda).

Die Station ist die zentrale Box. Sie versorgt die Wand, beheimatet beide
ESP‑Module und generiert den Sound. Im Schaltplan ist sie in zwei Bereiche
geteilt: **Sound** (links) und **Wand** (rechts). Beide ESPs sitzen als
DIP‑Module auf 20‑pol Pinheadern (`22850120ANG1SYA01`).

## Gehäuse (bestätigt via Foto 2026‑05‑07)

- **Box:** Standard‑Industrie‑Aufputz‑Verteilerkasten (orange, Kunststoff,
  IP‑Schutzklasse ≥ IP55). Zweiteilig mit Aufklappdeckel, verschraubt.
- **Inneneinteilung:** 3D‑gedruckte Trennwand teilt das Innere in
  **linke Kammer** (Lautsprecher) und **rechte Kammer** (Elektronik).
- **Lautsprecher‑Halterung:** 3D‑gedruckt, Aufschrift „SF22" auf dem Teil –
  vermutlich interne Versions‑/Teilenummer des Druckteils.
- **Außenanschlüsse (linke Gehäuse‑Wand, Foto 2026‑05‑07):**
  - **Oben links – kleiner GX16‑7:** Signalanschluss zum Zauberstab,
    7‑polig (Regenbogen‑Flachbandkabel im Foto). Im Schaltplan
    `CN1`. Die Station unterstützt **genau einen Stab gleichzeitig** –
    nicht zwei, wie eine ältere Version dieser Doku behauptet hat.
  - **Unten links – großer GX16:** Stromzuführung vom externen
    5 V‑Netzteil (schwarz/rotes Kabel im Foto). Geht innerhalb der Box
    auf ein kleines Adapter‑PCB mit JST‑XH‑Verteilung weiter zum
    Hauptboard. Custom‑konfektioniert, Verbesserungs‑Vorschläge in
    [`11-offene-punkte.md`](11-offene-punkte.md), Punkt 27.
- **Träger‑PCB:** Die ESP‑Module und der MAX98357‑Breakout sind auf dem
  Dev Board V1 gesteckt und mit dem Gehäuse verschraubt.

## Stromversorgung

| Eintrag | Bauteil / Wert | Bemerkung |
| --- | --- | --- |
| `PWR` | `S2B‑XH‑A‑(LF)(SN)` 2‑pol JST‑XH (Schaltplan) | externer 5 V Eingang (z. B. Steckernetzteil) |
| **Externer Anschluss** | **Großer GX16‑Stecker** an Gehäuse‑Außenwand | Stromzuführung vom Netzteil via Custom‑Kabel – Verbesserung geplant, siehe [`11-offene-punkte.md`](11-offene-punkte.md) Punkt 27 |
| 5 V Schiene | direkt an Pin 1 von PWR | versorgt MAX98357, Wand 5 V, evtl. ESPs über VIN |
| GND | direkt an Pin 2 von PWR | gemeinsame Masse |
| 3,3 V | wird von den ESP‑Modulen selbst erzeugt (interne LDOs) und über die Header `ESP_S3_3V3` bzw. `ESP_S_3V3` auf der Platine verteilt |

Es gibt keine eigene 5 V→3,3 V‑Stage auf der Station; die 3,3 V kommen aus den
ESP‑Modulen. Wenn die ESP‑Module das nicht liefern können (Strombudget), muss
man das ergänzen → `11-offene-punkte.md`.

## Block „Sound“ (linke Seite)

### ESP‑Slot Sound

Zwei 20‑Pin‑Pinheader `ESP_S3_LEFT` + `ESP_S3_RIGHT` für ein ESP32‑S3
DevKit‑Modul. Verwendete Pins:

| Header | Pin | Net | Verbindung |
| --- | --- | --- | --- |
| ESP_S3_LEFT | 1 | `ESP_S3_3V3` | 3,3 V Schiene |
| ESP_S3_LEFT | 20 | `5V` | 5 V Schiene |
| ESP_S3_RIGHT | 1 | `GND` | Masse (über Pin 20 RIGHT auch) |
| ESP_S3_RIGHT | 3 | `ESP_S3_IO1` | I²S LRC → MAX98357 Pin 1 |
| ESP_S3_RIGHT | 4 | `ESP_S3_IO2` | I²S BCLK → MAX98357 Pin 2 |
| ESP_S3_RIGHT | 16 | `ESP_S3_IO47` | I²S DIN → MAX98357 Pin 3 |
| ESP_S3_RIGHT | 18 | `I2C_SCL` | gemeinsamer I²C‑Bus, am S3 = GPIO20 |
| ESP_S3_RIGHT | 19 | `I2C_SDA` | gemeinsamer I²C‑Bus, am S3 = GPIO19 |
| ESP_S3_RIGHT | 20 | `GND` | Masse |

> **Pin‑Belegung MAX98357 verifiziert via Code:** Der produktive Sound‑Sketch
> `ESP32I2S_S3_MMC_Test.ino` setzt `audio.setPinout(I2S_BCLK=2, I2S_LRC=1,`
> `I2S_DOUT=47)` und I²C auf `(SDA=19, SCL=20)`. Das passt exakt zum Schaltplan,
> wenn man berücksichtigt, dass MAX98357 Pin 1 = LRC, Pin 2 = BCLK, Pin 3 = DIN
> ist (Datasheet). Eine frühere Version dieser Doku hatte LRC/BCLK vertauscht –
> jetzt korrigiert. Siehe [`07-software-wand-sound.md`](07-software-wand-sound.md).

Welcher genaue ESP32‑S3‑Devkit‑Footprint hier vorgesehen ist, hängt vom Header
`22850120ANG1SYA01` ab (20‑Pin THT, 2,54 mm Raster). Praktisch passt jedes
Standard‑S3‑DevKit dieser Pinzahl. Die GPIO‑Nummern (`IO1`, `IO2`, `IO47`)
stehen direkt im Schaltplan an den Net‑Namen.

> **Bestätigt (2026‑05‑07):** Tobias hat einen **ESP32‑S3‑DevKit‑Variante mit
> eingebautem MicroSD‑Slot** auf der Modul‑Platine eingesetzt. Damit liegen
> die im Sound‑Sketch (`ESP32I2S_S3_MMC_Test.ino`) per `SD_MMC.setPins(39, 38, 40)`
> angesprochenen Pins als CMD/CLK/D0 schon auf dem Modul – das Wand‑Station‑PCB
> braucht selbst keinen SD‑Slot. Die genaue Modul‑Bezeichnung wird in
> [`11-offene-punkte.md`](11-offene-punkte.md) als nachzureichende
> Information geführt.

### Audio‑Stage MAX98357

> **Hinweis (Foto 2026‑05‑07):** Im realen Aufbau ist der MAX98357 **kein
> THT‑Bauteil direkt auf der Station‑Platine**, sondern ein kleines grünes
> **Breakout‑Board** (typisch: Adafruit MAX98357A Breakout oder kompatibles
> Modul), das auf dem unteren Teil des Dev Board V1 eingesteckt ist.
> Der Schaltplan‑Anschluss `MTF185‑107SY1` ist der Board‑to‑Board‑Header,
> auf den das Breakout gesteckt wird.

`MAX98357` (`MTF185‑107SY1`, 7‑pol THT‑Header) – ein I²S‑Class‑D‑Verstärker
mit fester Mono‑Ausgangsstufe.

| MAX98357‑Pin | Funktion | Verbindung |
| --- | --- | --- |
| 1 | LRC | `ESP_S3_IO1` |
| 2 | BCLK | `ESP_S3_IO2` |
| 3 | DIN | `ESP_S3_IO47` |
| 4 | GAIN | über `GAIN`‑Header `PZ254V‑11‑03P` einstellbar (siehe unten) |
| 5 | SD | im Schaltplan als „X“ markiert → unbeschaltet (Modul aktiv) |
| 6 | GND | Masse |
| 7 | VIN | `ESP_S3_3V3` (3,3 V) – nicht 5 V! |

**Gain‑Konfiguration:**

- 3‑pol Header `GAIN`
- Pin 2 ist über `R1 = 100 kΩ` nach GND und gleichzeitig MAX98357‑Pin 4
- Mit der Steckbrücke wählt man zwischen den drei Modi (siehe MAX98357‑Datasheet:
  9 dB / 12 dB / 15 dB / 6 dB / 3 dB je nach Beschaltung).
- Default in der Doku: 12 dB (mit R1=100 kΩ direkt nach GND).

> ⚠️ **VIN am MAX98357 = 3,3 V** ist möglich, aber begrenzt die Lautstärke
> deutlich (max. ca. 700 mW an 4 Ω). Wenn lauter benötigt: VIN auf 5 V umlegen
> – Pull/Trace‑Mod auf der Platine notwendig.

### Lautsprecher

Verbaut: **Visaton VS‑FR7/4** (Mini‑Breitbänder, 7 cm Membran, 4 Ω).
Vom Händler "Best Price Square" verkauft, der eigentliche Hersteller ist
Visaton.

| Parameter | Wert |
| --- | --- |
| Membrandurchmesser | 7 cm |
| Impedanz | 4 Ω |
| Nennbelastbarkeit (RMS) | 5 W (Visaton‑Originalspec) |
| Spitzenleistung (PMPO) | 10 W (Händler‑Angabe) |
| Frequenzbereich | ca. 150 Hz – 20 kHz (Visaton); 130 Hz–20 kHz beim Händler |
| Resonanzfrequenz f₀ | ~250 Hz |
| Empfindlichkeit (1 W / 1 m) | ~79 dB |
| Empfohlenes Gehäusevolumen | 0,3 – 0,7 l geschlossen |
| Anschluss | direkt an die `+`/`−`-Pads des MAX98357 (kein eigener Steckverbinder im Schaltplan) |

**Auslegung passt:** 4 Ω ist die untere Grenze des MAX98357, die Belastbarkeit
des Speakers (5 W RMS) liegt deutlich über der maximalen Ausgangsleistung
des Verstärkers (≤ 3,2 W RMS auch bei 5 V VIN). Das System klippt nie am
Speaker, sondern höchstens am Verstärker.

**Optimierungs‑Hebel** (Reihenfolge nach Wirkung):

1. VIN am MAX98357 auf **5 V** statt 3,3 V – Lautstärke‑Verdrei‑/vierfachung
   ohne Speaker‑Risiko (siehe [`11-offene-punkte.md`](11-offene-punkte.md), Punkt 14).
2. Geschlossenes 3D‑gedrucktes **Gehäuse mit 0,3–0,5 l** Volumen für den
   Speaker. Bringt den größten Klanggewinn nach Punkt 1.
3. **GAIN‑Brücke** auf 15 dB (höchste Stellung).

**Mögliche Speaker‑Upgrades** (falls die Box neu gemacht wird):

- **Visaton FRS 7** (8 Ω, ähnliche Bauform, robusterer Korb).
- **Visaton FR 10/4** (10 cm, fu ≈ 80 Hz, 4 Ω) – deutlich tieferer
  Frequenzbereich, gut für Werwolf/Geistergrollen, braucht aber größeres Gehäuse.

## Block „Wand“ (rechte Seite)

### ESP‑Slot Logic

Zwei 20‑Pin‑Pinheader `ESP_S_LEFT` + `ESP_S_RIGHT` für ein zweites
ESP32‑Modul. Verwendete Pins gemäß Schaltplan:

| Header | Pin | Net | Verbindung |
| --- | --- | --- | --- |
| ESP_S_LEFT | 1 | `ESP_S_3V3` | 3,3 V Schiene Logic‑ESP |
| ESP_S_LEFT | 9 | `ESP_S_IO25` | → CN1 Pin 3 (Trigger_ESP) |
| ESP_S_LEFT | 11 | `ESP_S_IO27` | → CN1 Pin 5 (LED_ESP) |
| ESP_S_LEFT | 12 | `ESP_S_IO14` | → CN1 Pin 6 (Laser_ESP) |
| ESP_S_LEFT | 14 | `GND` | Masse |
| ESP_S_LEFT | 19 | `5V` | 5 V Schiene |
| ESP_S_RIGHT | 1 | `GND` | Masse |
| ESP_S_RIGHT | 8 | `I2C_SCL` | gemeinsamer I²C‑Bus |
| ESP_S_RIGHT | 9 | `I2C_SDA` | gemeinsamer I²C‑Bus |
| ESP_S_RIGHT | 13 | `ESP_S_IO4` | → CN1 Pin 4 (IR_ESP) |

Der Code legt I²C auf `Wire.begin(SDA=18, SCL=19)`. Das passt zu Header‑Pins
`ESP_S_RIGHT` 8/9 nur, wenn der Slot mit einem ESP32‑DevKit befüllt wird,
dessen Pinheader an diesen Positionen GPIO19 (SCL) und GPIO18 (SDA) trägt.

> **Bestätigt (Foto 2026‑05‑07):** Der Logic‑Slot ist bestückt mit einem
> **AI‑Thinker ESP32‑S** im NodeMCU‑Format (30‑Pin, PCB‑Trace‑Antenne,
> Aufschrift „AI‑Deliwery ESP32S"). Das ist ein klassischer ESP32 (Xtensa
> LX6 Dual‑Core) – kein ESP32‑S2 oder S3. GPIO19 + GPIO18 liegen beim
> AI‑Thinker ESP32‑S an den dokumentierten Header‑Positionen. ✅

### Connector zur Wand

`CN1 = S7B‑XH‑A‑(LF)(SN)` 7‑pol JST‑XH. Pin 1 ist der erste Pin im Schaltplan
(roter Punkt).

| Pin | Net | GPIO am Logic‑ESP | Funktion |
| --- | --- | --- | --- |
| 1 | `ESP_S_3V3` | – | 3,3 V zur Wand |
| 2 | `5V` | – | 5 V zur Wand |
| 3 | `ESP_S_IO25` | GPIO25 | Trigger (Eingang) |
| 4 | `ESP_S_IO4` | GPIO4 | IR‑LED Steuerung (Ausgang) |
| 5 | `ESP_S_IO27` | GPIO27 | NeoPixel Daten (Ausgang) |
| 6 | `ESP_S_IO14` | GPIO14 | Laser Steuerung (Ausgang) |
| 7 | `GND` | – | Masse |

## I²C‑Bus

Beide ESPs hängen am gleichen I²C‑Bus (Net‑Namen `I2C_SCL`, `I2C_SDA`). Im
Code:

- Logic‑ESP (`InfinitagWand.ino`): `Wire.begin(I2C_SDA=18, I2C_SCL=19)` mit
  `I2C_DEV_ADDR = 1` als Master, der dem Slave 1 das Sound‑Byte sendet.
- Sound‑ESP (`ESP32I2S_S3_MMC_Test.ino`):
  `Wire.begin((uint8_t)I2C_DEV_ADDR=1, I2C_SDA=19, I2C_SCL=20, 0)` als
  Slave mit `Wire.onReceive(onReceive)` – passt zum Logic‑ESP. Die
  unterschiedlichen GPIO‑Nummern (18/19 vs 19/20) ergeben sich daraus,
  dass am Sound‑Slot ein ESP32‑S3 sitzt, am Logic‑Slot ein klassischer
  ESP32, und beide Pin‑Header‑Positionen 18/19 auf jeweils andere interne
  GPIOs gemappt sind.

Pull‑Ups auf SCL/SDA sind im Schaltplan nicht eingezeichnet – der ESP32‑I²C
schaltet seine internen Pullups (~45 kΩ) ein, was bei kurzen Leitungen
funktioniert. Für Robustheit kann man externe 4,7 kΩ Pullups auf die 3,3 V
Schiene legen.

## Layout‑Aspekte (Dev Board V1)

- 5 Bohrungen für M3 / Distanzbolzen (4 Ecken + 1 mittig oben).
- Sound‑Stage in der rechten unteren Ecke separiert (MAX98357 + GAIN +
  PWR‑Stecker), damit GND‑Loop für Audio kurz bleibt.
- Wand‑Connector oben rechts.
- Kein dedizierter Lautsprecher‑Anschluss im Schaltplan sichtbar – Lautsprecher
  geht direkt von den +/− Pads des MAX98357 ab.
