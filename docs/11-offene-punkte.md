# 11 – Offene Punkte / Diskrepanzen / Roadmap

Lebende Liste. Bei jedem Eintrag steht die Quelle (Datei + Zeile bzw.
Schaltplan‑Block), damit sich beim Aufgreifen sofort verifizieren lässt.

## Diskrepanzen Code ↔ Hardware

### 1. Welcher Sound‑Sketch läuft auf dem Sound‑ESP? ✅ geklärt (2026‑05‑07)
- **Produktiv:** `ESP32I2S_S3_MMC_Test.ino`
  (`/Volumes/basteln/Arduino/ESP32I2S_S3_MMC_Test/`, Stand 2023‑10‑01) –
  MAX98357 + SD_MMC + I²C‑Slave auf Adresse 1, Switch über Sound‑IDs 0..13.
  Bestätigt von Tobias.
- **Experiment, nicht produktiv:** `InfinitagWandStation.ino`
  (`/Volumes/basteln/Arduino/InfinitagWandStation/`, Stand 2024‑10‑05) –
  AudioKit + Touch‑Trigger. Nur als Parallel‑Test auf einem
  AI‑Thinker‑LyraT‑Board entstanden.
- **Aufräum‑Empfehlung:** den Experiment‑Sketch entweder mit einem deutlichen
  „NICHT FÜR DIE WAND STATION“‑Header versehen oder in einen
  `experimental/`‑Unterordner verschieben, damit künftige Sessions nicht
  wieder zweifeln. Siehe [`07-software-wand-sound.md`](07-software-wand-sound.md).

### 1a. Sound‑ID‑Mapping vs. Dateinamen auf SD/NAS
- **Quelle Code:** `ESP32I2S_S3_MMC_Test.ino::chooseSound()` mappt
  ID 0 → `/sound-effects/1_booAndLaugh.mp3`, ID 1 → `/sound-effects/2_bubbles.mp3`,
  …, ID 13 → `/sound-effects/14_witch.mp3` (1‑basierte Numerierung,
  `1_*.mp3` … `14_*.mp3`).
- **Quelle NAS:** `/Volumes/basteln/Halloween/Lasertag/StationSounds/sound-effects/`
  enthält `0_booAndLaugh.mp3` … `13_witch.mp3` (0‑basierte Numerierung).
- **Konsequenz:** Wenn man die Files vom NAS 1:1 auf die SD‑Karte legt,
  findet der Sketch keine der Dateien.
- **Was zu tun ist:** Entweder die SD‑Karte mit umbenannten Dateien
  bespielen (1‑basiert wie im Code) oder den Sketch auf 0‑basierte Namen
  umstellen. Letzteres wäre konsistenter mit dem NAS‑Stand.

### 1b. SD‑Karten‑Anschluss am Sound‑ESP ✅ geklärt (2026‑05‑07)
- **Auflösung:** Tobias hat für den Sound‑Slot ein ESP32‑S3‑DevKit mit
  **eingebautem MicroSD‑Slot** verwendet. GPIO38/39/40 sind dort schon
  auf der Modul‑Platine als SDIO‑CMD/CLK/D0 verdrahtet, deshalb taucht im
  Wand‑Station‑Schaltplan kein eigener SD‑Slot auf.
- **Was zu tun ist (Doku):** Genauen DevKit‑Typ noch ergänzen, sobald
  Tobias bei der Hardware ist (Hersteller / Bezeichnung).

### 2. Hartcodierte HTTP‑URL im Target
- **Quelle:** `Infinitag Target/src/main.cpp::hitAction()`, Zeile mit
  `http.begin("192.168.178.154", 80, "/trigger_effect?sound=6");`.
- **Erwartetes Verhalten:** dynamisch zusammengebauter `serverPath` mit
  IP‑Prefix + IP‑Block + Sound‑ID.
- **Konsequenz:** Targets feuern aktuell immer Sound 6 an die feste IP
  192.168.178.154. Die schöne `serverPath`‑Logik ist vorhanden, aber wird
  nicht verwendet. Vor dem nächsten Halloween fixen.

### 2b. Zwei GX16‑Stecker an der Station – geklärt ✅ (2026‑05‑07)

- **Kleiner GX16** → Wand‑/Zauberstab‑Anschluss (7‑polig, Signal + Versorgung)
- **Großer GX16** → Stromversorgung der Station (Custom‑Kabel vom Netzteil)
- Beide sind aktuell mit Custom‑Kabeln konfektioniert und haben dieselben
  Probleme: aufwändige Fertigung, kein Schutz gegen unbeabsichtigtes
  Abziehen. Lösungsansätze siehe Punkt 23 (Wand‑Kabel) und Punkt 27 (Stromversorgung).

### 3. Wand‑Connector‑Pinbelegung ist gespiegelt
- **Quelle:** Schaltplan Wand V2.1 (Pin 1 = `VCC_5V`, Pin 2 = `VCC_3.3V`),
  Schaltplan Wand‑Station Dev Board V1 (Pin 1 = `ESP_S_3V3`, Pin 2 = `5V`).
- **Konsequenz:** Wenn man ein 1:1 Kabel verwendet, vertauscht man 5 V und
  3,3 V – das zerschießt zumindest den Trigger‑Pullup, im schlimmsten Fall
  die NeoPixel oder den Laser.
- **Verifikationspunkt:** Existiert ein „Cross‑Cable“ zwischen den Boards?
  Oder hat sich beim Schaltplan‑Editieren ein Pin‑Reverse eingeschlichen?
  Vor dem nächsten Wand/Station‑Verbinden mit Multimeter prüfen.

### 3a. NeoPixel‑Datensignal über USB‑C‑CC‑Pin kommt nicht durch (V2) ✅ GELÖST in V3 (2026‑06‑21)

- **Symptom:** Die 4 SK6812/WS2812B‑4020 am Zauberstab flackern, zeigen
  falsche Farben („Weiß wird Rot"), verlieren Bits, die Fehler werden zur
  LED4 hin schlimmer (LED4 bleibt dunkel), bei jedem ESP‑Reset anders. Die
  Stations‑LEDs an H1 laufen dagegen einwandfrei.
- **Diagnose‑Kette (Oszilloskop Rigol DS1202, 10×‑Tastkopf):**
  1. **Stromversorgung ok** – Laser und (Test‑)weiße LED am Stab laufen
     tadellos und ziehen mehr Strom als die SK‑LEDs. VBUS/GND über das
     Kabel sind also belastbar.
  2. **Level‑Shifter U4 (74AHCT125) ok** – VCC (über C11) = 4,95 V; genutztes
     Gate 1 ist eingeschaltet (Pin 1 = 1OE = 0 V; Pin 4/10/13 = ~4,95 V =
     ungenutzte Gates korrekt aus). Ausgang **NEO_DATA_5V_AHCT (Pin 3)** =
     sauberes ~4,7‑V‑Signal mit steilen Flanken (~22 ns).
  3. Die zuerst gemessenen **3,25 V** lagen an der **Stations‑LED**, die an
     der **3,3‑V‑Eingangsseite (NEO_DATA_33V, Pin 2)** hängt – das war eine
     falsche Fährte, nicht das eigentliche Signal zum Stab.
  4. Kabel‑DC‑Widerstand der Datenader wurde mit ~1,1 Ω gemessen
     (vermutlich anderes Kabel/andere Ader – mit genau diesem 40G‑Kabel auf
     CC gegenchecken, siehe unten).
  5. **Entscheidend:** Mit einem **USB‑C‑Breakout nach dem Kabel** gemessen –
     auf **CC1 UND CC2** kommt **kein Datensignal** an, nur Rauschen
     (±100–200 mV bei 50 mV/div). Von den ~4,7 V am Ausgang bleibt nichts.
- **Ursache:** **NEO_DATA liegt auf den CC‑Pins (CC1=A5, CC2=B5)** des
  Stab‑Connectors. **USB‑C‑Kabel verbinden CC NICHT als durchgehende
  Datenader** – CC ist für Orientierungs‑Erkennung, Konfiguration, VCONN und
  (bei E‑Marker‑Kabeln) den E‑Marker‑Chip reserviert. Das verwendete
  **40 Gbps / 240 W USB4‑Kabel** führt CC über den E‑Marker → Datensignal
  blockiert; nur Reste koppeln durch.
- **Warum das alles erklärt:** Die anderen Signale liegen auf Pins, die das
  Kabel **durchverbindet**, und funktionieren deshalb:
  | Signal | USB‑C‑Pin | Kabel führt durch? | funktioniert? |
  |---|---|---|---|
  | Laser | D+ (Dp1/Dp2) | ja (USB2‑Paar) | ✅ |
  | Trigger | D− (Dn1/Dn2) | ja | ✅ |
  | IR | SBU1/SBU2 | ja (in USB4 verdrahtet) | ✅ |
  | **NeoPixel** | **CC1/CC2** | **nein** | ❌ |
- **Sofort‑Workaround (V2, für die Diagnose):** einzelner Draht von
  **NEO_DATA_5V_AHCT** (U4 Pin 3) direkt an **Stab‑LED1‑DIN**, am Kabel vorbei,
  gemeinsame GND. Strom/Laser/Trigger/IR weiter über USB‑C.

#### ✅ Umgesetzte Lösung in V3 (neues PCB, 2026‑06‑21)

Der **USB‑C‑Connector wurde komplett aufgegeben.** Stab und Station sind jetzt
über ein **direktes Kabel mit 2×3‑Header (J1, X3‑2.54)** verbunden, NEO_DATA
liegt auf einer **eigenen, sicher durchverbundenen Ader** – das CC‑Problem ist
damit konstruktiv erledigt. Beim Inbetriebnehmen des neuen PCB kam dann eine
ganze Kette weiterer Befunde, alle gelöst:

1. **Laser leuchtete nicht (LED/IR schon).** Treiber‑Topologie IR und Laser
   identisch (BSS138 Low‑Side). Messung: Gate ~3,3 V (Signal da), Drain 4,35 V
   (statt ~0 V) → Q2 schaltete nicht. **Ursache: kalte Lötstelle an Q2.**
   Nachgelötet → Laser läuft. (Merksatz: Laser wird im Test‑Sketch **nur über
   K3** geschaltet, nicht über den Trigger.)
2. **Grün‑Aussetzer am Stab, Station sauber.** Reine Signalintegrität auf dem
   Sprung Station‑LED8‑DOUT → Stab‑LED1‑DIN. GRB(W)‑Reihenfolge: Grün ist das
   erste Byte → bei degradierter Leitung am anfälligsten. **Serienwiderstand
   war mit 100 Ω (Station R1) + 470 Ω (Stab R1) = 570 Ω zu hoch** (überdämpft).
   - 570 Ω → Grün fällt aus, 100 Ω → Überschwingen (Grün zu hell). **Sweet Spot
     ≈ 150 Ω.**
3. **4020‑LEDs grundsätzlich unzuverlässig.** Mehrfach getauscht, immer dieselben
   Fehler; die winzigen WS2812B‑4020 sind beim Handlöten zu heikel (Hitze, Pads
   darunter). **→ Wechsel auf SK6812 5050 RGBW** – lief sofort, robuster, größere
   Pads. Damit kommt der **eigene Weiß‑Kanal** dazu → Firmware auf **`NEO_GRBW`**
   (32 Bit/Pixel) umgestellt.
4. **Bei 100 % Helligkeit fehlte Magenta (R+B) – nur Rot sichtbar.** Kein
   Die‑/Strom‑Defekt (Blau solo + Cyan liefen). In GRBW liegt das **Rot‑Byte
   direkt vor dem Blau‑Byte**; bei vollen `0xFF`‑Bytes „verschmiert" Rot ins
   Blau → Blau wird als 0 gelesen. Bei 20 %/`0x33` trat es nie auf, erst 100 %
   (`0xFF`) macht es sichtbar. Gegenprobe Helligkeit 110 → Magenta korrekt.
   → Bestätigt: **Signalmarge bei vollen Bytes**, Strecke grundsätzlich grenzwertig.
- **Finale Lösung am Stab: 74AHCT1G125‑Buffer am Stab‑Eingang.** Single‑Gate‑
  Variante des Station‑U1 (74AHCT125), SOT‑23‑5, active‑LOW /OE. Verdrahtung:
  /OE (Pin 1) → GND, A (Pin 2) ← NEO‑Daten vom Kabel, GND (Pin 3) → GND,
  Y (Pin 4) → 0‑Ω‑Platzhalter → erste LED‑DIN, VCC (Pin 5) → +5 V, **100 nF an
  VCC/GND**. Der Buffer regeneriert das Signal lokal auf saubere 5‑V‑Flanken →
  Helligkeit, Bitfolge, Kabellänge und Serienwiderstand werden unkritisch.
- **Bauteil‑/Bestückungs‑Entscheidungen Stab V3 (verbindlich):**
  - LEDs: **SK6812 5050 RGBW** (statt WS2812B‑4020), Datenformat `NEO_GRBW`.
  - **74AHCT1G125 (1G‑Single‑Gate)** als Re‑Driver am Stab‑Eingang, /OE→GND.
  - Serienwiderstand am Buffer‑**Ausgang** als **0‑Ω‑Platzhalter** (Tuning‑Option,
    ~50–100 Ω falls der Ausgang in die LEDs klingelt) – am Eingang nicht nötig.
  - Entkopplung: **100 nF je LED** + **ein 10–22 µF Bulk** nah am LED‑Cluster
    (fängt den Weiß‑Kanal‑Stromstoß ab; bei 2 LEDs unkritisch, aber empfohlen).
  - Stab‑Bestückung aktuell **2 LEDs** (Plan: 2–4).
- **Firmware‑Stand `Station/src/main.cpp` (2026‑06‑21):** `NEO_GRBW`,
  `NEOPIXEL_COUNT 2`, Helligkeit **110** (bis Buffer verbaut; danach auf 255).
  Dauer‑Sichttest `neopixelCyclePalette()` (R→Gelb→Grün→Cyan→Blau→Magenta→W,
  dann Mischfarben mit W‑Kanal). Der frühere „Oszi‑Modus" (Dauer‑Weiß) ist
  entfernt.

### 4. Verifizierung des IR‑Receiver‑GPIO im Target
- **Quelle:** Code (`#define IR_PIN 15`) vs. Schaltplan‑Net `IR_RCV` ohne
  klar lesbare GPIO‑Nummer im Bild.
- **Was zu tun ist:** Beim nächsten Flash über Serial Monitor mit IR‑Sendung
  prüfen, ob `IrReceiver.decode()` true wird. Falls nicht: alternative
  GPIOs (z. B. 7) testen oder PCB‑Layout in Easyeda öffnen und Net `IR_RCV`
  rückwärts verfolgen.

### 5. Webserver‑Port unklar (8080 vs. 80)
- **Quelle:** `InfinitagWand.ino` startet `WebServer server(8080)`,
  `InfinitagTargetTest.ino` ruft `http://...:8080/...` auf. Das produktive
  PlatformIO‑Target benutzt `http.begin(serverPath.c_str())` ohne expliziten
  Port (also 80) – aber wegen Punkt 2 wird das aktuell gar nicht ausgeführt.
- **Was zu tun ist:** Bei Fix von Punkt 2 explizit Port 8080 in der URL
  setzen oder den Webserver auf Port 80 ändern. Letzteres ist sauberer.

### 6. Sound‑Liste im Code ≠ Sound‑Liste auf Platte
- **Quelle:** `InfinitagWandStation.ino` enthält 9 Einträge,
  `/Volumes/basteln/Halloween/Lasertag/StationSounds/sound-effects/` enthält
  14 mp3 mit eigenem Numerierungsschema (`0_*.mp3` … `13_*.mp3`).
- **Konsequenz:** Sound‑IDs, die das Target schickt (Wert 0..13), passen
  nicht zur Array‑Reihenfolge im Sketch.
- **Was zu tun ist:** Beim Sound‑ESP‑Refactor (Punkt 1) Mapping ID → Datei
  fest definieren – am einfachsten über das numerische Präfix im Dateinamen
  beim Mounten der Karte.

### 7. Sound‑Quelle SD vs. Flash
- **Quelle:** Beide Sound‑Sketche (`ESP32I2S_S3_MMC_Test`,
  `InfinitagWandStation`) nutzen eine SD‑Karte; das Wand‑Station Dev Board V1
  hat aber keinen SD‑Slot eingezeichnet.
- **Was zu tun ist:** Klären wie die SD heute angeschlossen ist (siehe
  Punkt 1b). Falls dauerhaft unhandlich, die MP3s in den 8 MB Flash der
  ESP32‑S3 packen (LittleFS / SPIFFS) und auf `connecttoFS` mit LittleFS
  umstellen.

### 8. „Wand selbst sollte Spell‑Sound spielen“ ist nicht implementiert
- **Quelle:** `Halloween/Lasertag/Spell/*.mp3` existiert, im Code wird
  weder beim Trigger noch beim Schussbestätigen ein Spell‑Sound abgespielt.
- **Was zu tun ist:** beim Sound‑ESP‑Refactor zusätzlich einen
  "Spell"-Bereich im Befehlsraum reservieren (z. B. Sound‑IDs ≥ 100 spielen
  aus `/spell/`), und im `InfinitagWand.ino::shoot()` zusätzlich zur Schuss‑
  Logik einen `sendInt()` für den Spell auslösen.

## Architekturfragen / Refactor

### 9. MQTT statt HTTP?
- **Tobias' Aussage in der Konversation:** „wenn ein Treffer per MQTT
  gemeldet wurde sendet der eine arduino zum anderen den internen befehl…“
- **Code‑Stand:** kein MQTT vorhanden, weder in der Wand‑Station noch im
  Target. Aktuell HTTP‑GET.
- **Fragen, die wir gemeinsam klären sollten:**
  - Ist MQTT schon angedacht/getestet, oder verwechselte sich der Begriff
    mit HTTP?
  - Falls MQTT: würde sich ein kleiner MQTT‑Broker (Mosquitto) auf einem
    Raspberry Pi im Netz lohnen? Vorteil: viele Targets, ein zentrales
    Spielsteuerungs‑Topic, einfaches Logging.

### 10. Spielmodi außerhalb Halloween
- Original‑Infinitag war für Mehrspieler/Mehrteam‑Spiele gedacht.
  `Infinitag_Core` reserviert dafür schon Felder (`gameId`, `teamId`,
  `playerId`).
- Roadmap‑Idee: ein „Game‑Manager“ als kleiner Service (Pi, Web‑UI) der
  sieht, welche Targets/Wands online sind, Treffer zählt, Spiele startet.
- Erstmal niedrige Prio, aber lohnt sich in einer eigenen Datei (Vorschlag:
  `12-roadmap-spielmodi.md`) festzuhalten, wenn wir konkret werden.

### 11. Ein „Spieler‑Wand“ mit eigener Sound‑Box pro Spieler
- Ursprüngliches Konzept (siehe Sensor‑Befehle in `Infinitag_Core`):
  Westen mit eigenen IR‑Sensoren, jeder Spieler hat seine eigene Box.
- Halloween‑Setup nutzt das nicht; aber wenn man später mit den Kindern
  „echtes“ Lasertag spielen möchte, ist das die Richtung.

## Hardware‑To‑dos

### 12. Welche ESP‑Module konkret in der Wand Station ✅ teilweise geklärt
- Sound‑Slot (`ESP_S3_*`): **ESP32‑S3‑DevKit mit eingebautem MicroSD‑Slot**
  (bestätigt 2026‑05‑07). Konkrete Modul‑Bezeichnung noch nachreichen.
- Logic‑Slot (`ESP_S_*`): vermutlich klassisches ESP32‑DevKit (NodeMCU‑32S /
  DevKit V1) – die GPIOs (4, 14, 25, 27 + I²C 18/19) und das Pin‑Mapping
  passen dazu. Bestätigung am realen Aufbau steht aus.

### 13. Pull‑ups auf I²C
- Im Schaltplan keine Pull‑ups auf SCL/SDA. Funktioniert mit den internen
  Pull‑ups (~45 kΩ); für robusteren Bus 4,7 kΩ extern auf 3,3 V legen.

### 14. Niedrigere Lautstärke an MAX98357
- **Aktuell:** VIN am MAX98357 ist im Schaltplan auf 3,3 V geroutet → max.
  ~0,5–0,7 W RMS an dem 4 Ω Lautsprecher.
- **Verbauter Speaker:** Visaton VS‑FR7/4 (4 Ω, 5 W RMS Nennbelastbarkeit).
  Der Speaker ist also **deutlich überdimensioniert**, der Verstärker bremst.
- **Mod:** VIN‑Trace auf 5 V umlegen → ~1,8 W RMS (typisch) bzw. bis 3,2 W
  bei 5,5 V. Speaker hält das problemlos aus.
- **Was zu tun ist:** Bei nächster PCB‑Revision direkt 5 V vorsehen;
  am bestehenden Board Trace‑Cut und Drahtbrücke. Empfohlen ist diese Mod
  vor jedem weiteren „lauter machen“-Versuch, weil sie der mit Abstand
  größte Hebel ist.

### 15. Wand‑Gehäuse als 3D‑Druck dokumentieren
- Aktuell nicht im Halloween‑Ordner zu finden. Tobias druckt das auf einem
  3D‑Drucker; Datei nachlegen oder Modell‑Quelle (Printables/Thingiverse)
  hier verlinken.

### 16. Halloween‑Props als eigene Wissensbasis‑Sektion
- **Quelle:** PlatformIO‑Projekt `Infinitag Prison` (ESP32‑S3,
  Servo‑gesteuertes Gefängnis‑Modell mit Trigger‑Eingang) und Vorgänger
  `HalloweenGefaengnis.ino` (Arduino‑Variante, fast identisch).
- **Was zu tun ist:** Eigene MD `12-halloween-props.md` anlegen, mit allen
  Halloween‑Effekt‑Modulen, die per Schaltausgang vom Target getriggert
  werden (Prison, ggf. weitere Spider/Skeleton‑Sketches).

### 17. Veraltete oder leere Schwester‑Projekte einsortieren
- `/Volumes/basteln/PlatformIo/Projects/Infinitag Tagger/` – PlatformIO‑Skeleton
  für ESP32‑C3, **leer** (Default `myFunction(2,3)` aus Template). Vermutlich
  als zukünftige Refactor‑Plattform vorbereitet, aber nichts implementiert.
- `/Volumes/basteln/PlatformIo/Projects/Infinitag Prison - Kopie/` – Kopie
  des Prison‑Projekts, vermutlich Backup/Variante. Inhaltlich prüfen, ob
  abweichend.
- `/Volumes/basteln/Arduino/Target/` – älteres Target‑Projekt mit
  KiCad‑Schaltplan (Arduino‑Nano‑basiert, 2022). Vorgänger des heutigen
  ESP32‑S3‑Targets V3.2.
- `/Volumes/basteln/Arduino/HalloweenWand/` – einfacher Wand‑Sketch
  (2022‑09‑25), ohne WiFi, sendet konstant `cmdValue=255`. Wahrscheinlich
  der ursprüngliche Wand vor dem Logic‑ESP‑in‑Station‑Refactor.
- **Was zu tun ist:** entscheiden, was archiviert wird (Verzeichnis
  `legacy/` o. ä.) und in der Wissensbasis benennen, damit künftige
  Sessions sich nicht wundern.

## Architektur‑Refactor (Stand 2026‑05‑07: V2‑Konzept festgeschrieben)

> **Statusüberblick** (entstanden in der Konzept‑Session 2026‑05‑07):
>
> | Punkt | Entscheidung |
> |---|---|
> | Plattform Station | **ESP32‑S3‑DevKitC‑1U N16R8** (16 MB Flash, 8 MB PSRAM, Dual‑Core, externe Antenne via Pigtail) ✅ |
> | Anzahl Stäbe pro Station | **genau 1** (Multi‑Spieler über mehrere Stationen) ✅ |
> | Kommunikation Station ↔ Targets | **ESP‑NOW** (kein HTTP, kein Router) ✅ |
> | Audio‑Pfad | **I²S → PCM5102A → TPA3116D2 → Visaton FR 10/4** ✅ |
> | Audio‑Format | **WAV 22 kHz Mono 16‑Bit** ✅ |
> | Audio‑Quelle | **LittleFS im internen 16 MB Flash** (kein SD‑Modul) ✅ |
> | Konfiguration | **OLED 1,3" SH1106 + 4‑Tasten‑Modul, hinter Service‑Klappe** ✅ |
> | Live‑Status außen | **1 SK6812RGBW** auf Außenwand (Daisy‑Chain mit Stab‑LEDs) ✅ |
> | Erster Hardware‑Schritt | **Lochraster‑Prototyp mit DevKitC + GPIO‑Extension‑Board** ✅ |
> | Folgeschritt | eigenes PCB für Halloween 2026 ✅ |
>
> Detail‑Diskussion und GPIO‑Plan v7 in
> [`12-refactor-station-v2.md`](12-refactor-station-v2.md).
>
> **Plattform‑Wechsel‑Notiz:** Die Diskussion startete mit ESP32‑C3 SuperMini.
> Wechsel auf S3‑DevKitC‑1U am 2026‑05‑07 wegen GPIO‑Reserve für OLED + 4 Tasten,
> 16 MB Flash erlaubt LittleFS statt SD, Dual‑Core entspannt Audio‑Architektur.

### 18. Station‑Umbau: ein ESP32 statt zwei ESPs ✅ Plattform gesetzt (S3)

- **Motivation:** Das aktuelle Zwei‑ESP‑Design (Logic‑ESP + Sound‑ESP, I²C‑Bus
  dazwischen) ist aufwendig und benötigt ein eigenes PCB. Für den
  Outdoor‑Halloween‑Betrieb ist eine externe WLAN‑Antenne sehr wünschenswert
  (bessere Reichweite im Vorgarten).
- **Gesetztes Board:** **ESP32‑S3‑DevKitC‑1U N16R8** (offizielles
  Espressif‑Referenzdesign, Klone u. a. von diymore, AZ‑Delivery, FREENOVE,
  WaveShare). 16 MB Flash, 8 MB Octal PSRAM, Dual‑Core LX7 @ 240 MHz,
  IPEX‑Anschluss für externe 2,4 GHz‑Antenne.
- **Begründung Plattform‑Wahl:** Die Diskussion startete mit dem ESP32‑C3
  SuperMini. Wechsel auf S3 wegen drei Argumenten:
  1. **OLED‑Konfiguration mit 4 Tasten** kostet 6 GPIOs – beim C3 (~13 nutzbare
     GPIOs) wäre das mit den Pflicht‑Funktionen kollidiert; beim S3
     (~38 nutzbare GPIOs) ein Bruchteil des Budgets.
  2. **16 MB Flash + 8 MB PSRAM** macht eine SD‑Karte überflüssig –
     LittleFS reicht für die komplette Sound‑Bibliothek mit ~6× Reserve.
  3. **Dual‑Core** entspannt Audio + WiFi/ESP‑NOW gleichzeitig – keine
     Single‑Core‑Workarounds nötig.
- **Standardisierung:** Das DevKitC‑1U‑Format ist offizielles Espressif‑Layout
  und langfristig von vielen Lieferanten verfügbar – PCB‑Lock‑In‑Risiko
  geringer als beim C3‑SuperMini‑Klon‑Format.
- **GPIO‑Budget (Plan v7):** 13 Pins genutzt, ~25 Pins frei für Erweiterungen.
  Vollständiger Plan in
  [`12-refactor-station-v2.md`](12-refactor-station-v2.md), Abschnitt 7.
- **Erster Schritt: Lochraster‑Prototyp ✅** mit DevKitC + GPIO‑Extension‑Board.
  Erst Funktions‑Proof aller Subsysteme einzeln und kombiniert, dann
  PCB‑Layout für Halloween 2026.
- **Bootloader/Flash:** S3 nutzt USB‑Serial/JTAG sowie USB‑OTG nativ – kein
  externer USB‑UART‑Chip nötig.
- **Detaildokument:** [`12-refactor-station-v2.md`](12-refactor-station-v2.md)
  enthält den kompletten Plan inkl. GPIO‑Belegung, Audio‑Kette,
  Software‑Konventionen, Mechanik, Iterationsplan.
- **Priorität:** Hoch – Voraussetzung für alle weiteren Refactors.

### 19. Audio‑Pfad ✅ Entschieden (I²S → PCM5102A → TPA3116D2)

- **Entscheidung 2026‑05‑07:** Audio läuft über den I²S‑Bus des S3 in einen
  externen DAC (PCM5102A), von dort als Line‑Level in einen TPA3116D2
  Class‑D‑Verstärker, dann auf den Visaton FR 10/4. WAV 22 kHz Mono 16‑Bit
  aus LittleFS gestreamt – **kein DFPlayer**, weil dessen DAC der
  Bottleneck wäre. Details in
  [`12-refactor-station-v2.md`](12-refactor-station-v2.md), Abschnitt 2.3.
- **Historischer Kontext (DFPlayer‑Diskussion):**
  Bei der ursprünglichen C3‑Plattform‑Variante war der DFPlayer Mini eine
  ernsthafte Alternative wegen Software‑Einfachheit. Mit dem S3‑Wechsel und
  ausreichend PSRAM/Flash ist der I²S‑Pfad eindeutig die bessere Wahl
  (saubererer DAC, höhere Lautstärke‑Headroom, beliebige Software‑EQ).
- **Kandidat (historisch):** *DFPlayer Mini* – spielt MP3/WAV von MicroSD via UART‑Befehl
  (einfaches 2‑Byte‑Protokoll).
  - Vorteil: kein I²S‑Timing‑Problem, kein RAM‑Puffer‑Stress auf dem ESP.
  - **Bekanntes Problem:** Im Markt kursieren sehr viele Billigkopien
    (erkennbar an fehlender oder anderer Chip‑Beschriftung). Diese haben
    oft deutliche Audio‑Latenz (bis ~500 ms nach Befehl) und gelegentliche
    Aussetzer. Originale Chips (YX5200) sind deutlich zuverlässiger.
  - Alternative Chips: *MH2024K‑24SS* (häufige Kopie, oft ok),
    *GD3200B* (problematisch). Beim Kauf: explizit nach
    YX5200‑Chip fragen oder aus verlässlicher Quelle bestellen.
- **Weitere Optionen (falls DFPlayer zu unzuverlässig):**
  - **MP3‑TF‑16P** (baugleich, aber manchmal bessere Chipqualität)
  - **Catalex Serial MP3** (weniger verbreitet)
  - **YX5300** (älterer DFPlayer‑Vorgänger, oft besser)
  - **I²S direkt am C3** mit kleinem Verstärker‑Breakout (MAX98357 oder
    NS4168) – wäre wie bisher, aber nun mit weniger RAM‑Druck weil kein
    paralleler WLAN‑Stack (C3 hat bessere Trennung als S3 bei I²S + WiFi?)
    – **Verifikationspunkt.**
- **Was zu klären ist:**
  - DFPlayer‑Chip‑Variante beim nächsten Kauf identifizieren und Latenz
    messen.
  - Reicht der interne Verstärker (3 W an 4 Ω) für den Outdoor‑Einsatz?
  - Sound‑Dateinamen‑Schema (aktuell `0001.mp3` … für DFPlayer vs.
    `0_*.mp3` auf NAS) abstimmen.

### 20. Kommunikations‑Refactor: ESP‑NOW statt HTTP ✅ gesetzt (2026‑05‑07)

- **Entscheidung:** Beim Station‑Refactor (Punkt 18) wird direkt auf
  **ESP‑NOW** gewechselt. Kein Zwischenschritt über HTTP, kein
  WiFiManager‑Captive‑Portal, kein Webserver auf der Station.
- **Motivation:** HTTP setzt einen funktionierenden WLAN‑Router voraus.
  Im Vorgarten ist das zwar gegeben, aber ein routerloser Peer‑to‑Peer‑Bus
  wäre robuster und würde die Latenz reduzieren.
- **ESP‑NOW Grundprinzip:** Proprietäres 802.11‑basiertes L2‑Protokoll
  von Espressif; Reichweite ~200 m frei, Latenz ~1 ms,
  kein WLAN‑Router nötig, Broadcast und Unicast möglich.
- **Geplanter Datenfluss mit ESP‑NOW:**
  1. Station sendet IR‑Code mit ihrer **Stations‑ID** (1..N) per IR.
  2. Target empfängt IR‑Code, erkennt Treffer, baut ESP‑NOW‑Paket:
     `{station_id: X, sound_id: Y, target_id: Z}`.
  3. Station mit passender ID empfängt Paket und spielt Sound Y ab.
  4. (Optional) Andere Stationen ignorieren das Paket (ID‑Filter).
- **Skalierung:** 8–15 Targets und 3–4 Stationen problemlos im
  ESP‑NOW‑Adressraum (max. 20 Peers im Unicast‑Modus; Broadcast
  unbegrenzt). Bei Broadcast‑Ansatz braucht jede Station nur auf
  ihre eigene `station_id` im Paket reagieren.
- **Konsequenz für bestehenden Code:**
  - `hitAction()` im Target: `WiFiClient` + `HTTPClient` ersetzen durch
    `esp_now_send()`.
  - `InfinitagWand.ino`: `WebServer`‑Instanz und `/trigger_effect`‑Route
    entfernen; stattdessen ESP‑NOW‑Receive‑Callback.
  - WiFiManager‑Captive‑Portal für IP‑Konfiguration entfällt; stattdessen
    nur noch SSID/PW für eventuelle OTA‑Updates nötig (oder SSID entfällt
    ganz, wenn kein OTA).
- **Offene Fragen:**
  - ESP32‑C3 unterstützt ESP‑NOW vollständig (bestätigt in Espressif‑Docs).
    Funktioniert ESP‑NOW + gleichzeitig WiFi‑STA (für OTA)? → Ja, möglich
    im gemischten Modus, aber Kanal muss übereinstimmen. **Verifikationspunkt.**
  - Wie sollen die MAC‑Adressen der Stationen in die Targets kommen?
    Optionen: a) Broadcast + ID‑Filter (einfach, kein Pairing),
    b) Einmalig per Captive‑Portal MAC eingeben,
    c) Auto‑Discovery über Beacon‑Broadcast beim Bootvorgang.
  - Punkt 2 (hartcodierte HTTP‑URL) wäre mit ESP‑NOW automatisch erledigt.

### 22. Konfiguration ohne externes Gerät – ✅ Gelöst (OLED + 4 Tasten + Service‑Klappe)

- **Entscheidung 2026‑05‑07:** Die Konfiguration läuft über ein **OLED 1,3" SH1106
  mit integriertem 4‑Tasten‑Modul** (K1–K4). Hinter einer **Service‑Klappe**
  im Gehäuse, mit Schaumdichtung gegen Spritzwasser und 8‑pol JST‑XH‑Stecker
  zum Hauptboard. Stations‑ID, Lautstärke, Sound‑Test und Diagnose alle
  über das Menü. ID wird im NVS gespeichert, übersteht Reboot.
  Live‑Status nach außen über eine zusätzliche **NeoPixel‑LED** auf der
  Außenwand (Daisy‑Chain mit Stab‑LEDs, kein extra GPIO).
  Details in [`12-refactor-station-v2.md`](12-refactor-station-v2.md),
  Abschnitt 2.5 + 2.6.
- **Historischer Kontext (DIP‑Diskussion):**
  Ursprünglich war ein 2‑Bit‑DIP‑Schalter (4 IDs) angedacht. Bei der
  C3‑Plattform‑Variante mit knappem Pin‑Budget war das die naheliegende
  Lösung. Mit dem S3‑Wechsel und der OLED‑Option (Pin‑Budget kein Engpass
  mehr) wurde die DIP‑Variante zugunsten beliebig vieler IDs + Lautstärke‑
  und Diagnose‑Menü verworfen.

#### Historische Anforderungs‑Notizen (vor Lösung)

- **Anforderung:** Unerfahrene Helfer sollen das System an Halloween ohne
  Smartphone, Laptop oder Captive‑Portal aufstellen können. Gleichzeitig soll
  die Konfiguration im Betrieb gegen versehentliche (oder mutwillige)
  Veränderung durch Kinder geschützt sein.

#### Welche Parameter müssen überhaupt konfiguriert werden?

| Parameter | Heute | Mit ESP‑NOW | Pflicht? |
|---|---|---|---|
| WLAN‑SSID / Passwort | Captive‑Portal | entfällt (kein Router nötig) | nein |
| IP‑Konfiguration | Captive‑Portal | entfällt | nein |
| **Stations‑ID (1–4)** | implizit via IP | **explizit nötig** | **ja** |
| Lautstärke | nicht vorhanden | nice‑to‑have | nein |
| Sound‑Test / Diagnose | nicht vorhanden | nice‑to‑have | nein |

→ **Bei konsequentem ESP‑NOW‑Umbau ist die Stations‑ID der einzige
Pflichtparameter an der Station.**

#### Option A – DIP‑Schalter (empfohlen wenn nur ID nötig)

- 2‑poliger DIP‑Schalter (2 Bit = 4 Stationen) direkt auf dem PCB,
  zugänglich über eine kleine Aussparung im Gehäuse.
- Wert wird beim Booten einmal gelesen – Änderung nur bei
  ausgeschaltetem Gerät möglich → Missbrauchsschutz im Betrieb ist
  inhärent.
- **Vorteile:** extrem einfach, kein Software‑Aufwand, keine
  Fehlerquelle, billig (~0,20 €).
- **Nachteil:** keine Rückmeldung ob der ESP die ID korrekt gelesen hat.
  Empfehlung: Status‑LED oder kurze NeoPixel‑Blinksquenz beim Boot
  zeigt ID an (1× blinken = ID 1, 2× = ID 2, …).

#### Option B – OLED + Tasten (wenn mehr Konfiguration gewünscht)

- **Modul:** 0,96″ OLED IIC 128 × 64 (SSD1315) mit 4 × 4‑Tastenplatine.
  Anschluss über I²C (SDA/SCL) + 4 digitale Eingänge für die
  Navigationstasten (Auf/Ab/OK/Zurück reichen aus, 4 × 4 wäre
  Luxus).
- **Mögliches Menü:**
  ```
  [INFINITAG STATION]
  > ID einstellen    ← 1 / 2 / 3 / 4
    Lautstärke       ← 1‑10
    Sound‑Test       ← spielt Sound X ab
    Status           ← zeigt MAC, RSSI, letzte Treffer‑ID
  ```
- **Missbrauchsschutz:** Menü nur erreichbar nach langem Druck (3 s)
  auf eine versenkte oder schwer zugängliche Taste. Im Normalbetrieb
  zeigt das OLED nur eine Statuszeile (ID, Batterie/Spannung,
  letzter Treffer) – kein Menü ohne Freischalte‑Geste.
- **Vorteile:** visuelle Rückmeldung, erweiterbar, Debug‑Infos
  direkt lesbar.
- **Nachteile:** mehr Code, mehr GPIOs, mehr Bauteile, das Gehäuse
  braucht ein Display‑Fenster.

#### Empfehlung (Stand 2026‑05‑07)

**Schritt 1:** DIP‑Schalter + Boot‑Blink für die ID – reicht für den
nächsten Halloween‑Aufbau und ist sofort umsetzbar.
**Schritt 2:** OLED nur nachrüsten, wenn sich im Betrieb zeigt, dass
mehr Konfiguration gebraucht wird (Lautstärke‑Anpassung vor Ort,
Diagnose‑Anzeige).

- **Was zu klären ist:**
  - Passt ein 2‑poliger DIP‑Schalter auf das neue C3‑Super‑Mini‑PCB?
  - Falls OLED: Welche GPIOs am C3 bleiben für I²C + 4 Tasten frei,
    nachdem IR‑LED, Laser, NeoPixel und DFPlayer belegt sind?
  - Gehäuse‑Design: Aussparung für DIP oder Fenster für OLED einplanen.

### 21. Skalierung: Multi‑Station‑Betrieb (3–4 Stationen, 8–15 Targets)

- **Aktueller Stand (bestätigt 2026‑05‑07):** 1 Station versorgt
  **genau 1 Stab**. Die zweite GX16‑Buchse am Gehäuse ist die
  Stromzuführung, kein zweiter Stab‑Anschluss.
- **Geplante Erweiterung:** Mehrere Kinder spielen gleichzeitig.
  Statt eine Station mit mehreren Stäben zu erweitern, werden
  **mehrere identische Stationen aufgestellt** – jede mit eigenem
  Stab, eigener Sound‑Pipeline und eigener Stations‑ID.
- **ID‑Konzept:**
  - Jede Station bekommt eine eindeutige **Stations‑ID** (1, 2, 3, 4).
  - Diese ID steckt im IR‑Code‑Feld `playerId` (bereits im
    `Infinitag_Core`‑Protokoll vorgesehen, 8 Bit).
  - Jedes Target ist **per Captive‑Portal konfiguriert**, welche Sound‑ID
    es zurückmeldet und wie lange das Prop läuft.
  - Beim Treffer sendet das Target die Stations‑ID (aus dem empfangenen
    IR‑Code) + die konfigurierte Sound‑ID zurück.
  - Die Station mit passender ID spielt den Sound; alle anderen ignorieren
    das Paket.
- **Was zu klären ist:**
  - Darf ein Target von mehreren Stationen gleichzeitig getroffen werden?
    Wenn ja: welche Reaktion hat Vorrang?
  - Sollen Targets eine Abklingzeit haben (nach Treffer X Sekunden nicht
    re‑triggerbar), um Mehrfach‑Sounds zu vermeiden?
  - Sound‑Bibliothek pro Station: jede Station hat eigene MP3s auf ihrer
    MicroSD/Flash. Gleiche Sound‑ID = gleicher Effekt auf allen Stationen,
    oder individuelle Belegung?

## Wand (Zauberstab) – Mechanik & Fertigung

### 23. Kabel-Verbindung Wand ↔ Station: GX16 ersetzen

- **Aktueller Stand:** 7-poliges Kabel mit **GX16-Luftfahrtstecker** (Schraubanschluss,
  eigenhändig gecrimpt). Sitzt fest, sieht professionell aus.
- **Probleme:**
  - Kabel knickt direkt am Stecker ab (keine Zugentlastung im Stecker selbst) –
    häufigste Schadstelle.
  - GX16 ist aufwändig zu konfektionieren (Lötarbeit an 7 dünnen Adern im
    Metallgehäuse, Zugentlastungsring).
  - Im Notfall (Feiertag, draußen) kaum schnell zu ersetzen.
- **Anforderungen an die neue Lösung:**
  - Schnell wechselbar ohne Werkzeug.
  - Standardkabel aus dem Handel (kein Custom-Crimpen).
  - Zugentlastung und Knickschutz am Stecker integriert oder 3D-druckbar.
  - Schutz vor versehentlichem Abziehen im Betrieb (Kinder).
  - 7 Adern oder mehr.
- **Optionen:**

  | Option | Stecker | Kabel | Locking? | Aufwand |
  |---|---|---|---|---|
  | A | **GX16-7 beibehalten** + 3D-Druck-Knickschutzmanschette + Überwurf-Sicherung | Silikonkabel (flexibel) | Überwurf 3D-Druck | mittel |
  | B | **XLR 7-pin** (z. B. Neutrik NC7FXX) | Mikrofonkabel | Bajonett-Sicherung, industr. Standard | hoch (teuer) |
  | C | **RJ45 / Ethernet-Kabel** | Cat6-Patchkabel | Clip (schwach) + 3D-Druck-Haube | niedrig (billig, sofort verfügbar) |
  | D | **USB-A / USB-B** | Standard-USB-Kabel | keine | sehr niedrig – aber nur 4 Adern, reicht nicht |
  | E | **JST-XH 8-pol** (2,54 mm) | eigene Konfektionierung | keine, aber Haube 3D-drucken | niedrig |

- **Empfehlung zur Diskussion:**
  Option A (GX16 + Silikonkabel + 3D-Druck-Schutz) ändert das wenigste am
  bestehenden System und löst die Knick-/Wechsel-Problematik.
  Option C (RJ45) wäre radikal einfach (Kabel überall kaufbar, Clip zieht
  nicht von selbst raus) – erfordert aber 8→7-Pin-Adapter-Belegung und
  ein 3D-gedrucktes Sicherungsklipp-Cover.
- **Was zu klären ist:**
  - Welches Kabel liegt im Betrieb: direkt auf dem Boden oder an einem
    Mast/Arm? (Biegebelastung vs. Zugbelastung)
  - Soll das Kabel austauschbar OHNE Lötkolben sein?

- **Entschieden 2026‑05‑08 (Station V2 Refactor):** **Option F – USB‑C
  3.x mit 6 Adern**, kombiniert mit Wand‑Modifikation (Trigger active‑LOW,
  3,3‑V‑Ader entfällt). USB‑C ist im Handel überall verfügbar, als
  Spiralkabel praktisch (Acecene‑artig, ~5 €/Stück), Knickschutz im Stecker
  bereits integriert, austauschbar ohne Lötkolben. Verwechslungsschutz
  (Handy‑Ladegerät → ESP‑GPIO‑Killer) durch TVS‑Diode‑Array auf dem
  Adapter‑PCB + mechanische Markierung an der Buchse. Details:
  [`12-refactor-station-v2.md`](12-refactor-station-v2.md), Abschnitt 8.11.

### 24. IR-LED-Linse: Toleranz- und Ausrichtungsproblem

- **Aktueller Stand:** IR-LED (TSAL6200) sitzt auf dem Wand-PCB V2.1
  zusammen mit Taster und SK6812-LEDs. Eine 3D-gedruckte Scheibe hält die
  IR-LED mit definiertem Abstand zur Linse (Linse ist ebenfalls gedruckt
  oder eingeklebt).
- **Probleme:**
  - 3D-Druck-Toleranzen (±0,2–0,4 mm) verschieben den LED-Mittelpunkt
    relativ zur optischen Achse der Linse → Keule dezentriert, Reichweite
    und Winkel variieren von Wand zu Wand.
  - Schlechte Wiederholbarkeit: jeder Druck erzeugt leicht andere Geometrie.
  - Nachjustieren nicht möglich ohne Demontage.
- **Lösungsoptionen:**

  | Option | Prinzip | Aufwand | Repeatability |
  |---|---|---|---|
  | A | **Separate IR-LED-Platine** (klein, an Drähten) mit Zentrierpin im Linsenhalter | PCB-Order | hoch |
  | B | **Selbstzentrierender Press-fit-Linsenhalter** (Innendurchmesser = LED-Außenmaß) | nur 3D-Druck | mittel |
  | C | **Fertige Kollimator-Linse** (z. B. 5 mm Linse mit Halter wie bei Flashlight-Optiken) – selbst zentrierend durch mechanischen Sitz | Kauf (~1 €) | hoch |
  | D | **IR-LED auf flexibler Stecker-Platine** direkt am Linsenhalter montiert, nicht auf der Haupt-PCB | PCB-Order | hoch |

- **Empfehlung:** Option C oder D. Die LED vom Haupt-PCB trennen und direkt
  am optischen Element befestigen eliminiert die Toleranzkette komplett.
  Dann ist der Linsenkörper das Referenzbauteil, nicht das PCB.
- **Entschieden 2026‑05‑08 (Wand V3 Refactor):** Lösung im
  [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md), Abschnitte 3.1–3.3.
  Optik‑Block als gemeinsamer Druckteil für IR‑LED + OM3 + Laser =
  alleiniges Referenzbauteil. IR‑LED auf Tochter‑PCB im Optik‑Block,
  dreifach redundante Zentrierung (Reibsitz Ø 4,9 mm + Konus +
  LED‑Beine als Selbstführung). LED‑Position 28 mm hinter Linse
  (2 mm Defokus → ±2° Beam, 17 cm Cone bei 5 m).
- **Linsen-Katalog:** Vollständige Übersicht aller AstroMedia‑Linsen mit
  IR‑Eignungsbewertung in
  [`10-bill-of-materials.md`](10-bill-of-materials.md), Abschnitt „Optik".
  Kandidaten für den nächsten Versuch: **OM11/OM12** (Ø 25 mm, bikonvex,
  mehr Lichtausbeute) oder **OM2** (f = 15 mm, sehr kompakt).
- **Update 2026‑05‑09 – Linsen‑Auswahl entschieden:** OM3 vs. OM2/OM4
  systematisch durchgerechnet in
  [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md) Abschnitt 3.1.1.
  Tobias hat OM4 im Bestand, aber die wird nicht eingesetzt (~35 %
  Lichtverlust, weil Apertur zu klein für TSAL6100 ±10° bei f = 49 mm).
  **10 Stück OM3 bei AstroMedia bestellt (Best.‑Nr. 304.OM3)** – 4 Stäbe
  + 6 Reserve für Phase‑1‑Defokus‑Tests.
- **Was zu klären ist:**
  - Welchen Außendurchmesser hat die TSAL6200 (5 mm-Gehäuse)?
    → Standardgehäuse 5 mm, gut für Press-fit-Halter.
  - Wird eine Kollimationslinse wirklich gebraucht, oder reicht der
    natürliche ±17°-Abstrahlwinkel der TSAL6200?
  - Wie tief darf der Linsenhalter im Gehäuse maximal sein?
    (Bestimmt ob OM11/OM12 mit f ≈ 42–45 mm reinpassen oder ob OM3
    mit f = 30 mm das Limit ist.)

### 25. Laserpointer: Ausrichtung und Befestigung

- **Aktueller Stand:** KY-008-artiges Lasermodul (5 mm Rotes Dot-Laser,
  5 V, 2-poliger Stecker) ist im Zauberstab-Gehäuse verbaut.
  Wird beim Schuss für `laserDelay = 1000 ms` eingeschaltet.
- **Probleme:**
  - Lasermodul-Kegel ist selbst nicht kollimiert/justierbar – Streupunkt
    je nach Exemplar leicht unterschiedlich.
  - Einbau mit Heißkleber: Modul kann sich beim Aushärten verdrehen,
    Heißkleber-Hitze beschädigt manchmal die Plastiklinse oder den
    Treiber-IC des Moduls.
  - IR-Keule und Laserstrahl sind nicht koaxial → Kind zielt mit Laser,
    trifft aber IR-seitig daneben (oder umgekehrt).
- **Lösungsoptionen:**

  | Option | Prinzip | Aufwand |
  |---|---|---|
  | A | **Gemeinsamer optischer Block** für IR-LED + Laser: beide in einem 3D-gedruckten Halter mit definiertem Versatz, Halter einmalig ausrichten und verkleben | 3D-Druck |
  | B | **Justierbarer Laserhalter** mit 2 Einstellschrauben (Pitch + Yaw), wie ein Zielfernrohr-Ring | 3D-Druck + M2-Schrauben |
  | C | **Laser weglassen**, stattdessen hellen weißen/blauen NeoPixel-Blitz als „Schuss-Feedback" – Kinder orientieren sich am Lichtblitz der Stab-LEDs | Software |
  | D | **Kaltkleber / UV-Kleber** statt Heißkleber für feste, hitzearme Fixierung | Kauf |

- **Empfehlung:**
  - Kurzfristig: Option D (UV-Kleber) für die Montage, damit Lasermodule
    nicht mehr durch Hitze sterben.
  - Mittelfristig: Option B (Justierschrauben) für verlässliche Ausrichtung.
    2 × M2-Madenschrauben + Federblech reichen für ±5° Justage.
- **Entschieden 2026‑05‑08 (Wand V3 Refactor):** Lösung im
  [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md), Abschnitt 3.4.
  Lasermodul KY‑008 sitzt im selben Optik‑Block wie IR‑LED + Linse,
  3 × M2‑Madenschrauben um 120° versetzt erlauben Feinjustage. Versatz
  zur IR‑Achse konstruktiv 12 mm vertikal (bekannt und reproduzierbar,
  Spieler kompensiert mental). Endfixierung mit Loctite 243.
  - Diskussionswürdig: Option C – bei kurzer Schießbuden-Distanz (2–5 m)
    kann ein heller NeoPixel-Blitz ausreichen und eliminiert ein weiteres
    Fehler-anfälliges Bauteil. Laser als Feature vs. Laser als Problem?
- **Was zu klären ist:**
  - Wie groß ist der Versatz zwischen IR-Keule-Achse und Laserstrahl-Achse
    aktuell in der Praxis? (Grob abschätzen: bei 3 m Abstand um wie viel cm
    daneben?)
  - Akzeptieren Kinder einen „Lichtblitz statt Punkt"?

### 27. Station: Stromversorgungs‑Anschluss standardisieren + gegen Abziehen sichern

- **Aktueller Stand:** Stromeingang der Station über einen **großen GX16‑Stecker**
  mit Custom‑Kabel vom Netzteil. Gleiche Probleme wie beim Wand‑Kabel:
  aufwändige Konfektionierung, kein schneller Tausch möglich.
- **Zusatzproblem:** Im Outdoor‑Betrieb könnten neugierige Kinder (oder
  unabsichtliche Passanten) das Stromkabel ziehen → System fällt aus.

#### Spannungsebene: 5 V direkt oder 12 V + Konverter?

| Option | Vorteil | Nachteil |
|---|---|---|
| **5 V direkt** | kein Wandler nötig, System‑PCB bleibt wie es ist | 5 V Netzteile für Außen eher selten; Spannungsabfall bei längerem Kabel (>2 m) relevant |
| **12 V + Schaltregler** | 12 V Outdoor‑Netzteile (z. B. LED‑Treiber, IP67) überall günstig; weniger Spannungsabfall auf langen Kabeln; ein kleiner LM2596‑/MP2307‑Modul (~1 €) wandelt auf 5 V | eine Komponente mehr im Gehäuse |

**Empfehlung:** 12 V Eingang + eingebetteter Step‑Down‑Regler (z. B. LM2596
Einstellbar, auf 5,1 V eingestellt, 3 A). Vorteil im Außenbereich: Kabel
dünner, Netzteil‑Auswahl größer, robuster gegen Länge. Den Regler direkt
auf dem neuen C3‑Träger‑PCB oder als Breakout in der Box platzieren.

#### Anschluss‑Konzept: Standard‑Buchse + Sicherungsblende

- **Buchse:** Standard **DC‑Hohlbuchse 5,5/2,5 mm** (12 V Eingang) –
  überall erhältlich, kompatibel mit gängigen IP67‑Netzteilen.
  Einbau in die Gehäuse‑Wand über eine 12 mm Bohrung.
- **Sicherung gegen Abziehen:**
  3D‑gedruckte **Kabelblende** (L‑förmige oder U‑förmige Abdeckung),
  die über das eingesteckte Kabel geschoben wird und mit einer
  **M3‑Schraube** am Gehäuse fixiert wird. Das Kabel kann nur entfernt
  werden, wenn die Schraube gelöst wird – für Kinder im Betrieb nicht
  erreichbar wenn die Schraube auf der Rückseite/Unterseite sitzt.
  Gleiches Prinzip wie bei Punkt 23 (Wand‑Kabel).

  ```
  Gehäuse-Wand (Querschnitt):
  ────────────────────────
   Blende ──┐
    (3D)    │  ┌── Kabel
            ▼  ▼
  ══════[Buchse]══════
            │
         M3-Schraube
  ────────────────────────
  ```

- **Was zu klären ist:**
  - Passt eine 5,5/2,5 mm Hohlbuchse ins bestehende orange Gehäuse,
    oder muss eine neue Bohrung gesetzt werden?
  - Ist 12 V → 5 V Step‑Down direkt in der Box sinnvoll, oder bleibt
    man bei 5 V und kauft ein passendes IP‑Netzteil?
  - Sollen alle Stationen das gleiche Netzteil nutzen (gleiche Buchse,
    gleiche Spannung) → Austauschbarkeit im Notfall.

### 26. Zauberstab: Gesamtmontage zu aufwändig, viele Fehlerquellen

- **Beschreibung:** Jede Wand erfordert derzeit viele Einzelschritte:
  PCB bestücken/löten, Kabel konfektionieren (GX16), Linse einstellen,
  Laser ausrichten, alles verkleben und ins Gehäuse schrauben.
  Mehrere Versuche sind pro Einheit üblich; Ausschuss durch
  Heißkleber-Schäden und 3D-Druck-Nacharbeit erhöht den Aufwand weiter.
- **Ziel:** Wand reproduzierbar in <30 min bauen können, auch von einer
  zweiten Person ohne Vorerfahrung.
- **Strukturelle Maßnahmen:**

  1. **Optischen Subassembly** als eigenständige Einheit designen:
     IR-LED + Lasermodul in einem gemeinsamen Halter (Punkt 24 + 25),
     der als Ganzes eingeschoben und gesichert wird. Wenn ein Laser
     stirbt, wird nur der Subassembly getauscht.
  2. **Kabel-Standardisierung** (Punkt 23) – konfektionierte Kabel
     aus dem Handel eliminieren den größten Zeitfresser.
  3. **Montageanleitung** als einfache Schritt-für-Schritt-Karte
     (laminiertes A5-Blatt) für Helfer – in
     [`09-halloween-setup.md`](09-halloween-setup.md) einbetten.
  4. **Klebestrategie:** Heißkleber nur für temporäre Fixierung während
     Ausrichtung; endgültige Fixierung mit UV-Kleber oder M2-Schrauben.
- **Was zu klären ist:**
  - Wie viele Wände werden gleichzeitig benötigt? (3–4 laut Punkt 21)
  - Soll das Wand-PCB V2.1 beibehalten werden oder im Zuge des
    Station-Refactors (Punkt 18) ebenfalls überarbeitet werden?
- **Entschieden 2026‑05‑08 (Wand V3 Refactor):** Komplettes Wand‑Refactor
  parallel zur Station V2. Optik‑Block als modularer Subassembly + neues
  Halbschalen‑Gehäuse (in 2 Modulen) + neues Wand‑PCB V3 mit USB‑C +
  active‑LOW‑Trigger. Ziel: < 30 min Montage pro Stab, vier Stäbe für
  Halloween 2026. Vollständiges Konzept in
  [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md).

## Doku‑To‑dos

- 3D‑Druck‑Parameter (Filament, Schichthöhe, Infill) für Targets ergänzen.
- Foto/Skizze des realen Aufbaus im Vorgarten in
  [`09-halloween-setup.md`](09-halloween-setup.md) einbinden.
- Eigene MD für Halloween‑Props (Punkt 16) – sobald wir das angehen.
- Sobald Punkt 1 (Sound‑ESP‑Klarheit) bestätigt: ein eigenes
  „Software‑Update‑Log” einführen, das pro Sketch‑Version den Stand
  festhält.

### 28. TPA3110 XH‑A232 (V928): STBY‑Pin nicht herausgeführt

- **Festgestellt:** 2026‑05‑12 beim Lochraster‑Prototyp‑Aufbau.
- **Befund:** Das vorliegende TPA3110‑Modul (XH‑A232, Aufdruck V928)
  führt den STBY‑Pin des TPA3110D2 nicht als Lötpad heraus. Der Amp
  ist intern dauerhaft aktiv (STBY intern auf HIGH gezogen).
- **Konsequenz:** Mute‑Strategie B (GPIO11 → STBY) aus
  [`12-refactor-station-v2.md`](12-refactor-station-v2.md), Abschnitt 5.5,
  lässt sich mit diesem Modul **nicht direkt** umsetzen.
- **Alternativen:**
  1. **Plan A (bevorzugt):** Mute über PCM5102A XSMT‑Pin (GPIO12). H3L‑Lötbrücke
     auf dem GY‑PCM5102‑Modul öffnen (falls noch gebrückt), XSMT‑Pin mit
     Draht auf GPIO12 legen. GPIO12 ist im Plan v7 bereits als Reserve/XSMT
     vorgesehen. Soft‑Fade‑In/Out des DAC ersetzt STBY‑Steuerung.
  2. **Plan B:** Auf PCB‑Ebene STBY‑Pin des TPA3110‑ICs direkt abgreifen
     (SMD‑Lötpunkt, aufwändig).
  3. **Plan C:** Anderes Verstärker‑Modul mit zugänglichem STBY wählen.
- **Für den Prototyp‑Test:** STBY‑Steuerung entfällt; Amp läuft dauerhaft.
  Stille‑Samples via `i2s_zero_dma_buffer()` verhindern Hiss zwischen Sounds.
  Soft‑Mute über XSMT (Plan A) als nächster Schritt nach erfolgreichem Audio‑Test.

### 29. Box B: Bassreflex‑Port für FR 10/4 nachrechnen ✅ erledigt (2026‑05‑13)

- **Ergebnis:** Port **Ø 20 mm × 56 mm** physikalische Länge → f_b = 80 Hz
- **Berechnung:** fs = 90 Hz, Vb = 2,06 l, Leff = 71 mm, Endkorrektur 14,6 mm
  (0,85×r innen + 0,61×r außen bei r = 10 mm) → L_phys = 56 mm
- **Hinweis:** Original‑Wert 30 mm hätte f_b ≈ 101 Hz ergeben (über fs → Bassbuckel
  statt Verlängerung). 56 mm gibt f_b = 80 Hz mit flacherem Tiefbassgang.
- **Quelle:** 12-refactor-station-v2.md Abschnitt 8.3

### 30. Box B: Korb‑Außen‑Ø und Stufenbohrung am FR 10/4 mit Schieblehre verifizieren

- **Festgestellt:** 2026‑05‑13. Etikettmaß: Korb‑Außen‑Ø = 105 mm (breiteste
  Stelle im Seitenriss), Stufentiefe = 3 mm. Ø 115 mm ist der Lochkreis‑Ø
  (Schrauben‑Positionen), nicht der Korb‑Außen‑Ø.
- **Bestätigt aus Etikettmaß + Schieblehre (2026‑05‑13):**
  - Membran‑Ø / Durchgangsloch: **Ø 100 mm** ✅ (innerer Kreis Frontansicht)
  - Korb‑Außen‑Ø / Stufenbohrung: **Ø 105 mm** ✅ (Gesamthöhe Seitenansicht)
  - Flanschhöhe / Stufentiefe: **3 mm** ✅
- **Noch zu messen:**
  - Quadrat‑Flansch Seitenmaß (Schätzung ~90 mm aus Diagonale Ø 127,5 mm) → bestimmt Block‑Außenmaß
- **Innenhöhe:** 110 mm Innenraum bei Ø 105 mm Korb = 2,5 mm Luft je Seite beim Einführen ✓ → **✅ erledigt**
- **Quelle:** 12-refactor-station-v2.md Abschnitt 8.3

### 31. Wand V3 – Trigger-Hardware Gateron Green: Probedrucke + Haptik-Test

- **Beschluss 2026‑05‑17:** Wand‑V3‑Trigger ist Gateron Green
  (Cherry MX Green Klon, 5‑Pin PCB‑Mount, direkt aufgelötet),
  Edelstein‑förmiger Keycap aus transparentem PETG auf dem MX‑Plus‑Stem,
  4× SK6812 5050 als Halo auf der Haupt‑PCB. Details:
  [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md) §4.1 + §4.4.
- **Bestellung ✅ erledigt 2026‑05‑17:** 36er‑Pack Gateron Green via
  DRAOZA / Amazon, 10,99 € (Variante „Grün" – das Listing ist generisch
  „Cremiger Gelber" beschriftet, aber die Grün‑Variante liefert
  tatsächlich Gateron Green mit 80 ± 15 gf Aktivierung, 2,3 mm Pretravel,
  4,0 mm Total Travel, Click‑Jacket, 50 M Cycles).
- **Cherry‑MX‑Blue‑Fallback entfällt:** Ursprünglich war ein 4er‑Set
  Cherry MX Blue (50 cN Aktivierung) als Kinder‑Variante eingeplant.
  Tobias‑Entscheidung 2026‑05‑17: erst mit Gateron Green probetragen,
  Blue/Yellow nur nachkaufen, falls 80 gf von Kindern als zu hart
  empfunden werden.
- **Was nach Lieferung zu verifizieren ist:**
  1. **QC‑Check beim Auspacken** – 36 Switches einzeln testen, die
     besten 8 (= 4 Stäbe + 4 Reserve) auswählen. Bei Bulk‑Importen
     gelegentlich 1–3 schwergängige oder verklemmende Exemplare im Pack.
  2. **Haptik‑Test in der Hand** – Switch in einem Probedruck‑
     Halbschalen‑Stück mit eingestecktem Edelstein‑Keycap halten, von
     Kindern und Erwachsenen probedrücken lassen. Entscheidung: bleiben
     wir bei 80 gf Green oder beschaffen wir nachträglich Blue/Yellow
     für eine Kinder‑Variante?
  3. **Probedruck Edelstein-Keycap** – hexagonal Ø 14 × 8 mm, Plus‑Stem
     4 × 4 × 4 mm tief mit +0,1 mm Spiel, transparentes PETG. Sitz auf
     einem losen Gateron‑Stem prüfen, Lichtdurchlässigkeit gegen
     SK6812 testen.
  4. **Probe‑PCB mit MX‑Footprint + 4 NeoPixel + JST** bei JLCPCB
     bestellen (~5 €/5 Stück), Halo‑Look gegen Edelstein validieren,
     bevor das finale Haupt‑PCB layoutet wird.
- **Status:** Konzept ✅, Bestellung ✅, Probedrucke und Haptik‑Test
  offen (warten auf Switch‑Lieferung).

### 33. Wand V3 – Art-Nouveau-Reliefs auf Halbschalen-Außenflächen

- **Beschluss 2026‑05‑17:** Halbschalen bekommen Art‑Nouveau‑
  Ranken‑Reliefs an drei Slot‑Positionen (Spitze‑Schaft‑Übergang,
  Trigger‑Bereich, Knauf‑Übergang), Erhebung 1,0 mm, mono‑material
  in V1 (multi‑color erst V2). Auslöser war die KI‑Render‑Iteration
  am 2026‑05‑17, bei der das Image‑Modell solche Reliefs
  selbstständig vorgeschlagen hat und der Look von „Tech‑Stab" zu
  „Magic Wand" gehoben hat. Vollständige Spezifikation:
  [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md) §5.8.
- **Tobias‑Hinweis 2026‑05‑17:** „Weiß noch nicht, wie man diese in
  Fusion 360 umsetzen kann" – CAD‑Workflow muss noch erarbeitet werden.
- **Empfohlener Workflow (siehe §5.8):**
  1. **SVG‑Beschaffung:** Art‑Nouveau‑Vektor‑Ornament aus Freepik /
     Noun Project / Vecteezy ziehen (Suchbegriffe „art nouveau border",
     „filigree vector", „art nouveau scroll"), oder Konturen aus dem
     KI‑Render‑Bild in Inkscape nachvektorisieren, oder Claude/ChatGPT
     um SVG‑Code für „art nouveau scroll vine SVG, 80 × 30 mm,
     suitable for embossing on 3D-printed wand grip" bitten.
  2. **Fusion‑Import:** `Insert > Insert SVG` auf eine Hilfs‑Ebene
     tangential zur Halbschale.
  3. **Projektion:** `Sketch > Project / Include > Project to Surface`,
     Type „Closest Point", Sketch auf Halbschalen‑Außenfläche.
  4. **Emboss:** `Create > Emboss`, Profile = projizierter Sketch,
     Face = Halbschalen‑Außenfläche, Effect = Raised, Depth = 1,0 mm,
     Direction = Normal to Face.
  5. **Wiederholung** für die zwei weiteren Slot‑Positionen.
  6. **Spiegelung** auf die zweite Halbschale (Mirror oder erneute
     Anwendung).
- **Alternative Methode bei umlaufenden Reliefs:** Form Workspace →
  T‑Spline → Wrap to Surface (komplexer, nur falls SVG‑Projektion zu
  starke Verzerrung zeigt).
- **Test‑Workflow:**
  1. Flaches 40 × 60 mm Test‑Stück mit einem Relief‑Probedruck (1,0 mm
     Erhebung) → Sicht‑Check.
  2. Bei zufriedenstellender Optik: Reliefs in die finalen Halbschalen‑
     STLs einbauen, Halbschalen drucken.
  3. Falls Reliefs zu fein/zu grob: Erhebung 0,8 ↔ 1,2 mm anpassen,
     Linienbreite ≥ 0,8 mm sicherstellen.
- **Fallback:** Falls Reliefs‑CAD nicht rechtzeitig vor Halloween 2026
  fertig: Erstauflage **ohne Reliefs** drucken. Reliefs sind ein
  Optik‑Bonus, kein Funktions‑Bauteil – das Stab‑Design funktioniert
  auch ohne. Spätere Halbschalen‑Generation kann nachgerüstet werden.
- **Status:** Konzept ✅, SVG‑Quelle offen, CAD‑Modellierung offen.

### 34. Wand V3 – Kompakte PCB + festes USB4‑Kabel (Architektur‑Update)

- **Beschluss 2026‑05‑17:** Nach dem ersten PCB‑Layout‑Wurf in EasyEDA
  hat sich gezeigt, dass alle aktiven Bauteile in **62 × 24 mm
  rechteckig** Platz finden – die ursprünglich geplante T‑Form
  (130 × 16/22 mm) ist nicht nötig. Architektur‑Konsequenzen:
  1. **PCB sitzt nur unter dem Trigger zentriert** im Griff (nicht über
     den ganzen Griff). Restlicher Innenraum frei für Kabel.
  2. **USB‑C‑Buchse auf dem PCB**, fest installiert.
  3. **Stab‑Kabel = ummanteltes USB4‑Kabel**, fest mit der internen
     USB‑C‑Buchse verbunden. Nur das Kabel selbst tritt am Knauf aus.
  4. **Kein externer USB‑C‑Stecker mehr** → kindersicher, das Kabel
     kann nur durch Aufschrauben der Halbschalen gelöst werden.
  5. **Klemmbacken‑Zugentlastung im Knauf** (Variante A): zwei
     gedruckte Backen + M3‑Schraube klemmen das Kabel 15 mm hinter der
     USB‑C‑Buchse fest. Zugkraft → Halbschale → Verschraubung,
     nicht auf den Stecker.
  6. **TPU‑Knickschutz‑Tülle entfällt** – die Kabel‑Ummantelung
     übernimmt die Funktion.
- **Vorteile gegenüber Doku‑Vorgängerplan (Spiralkabel + externer
  USB‑C‑Stecker im Knauf):**
  - kindersicher gegen versehentliches Abziehen
  - robuster Kabel‑Mantel als integrierter Knickschutz
  - optisch unauffälliger (sieht aus wie ein Headset‑Kabel)
  - einfachere Knauf‑Konstruktion (kein 90°‑Bogen mehr)
- **Doku‑Updates erledigt 2026‑05‑17:** [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md)
  §1, §4.3, §5.1, §5.4, §5.5, §5.7, §6, §6.1, §11.
- **Status offen:** mechanische Verifikation der Klemmbacken‑Wirkung
  bei realem Zugtest (= 3 kg Kabel‑Zug muss die Backen halten, ohne
  dass der Stecker rutscht).
- **Status:** Konzept ✅, Doku ✅, Klemmbacken‑CAD und Druck‑Test offen.

### 35. Wand V3 – PCB‑Bestellung erledigt, Folge‑Verifikationen offen

- **Erledigt 2026‑05‑17:**
  - Schaltplan komplett in EasyEDA (5 Funktionsblöcke: Station‑Connector,
    Trigger, Trigger‑Beleuchtung, IR‑LED‑Treiber, Laser‑Treiber)
  - PCB‑Layout komplett (Wand 62 × 24 mm + Optik 10 × 14 mm)
  - DRC sauber (nur die bekannten USB‑C‑Pad‑Clearance‑Warnings, von
    JLCPCB problemlos fertigbar)
  - Beide PCBs bei JLCPCB bestellt:
    - Wand‑PCB: weiß, 1,6 mm, HASL with Lead, 5 Stück, $2,10
    - Optik‑PCB: schwarz, 1,6 mm, HASL with Lead, 5 Stück, $4,20
    - Subtotal $6,30 + DHL Express ~$15 = ~$21 Gesamt
- **Wichtige Designentscheidungen:**
  - **Wand‑PCB weiß**, weil weiße Lötmaske das Halo‑NeoPixel‑Licht
    reflektiert → verstärkter Edelstein‑Glow
  - **Optik‑PCB schwarz**, weil schwarze Lötmaske IR‑Streulicht
    absorbiert → cleaner Lichtkegel durch die Linse
  - **HASL with Lead** statt bleifrei spart ~$26 (für Privatprojekt OK)
  - **NeoPixel‑Footprint Standard 4020 mit erweiterten Pads** – sowohl
    liegende als auch stehende Montage möglich, Final‑Entscheidung
    nach LED‑Lieferung
- **Folge‑Aufgaben (offen):**
  1. **Klemmblock‑CAD** in Fusion 360 (10 × 14 × 6 mm Druckteil, das
     das Optik‑PCB von hinten gegen die LED‑Aufnahme drückt). Siehe
     [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md) §3.3.1.
  2. **Optik‑Block‑CAD** in Fusion 360 (28 × 28 × 35 mm mit Linsen‑
     Aufnahme, Laser‑Madenschrauben‑Halterung, LED‑Aufnahme,
     Klemmblock‑Sitz, M3‑Insert im Boden).
  3. **NeoPixel‑Footprint‑Verifikation** nach LED‑Lieferung – castellated
     Pads checken, dann Entscheidung liegend vs. stehend.
  4. **Halbschalen‑CAD** (290 × 38 mm mit Aussparungen für Switch,
     PCB‑Stand‑offs, Trichter, Klemmbacken im Knauf).
  5. **Edelstein‑Keycap STL** (hexagonal Ø 14 × 8 mm transparentes PETG).
  6. **Sammelbestellung Bauteile bei Reichelt**: USB‑C‑Buchsen,
     TSAL6100, BSS138, R/C‑Sortiment, JST‑PH‑Stecker.
  7. **USB4‑Kabel bestellen** (1,5 m, ummantelt, USB‑C male auf male).
  8. **Bestückung der PCBs** nach Lieferung (Hand‑Lötkolben mit
     verbleitem Lot).
  9. **Erst‑Test mit Station V2**: USB‑C verbinden, Trigger drücken,
     IR + Laser + Halo prüfen.
- **Status:** PCB ✅, Bestellung ✅, mechanische CAD und Bestückung
  offen.

### 32. Wand V3 – Linsen-Schutzdistanz und Spitzen-Schutz-Trichter

- **Beschluss 2026‑05‑17:** OM3 sitzt 15 mm hinter der Stab‑Frontkante
  (statt vorher 5 mm). Die Spitze hat einen konischen Innen‑Trichter
  mit Front‑Öffnung Ø 14 mm, der den IR‑Strahl nicht beschneidet
  (Strahl‑Mathematik in [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md) §3.4.1),
  aber die kratzempfindliche Acryl‑Linse vor direktem Pfosten‑/Finger‑
  kontakt schützt. Front‑Stoßlast geht über die Halbschalen‑Vorderkante
  in die Halbschalen, nicht über die Linsen‑Frontplatte und ihre
  M2‑Schrauben.
- **Was zu verifizieren ist:**
  1. **Probedruck Spitze + Schutz‑Trichter** – ~80 mm langes Test‑Stück
     (Halbschalen‑Vorderkante mit konischem Innen‑Trichter Ø 14 → 26 mm
     + Optik‑Block‑Sitz), mit einer eingelegten OM3 + TSAL6100 auf einem
     Test‑Treiber bei 5 m und 10 m schießen. Beam‑Durchmesser an der
     Front‑Öffnung mit Smartphone‑Kamera (sieht 940 nm als violetten
     Schein) prüfen.
  2. **Front‑Aufprall‑Test:** Test‑Stück mit eingelegter Linse auf eine
     Holzkante fallen lassen (1 m Höhe) und auf Linsen‑Kratzer / Block‑
     Verschiebung prüfen. Falls die Halbschalen‑Front nach 10 Drops
     sichtbar beschädigt ist → TPU‑Schutzring (Innen‑Ø 30 mm,
     Außen‑Ø 34 mm, Länge 15 mm) als Option in Erstauflage aufnehmen.
- **IR‑Schutzscheibe (Variante C):** für V1 nicht bestückt. Slot im
  Optik‑Block aber so vorbereiten, dass eine 1 mm IR‑durchlässige
  Acryl‑Scheibe (z. B. Plexiglas IR Acryl GS 1146, ~5 €/100 × 100 mm)
  nachgerüstet werden kann, falls Halloween 2026 sichtbare Linsen‑
  Kratzer zeigt.
- **Status:** Konzept ✅, Probedruck + Aufpralltest offen.

### 36. Target‑Firmware: WiFiManager durch ESP‑NOW‑Config ablösen

- **Beschluss 2026‑05‑18:** Mit der Einführung der zentralen Config‑Box
  ([`18-config-tool.md`](18-config-tool.md)) wird die heutige Captive‑
  Portal‑Konfiguration am Target unnötig und sollte entfernt werden.
- **Quelle (heute):** `Infinitag Target/src/main.cpp` nutzt
  `WiFiManager` mit vier Custom‑Parametern (`sound_id`, `hit_time`,
  `sw_animation`, `ip_prefix`); Konfig‑Portal wird per System‑IR‑Frame
  (`isSystem=true, cmd=1`) ausgelöst. Siehe
  [`08-software-target.md`](08-software-target.md), Abschnitt
  „Konfiguration via Captive‑Portal".
- **Neuer Stand:** Target bootet im STA‑Mode (`WiFi.mode(WIFI_STA)`),
  initialisiert ESP‑NOW (siehe [`12-refactor-station-v2.md`](12-refactor-station-v2.md)
  § 3.8), antwortet auf `DISCOVER_REQ` und akzeptiert `CFG_WRITE` zum
  Persistieren. Keine SSID, keine Portal‑Seite, keine `WiFiManager`‑Lib
  mehr im `lib_deps`.
- **Stufenplan:**
  1. **V1‑Übergang:** ESP‑NOW parallel zum WiFiManager im Code –
     Target funktioniert sowohl mit alter Captive‑Portal‑Konfiguration
     als auch mit Config‑Box. Beide schreiben in dieselben
     `Preferences`‑Keys (`soundid`, `hittime`, `swani`).
  2. **V2:** WiFiManager und `ip_prefix` raus. `HTTP`‑Client raus.
     `HIT_REPORT` per ESP‑NOW statt `http.begin(serverPath)`.
     Damit löst sich Punkt 2 (hartcodierte 192.168.178.154‑URL) gleich
     mit auf.
- **Was zu verifizieren ist:**
  1. Co‑Existenz von ESP‑NOW und WiFiManager im STA‑Mode – wechselt
     `WiFiManager.startConfigPortal()` den Mode auf `AP_STA` und reißt
     ESP‑NOW ab? Wenn ja: in V1‑Übergang nur eine Modalität gleichzeitig
     aktiv halten (Default ESP‑NOW, WiFiManager nur via System‑IR‑Frame
     eingeschaltet).
  2. `Preferences`‑Namespace `target-data` bleibt erhalten, neue Felder
     `cooldown_ms` (`cooldown`) und `sw_channels` (`swch`) ergänzen –
     siehe Punkt 37.
- **Status:** Konzept ✅, Code‑Umsetzung offen.

### 37. Target: Cooldown‑Feld und Sw‑Channel‑Maske einführen

- **Beschluss 2026‑05‑18:** Im Zuge des Config‑Tool‑Konzepts
  ([`18-config-tool.md`](18-config-tool.md) § 6.2) bekommt das Target zwei
  neue Konfigurationsfelder, die heute fehlen.
- **Heutiger Stand:** Target wechselt nach Ablauf von `hit_time` sofort
  zurück in Idle und ist wieder triggerbar. Alle drei Schaltausgänge
  (SW1 potentialfrei, SW_5V, SW_3V3) schalten synchron. Siehe
  [`08-software-target.md`](08-software-target.md) Z. 70–84.
- **Neue Felder:**

  | Feld | Default | Bedeutung |
  |---|---|---|
  | `cooldown_ms` | 2000 | Nach Ablauf von `hit_time_ms`: weitere Treffer für diese Zeit ignorieren, LED‑Ring schwarz. Verhindert das Endlos‑Triggern einer Halloween‑Prop. |
  | `sw_channels` | `0b111` | Bitmaske: bit0 = SW1, bit1 = SW_5V, bit2 = SW_3V3. Damit lassen sich die drei Ausgänge gezielt einzeln aktivieren statt immer alle drei. |

- **Was zu tun ist:**
  1. Neue Keys `cooldown` (uint16) und `swch` (uint8) im `Preferences`‑
     Namespace `target-data` ergänzen, mit oben genannten Defaults bei
     fehlendem Eintrag.
  2. State‑Machine im `loopAnimation()`/`setSw()`‑Pfad um `COOLDOWN`‑Zustand
     erweitern: nach `targetHit()`‑Ablauf nicht direkt zurück in Idle,
     sondern in `COOLDOWN` für `cooldown_ms`.
  3. `setSw()` auf drei getrennte Channels splitten und `sw_channels`‑
     Bitmaske auswerten.
- **Status:** Konzept ✅, Code‑Umsetzung offen. Zielfenster: vor
  Halloween 2026.

### 38. Target: IR‑System‑Befehl `SELECT_FOR_CONFIG` latent vorhalten

- **Beschluss 2026‑05‑18:** Der ursprünglich angedachte IR‑Pointer am
  Config‑Tool wurde zugunsten von Discovery + Identify‑Blink verworfen
  ([`18-config-tool.md`](18-config-tool.md) Abschnitt „IR‑Pointer").
  Der Protokoll‑Hook bleibt aber reserviert, um in einer späteren Saison
  ohne Hardware‑Änderung am Target nachrüstbar zu sein.
- **Was zu tun ist:**
  1. Im Target‑Code in `checkIrData()` einen Else‑Zweig für
     `isSystem == true && cmd == 0xC0` ergänzen: empfangenes
     `cmdValue`‑Byte (Token) als Echo im `IR_SELECT_ECHO`‑Paket
     broadcasten (siehe [`12-refactor-station-v2.md`](12-refactor-station-v2.md)
     § 3.5).
  2. Konstante `IR_CMD_SELECT_FOR_CONFIG = 0xC0` in `Infinitag_Core`
     definieren.
- **Aufwand:** ~5 Zeilen Code, kein zusätzlicher Test, kein Hardware‑
  Eingriff. Macht das Tor zu „IR‑Pointer am Config‑Tool" offen.
- **Status:** Konzept ✅, Code‑Umsetzung optional – kann zusammen mit
  Punkt 36 erledigt werden.

### 39. Optik‑Block‑Justage: VLHW5100 vs. TSAL6100 Chip‑Z‑Position verifizieren

- **Hintergrund (2026‑05‑23):** Für die Optik‑Block‑Justage wird die
  weiße Vishay VLHW5100 als sichtbarer Zwilling der unsichtbaren
  TSAL6100 eingesetzt (Bestellung Farnell BestNr. 3777973, 100 Stück
  bei 0,21 €/Stk, Express‑Lieferung erwartet 27.05.2026). Beide LEDs
  haben das gleiche 5 mm T‑1¾ Gehäuse, ±10° Halbwertswinkel und kommen
  vom gleichen Hersteller. Details:
  [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md) § 3.2.1.
- **Annahme:** Die Z‑Position des Dies im Gehäuse ist bei beiden Vishay‑
  LEDs gleich, sodass der visuell mit der VLHW5100 eingestellte Brennpunkt
  1:1 auf die TSAL6100 übertragbar ist.
- **Risiko:** Die Annahme ist datenblatt‑gestützt, aber nicht direkt
  spezifiziert. Eine Abweichung von 0,3–0,5 mm zwischen den Familien
  würde den 2 mm‑Defokus messbar verschieben (Cone‑Durchmesser bei 5 m
  verändert sich um ~3 cm pro 0,5 mm Defokus‑Drift).
- **Was zu prüfen ist:**
  1. **Mechanik:** Beide LEDs nebeneinander mit Schieblehre messen –
     Sitz‑Tiefe (Boden bis Dome‑Oberkante) und Dome‑Radius. Sollte
     identisch sein.
  2. **Sichtbar:** Mit VLHW5100 im Optik‑Block den Defokus an einer
     weißen Wand bei 5 m auf 13 cm Cone einstellen.
  3. **IR‑Check ohne Umbau:** VLHW5100 herausziehen, TSAL6100 einsetzen
     (Press‑Fit‑Sitz). Mit Handy‑Frontkamera (oft besserer IR‑Durchlass)
     bei 5 m fotografieren – das Lila‑Glühen am Ziel sollte ähnlich
     scharfen Kreis ergeben wie der weiße Spot. Sichtbar abweichend?
     Defokus‑Drift dokumentieren.
  4. **Empfänger‑Reichweite:** Mit TSOP38338 am 5 m‑Ziel: Schuss‑Trefferquote
     vergleichen mit alter Konfiguration. Soll mindestens gleich, idealerweise
     besser sein.
  5. Ergebnis (Abweichung in mm, Cone‑Differenz) zurück in
     [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md) § 3.2.1
     dokumentieren.
- **Fallback bei Abweichung:** Distanzscheibe (0,1–0,5 mm) zwischen LED
  und Optik‑Block‑Bohrungsboden einlegen, dann TSAL6100 final montieren.
  Mehr ist nicht nötig – Defokus ist eine reine Z‑Verschiebung.
- **Status:** Bauteil bestellt (Farnell, Bestätigung steht noch aus,
  Lieferung erwartet 27.05.2026). Verifikation ausstehend, bis erste
  Optik‑Blöcke gedruckt sind (Phase 1 Wand V3, siehe
  [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md) § 1).

### 40. Stab‑Connector: USB‑C → 6‑pol Schraubklemmstecker + PCB‑Split (Entscheidung 2026‑06‑08)

> **Stand 2026‑06‑08: umgesetzt.** Alle drei PCBs (Data, Audio, Wand) sind in
> EasyEDA geroutet und DRC‑sauber (V1), bereit zur Bestellung. Wand‑Kabel
> landet auf einem 2×3‑Lötheader `J1`. Finaler Layout‑Stand + Pre‑Order‑
> Checkliste: [`19-pcb-split-netliste.md`](19-pcb-split-netliste.md) § 8.

- **Eigentliche Motivation (Tobias):** Die bisherige Schaltung lief im
  Betrieb einwandfrei – **es wird nichts an der Technik geändert**. Zwei
  Ziele: (a) **Connector‑Tausch** USB‑C → Schraubklemme, weil die
  NeoPixel‑Daten zum Zauberstab über den USB‑C‑Pin **nicht durchkamen**
  (= Punkt 3a; die SK‑LEDs am Stab gingen nicht) → der Klemmstecker gibt
  `NEO_DATA` eine eigene, durchgehende Ader. (b) **PCB‑Split** auf zwei
  Platinen rein zur **Platzoptimierung** im kleinen Gehäuse. → **Behebt
  Punkt 3a.**
- **Entscheidung (Tobias, 2026‑06‑08):**
  1. **Stab‑Connector** wechselt von USB‑C auf einen **grünen 6‑pol
     PCB‑Schraubklemmstecker** auf der Station; das **Stab‑Kabel wird
     wandseitig direkt angelötet** (kein Stecker am Stab).
  2. **ESD:** Das alte 4‑Kanal `USBLC6‑4SC6Y` (D2 am Stab‑Connector der
     Station) wird durch **2× `USBLC6‑2SC6` (D1 + D2)** ersetzt – **bleibt auf
     dem Station‑Data‑Board an CN1** (Pass‑Through inline, VBUS an +5 V),
     R1–R4 (100 Ω) bleiben. Die **Wand‑PCB bekommt kein eigenes ESD‑Array**.
     USB‑Verwechslungsschutz/PD‑Klemmung hinfällig. `SP3012‑04UTG` verworfen
     (in DE schlecht verfügbar). *(Korrigiert 2026‑06‑08: ESD doch auf der
     Station, nicht auf der Wand – entspricht dem funktionierenden Aufbau.)*
  3. **Station‑PCB‑Split:** zwei gestapelte PCBs (Logic + Audio,
     Rücken‑an‑Rücken, M3‑Nylon‑Standoffs, Inter‑Board‑Kabel JST‑XH 8‑pol).
     12 V + Buck aufs Audio‑Board, 5 V rüber zum Logic‑Board.
  4. **OLED + 4 Tasten = optional** (Konfig primär übers Config‑Tool per
     ESP‑NOW, Doc 18); Footprint bleibt.
  5. **EasyEDA:** ein Projekt, **zwei getrennte Schaltpläne** → je ein PCB.
- **Bereits dokumentiert:**
  [`12-refactor-station-v2.md`](12-refactor-station-v2.md) § 8.11 + neuer
  § 8.14, Statustabelle + Checkliste; [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md)
  Statustabelle, § 4.1/4.2, BOM § 6/6.1, Q&A, Checkliste (mit Update‑Banner
  in § 4).
- **Noch offen – Cross‑Doc‑Sync (Verifikationspunkt):** Folgende abgeleitete
  Docs tragen noch die alte USB‑C‑/USBLC6‑4SC6‑/SP3012‑Formulierung und sind
  in einem Folgedurchgang nachzuziehen:
  - [`14-pcb-readiness.md`](14-pcb-readiness.md) – Test 2.10 (USB‑C 6‑Adern),
    ESD/Filter‑Beschaltung „an der USB‑C‑Buchse", Pin‑Layout‑Tabellen.
  - [`15-bestellliste-station-v2.md`](15-bestellliste-station-v2.md) –
    USB‑C‑Buchse C165948, USBLC6‑4SC6 C7415, USB4‑/Anker‑Kabel‑Recherche
    (§ 3.6 / § 6.2) → ersetzen durch Klemmstecker + 2× USBLC6‑2SC6 +
    6‑adriges Mantelkabel.
  - [`16-pcb-layout-hinweise.md`](16-pcb-layout-hinweise.md) – § 5
    „Stab‑Connector (USB‑C)", VBUS‑Pad‑Rule, Routing‑Erwähnungen.
  - [`17-bring-up-prototyp-1.md`](17-bring-up-prototyp-1.md) – Schritt 4
    „USB‑C Buchse", § 5.3 Durchgangstest (historisches Log – nur Hinweis,
    nicht umschreiben).
  - [`10-bill-of-materials.md`](10-bill-of-materials.md) – 1 USB‑C‑Eintrag.
- **Begründung der Aufteilung:** Die USB‑C‑Lösung brachte Verwechslungs‑
  gefahr, Reversibilitäts‑Doppelverdrahtung und teure Kabel‑Beschaffung mit
  eMarker‑Test – alles hinfällig, da Station festgeschraubt + Stab angelötet
  (kein Hot‑Plug). Volle Begründung in § 8.11.

### 41. Infinitag Now: IR‑Schuss‑Telegramm festlegen (mit Target‑Firmware)

**Stand 2026‑07‑13:** Die Station schießt im Live‑Betrieb (Station‑PR #14)
einen **rohen 5‑ms‑Burst @ 38 kHz** – bewusst noch ohne Datentelegramm.
Das Target braucht den Schützen nicht zu identifizieren (es kennt „seine"
Station aus der Config, `station_mac` im Target‑Blob, siehe `PROTOCOL.md`),
aber ein reiner Burst ist nicht gegen Fremd‑IR (Fernbedienungen,
Sonnenlicht‑Flackern) abgesichert.

**Zu entscheiden mit der Target‑Firmware (`infinitag-now-target`):**

- Klassisches 24‑Bit‑RC5‑Telegramm aus Doc 05 wiederverwenden
  (`Infinitag_Core::irEncode`, `isSystem=false, cmd=1`) – Vorteil:
  vorhandene, erprobte Decode‑Routine; Nachteil: IRremote‑Abhängigkeit.
- Oder eigenes minimales Now‑Format (z. B. Header‑Burst + 8 Bit Magic),
  bit‑gebangt über LEDC – Vorteil: schlank, RMT/LEDC ohne Lib.
- Latenz‑Budget beachten: < 50 ms IR‑Decode → `HIT_REPORT` → Sound‑Start
  (Doc 12 § 3.7).

Quelle: Station `src/main.cpp` (`IR_SHOT_MS`, Kommentar am Define),
PROTOCOL.md „Spielbetrieb". Der Kalibriermodus (DBG_CALIBRATE, Test 6)
ist davon unabhängig – er nutzt Dauerlicht statt Telegramm.
