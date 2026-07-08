# 06 – Software: Logic‑ESP der Station (`InfinitagWand.ino`)

Quelldatei: `/Volumes/basteln/Arduino/InfinitagWand/InfinitagWand.ino`.
Trotz des Sketch‑Namens ist das **nicht** der Wand selbst, sondern der
Logic‑ESP des Wand‑Station‑PCBs. Geschichtlich kommt der Name aus der Zeit,
als der Wand noch ein eigenständiger ESP war.

## Aufgabe

- Trigger‑Eingang (vom Wand‑Kabel) lesen.
- Beim Tastendruck: IR‑Schuss senden (RC5, 24 Bit, kodiert via `Infinitag_Core`),
  Laser für 1 s aktivieren, Trigger für 2 s sperren.
- NeoPixel‑Beleuchtung der Wand (zwei SK6812RGBW) ansteuern.
- WiFi‑Anbindung über `WiFiManager`.
- HTTP‑Webserver auf Port 8080: nimmt `/trigger_effect?sound=N` von den
  Targets entgegen.
- Empfangene Sound‑ID per I²C an den Sound‑ESP (Adresse `0x01`) durchreichen.

## Wichtige Konstanten

| Symbol | Wert | Bemerkung |
| --- | --- | --- |
| `LED_PIN` | 27 | NeoPixel Datenleitung (Net `LED_ESP` → Wand) |
| `LED_COUNT` | 2 | zwei SK6812RGBW im Wand‑Stab |
| `LED_BRIGHTNESS` | 255 | volle Helligkeit |
| `I2C_DEV_ADDR` | 1 | Slave‑Adresse Sound‑ESP |
| `I2C_SDA` | 18 | I²C SDA Pin am Logic‑ESP |
| `I2C_SCL` | 19 | I²C SCL Pin am Logic‑ESP |
| `TRIGGER_PIN` | 25 | Trigger‑Eingang (Net `Trigger_ESP` von der Wand) |
| `IR_SEND_PIN` | 4 | IR‑LED‑Treiber (Net `IR_ESP` zur Wand) |
| `LASER_PIN` | 14 | Laser‑Treiber (Net `Laser_ESP` zur Wand) |
| `wifiName` / `wifiPassword` | `InfinitagWandStation` / `password` | AP‑Name & PW im Captive‑Portal |
| `shootDelay` | 2000 ms | Cooldown zwischen Schüssen |
| `laserDelay` | 1000 ms | Maximale Laser‑An‑Zeit pro Schuss |

## Setup‑Reihenfolge

```cpp
Serial.begin(115200);
Wire.begin(I2C_SDA, I2C_SCL);
pinMode(TRIGGER_PIN, INPUT);
pinMode(LASER_PIN, OUTPUT);  digitalWrite(LASER_PIN, LOW);
IrSender.begin(IR_SEND_PIN);    // (oder im WHITE_FOCUS_LED‑Mode digitalWrite)
setupButton();                  // EasyButton on TRIGGER_PIN
setupLeds();                    // strip.begin(); strip.show();
setupWifi();                    // WiFiManager + WebServer auf Port 8080
resetShoot();                   // canShoot = true; LEDs grün/blau
```

## Hauptschleife

Sehr schlank:

```cpp
void loop() {
  triggerButton.read();   // Tastendruck pollen
  timer.tick();           // arduino-timer Callbacks abarbeiten
  server.handleClient();  // Webserver-Requests verarbeiten
}
```

Der Trigger ruft `shoot()` auf:

1. Wenn `canShoot == false` → return.
2. `canShoot = false`, Laser `HIGH`.
3. NeoPixel auf reines Weiß (W‑Channel) → optisches Feedback.
4. IR‑Frame berechnen: `irEncode(false, 0, 0, 0, 1, lastIpBlock)` und mit
   `IrSender.sendRC5(value, 24)` rausschicken.
5. Timer `resetShootTimer` in 2000 ms (entsperrt) und `resetLaserTimer` in
   1000 ms (Laser aus).

## Webserver‑Endpunkt

```cpp
server.on("/trigger_effect", triggerEffect);
```

`triggerEffect()` zieht den Query‑Parameter `sound`, parst ihn als `int`,
schickt ihn per `sendInt()` über I²C an Adresse `0x01` (Sound‑ESP) und
antwortet mit HTTP 200. Vor dem `server.send(200)` steht ein `delay(30)` –
Tobias dürfte das eingebaut haben, damit die I²C‑Übertragung verlässlich
durch ist; technisch ist das nicht zwingend nötig, weil
`Wire.endTransmission()` blockierend ist.

## NeoPixel‑Logik

`setColor()` setzt einfach alle LEDs auf eine Farbe. Drei Zustände:

| Zustand | Farbe | Wo |
| --- | --- | --- |
| WiFi‑Verbinden | Rot (`Color(255,0,0,0)`) | `setupWifi()` |
| Bereit | Blau (`Color(0,0,255,0)`) | `resetShoot()` |
| Schuss aktiv | Weiß (W‑Channel `Color(0,0,0,255)`) | `shoot()` |

## Bibliotheken

- `arduino-timer` (contrem)
- `Adafruit_NeoPixel`
- `EasyButton`
- `Ethernet` (Achtung: nur als Header eingebunden, aktiv genutzt wird `WiFi`)
- `Infinitag_Core` (siehe Lib‑Ordner)
- `IRremote.hpp` (z3t0/IRremote)
- `WiFiManager` (tzapu / wnatth3)
- `Wire` (I²C)

## Bekannte Stolperfallen / Notizen

- Die `Ethernet.h` wird inkludiert, aber nicht verwendet – sieht aus wie ein
  Lazy‑Include‑Erbe. Lässt sich gefahrlos entfernen und spart etwas Compile‑Zeit.
- `WHITE_FOCUS_LED_INSTALLED` ist eine alternative Build‑Variante: anstatt
  IR‑LED wird ein weißes Fokuslicht über `IR_SEND_PIN` digital geschaltet.
  Das ist nur sinnvoll bei Demo/Testaufbauten ohne IR – im Halloween‑Setup
  bleibt es auskommentiert.
- `wm.autoConnect()` ist **blockierend**: bei fehlendem WLAN bleibt der ESP
  im Captive‑Portal stehen, NeoPixel rot. Erst nach Verbindung läuft
  `loop()`.
- `triggerEffect()` macht keine Plausibilisierung des Sound‑ID‑Werts. Wenn
  ein Target aus Versehen `sound=999` schickt, wird das `999 mod 256` als
  Byte über I²C verschickt → potenziell undefiniert. Defensiver wäre eine
  Begrenzung auf `0..SOUND_EFFECT_SIZE‑1`.

## Datei‑Hash / Stand

Der Sketch ist mit Stand Mai 2026 inhaltlich identisch zu Anfang 2024 und
wurde laut Tobias zuletzt für das Halloween‑2024‑Setup angepasst. Bei
größeren Änderungen sollte in dieser Doku ein Versions‑Tag eingebaut werden.
