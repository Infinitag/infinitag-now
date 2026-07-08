# 05 – Protokoll: IR + I²C

Quelle: `Infinitag_Core.{h,cpp}` (2017, Tobias Stewen / Jani Taxidis /
Florian Kleene). Identisch in beiden vorhandenen Kopien:

- `/Volumes/basteln/Arduino/Infinitag_Core/`
- `/Volumes/basteln/PlatformIo/Projects/Infinitag Target/lib/Infinitag_Core/`

## IR‑Protokoll

24 Bit als RC5‑Frame (`IrSender.sendRC5(value, 24)` beim Wand,
`IrReceiver.decodedIRData.decodedRawData` am Target). Die Bitreihenfolge
ist MSB‑first.

### Bit‑Layout

```
Bit  23 22 21 20 19 18 17 16 15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0
     ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
     │S │ G  G │ T  T  T │  P  P  P  P  P  │ C  C  C  C │  V  V  V  V  V  V  V  V │ X │
     └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
       │  │  │ │       │ │              │ │            │  │                     │
       │  │  │ │       │ │              │ │            │  │                     └ Parity (1 Bit, gerade Parität über Bits 23..1)
       │  │  │ │       │ │              │ │            │  └─ cmdValue (8 Bit) – kontextabhängig (z. B. letzter IP‑Block des Schützen)
       │  │  │ │       │ │              │ └─ cmd (4 Bit) – Befehlsnummer
       │  │  │ │       │ └─ playerId (5 Bit)
       │  │  │ └─ teamId (3 Bit)
       │  └─ gameId (2 Bit)
       └─ isSystem (1 Bit) – `true` = System‑/Steuerbefehl, `false` = Spielereignis
```

Encoding/Decoding läuft über `Infinitag_Core::irEncode()` und
`Infinitag_Core::irDecode()`. Beide Seiten (Wand & Target) nutzen dieselbe
Library, dieselben Konstanten – das ist das einzige, was den Bus „kompatibel“
hält.

### Belegung der Felder im Halloween‑Setup

Im aktuellen Stand wird das Protokoll bewusst sparsam genutzt – nur drei
Felder sind belegt:

| Feld | Heutiger Wert | Bedeutung |
| --- | --- | --- |
| `isSystem` | `false` (Schuss), `true` (Reset) | Spiel‑Event vs. System‑Event |
| `gameId` | 0 | nicht verwendet |
| `teamId` | 0 | nicht verwendet |
| `playerId` | 0 | nicht verwendet |
| `cmd` | `1` | „Hit“ (bzw. „Open Config Portal“ wenn `isSystem == true`) |
| `cmdValue` | letztes Oktett der Wand‑IP | wird vom Target genutzt, um dem richtigen Schützen zu antworten |

Mehr Felder zu nutzen (Teams, Spieler, Game‑IDs) ist möglich; der Code im
Target prüft sie aber aktuell nicht auf Filter, sondern reagiert auf jeden
sauber decodierten Frame mit `cmd == 1`.

### Beispiel: Schuss vom Wand

```cpp
// In InfinitagWand.ino, Funktion shoot():
IPAddress ip = WiFi.localIP();
int lastIpBlock = ip[3];
unsigned long shotValue = infinitagCore.irEncode(
    /*isSystem*/ false,
    /*gameId  */ 0,
    /*teamId  */ 0,
    /*playerId*/ 0,
    /*cmd     */ 1,
    /*cmdValue*/ lastIpBlock);
IrSender.sendRC5(shotValue, 24);
```

Wenn die Station‑IP z. B. `192.168.0.42` ist, wird `cmdValue = 42` mitgeschickt.
Das Target liest das aus, hängt es an seinen IP‑Prefix `192.168.0.` an und
ruft so gezielt den passenden Schützen an, falls mehrere Stationen aktiv
wären.

### Beispiel: Target‑Reset (Admin‑Tool)

Im Sketch `InfinitagTargetResetter` wird ein „System‑Hit“ verschickt:

```cpp
unsigned long shotValue = infinitagCore.irEncode(true, 0, 0, 0, 1, 0);
IrSender.sendRC5(shotValue, 24);
```

→ Target sieht `isSystem == true` + `cmd == 1` → öffnet das WiFi‑Captive‑Portal
für ~60 s. Sehr nützlich, ohne das Target physisch ans Power‑Cycle zu zwingen.

### Parität

`Infinitag_Core::irEncode()` setzt Bit 0 als gerade Parität über die Bits 23..1.
`irDecode()` prüft diese Parität als einzigen Integritätscheck. Das ist
eindeutig **kein** robustes ECC; in einem überfüllten IR‑Umfeld (Sonnenlicht,
fluoreszierende Beleuchtung) kann ein einzelner Bitflip unentdeckt
durchschlüpfen, wenn er die Parität nicht verändert. Praktisch reicht es im
Halloween‑Einsatz aus.

## I²C‑Protokoll (Station‑intern)

Logic‑ESP (Master) → Sound‑ESP (Slave, Adresse `0x01`).

### Heute genutzt

| Aktion | Bytes | Bemerkung |
| --- | --- | --- |
| Sound abspielen | `[ soundId : uint8 ]` | wird in `InfinitagWand.ino::sendInt()` per `Wire.write(byte(...))` gesendet. Der Sound‑ESP soll den Index in `soundEffects[]` lesen und die zugehörige MP3 starten. |

### Aus `Infinitag_Core` übernommen (nicht im Halloween‑Code aktiv)

`Infinitag_Core` definiert eine zweite I²C‑Adresse `0x22` und einen
Befehlssatz für „Sensors“ (gedacht für ein Westen‑/Mehrsensor‑Setup). Diese
sind **im Halloween‑Setup nicht in Verwendung**, aber dokumentiert, weil sie
die Struktur künftiger Erweiterungen vorgeben:

| Code | Funktion | Payload |
| --- | --- | --- |
| `0x01` | SetGameId | 1 Byte |
| `0x02` | SetTeamId | 1 Byte |
| `0x03` | SetPlayerId | 1 Byte |
| `0x04` | SetAnimation | 8 Byte (animationId, duration LSB, duration MSB, R, G, B, W, repeat) |
| `0x05` | SetSensorId | 1 Byte |
| `0x06` | Ping | 1 Byte (senderId) |
| `0x07` | PingSetAlive | 1 Byte (0/1) |

Wenn man später echte Spielerwesten bauen würde, wäre das die Stelle, an
der man mit den Sensor‑Boards spricht (siehe
[`11-offene-punkte.md`](11-offene-punkte.md), Abschnitt „Roadmap“).

## „Wifi‑Encoding“ (im Code vorhanden, aktuell unbenutzt)

`Infinitag_Core::wifiEncode()` / `wifiDecode()` machen formal dasselbe wie das
IR‑Pendant, nur ohne `bitRead`-/`bitWrite`-Schleifen (direkt mit Shift‑Operatoren).
Die ursprüngliche Idee war, denselben 24‑Bit‑Frame auch über UDP/MQTT zu
schicken, statt nur über IR. Im aktuellen Halloween‑Code wird das nicht
verwendet – die Targets schreiben einfach klassisch HTTP‑GETs.

> ℹ️ Tobias hat in der Konversation MQTT als Transport erwähnt. Im
> aktuell vorhandenen Code findet sich keine MQTT‑Implementierung. Falls
> MQTT geplant ist, ist das ein offener Punkt – siehe
> [`11-offene-punkte.md`](11-offene-punkte.md).

## HTTP‑„Protokoll“ (Target → Wand Station)

Reines REST‑GET, ohne Body, ohne Auth.

```
GET /trigger_effect?sound=<soundId> HTTP/1.1
Host: <ipPrefix><ipBlock>:8080
```

- Target ruft das auf Port 80 oder 8080? → **Diskrepanz, siehe
  [`11-offene-punkte.md`](11-offene-punkte.md)**: Der Wand‑Logic‑ESP startet
  seinen Webserver explizit auf Port **8080** (`WebServer server(8080);`),
  während der Target‑Code den Port aktuell als 80 (`http.begin(...)` ohne
  `:8080`) bzw. hartcodiert auf eine andere IP setzt. Die heute gebauten
  Konfigurationen funktionieren, weil `InfinitagTargetTest.ino` beim
  Schießen `http://192.168.1.<ipBlock>:8080/trigger_effect?sound=<id>` baut
  – das ist die saubere Variante.
- Antwort: HTTP 200 ohne Body. Der Wand‑Logic‑ESP gibt nach `delay(30)` ein
  `server.send(200)` zurück.
