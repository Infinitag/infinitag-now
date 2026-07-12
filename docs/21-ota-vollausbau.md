# 21 – OTA-Vollausbau: Internet-Selbstversorgung + ESP-NOW-Verteilung

Konzept, festgehalten 2026-07-12. Baut auf dem SoftAP-Update-Modus
(Doc 18 § 6.1) und der Variante-B-Skizze („Update-Maultier", Doc 18 § 12)
auf und ersetzt beide Planungsstände als maßgebliches Zielbild.

## 1. Leitplanke (das eigentliche Ziel)

**Die Config-Box muss von jemandem ohne PC-Kenntnisse bedienbar sein.**
Tobias drückt einem Helfer die Box in die Hand und sagt „mach bitte alle
Geräte aktuell" – der Helfer braucht dafür weder PC noch Smartphone noch
GitHub-Wissen. PC-Momente sind nur bei der einmaligen Einrichtung durch
Tobias erlaubt (WLAN-Zugangsdaten hinterlegen, Partitions-Erstflash).

## 2. Ablauf aus Nutzersicht (Endzustand)

```
Helfer:  Tools → „Nach Updates suchen"
Box:     verbindet sich mit dem Heim-WLAN
         fragt GitHub nach den neuesten Releases (Box/Station/Target)
         zeigt: „Neu: Box v0.3.1 · Station v0.3.0 · Target v0.3.0"
Helfer:  „Alle aktualisieren" bestätigen
Box:     lädt Station-/Target-Images in ihren Dateispeicher
         aktualisiert SICH SELBST (Stream direkt in den OTA-Slot) → Reboot
         (Images überleben den Reboot im LittleFS)
Box:     zurück auf ESP-NOW, Discovery → Liste aller Geräte mit ^-Markern
Helfer:  läuft mit der Box in Funkreichweite der Geräte,
         „Alle aktualisieren" → Box schiebt die Images per ESP-NOW-Push
         nacheinander auf jedes veraltete Gerät (Fortschrittsbalken)
Box:     Abschlussbericht: „3 aktualisiert, 1 nicht erreicht (4B11FE)"
```

Kein AP-Wechsel, kein Browser, kein Dateidownload durch den Helfer.
Der heutige SoftAP-Weg (Doc 18 § 6.1) bleibt als Fallback und für die
Entwicklung erhalten.

## 3. Bausteine

### 3.1 WLAN-Provisioning (einmalig, durch Tobias)

- Neue Seite auf dem vorhandenen SoftAP-Webserver der Box:
  SSID + Passwort eingeben → NVS (`inow-config`, Keys `wifi_ssid`/`wifi_pass`).
- Kein Dauerbetrieb im WLAN: Die Box verbindet sich **nur im
  Update-Modus** mit dem Heim-WLAN und kehrt danach zu ESP-NOW zurück
  (Reboot, wie beim heutigen Update-Modus – sauberer Funk-Zustand).
- Reichweiten-Realität: „Nach Updates suchen" passiert dort, wo das
  Heim-WLAN trägt; der Verteil-Rundgang danach ist WLAN-unabhängig.

### 3.2 Update-Quelle: GitHub-Releases

- Quelle der Wahrheit bleiben die GitHub-Releases der drei Geräte-Repos
  (Assets `infinitag-<typ>-vX.Y.Z.bin`, seit v0.2.1 etabliert).
- Abruf: `GET api.github.com/repos/Infinitag/<repo>/releases/latest`
  (Tag = Version) → `browser_download_url` des `.bin`-Assets →
  HTTPS-Download (folgt Redirect auf `objects.githubusercontent.com`).
- **Voraussetzung: Repos öffentlich.** Private Assets verlangen einen
  Token – der müsste in die Box-Firmware und wäre damit de facto
  veröffentlicht. Deshalb Reihenfolge: erst Public-Checkliste
  (Doc 18 § 12: Sound-Historie bereinigen, Branch-Protection), dann
  dieser Ausbau. Rate-Limit (60 Requests/h anonym) ist bei 3 Abfragen
  pro Update-Lauf irrelevant.
- TLS: GitHub-Root-CA (DigiCert) fest in der Firmware verankern
  (`WiFiClientSecure::setCACert`), inkl. Hinweis im Code, dass ein
  CA-Wechsel bei GitHub ein Firmware-Update erfordert. Bewusst KEIN
  `setInsecure()` – die Box flasht anschließend Geräte, da soll die
  Kette stimmen.

### 3.3 Eigenes Update der Box

- Der eigene neue Stand wird **direkt in den inaktiven OTA-Slot
  gestreamt** (`Update.begin/write/end` während des HTTPS-Downloads) –
  braucht keinen Dateispeicher.
- Reihenfolge bewusst: **Geräte-Images zuerst herunterladen, dann sich
  selbst updaten.** Begründung: Nach dem Self-Update-Reboot liegen die
  Images schon im LittleFS; und eine neue Box-Firmware kann neue
  Protokoll-Details der neuen Geräte-Firmwares bereits sprechen.
- Bestehende Schutzmechanik unverändert (Chip-Check, CRC, Timeout,
  Batterie-Gate ≥ 3,6 V bzw. USB).

### 3.4 Image-Speicher auf der Box

- Neue **Partitionstabelle für den C3** (4 MB): OTA-Slots von 2× 1280 KB
  auf 2× ~1088 KB verkleinern → LittleFS wächst auf ~1,85 MB → Platz für
  **Station- UND Target-Image gleichzeitig** (je ~0,9 MB, Zielbild
  „ein Rundgang für alles").
- Kosten: Box-Firmware (aktuell ~0,83 MB) hat dann ~25 % Luft statt 35 % –
  im Blick behalten, wenn die Web-UI V0.2 kommt. **Partitionswechsel geht
  nicht per OTA** → einmaliger USB-Flash, idealerweise bevor die Box
  endgültig zugeschraubt wird.
- Ablage: `/img/station.bin`, `/img/target.bin` + Metadatei mit Version
  und CRC. Beim Laden eines Images wird der **App-Descriptor**
  (`esp_app_desc_t`, fester Offset 0x20 im Image: Version, Projektname)
  geparst → Plausibilitätscheck („ist das wirklich Station-Firmware?")
  und **Stufe 2 des Versions-Checks**: Die Image-Version wird dritter
  max()-Term des `^`-Markers (neben Listen-Maximum und VersionMemo,
  Doc 18 § 6.1 A3).

### 3.5 ESP-NOW-Push (Kern der Variante B)

Steuerung über normale 36-Byte-Pakete (Reservebereich 0xF4 ff.),
Datenphase über **rohe ESP-NOW-Frames voller Größe** (250 B: Mini-Header
mit Magic + Sequenznummer, ~240 B Nutzdaten) – das 26-Byte-Payload des
Standardpakets wäre für ~1 MB Firmware absurd (~37.000 Pakete).

| Code | Name | Richtung | Inhalt |
|---|---|---|---|
| 0xF4 | `UPDATE_PUSH_BEGIN` | Box → Gerät (Unicast) | Gesamtgröße, CRC32, Version, Fenstergröße |
| 0xF5 | `UPDATE_PUSH_ACK` | Gerät → Box | Fenster-Nr. + Bitmap fehlender Frames (selektive Nachlieferung) |
| 0xF6 | `UPDATE_PUSH_END` | Box → Gerät | Abschluss; Gerät prüft CRC → `Update.end` → Reboot |
| – | Datenframes | Box → Gerät | roher ESP-NOW-Frame, eigener Magic-Header + Seq + 240 B Daten |

- Gerät bleibt durchgehend im ESP-NOW-Modus (kein AP), schreibt per
  `Update.h` in den inaktiven OTA-Slot; Abbruch = altes System bootet.
- Fenster-Logik (z. B. 16 Frames pro Fenster, ACK mit Fehl-Bitmap)
  gleicht Funkverluste gezielt aus. Erwarteter Durchsatz grob
  30–100 KB/s → ~0,5–3 min pro Gerät.
- Abschlussbestätigung wie heute: Box pollt per Discovery, vergleicht
  die gemeldete Version, zeigt „Update OK: vX.Y.Z".
- Empfänger-Code wird geteiltes Core-Modul (analog `WebUpdateService`),
  damit Station und Target identisch funktionieren.

### 3.6 Geführter Modus „Alle aktualisieren"

- Box iteriert die Discovery-Liste, überspringt aktuelle Geräte,
  schiebt auf jedes veraltete das passende Image (Typ → Image),
  Fortschritt pro Gerät + Gesamtliste, am Ende Bericht
  („3 aktualisiert, 1 nicht erreicht, 1 übersprungen (aktuell)").
- Nicht erreichte Geräte bleiben markiert → Rundgang einfach wiederholen.

## 4. Sicherheit (bewusste Haltung, offene Punkte)

- Doc 12 § 3.9 gilt weiter: bewusst keine Funk-Verschlüsselung
  (Halloween-Vorgarten, kein Angriffsszenario). Konsequenz hier:
  `UPDATE_PUSH` ist unauthentifiziert – theoretisch könnte jeder
  ESP-NOW-Sender einem Gerät Firmware unterschieben. Praktisch braucht
  er dafür unser Protokoll + physische Nähe.
- Wenn das je stört (öffentlichere Einsätze): einfachster wirksamer
  Schritt wäre eine **CRC/HMAC mit gemeinsamem Geheimnis im NVS** über
  das Image, verteilt bei der Erstkonfiguration. Als offener Punkt
  notiert, nicht Teil des Erstausbaus.
- Download-Kette: TLS mit festgenagelter CA (3.2) statt blindem Vertrauen.

## 5. Voraussetzungen & Etappen (PR-weise)

**Voraussetzung V0 – Public-Gang (gate für Etappe 2):**
Sound-Historie bereinigen (`git filter-repo`), Repos öffentlich,
Branch-Protection aktivieren (alles bereits auf der Checkliste
Doc 18 § 12).

| Etappe | Inhalt | Repos | USB nötig? |
|---|---|---|---|
| 1 | Neue Partitionstabelle Box + Image-Store (LittleFS) + App-Descriptor-Parser + Stufe-2-Term im `^`-Marker; Image-Upload zunächst über die vorhandene SoftAP-Seite | config | **ja, einmalig** |
| 2 | WLAN-Provisioning (SoftAP-Seite) + HTTPS-Fetcher (GitHub Releases) + Self-Update aus dem Netz + Geräte-Images in den Store | config | nein |
| 3 | ESP-NOW-Push: Protokoll (Core: Nachrichten + PROTOCOL.md + Tests), Empfänger-Modul (Core, geteilt), Sender + Fortschritts-UI (Box), Empfänger-Einbau Station | core, config, station | nein |
| 4 | „Alle aktualisieren"-Flow + Abschlussbericht; Target-Empfänger, sobald `infinitag-now-target` existiert | config, target | nein |

Jede Etappe ist für sich nützlich: Nach 1 hat die Box Stufe-2-Marker,
nach 2 aktualisiert sie sich selbst ohne PC, nach 3 fällt das
AP-Hüpfen für Stationen weg, nach 4 ist das Zielbild komplett.

## 6. Entscheidungen (getroffen 2026-07-12)

- GitHub-Releases-API direkt als Quelle (kein eigener Manifest-Server).
- Partitionstabelle mit zwei Image-Plätzen (Station + Target
  gleichzeitig), Erstflash per USB vor dem endgültigen Zuschrauben.
- Self-Update streamt direkt in den OTA-Slot (kein Speicherbedarf).
- Reihenfolge im Update-Lauf: Images laden → Self-Update → Verteilen.
- SoftAP-Weg bleibt als Fallback/Entwicklungsweg bestehen.

## 7. Verweise

- Doc 18 § 6.1 (heutiger SoftAP-Flow, Versions-Check-Formel),
  § 12 (Public-Checkliste; der dortige ESP-NOW-OTA-Punkt zeigt hierher)
- Doc 12 § 3 / `PROTOCOL.md` im Core-Repo (Paketformat, Reservebereiche)
