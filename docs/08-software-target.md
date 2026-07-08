# 08 – Software: Target (PlatformIO‑Projekt)

Quelle: `/Volumes/basteln/PlatformIo/Projects/Infinitag Target/`.

PlatformIO‑Konfiguration (`platformio.ini`):

```ini
[env:esp32-s3-devkitc-1]
platform = espressif32
board = esp32-s3-devkitc-1
framework = arduino
lib_deps =
    z3t0/IRremote@^4.2.0
    adafruit/Adafruit NeoPixel@^1.12.0
    wnatth3/WiFiManager@^2.0.16-rc.2
    contrem/arduino-timer@^3.0.1
```

`Infinitag_Core` ist als lokale Lib unter `lib/Infinitag_Core` eingecheckt
(eigenes Git‑Repo).

## Kernfunktionen

| Funktion | Aufgabe |
| --- | --- |
| `setup()` | Animation‑Loop starten, Config laden, LEDs/WiFi/HTTP/Switches/IR initialisieren |
| `loop()` | nichts außer `checkIrData()` aufrufen |
| `loopAnimation()` | läuft auf separatem Core (`xTaskCreatePinnedToCore`, Core 1) und kümmert sich um LED‑Animation, Switch‑Animation und Timer‑Ticks |
| `checkIrData()` | Decodiert eingehenden IR‑Frame, ruft je nach `cmd`/`isSystem` `hitAction()` oder `openConfigPortalAction()` |
| `hitAction()` | Setzt Hit‑State, startet Reaktivierungs‑Timer, sendet HTTP‑GET an die Wand |
| `enableTarget()` / `targetHit()` | Wechseln zwischen Idle und Hit‑Modus, schalten LED‑Animation und Switches um |
| `saveWifiParamsCallback()` | Persistiert Custom‑Params (Sound‑ID, Hit‑Time, SW‑Animation, IP‑Prefix) in `Preferences` |

## GPIO‑Belegung im Code

Dieselbe Tabelle wie in [`04-hardware-target.md`](04-hardware-target.md) –
hier nochmal aus Code‑Sicht:

```cpp
#define LED_PIN     38
#define LED_COUNT   12
#define BRIGHTNESS 255

#define IR_PIN      15

#define SW_PIN      21      // potentialfrei (Optokoppler)
#define SW_5V_PIN   46      // 5 V Switch
#define SW_33V_PIN  48      // 3,3 V Switch
```

## Konfiguration via Captive‑Portal

Der WiFiManager wird mit vier Custom‑Parametern erweitert:

| Parameter | Default | Bemerkung |
| --- | --- | --- |
| `sound_id` | 1 | Index des Sounds, den der Sound‑ESP der Station spielt |
| `hit_time` | 10000 (ms) | Wie lange das Target nach einem Treffer „getroffen“ bleibt – steuert auch die Switch‑Animation |
| `sw_animation` | 0 | Pattern‑Index für die Schaltausgänge |
| `ip_prefix` | `192.168.0.` | Erste 3 Oktette der Wand‑Station‑IP |

Persistenz: ESP‑`Preferences`, Namespace `target-data`, Schlüssel `soundid`,
`hittime`, `swani`, `ipprefix`.

Zum Öffnen des Captive‑Portals reicht ein „System‑Hit“ per IR
(`isSystem=true, cmd=1`) – z. B. mit dem `InfinitagTargetResetter` Sketch
auf einem zweiten ESP. Sehr praktisch im Aufbau.

## Switch‑Animationen

Im Code sind zwei Pattern hardcoded:

| Index | Pattern | Bedeutung |
| --- | --- | --- |
| 0 | `[on, off]` mit Zeiten `[hitTime, 0]` ms | Während der gesamten Hit‑Time an, dann aus |
| 1 | `[on, off, on, off, on, off]` mit Zeiten `[700, 200, 200, 200, 200, 0]` ms | Drei kurze Blink‑Impulse |

`sw_pattern_times[0][0]` wird beim Laden mit `hitTime` überschrieben, sodass
Pattern 0 immer die volle Hit‑Time abdeckt.

Alle drei Schaltausgänge (`SW`, `SW_5V`, `SW_3.3V`) werden synchron
geschaltet – sie haben kein eigenes Pattern. Wenn man später unterschiedliche
Effekte fahren will (z. B. potentialfrei = Pumpe, 5 V = LEDs, 3,3 V = Sensor),
müsste man die `setSw()`-Funktion auf drei einzelne Channels splitten.

## LED‑Animation

- **Idle** (`rainbow`): klassisches `strip.rainbow(animationStep)` mit
  Schrittweite 256, alle 10 ms.
- **Hit** (`animationHit`): Eine rote „Welle“, die sich um den Ring dreht, mit
  abnehmender Maximalhelligkeit über die Zeit. Sehr schöner Cooldown‑Effekt.

## HTTP‑Client für Treffer‑Meldung

```cpp
String serverPath = "http://";
serverPath.concat(ipPrefix);            // z. B. "192.168.0."
serverPath.concat(ipBlock);             // letztes Oktett aus IR cmdValue
serverPath.concat("/trigger_effect?sound=");
serverPath.concat(soundId);

// ABER: tatsächlich wird hartcodiert aufgerufen:
http.begin("192.168.178.154", 80, "/trigger_effect?sound=6");
```

⚠️ **Bug / Debug‑Rest:** Im aktuellen `main.cpp` ist die `http.begin()` mit
festen Werten hartcodiert (`192.168.178.154`, Port 80, sound=6). Der korrekt
gebaute `serverPath` wird nur per `Serial.println(...)` ausgegeben. Vor dem
nächsten Halloween‑Einsatz muss die Zeile auskommentiert und durch
`http.begin(serverPath.c_str())` ersetzt werden, sonst klingelt jedes Target
immer denselben Server mit Sound 6 an.

→ Eintrag in [`11-offene-punkte.md`](11-offene-punkte.md).

## Boot‑Verhalten

```cpp
void setup() {
  booting = true;
  Serial.begin(115200);
  setupAnimationLoop();   // erstellt Task auf Core 1
  loadConfig();
  setupLeds();
  setupWifi();            // blockierend bis WLAN steht oder Portal aktiv
  setupHttpClient();
  setupSwitches();
  setupIr();
  booting = false;
  congiure = false;
}
```

Solange `booting == true`, leuchtet der Ring gelblich (`Color(100,100,0,0)`).
Während der Captive‑Portal‑Öffnung wird `congiure = true` gesetzt – derselbe
gelbliche Zustand. Sehr klares visuelles Feedback ohne separaten Status‑LED.

## Hilfssketche

Im Arduino‑Ordner liegen drei Sketche, die nicht Teil der eigentlichen
Target‑Firmware sind, aber rund um Targets nützlich sind:

- `InfinitagTargetTest/InfinitagTargetTest.ino` – ältere, funktional fast
  identische Variante des Targets als Arduino‑Sketch (vor Umzug auf
  PlatformIO). Hilfreich als Referenz für ein einfaches Target ohne
  Switch‑Animation.
- `InfinitagTargetResetter/InfinitagTargetResetter.ino` – schickt einmalig
  einen System‑Hit (`isSystem=true, cmd=1`) per IR und beendet sich. Damit
  öffnet jedes Target in Sichtweite sein Captive‑Portal.
- `InfinitagTargetAdminTool/InfinitagTargetAdminTool.ino` – simpler
  Trigger/Setup‑Button‑Sketch (zwei Buttons → zwei IR‑Codes), nutzt aber
  ein **anderes** IR‑Encoding (eigene `irEncode()`‑Funktion mit
  IP‑Block‑basierten Bits, nicht das Infinitag‑Protokoll). Eher Demo/Workshop‑
  Stand. Nicht für den Halloween‑Betrieb verwenden – die Targets reagieren
  darauf nicht.
