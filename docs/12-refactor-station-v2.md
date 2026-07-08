# 12 – Refactor: Wand Station V2 (ESP32‑S3 + ESP‑NOW + LittleFS)

> **Status: lebendes Konzept‑Dokument** – wird laufend aktualisiert,
> bis eine getestete und für Halloween 2026 produktiv eingesetzte
> Version steht. Dann werden die finalen Inhalte in die regulären
> Hardware‑/Software‑MDs migriert.

Beginn der Diskussion: 2026‑05‑07. Vorgänger: das aktuelle Zwei‑ESP‑Dev‑Board V1
(siehe [`03-hardware-wand-station.md`](03-hardware-wand-station.md)).

> **Plattform‑Wechsel 2026‑05‑07:** Die Architektur‑Diskussion startete mit
> dem ESP32‑C3 SuperMini, ist aber im selben Tag auf den **ESP32‑S3‑DevKitC‑1U
> N16R8** umgestellt worden. Begründung in Abschnitt 2.1. Frühere Inhalte zu
> C3‑spezifischen Workarounds sind weiterhin als Erfahrungs‑Hintergrund hier
> dokumentiert, gelten in der V2 aber nur eingeschränkt.

---

## 1. Status‑Übersicht

| Bereich | Entscheidung | Status |
|---|---|---|
| MCU | **ESP32‑S3‑DevKitC‑1U N16R8** (16 MB Flash, 8 MB Octal PSRAM, Dual‑Core LX7 @ 240 MHz, BLE 5.0, WiFi b/g/n) | ✅ gesetzt |
| WLAN‑Antenne | externe 2,4 GHz SMA‑Antenne via Pigtail‑Kabel, SMA‑Buchse in Gehäusewand verschraubt | ✅ gesetzt |
| Stäbe pro Station | genau 1 (Multiplayer über mehrere identische Stationen) | ✅ gesetzt |
| Kommunikation Station ↔ Targets | **ESP‑NOW** (kein HTTP, kein Router) | ✅ gesetzt |
| Stations‑Versorgung | 12 V Eingang, Buck‑Down auf 5 V intern | ✅ gesetzt |
| Audio‑Kette | I²S vom S3 → PCM5102A‑DAC → **TPA3110 XH‑A232‑Verstärker** → Speaker | ✅ gesetzt (Modul angepasst von TPA3116 auf TPA3110, da kompakter und für 30 W Speaker passender dimensioniert) |
| **Audio‑Mute‑Strategie** | **Variante B: Continuous I²S‑Stream + STBY‑Mute am TPA3110** (kein DAC‑Mod) – Plan A (XSMT‑Brücken‑Mod) als Reserve | ✅ gesetzt 2026‑05‑08, im Lochraster verifizieren |
| Lautsprecher Phase A | **Vergleichsbau zwei Boxen**: Box A mit Visaton FR 7/4 (7 cm, f₀ ~250 Hz, 5 W, 4 Ω, vorhanden) geschlossen; Box B mit Visaton **FR 10/4** (Art. 2020, 10 cm, f₀ ~90 Hz, 30 W, 4 Ω, vorhanden) Bassreflex | ✅ gesetzt 2026‑05‑08, Speaker 2026‑05‑13 auf FR 10/4 zurückgestellt (Exemplar physisch vorhanden, Etikettmaße bestätigt) – früherer Zwischenschritt FRS 8 entfällt; Details in Abschnitt 8 |
| Lautsprecher Upgrade später | offen je nach A/B‑Hörtest – mögliche Optionen: JBL Stage1 41F Coax (~14 €/Stk., 4″, 2‑Wege), Dayton ND91‑4, Tang Band W3‑1876S | offen, nach Hörtest entscheiden |
| Audio‑Format | WAV 22 kHz Mono 16‑Bit | ✅ gesetzt |
| **Audio‑Quelle** | **LittleFS im 16 MB Flash** (kein SD‑Modul) | ✅ gesetzt (geändert von SD) |
| Sound‑Mixing | nur **ein Sound gleichzeitig** (Atmo läuft separat über externes Soundsystem) | ✅ gesetzt |
| **Konfiguration** | **OLED 1,3" SH1106 mit integriertem 4‑Tasten‑Modul, hinter Service‑Klappe – jetzt OPTIONAL** (Anschluss‑Footprint bleibt, Konfig läuft primär übers Config‑Tool per ESP‑NOW) | ✅ als optional gesetzt 2026‑06‑08, s. 8.14 |
| **Live‑Status außen** | **1 SK6812RGBW auf Gehäusewand** (Daisy‑Chain mit Stab‑LEDs, kein Extra‑GPIO) | ✅ gesetzt |
| GPIO‑Plan | Plan v7, siehe Abschnitt 7 | ✅ Konzept festgeschrieben, im Lochraster zu verifizieren |
| Erster Hardware‑Schritt | Lochraster‑Prototyp mit DevKitC‑1U + GPIO‑Extension‑Board | ✅ gesetzt |
| Folgeschritt | eigenes PCB für Halloween 2026 nach erfolgreichem Lochraster | ✅ gesetzt |
| **Gehäuse‑Architektur** | **Einteiliger Korpus** mit fest mitgedruckter Trennwand zwischen Speaker‑ und Elektronik‑Kammer, jeweils mit eigener Service‑Öffnung (Speaker‑Rückwand + Boden‑Klappe) | ✅ gesetzt 2026‑05‑09 (vereinfacht von zweiteiliger Modul‑Konstruktion, weil bei Druckorientierung „Front auf Bauplatte" die Druckhöhe nur die Box‑Tiefe ist – Box B 130 mm, Box A 75 mm – einteilig also problemlos) |
| **Stab‑Connector** | **grüner 6‑pol PCB‑Schraubklemmstecker** (5 V, GND, Trigger, NeoPixel, IR, Laser); Trigger active‑LOW, Wand‑Kabel direkt angelötet. **Ersetzt USB‑C** (2026‑05‑08), s. 8.11 | ✅ geändert 2026‑06‑08 |
| **PCB‑Architektur** | **2 PCBs gestapelt** (Logic‑Board + Audio‑Board, Rücken‑an‑Rücken, M3‑Nylon‑Standoffs, Inter‑Board‑Kabel JST‑XH) | ✅ gesetzt 2026‑06‑08, s. 8.14 |
| **Stab‑ESD‑Schutz** | **2× USBLC6‑2SC6 auf dem Station‑Data‑Board** (D1 + D2 am Stab‑Connector CN1, Pass‑Through inline), ersetzt das alte 4‑Kanal‑USBLC6‑4SC6Y; R1–R4 Serien‑R bleiben | ✅ gesetzt 2026‑06‑08 |
| **Antenne** | Option B: ESP32‑S3‑DevKitC‑1U + IPEX‑Pigtail + interne Whip‑Antenne an der Service‑Rückwand (vertikal, im Plastik) – kein SMA außen | ✅ gesetzt 2026‑05‑08 |
| **Pfostenhalter** | Adapter‑Druckteil **seitlich** am Korpus auf Höhe der Elektronik‑Kammer (eine Seite Box, andere Seite Stab‑Halterung) | ✅ gesetzt 2026‑05‑08 |

---

## 2. Hardware‑Plattform

### 2.1 Board: ESP32‑S3‑DevKitC‑1U N16R8

**Begründung Plattform‑Wechsel C3 → S3:**

Der ursprüngliche Plan mit dem ESP32‑C3 SuperMini wäre für das aktuelle
Lastprofil (1 Sound gleichzeitig, WAV statt MP3, sporadisches IR/NeoPixel,
ESP‑NOW) technisch ausreichend gewesen. Drei Argumente haben den Wechsel
trotzdem ausgelöst:

1. **OLED‑Konfiguration mit 4 Tasten** kostet 6 GPIOs. Der C3 hatte nur
   13 nutzbare und davon waren bereits alle für Pflicht‑Funktionen verplant –
   für Konfig‑Komfort musste etwas weg. Beim S3 (~38 nutzbare GPIOs) ist
   das ein Bruchteil des Budgets.
2. **16 MB Flash + 8 MB PSRAM** statt 4 MB Flash. LittleFS bekommt damit
   ~8–10 MB Sound‑Speicher → kein SD‑Modul nötig, 4 GPIOs gespart, eine
   Hardware‑Komponente weniger.
3. **Dual‑Core** entspannt die Audio‑Architektur. Audio‑Task auf Core 1,
   WiFi/ESP‑NOW auf Core 0 → keine Single‑Core‑Workarounds nötig.
   `WiFi.setSleep(false)` und `Serial`‑Hot‑Path‑Schutz bleiben als Best
   Practice, sind aber nicht mehr existenziell.

**Standardisierung des Form‑Faktors:**

Das **DevKitC‑1U‑Layout** ist ein offizielles Espressif‑Referenzdesign mit
festgelegter Pinbelegung (2 × 22‑Pin‑Header in 2,54 mm Raster, 25 × 70 mm).
Hersteller wie Espressif, AZ‑Delivery, Joy‑IT, FREENOVE, WaveShare, diymore
und etliche AliExpress‑Anbieter produzieren das gleiche Layout. Ein PCB,
das auf dieses Footprint ausgelegt ist, bleibt mit verschiedenen Lieferanten
kompatibel – Langzeit‑Verfügbarkeit deutlich besser als beim C3‑SuperMini‑
Klon‑Markt.

**Spec‑Tabelle:**

| Spec | Wert |
|---|---|
| Modul | ESP32‑S3‑WROOM‑1U mit IPEX/U.FL |
| Architektur | Xtensa LX7, Dual‑Core, 240 MHz |
| SRAM | 512 KB |
| ROM | 384 KB |
| Flash | **16 MB** integriert |
| PSRAM | **8 MB Octal** |
| WLAN | 802.11 b/g/n, Station + SoftAP + Mixed |
| Bluetooth | BLE 5.0 + Bluetooth Mesh |
| GPIO total | ~38 nutzbar |
| I²S Controller | 2 |
| RMT Channels | 8 |
| SPI Controller | 4 |
| I²C Controller | 2 |
| UART Controller | 3 |
| Touch‑Sensoren | 14 Touch‑Pins (in V2 nicht genutzt) |
| ADC | 2 × 12‑Bit, 20 Kanäle |
| USB | OTG nativ + USB‑Serial/JTAG |
| Antenne | IPEX/U.FL Buchse, externes Pigtail‑Kabel |
| Maße Modul | 25 × 70 mm |
| Versorgung | 5 V via USB‑C oder via 5Vin‑Pin |

**Konkretes Bestell‑Board:** diymore „ESP32‑S3 DevKitC‑1 N16R8 mit externer
Antenne". Identisches Layout zum offiziellen ESP32‑S3‑DevKitC‑1U. Falls
diymore in Zukunft nicht liefert: gleichwertige Boards von WaveShare,
FREENOVE, AZ‑Delivery, Lonely Binary oder direkt von Espressif passen
auf das gleiche PCB‑Footprint.

### 2.2 Versorgung der Station

- Eingang: **12 V DC** über DC‑Hohlbuchse 5,5/2,5 mm an der Gehäusewand
  (siehe Punkt 27 in [`11-offene-punkte.md`](11-offene-punkte.md))
- Step‑Down‑Modul (LM2596 oder MP2307, 3 A) erzeugt **5 V** für:
  - ESP32‑S3‑DevKitC (5Vin‑Pin)
  - PCM5102A DAC (3,3–5 V VIN)
  - SK6812RGBW LEDs (Status + Stab)
  - 3,3 V wird vom Onboard‑LDO des DevKitC erzeugt und steht am 3V3‑Pin zur Verfügung
- 12 V geht **direkt** auf den TPA3116D2‑Verstärker → dort kommt der Bass‑Headroom her

### 2.3 Audio‑Kette

```
ESP32‑S3 ──I²S──► PCM5102A ──Line‑Level──► TPA3116D2 ──L+/L‑──► Visaton FR 10/4
   3,3 V          3,3–5 V                   12 V
```

| Komponente | Bezeichnung | Versorgung | Preis |
|---|---|---|---|
| DAC | PCM5102A‑Modul (z. B. „GY‑PCM5102") | 3,3–5 V | ~3 € |
| Verstärker | TPA3116D2‑Modul, Mono‑Bridge oder 1 Kanal vom Stereo‑Modul | 12 V | ~6 € |
| Lautsprecher | Visaton FR 10/4 | – | ~16 € |
| **Audio‑Kette gesamt** | | | **~25 €** |

### 2.4 Datenträger Audio: LittleFS im internen Flash

**Entscheidung:** LittleFS auf der Flash‑Partition des S3 – **kein SD‑Modul**.

Begründung:

- 16 MB Flash brutto am S3‑DevKitC‑1U. Nach Bootloader, Partitionstabelle,
  zwei OTA‑App‑Slots à ~3 MB bleiben **~8–10 MB** für die LittleFS‑Partition.
- WAV 22 kHz Mono 16‑Bit = 44 KB/s. 8 MB / 44 KB ≈ **180 Sekunden Audio**.
- Aktuelle Halloween‑Bibliothek: 14 Treffer + 5 Spell ≈ 30 Sekunden gesamt.
  → ~6× Reserve im Flash.
- Keine SD‑Hardware = keine SPI‑Pins belegt, kein mechanischer Wackelkontakt
  am Karten‑Slot, kein „Karte fehlt"‑Fehlerfall.
- Sound‑Updates: per OTA über das WLAN oder per USB‑Reflash (Flash‑Image
  enthält dann auch die WAVs).

LittleFS ist im Arduino‑ESP32‑Core enthalten und im PlatformIO‑Standard
eingebunden. Upload der WAV‑Files: `pio run --target uploadfs`.

### 2.5 Konfiguration: OLED + 4‑Tasten‑Modul + Service‑Klappe

**Eingesetztes Modul:** OLED 1,3" SH1106 (128 × 64 Pixel) mit **integrierter
4‑Tasten‑Platine**. Anschluss 8‑polig: GND, VCC (3,3–5 V), SCL, SDA, K1, K2,
K3, K4. Die Tasten K1–K4 sind direkt auf dem Modul‑PCB verbaut, jeweils
gegen GND schaltend (interner ESP‑Pullup auf den GPIO‑Pins reicht).

**Konkretes Modul (verifiziert 2026‑05‑14):** Aliexpress‑Variante mit
Aufdruck **`DST‑015‑0`** auf der Rückseite. Pin‑Reihenfolge auf dem Modul
(von Modul‑Rand aus): `GND · VCC · SCL · SDA · K4 · K3 · K2 · K1`. Die
Tasten haben die Symbole ∧ (K2) / ∨ (K3) / # (K4) / ∗ (K1) auf dem PCB.
Bus‑Adresse 0x3C, I²C läuft am Breadboard mit 100 kHz stabil; 400 kHz
ggf. erst am fertigen PCB testen.

**SH1106‑Spaltenoffset (Bring‑up‑Erfahrung):** U8g2‑Konstruktor
`U8G2_SH1106_128X64_NONAME_F_HW_I2C` setzt intern `x_offset = 2`, weil
der typische SH1106 das 128‑Pixel‑Sichtfenster auf Adressen 2..129 legt.
Das DST‑015‑0‑Modul zeigt jedoch **Adressen 0..127** an. Folge:
NONAME‑Default rendert 2 Pixel‑Spalten Müll am linken Rand und schneidet
2 Spalten rechts ab. Korrektur direkt nach `u8g2.begin()`:

```cpp
u8g2.getU8x8()->x_offset = 0;
u8g2.clearDisplay();
```

Falls bei einer späteren Charge die Reste auf die rechte Seite wandern,
ist es eine NONAME‑Variante – dann `x_offset = 2` zurücksetzen oder
direkt einen anderen U8g2‑Konstruktor probieren (`_VCOMH0_`, `_WINSTAR_`).

**Bedienung:**

| Taste | Funktion |
|---|---|
| K1 | Menü öffnen / Zurück |
| K2 | Auf / Wert erhöhen |
| K3 | Ab / Wert verringern |
| K4 | OK / Bestätigen |

**Konfigurations‑Optionen, die das Menü bietet:**

- Stations‑ID (1–99 oder mehr, im NVS gespeichert)
- Lautstärke (0–100 %, Software‑Skalierung der I²S‑Samples)
- Sound‑Test (jeden gespeicherten Sound einmal abspielen)
- Diagnose (RSSI letzter ESP‑NOW‑Verbindung, Letzte Sound‑ID, Bootcounter,
  Uptime, Temperatur des S3)
- Setup‑Modus speichert in NVS, übersteht Reboot und Stromausfall.

**Service‑Klappe (Mechanik):**

- OLED + 4‑Tasten‑Modul **innerhalb** der Box hinter einer schwenkbaren oder
  abnehmbaren Klappe.
- Klappe per **M3‑Gewindeeinsatz** + Innensechskant‑Schraube oder per
  Magnet (Neodym‑Klebestücke) gesichert.
- **Schaumstreifen 1 mm** am Klappenrand → Spritzwasser‑Schutz.
- 8‑poliger **JST‑XH‑Stecker** zwischen OLED‑Modul und Hauptboard, damit die
  Klappe komplett abnehmbar ist (Service‑Tausch ohne Lötkolben).

**Vorteile gegenüber DIPs / Drehkodier‑Schalter:**

- Beliebig viele IDs, nicht auf 4 oder 16 begrenzt.
- Lautstärke und Diagnose direkt am Gerät, ohne USB‑Kabel.
- Schutz vor versehentlichem Verstellen während des Spiels (Klappe zu).
- Schutz vor Witterung (Klappe dichtet ab).
- Klemmen statt Löten → OLED bei Defekt schnell wechselbar.

**Trade‑off:** Live‑Status während des Spiels ist nur durch Klappenöffnen
sichtbar. Wird über die Außen‑Status‑LED kompensiert (siehe 2.6).

### 2.6 Live‑Status: NeoPixel an Außenwand

Eine einzelne **SK6812RGBW** sitzt sichtbar auf der Außenwand der Station
(kleine Bohrung mit Klarsicht‑Filament‑Inlay oder Heißkleber‑Tropfen als
Diffusor). Sie hängt am gleichen NeoPixel‑Bus wie die Stab‑LEDs – die erste
Position der Daisy‑Chain ist die Status‑LED, danach folgen die Stab‑LEDs
über das Stab‑Kabel.

**Vorteil:** kein zusätzlicher GPIO‑Pin nötig.

**Farbcode (Vorschlag, im Code als Default):**

| Farbe | Bedeutung |
|---|---|
| Gelb | Boot‑Sequenz (kurz nach Power‑On) |
| Blau (statisch) | Bereit, ESP‑NOW‑Stack initialisiert |
| Grün‑Blink | Treffer empfangen, Sound spielt |
| Rot (statisch) | Fehler / WLAN‑Verlust / kein Audio |
| Lila (statisch) | Setup‑Modus aktiv (Klappe offen, Konfiguration läuft) |
| Weiß‑Blink | Schuss abgegeben (Trigger gedrückt) |

**ID‑Anzeige beim Boot:** nach dem Yellow‑Flash blinkt die LED in der
Stations‑Farbe (z. B. Cyan) **N‑mal**, wobei N die aktuelle Stations‑ID ist.
Damit erkennt man aus 2 m Entfernung, ob die Box mit der richtigen ID
geladen ist.

---

## 3. Kommunikation: ESP‑NOW

### 3.1 Topologie

Drei Geräte‑Typen im ESP‑NOW‑Netz, alle gleichrangig auf der MAC‑Layer:

| Typ | Anzahl | Sendet | Empfängt |
|---|---|---|---|
| **Station** | 1..N | Treffer‑Bestätigungen (optional), `SETUP_TAKE`, `DISCOVER_REPLY`, `CFG_ACK` | `HIT_REPORT`, `SETUP_BEGIN`, `DISCOVER_REQ`, `CFG_WRITE`, `IDENTIFY` |
| **Target** | 1..N | `HIT_REPORT` (Treffer), `DISCOVER_REPLY`, `CFG_ACK` | `DISCOVER_REQ`, `CFG_WRITE`, `IDENTIFY`, `SETUP_BEGIN` |
| **Config‑Box** | 0..1 | `DISCOVER_REQ`, `IDENTIFY`, `CFG_WRITE`, `SETUP_BEGIN` | `DISCOVER_REPLY`, `CFG_ACK`, `SETUP_TAKE`, `HIT_REPORT` (Monitor) |

Identifikation:

- Jedes Gerät identifiziert sich primär über seine **48‑Bit MAC‑Adresse** aus
  dem eFuse (`esp_read_mac()` / `WiFi.macAddress()`). Die MAC ist weltweit
  eindeutig, unveränderbar und gleichzeitig die ESP‑NOW‑Adresse → kein
  Pairing‑Schritt nötig, jedes Gerät kann von Geburt an angesprochen werden.
- Die **anwenderlesbare ID** (`station_id`, `target_id`, jeweils 1..99) wird
  im NVS gespeichert und über die Config‑Box gesetzt. Ein frisch geflashtes
  Gerät meldet sich mit ID = 0 („ungesetzt") und bittet damit visuell um
  Konfiguration.
- Stationen und Targets sind **stateless gegenüber der Config‑Box**: sie
  kennen die Config‑Box nicht und merken sie sich nicht. Jeder Discovery‑
  Zyklus baut die Liste neu auf.

Adressierung:

- **Broadcast (FF:FF:FF:FF:FF:FF)** für `DISCOVER_REQ`, `SETUP_BEGIN`,
  `SETUP_TAKE`, `HIT_REPORT`. Damit erreichen wir alle Geräte ohne vorherige
  Peer‑Registrierung.
- **Unicast (an MAC)** für `IDENTIFY`, `CFG_WRITE`, `CFG_ACK`,
  `DISCOVER_REPLY`. Empfänger‑MAC ist bekannt (entweder aus eigener Tabelle
  beim Antworten oder aus dem Discovery‑Reply beim Schreiben).
- Vor jedem Unicast muss der Sender den Peer per `esp_now_add_peer()`
  registrieren. Eleganter Workaround: bei jedem eingehenden Paket den
  Absender‑Peer flüchtig hinzufügen (LRU‑Cache mit z. B. 20 Slots), beim
  Verdrängen wieder entfernen.

Filterung im Empfänger:

- Stationen verarbeiten `HIT_REPORT` nur, wenn `station_id` im Paket der
  eigenen ID entspricht. Targets ignorieren `HIT_REPORT` komplett.
- Im **Setup‑Mode** ändert sich die Bedeutung des Trigger‑Eingangs an der
  Station: er ist Bestätigungstaster, nicht Schuss‑Auslöser. Den
  vollständigen Flow (`SETUP_BEGIN` → Stab‑Trigger → `SETUP_TAKE`)
  beschreibt [`18-config-tool.md`](18-config-tool.md) § 6.1.

### 3.2 Vorteile gegenüber HTTP

| Aspekt | HTTP heute | ESP‑NOW V2 |
|---|---|---|
| Router nötig | ja | nein |
| Latenz | ~50–200 ms | <10 ms |
| Konfiguration | WiFiManager, IP‑Setup | OLED‑Menü |
| Reichweite | abhängig vom Router | direkt zwischen den Modulen, ~50–100 m frei |
| Robustheit | Single Point of Failure (Router) | mesh‑artig peer‑to‑peer |

### 3.3 ESP‑NOW + WiFi‑Modem‑Sleep

ESP‑NOW läuft auf der gleichen WiFi‑MAC‑Layer wie regulärer WiFi‑Verkehr.
Der klassische C3‑Single‑Core‑Workaround `WiFi.setSleep(WIFI_PS_NONE)` ist
auf dem **S3 mit Dual‑Core deutlich weniger kritisch**, weil Audio‑I²S‑DMA
auf einem eigenen Core läuft und nicht mehr unter WiFi‑Modem‑Sleep
„hängt". Trotzdem als Best Practice:

```cpp
WiFi.mode(WIFI_STA);
WiFi.setSleep(WIFI_PS_NONE);
```

Strom‑Penalty: ~80 mA dauerhaft. Bei 12 V Outdoor‑Netzteil egal.

### 3.4 Paket‑Format (V1)

Festgeschrieben 2026‑05‑18 zusammen mit dem Config‑Tool‑Konzept
([`18-config-tool.md`](18-config-tool.md)). Wichtig: alle Multi‑Byte‑Felder
sind **little‑endian** (ESP32‑Native), die Struktur ist `__attribute__((packed))`
ausgelegt, damit Sender und Empfänger byteweise gleich sind ohne
Padding‑Überraschungen.

```c
// Infinitag ESP-NOW packet format, version 0x01
// Fixed size: 36 bytes
typedef struct __attribute__((packed)) {
  uint8_t  version;        // 0x01
  uint8_t  msg_type;       // siehe Tabelle 3.5
  uint8_t  device_type;    // 1 = STATION, 2 = TARGET, 3 = CONFIG_BOX, 0xFF = ANY
  uint8_t  station_id;     // 1..99, 0 = ungesetzt / nicht relevant
  uint8_t  target_id;      // 1..99, 0 = ungesetzt / nicht relevant
  uint8_t  flags;          // bit0 = ACK_REQUIRED, bit1 = SETUP_MODE_ONLY, …
  uint8_t  token;          // Zufallswert, Echo‑Schutz bei Discovery / IR‑Select
  uint8_t  reserved;       // = 0, Padding auf 8 Byte Header
  uint8_t  payload[26];    // typabhängig, siehe Abschnitt 3.6
  uint16_t crc16;          // CRC‑16/CCITT‑FALSE über Bytes 0..33
} infinitag_packet_t;
```

`device_type` im Header ist redundant zu `station_id` vs. `target_id`, macht
aber das Filtern auf Empfängerseite trivial (`if (pkt->device_type ==
DEV_STATION) { … }`) ohne in den Payload zu schauen.

Der CRC ist nicht zwingend für Datenintegrität (ESP‑NOW hat schon eine
Layer‑2‑CRC), aber er fängt **falsche Paket‑Versionen** und **fremde Pakete
mit gleichem ESP‑NOW‑Header** ab, falls in der Garage zufällig irgendein
anderer ESP‑NOW‑Sender mitfunkt.

### 3.5 Nachrichtentypen

| Code | Name | Sender | Empfänger | Adressierung | Payload‑Bedeutung |
|---|---|---|---|---|---|
| 0x01 | `DISCOVER_REQ` | Config‑Box | alle (Filter `device_type`) | Broadcast | `payload[0]` = gewünschter `device_type` Filter |
| 0x02 | `DISCOVER_REPLY` | Station / Target | Config‑Box | Unicast an Absender | siehe 3.6.1 |
| 0x03 | `IDENTIFY` | Config‑Box | adressiertes Gerät | Unicast | `payload[0]` = Dauer in 100 ms (Default 7 = 700 ms) |
| 0x10 | `HIT_REPORT` | Target | Stationen | Broadcast | `payload[0]` = `sound_id` (Absender‑MAC kommt ohnehin im ESP‑NOW‑recv‑Callback mit) |
| 0x20 | `SETUP_BEGIN` | Config‑Box | alle Stationen | Broadcast | `payload[0]` = Timeout in Sekunden (Default 60) |
| 0x21 | `SETUP_TAKE` | Station | alle | Broadcast | `payload[0]` = neue `station_id` (Absender‑MAC kommt aus dem ESP‑NOW‑recv‑Callback) |
| 0x30 | `CFG_WRITE` | Config‑Box | Station oder Target | Unicast | siehe 3.6.2 / 3.6.3 |
| 0x31 | `CFG_ACK` | Station / Target | Config‑Box | Unicast | `payload[0]` = Status (0 = OK, 1 = NACK Persistierung, 2 = NACK Validierung) |
| 0x32 | `CFG_TEST_SOUND` | Config‑Box | Station | Unicast | `payload[0]` = `sound_id` – nur abspielen, **nicht** persistieren. Festgelegt 2026‑07‑08 (vorher offener Punkt in Doc 18 § 6.3) |
| 0xC0 | `IR_SELECT_ECHO` | Target | Config‑Box | Broadcast | `payload[0]` = Token aus IR‑Frame (Echo‑Schutz). Reserviert/latent – siehe 3.7. |

Reservierte Bereiche:

- `0x40..0x4F` – zukünftige Read‑Befehle, falls Discovery + Reply für Reads
  nicht reicht (z. B. Live‑Sensor‑Werte)
- `0x80..0xBF` – zukünftige Telemetry / Heartbeats
- `0xF0..0xFF` – Debug / OTA

### 3.6 Payload‑Layout pro Nachrichtentyp

#### 3.6.1 `DISCOVER_REPLY` (0x02)

```
payload[0]      fw_version_major
payload[1]      fw_version_minor
payload[2]      fw_version_patch
payload[3]      rssi_self  (eigener Empfangs‑RSSI des Discover_Req, int8)
payload[4..5]   uptime_min  (uint16, in Minuten)
payload[6]      config_blob_len  (Anzahl folgender Bytes mit Config, max 19)
payload[7..25]  config_blob      (typabhängig, siehe 3.6.2/3.6.3, bis zu 19 Byte)
```

`device_type` im Header sagt, ob `config_blob` als Station‑ oder
Target‑Config zu interpretieren ist. Die Absender‑MAC steht ohnehin im
ESP‑NOW‑Layer‑2‑Header und wird beim `recv_cb` mit übergeben – sie
muss nicht zusätzlich im Payload mitfahren.

#### 3.6.2 Station‑Config‑Blob (`CFG_WRITE` mit `device_type=1`)

| Offset | Feld | Typ | Default | Bemerkung |
|---|---|---|---|---|
| 0 | `station_id` | uint8 | 1 | 1..99, im NVS persistiert |
| 1 | `volume_pct` | uint8 | 80 | 0..100, Software‑Skalierung der I²S‑Samples |
| 2 | `default_setup_sound` | uint8 | 13 | Sound‑Index, beim Trigger im Setup‑Mode zur Bestätigung gespielt |
| 3..15 | reserved | – | 0 | für künftige Felder, Sender setzt 0 |

#### 3.6.3 Target‑Config‑Blob (`CFG_WRITE` mit `device_type=2`)

| Offset | Feld | Typ | Default | Bemerkung |
|---|---|---|---|---|
| 0 | `target_id` | uint8 | 1 | 1..99 |
| 1 | `station_id` | uint8 | 1 | welche Station bedienen (Filter im `HIT_REPORT`) |
| 2 | `sound_id` | uint8 | 1 | Sound‑Index für `HIT_REPORT` |
| 3..4 | `hit_time_ms` | uint16 | 10000 | wie lange Switches AN / Hit‑LED‑Animation läuft |
| 5..6 | `cooldown_ms` | uint16 | 2000 | nach Hit: weitere Treffer ignorieren, LED‑Ring schwarz |
| 7 | `sw_animation` | uint8 | 0 | Pattern‑Index (heute 0 oder 1, später mehr) |
| 8 | `sw_channels` | uint8 | 0b00000111 | Bitmaske: bit0 = SW1 (potentialfrei), bit1 = SW_5V, bit2 = SW_3V3 |
| 9..15 | reserved | – | 0 | für künftige Felder |

Das Feld `cooldown_ms` ist neu gegenüber dem heutigen Target‑Code – siehe
[Doc 11 / Cooldown‑Punkt](11-offene-punkte.md). Das Feld `sw_channels` ersetzt
die heutige Hartverdrahtung „alle drei Switches gleichzeitig"
([`08-software-target.md`](08-software-target.md) Z. 81–84).

### 3.7 Flows: Spielbetrieb, Setup, Konfiguration

Drei Abläufe nutzen alle das gleiche Paket‑Format:

**1. Spielbetrieb (regulärer Treffer):**

```
Target ──(IR‑Treffer empfangen)──► HIT_REPORT (Broadcast, station_id=X, sound_id=Y)
                                       │
                                       ▼
                          Stationen mit station_id == X spielen Sound Y
```

Latenz‑Zielwert: < 50 ms vom IR‑Decode bis Sound‑Start. ESP‑NOW selbst
liefert in < 10 ms; der Rest geht für Decode + LittleFS‑Open + I²S‑Anlauf
drauf.

**2. Stations‑ID setzen per Stab‑Trigger:**

```
Config‑Box ──SETUP_BEGIN(timeout=60s)──► alle Stationen
                                              │
                                              ▼
                            Stationen: Status‑LED + Stab‑LEDs lila
                                              │
                                  Tobias drückt Trigger an Wunsch‑Station
                                              │
                                              ▼
Wunsch‑Station ──SETUP_TAKE(mac=…, new_id=03)──► Broadcast
                                              │
                                              ▼
              Andere Stationen sehen SETUP_TAKE → zurück IDLE
              Wunsch‑Station persistiert ID, blinkt 3× Cyan
              Config‑Box sieht SETUP_TAKE → Menü „Setup abgeschlossen"
```

**3. Konfiguration per Discovery + Identify‑Blink (Standardweg):**

```
Config‑Box ──DISCOVER_REQ(device_type=TARGET)──► Broadcast
                                              │
                          Targets antworten je mit DISCOVER_REPLY (Unicast)
                                              │
                                              ▼
                        Config‑Box baut Liste auf OLED auf
                                              │
                          Tobias dreht Encoder → Cursor auf Eintrag K
                                              │
              Config‑Box ──IDENTIFY(mac=Target_K, dur=700ms)──► alle 500 ms
                                              │
                                Target_K blinkt LED‑Ring weiß
                                              │
                          Tobias drückt Encoder‑Push → Edit‑Modus
                                              │
              Config‑Box ──CFG_WRITE(target_blob)──► Target_K (Unicast)
                                              │
                                Target_K persistiert in NVS
                                              │
              Target_K ──CFG_ACK(status=OK)──► Config‑Box (Unicast)
                                              │
                              Config‑Box: OLED „Gespeichert"
```

**4. IR‑Select (latent / Reserve, in V1 nicht produktiv):**

Der `IR_SELECT_ECHO`‑Befehl ist in der Protokolltabelle reserviert, damit
das Target‑Firmware‑Skelett von Anfang an einen Hook hat, falls wir in
einer späteren Halloween‑Saison doch noch einen IR‑Pointer am
Config‑Tool nachziehen wollen (z. B. „Welche von drei verbauten Targets
ist Target 7?"). Im aktuellen Config‑Tool ist diese Funktion **nicht
ausgebaut** – siehe [`18-config-tool.md`](18-config-tool.md) Abschnitt
„IR‑Pointer". Die Code‑Stelle im Target ist minimal: bei `isSystem=true,
cmd=0xC0` ein `IR_SELECT_ECHO` broadcasten und gut.

### 3.8 Peer‑Management auf der ESP‑NOW‑API

ESP‑NOW verlangt, dass Unicast‑Empfänger vorher per `esp_now_add_peer()`
registriert werden. Empfehlung für alle drei Gerätetypen:

- **Broadcast‑Peer dauerhaft eingetragen** (MAC = FF:FF:FF:FF:FF:FF), Kanal 1.
- **Peer‑Cache mit max. 20 Einträgen**, LRU. Wann eintragen:
  - Beim Empfang eines Pakets von einer unbekannten MAC (für eventuellen
    Reply).
  - Beim Senden eines Unicasts (vor dem Send).
- **Wenn Cache voll**: ältesten Eintrag verdrängen (`esp_now_del_peer()`,
  dann `esp_now_add_peer()` für den neuen).
- Kein WiFi‑Connect nötig, `WiFi.mode(WIFI_STA)` reicht. Aber:
  `WiFi.disconnect()` einmal direkt nach `WiFi.mode()` aufrufen, sonst
  versucht der ESP eine alte SSID zu joinen und blockiert für ein paar
  Sekunden.

```cpp
WiFi.mode(WIFI_STA);
WiFi.disconnect();        // kein STA‑Connect erwünscht
WiFi.setSleep(WIFI_PS_NONE);
esp_now_init();

esp_now_peer_info_t bcast = {};
memset(bcast.peer_addr, 0xFF, 6);
bcast.channel = 1;
bcast.encrypt = false;
esp_now_add_peer(&bcast);
```

### 3.9 Sicherheit / Verschlüsselung

ESP‑NOW kann optional **AES‑CCM** mit einem 16‑Byte PMK + 16‑Byte LMK pro
Peer. Für ein Halloween‑Setup im Vorgarten **nicht aktiviert** – kostet
Code‑Komplexität und Konfigurations‑Aufwand (Schlüssel müssen auf allen
Geräten identisch im NVS liegen), bringt aber keinen relevanten Schutz
gegen Reverse‑Engineering, dem das System gar nicht ausgesetzt ist.
Bleibt offen, falls das System mal auf einem Lasertag‑Event neben anderen
2,4‑GHz‑Funkern stehen soll.

---

## 4. Audio‑Format und Klang

### 4.1 WAV vs. MP3

WAV ist verlustfrei. Bei gleicher Sample‑Rate ist WAV klanglich grundsätzlich
besser als MP3.

| Sample‑Rate | obere Frequenzgrenze | Datenrate Mono 16‑Bit | Klangcharakter |
|---|---|---|---|
| 8 kHz | ~3,5 kHz | 16 KB/s | Telefonqualität |
| 16 kHz | ~7 kHz | 32 KB/s | UKW‑Radio mau |
| **22 kHz** | **~10 kHz** | **44 KB/s** | **AM‑Radio plus, ausreichend für Halloween** |
| 32 kHz | ~14 kHz | 64 KB/s | UKW‑Radio gut |
| 44,1 kHz | ~20 kHz | 88 KB/s | CD‑Qualität (Mono) |

Halloween‑Sounds (Werwolf, Geist, Skelett, Hexe) liegen im Bereich
**50 Hz – 5 kHz**. **22 kHz Mono 16‑Bit ist klanglich völlig in Ordnung.**

Mit 8–10 MB LittleFS‑Platz wäre auch **44,1 kHz** drin (88 KB/s × 30 Sek
≈ 2,6 MB). Wenn wir in den Hörtests merken, dass 22 kHz „zu mau" klingt,
können wir hochrampeln, ohne Hardware‑Änderung.

### 4.2 Aufbereitung der Halloween‑MP3s

Einmalig in Audacity pro File:

1. Mono‑Mix.
2. Bass‑EQ: +3 dB Bell‑Filter @ 100 Hz, Q = 0,7.
3. Loudness‑Normalisierung auf −14 LUFS (gleichmäßige Lautstärke aller Files).
4. Resample auf 22 kHz (oder 44,1 kHz, falls Hörtest dafür spricht).
5. Export als 16‑Bit‑WAV PCM, Dateiname `NN_kurzname.wav`.

ID‑Mapping (1‑basiert vs. 0‑basiert) endgültig entscheiden – siehe Punkt 1a
in [`11-offene-punkte.md`](11-offene-punkte.md).

### 4.3 Aktuelle Test‑Sound‑Bibliothek (Stand 2026‑05‑16)

Die WAV‑Bibliothek liegt im **aktiven PlatformIO‑Projekt unter**
`/Volumes/basteln/Infinitag/Station/data/` und wird per
`pio run -t uploadfs` (bzw. nach jedem `Upload` automatisch via
`scripts/upload_fs_after_upload.py`) ins LittleFS geschoben. Quellen: alle
14 Halloween‑MP3s aus
`/Volumes/basteln/Halloween/Lasertag/StationSounds/sound-effects/`,
konvertiert mit `ffmpeg -ac 1 -ar 22050 -sample_fmt s16 -c:a pcm_s16le`.

> ⚠️ **Test‑Konvertierung, nicht final.** Diese Files sind nur eine
> Pass‑Through‑Konvertierung (Mono, 22 kHz, 16‑Bit). Die in 4.2 beschriebene
> Audacity‑Aufbereitung (Bass‑EQ +3 dB @ 100 Hz, LUFS‑Normalisierung auf
> −14 LUFS) ist **noch nicht** angewandt – das passiert erst, wenn der
> Hardware‑Klang im Lochraster geprüft ist und wir die End‑Kette kennen.

> 📝 **Hinweis Pfad‑Migration (2026‑05‑16):** Es gibt zusätzlich noch das
> verwaiste Stub‑Projekt unter
> `/Volumes/basteln/PlatformIo/Projects/InfinitagStation/`. Dort liegt
> nur der erste Audio‑Stub vom 12.05. (Plan‑B / STBY). **Aktiv ist
> ausschließlich `/Volumes/basteln/Infinitag/Station/`** mit OLED,
> NeoPixel, IR, Laser, Plan‑A‑Mute via XSMT.

| ID | Dateiname                     | Quelle (`.mp3`)              | Dauer ≈ | Größe |
| -: | ----------------------------- | ---------------------------- | ------: | ----: |
|  1 | `01_test.wav`                 | (eigenes Test‑Sample)        |  4.3 s | 184 KB |
|  2 | `02_door_bang.wav`            | (eigene Aufbereitung)        |  2.4 s | 103 KB |
|  3 | `03_boo_and_laugh.wav`        | `0_booAndLaugh`              |  3.5 s | 153 KB |
|  4 | `04_bubbles.wav`              | `1_bubbles`                  |  7.5 s | 326 KB |
|  5 | `05_cat_meow.wav`             | `2_catMeow`                  |  1.6 s |  69 KB |
|  6 | `06_daemon_kinderliebe.wav`   | `3_daemonIchLiebeKinder`     | 17.9 s | 771 KB |
|  7 | `07_gears.wav`                | `5_gears`                    |  8.8 s | 381 KB |
|  8 | `08_little_girl.wav`          | `6_littleGirl`               |  7.4 s | 322 KB |
|  9 | `09_owl_hooting.wav`          | `7_owlHooting`               |  3.6 s | 154 KB |
| 10 | `10_psycho_sound.wav`         | `8_psychoSound`              |  5.4 s | 232 KB |
| 11 | `11_scary_clock.wav`          | `9_scaryClock`               | 11.3 s | 485 KB |
| 12 | `12_spooky_skeleton.wav`      | `10_spookyScarySkeleton`     |  7.5 s | 325 KB |
| 13 | `13_werewolf.wav`             | `11_werewolf`                |  4.3 s | 184 KB |
| 14 | `14_werewolf_growl.wav`       | `12_werewolfGrowl`           |  8.8 s | 380 KB |
| 15 | `15_witch.wav`                | `13_witch`                   |  8.1 s | 349 KB |

> `4_doorBang.mp3` wurde **nicht** zusätzlich konvertiert – die schon
> aufbereitete `02_door_bang.wav` bleibt der kanonische Door‑Bang‑Sound.

**Speicherverbrauch:** 15 WAVs ≈ **4,4 MB** in der 10,4 MB‑LittleFS‑Partition
→ rund **5 MB frei** für weitere Sounds, OTA‑Daten oder Spell‑Loops.

**Sound‑Auswahl im Test‑Sketch (`main.cpp`):**

Im echten Station‑Test‑Sketch sind die Tasten‑Belegung und Trigger‑Logik
schon weiter als im ursprünglichen Audio‑Test. Stand 2026‑05‑16 (Bring‑up):

| Taste     | GPIO | Funktion |
| --------- | ---: | -------- |
| K1        | 35   | **Volume cyclen** (`gVolume += VOLUME_STEP`, am Maximum zurück auf `VOLUME_MIN`) |
| K2        | 36   | **Sound cyclen** (`gSoundIdx +1 mod 15`, nur Auswahl – KEIN Abspielen) |
| K3        | 37   | Laser‑Toggle (active‑HIGH auf GPIO 7 → BC546C → Laser) |
| K4        | 38   | **Vorhören**: aktuell ausgewählten Sound abspielen, ohne IR‑Burst |
| Trigger   |  4   | Stab‑Trigger → IR‑Burst + aktuell ausgewählten Sound abspielen |

Test‑Ablauf am Tisch: K1 stellt die Lautstärke ein, K2 wählt den Sound aus,
dann entweder K4 (lokales Vorhören) oder Trigger (echter „Schuss" mit IR).

Im Display (SH1106 128×64) wird der aktuell ausgewählte Sound in
Zeile y=49 angezeigt, Format `S<NN>:<KurzName>` (z. B. `S04:Bubbles`). Der
Footer zeigt `K1V+ K2S+ K3L K4Play` als Bedienhinweis. Der Speaker‑Name
steht nur noch im Boot‑Log – im Live‑Betrieb interessiert der aktuell
ausgewählte Sound mehr.

Im Produktiv‑Sketch (nach Bring‑Up) übernimmt das OLED‑Menü die Auswahl,
und die Sound‑ID kommt per ESP‑NOW vom Wand‑Stab (siehe Abschnitt 3).

---

## 5. Software‑Konventionen

Auf dem **Dual‑Core S3** sind die folgenden Konventionen weniger existenziell
als auf dem C3, bleiben aber als **Best Practice** gesetzt – damit die
Software auch auf einem späteren Single‑Core‑Modul (falls wir die Wand
selbst noch refactoren) funktioniert.

### 5.1 Audio‑Task auf Core 1, WiFi auf Core 0

```cpp
xTaskCreatePinnedToCore(audioTask, "audio", 4096, NULL, 5, NULL, 1);
// WiFi/ESP-NOW läuft per Default auf Core 0
```

Damit hat Audio‑DMA‑Refill einen eigenen Core und konkurriert nicht mit dem
WiFi‑Subsystem.

### 5.2 WiFi‑Sleep aus

```cpp
WiFi.mode(WIFI_STA);
WiFi.setSleep(WIFI_PS_NONE);
```

### 5.3 Logging im Audio‑Pfad

`Serial.println()` in Setup‑Phase und Boot ist OK. Im Hot‑Path
(Audio‑Loop, Trigger‑Handler, ESP‑NOW‑Callback) bleibt es draußen.
Compile‑Time‑Schalter `#define INFINITAG_DEBUG 0/1`. In Production:
Events in `xQueue`, Low‑Prio‑Task gibt sie auf `Serial` aus.

### 5.4 NeoPixel mit RMT‑Backend

`Adafruit_NeoPixel` deaktiviert für Bit‑Banging die Interrupts – auf S3
weniger schlimm als auf C3, aber bleibt vermeidbar. Verwenden:

- `FastLED` mit RMT‑Backend, **oder**
- ESP‑IDF‑Komponente `led_strip` direkt.

### 5.5 Audio‑Mute‑Strategie (Variante B – Software‑Mute)

**Hintergrund:** Bei nur 20 cm Hörabstand (Kinder stehen direkt vor der Box)
ist jedes Class‑D‑Restrauschen, Power‑On‑Pop oder I²S‑Disconnect‑Knack
hörbar. Wir nutzen daher eine **zweistufige Mute‑Strategie**, die **ohne
Hardware‑Modifikation am DAC** auskommt:

**Stufe 1: Continuous I²S‑Stream**

Der I²S‑Output‑Stream läuft **dauerhaft**, von Boot bis Power‑Off. Niemals
`i2s_stop()`. Wenn kein Sound spielt, schreiben wir **Null‑Samples (Stille)**
in den DMA‑Buffer. Dadurch:

- Der PCM5102 verliert nie den I²S‑Lock → kein Disconnect‑Knack
- Der DAC gibt sauber 0 V (Stille) raus, ohne Driften

**Stufe 2: TPA3110 STBY‑Pin software‑gesteuert**

Der TPA3110 hat einen STBY‑Pin (auf der Header‑Leiste des XH‑A232 zugänglich),
der den Verstärker komplett abschaltet. Wir steuern ihn vom S3 mit einem
GPIO an:

- Boot‑Default: GPIO LOW → STBY aktiv → Verstärker aus → kein Power‑On‑Pop
- Sound‑Start: GPIO HIGH → STBY inaktiv → Verstärker fadet sanft hoch (~50 ms)
- Sound‑Ende: GPIO LOW → STBY aktiv → Verstärker aus, kein Idle‑Hiss

**Sequenz im Code:**

```cpp
void playSound(const char* path) {
  startWavStream(path);        // I²S spielt nun WAV‑Samples
  delay(10);                   // ~10ms warten bis Stream stabil
  digitalWrite(STBY_PIN, HIGH); // Verstärker an, fadet hoch
  // ... Sound läuft ...
  digitalWrite(STBY_PIN, LOW);  // Verstärker aus, fadet ab
  delay(10);
  switchWavToNullSamples();    // I²S spielt jetzt Stille‑Samples
}
```

**Resultat:** im Standby vollständige Stille, beim Sound‑Start kein Pop,
beim Sound‑Ende kein Knack, beim laufenden Sound nur das vom Sound
überdeckte Class‑D‑Restrauschen.

#### Eskalations‑Plan: Variante A (Brücken‑Modifikation am PCM5102)

Falls Variante B im Lochraster‑Test bei sehr leisen Stellen im Sound noch
hörbares Hiss durchlässt, lässt sich der DAC zusätzlich gemutet bekommen,
indem die Werks‑Brücke H3 (XSMT auf 3,3 V) auf der Modul‑Rückseite getrennt
und der XSMT‑Pin auf einen GPIO geführt wird:

- GPIO HIGH → DAC aktiv mit Soft‑Fade‑In
- GPIO LOW → DAC mutet sanft (eingebauter Soft‑Fade‑Out)

Der GPIO **GPIO12** ist im Plan v7 als „Reserve / optional XSMT" reserviert.
Wenn Plan A nötig wird, einfach den Pin verkabeln und einen zusätzlichen
Mute‑Schritt vor dem STBY in der Sequenz oben einbauen.

#### Eskalations‑Plan C: NJM2761 als Hardware‑Mute‑IC

Falls auch A+B noch zu hörbar ist (sehr unwahrscheinlich): ein
**NJM2761** zwischen DAC‑Output und Verstärker‑Input schaltet das Audio‑
Signal vollständig auf GND. Dediziertes Audio‑Mute‑IC, ~3 €, in der BOM
als Plan‑C‑Reserve notiert.

### 5.6 Bypass‑Kondensatoren für die Audio‑Kette (PCB‑Hinweise)

**Erkenntnisse aus dem Lochraster‑Test (2026‑05‑12):** Auf dem Lochraster
koppeln die ca. 15–20 cm langen I²S‑Jumper‑Drähte (BCLK, LRC, DOUT) ihre
Schaltflanken in die analoge Leiterstrecke zwischen PCM5102A‑Ausgang und
TPA3110‑Eingang. Das äußert sich als hochfrequentes Rauschen in den
Ruhephasen trotz aktiver XSMT‑Mute. Auf dem finalen PCB lässt sich das
durch kurze Leiterbahnen, eine durchgehende Massefläche und gezielte
Bypass‑Kondensatoren vollständig vermeiden.

**Empfohlene Bypass‑Kondensatoren für das PCB:**

| Bauteil | Wert | Typ | Platzierung | Funktion |
|---|---|---|---|---|
| C_PCM_VCC_A | **100 nF** | Keramik (X7R, 0402 oder 0603) | direkt an PCM5102A‑Modul‑Stiftleiste: VCC gegen GND, so nah wie möglich am Pin | Filtert I²S‑Schaltspitzen (~700 kHz), die in die 3,3‑V‑Rail des DAC koppeln |
| C_PCM_VCC_B | **10 µF** | Elko oder Tantal, 10 V | parallel zu C_PCM_VCC_A (gleiche Stiftleiste) | Bulk‑Kapazität für Transienten beim DAC‑Aktivieren |
| C_TPA_VCC_A | **100 nF** | Keramik (X7R) | direkt an TPA3110‑Modul‑Stiftleiste: VCC (12 V) gegen GND | Entkoppelt die 12‑V‑Rail des Class‑D‑Verstärkers |
| C_TPA_VCC_B | **100 µF** | Elko, 25 V | parallel, nah am TPA3110‑Eingang | Puffert Pegel‑Peaks des Class‑D‑Ausgangs |
| C_LINE_A | **100 nF** | Keramik | in der analogen Line‑Leitung zwischen PCM5102A‑OUT und TPA3110‑IN (gemeinsam als RC mit einem 10 Ω Serienwiderstand) | Filtert HF‑Einstreuungen auf der analogen Signal‑Strecke |

**Layout‑Regeln für das PCB:**

- **Digitale (I²S) und analoge Leiterbahnen physisch trennen:** I²S‑Bahnen
  (BCLK, LRC, DOUT) auf der Leiterplatte weit von der analogen Signal‑Strecke
  (PCM5102A‑OUT → TPA3110‑IN) wegführen, möglichst auf unterschiedlichen
  Lagen oder zumindest mit Masse‑Polygon dazwischen.
- **Massefläche (Polygon Fill GND)** auf beiden Lagen: reduziert Schleifen
  und gibt den Bypass‑Kondensatoren einen niederohmigen Rückpfad.
- **Kurze Leiterbahnen zu den Modulen:** Stiftleisten‑Footprints möglichst
  direkt neben den Modul‑Pads, nicht über lange Bahnen geführt.
- **Analoge Masse (AGND) vom PCM5102A‑Modul** sternförmig zum Haupt‑GND
  führen, nicht durch das Digitalteil.

> **Hinweis:** Die Bypass‑Kondensatoren auf den fertigen Modul‑Platinen
> (GY‑PCM5102, XH‑A232) sind bereits vorhanden. Die obigen Empfehlungen
> beziehen sich auf **zusätzliche Kondensatoren am Übergang
> Modul‑Stiftleiste → Hauptplatine**, um die Verbindungsleitungen zwischen
> den Modulen zu entkoppeln.

### 5.8 DMA‑Buffer für I²S großzügig

Mit PSRAM auf dem S3 dürfen die DMA‑Buffer ruhig groß ausfallen.

| Parameter | Wert |
|---|---|
| `dma_buf_count` | 8 |
| `dma_buf_len` | 1024 |
| Audio pro Buffer (22 kHz Mono) | ~46 ms |
| Reserve gegen CPU‑Stalls | ~370 ms |
| RAM‑Verbrauch | 16 KB (kein Problem mit 512 KB SRAM + 8 MB PSRAM) |

```cpp
i2s_config_t i2s_cfg = {
  .mode = I2S_MODE_MASTER | I2S_MODE_TX,
  .sample_rate = 22050,
  .bits_per_sample = I2S_BITS_PER_SAMPLE_16BIT,
  .channel_format = I2S_CHANNEL_FMT_ONLY_LEFT,
  .communication_format = I2S_COMM_FORMAT_STAND_I2S,
  .intr_alloc_flags = ESP_INTR_FLAG_LEVEL1,
  .dma_buf_count = 8,
  .dma_buf_len = 1024,
  .use_apll = true,
  .tx_desc_auto_clear = true
};
```

`use_apll = true` aktiviert die Audio‑PLL für saubere Sample‑Clock.

### 5.9 NeoPixel‑Anzahl

Mit S3 (8 RMT‑Channels, Dual‑Core) sind LED‑Zahlen weit über 100 problemlos
machbar. Für unseren Use‑Case (1 Status + 2 Stab = 3 LEDs) absolut
unkritisch.

---

## 6. Konzeptioneller Datenfluss

```
┌─────────────────────────────────────────────────────────────────┐
│                     WAND STATION V2 (S3)                        │
│                                                                 │
│  OLED + 4 Tasten ──I²C──┐                                       │
│  (hinter Service‑Klappe)│                                       │
│                         ▼                                       │
│  Trigger ◄──Stab‑Kabel──┐                                       │
│                          ESP32‑S3‑DevKitC‑1U                    │
│  IR‑LED ◄────────────────┤  (Core 0: WiFi+ESP‑NOW)             │
│  Laser ◄─────────────────┤  (Core 1: Audio I²S)                 │
│  Status‑LED ◄──┐         │                                      │
│  Stab‑LEDs  ◄──┴NeoPixel─┤                                      │
│                          │                                      │
│  WAV (LittleFS) ─────────┴── I²S ──► PCM5102A ──► TPA3116D2 ──► │
│                                                                 │
│  SMA‑Antenne ◄─Pigtail── IPEX                                   │
│                          │                                      │
│  12 V Eingang ─► Buck‑Down ─► 5 V intern, 12 V → Verstärker     │
└─────────────────────────────────────────────────────────────────┘
                          │ ESP‑NOW
                          │ {station_id, sound_id, target_id}
              ┌───────────┴──────────┐
              │       Targets        │
              │ (ESP32‑S3, IR‑Empf., │
              │  LED‑Ring, Switches) │
              └──────────────────────┘
```

---

## 7. GPIO‑Plan v7

> Status: **festgeschrieben in der Konzeptphase, im Lochraster zu
> verifizieren**. Pinwahl orientiert sich an den frei nutzbaren GPIOs des
> ESP32‑S3‑DevKitC‑1U und vermeidet Strapping/USB‑Pins (GPIO0, GPIO3,
> GPIO45, GPIO46, GPIO19/20).

| GPIO | Funktion | Anmerkung |
|---|---|---|
| **GPIO4** | Trigger‑Eingang vom Stab | Input mit internem Pullup |
| **GPIO5** | IR‑LED‑Steuersignal (RMT, 38 kHz Träger) | **3,3 V‑Logiksignal direkt zum Stab** – MOSFET sitzt im Stab (Doc 13), nicht im Station‑PCB. Entscheidung 2026‑05‑14 |
| **GPIO6** | NeoPixel‑Daten (RMT) | Daisy‑Chain: Status‑LED → Stab‑LEDs; in der Station über 74AHCT125 auf 5 V geshiftet |
| **GPIO7** | Laser‑Steuersignal | **3,3 V‑Logiksignal direkt zum Stab** – MOSFET sitzt im Stab (Doc 13), nicht im Station‑PCB. Entscheidung 2026‑05‑14 |
| **GPIO8** | I²C SDA (zum OLED) | |
| **GPIO9** | I²C SCL (zum OLED) | |
| **GPIO11** | TPA3110 STBY (Verstärker‑Mute) | Output, LOW = Standby/Mute, HIGH = aktiv |
| **GPIO12** | Reserve / optional PCM5102 XSMT (Plan A) | Output, nur belegt falls Variante A nachgezogen wird |
| **GPIO15** | I²S BCLK | zum PCM5102A |
| **GPIO16** | I²S LRC | zum PCM5102A |
| **GPIO17** | I²S DOUT | zum PCM5102A |
| **GPIO35** | OLED‑Taste K1 (Menü/Zurück) | Input mit internem Pullup |
| **GPIO36** | OLED‑Taste K2 (Auf) | Input mit internem Pullup |
| **GPIO37** | OLED‑Taste K3 (Ab) | Input mit internem Pullup |
| **GPIO38** | OLED‑Taste K4 (OK) | Input mit internem Pullup |

**Bilanz:** 14 Pins fest genutzt + 1 Reserve‑Pin (GPIO12 für optionales
XSMT) = 15 Pins, **~23 Pins weiter frei** für Erweiterungen
(Bluetooth‑Audio, weitere Sensoren, zweiter NeoPixel‑Strang, ADC für
Spannungsmessung am 12‑V‑Eingang etc.).

**Verifikation GPIO 35–38 vs. Octal PSRAM (2026‑05‑14):**

Auf dem ESP32‑S3‑WROOM‑1U mit `R8`‑Suffix (Octal PSRAM) sind die internen
SPI‑Octal‑Leitungen typischerweise GPIO 33–37. Es bestand der Verdacht,
dass die im Plan v7 für K1–K3 vorgesehenen Pins (GPIO 35/36/37) dadurch
am DiYmore‑Board nicht frei als Input nutzbar sind. Lochraster‑Test mit
dem aktuellen `Station/`‑Sketch (`pinMode(35/36/37/38, INPUT_PULLUP)` +
periodisches Roh‑Logging) hat gezeigt: **alle vier Pins stehen stabil
HIGH** und schalten beim Tastendruck sauber auf LOW. Das konkrete Board
verwendet PSRAM also nicht im Octal‑Mode bzw. die Pins sind herausgeführt
und nutzbar. K1=35, K2=36, K3=37, K4=38 wird **so beibehalten**.
Verifikation wiederholen, falls eine andere Lieferantenrevision verbaut
wird.

**Beschaltungs‑Hinweise:**

- **Trigger (GPIO4):** Stab‑Trigger zieht gegen GND, internen Pullup
  aktivieren (`pinMode(4, INPUT_PULLUP)`).
- **IR‑LED + Laser (GPIO5/7):** ~~lokaler MOSFET-Schalter im Station‑PCB~~ →
  **am 2026‑05‑14 verworfen**. Die zwei Pins gehen direkt als 3,3 V-
  Logiksignal auf den 6‑pol Stab‑Klemmstecker (Pin 5/6). Der **MOSFET-Schalter
  sitzt im Stab-PCB** (siehe Doc 13). Begründung: 100 mA mit 38 kHz Modulation
  würden 1,5 m Kabel als Antenne nutzen und zusätzlich durch
  Kabelwiderstand 30 mV abfallen → schlechtere IR-Reichweite + EMV.
- **NeoPixel (GPIO6):** 3,3 V vom S3 reicht oft direkt für SK6812 bei kurzer
  Leitung. Bei ≥ 5 LEDs oder längerer Leitung **Level‑Shifter** (74AHCT125)
  einplanen.
- **I²C (GPIO8/9):** externe 4,7 kΩ Pullups gegen 3,3 V (manche OLED‑Module
  haben die schon onboard – prüfen).
- **OLED‑Tasten (GPIO35–38):** schalten gegen GND, interne Pullups
  aktivieren. Bei langer Leitung 100 nF Entstörkondensator gegen GND.
- **TPA3110 STBY (GPIO11):** direkt mit dem STBY‑Pin der XH‑A232‑Header‑Leiste
  verbunden. Standby = Verstärker komplett aus. Boot‑Default LOW (= Standby),
  damit beim Power‑On kein Pop entsteht.
- **PCM5102 XSMT‑Reserve (GPIO12):** im Plan B unbeschaltet. Falls Plan A
  („Brücken‑Modifikation des H3‑Lötjumpers") später nachgezogen wird, geht
  von hier ein Drahtanschluss zum XSMT‑Pin der Header‑Leiste.

---

## 8. Mechanik: Boxen, Module, Anschlüsse, Stab‑Connector

### 8.1 Box‑Konzept: Vergleichsbau zweier Speaker‑Kammern

Zwei Boxen entstehen **parallel** für einen A/B‑Hörtest. Box B nutzt den
vorhandenen **Visaton FR 10/4** (Art. 2020 – Etikettmaße 2026‑05‑13 bestätigt).
Ein früherer Zwischenschritt hatte den FR 10/4 auf einen FRS 8 geändert (2026‑05‑08),
weil das damalige Zielvolumen von 1,1 l für den FR 10/4 zu klein gewesen wäre
(Visaton empfiehlt ≥ 2,5 l). Mit dem neu ausgelegten Volumen von 2,0 l Bassreflex
ist der FR 10/4 wieder die erste Wahl und der FRS 8 entfällt.

| | **Box A** | **Box B** |
|---|---|---|
| Lautsprecher | Visaton FR 7/4 (vorhanden) | Visaton FR 10/4 (Art. 2020, vorhanden) |
| Auslegung | geschlossen | Bassreflex |
| Innenkammer netto | 0,5 l | 2,0 l |
| f₃ (untere Grenzfreq.) | ~270 Hz | ~80–85 Hz |
| Klangcharakter | knackige Mitten, kein Tiefbass | voller Mittelton, deutlicher Bass‑Hügel (f₀ ~90 Hz, 30 W RMS, 4 Ω) |
| Ausgangsbasis | aktuelle 3D‑Druck‑Box 106 × 84 × 66 mm | leicht größer, Maße in 8.3 |
| Grund für Vergleich | reicht der vorhandene Speaker mit den neuen Komponenten? | rechtfertigen +13 € + größere Box den Klanggewinn? |

Beide Boxen werden am gleichen TPA3110/PCM5102‑Aufbau betrieben, damit
der einzige Unterschied der Speaker und das Gehäuse ist – sauberer
A/B‑Vergleich.

> **⚠️ Gain‑Warnung Box A (2026‑05‑10):** Der TPA3110 kann bei 12 V in
> 4 Ω bis zu ~12–15 W liefern. Der Visaton FR 7/4 ist nur für **5 W RMS**
> ausgelegt – Vollgas würde ihn zerstören. Vor dem ersten Test den
> **Gain‑Trimmer / Gain‑Jumper am TPA3110‑Modul** so einstellen, dass
> die Ausgangsleistung auf ≤ 4 W begrenzt bleibt. Box B (FR 10/4, 30 W, 4 Ω)
> hat dieses Problem nicht.

> **Wichtige Erkenntnis (2026‑05‑08):** Der aktuelle Klang („mist") kommt
> zu großen Teilen vom MAX98357 mit nur 3,3 V VIN (max ~0,7 W) und der
> akustisch nicht ausgelegten Verteilerkasten‑Box, **nicht** primär vom
> Speaker. Mit dem Refactor (TPA3110 + 12 V → ~10× mehr Pegel,
> dichte 3D‑Druck‑Box, Mute‑Strategie) holen wir auch mit dem **gleichen
> FR 7/4** etwa 70–80 % des hörbaren Sprungs raus. Box B ist nur dann
> nötig, wenn dem FR 7/4 nach diesem Sprung immer noch der Bass fehlt.

### 8.2 Speaker‑Kammer Box A (FR 7 / FR 7‑4Ω geschlossen)

> **Speaker verifiziert 2026‑05‑09:** Visaton FR 7, Art.‑Nr. 2015 (4 Ω),
> Korbflansch quadratisch 66 × 66 mm mit abgerundeten Ecken, Lochkreis
> Ø 74 mm, 4 Schraubenlöcher Ø 4,2 mm, Korbflansch‑Dicke ~1,7–2 mm,
> Gesamttiefe Korb+Magnet 61 mm, Korb‑Vorderring Ø 64 mm × ~2 mm hoch,
> Frequenzgang (−10 dB) 130–20.000 Hz. Maße aus Datenblatt + Schieblehre
> bestätigt.

> **Montage‑Variante: Innen‑Montage (gesetzt 2026‑05‑09).** Speaker wird
> von der Speaker‑Kammer‑Innenseite eingesetzt, Membran zeigt durch das
> Schallwand‑Loch nach außen, Schrauben werden von innen verschraubt –
> **keine Schrauben von außen erreichbar**. Begründung: Halloween‑Outdoor
> mit Kindern, Vandalismus‑Schutz, optisch sauberere Außenseite, kein
> Zugriff auf die Membran. Der Korb‑Vorderring (Ø 64 × 2 mm) sitzt in
> einer **innenseitigen Stufenbohrung**, der Quadrat‑Flansch (66 × 66)
> liegt auf dem 67 × 67 Block der Schallwand‑Innenseite auf.

| Parameter | Wert | Begründung |
|---|---|---|
| Außenmaße (Speaker‑Kammer‑Bereich) | 106 × 84 × 75 mm (B × H × T) | aus aktueller 3D‑Druck‑Box übernommen; Tiefe auf 75 mm gesetzt für 14 mm Atmungsraum hinter Magnet (Korbtiefe 61 mm) |
| Innen B × H × T | 98 × 76 × 75 mm | bei 4 mm Wand seitlich/oben + 4 mm Trennwand unten + 6 mm Schallwand vorn + 4 mm Rückwand hinten |
| Brutto Innenkammer | ~0,56 l | |
| **Netto Speaker‑Kammer** | **~0,5 l** | Ziel: Q_tc ≈ 1,0 (Butterworth‑Optimum), f₃ ≈ 270 Hz |
| Schallwand‑Stärke | 6 mm + 6 mm Innen‑Block 67 × 67 mm | Innen‑Block dient gleichzeitig als Auflage für Quadrat‑Flansch und als Material für die M4‑Inserts; gesamt 12 mm Materialtiefe an der Schraubenposition |
| Andere Wände | 4 mm | |
| Rückwand (Speaker‑Rückwand, separates Druckteil) | 4 mm, **flach** (kein 5°) | flach gewählt 2026‑05‑09 – akustischer Mehrwert der 5°‑Neigung bei 0,5 l‑Box minimal (Eigenmoden > 1 kHz, da hilft 5° kaum), kompensiert durch 40 % Bedämpfung statt 30 %; flach = einfacherer Druck, sauberer Spritzwasser‑Schutz, gleichmäßige umlaufende Dichtung möglich |
| **Membran‑Durchgangsloch (außen)** | **Ø 61 mm durchgehend** | = Korb‑Innendurchmesser; Membran schaut durch, kein Spiel zur Sicke nötig (Sicke schwingt axial im Korb) |
| **Korb‑Ring‑Stufenbohrung (innen)** | **Ø 64 mm × 2 mm tief, von Block‑Innenseite aus** | nimmt den 2 mm hohen Korb‑Vorderring auf, bündig mit Block‑Innenseite |
| **Quadrat‑Flansch‑Auflage** | flache Block‑Innenseite 67 × 67 mm | Quadrat‑Flansch 66 × 66 liegt mit 0,5 mm Spiel pro Seite plan auf, EPDM‑Dichtstreifen 1 mm umlaufend |
| **Lochkreis Speaker** | **Ø 74 mm**, 4 Schrauben bei 45°/135°/225°/325° | aus Datenblatt FR 7 |
| **Schraubenbohrungen (von Block‑Innenseite)** | **4 × Ø 5,6 mm Sackloch, 9,1 mm tief** | M4‑Insert wird von innen reingedrückt, Schraube M4 × 10 mm von innen ins Insert |
| **Speaker‑Versatz** | **+6 mm in X‑Richtung** (gegen die Symmetrie des Schallwand‑Rechtecks) | gegen Mid‑Mode‑Auslöschungen (8.4); Y mittig, weil in Y nur 4,5 mm Reserve zur Innenwand sind |
| **Schutzgitter außen** | **Wabenmuster: 8 mm Hexagon‑Löcher, 1,5 mm Stege, 2 mm tief** von Schallwand‑Außenseite eingefräst | gleich mit dem Korpus gedruckt; Halloween‑Kinder kommen nicht an die Membran; akustisch fast neutral (~75 % offene Fläche) |
| **Loch‑Innenkante** | optional R2 abgerundet auf der Schallwand‑Außenseite | reduziert Strömungsgeräusche bei lauten Bässen, optisch sauberer |
| Bedämpfung | **~40 %** Polyesterwatte (Visaton VS‑WOOL2 oder gleichwertig) | locker an Wänden, hinter Speaker 14 mm Luft frei (Korb 61 mm in 75 mm Innentiefe); 40 % statt 30 %, weil flache Rückwand ohne 5°‑Neigung |
| Innenrippe | kreuzförmige Verrippung an Schallwand‑Innenseite | versteift Schallwand, schiebt Wand‑Eigenresonanz von ~250 auf ~600 Hz |
| Druckhöhe Speaker‑Kammer | ~75 mm | Front auf Bauplatte, wächst nach hinten/oben (entspricht Box‑Tiefe, nicht Box‑Höhe – siehe 8.6) |

**Schichtaufbau Schallwand auf Höhe einer Schraube (von außen nach innen):**

```
außen
 │ Schallwand 6 mm voll, Wabenmuster 2 mm tief eingefräst
 │ → Loch Ø 61 durchgehend für Membran
 │
Block 6 mm (67 × 67 mm zentriert um Speaker, +6 mm in X)
 │ → Loch Ø 61 in den ersten 4 mm
 │ → Stufe auf Ø 64 in den letzten 2 mm (Korb‑Ring sitzt rein)
 │ → 4 × Ø 5,6 mm Sackloch 9,1 mm tief für M4‑Insert
 │
innen (Speaker‑Kammer)
 │ Speaker liegt mit Quadrat‑Flansch 66 × 66 plan auf Block‑Innenseite
 │ EPDM‑Streifen 1 mm umlaufend zwischen Flansch und Block
 │ M4 × 10 mm Schraube von innen durch Quadrat‑Flansch ins Insert
```

### 8.3 Speaker‑Kammer Box B (FR 10/4 Bassreflex)

| Parameter | Wert | Begründung |
|---|---|---|
| Innen B × H × T | 140 × 110 × 130 mm | Korb‑Außen‑Ø FR 10/4 = 105 mm → 2,5 mm Luft je Seite beim Einführen ✓; Tiefe 130 mm bei Korb‑Tiefe 47 mm = 83 mm Luft hinter Magnet |
| Brutto Innenkammer | ~2,0 l | |
| **Netto Speaker‑Kammer** | **~1,9 l** | Visaton empfiehlt ≥ 2,5 l geschlossen für FR 10/4; Bassreflex mit ~1,9 l ist akzeptabel, Tiefbass‑Abstimmung per Port |
| Auslegung | Bassreflex, optional als geschlossen nutzbar (gedruckter Stopfen für Port) | gibt Hörtest‑Flexibilität ohne neue Box |
| **Bassreflex‑Port** | **Ø 20 mm × 56 mm Länge** (physikalisch), gerader Tunnel hinten unten | f_b = 80 Hz (berechnet 2026‑05‑13 aus T/S‑Daten: fs = 90 Hz, Vb = 2,06 l, Leff = 71 mm − 14,6 mm Endkorrektur = 56 mm); Tunnel ragt ~52 mm in die Box, 4 mm durch Rückwand |
| Port‑Strömungsgeschwindigkeit | ~7 m/s bei voller Auslenkung | unkritisch (Grenze ~17 m/s für hörbare Strömungsgeräusche) |
| Port‑Mündung innen | mit 3 mm Radius abgerundet | halbiert Strömungsgeräusche |
| Port‑Mündung außen | mit gedrucktem Insektenschutz‑Gitter (Stege Ø 1,5 mm, Lochweite ~3 mm) | akustisch fast neutral, Outdoor‑tauglich |
| Schallwand‑Stärke | 6 mm (+ Block 4 mm an Schraubpunkten) | wie Box A |
| Andere Wände | 4 mm + horizontale Querstrebe in halber Tiefe | Versteifung wegen größerer Flächen |
| Rückwand | 4 mm, 5° schräg | gegen stehende Wellen |
| **Membran‑Durchgangsloch (außen)** | **Ø 100 mm** durchgehend | = innerer Korb‑Ø / Membran‑Ø (Etikettmaß Frontansicht, mit Schieblehre bestätigt) |
| **Korb‑Ring‑Stufenbohrung (innen)** | **Ø 105 mm × 3 mm tief**, von Block‑Innenseite | Korb‑Außen‑Ø = 105 mm (Etikettmaß Seitenansicht, Gesamthöhe); Stufentiefe 3 mm = Flanschhöhe (bestätigt) |
| **Quadrat‑Flansch‑Auflage** | Block‑Innenseite ~91 × 91 mm (0,5 mm Spiel zum ~90 × 90 mm Flansch) | Flansch‑Außen‑Diagonale Ø 127,5 mm → Seite ≈ 90,2 mm (Etikettmaß); EPDM‑Dichtstreifen 1 mm umlaufend |
| **Lochkreis Speaker** | **Ø 115 mm** (Schlitze 5 × 7 mm auf diesem Kreis); entspricht 4 × Schlitzpositionen im **81 × 81 mm Quadrat‑Raster** (81 = 115 / √2 ≈ 81,3 mm, je ±40,5 mm vom Mittelpunkt) | aus Etikettmaß bestätigt; M4‑Inserts wie Box A |
| Bedämpfung | ~30 % Polyesterwatte | FR 10/4 hat niedrigeren Q_ts als FRS 8 → weniger Bedämpfung nötig; Port‑Bereich + 5 cm rundum frei halten |
| Druckhöhe Speaker‑Kammer | ~130 mm | Front auf Bauplatte, ggf. Drucker‑Standfestigkeit prüfen |

### 8.4 Konstruktive Konventionen für beide Boxen

**Geometrie:**

- Keine drei gleichen Wand‑Abstände → Verhältnis ~1 : 1,3 : 1,6 (Annäherung Goldener Schnitt)
- Mindestens eine Wand schräg (Rückwand 5°) → vermeidet symmetrische Reflexionen
- Speaker leicht aus der Mittenachse versetzt (~60 % zu einer Seite) → asymmetrische Reflexionen, weniger Mid‑Mode‑Auslöschungen

**Material und Druck:**

| Eigenschaft | Wert |
|---|---|
| Material | PETG (UV‑stabil, akustisch ok, gut abdichtbar mit Sikaflex) |
| Slicer Perimeter | 6 (statt der typischen 3) |
| Infill in Bossen / Schraubpunkten | 100 % |
| Infill Wand sonst | 30–40 % Gyroid |
| Layer‑Höhe | 0,2 mm Standard, Schallwand‑Außenseite 0,15 mm für saubere Front |

**Bedämpfung:**

| Eigenschaft | Wert |
|---|---|
| Material (Standard) | **Visaton VS‑WOOL2** Polyester 5070, 125 g (~11 €, ausreichend für ~20 l Volumen → 1 Beutel reicht für mehrere Stationen) |
| Material (Alternativen) | IKEA‑Kissenfüllung, Sonofil, Schafwolle – alle akustisch vergleichbar |
| Verteilung | locker an die Wände drücken, **nicht** zentral stopfen |
| Hinter Speaker | hinter dem Magnet etwas Luft freilassen (Membran braucht Atmungsraum) – bei FR 7 mit 14 mm Restluft eng, locker bedämpfen ohne den Magnet einzukapseln |
| **Box A** | **~40 %** Volumenfüllung (kompensiert die flache Rückwand ohne 5°‑Neigung) |
| Box B | ~30 % (FR 10/4 hat niedrigeren Q_ts als FRS 8, weniger Bedämpfung nötig) |
| Bei Bassreflex (Box B) | Port‑Bereich + 5 cm rundum **frei** halten |

**Dichtigkeit (kritisch, weil Class‑D mit 12 V deutlich mehr Membranhub erzeugt als der 3,3 V MAX98357 vorher):**

| Stelle | Maßnahme |
|---|---|
| Speaker‑Korbflansch | umlaufender EPDM‑Streifen 1–2 mm zwischen Korb und Versenkung, **nicht** Plastik auf Plastik |
| Speaker‑Schraubpunkte | M4‑Messing‑Inserts in lokalen Bossen (siehe 8.5), Schrauben mit Dichtscheibe (Polyamid + Gummi) |
| Übergang zur Elektronik‑Kammer | Dichtschnur in gedruckter Nut oder Schaumstoff‑Streifen (kommt mit der Elektronik‑Kammer im nächsten Doku‑Abschnitt) |
| Kabel‑Durchführung | aus der Elektronik‑Kammer, nicht aus der Speaker‑Kammer → Speaker‑Kammer bleibt komplett geschlossen |
| Bassreflex‑Port (nur Box B) | gedruckter Tunnel ist von Haus aus dicht; Stopfen für „geschlossen‑Modus" mit O‑Ring |

### 8.5 Einpressmuttern (Ruthex) und Verschraubungen

Tobias hat Ruthex‑Inserts in M2, M3, M4, M5 vorrätig. Für die Station
kommen wir mit **M3 + M4** aus – M4 nur für die 4 Speaker‑Schraubpunkte
(weil das FR 7 Datenblatt 4,2‑mm‑Schraubenlöcher im Korbflansch vorgibt),
M3 für alles andere. Nichts neu zu bestellen.

**Speaker‑Befestigung – Innen‑Block 67 × 67 × 6 mm an der Schallwand‑Innenseite (M4):**

Statt freistehender Säulen pro Schraubpunkt wird ein **durchgehender Block
67 × 67 mm × 6 mm hoch** auf die Schallwand‑Innenseite gedruckt. Der
Block dient gleichzeitig als (a) Stufenbohrungs‑Material für den
Korb‑Vorderring, (b) Auflage für den Quadrat‑Flansch und (c) Material
für die M4‑Inserts. Gegenüber freistehenden Säulen druckbar **deutlich
stabiler** (keine schlanken Z‑Strukturen, die brechen können) und
versteift die Schallwand zusätzlich. Volumenverlust gegenüber Säulen:
~27 cm³ = ca. 5 % der Speaker‑Kammer – akustisch vernachlässigbar.

| Maß | Wert | Begründung |
|---|---|---|
| Block‑Außenmaß | 67 × 67 × 6 mm | 0,5 mm Spiel pro Seite zum Speaker‑Quadrat‑Flansch (66 × 66), Auflagefläche für Flansch |
| Bohrung Ø | 5,6 mm (= d3 nach Ruthex‑Tabelle) | für M4 Standard‑Insert |
| Bohrung Tiefe ab Block‑Innenseite | **9,1 mm Sackloch** | L = 8,1 mm + 1 mm Reserve, Sackloch nach Datenblatt; bleibt 2,9 mm Material zur Schallwand‑Außenseite (kein Durchbruch nötig, da Schraube von innen kommt) |
| Insert | **M4 Standard** (L = 8,1 mm, Art. RX‑M4×8,1) | gute Auszugskraft, passt zum 4,2 mm Schraubloch im Korbflansch |
| Schraube | **M4 × 10 mm** (Senkkopf oder Linsenkopf) | Quadrat‑Flansch 1,7 mm + Insert‑Sitz 8,1 mm = ~10 mm Schraubenlänge unter Kopf |
| Schraub‑Richtung | **von innen ins Insert** | Vandalismus‑Schutz, keine Schrauben außen |
| Block‑Druckbarkeit | flache Erhebung, wächst mit der Schallwand mit | bei Druck „Front auf Bauplatte" kein Stützmaterial nötig, kein freistehendes Säulen‑Risiko |

**Andere Verschraubungen (M3) – Bosse oder direkt in 6 mm Wand:**

| Stelle | Insert | Bohrung | Boss nötig? |
|---|---|---|---|
| Speaker‑Rückwand → Korpus (4×, Box A; 6×, Box B) | M3 Short (L = 4 mm) | Ø 4,0 mm × 5 mm tief | nein, direkt in 4‑mm‑Wandkante (Bosse mit Wandverdickung an der Schraubposition) |
| Boden‑Klappe → Korpus (4×) | M3 Short | Ø 4,0 mm × 5 mm tief | nein, direkt in 4‑mm‑Bodenkante |
| Pfostenhalter → Korpus‑Seitenwand (4×) | M3 Short | Ø 4,0 mm × 5 mm tief | nein, direkt in 4‑mm‑Seitenwand |
| OLED‑Modul → Boden‑Klappe (4×) | M3 Short | Ø 4,0 mm × 5 mm tief | ja, kleine Bosse an der Klappen‑Innenseite |
| PCB / Modul‑Halter intern | M3 Short oder direkt M3‑Schrauben in M3‑Gewinde‑Bohrungen | je nach Modul | je nach Modul |

**Einpressen praktisch:**

- Lötkolben‑Spitze mit kegelförmigem Insert‑Aufsatz (ggf. Ruthex‑Set mit Spitze)
- PETG‑Schmelztemperatur: ~230 °C
- Insert auf Spitze stecken, langsam reinschieben bis bündig mit Materialoberfläche
- Boss‑Außenwand kann sich beim Einpressen minimal ausbeulen – normal, kein Schaden

### 8.6 Druck‑Orientierung: Front auf Bauplatte

**Begründung:** Sauberster Druck der Schallwand‑Außenseite (glatt durch
Bauplatten‑Kontakt) und scharfer Druck der gedruckten Schutzgitter‑Stege
direkt am Speaker‑Ausschnitt. Box wächst nach hinten/oben weg von der
Bauplatte.

**Konsequenzen, die in der Konstruktion mitgedacht sind:**

1. **Layer‑Linien parallel zur Schallwand** → schwächste Z‑Anhaftung
   wirkt senkrecht zur Membran‑Bewegung. Gegenmaßnahme: Schallwand 6 mm
   statt 5 mm, plus interne Verrippung.
2. **Innenrippen müssen selbsttragend sein** (≤ 45° Überhang oder kurze
   Brücken). Verrippung ist als schräg ansteigende Streben von der
   Schallwand‑Innenseite zu den Seitenwänden konstruiert. Eine
   horizontale Querstrebe in halber Tiefe (Box B) wirkt zusätzlich als
   Versteifung und ist Auflage für das Bedämpfungs‑Material.
3. **Schräge Rückwand kippt nach hinten‑oben** → in Druckrichtung weg
   vom Druckkopf, daher selbsttragend ohne Stützen.
4. **Service‑Klappe sitzt auf der weg‑wachsenden Seite** (oben oder
   hinten, wird in der Elektronik‑Kammer entschieden). Maßungenauigkeit
   der zuletzt gedruckten Wand wird durch die Schaumstoff‑Dichtung am
   Klappenrand kompensiert.
5. **Bosse wachsen nach oben** in die Innenkammer – kein Überhang,
   einfacher selbsttragender Druck.
6. **Box‑Tiefe = Druckhöhe** (nicht Box‑Höhe!). Da die Front auf der
   Bauplatte liegt und die Box nach hinten/oben wächst, entspricht die
   Z‑Druckhöhe der Box‑Tiefe. Speaker‑ und Elektronik‑Kammer wachsen
   **nebeneinander** in die Höhe (entlang einer X/Y‑Achse), nicht
   aufeinander. Damit ist die Druckhöhe für Box A ~75 mm, für Box B
   ~130 mm – beides auf einem normalen FDM‑Drucker (200 × 200 × 200)
   entspannt. **Voraussetzung:** Speaker‑ und Elektronik‑Kammer haben
   die gleiche Tiefe (was beim einteiligen Korpus ohnehin der Fall ist).

### 8.7 Einteiliger Korpus mit zwei Kammern

Die Station wird als **einteiliger Korpus** gedruckt, in dem Speaker‑
und Elektronik‑Kammer durch eine **fest mitgedruckte, durchgehende
Trennwand** voneinander getrennt sind. Beide Kammern haben eigene
Service‑Öffnungen nach außen (Speaker‑Rückwand oben, Boden‑Klappe unten),
sodass der Korpus selbst nie geteilt werden muss.

**Begründung gegenüber der ursprünglichen zweiteiligen Modul‑Konstruktion**
(siehe Diskussion 2026‑05‑09):

- **Druckhöhe ist kein Argument für Teilung.** Bei der gesetzten
  Druckorientierung „Front auf Bauplatte" (siehe 8.6) ist die Z‑Druckhöhe
  identisch mit der Box‑Tiefe, nicht mit der Box‑Höhe. Speaker‑ und
  Elektronik‑Kammer wachsen **nebeneinander** in Z‑Richtung, nicht
  aufeinander. Druckhöhe ist damit Box A ~75 mm, Box B ~130 mm – beides
  auf einem normalen FDM‑Drucker entspannt.
- **Service‑Strategie unverändert.** Speaker‑Kammer wird über die
  Speaker‑Rückwand geöffnet (oder nie), Elektronik‑Kammer über die
  Boden‑Klappe. Beide Service‑Öffnungen existieren auch in der einteiligen
  Variante – sie hängen nicht davon ab, ob der Korpus selbst geteilt ist.
- **Dichtigkeit besser einteilig.** Die Trennwand wird im einteiligen
  Druck zur durchgehenden Innenstruktur, ohne Toleranz‑Spalt zwischen
  zwei Druckteilen, ohne Dichtschnur, ohne Schrauben im Lastpfad. Bei
  12 V Class‑D ist jede vermiedene Dichtungsstelle ein Pluspunkt.
- **Insert‑Aufwand sinkt.** Die 4× M3‑Inserts auf der Trennwand‑Außenseite
  und die zugehörigen 4× M3‑Schrauben entfallen.
- **Speaker‑Kabel‑Durchführung: PG7‑Kabelverschraubung (gesetzt 2026‑05‑10).**
  Loch in der Trennwand: **Ø 12,2 mm** für eine Standard‑PG7‑Kabelverschraubung.
  Kabel: **2×0,75 mm² Lautsprecherkabel** (z. B. sonero 2×0,75 mm²,
  Querschnitt nötig für TPA3110 bei 12 V / ~2 A). Verbindung auf beiden Seiten
  per **2‑pol Schnellverbinder (Schnellklemme, 18–22 AWG)** – kein Löten
  notwendig. Montage‑Reihenfolge: Kabel zuerst durch PG7 fädeln (Flachkabel
  45° drehen), dann Schnellverbinder crimpen, dann PG7‑Mutter festziehen.
  Die PG7 dichtet gleichzeitig akustisch ab und übernimmt die Zugentlastung.

**Trade‑off:** Größerer Druck = höheres Failure‑Risiko bei einem Print
Fail. Bei Box B sind das ~10–12 h Druckzeit am Stück. Mitigation: saubere
Druckparameter (PETG getrocknet, Erste‑Layer‑Test mit der Speaker‑Rückwand
als kleinerem Vortest), notfalls Cancel und Re‑Druck. Bei Box A ist die
Druckzeit ohnehin entspannt.

**Druckteil‑Übersicht (3 Druckteile + Pfostenhalter):**

| Druckteil | Bestandteile | Wartungs‑Charakter |
|---|---|---|
| **Korpus** (einteilig) | Front + 4 Seitenwände + **fest mitgedruckte Trennwand** zwischen Speaker‑ und Elektronik‑Kammer + fest gedruckte Rückwand der Elektronik‑Kammer (mit Antennen‑Tasche innen). Speaker‑Kammer nach hinten/oben offen, Elektronik‑Kammer nach unten offen. | dauerhaft, wird nie zerlegt |
| **Speaker‑Rückwand** | eigenes kleines Druckteil mit 5° Neigung; bei Box B mit Bassreflex‑Port + Insektengitter | nach Aufbau dauerhaft geschlossen |
| **Boden‑Klappe (groß)** | beherbergt **alles**: DC‑Hohlbuchse + 6‑pol Stab‑Klemmstecker + (optional) OLED 1,3″ + 4 Tasten + Kabel‑Klemmleisten | wird beim Saison‑Aufbau geöffnet (1× pro Saison) und für Konfig‑Anpassungen |
| **Pfostenhalter‑Adapter** | seitlich am Korpus auf Höhe der Elektronik‑Kammer verschraubt | wechselbar je nach Pfostenmaß |

**Konstruktive Konsequenzen für Fusion 360:**

- Korpus wird als **ein Volumen** modelliert. Trennwand entsteht über ein
  zusätzliches Skizzen‑Feature auf einer horizontalen Skizzenebene
  zwischen den Kammern, dann als Extrusion mit ~4 mm Stärke nach beiden
  Seiten.
- Außenmaße sind ein einziger Quader: Box A 106 × 149 × 75 mm (B × H × T),
  Box B ~150 × 200 × 130 mm (B × H × T). H = H_speaker (~120 mm) + H_elektronik (~80 mm).
- Tiefen von Speaker‑ und Elektronik‑Kammer werden auf den **gleichen
  Wert** parametrisiert (= Außen‑Tiefe der Box). Box A 75 mm, Box B 130 mm.
  Damit ist auch die Z‑Druckhöhe konsistent.
- Die Trennwand kann gleichzeitig als konstruktive **Versteifung** der
  Seitenwände wirken → erlaubt evtl. dünnere Seitenwände (3 mm statt
  4 mm) oder reduzierten Gyroid‑Infill. Vor dem Druck mit einem
  Wandprobenteil verifizieren.
- **Ø 12,2‑mm‑Bohrung** in der Trennwand für PG7‑Kabelverschraubung
  (Speaker‑Kabel 2×0,75 mm²). Position: nahe der Rückwand, mittig in
  der Trennwand‑Höhe, damit das Kabel von hinten/unten in die
  Elektronik‑Kammer geführt werden kann.

**Begründung der Vereinfachung gegenüber dem allerersten Konzept
(Service‑Rückwand + kleine Anschluss‑Klappe getrennt):** Beim Saison‑Aufbau
wird die Box ohnehin aufgemacht zum Anschließen von Strom + Stab‑Kabel.
In genau dem Moment kann auch die Konfiguration über OLED + Tasten
passieren. Eine zusätzliche Service‑Rückwand wäre redundant.
Lautstärke‑Änderungen im laufenden Spielbetrieb sind selten und werden
über die Außen‑Status‑LED und das nächste Saison‑Setup kompensiert.

### 8.8 Korpus (Druckteil 1) – Konstruktion

**Was im Korpus enthalten ist:**

- Schallwand (Front, Speaker‑Kammer)
- Elektronik‑Front (= unterer Teil derselben durchgehenden Vorderseite)
- 4 Seitenwände (durchgehend von oben nach unten)
- **Fest mitgedruckte Trennwand** zwischen Speaker‑ und Elektronik‑Kammer
  (~4 mm dick, mit Ø‑12,2‑mm‑Bohrung für PG7‑Kabelverschraubung)
- **Fest gedruckte Rückwand der Elektronik‑Kammer** mit innenliegender
  Antennen‑Tasche (~12 × 70 × 8 mm vertikal)
- Schraubpunkt‑Bosse für den Speaker an der Schallwand‑Innenseite (siehe 8.5)
- M3‑Short Inserts an einer Seitenwand der Elektronik‑Kammer (4×) für
  den Pfostenhalter‑Adapter
- M3‑Short Inserts an der Bodenkante der Elektronik‑Kammer (4×) für
  die Boden‑Klappe
- M3‑Short Inserts in den 4 (Box A) bzw. 6 (Box B) **diagonalen Eckblöcken**
  der Speaker‑Kammer für die Speaker‑Rückwand (siehe 8.9 für Geometrie)
- Speaker‑Kammer nach **hinten/oben offen** – schließt sich erst durch
  die Speaker‑Rückwand (Druckteil 2)
- Elektronik‑Kammer nach **unten offen** – schließt sich erst durch die
  Boden‑Klappe (Druckteil 3)

**Druck‑Eckdaten:**

| Parameter | Box A | Box B |
|---|---|---|
| Außenmaße (B × H × T) | 106 × 149 × 75 mm | ~150 × 200 × 130 mm (Speaker‑Kammer‑H ~120 mm + Elektronik‑Kammer‑H ~80 mm) |
| Druckorientierung | Front auf Bauplatte | Front auf Bauplatte |
| Z‑Druckhöhe | ~75 mm | ~130 mm |
| Druckzeit (Schätzung, PETG, 0,2 mm) | ~6–8 h | ~10–12 h |
| Material | PETG | PETG |
| Perimeter | 6 (Schallwand) / 4 (Rest) | 6 (Schallwand) / 4 (Rest) |
| Infill Wand | 30–40 % Gyroid | 30–40 % Gyroid |
| Bosse / Schraubpunkte | 100 % Infill | 100 % Infill |

**Antennen‑Halterung in der mitgedruckten Rückwand:**

| Element | Position | Befestigung |
|---|---|---|
| Antennen‑Tasche | innen an der Rückwand der Elektronik‑Kammer, vertikal | gedruckte Vertiefung ~12 × 70 × 8 mm |
| Whip‑Antenne (2,4 GHz, 6 cm) | in der Tasche | mit 2K‑Kleber oder Heißkleber fixiert |
| IPEX‑Pigtail | von Antenne zum DevKitC‑IPEX | 5 cm reichen, weil Antenne im gleichen Korpus sitzt |

### 8.9 Speaker‑Rückwand (Druckteil 2) – Konstruktion

> **Geometrie gesetzt 2026‑05‑09:** Flacher Deckel mit umlaufender 2‑mm‑Erhöhung,
> die in eine 3‑mm‑Vertiefung im Korpus greift. EPDM‑Schaumstoff 3 × 2 mm
> (selbstklebend, halbiert aus 6 × 2 mm Rolle) auf die Erhöhungs‑Innenseite
> geklebt. Im verschraubten Zustand auf 1 mm = 50 % komprimiert.
> 5°‑Neigung verworfen (siehe 8.2).

**Maße Box A:**

| Parameter | Wert |
|---|---|
| Außenmaße | 106 × 84 mm (volle Korpus‑Breite × Speaker‑Kammer‑Höhe) |
| Materialdicke (Grundplatte) | 4 mm |
| Umlaufende Erhöhung | 2 mm hoch, 2,5 mm breit, am Rand |
| Schraublöcher | 4 × Ø 3,3 mm in den Eckpositionen, passend zu den M3‑Eckblöcken im Korpus |
| Senkungen für Schraubenköpfe | außen Ø 6 mm × 2 mm tief (Senkkopf‑Aufnahme) |

**Maße Box B (zusätzlich):**

| Parameter | Wert |
|---|---|
| Außenmaße | ~150 × 120 mm (volle Korpus‑Breite × Box‑B‑Speaker‑Kammer‑Höhe: 110 mm innen + 6 mm Schallwand + 4 mm Rückwand) |
| Schraublöcher | 6 × Ø 3,3 mm (zusätzlich 2 in Mitte oben/unten gegen Wand‑Eigenresonanz) |
| Bassreflex‑Port | gerader Tunnel **Ø 20 × 56 mm Länge** (physikalisch), integriert in die Rückwand, hinten unten platziert; Tunnel ragt ~52 mm in die Box (f_b = 80 Hz) |
| Insektengitter am Port außen | Stege Ø 1,5 mm, Lochweite ~3 mm, mitgedruckt |
| Innen am Port | 3 mm Eckradius zur Strömungsoptimierung |

**Korpus‑Gegenstück (gehört zum Korpus, hier zur Vollständigkeit):**

| Parameter | Wert |
|---|---|
| Vertiefung umlaufend | **3 mm tief, 2,5 mm breit** an der Speaker‑Kammer‑Hinterkante |
| Vertiefung an der Trennwand | **1 mm tief** (weil die Trennwand sich Auflagefläche mit der Elektronik‑Klappe teilt) |
| Eckblöcke | diagonal 8 mm Schräge × 4 mm Tiefe in jeder Korpusecke; M3‑Insert Ø 8 mm Boss; Insert‑Mittelpunkt 3 mm von der Außenkante |
| Inserts | M3 Short (L = 4 mm), Ø 4,0 × 5 mm Bohrung |

**Dichtung – Komprimierungs‑Bilanz:**

| Element | Wert |
|---|---|
| Korpus‑Vertiefung | 3 mm |
| Deckel‑Erhöhung | 2 mm |
| Restspalt | 1 mm |
| EPDM‑Schaumstoff (Roh) | 2 mm dick × 3 mm breit (aus 6 × 2 mm Rolle halbiert) |
| Schaum komprimiert | auf 1 mm = **50 % Komprimierung** ✓ |

**Dichtungs‑Material:**

- esnado EPDM‑Schaum schwarz, einseitig selbstklebend, 6 × 2 mm Rolle (10 m, ~7 €)
- Längsseits halbieren mit Cutter → 3 × 2 mm Streifen
- Auf die Erhöhungs‑Innenseite des Deckels umlaufend kleben (nicht auf den Korpus –
  Schaum bleibt am Deckel haften, wenn dieser abgenommen wird; saubere Kante
  zur Speaker‑Kammer hin)

**Verschraubung:**

- 4 × M3 × 10 mm Senkkopfschraube von außen
- Anziehen über Kreuz, schrittweise in 3 Durchgängen (handfest → 70 % → final),
  damit die Dichtung gleichmäßig komprimiert wird

**Bassreflex‑Stopfen für Box B (optional):**

- Falls Hörtest „geschlossen" gewinnt: gedruckter Stopfen Ø 20 × 30 mm
  mit O‑Ring, lässt sich nachträglich in den Port einsetzen
- Alternative: zweite Rückwand ohne Port drucken

### 8.10 Boden‑Klappe (Druckteil 3) – das zentrale Service‑Element

Die Boden‑Klappe ersetzt die zuvor diskutierte Service‑Rückwand und die
kleine Anschluss‑Klappe – sie vereint **alle externen Schnittstellen +
Konfiguration** in einem Druckteil. Dieses Konzept wurde 2026‑05‑08 nach
einer Tobias‑Vereinfachung gesetzt.

**Was alles auf der Boden‑Klappe sitzt:**

| Element | Position auf Klappe | Befestigung / Hinweise |
|---|---|---|
| **DC‑Hohlbuchse 5,5/2,5 mm** | seitlich, durchgehende Bohrung | Buchse auf dem **Audio‑PCB** (Power‑Entry, s. 8.14), Bohrung im Klappenmaterial |
| **6‑pol Stab‑Klemmstecker** | gegenüber DC‑Buchse | grüner Schraubklemmstecker auf dem **Logic‑PCB**, Stab‑Kabel wird eingeschraubt |
| **OLED 1,3″ SH1106** *(optional)* | mittig | 4× M3‑Short Inserts, OLED‑Modul von innen angeschraubt, **kein** Sichtfenster nach außen – nur bestückt, falls gewünscht (s. 8.14) |
| **4 Tasten K1–K4** *(optional)* | unter OLED in einer Reihe, von innen erreichbar | Modul mit OLED + Tasten in einem, **nicht von außen erreichbar** |
| **Kabel‑Klemmleisten** | gedruckt in der Klappen‑Innenseite um DC + Stab‑Klemmstecker | klemmen Stromkabel + Stab‑Kabel als Zugentlastung |
| **JST‑XH 8‑pol Verbindung** | innen am Klappenrand | OLED‑Modul → Hauptboard (I²C SDA/SCL/3,3V/GND + K1–K4 + 5V) |

**Wichtig zur Bedienung (Tobias‑Entscheidung 2026‑05‑08):**

> OLED **nicht** durch ein Klarsicht‑Fenster nach außen sichtbar. Tasten
> **nicht** von außen erreichbar. Konfiguration ist nur möglich, wenn
> die Boden‑Klappe geöffnet ist – das passiert ohnehin beim Saison‑Aufbau
> beim Anschließen der Kabel. Damit ist die Klappe aus PETG geschlossen
> und vollständig spritzwassergeschützt, ohne Klarsicht‑Inlay oder
> Tastenausschnitte.

**Klappen‑Mechanik:**

| Eigenschaft | Wert |
|---|---|
| Außenmaß Box A | ~106 × 84 mm (gesamter Boden) |
| Außenmaß Box B | ~140 × 110 mm (gesamter Boden) |
| Klappen‑Materialdicke | 4 mm + 6 mm interne Bosse für OLED |
| Befestigung | 4× M3‑Short Inserts in der Bodenkante des Korpus + Innensechskant‑Schrauben |
| Dichtung | umlaufender Schaumstoffstreifen 1 mm Klappenrand außen |
| Position | unten am Elektronik‑Modul (Schwerkraft, Wasser läuft am Kabel ab) |

**Vorteil:** Wenn das OLED‑Modul später wechselt (anderes Display,
andere Tasten‑Belegung) oder die Buchsen sich ändern – nur die
Boden‑Klappe neu drucken, Korpus bleibt unangetastet.

**Vorgehen beim Saison‑Aufbau:**

1. Box am Pfostenhalter befestigen
2. Boden‑Klappe abschrauben (4 Schrauben)
3. Stromkabel durch die Klemmleiste führen, auf DC‑Buchse stecken
4. Stab‑Kabel durch die Klemmleiste führen, in den 6‑pol Klemmstecker schrauben
5. Strom anschließen; ID/Lautstärke entweder am OLED (falls bestückt) **oder**
   übers Config‑Tool per ESP‑NOW (s. 8.14 / Doc 18) setzen
6. Stations‑ID wählen, Lautstärke einstellen, Sound‑Test fahren
7. Boden‑Klappe wieder anschrauben
8. Saison läuft – Klappe bleibt zu

### 8.11 Stab‑Connector (6‑pol Schraubklemmstecker, 6 Adern, Trigger active‑LOW)

> **Wechsel 2026‑06‑08 (Tobias): USB‑C → grüner 6‑pol Schraubklemmstecker.**
> Die USB‑C‑Buchse aus dem Stand 2026‑05‑08 wird verworfen. Stattdessen sitzt
> auf dem Logic‑PCB ein **grüner 6‑poliger PCB‑Schraubklemmstecker** (Phoenix‑
> kompatibler Steckverbinder, z. B. DIFLAX 6‑polig, 8 A, Raster am Bauteil
> **vor dem Layout mit Multimeter/Schieblehre prüfen** – Hersteller‑Angaben
> schwankten zwischen 3,81 mm und 5,08 mm). Auf der **Wand‑Seite wird das
> Kabel künftig direkt angelötet** (kein Stecker am Stab).
>
> **Begründung:** Die USB‑C‑Lösung brachte drei Nachteile, die mit dem
> Klemmstecker entfallen – (1) Verwechslungsgefahr mit Ladegeräten, (2)
> Reversibilitäts‑Verdrahtung (jedes Signal doppelt auf A/B‑Pins) und (3)
> die teure Kabel‑Beschaffung mit eMarker‑Durchgangstest. Da Station =
> festgeschraubt und Stab = fest angelötet, gibt es **keinen Hot‑Plug mehr**.

> **Wichtig (Wand‑Seite):** Die Wand selbst muss für den Refactor minimal
> modifiziert werden – Trigger‑Pull‑down R1 entfernen, Trigger gegen GND
> statt gegen 3,3 V führen. Damit fällt die 3,3‑V‑Ader im Kabel weg.
> Siehe auch [`02-hardware-wand.md`](02-hardware-wand.md), Abschnitt
> „Trigger", [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md) und Punkte
> 23/26 in [`11-offene-punkte.md`](11-offene-punkte.md).

**Adern‑Belegung (6 Adern, jeweils auf genau einem Klemmen‑Pin):**

| Pin | Signal | Funktion | Wand‑Seite | Station‑Seite |
|---|---|---|---|---|
| 1 | 5 V | Versorgung Stab (IR‑LED, Laser, NeoPixel) | direkt angelötet | Buck‑Down 5 V (übers Inter‑Board‑Kabel, s. 8.14) |
| 2 | GND | Masse | direkt angelötet | Masse |
| 3 | NeoPixel‑Daten | 3,3 → 5 V‑Logik | direkt zur ersten SK6812 | ESP RMT‑Output (über 74AHCT125, R2) – **kein Filter‑Cap** (800 kHz) |
| 4 | IR‑LED Steuerung | 3,3‑V‑Logik, getrennt | zu Q1 (BSS138) Gate | ESP RMT‑Output (R1) – **kein Filter‑Cap** (38 kHz Träger) |
| 5 | Laser Steuerung | 3,3‑V‑Logik, getrennt | zu Q2 (BSS138) Gate | ESP GPIO (R4 + C8 100 nF) |
| 6 | Trigger (active‑LOW) | Schalter gegen GND | MX‑Switch direkt zwischen GND und Pin | ESP `INPUT_PULLUP` (R3 + C7 100 nF Debounce), Logik invertiert |

> **Reihenfolge‑Begründung (verifiziert im Schaltplan 2026‑06‑08):**
> GND auf Pin 2 puffert die 5 V gegen die Signal‑Pins – eine abstehende Litze
> kann 5 V nur auf GND kurzschließen (Sicherung), nicht auf einen 3,3‑V‑GPIO.
> Der Eingang (Trigger) liegt am gegenüberliegenden Ende (Pin 6), weg von den
> schnell schaltenden IR-/NeoPixel‑Flanken. **Filter‑Caps nur auf TRIG/LASER**
> (C1/C2); IR und NeoPixel bekommen nur den 100‑Ω‑Serien‑R (R2/R1), kein C –
> sonst würde der 38‑kHz‑IR‑Träger bzw. die NeoPixel‑Flanke verschliffen.
> Silkscreen am Stecker beschriften: `1:5V 2:GND 3:NEO 4:IR 5:LAS 6:TRIG`.

> **Warum IR und Laser separat (nicht combined):** Tobias möchte den Laser
> auch ohne Schuss zum **Zielen** aktivieren können (entweder dauerhaft
> bei Setup oder für x Sekunden nach Trigger). Das geht nur mit getrennter
> Steuerung. Im Halloween‑Setup im Dunklen ist Schuss mit „Hier‑zielen‑Laser"
> deutlich angenehmer. Mit dem 6‑pol Klemmstecker (genau 6 Adern, keine
> Reversibilität) ist die alte „Plan A / Plan B"‑Frage (SBU‑Ader vorhanden?)
> hinfällig – IR und Laser sind **immer** getrennt verdrahtet.

**Steckverbinder:**

- **Station‑Seite:** `CN1` = **KF2EDGRC‑5.08‑6P** (Cixi Kefa, gewinkelter
  THT‑Header, 5,08 mm, ~8 A), auf dem Logic‑PCB. Der gekaufte grüne
  Schraub‑Plug (DIFLAX) ist das mating‑Gegenstück (KF2EDGK‑5.08‑6P‑Norm) –
  passt herstellerübergreifend auf jeden 5,08‑mm‑„2EDG"‑Header. **Raster am
  echten Plug gegenprüfen** (Anzeige war zwischen 3,81 und 5,08 mm
  widersprüchlich; 2EDG‑Bezeichnung + 8 A sprechen für 5,08 mm).
- **Wand‑Seite:** Kabel **direkt an die Wand‑PCB angelötet** (Zugentlastung
  über gedruckte Klemmleiste / Knoten im Knauf), kein Stecker.
- **Kabel:** ein einfaches **6‑adriges Mantelkabel** (z. B. summenge­schirmtes
  Steuerkabel oder 6× Litze ~0,25 mm²). Kein USB‑Spezialkabel, kein
  eMarker‑Durchgangstest mehr nötig. Für 5 V auf 1,5 m reicht 0,25 mm²;
  wer auf Nummer sicher geht, nimmt für 5 V + GND je 0,34 mm².

**ESD‑Schutz (auf dem Station‑Data‑Board, wie im funktionierenden Aufbau):**
Das ursprüngliche 4‑Kanal‑Array `USBLC6‑4SC6Y` (im alten USB‑C‑Board am
Stab‑Connector) wird durch **2× `USBLC6‑2SC6`** (je 2 Kanäle) ersetzt –
`USBLC6‑4SC6Y` war schlecht verfügbar, der Dual‑Typ dafür gut
(Reichelt/AliExpress, ~0,30 €). Sie bleiben am gewohnten Ort, dem
**Stab‑Connector CN1 auf dem Data‑Board** – dort schützen sie die ESP‑GPIOs
direkt am Kabel‑Eingang:

- **D1** schützt NeoPixel‑Daten + Trigger, **D2** schützt IR_TX + LASER_TX.
  Pin‑Belegung je IC: I/O1 = Pin 1 & 6, I/O2 = Pin 3 & 4, **GND = Pin 2,
  VBUS = Pin 5 → +5 V** (definiert die obere Klemmgrenze – nicht GND!).
- **Pass‑Through (inline) erzwungen:** das Signal läuft `_F` → Pin 1 → Pin 6 →
  `_FO` → R → CN1, damit das Bauteil im Layout direkt auf der Bahn sitzt
  (kein Stub).
- Die **Serien‑Widerstände R1–R4 (100 Ω)** bleiben (begrenzen ESD‑Strom in die
  GPIOs, dämpfen Ringing/EMV), ebenso `C1/C2` (100 nF) auf Trigger/Laser.
- Die früheren USB‑C‑Aufgaben (Hot‑Plug‑ESD, PD‑Verwechslungsschutz) entfallen –
  in eine grüne Schraubklemme passt kein Ladegerät.

> **Hinweis Signal‑Geschwindigkeit:** Die ultraniedrige Kapazität von
> USB‑ESD‑Arrays ist für uns irrelevant (NeoPixel ~800 kHz, IR 38 kHz,
> Trigger/Laser quasi DC). Der USBLC6‑2SC6 wurde nur wegen Verfügbarkeit
> gewählt; jedes 5‑V‑TVS‑Array täte es auch.

### 8.12 Pfostenhalter (seitlich)

Auf dem alten Halloween‑Setup‑Foto (siehe Bild 2026‑05‑08): Box auf einer
Seite eines Stahl‑Lochpfostens, Stab‑Halterung gegenüber. Genau dieses
Konzept übernehmen wir.

**Konstruktion:**

- M3‑Inserts (4×) in einer **Seitenwand** des Elektronik‑Moduls (nicht
  in der Service‑Rückwand → Pfosten bleibt fest, auch wenn Service
  geöffnet wird)
- Pfostenhalter‑**Adapter** als separates 3D‑Druckteil mit Schraubschlitzen
  zum Pfosten‑Lochbild
- Adapter wird mit M3 in die Box‑Inserts geschraubt + mit M5/M6 in den
  Pfosten
- Bei Pfosten‑Wechsel: nur Adapter neu drucken, Box bleibt unverändert

**Stab‑Halterung:**

- Auf der **gegenüberliegenden Seite** des Pfostens, mit eigenem
  Druckteil
- Hält den Stab in Ruheposition (z. B. mit gedrucktem Clip oder
  Magnethalterung)
- Ebenfalls über eigenen Adapter am Pfosten

**Vorteil seitlich vs. rückseitig:** Wenn der Pfosten‑Halter rückseitig
am Box wäre, würde die Service‑Rückwand bei jedem Pfosten‑Aufbau
betroffen sein. Seitlich montiert ist die Box „neutral" – Service geht
ohne Pfosten‑Demontage.

### 8.13 Anschluss‑Layout an den Außenwänden (Übersicht)

| Wand | Anschluss / Element | Begründung |
|---|---|---|
| **Front (Schallwand Speaker‑Modul)** | gedrucktes Schutzgitter um den Speaker, Status‑LED‑Bohrung Ø 5 mm | sichtbar von vorne, vom Spieler aus |
| **Front (Elektronik‑Modul)** | leer | keine Anschlüsse vorne, optisch ruhig |
| **Boden (Elektronik‑Modul)** | **Boden‑Klappe** mit DC‑Hohlbuchse + 6‑pol Stab‑Klemmstecker + (optional) OLED + Tasten (alles innen) | Schwerkraft, Wasser läuft ab; Konfig + Anschlüsse vereint |
| **Seitenwand (eine, Elektronik‑Modul)** | Pfostenhalter‑Adapter‑Inserts | seitlich am Pfosten montiert |
| **Rückwand (Elektronik‑Modul)** | fest gedruckt, innen Antennen‑Tasche | nicht greifbar, robust |
| **Speaker‑Rückwand (Speaker‑Modul)** | fest verschraubt, dauerhaft zu, 5° schräg integriert | Akustik (stehende Wellen) + Robustheit |
| **Außenwand‑LED** | SK6812RGBW hinter Klarsicht‑Inlay, Bohrung Ø 5 mm | sichtbar von vorne, in der Schallwand des Speaker‑Moduls |
| **Antenne** | intern in der festen Rückwand des Elektronik‑Moduls, kein Außenanschluss | nicht greifbar, robust, kurzes Pigtail (~5 cm) |

**Außenwand‑LED Detail:**

- Klarsicht‑Filament‑Inlay (PETG transparent) als „Lens" eingedruckt,
  oder Heißkleber‑Tropfen als Diffusor
- LED mit dem Hauptboard via 3‑pol JST‑PH gesteckt (austauschbar)
- Position: oben rechts in der Schallwand des Speaker‑Moduls (außerhalb
  des Speaker‑Volumens – Bohrung führt von außen ins Druck‑Material,
  nicht in die Speaker‑Kammer rein)

### 8.14 PCB‑Split: Logic‑Board + Audio‑Board (Rücken‑an‑Rücken‑Stack)

> **Entscheidung 2026‑06‑08 (Tobias):** Die Station‑Elektronik wird auf
> **zwei gestapelte PCBs** aufgeteilt, mit Nylon‑Abstandshaltern verschraubt
> und über ein kurzes Kabel verbunden. Hintergrund: Im kleinen
> Elektronik‑Kammer‑Volumen lässt sich so Grundfläche sparen, indem
> Audio‑Teil und Logic‑Teil übereinander statt nebeneinander liegen.
>
> **Detaillierte Bauteil‑ und Netz‑Aufteilung** (für die EasyEDA‑Umsetzung in
> zwei Schaltpläne): [`19-pcb-split-netliste.md`](19-pcb-split-netliste.md).
> **Stand 2026‑06‑08: Data‑ + Audio‑PCB sind geroutet/DRC‑sauber (V1)** –
> finaler Layout‑Stand, Design‑Rules und Pre‑Order‑Checkliste in Doc 19 § 8.

**Board‑Aufteilung:**

| Board | Bestückung |
|---|---|
| **Logic‑PCB** | ESP32‑S3‑DevKitC (BB1), NeoPixel‑Levelshifter (U1, 74AHCT125), Signal‑Filter (R1–R4, C1/C2) + ESD (D1/D2), 6‑pol Stab‑Klemmstecker (CN1), OLED‑Anschluss (CN2, optional), Status‑LED (H1), Inter‑Board (CN3). Bekommt seine **5 V vom Audio‑Board** übers Inter‑Board‑Kabel. |
| **Audio‑PCB** | DC‑Hohlbuchse 12 V (J1), F1 Polyfuse, **D3 SS34** (Verpolschutz), **MP2307‑Buck (5 V)**, PCM5102‑DAC (U2), TPA3110‑Verstärker (U3), Speaker‑Klemme (CN4), Inter‑Board (CN5). Power‑Entry‑Board. |

**Strom‑Architektur (umgedreht ggü. dem alten Single‑Board):** Das
**12 V kommt aufs Audio‑Board** (DC‑Buchse + Schutz + Buck dort), geht lokal
und kurz in den TPA3110 (gut für die Bass‑Transienten). Der Buck macht die
5 V, die übers Inter‑Board‑Kabel zum Logic‑Board gehen.

```
 VORNE (tief im Gehäuse)                    HINTEN (Boden-Klappe, Blick)
 ──────────────────────────────────────────────────────────────────────►

  ESP-DevKit + Module          GAP (M3-Nylon)          PCM5102 + TPA3110
  ▐██████████┐             ╎             ╎             ┌██████████▌
  ▐ nach     │  Logic-PCB  ╎             ╎  Audio-PCB  │     nach  ▌
  ▐ vorne    │═════════════╎             ╎═════════════│    hinten ▌
  ▐██████████┘   flache    ╎             ╎   flache    └██████████▌
                 Innenseite ╎             ╎ Innenseite
                            └─ nur niedrige SMD ─┘
                 Anschlüsse an der oberen Kante (zur Klappe)
```

**Mechanik (Rücken‑an‑Rücken / Clamshell):**

- Logic‑PCB so eingebaut, dass der **ESP nach vorne** zeigt (aus der
  Blickrichtung, wenn man von hinten in die Boden‑Klappe schaut).
- Audio‑PCB darüber gestapelt, Bauteile zeigen **entgegengesetzt** (nach
  hinten zur Klappe).
- Die beiden **flachen Innenseiten** schauen sich an → der Gap muss nur die
  niedrigen SMD‑Teile freihalten, **kein hohes Modul dazwischen**. Damit
  reicht ein **kurzer Standoff** (Höhe = höchstes Innenseiten‑Teil + Reserve)
  statt der ~20 mm aus dem ersten Entwurf.
- **Abstandshalter: M3 Nylon** (Tobias‑Wahl, konsistent mit den M3‑Inserts
  im Gehäuse) → Bohrungen auf beiden PCBs **Ø 3,2 mm**.
- Regel: auf die Innenseiten nur **niedrige Bauteile**, alle hohen Module
  nach außen. Alle **Anschlüsse an die obere (zur Klappe zeigende) Kante**,
  damit sie im eingebauten Zustand steckbar bleiben.

**Inter‑Board‑Kabel (die „wichtigsten Funktionen"):**

| Ader | Richtung | Hinweis |
|---|---|---|
| 5 V | Audio → Logic | trägt ESP + Stab‑LEDs (6× SK6812 über die Stab‑Klemme) + Status‑LED → ~1 A Spitze |
| GND | — | Rückleiter |
| I²S BCLK | Logic → Audio | |
| I²S LRCK | Logic → Audio | |
| I²S DIN | Logic → Audio | |
| XSMT | Logic → Audio | Mute‑Leitung vom ESP (XSMT‑Pin lt. Schaltplan) → PCM5102 (**wie im funktionierenden Single‑Board**, keine Änderung; GPIO‑Nr. mit § 7 abgleichen) |

= 6 Signale. **Stecker: JST‑XH 8‑polig gewinkelt (S8B‑XH‑A‑1, LCSC C5306549),
26 AWG, 5 V und GND je gedoppelt** – drückt den Spannungsabfall (die
Stab‑LED‑5 V müssen zusätzlich 1,5 m bis in den Stab) und gibt Reserve.
Designatoren: **`CN3` auf dem Logic‑Board, `CN5` auf dem Audio‑Board** – beide
**identisch belegt** (1/2:5V, 3/4:GND, 5:BCLK, 6:LRC, 7:DOUT, 8:XSMT), daher
**Kabel 1:1 gerade crimpen** (Pos 1↔1 … 8↔8), sonst läge 5 V auf XSMT.

> **26 AWG reicht locker:** Der Inter‑Board‑Hop ist nur ~5–10 cm. 26 AWG
> (~0,134 Ω/m) ergibt über die Schleife 5 V + GND ~0,027 Ω → bei 1 A Spitze
> ~27 mV Abfall, mit gedoppeltem 5 V/GND ~13 mV. Der JST‑XH‑Kontakt ist mit
> 3 A spezifiziert (AWG 22–30), die ~1 A liegen weit darunter. **Nicht
> verwechseln** mit dem 1,5‑m‑**Stab‑Kabel**, das dickeres 5 V/GND will
> (~0,34 mm² ≈ 22 AWG) – das ist eine separate Leitung.

`SCK`, `FMT`, `FLT`, `DEMP` der PCM5102 bleiben **lokal** auf dem Audio‑Board
(kein MCLK nötig, interne PLL; Pegel wie auf dem GY‑PCM5102‑Modul gesetzt).
**`XSMT` dagegen kommt wie im funktionierenden Single‑Board vom ESP
(XSMT‑Pin lt. Schaltplan) über `CN3`→`CN5`** – die Mute‑Steuerung bleibt
unverändert.

> **Beide Kabel‑Hops im 5‑V‑Pfad:** Die Stab‑LEDs hängen jetzt an
> Buck → Inter‑Board → Logic → 1,5 m Stab. Das gedoppelte 5 V/GND + ein
> kurzes Inter‑Board‑Kabel halten den Drop im Rahmen.

**OLED + 4 Tasten = optionales Modul (Entscheidung 2026‑06‑08):** Der
OLED‑Anschluss (8‑pol JST‑XH: GND, VCC, SCL, SDA, K1–K4) bleibt als
**Footprint auf dem Logic‑PCB**, wird aber nur in Stationen bestückt, die
ihn brauchen. Begründung: Die Konfiguration läuft künftig übers **Config‑Tool
per ESP‑NOW** (siehe [`18-config-tool.md`](18-config-tool.md)) – `SETUP_BEGIN`
→ Stab‑Trigger → `SETUP_TAKE` für die ID, `DISCOVER`/`CFG_WRITE` für
Lautstärke etc. Damit braucht **nicht jede Station ein eigenes OLED**.

- Die 4 Tasten‑GPIOs (35–38) + I²C bleiben geroutet, in OLED‑losen Stationen
  einfach frei.
- Die **Wand‑Status‑LED** (1× SK6812 außen) wird damit zur einzigen
  Pflicht‑Rückmeldung pro Station; der **Boot‑ID‑Blink** (N× Stationsfarbe,
  s. 2.6) bleibt wichtig, um die ID auch ohne Display abzulesen.
- Firmware‑Konzept: jede OLED‑Funktion muss auch übers Config‑Tool
  erreichbar sein (im Doc‑18‑Flow bereits so angelegt). Das OLED wird damit
  reiner Service‑Komfort für die „Meister‑Station" am Basteltisch.

**EasyEDA‑Organisation (zwei PCBs, ein Projekt):** Du brauchst **keine zwei
getrennten EasyEDA‑Projekte**. Lege im selben Projekt **zwei separate
Schaltpläne** an (nicht zwei Sheets eines Schaltplans!) – `Logic.sch` und
`Audio.sch` – und konvertiere **jeden in sein eigenes PCB** (`Convert to
PCB` pro Schaltplan). Getrennte Schaltpläne = getrennte Netzlisten =
getrennte Boards mit eigenem Umriss, eigener DRC und eigenem Gerber. Würde
man beides als zwei *Sheets eines* Schaltplans führen, packt EasyEDA alles
auf **ein** PCB – das wollen wir nicht.

- Das Inter‑Board‑Kabel ist auf beiden Schaltplänen je ein **JST‑XH‑8‑Symbol**
  mit **identisch benannten Netzen** (5V, GND, BCLK, LRCK, DIN, STBY) – so
  lässt sich die Belegung per Auge abgleichen. Eine board‑übergreifende ERC
  gibt es nicht, deshalb die Pinbelegung als Tabelle auf beide Sheets
  schreiben.
- Bestellung: zwei Gerber‑ZIPs bei JLCPCB; bei Bedarf zum Sparen gemeinsam
  panelisieren.

---

## 9. Iterationsplan

### Phase 1 – Lochraster‑Prototyp

**Hardware:** ESP32‑S3‑DevKitC‑1U N16R8 + GPIO‑Extension‑Board
(diymore „ESP32‑S3 GPIO Extension Board") als komfortable Anschluss‑Basis.
Alle Module per Schraubklemmen oder Pinheader anschließen, ohne Lötkolben.

**Test‑Sequenz:**

1. DevKitC einsetzen, Pigtail‑Antenne montieren, 12 V einspeisen.
2. PCM5102A + TPA3110 XH‑A232 + FR 10/4 anschließen, **Test‑Ton aus
   PSRAM/LittleFS** spielen. Dabei direkt die **Audio‑Mute‑Strategie B**
   prüfen: kein Pop beim Power‑On (STBY default LOW), kein Hiss zwischen
   Sounds (STBY in Standby), sauberer Übergang beim Sound‑Start. Falls
   bei leisen Sound‑Stellen Restrauschen hörbar: Plan A aktivieren
   (XSMT‑Brücken‑Modifikation am DAC, GPIO12 belegen).
3. ESP‑NOW Sender/Empfänger‑Test gegen ein zweites S3 (oder gegen ein
   Target‑Modul).
4. RMT IR‑Senden mit Test‑Empfänger validieren.
5. NeoPixel (Status + Stab‑LEDs) am RMT‑Pin testen (FastLED).
6. OLED + 4 Tasten am I²C/GPIO35–38 testen, Menü‑Skelett implementieren.
7. **Kombinationstest:** Audio + ESP‑NOW + RMT + OLED gleichzeitig –
   Audio darf keine Aussetzer haben, NeoPixel darf nicht flackern.

### Phase 2 – PCB für Halloween 2026

Sobald Phase 1 stabil läuft:

- KiCad/EasyEDA‑Layout für die Station, integrierter Buck‑Down 12 V → 5 V,
  alle Anschlüsse und Bauteile auf einem PCB.
- DevKitC sitzt auf 2 × 22 Pin‑Header‑Sockeln (steckbar, austauschbar).
- PCM5102A + TPA3116D2 + Buck‑Down ebenfalls als steckbare Module
  (einfacherer Service).
- Fertigung in JLCPCB‑Standard‑Auftrag.
- Bestückung von Hand oder mit JLC‑Assembly für SMD‑Teile.
- Test‑Aufbau, Klangprobe outdoor.

### Phase 3 – Optimierung danach

- **A/B‑Hörtest Box A vs. Box B** auswerten und Standard‑Speaker für die
  Serienfertigung festlegen. Mögliche Ausgänge:
  - Box A reicht → FR 7/4 bleibt, kompakte Bauform setzt sich durch
  - Box B gewinnt klar → FRS 8 wird Standard, größere Bauform
  - Beide haben Schwächen → dritter Treiber als Premium‑Option testen
    (z. B. JBL Stage1 41F Coax, Dayton ND91‑4, Tang Band W3‑1876S)
- Bassreflex‑Tuning der Box B feinjustieren (Port‑Länge mit gedruckten
  Einsätzen variieren).
- Spell‑Sounds in der Wand selbst (eigene Sound‑Engine pro Stab) –
  größeres Refactor.
- Optional Touch‑Buttons als alternative Bedienung (S3 hat 14 Touch‑Pins
  frei).

---

## 10. Beantwortete Detailfragen

Diese Fragen wurden im Verlauf der Konzeptionsdiskussion 2026‑05‑07
geklärt und hier zur Nachvollziehbarkeit gesammelt.

### Klingt WAV "noch super"?

Ja. WAV ist verlustfrei, also klanglich grundsätzlich besser als MP3 bei
gleicher Sample‑Rate. **22 kHz Mono 16‑Bit** ist für Halloween‑Sounds
absolut ausreichend. Mit dem 16‑MB‑Flash des S3 wäre auch 44,1 kHz drin,
falls Hörtests das nahelegen.

### Gilt der `WiFi.setSleep(false)`‑Workaround auch bei ESP‑NOW?

Ja. ESP‑NOW läuft auf der gleichen WLAN‑MAC‑Layer wie Standard‑WiFi.
Modem‑Sleep schaltet das Funkmodul kurzzyklisch ab; in diesen Phasen
werden eingehende ESP‑NOW‑Pakete verworfen. Auf S3 mit Dual‑Core ist das
weniger kritisch als auf C3, bleibt aber Best Practice.

### Logging: was statt `Serial.println`?

In Setup‑Phase und Boot ist `Serial.println` weiterhin OK. Im Hot‑Path
gehört es nicht hin. Ersatz‑Strategie: `#define INFINITAG_DEBUG`
Compile‑Time‑Schalter, in Production Events in `xQueue` mit separater
Low‑Prio‑Task. Field‑Debug optional via UDP‑Logger oder
ESP‑NOW‑Debug‑Topic.

### NeoPixel‑Anzahl

Mit S3 (8 RMT‑Channels, Dual‑Core) sind LED‑Zahlen über 100 problemlos
machbar. Für unseren Use‑Case (1 Status + 2 Stab = 3 LEDs) absolut
unkritisch. Erweiterungen bis ~50 LEDs ohne weitere Maßnahmen drin.

### Was ist ein DMA‑Buffer überhaupt?

Ein Speicherbereich, aus dem die Audio‑Hardware autonom Samples ausspielt.
Die CPU füllt den Buffer nach. Großzügige Buffer (mehr Samples × mehr
Buffer‑Stücke) geben Reserve gegen CPU‑Stalls, kosten dafür minimal RAM
und etwas Trigger‑Latenz. Für uns: 8 Buffer × 1024 Samples = 16 KB RAM,
~46 ms Latenz, ~370 ms Reserve. Code‑Snippet siehe 5.5.

### Was ist LittleFS‑Flash?

Ein Dateisystem, das **direkt im internen Flash des ESP32** läuft – wie
eine kleine eingebaute Festplatte. Files werden im Flash abgelegt,
programmatisch wie auf einer SD‑Karte gelesen, aber ohne die SD‑Hardware.
Wear‑Leveling und Power‑Fail‑Safety inklusive. Auf dem S3 mit 16 MB Flash
bleibt nach Code‑Partition genug Platz (~8–10 MB) für die WAV‑Bibliothek.

### Reicht 4 IDs nicht doch? Wie viele Stations‑IDs sind sinnvoll?

Mit dem OLED‑Menü beliebig viele. Empfehlung: 1–99 als Wertebereich, im
NVS gespeichert. Für die heutige Realität (3–4 Stationen) reichen die
ersten 4 Werte, der Rest ist Reserve.

### Service‑Klappe vs. Außenwand‑Display?

Service‑Klappe gewählt. Begründung:
- Outdoor‑Spritzwasser‑Schutz trivial (Schaumstreifen)
- Manipulationsschutz (Kinder kommen nicht ran)
- Klemmen statt Löten = OLED schnell tauschbar
- Live‑Status wird über die externe NeoPixel‑Status‑LED kompensiert.

### C3 vs. S3 – warum doch S3?

Drei Argumente: GPIO‑Bedarf für OLED + 4 Tasten überschreitet C3‑Reserve;
16 MB Flash + 8 MB PSRAM macht SD‑Karte überflüssig (4 GPIOs gespart);
Dual‑Core entspannt Audio + WiFi + RMT gleichzeitig. Außerdem ist der
DevKitC‑1U‑Form‑Faktor langfristig besser standardisiert verfügbar als
das C3‑SuperMini‑Klon‑Format.

---

## 11. Lebende Diskussionspunkte / nächste Themen

- [x] **Bauteilliste für Lochraster bestellen** ✅ erledigt 2026‑05‑08
      (alle Bestellungen raus, Eigenbestände identifiziert – Detail in
      [`10-bill-of-materials.md`](10-bill-of-materials.md), Abschnitt
      „Konkreter Bestellstand 2026‑05‑08")
- [x] **ESP‑NOW‑Paket‑Format** festgeschrieben ✅ 2026‑05‑18 in Abschnitt 3.4–3.7
      zusammen mit dem Config‑Tool‑Konzept. 36‑Byte `infinitag_packet_t`,
      Versionsbyte 0x01, CRC‑16/CCITT‑FALSE, 9 Nachrichtentypen + reservierte
      Bereiche. Siehe auch [`18-config-tool.md`](18-config-tool.md).
- [ ] Sound‑ID‑Mapping (1‑basiert vs. 0‑basiert) endgültig entscheiden,
      siehe Punkt 1a in [`11-offene-punkte.md`](11-offene-punkte.md)
- [ ] OLED‑Menü‑Struktur konkret entwerfen (Bildschirm‑Skizzen)
- [ ] Audacity‑Batch‑Skript für die Sound‑Aufbereitung (mono, EQ, LUFS,
      22 kHz, WAV)
- [x] **Lochkreis‑Ø der FR 7 verifiziert** ✅ 2026‑05‑09 (Datenblatt + Schieblehre):
      Korbflansch quadratisch 66 × 66 mm, Lochkreis Ø 74 mm, 4 × Ø 4,2 mm
      Schraubenlöcher, Korbflansch‑Dicke 1,7–2 mm, Sicke Ø 64 mm. Siehe 8.2.
- [ ] **Lochkreis‑Ø der FRS 8** aus Visaton‑Datasheet verifizieren
      (geschätzt ~85–95 mm) – für die Box‑B‑Schallwand‑Geometrie
- [x] **Gehäuse konstruktiv festgelegt** ✅ 2026‑05‑08, vereinfacht 2026‑05‑09
  - **Einteiliger Korpus** mit fest mitgedruckter Trennwand (zuvor:
    zweiteiliges Modul‑Konzept – verworfen, weil Druckhöhe bei „Front auf
    Bauplatte" ohnehin nur die Box‑Tiefe ist)
  - 3 Druckteile: Korpus + Speaker‑Rückwand + Boden‑Klappe (+ Pfostenhalter‑Adapter)
  - Boden‑Klappe vereint **alles**: DC‑Buchse, 6‑pol Stab‑Klemmstecker,
    (optional) OLED + 4 Tasten, Klemmleisten
  - Antenne intern in der mitgedruckten Rückwand der Elektronik‑Kammer
  - Pfostenhalter seitlich (eine Seite Box, andere Seite Stab‑Halter)
  - Details: Abschnitte 8.7–8.14
- [x] **Stab‑Connector neu definiert** ✅ 2026‑05‑08, **geändert 2026‑06‑08**
  - ~~USB‑C 3.x mit 6 Adern~~ → **grüner 6‑pol PCB‑Schraubklemmstecker**
    (2026‑06‑08), Wand‑Kabel direkt angelötet
  - Trigger umverdrahten auf active‑LOW (Wand‑Modifikation, R1 entfällt)
  - 3,3‑V‑Ader entfällt (war nur Trigger‑Pullup)
  - IR + Laser separat (Tobias möchte Laser‑Zielen ohne Schuss) – mit dem
    6‑pol Stecker immer getrennt, keine Plan‑A/Plan‑B‑Frage mehr
  - Details: Abschnitt 8.11
- [x] **PCB‑Split festgelegt** ✅ 2026‑06‑08
  - 2 PCBs gestapelt: Logic‑Board + Audio‑Board, Rücken‑an‑Rücken,
    M3‑Nylon‑Standoffs, Inter‑Board‑Kabel JST‑XH 8‑pol
  - 12 V + Buck aufs Audio‑Board, 5 V rüber zum Logic‑Board
  - OLED optional, EasyEDA: ein Projekt, zwei Schaltpläne → zwei PCBs
  - Details: Abschnitt 8.14
- [x] **Antennen‑Strategie festgelegt** ✅ 2026‑05‑08
  - Option B: IPEX‑Pigtail mit interner Whip‑Antenne, vertikal an der
    Service‑Rückwand fixiert, kein SMA außen
- [x] **Stab‑Kabel‑Beschaffung vereinfacht** ✅ 2026‑06‑08
  - USB‑C‑Spezialkabel + eMarker‑Durchgangstest **entfällt** mit dem
    Klemmstecker. Stattdessen einfaches 6‑adriges Mantelkabel
    (5 V/GND je ~0,34 mm², Signale ~0,25 mm²). Kein SBU‑Test mehr nötig.
- [x] **ESD‑Schutz Stab festgelegt + im Schaltplan umgesetzt** ✅ 2026‑06‑08
  - 4‑Kanal `USBLC6‑4SC6Y` → **2× `USBLC6‑2SC6`** (D1 + D2) **auf dem
    Station‑Data‑Board, am Stab‑Connector CN1**, Pass‑Through inline
    (`_F` → Pin 1/6 → `_FO`). VBUS (Pin 5) an +5 V, GND an Pin 2.
    R1–R4 (100 Ω) bleiben. USB‑Verwechslungsschutz/PD‑Klemmung hinfällig.
- [ ] **Wand‑Hardware umbauen**: Trigger umverdrahten gegen GND, R1 (10 kΩ)
      entfernen, Kabel direkt anlöten (kein Stecker). ESD sitzt auf der
      Station (s. o.), die Wand‑PCB braucht **kein** eigenes ESD‑Array.
      Siehe [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md),
      [`02-hardware-wand.md`](02-hardware-wand.md) und Punkt 23/26 in
      [`11-offene-punkte.md`](11-offene-punkte.md)
- [ ] **3D‑Modelle bauen** in Fusion 360: Box A (FR 7/4 geschlossen) und
      Box B (FRS 8 Bassreflex) als **eine parametrisierte Konstruktion mit
      zwei Konfigurationen**, einteiliger Korpus (siehe 8.7–8.9)
- [ ] **A/B‑Hörtest planen**: gleicher TPA3110/PCM5102‑Aufbau, Boxen
      einzeln anschließen, Trefferlaut‑Bibliothek durchspielen,
      subjektive Bewertung notieren

---

## 12. Verweise

- Übergeordneter Kontext: [`01-system-uebersicht.md`](01-system-uebersicht.md)
- Vorgänger‑Hardware: [`03-hardware-wand-station.md`](03-hardware-wand-station.md)
- Offene‑Punkte‑Liste mit Punkt 18 (Refactor): [`11-offene-punkte.md`](11-offene-punkte.md)
- Bauteilstamm: [`10-bill-of-materials.md`](10-bill-of-materials.md)
- ESP32‑S3 Referenzen:
  - [Espressif ESP32‑S3‑DevKitC‑1 Getting Started](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/hw-reference/esp32s3/user-guide-devkitc-1.html)
  - [Lonely Binary ESP32‑S3 IPEX External Antenna Board](https://lonelybinary.com/en-us/products/esp32-s3-ipex)
- ESP‑NOW + I²S Erfahrungsberichte:
  - [Phil Schatzmann – Audio Tools für ESP32](https://www.pschatzmann.ch/home/2022/04/15/using-a-esp32-c3-with-my-arduino-audio-tools-library/)
  - [Espressif I²S Programming Guide](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/api-reference/peripherals/i2s.html)
- LittleFS:
  - [LittleFS GitHub](https://github.com/littlefs-project/littlefs)
  - [Arduino‑ESP32 LittleFS Guide](https://github.com/lorol/LITTLEFS)
