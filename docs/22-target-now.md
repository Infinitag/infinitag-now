# 22 – Target-Firmware „Infinitag Now" (`infinitag-now-target`)

**Status:** Lebendes Dokument. FW 0.1.0 erstellt am 2026-07-24, Build
grün, **auf Hardware ungetestet**. Löst die alte WLAN/HTTP-Firmware
(Doc 08, `PlatformIo/Projects/Infinitag Target`) ab.

Repo: `github.com/Infinitag/infinitag-now-target`, Arbeitskopie in
`/Volumes/Basteln/Infinitag/repos/infinitag-now-target/`.

## 1. Was sich gegenüber der alten Firmware ändert

| Thema | Alt (Doc 08) | Neu (Infinitag Now) |
| --- | --- | --- |
| Funk | WLAN via WiFiManager, HTTP-GET an die Wand | ESP-NOW v0x02, `HIT_REPORT`-Broadcast |
| Treffer-Meldung | `http://<prefix><block>/trigger_effect?sound=N` (hartcodiert auf `192.168.178.154`, Punkt 2 der offenen Punkte – damit erledigt) | `HIT_REPORT` mit `station_mac` + `sound_id` aus der NVS-Config |
| IR-Erkennung | IRremote + `Infinitag_Core::irDecode` (24-Bit-Telegramm) | **Burst-Erkennung** per GPIO-Interrupt, Pulsfenster 2,5–9 ms (Punkt 41, entschieden) |
| Konfiguration | Captive-Portal (System-IR-Hit öffnet WiFiManager) | Config-Box per Funk: `CFG_WRITE` → NVS, Discovery/Identify |
| Firmware-Update | WiFiManager-Update-Menü | `UPDATE_BEGIN` → SoftAP `infinitag-tgt-<MACSUFFIX>` (`WebUpdateService`, zwei OTA-Slots) |
| Architektur | `loopAnimation`-Task auf Core 1 + `loop()` auf Core 0 | Alles non-blocking in `loop()`, kein zweiter Task |
| Libraries | IRremote, WiFiManager, arduino-timer, HTTPClient, Infinitag_Core | nur noch Adafruit NeoPixel + `infinitag-now-core` |

Unverändert: GPIO-Belegung (V3.2-PCB, Doc 04), LED-Animationen
(Rainbow/rote Treffer-Welle), die zwei Switch-Patterns.

## 2. Hardware-Belegung (Target-V3.2-PCB, Doc 04)

| Funktion | GPIO | Anmerkung |
| --- | --- | --- |
| TSOP4138 IR-Empfänger | 15 | OUT active-LOW, `INPUT_PULLUP`; **Pin-Annahme aus altem Code – beim Bring-up verifizieren (Doc 04)** |
| LED-Ring 12× SK6812 RGBW | 38 | `NEO_GRBW`, Brightness 255 |
| SW1 potentialfrei (Optokoppler) | 21 | `sw_channels` bit0 |
| SW-5V (NPN) | 46 | `sw_channels` bit1 |
| SW-3.3V (NPN) | 48 | `sw_channels` bit2 |

ESP32-S3-WROOM-1 **N8R8** → `board_upload.flash_size = 8MB`, eigene
`partitions_8MB.csv` mit zwei 3,3-MB-OTA-Slots (app0/app1) + spiffs.

## 3. IR-Burst-Erkennung (Punkt 41)

Die Station schießt einen rohen 38-kHz-Burst von 5 ms (`IR_SHOT_MS`).
Der TSOP zieht seinen Ausgang für etwa diese Dauer auf LOW. Die ISR
(`CHANGE`-Interrupt) misst die Low-Zeit und übergibt Pulse im Fenster
**`BURST_MIN_US` 2500 µs … `BURST_MAX_US` 9000 µs** an `loop()`.

- Fernbedienungen: viele kurze Pulse (< 2 ms) → fallen unten raus.
- Dauer-Störer (Sonne, LED-Flackern): selten ein sauberer 3–9-ms-Puls.
- Ein Fehltrigger kostet nur einen Sound – für die Schießbude okay.
- 24-Bit-Telegramm bleibt spätere Option (siehe Punkt 41).

## 4. Zustandsmaschine

```
        IR-Puls im Fenster
ARMED ───────────────────────► HIT ────────────► COOLDOWN ────► ARMED
(Rainbow,                      (hit_time_ms:     (cooldown_ms:  (wieder
 10-ms-Schritt)                 rote Welle,       gedimmtes      scharf)
                                Switch-Pattern,   Blau-Pulsen,
                                80-ms-Schritt)    Treffer ignor.)
```

- `HIT_REPORT` geht **sofort** beim Übergang nach HIT raus (vor LED/
  Switch-Start) – Latenzbudget < 50 ms bis Sound-Start (Doc 12 § 3.7).
- `station_mac` = 0 (ungesetzt): Treffer läuft lokal (LEDs + Switches),
  aber kein Funk („Treffer verpuffen", PROTOCOL.md).
- `cooldown_ms = 0`: direkt zurück nach ARMED.
- Während HIT/COOLDOWN werden weitere IR-Pulse ignoriert (ersetzt die
  alte `hit`-Sperre).
- IDENTIFY der Config-Box überlagert jede Animation mit weißem
  200-ms-Blinken (selbstlöschendes Fenster).

## 5. Switch-Patterns (aus der alten Firmware portiert)

| Pattern (`sw_animation`) | Ablauf |
| --- | --- |
| 0 | An für die komplette `hit_time_ms` (erster Step wird bei jedem Treffer damit überschrieben) |
| 1 | Drei Impulse: 700 an / 200 aus / 200 an / 200 aus / 200 an / aus |

Neu gegenüber alt: `sw_channels` (Bitmaske) schaltet nur die
freigegebenen Ausgänge – die alte Firmware schaltete immer alle drei
synchron. Default 0b111 = Verhalten wie früher.

## 6. ESP-NOW-Gerätelogik (`NowTarget`, Protokoll v0x02)

| Message | Verhalten des Targets |
| --- | --- |
| `DISCOVER_REQ` (Filter `DEV_TARGET`/`DEV_ANY`) | `DISCOVER_REPLY` per Unicast: FW-Version, uptime_min, Target-Config-Blob (16 Byte) |
| `IDENTIFY` | Weißes Blinken für payload[0] × 100 ms |
| `CFG_WRITE` (Blob) | Validieren → NVS speichern → `CFG_ACK` (OK / NACK-Validierung) |
| `UPDATE_BEGIN` | `UPDATE_ACK`, dann blockierender SoftAP-Update-Modus (`infinitag-tgt-<MACSUFFIX>`, Datei-Präfix `infinitag-target`), endet immer in `ESP.restart()` |
| `DEBUG_CMD` Test 2 (DBG_LED) | Ring-Testmuster R/G/B/W, `DBG_RES_OK` |
| `DEBUG_CMD` Test 5 (DBG_TRIGGER) | **Umgedeutet als IR-Empfangs-Test:** nächster erkannter Burst innerhalb des Timeouts → `DBG_RES_OK` (grüner Blitz, KEINE Hit-Aktion – zum Einschießen), sonst `DBG_RES_TIMEOUT` |
| `DEBUG_CMD` sonstige (Sound/Laser/IR-Senden/Kalibrierung) | `DBG_RES_UNSUPPORTED` (Target hat weder Sound noch Sender) |
| `PUSH_BEGIN`/`PUSH_END` | Noch ignoriert – Funk-Push-Update ist Etappe 4 (Doc 21 § 5), nächster geplanter Schritt |

Validierungsgrenzen bei `CFG_WRITE` (an die Config-Box-Ranges
angelehnt): `sound_id` < SOUND_COUNT, `hit_time_ms` 100–60000,
`cooldown_ms` ≤ 60000, `sw_animation` ≤ 1, `sw_channels` ≤ 0b111.

NVS-Namespace `inow-target`: `stamac`, `sound`, `hittime`, `cooldn`,
`swani`, `swchan`.

## 7. Repo-Aufbau (Muster von `-station`/`-config`)

```
infinitag-now-target/
├── platformio.ini        # espressif32@^6.7.0, 8 MB, Core-Symlink
├── partitions_8MB.csv    # 2× OTA-Slot à 0x330000
├── release.sh            # baut, taggt vX.Y.Z, GitHub-Release + .crc32
├── src/
│   ├── main.cpp          # Hardware, ISR, Zustandsmaschine, Update-Modus
│   ├── NowTarget.{h,cpp} # Protokoll-Handler; TARGET_FW_MAJOR/MINOR/PATCH
│   └── TargetSettings.{h,cpp}  # NVS
├── .github/              # PR-Template + release.yml (Label-Gruppierung)
├── CLAUDE.md / README.md / LICENSE (PolyForm NC 1.0.0)
```

`INOW_FW_MARKER(inow::DEV_TARGET, …)` ist in `main.cpp` eingebettet
(Doc 21 E1, Image-Store der Box erkennt Gerätetyp + Version).

## 8. Erster Hardware-Test (Checkliste)

1. `pio run -t upload` per USB (erstes Flashen der OTA-FW immer per
   Kabel!). Boot-Log prüfen: MAC, geladene Config, „TSOP an GPIO15".
2. **GPIO15-Annahme verifizieren** (Doc 04): Fernbedienung auf das
   Target → im Log dürfen KEINE Bursts erscheinen (zu kurz), aber der
   Roh-Pegel muss sich bewegen (notfalls ISR-Log kurz erweitern).
3. Config-Box: Discovery → Target erscheint mit MAC-Suffix →
   Identify-Blinken prüfen.
4. Edit-Menü: Station zuweisen, Sound wählen → Speichern → `CFG_ACK`
   OK? Reboot → Config noch da (NVS)?
5. Station schießen: Burst-Log am Target, Sound an der Station,
   rote Welle + Switch-Pattern, danach Cooldown → Rainbow.
6. Selbsttests über die Box: DBG_LED, DBG_TRIGGER (IR-Empfangs-Test).
7. Update (OTA): SoftAP `infinitag-tgt-…` → neue .bin hochladen →
   Version im Discovery-Vergleich der Box prüfen.

## 9. Offene Punkte / bewusste Entscheidungen

- **Cooldown-Semantik:** COOLDOWN startet NACH Ablauf von
  `hit_time_ms` (Gesamt-Sperrzeit = hit_time + cooldown). PROTOCOL.md
  definiert das nicht explizit – so umgesetzt, bei Bedarf im Core
  nachdokumentieren.
- DBG_TRIGGER-Umdeutung als IR-Empfangs-Test ist Target-lokal; die
  Box zeigt weiterhin ihren generischen Trigger-Test-Text. Ggf.
  Box-UI-Text für Targets anpassen.
- Externer LED-Ausgang (GPIO 47, Doc 04) weiterhin ungenutzt.
- Funk-Push-Update (EspNowPushReceiver) fehlt noch – Etappe 4.
