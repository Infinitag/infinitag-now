# 18 – Config‑Tool: Zentrale Konfigurations‑Box für Stationen & Targets

> **Status: lebendes Konzept‑Dokument** – beschreibt das Werkzeug, mit dem
> Tobias seine Station‑V2‑Geräte und Targets im Vorgarten konfiguriert,
> ohne sich auf jedes Gerät einzeln per Captive‑Portal aufzuschalten.
> Wird laufend aktualisiert, bis eine getestete und für Halloween 2026
> produktive Version steht.

Beginn der Diskussion: 2026‑05‑18 (Chat mit Claude). Hintergrund: das
heutige Target‑Setup über WiFiManager‑Captive‑Portal ist umständlich – pro
Target einmal Phone in den eigenen Hotspot des Targets buchen, Werte
eingeben, Hotspot wieder verlassen. Mit ESP‑NOW als Backbone (siehe
[`12-refactor-station-v2.md`](12-refactor-station-v2.md) § 3) lohnt sich
ein zentrales Werkzeug, das alle Geräte über das gleiche Funkprotokoll
erreicht.

> **Revision 2026‑07‑08** (Chat mit Claude): Hardware‑Basis geändert –
> **ESP32‑C3 Super Mini** statt S3‑DevKitC‑1U, **4×AA‑Versorgung** statt
> USB‑C‑only, **0,96"‑OLED (SSD1315) mit 4 Tasten** statt 1,3"‑SH1106
> (DST‑015‑0). Begründung und Konsequenzen in § 4. Bedienkonzept, Menü,
> Flows und Protokoll (Doc 12 § 3) bleiben unverändert gültig.

---

## 1. Status‑Übersicht

| Bereich | Entscheidung | Status |
|---|---|---|
| MCU | **ESP32‑C3 Super Mini** (Variante mit IPEX + externer Antenne). Ersetzt den S3‑DevKitC‑1U vom 18.05. – kleiner, günstiger, reicht für die Aufgabe; Trade‑offs siehe § 4.1 | ✅ geändert 2026‑07‑08 |
| Antenne | Externe 2,4‑GHz‑Antenne über IPEX‑Pigtail (beim C3‑Board dabei) | ✅ geändert 2026‑07‑08 |
| Versorgung | **4×AA** über Schiebeschalter + Schottky‑Diode an 5V‑Pin; USB‑C fürs Flashen/Service – **aber nicht gleichzeitig mit frischen Alkalines** (Backfeed ~6 V in den Host, Details Doc 20 § 2). Regel: erst Schalter aus, dann USB rein. Ersetzt „USB‑C only" vom 18.05. | ✅ geändert 2026‑07‑08, Einschränkung 2026‑07‑09 |
| **Persistenz‑Modell** | **Stateless** – Config‑Box speichert nichts dauerhaft über Stationen/Targets. Jeder Discovery‑Zyklus baut die Liste neu. Source of Truth ist das jeweilige Gerät selbst (NVS). | ✅ gesetzt 2026‑05‑18 |
| Bedien‑Eingaben | **Rotary Encoder (KY‑040 = EC11 mit Push auf Breakout)** als Primärbedienung, **OLED + 4‑Tasten‑Modul** als sekundäre/spezialisierte Eingabe | ✅ gesetzt 2026‑05‑18, KY‑040 bestätigt 2026‑07‑08 |
| OLED | **0,96" SSD1315 128×64 I²C** mit integriertem 4‑Tasten‑Board (K1–K4). Ersetzt das 1,3"‑SH1106/DST‑015‑0 vom 18.05.; Tastenbelegung bleibt identisch zur Station | ✅ geändert 2026‑07‑08 |
| Web‑UI | **SoftAP** auf der Config‑Box (`Infinitag-Config` SSID), Mini‑Webserver mit Übersicht + Edit‑Forms, Sound‑Namen statt ‑IDs | ✅ gesetzt 2026‑05‑18 |
| Geräte‑Auswahl | **Discovery + Identify‑Blink** (Standardweg). IR‑Pointer **nicht** in V1 (Tobias hat ihn 2026‑05‑18 verworfen zugunsten des Encoders); Protokoll‑Hook (`IR_SELECT_ECHO`) bleibt latent reserviert | ✅ gesetzt 2026‑05‑18 |
| Status‑LED | **Gestrichen** – das OLED zeigt alle Status (Speichern/Fehler/Setup) als Text; freigewordener Pin ging an die Batteriemessung. Ersetzt SK6812‑Plan vom 18.05. | ✅ geändert 2026‑07‑09 |
| Batterieanzeige | **VBAT‑Messung** über 100k/22k‑Teiler an GPIO3 (ADC1_CH3); Anzeige im Tools‑Menü, < 3 V ⇒ „USB". (Ursprünglich 47k unten geplant, gebaut mit 10k+12k in Serie) | ✅ neu 2026‑07‑09, Teiler angepasst 2026‑07‑09 |
| Gehäuse | 3D‑Druck‑Box, OLED‑Fenster + Encoder‑Achse + 4 Drucktasten an der Frontplatte, USB‑C an der Rückseite | offen, CAD nach Lochraster |
| GPIO‑Plan | siehe Abschnitt 4 | ✅ Konzept, im Lochraster zu verifizieren |
| Erster Hardware‑Schritt | Lochraster‑Prototyp auf Steckbrett (C3 Super Mini + OLED + Encoder lose verkabelt) | offen, nach Bestellung |
| **Firmware V0.1** | PlatformIO‑Projekt in `/Volumes/basteln/Infinitag/firmware/config-box/` – Kern ohne Web‑UI (Discovery, Identify, Editor, Setup‑Flow, Sound‑Test, Live‑Monitor). Protokoll‑Lib `InfinitagNow` mit Unit‑Tests, für spätere Wiederverwendung in Station/Target | ✅ geschrieben 2026‑07‑08, ungetestet auf HW |

---

## 2. Worum es geht / Warum überhaupt

**Pain Point heute (Halloween 2025):**

- Pro Target muss man sich einzeln per Phone auf dessen WiFiManager‑Hotspot
  buchen, Captive‑Portal aufrufen, die vier Custom‑Params eingeben, Hotspot
  verlassen, Target neu starten.
- Sound‑ID, Hit‑Time, Switch‑Animation und IP‑Prefix sind nur Zahlen – „welche
  Sound‑ID war noch mal der gute Schrei?" steht nirgendwo.
- Die Station hat zwar Service‑Klappe + OLED, aber die ID lässt sich heute
  nur per Reflash setzen (DIPs sind im Schaltplan nicht vorgesehen).
- Hartcodiertes `192.168.178.154:80` im Target‑Code (siehe Punkt 2 in
  [`11-offene-punkte.md`](11-offene-punkte.md)) ist Notnagel und bricht beim
  nächsten Routerwechsel.

**Was die Config‑Box löst:**

- Ein Gerät, ein Bedien‑Modell, alle Stationen + Targets erreichbar.
- Geräte‑Auswahl visuell durch Blink‑Identify – kein „welche Box ist Target 3".
- Sound‑Namen statt ‑Nummern (Web‑UI).
- Kein WLAN‑Router mehr nötig, da ESP‑NOW Peer‑to‑Peer läuft (Hauptgrund
  steht in Doc 12).
- WiFiManager am Target kann entfallen (Doc 11 / neuer Punkt).

**Was die Config‑Box ausdrücklich nicht tut:**

- Sie ist **kein Game‑Master** – sie schaltet sich im Spiel nicht ein, hat
  keine Aufgabe in der Treffer‑Pipeline. Sie wird zur Setup‑Zeit angesteckt
  und vor dem Spielbetrieb wieder weggepackt.
- Sie speichert **keine Soll‑Konfiguration** der Geräte. Wer im Garten
  steht, ist der Master.
- Sie ist **kein WLAN‑Bridge** zum Internet. Ihr SoftAP ist nur für den
  lokalen Web‑UI‑Zugriff.

---

## 3. Bedienkonzept: drei Eingaben für drei Komfortstufen

| Eingabe | Was geht damit | Wann nutze ich's |
|---|---|---|
| **Rotary Encoder** (drehen + drücken) | komplett alles, sehr flink durch Listen | Standardweg, immer zuerst greifbar |
| **4 OLED‑Tasten** | komplett alles + spezialisierte Modifier (großer Schritt, Identify‑Toggle) | wenn der Encoder nicht reicht (z. B. Springen in langer Liste) oder als Fallback |
| **Web‑UI im SoftAP** | komplett alles + Sound‑Namen statt ‑IDs + tabellarische Übersicht über alle Geräte | wenn ich mehrere Geräte am Stück konfiguriere oder Sound‑Klartext brauche |

**Encoder‑only (Revision 2026‑07‑11):** Alle Menüs sind **vollständig ohne
die K‑Tasten bedienbar** – jede Ebene hat einen expliziten „< Zurück"‑Eintrag
(in Gerätelisten zusätzlich „Neu suchen"), und schnelles Drehen wirkt im
Wert‑Editor automatisch als ×10. K1–K4 bleiben als Shortcuts erhalten
(einzige Ausnahme: der Identify‑Blink‑Toggle liegt nur auf K3).

**Bewusste Redundanz:** Encoder und K2/K3/K4 können *dasselbe* Bedien‑Ziel
erreichen. Das ist gewollt, weil die Bedien‑Hand zwischen „Knopf greifen"
und „Tasten tippen" wechseln darf, ohne Kontext zu verlieren.

**Tastenbelegung:**

| Eingabe | In Menü/Liste | Im Edit‑Wert |
|---|---|---|
| Encoder drehen | Cursor auf/ab | Wert ±1 |
| Encoder drehen + K2 gehalten | Cursor ±10 (Schnellsprung) | Wert ±10 |
| Encoder Push | OK / Enter | OK / Wert übernehmen |
| K1 (Menü/Zurück) | Zurück eine Ebene | Abbrechen ohne Speichern |
| K2 (∧) | Modifier „großer Schritt" (Halten) | Modifier „großer Schritt" (Halten) |
| K3 (∨) | Identify‑Blink an/aus toggeln | – |
| K4 (OK) | OK (gleichwertig zu Encoder‑Push) | OK (gleichwertig) |

K1/K4‑Semantik (Zurück / OK) bleibt **identisch zur Station** – wer am
OLED‑Menü der Station gewohnt ist, findet sich an der Config‑Box ohne
Umlernen zurecht.

K3 als „Identify‑Blink aus" ist praktisch, wenn man Halloween‑Abend im
Garten steht, die Boxen schon eingebaut sind und nicht jedes Mal die
LEDs blinken sollen, wenn man die ID checkt.

---

## 4. Hardware‑Plattform

### 4.1 Board: ESP32‑C3 Super Mini (IPEX + externe Antenne)

**Revision 2026‑07‑08:** ursprünglich war der S3‑DevKitC‑1U gesetzt
(gleiches Board wie Station V2). Tobias hat auf den **C3 Super Mini**
umentschieden. Gründe:

- Deutlich kleiner → Handheld‑Gehäuse schrumpft spürbar.
- Günstig (~7–8,50 € inkl. IPEX‑Pigtail + SMA‑Antenne, z. B. RCmall).
- Die Aufgabe braucht weder PSRAM noch Dual‑Core noch viel Flash –
  Menü, ESP‑NOW, SoftAP‑Web‑UI und LittleFS (`sounds.json`) laufen auf
  dem C3 uneingeschränkt.

**Bewusst in Kauf genommene Trade‑offs** (Gegenargumente vom 18.05.):

- **GPIO‑Budget knapp**: 10 von ~12 nutzbaren Pins belegt (§ 4.3).
  Erweiterungen wie SD‑Slot oder zweiter Encoder entfallen.
- **Kein PCNT‑Peripheral im C3** → Encoder wird per GPIO‑Interrupt
  gelesen statt über den Hardware‑Pulszähler. Bei ≤ 50 Pulsen/s
  (schnelles Drehen von Hand) unkritisch.
- **Eigene PlatformIO‑Env** (`esp32-c3-devkitm-1` o. ä.) neben der
  S3‑Env der Station – zwei Board‑Typen im Projekt.

**Wichtig beim Bestücken:** die Variante **mit IPEX‑Buchse und externer
Antenne** verwenden. Die Onboard‑Keramikantenne der Super‑Mini‑Klone ist
notorisch schwach – für Discovery quer durch den Vorgarten die externe
Antenne anschrauben.

### 4.2 Versorgung: 4×AA + USB‑C parallel

**Revision 2026‑07‑08:** statt „USB‑C only" jetzt Batteriebetrieb:

```
4×AA (6,4…4,0 V) ── Schiebeschalter ── Schottky 1N5817 ── 5V‑Pin C3
                                                            │
USB‑C (Flashen/Service) ────────────────────────────────────┘
```

- Der Onboard‑LDO des Super Mini (ME6211, **max. 6,5 V Eingang**)
  bekommt durch die Diode (~0,3–0,4 V Drop) höchstens ~6,1 V ab –
  auch mit vier frischen Alkalines im grünen Bereich. **Diode ist
  Pflicht, nicht optional.**
- Die Diode verhindert zugleich Rückspeisung in die Batterien, wenn
  USB‑C angesteckt ist → Flashen am Schreibtisch jederzeit möglich,
  mit oder ohne eingelegte Zellen.
- Stromaufnahme geschätzt **80–100 mA** (C3 mit aktivem WiFi + OLED +
  LED) → mit 2000‑mAh‑Alkalines **15–20 h** Laufzeit. Für ein
  Setup‑Werkzeug mehr als genug.
- Leere Batterien (< ~4,0 V gesamt) äußern sich als Brownout‑Resets.
  Eine Batteriespannungs‑Messung per Spannungsteiler ist mangels
  freier Pins (und wegen Strapping‑Randbedingungen an GPIO2) **nicht**
  vorgesehen – Zellen wechseln und gut.

### 4.3 GPIO‑Plan (C3 Super Mini, Konzept v3)

**Revision 2026‑07‑09 (v3):** Status‑LED gestrichen (OLED zeigt alles).
Der dadurch mögliche Umbau: Encoder‑Push wandert von GPIO3 auf GPIO21,
GPIO3 wird **Batteriespannungs‑Messung** – denn ADC ist auf dem C3 nur
auf GPIO0–4 nutzbar (GPIO5 = ADC2, mit aktivem WLAN unbrauchbar), und
GPIO3 war der einzige sinnvoll freischaufelbare ADC‑Pin.

Nutzbare Pins am Super Mini: GPIO 0–10, 20, 21. Strapping‑Pins: GPIO2
(muss beim Boot high/floating sein), GPIO8 (Onboard‑LED, muss beim Boot
high sein), GPIO9 (BOOT‑Taster). GPIO 20/21 sind zwar UART0‑RX/TX, aber
geflasht/geloggt wird über den nativen USB‑Serial‑JTAG → frei nutzbar.

| GPIO | Funktion | Anmerkung |
|---|---|---|
| **GPIO0** | Encoder A | Interrupt‑Input (kein PCNT im C3) |
| **GPIO1** | Encoder B | Interrupt‑Input |
| **GPIO3** | **VBAT‑Messung** | ADC1_CH3; Teiler VBAT –[100k]– GPIO3 –[10k+12k=22k]– GND (6,4 V → ~1,15 V) |
| **GPIO4** | OLED‑Taste K1 (Menü/Zurück) | Input mit internem Pullup |
| **GPIO5** | OLED‑Taste K2 (∧ / Modifier) | Input mit internem Pullup |
| **GPIO6** | I²C SDA (OLED) | `Wire.begin(6, 7)` |
| **GPIO7** | I²C SCL (OLED) | |
| **GPIO10** | OLED‑Taste K3 (∨ / Identify‑Toggle) | Input mit internem Pullup |
| **GPIO20** | OLED‑Taste K4 (OK) | UART‑RX0, frei bei USB‑CDC |
| **GPIO21** | Encoder Push | UART‑TX0, frei bei USB‑CDC; Input mit internem Pullup |
| GPIO2 | **frei / Reserve** | Strapping: nur Lasten ohne Pulldown (z. B. passiver Piezo) |
| GPIO8 | Onboard‑LED (blau, invertiert) | ohne Verdrahtung als Heartbeat nutzbar |
| GPIO9 | BOOT‑Taster onboard | frei lassen |

**Bilanz:** 10 Pins fest, GPIO2 als einzige Reserve. Der latente
IR‑Pointer‑Ausgang (§ 10) müsste sich GPIO2 nehmen oder den Piezo
verdrängen – bewusster Verzicht auf Erweiterungskomfort.

**VBAT‑Anzeige in der Firmware:** `analogReadMilliVolts(3)` ×
Teilerfaktor 122/22; im Tools‑Menü als „Batt x,xx V". Liest der Pin
< 3 V (keine Batterien, Betrieb an USB), zeigt die Box „USB".

**Verifikationspunkte für den Lochraster‑Prototyp:**

1. **Encoder per Interrupt**: Lib‑Kandidat `AiEsp32RotaryEncoder` oder
   eigene 4‑State‑Quadratur‑ISR. `ESP32Encoder` (PCNT) scheidet auf dem
   C3 aus. Bei Prell‑Sprüngen RC‑Tiefpass (10 kΩ in Serie, 100 nF gegen
   GND) an A/B nachrüsten – erst ohne testen. Hinweis: das KY‑040‑Breakout
   hat bereits 10‑kΩ‑Pullups an CLK/DT onboard.
2. **SSD1315 unter U8g2**: SSD1315 ist SSD1306‑kompatibel –
   `U8G2_SSD1306_128X64_NONAME_F_HW_I2C` sollte laufen; neuere
   U8g2‑Versionen haben auch einen dedizierten SSD1315‑Konstruktor.
3. **K‑Tasten des OLED‑Moduls**: Beschaltung prüfen (erwartet:
   Taster gegen GND, active‑low). Ob das Modul eigene Pullups hat →
   sonst interne Pullups der GPIOs reichen.
4. **6‑V‑Kette messen**: Spannung hinter der 1N5817 bei frischen Zellen
   < 6,3 V bestätigen, bevor der C3 drankommt.

### 4.4 Bauteilliste (BOM, V2‑Lochraster)

| Posten | Bauteil | Stück | Preis ca. | Quelle |
|---|---|---|---|---|
| MCU | ESP32‑C3 Super Mini **mit IPEX + SMA‑Antenne** (RCmall o. ä.) | 1 | 7–8,50 € | AliExpress (Tobias' Fund 2026‑07‑08) |
| OLED + Tasten | 0,96" SSD1315 128×64 I²C mit 4‑Tasten‑Board | 1 | 4,50–6 € | AliExpress (Tobias' Fund) |
| Rotary Encoder | KY‑040 (EC11, 20 Detents, mit Push) | 1 | ~2 € | Amazon 5er‑Pack ~10 € (GIAK, Tobias' Fund) |
| Encoder‑Knopf | beim KY‑040‑Pack dabei, alternativ PETG‑Druck | 1 | 0 € | enthalten |
| Batteriehalter | 4×AA mit Anschlusskabel (GTIWUNG 6er‑Pack) | 1 | ~1,20 € | Amazon 6er‑Pack 6,99 € (Tobias' Fund) |
| Schiebeschalter | beliebiger 1‑pol Schiebe‑/Kippschalter | 1 | 0,30 € | vorhanden |
| Verpolungs-/LDO‑Schutz | Schottky **SS34** (40 V/3 A, SMA‑SMD; ursprünglich 1N5817 geplant, elektrisch gleichwertig) | 1 | 0,10 € | vorhanden |
| VBAT‑Teiler | Widerstände 100 kΩ + 10 kΩ + 12 kΩ (unten 10k+12k in Serie = 22k) | je 1 | 0,05 € | vorhanden |
| Gehäuse | 3D‑Druck PETG | 1 | 0 € | Eigenfertigung |
| USB‑C‑Kabel | beliebiges Datenkabel (nur Service) | 1 | – | vorhanden |
| Lochraster | 50×70 mm Streifenraster | 1 | 0,50 € | vorhanden |
| Drähte | AWG24/26 Silikon | – | – | vorhanden |
| **Gesamt** | | | **~15–18 €** | |

---

## 5. Menü‑Struktur

OLED‑Layout: 128 × 64 Pixel, U8g2 Library, Font `u8g2_font_6x10_mf` für
Listen (6 Zeilen × 21 Zeichen sichtbar) und `u8g2_font_helvB10_tf` für
Überschriften.

**Revidiert 2026‑07‑11 – geräte‑zentriert:** erst das Gerät aus der Liste
wählen, danach öffnet sich dessen Geräte‑Menü mit allen Aktionen
(Prusa‑Logik). Die früheren aktions‑zentrierten Untermenüs entfallen.

```
Hauptmenü
├── Stationen                ← Geräteliste: < Zurück / Neu suchen /
│   │                          Neue Station (Stab) / St.01, St.02, …
│   └── Station wählen       ← Identify‑Blink folgt dem Cursor UND bleibt
│       │                      im Geräte‑Menü aktiv („wen konfiguriere ich?")
│       ├── Konfigurieren    ← Editor (ID, Volume, Setup‑Sound) + Speichern
│       ├── Sound testen     ← Sound wählen + abspielen (0x32)
│       └── Selbsttest       ← Prusa‑artig: Sound/LEDs/Laser/IR/Trigger
│                              einzeln oder „Alle testen" (0xF0/0xF1)
├── Targets                  ← Geräteliste: < Zurück / Neu suchen / T.01, …
│   └── Target wählen        → Konfigurieren (Editor)
├── Live‑Monitor             ← zeigt eingehende HIT_REPORTs als Tickerzeile
├── Web‑UI                   ← SoftAP an/aus, zeigt IP + QR‑Code
└── Tools
    ├── ESP‑NOW Sniffer      ← alle Pakete loggen (Debug)
    ├── Firmware‑Info        ← eigene Version, MAC, freier Heap
    └── OTA‑Update           ← lädt fw.bin von SoftAP‑Client per HTTP
```

### 5.1 OLED‑Skizze: Listen‑Ansicht „Stationen"

```
┌─[Stationen]─────────────[3]─┐
│ ▶01  Station-A      -42dBm │  ← Cursor: blinkt Station-A
│  02  Station-B      -68dBm │
│  03  Station-C      -55dBm │
│  ??  ungesetzt      -38dBm │  ← ID=0, frisch geflasht
│  K3=Blink an  K4=Edit       │
└─────────────────────────────┘
```

- `[3]` rechts oben = Anzahl gefundener Geräte
- `??` = `station_id == 0`, also nicht konfiguriert
- RSSI rechts erleichtert „welche Box ist nah dran?"

### 5.2 OLED‑Skizze: Edit‑Ansicht „Station"

```
┌─[Station-A AABBCC]─────────┐
│ ID         : ▶02◀          │  ← Cursor auf Feld, ◀▶ = im Wert‑Edit
│ Volume     :  80 %         │
│ Setup‑Sound:  13           │
│ LED bereit :  G            │  ← Stab‑Farbe „schussbereit"
│ LED aktiv  :  R            │  ← Stab‑Farbe „beschäftigt" (Audio spielt)
│ Sound testen >             │
│ Speichern    ⏎             │
└────────────────────────────┘
```

- Cursor `▶ ◀` markiert ein Feld; Encoder‑Push wechselt in den Wert‑Edit‑Modus
  (Cursor wird `▶02◀`), erneut Push übernimmt.
- `LED bereit`/`LED aktiv`: Kanal‑Maske der SK6812‑RGBW‑Dies, Anzeige als
  Buchstaben der aktiven Kanäle (`R`, `RG`, `RW`, … `RGBW`). Encoder‑Drehen
  läuft durch alle 15 Kombinationen (Bitmaske 1–15, bit0 = R … bit3 = W).
  Blob‑Offsets 3/4, siehe Doc 12 § 3.6.2 (ergänzt 2026‑07‑11).
- `Speichern ⏎` schreibt per `CFG_WRITE` und wartet auf `CFG_ACK`.

### 5.3 OLED‑Skizze: Edit‑Ansicht „Target"

```
┌─[Target-NEU 11FF22]────────┐
│ Target‑ID  :  03           │
│ Station‑ID :  02           │
│ Sound      :  06 Schrei    │  ← Sound‑Klartext nur per Web‑UI; OLED Nummer
│ Hit‑Time   : 10000 ms      │
│ Cooldown   :  2000 ms      │
│ SW‑Anim    :  0 (an/aus)   │
│ SW‑Kanäle  : ●○● 1,3       │
│ Speichern    ⏎             │
└────────────────────────────┘
```

`SW‑Kanäle`‑Anzeige: drei runde Symbole für SW1 (potentialfrei), SW_5V,
SW_3V3, `●` = aktiv, `○` = inaktiv. Encoder‑Drehen in dem Feld toggelt
die drei Kanäle als 3‑Bit‑Wert (8 Kombinationen).

### 5.4 OLED‑Skizze: Live‑Monitor

```
┌─[Live‑Monitor]─────────────┐
│ 21:14:03 T07→S02 sound=06  │
│ 21:14:05 T03→S02 sound=02  │
│ 21:14:08 T07→S02 sound=06  │
│ 21:14:11 T05→S01 sound=11  │
│                            │
│ Filter: alle  K3=Filter    │
└────────────────────────────┘
```

- Zeigt die letzten 5 `HIT_REPORT`‑Broadcasts.
- K3 toggelt einen Filter (alle / nur Station X / nur Target Y).
- Nützlich zum Debuggen, wenn ein Target im Test "auslöst, aber kein Sound
  kommt" – sieht man im Monitor, ob das Paket überhaupt ankommt.

---

## 6. Konfigurations‑Flows

Drei Wege, alle aus dem Hauptmenü erreichbar. Die ESP‑NOW‑Pakete folgen
[`12-refactor-station-v2.md`](12-refactor-station-v2.md) § 3.4–3.7.

### 6.1 Flow A: Stations‑ID per Stab‑Trigger setzen

Vorteil: physisch eindeutig, kein „welche der drei Boxen ist's gleich".

```
Tobias    Config‑Box           Stationen
  │            │                   │
  │ "Neue     │ SETUP_BEGIN        │
  │ Station"  ├──────Broadcast─────►
  │           │                    │ alle: Status‑LED + Stab‑LEDs lila
  │           │                    │ Trigger umgedeutet auf "Bestätigen"
  │           │                    │
  │ geht zur Wunsch‑Station        │
  │ und drückt Trigger             │
  │           │                    │
  │           │ SETUP_TAKE(mac=…, id=03)
  │           ◄──────Broadcast─────┤
  │           │                    │ andere: SETUP_TAKE gesehen → zurück IDLE
  │           │                    │ gewählte: id persistieren, 3× Cyan
  │ OLED: "Station 03 gesetzt"    │
```

Timeout 60 s (im `SETUP_BEGIN`‑Payload). **Präzisiert 2026‑07‑11:** Die zu
vergebende ID wählt die Config‑Box vor dem Broadcast (Encoder‑Drehen im
Setup‑Screen, Vorschlag = höchste bekannte Stations‑ID + 1) und schickt sie
im Header‑Feld `station_id` mit; die Station persistiert genau diese ID. Während der Setup‑Zeit lassen
sich auch weitere Felder vorgeben (Volume, Default‑Setup‑Sound) und werden
in der Trigger‑Bestätigung mitgeschickt – die Station übernimmt dann alle
Werte auf einmal.

### 6.2 Flow B: Geräte über Liste auswählen + editieren

Der reguläre Weg für Targets und für „nachträgliches Editieren" von
Stationen.

```
Tobias    Config‑Box                  Targets
  │            │                          │
  │ "Targets" │ DISCOVER_REQ(type=TARGET)│
  │ Untermenü ├──────Broadcast────────────►
  │           │                          │ alle Targets antworten je
  │           │ DISCOVER_REPLY(payload=cfg)
  │           ◄──Unicast×n──────────────┤
  │           │                          │
  │           │ baut Liste auf OLED      │
  │           │                          │
  │ dreht     │                          │
  │ Encoder   │                          │
  │           │ IDENTIFY(mac=T_k, dur=7) │
  │           ├──Unicast alle 500ms─────►│ T_k blinkt LED‑Ring weiß
  │           │                          │
  │ drückt    │                          │
  │ Push      │                          │
  │           │                          │ wechselt in Edit‑Modus
  │           │                          │ Werte aus DISCOVER_REPLY anzeigen
  │           │                          │
  │ ändert    │                          │
  │ Werte     │                          │
  │           │                          │
  │ "Speich." │ CFG_WRITE(new cfg)       │
  │           ├──Unicast──────────────────►
  │           │                          │ persistiere NVS
  │           │ CFG_ACK(status=OK)       │
  │           ◄──Unicast──────────────────┤
  │           │                          │
  │ OLED: "Gespeichert"                  │
```

### 6.3 Flow C: Schnell mal eine Station live testen

```
Tobias    Config‑Box                  Stationen
  │ "Sound  │                              │
  │ testen" │ DISCOVER_REQ(type=STATION)   │
  │         ├──────Broadcast────────────────►
  │         │ DISCOVER_REPLY×n             │
  │         ◄──Unicast×n──────────────────┤
  │         │                              │
  │ wählt   │                              │
  │ Station │                              │
  │ + Sound │                              │
  │         │ CFG_WRITE(test_play_sound)   │
  │         ├──Unicast──────────────────────► gewählte Station spielt Sound
```

**Entschieden 2026‑07‑08:** eigener `msg_type` **0x32 `CFG_TEST_SOUND`**
(Unicast an die Station, `payload[0]` = `sound_id`, wird nur abgespielt,
nicht persistiert). In Doc 12 § 3.5 eingetragen und in der
Config‑Box‑Firmware V0.1 so implementiert.

---

## 7. Identify‑Blink‑Mechanik

Schlüssel‑Mechanik der Geräte‑Auswahl. Funktioniert ohne Start/Stop:

- Config‑Box sendet alle **500 ms** ein `IDENTIFY(mac=X, dur=7)` Unicast
  an das aktuell markierte Gerät.
- Gerät empfängt → schaltet seine Status‑LED (Station) bzw. seinen
  LED‑Ring (Target) für **700 ms** auf weiß; nach 700 ms automatisch
  zurück in den vorherigen Zustand.
- Solange Config‑Box weiterhin alle 500 ms sendet, überlappen sich die
  700‑ms‑Fenster → kontinuierliches Leuchten/Blinken.
- Cursor wechselt → neue MAC bekommt `IDENTIFY` → alte hat nach 700 ms
  ihr Fenster zu Ende, verstummt von selbst.
- Menü verlassen → keine `IDENTIFY` mehr → alles geht aus.

**Vorteile gegenüber Start/Stop‑Protokoll:**

- Keine „hängende" Blink‑LED, wenn die Config‑Box abstürzt oder den
  Stecker zieht – Blinken hört nach 700 ms von selbst auf.
- Cursor‑Wechsel braucht keine Stop‑Nachricht an das vorige Gerät.
- Verlorene `IDENTIFY`‑Pakete (ESP‑NOW‑Fehler) sind unkritisch – das
  Blinken aussetzt eine Periode, kommt mit dem nächsten Paket wieder.

**Visuelle Codierung:**

| Gerät | Identify‑Animation | Begründung |
|---|---|---|
| Station | Status‑LED außen weiß, **schnelles Pulsen** (200 ms an, 200 ms aus innerhalb der 700 ms) | unterscheidet sich vom regulären Spielbetrieb (grün‑blink) |
| Target | LED‑Ring komplett weiß, ~30 % Helligkeit, 700 ms statisch | unterscheidet sich vom Hit‑Pattern (rote Welle) |

Falls der Config‑Box‑User über den Eintrag „nur kurz drüber" wischt
(Encoder dreht zu schnell): die 500‑ms‑Taktung sorgt dafür, dass nur
das aktuell markierte Gerät blinkt – nicht jedes, über das man
hinwegblättert.

---

## 8. State‑Machine Config‑Box

```
BOOT
 │
 ▼
INIT (OLED‑Splash, ESP‑NOW init, Broadcast‑Peer)
 │
 ▼
MENU_MAIN
 │
 ├──► MENU_STATIONS ──► STATION_LIST ──► STATION_EDIT ──► CFG_WRITE ──► back
 │
 ├──► STATION_SETUP_MODE (SETUP_BEGIN aktiv)
 │      │
 │      ├──(SETUP_TAKE empfangen)──► STATION_SETUP_DONE ──► back
 │      └──(Timeout 60 s)──► back
 │
 ├──► MENU_TARGETS ──► TARGET_LIST ──► TARGET_EDIT ──► CFG_WRITE ──► back
 │
 ├──► LIVE_MONITOR (HIT_REPORT‑Stream auf OLED)
 │
 ├──► WEB_UI_ON (SoftAP gestartet, IP + QR auf OLED)
 │
 └──► TOOLS (Sniffer, Firmware‑Info, OTA)
```

In den Zuständen `STATION_LIST` und `TARGET_LIST` läuft im Hintergrund
ein 500‑ms‑Timer, der `IDENTIFY` an den Cursor‑Eintrag sendet (sofern
Identify nicht via K3 deaktiviert wurde).

In `STATION_SETUP_MODE` läuft ein 5‑s‑Timer, der `SETUP_BEGIN`
nachschickt – damit eine Station, die etwas später eingeschaltet wird,
auch noch in den Setup‑Modus geht.

---

## 9. Web‑UI (SoftAP)

### 9.1 Aufruf

- Im Hauptmenü „Web‑UI" → Config‑Box öffnet SoftAP mit fester SSID
  `Infinitag-Config`, WPA2, Passwort `infinitag` (konstant, im Code).
- IP‑Adresse fix `192.168.4.1`.
- OLED zeigt SSID + Passwort + IP, optional als QR‑Code (Lib `QRCode`
  von Richard Moore, klein).
- Solange Web‑UI aktiv, kein paralleles ESP‑NOW‑Senden (Kanal‑Konflikt
  vermeiden) – stattdessen pollt der Webserver bei jeder Request einen
  frischen Discovery‑Zyklus.

### 9.2 Seiten

| Pfad | Inhalt |
|---|---|
| `/` | Übersichts‑Tabelle: alle Stationen + Targets, Online/Offline, ID, MAC, RSSI, „Edit"‑Button pro Zeile |
| `/edit?mac=…` | Edit‑Form für das Gerät, abhängig von `device_type` |
| `/sounds` | Liste der Sound‑IDs ↔ Klartextnamen (statisch in der Config‑Box gepflegt; siehe 9.3) |
| `/about` | Firmware‑Info, freier Heap, MAC der Config‑Box |
| `/api/discover` | JSON‑Endpoint: triggert Discovery, gibt Liste zurück (für JS‑Refresh) |
| `/api/write` | POST mit JSON `{mac, cfg}` → schickt `CFG_WRITE`, wartet `CFG_ACK` |

### 9.3 Sound‑Klartext‑Liste

Die Klartextnamen der Sounds (`06 = Schrei kurz`, `13 = Halloween‑Trommel`)
sind **nur auf der Config‑Box** gepflegt, nicht im Target/Station‑NVS. Pflege
über `data/sounds.json` im LittleFS der Config‑Box, wird per `pio run -t
uploadfs` reingeschoben. Inhalt z. B.:

```json
{
  "1": "Geist",
  "2": "Werwolf",
  "3": "Hexe",
  "6": "Schrei kurz",
  "13": "Trommel"
}
```

Wenn das Target im NVS Sound 6 stehen hat, zeigt die Config‑Box „06 –
Schrei kurz" an. Wenn `sounds.json` einen anderen Mapping‑Stand hat als
die Sound‑Dateien auf der Station, ist die Klartext‑Anzeige falsch – aber
das Spiel selbst funktioniert weiter (es zählt nur die Nummer im Paket).
**Pflege‑Disziplin**: nach jedem neuen Sound‑Upload zur Station auch
`sounds.json` auf der Config‑Box mit aktualisieren.

### 9.4 Sicherheit

Wieder: Halloween‑Vorgarten, kein Reverse‑Engineering‑Risiko. Aber:

- SoftAP hat ein festes WPA2‑Passwort, damit nicht jeder Nachbar
  draufkommt.
- Webserver hat keine Auth – wer im SoftAP ist, darf alles. Reicht.
- Web‑UI ist im Hauptmenü explizit ein‑/ausschaltbar, nicht permanent
  aktiv. Damit kollidiert sie nicht mit dem ESP‑NOW‑Verkehr im
  Hauptbetrieb.

---

## 10. IR‑Pointer (latent, nicht in V1)

Im ursprünglichen Konzept war ein **IR‑Pointer** angedacht: Config‑Box
hat eine IR‑LED vorn, man richtet sie auf ein Target, drückt „Select",
Target meldet sich. Tobias hat das am 2026‑05‑18 zugunsten des Encoder‑
+ Discovery‑Wegs verworfen. Begründung: das visuelle Identify‑Blink
löst das gleiche Problem ohne zusätzliche Bauteile + ohne IR‑Sichtkontakt.

**Was bleibt latent:**

- Im Protokoll ist `msg_type = 0xC0 IR_SELECT_ECHO` reserviert (siehe
  Doc 12 § 3.5).
- Der GPIO7 in Abschnitt 4.3 ist als „IR‑LED‑Out" beschriftet, aber
  unbeschaltet.
- Im Target‑Code kann ein Hook gepflegt werden: bei `IR.isSystem=true,
  cmd=0xC0` ein `IR_SELECT_ECHO`‑Broadcast senden. Das sind 5 Zeilen
  Code, die niemandem weh tun und die Tür auflassen.

Falls in einer späteren Saison ein Target verbaut ist und sich nicht
mehr per Liste eindeutig identifizieren lässt (z. B. drei baugleiche
in einer Halloween‑Prop ohne sichtbaren LED‑Ring): IR‑Pointer
nachrüsten = IR‑LED + 2N7000/AO3400 + 100 Ω auf den GPIO7 löten, im
Code ein Menü „Tools → IR‑Pointer" aktivieren. Aufwand: ~ein Abend.

---

## 11. Mechanik / Gehäuse (Konzept)

```
Vorderansicht (~80 × 60 mm)
┌────────────────────────────────┐
│  ┌────────────────┐            │
│  │  OLED 0,96"    │     ⊙      │
│  │  128 × 64      │  (Encoder) │
│  └────────────────┘            │
│   K1   K2   K3   K4            │
│   ●    ●    ●    ●             │
└────────────────────────────────┘

Rückseite / Boden:
   ─ USB‑C (Service) ─ Schiebeschalter ─ Batteriefach 4×AA ─
```

- **OLED‑Fenster**: rechteckiger Ausschnitt 30 × 14 mm, OLED‑PCB klemmt
  innen gegen Frontplatte mit 4 M2‑Inserts.
- **Encoder**: M7‑Gewinde der Welle durch Frontplatte gesteckt, mit der
  mitgelieferten Mutter verschraubt; Knopf außen aufgesteckt.
- **4 Taster**: das DST‑015‑0‑Modul hat onboard‑Drucktaster. Frontplatte
  bekommt **vier zylindrische Aussparungen Ø 5 mm** über den
  Taster‑Stempeln. Optional Plunger‑Käppchen drucken (PETG natur),
  damit man die Taster außen erreicht.
- **USB‑C‑Buchse**: seitlich/hinten ausgespart (nur Service/Flashen),
  C3 Super Mini entsprechend ausgerichtet montiert.
- **Batteriefach**: 4×AA‑Halter im Gehäuseboden, Deckel geschraubt oder
  als Schiebedeckel; Schiebeschalter daneben von außen erreichbar.
- **Antenne**: SMA‑Buchse durchs Gehäuse oder Antenne intern an der
  Innenseite des Rückgehäuses fixiert, IPEX‑Pigtail zum Modul.

CAD wird im selben Stil wie das Stations‑Gehäuse gehalten, damit beide
Geräte zusammen aussehen. Detail‑Konstruktion erst nach erfolgreichem
Lochraster.

---

## 12. Lebende Diskussionspunkte / nächste Themen

- [ ] **Bestellen** (2026‑07‑08 rausgesucht): C3 Super Mini mit Antenne,
      SSD1315‑OLED mit Tasten, KY‑040‑Pack, 4×AA‑Halter‑Pack.
- [ ] **Lochraster aufbauen**: C3 Super Mini + OLED + KY‑040 + VBAT‑Teiler,
      Encoder per Interrupt‑Lib (kein PCNT auf C3!) testen. Verdrahtung
      und Bring‑up‑Plan: [`20-configbox-steckbrett.md`](20-configbox-steckbrett.md).
- [x] **ESP‑NOW‑Skeleton** auf der Config‑Box: als Teil der Firmware V0.1
      geschrieben (2026‑07‑08, `firmware/config-box/`). Noch offen: gegen
      einen zweiten ESP testen, der die Station‑Firmware in Stub‑Form spielt.
- [ ] **Target‑Firmware um ESP‑NOW erweitern**: `WiFi.mode(STA)` +
      `esp_now_init()` parallel zum WiFiManager‑Captive‑Portal (V1‑Kompromiss),
      dann WiFiManager raus (V2). Siehe neuer Punkt in
      [`11-offene-punkte.md`](11-offene-punkte.md).
- [ ] **Cooldown‑Feld am Target einführen**: heute fehlt der Cooldown,
      siehe [`08-software-target.md`](08-software-target.md) Z. 70 ff.
      Felder `cooldown_ms` und `sw_channels` in NVS und im Target‑Code
      ergänzen (vor dem Halloween 2026 fertig haben).
- [x] **Sound‑Test‑Befehl**: entschieden 2026‑07‑08 als eigener
      `msg_type` 0x32 `CFG_TEST_SOUND` (siehe Abschnitt 6.3 und Doc 12 § 3.5).
- [x] **OLED‑Menü‑Code**: als kleine State‑Machine im Hauptloop umgesetzt
      (`UiController` in Firmware V0.1), wie tendenziell geplant.
- [ ] **Web‑UI minimaler erster Wurf**: HTML‑Fragmente statisch, JS
      mit `fetch('/api/discover')`. Bootstrap o. ä. weglassen, das wird
      sonst eine Bibliotheks‑Schlacht für nichts.
- [ ] **Encoder‑Entprellung** im Lochraster verifizieren – falls Sprünge,
      RC‑Glied 10 kΩ / 100 nF an A/B nachrüsten.
- [ ] **3D‑CAD** Gehäuse, erst nach Lochraster wenn klar ist, welche
      Maße final sind.

---

## 13. Verweise

- [`12-refactor-station-v2.md`](12-refactor-station-v2.md) – Station V2,
  insbesondere § 3 (ESP‑NOW‑Protokoll, Paket‑Format) und § 7 (GPIO‑Plan
  als Vorbild)
- [`13-refactor-wand-v3.md`](13-refactor-wand-v3.md) – Wand V3, relevant
  für den Trigger‑Bestätigungs‑Flow (Flow A in Abschnitt 6.1)
- [`08-software-target.md`](08-software-target.md) – heutige
  Target‑Firmware (WiFiManager, fehlender Cooldown)
- [`11-offene-punkte.md`](11-offene-punkte.md) – Punkte zur Cooldown‑
  Einführung und WiFiManager‑Ablösung
- ESP‑NOW API:
  - [Espressif ESP‑NOW Guide](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/api-reference/network/esp_now.html)
- Rotary‑Encoder‑Library:
  - [igorantolic/AiEsp32RotaryEncoder](https://github.com/igorantolic/ai-esp32-rotary-encoder) –
    Interrupt‑basiert, läuft auf dem C3. (`ESP32Encoder` von madhephaestus
    nutzt PCNT und scheidet auf dem C3 aus.)
- OLED‑Library:
  - [olikraus/U8g2](https://github.com/olikraus/u8g2) – wie an der Station
