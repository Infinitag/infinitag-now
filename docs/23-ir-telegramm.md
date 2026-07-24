# 23 – IR-Schuss-Telegramm (Protokoll v0x03)

**Status:** Beschlossen 2026-07-24 (revidiert Punkt 41 aus Doc 11).
Implementiert in `infinitag-now-core` (`src/IrTelegram.*`, nativ
unit-getestet), Station (Sender) und Target (Empfänger). **Auf Hardware
ungetestet.** Verbindliche Kurzspezifikation zusätzlich in
`PROTOCOL.md` (Core-Repo) – bei Abweichungen gilt `PROTOCOL.md`.

## 1. Warum ein Telegramm? (Revision von Punkt 41)

Punkt 41 hatte am selben Tag zunächst **Burst-Erkennung** entschieden
(roher 5-ms-Burst, Pulsfenster 2,5–9 ms). Beim Zielbild-Abgleich fiel
die Entscheidung: **~15 Targets, 3+ Stationen, jeder Zauberstab schießt
auf jedes Ziel.** Ein roher Burst transportiert null Information – das
Target weiß nicht, *wer* geschossen hat, und der Sound kann nicht zur
richtigen Station geroutet werden. Die Zwischenlösung (statische
Target→Station-Zuordnung per `station_mac`) skaliert nicht auf dieses
Zielbild und hatte in der Praxis Konfigurations-Fallen (Null-MAC-Bug
der Config-Box, Discovery-Reihenfolge).

Abgewogene Alternativen:

| Option | Bewertung |
| --- | --- |
| **A – Burst-Längen-Slots** (3,5/5,5/7,5 ms je Station) | Minimal-invasiv, aber max. 4–5 unterscheidbare Stationen und keine weiteren Nutzdaten (Schaden, spätere Optionen) |
| **B – Zeitkorrelation** (Station spielt, wenn sie selbst < 100 ms zuvor geschossen hat) | Kein IR-Umbau, aber probabilistisch: gleichzeitige Schüsse → Fehlzuordnung genau dann, wenn viel los ist |
| **C – Datentelegramm** (gewählt) | Beliebig erweiterbar (Schützen-ID heute, Schaden/Hitpoints morgen), CRC-gesichert = störlichtfest; Preis: ~25 ms längere Sendezeit + Decoder-Komplexität |

Entscheidung Tobias, 2026-07-24: **Option C**, ausdrücklich wegen der
Erweiterbarkeit (spätere Optionen wie Schaden für Hitpoint-Targets
müssen im Schuss mitfahren können).

## 2. Physikalische Schicht

38-kHz-Träger (TX ~33 % Duty am TSAL6200, RX TSOP4138),
**Puls-Distanz-Codierung** (Information in der Space-Länge, wie NEC).
Alle Zeiten in µs:

| Element | Mark (Träger an) | Space (Träger aus) |
| --- | --- | --- |
| Header | 2400 | 600 |
| Bit „0" | 600 | 600 |
| Bit „1" | 600 | 1200 |
| Stop-Mark | 600 | – |

Empfänger-Toleranzen (`IrTelegram.h`): Header-Mark 1800–3200,
Bit-Mark 250–1000, Space < 900 = „0", 900–1700 = „1", darüber Reset.
Die Bänder decken die Pulsverformung des TSOP (AGC) von ±ein paar
hundert µs ab; unit-getestet bis ±200 µs Skew.

## 3. Frame-Format (16 Bit, MSB zuerst)

| Bits | Feld | Bedeutung |
| --- | --- | --- |
| 15–12 | `irt_version` | `0x1` – eigene Versionierung der IR-Schicht |
| 11–8 | `shooter_id` | `ir_id` der schießenden Station (0–15) |
| 7–4 | `damage` | Schadenswert; Sender clampt auf ≥ 1 (Reserve für Hitpoint-Targets) |
| 3–0 | `crc4` | CRC-4, Poly x⁴+x+1, Init 0, über Bits 15–4 |

- Fernbedienungen (NEC: 9-ms-Header, ~560-µs-Pulse) und der alte
  v0x02-Burst (5 ms) erzeugen keine gültigen Frames.
- Kollision (zwei Stationen treffen dasselbe Target überlappend):
  CRC schlägt fehl → kein Treffer; Kind schießt erneut. Bewusst so.
- `ir_id` 0 ist **gültig** („Werksgruppe"): ab Werk spielen alle
  Stationen gegenseitig ihre Treffer – Einzelplatz-Aufbau läuft ohne
  Konfiguration. Mehrere Stationen → eindeutige IDs 1–15 per
  Config-Box.

## 4. Latenzbudget (Ziel < 50 ms IR → Sound, Doc 12 § 3.7)

| Posten | Dauer |
| --- | --- |
| Telegramm (16 Bit) | 22,8 ms (alle 0) … 32,4 ms (alle 1) |
| Decode (letzte Flanke → `loop()`) | < 3 ms |
| `HIT_REPORT` (ESP-NOW, Broadcast) | 1–2 ms |
| Sound-Start Station | < 10 ms |
| **Summe (worst case)** | **≈ 45 ms** ✓ |

Der Schuss fühlt sich damit minimal träger an als der 5-ms-Burst
(~30 ms mehr bis zum Sound) – beim Hardware-Test bewusst gegenprüfen.

## 5. Routing-Kette (v0x03)

```
Station (ir_id=N) ──IR: [v1|N|dmg|crc]──▶ Target
                                            │ decode + CRC ok
                                            ▼
                        HIT_REPORT (Broadcast): shooter=N, sound, dmg
                                            │
        ┌───────────────────┬───────────────┴──────────────┐
        ▼                   ▼                              ▼
  Station ir_id=N      Station ir_id=M (≠N)         Config-Box
  → spielt sound       → ignoriert                  → Live-Monitor
```

- Sender: `sendIrTelegram()` in der Station (blockierend, LEDC-Gating
  nach der Timing-Tabelle aus `irtTimings()`).
- Empfänger: Target-ISR misst Mark/Space-Phasen → `IrtDecoder::feed()`.
- Der DBG_IR-Selbsttest der Station sendet weiterhin einen rohen Burst
  (reiner Hardware-Test, TSOP-Selbstempfang) – kein gültiges Telegramm.

## 6. Betroffene Komponenten / PRs (2026-07-24)

| Repo | Änderung | PR |
| --- | --- | --- |
| `infinitag-now-core` | `IrTelegram.*`, Protokoll v0x03 (HIT_REPORT, Blobs), PROTOCOL.md, 9 neue Tests | #13 |
| `infinitag-now-station` | `sendIrTelegram()`, `ir_id` in Settings/Blob, HIT_REPORT-Filter auf ir_id | #22 |
| `infinitag-now-target` | ISR → `IrtDecoder`, HIT_REPORT neu, `stationMac` entfernt | #5 |
| `infinitag-now-config` | Stations-Editor + IR-ID, Target-Editor ohne Station-Picker, Live-Monitor | #26 |

**Migration = Flag-Day:** v0x03-Pakete werden von v0x02-Geräten am
Versions-Byte verworfen (und umgekehrt), der alte Burst triggert den
Decoder nicht. Funk-Reihenfolge: **Geräte zuerst, Box zuletzt** – eine
bereits aktualisierte Box erreicht die alten Geräte nicht mehr (genau
das ist beim Umstieg am 2026-07-24 passiert; Ausweg war ein
Box-Downgrade per lokalem SoftAP-Update).

**Rettungsanker (seit Core v3.1.0, 2026-07-24):** Als Konsequenz sind
`DISCOVER_REQ`/`DISCOVER_REPLY` und `UPDATE_BEGIN`/`UPDATE_ACK`
**versionstolerant** – eine neuere Box findet Geräte jeder
Protokollversion (Listen-Marker `!`) und kann sie in den
SoftAP-Update-Modus schicken; alles andere (Config, Treffer, Push)
bleibt versionsgesperrt. Der eingefrorene Vertrag (Header, CRC,
Payload-Semantik der vier Nachrichten) steht in `PROTOCOL.md`.
Künftige Protokollbrüche brauchen damit weder USB noch eine
Update-Reihenfolge. Gilt ab den Firmwares Station 0.5.1 /
Config-Box 0.4.1 / Target 0.1.1; ältere Stände bleiben rückwirkend
nur per USB bzw. Box-Downgrade erreichbar.

## 7. Offen / später

- **Hitpoints:** Target-seitiger HP-Zähler, `damage` wird abgezogen,
  HIT erst bei 0. Feld fährt ab Tag 1 mit; Verhalten kommt später
  (Target-Blob-Reserve Bytes 7–15).
- Mehr als 15 Stationen oder weitere Felder → `irt_version` 0x2 mit
  längerem Frame (Bit-Budget-Erweiterung, gleiche Codierung).
- Hardware-Verifikation Reichweite/Streiftreffer: Telegramm ist länger
  als der Burst – Trefferquote bei schnellen Schwenks beobachten
  (Bring-up-Checkliste Doc 22 § 8).
