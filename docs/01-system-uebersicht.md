# 01 – System‑Übersicht

## Geschichte

Infinitag entstand 2017 als offenes Lasertag‑System (Tobias Stewen, Jani Taxidis,
Florian Kleene, siehe Header der `Infinitag_Core`‑Lib). Die ursprüngliche Idee
war ein modulares System mit beliebig vielen Spielern, Teams und IR‑Sensoren
("Sensors" genannt – an einer Weste oder Waffe), die per I²C miteinander
sprechen und per WLAN an einen Server berichten. Das Projekt wurde nicht
weiterentwickelt; der heutige Einsatz ist eine **Halloween‑Schießbude** im
Vorgarten:

- 1–4 × Wand Station (Box mit Lautsprecher), jede mit eigener **Stations‑ID**
- 1 × Wand (Zauberstab als Tagger) pro Station, am Kabel der jeweiligen Station
- 8–15 × Target (Halloween‑Figur / Druckteil mit IR‑Auge), getroffen werden →
  melden Sound‑ID an die treffende Station zurück, spielen Sound ab und können
  per Schaltausgang Halloween‑Props auslösen.

**Aktueller Aufbau (Stand 2026):** 1 Station, 1 Wand, mehrere Targets.
Ausbau auf 3–4 gleichzeitige Stationen/Kinder ist geplant (siehe
[`11-offene-punkte.md`](11-offene-punkte.md), Punkte 18–21).

## Heutige Architektur (Halloween‑Setup)

Drei Bauteile, die jeweils eigenständige Geräte sind:

```
   Kind drückt Trigger     Kind sieht Treffer        Halloween‑Sound
        │                         │                         │
        ▼                         ▼                         ▼
  ┌──────────┐              ┌──────────┐             ┌──────────────┐
  │   Wand   │  IR (RC5)    │  Target  │  HTTP GET   │ Wand Station │
  │ (Hardware│ ───────────► │ (ESP32‑S3│ ──────────► │ (Logic+Sound │
  │  only)   │  24 Bit      │ + Switch │  /trigger_  │  ESP, Lautsp.│
  └────┬─────┘  Code        │  +Props) │  effect?    │              │
       │                    └─────┬────┘  sound=N    └────┬─────────┘
       │ S7B 7‑pin Kabel          │                       │
       │                          │ Schaltausgang         │ I²C
       └──────► Wand Station ◄────┘ (5 V/3,3 V/Relais)   ▼
                              an Halloween‑Prop      Sound‑ESP
                                                     spielt MP3
```

### Vier zentrale Datenflüsse

1. **Trigger → Schuss.** Der Trigger‑Button der Wand zieht über das Kabel den
   `Trigger_ESP`-Pin an der Station auf Masse. Der Logic‑ESP sieht den Pegel,
   schaltet IR‑LED und Laser für `laserDelay = 1000 ms` an, sendet einen
   24‑Bit‑Code per RC5 und sperrt sich selbst für `shootDelay = 2000 ms`.
2. **Schuss → Treffer.** Das Target empfängt den IR‑Code (TSOP4138), validiert
   ihn über die `Infinitag_Core::irDecode`-Funktion und prüft `cmd`. Bei
   `cmd == 1` und `isSystem == false` ist es ein Treffer.
3. **Treffer → Sound.** Das Target zieht über WLAN per HTTP‑GET an die Wand
   Station: `http://<IP der Station>/trigger_effect?sound=N`. Der Logic‑ESP
   nimmt den Wert auf seinem Webserver an Port 8080 entgegen und gibt ihn per
   I²C als ein Byte an den Sound‑ESP (Adresse `0x01`) weiter. *Hinweis: der
   Sound‑ESP muss diesen I²C‑Empfang noch implementieren – siehe
   [`11-offene-punkte.md`](11-offene-punkte.md).*
4. **Treffer → Prop.** Parallel zur HTTP‑Meldung schaltet der Target‑ESP
   für `hitTime` Millisekunden seine drei Schaltausgänge (`SW`, `SW_5V`,
   `SW_3.3V`) – mit konfigurierbarem Pattern (Dauer‑an oder Blink). Daran
   können Halloween‑Props (Pumpkin, Spinne, Audio‑Effekt) hängen.

## Komponentensicht

### Wand (Stab)

Reine Analog‑Hardware **ohne eigenen ESP**. Trigger‑Mosfet, IR‑LED + Linse mit
Treibertransistor, Laserdiode mit Treibertransistor, zwei SK6812RGBW‑LEDs zur
Trigger‑Beleuchtung. Wird komplett über einen 7‑poligen Steckverbinder von der
Station aus versorgt und gesteuert. Details: [`02-hardware-wand.md`](02-hardware-wand.md).

### Wand Station (PCB)

Trägerplatine für **zwei ESP‑Module nebeneinander**:

| Slot | Aufgabe | Code |
| --- | --- | --- |
| `ESP_S3_LEFT/RIGHT` | Sound‑ESP – I²S‑Audio über MAX98357 + Lautsprecher | [`InfinitagWandStation.ino`](07-software-wand-sound.md) |
| `ESP_S_LEFT/RIGHT` | Logic‑ESP – WLAN, Webserver, IR‑Senden, Wand‑I/O | [`InfinitagWand.ino`](06-software-wand-logic.md) |

Die beiden ESPs hängen am gleichen I²C‑Bus (`I2C_SCL`/`I2C_SDA`). Der Logic‑ESP
ist Master und schickt dem Sound‑ESP (I²C‑Adresse `0x01`) ein einzelnes Byte
mit der Sound‑ID. Hintergrund: Tobias hatte massive Audio‑Aussetzer, wenn
WLAN‑Reception/Transmission und I²S parallel auf einem ESP liefen – die
saubere Trennung auf zwei Chips löst das Problem.

Details: [`03-hardware-wand-station.md`](03-hardware-wand-station.md).

### Target

Eigenes PCB **InfinitagTarget V3.2** mit ESP32‑S3‑WROOM‑1 (N8R8), USB‑UART
(CP2102N), 12er RGBW‑LED‑Ring (SK6812RGBW), TSOP4138 IR‑Empfänger, drei
Schaltausgängen (potentialfreier Optokoppler `SW1`, 5 V Transistor `SW_5V`,
3,3 V Transistor `SW_3.3V`) sowie Reset‑/Boot‑Buttons.

Configuration läuft komplett über das WiFiManager‑Captive‑Portal:

- Ziel‑IP‑Prefix der Station (z. B. `192.168.0.`)
- Sound‑ID, die beim Treffer ausgelöst wird
- Hit‑Zeit (wie lange das Target „getroffen“ bleibt)
- Switch‑Animation (welches Pattern beim Treffer auf den Schaltausgängen
  läuft)

Details: [`04-hardware-target.md`](04-hardware-target.md) und
[`08-software-target.md`](08-software-target.md).

## Netzwerk

- **AP‑Modus beim ersten Boot:** Logic‑ESP der Station öffnet einen AP namens
  `InfinitagWandStation` (PW: `password`). Targets öffnen einen AP namens
  `InfinitagTarget` (PW: `YourDefaultPassword`). Beide nutzen den
  `WiFiManager`.
- **Im Normalbetrieb (aktuell, HTTP):** Alle Geräte hängen im selben Heim‑WLAN.
  Die Targets sprechen die Station per HTTP an. Der Logic‑ESP der Station
  identifiziert sich beim Schuss‑IR‑Code mit den letzten 8 Bit seiner IP –
  das Target nimmt diese, hängt sie an den per Captive‑Portal eingestellten
  IP‑Prefix dran und schickt dort den HTTP‑Request hin (siehe
  [`05-protokoll-ir-i2c.md`](05-protokoll-ir-i2c.md), Abschnitt „cmdValue”).
- **Geplant (ESP‑NOW):** Kein Router nötig. Stations‑ID im IR‑Code wird vom
  Target für das Routing des ESP‑NOW‑Pakets genutzt. Jede Station filtert
  Pakete auf ihre eigene ID und reagiert nur auf die ihr geltenden Treffer‑
  meldungen. Details und offene Fragen: [`11-offene-punkte.md`](11-offene-punkte.md),
  Punkt 20.

## Aufbauprozess

1. Wand Station mit Strom + Lautsprecher + Wand‑Stecker verbinden,
   einschalten. Ggf. einmal AP konfigurieren.
2. Targets einschalten, ggf. Captive‑Portal aufrufen, IP‑Prefix + Sound‑ID
   einstellen.
3. Halloween‑Props an die Schaltausgänge der Targets hängen
   (potentialfrei via Optokoppler oder direkt 5 V / 3,3 V getriggert).
4. Wand am Kabel der Station anschließen, Targets aufstellen, Spaß haben.
