# 09 – Halloween‑Schießbude (Vorgarten‑Setup)

Das Infinitag‑System wird heute primär als Halloween‑Schießbude für Kinder
genutzt: ein „magischer Zauberstab“ (= Wand) mit Laserpointer, sichtbarer
Trigger‑Beleuchtung und IR‑Strahl, der Halloween‑Figuren („Targets“) im
Vorgarten zum Reagieren bringt. Reagieren = Sound + ggf. mechanischer Effekt
(z. B. zuckende Spinne, leuchtende Pumpkin‑Augen, Nebelmaschine kurz an).

## Aufbau

```
            ┌───────────────────────────┐
            │  Lautsprecher in der Box  │
            │  (Wand Station)           │
            │  — versorgt aus Steckdose │
            │  — WLAN: Heim‑WLAN        │
            └────────────┬──────────────┘
                         │ Kabel (S7B 7‑pin, ~1‑1,5 m)
                         ▼
                    ┌────────┐
                    │  WAND  │  (Kind hält den Zauberstab)
                    └────────┘
                         │ IR‑Schuss (Laserpointer als Visier)
       ┌─────────────────┼─────────────────┐
       ▼                 ▼                 ▼
   ┌────────┐        ┌────────┐        ┌────────┐
   │Target 1│        │Target 2│        │Target N│
   │Pumpkin │        │Spinne  │        │Geist   │
   │+ LEDs  │        │+ Motor │        │+ Sound │
   └────────┘        └────────┘        └────────┘
```

## Was eine Runde aussieht

1. Kind kommt vorbei, bekommt den Zauberstab in die Hand.
2. Drückt den Trigger → Laser zeigt das Ziel an, IR‑Strahl wird gleichzeitig
   gefeuert. Wand‑LEDs blitzen weiß.
3. Trifft das Kind ein Target, wechselt der LED‑Ring des Targets auf eine
   pulsierende rote Welle, und das Target schaltet seinen oder seine
   Halloween‑Effekt(e) für die konfigurierte Hit‑Zeit ein.
4. Parallel meldet das Target per HTTP an die Wand‑Station, dass es getroffen
   wurde, mit der konfigurierten Sound‑ID.
5. Die Wand‑Station spielt den passenden Halloween‑Sound aus dem Lautsprecher.
6. Nach Hit‑Time geht das Target in den Idle‑Modus zurück (Regenbogen‑
   Animation).

## Halloween‑Assets

Im Ordner `/Volumes/basteln/Halloween/Lasertag/` liegen alle Spielzutaten:

### Sound‑Effekte für die Wand Station

`StationSounds/sound-effects/` (14 MP3‑Files, durchnummeriert):

| ID | Datei | Idee |
| --- | --- | --- |
| 0 | `0_booAndLaugh.mp3` | klassisches Boo + Lachen |
| 1 | `1_bubbles.mp3` | Hexenkessel‑Bubbeln |
| 2 | `2_catMeow.mp3` | Katze faucht |
| 3 | `3_daemonIchLiebeKinder.mp3` | gruselige Stimme „Ich liebe Kinder“ |
| 4 | `4_doorBang.mp3` | Türknallen |
| 5 | `5_gears.mp3` | Maschinen‑Zahnräder |
| 6 | `6_littleGirl.mp3` | Kinderstimme (Gruseleffekt) |
| 7 | `7_owlHooting.mp3` | Eule |
| 8 | `8_psychoSound.mp3` | Streicher‑Stich (Psycho‑Anlehnung) |
| 9 | `9_scaryClock.mp3` | tickende Standuhr + Schlag |
| 10 | `10_spookyScarySkeleton.mp3` | „Spooky Scary Skeleton“‑Sample |
| 11 | `11_werewolf.mp3` | Werwolf heulen |
| 12 | `12_werewolfGrowl.mp3` | Werwolf knurren |
| 13 | `13_witch.mp3` | Hexenlachen |

### Spell‑Sounds für den Wand selbst

`Spell/`:

- `spell.mp3`
- `magic_spell_long.mp3`
- `darkSpell.mp3`
- `zapsplat_magic_spell_dark_evil_whoosh_hit_002_32051.mp3`
- `zapsplat_fantasy_magic_dark_thud_impact_001_45106.mp3`

Diese liegen extra (nicht in StationSounds), weil sie potenziell direkt
vom Wand getriggert werden sollen, sobald der Sound‑ESP einen entsprechenden
„Schuss‑Sound“-Befehl bekommt. Aktuell **nicht** im Code verdrahtet (siehe
[`11-offene-punkte.md`](11-offene-punkte.md)).

### 3D‑Druck‑Modelle für Targets

`Target/`:

- `Target_V3 v26.3mf` – Einzeltarget‑Gehäuse V3
- `4x_Target_V3 v26.gcode.3mf` – 4er‑Plate, vorbereitet zum Drucken
- `4xSlide.3mf` – Slide‑Mechanik (vermutlich ausschiebbares Target?)

Welche Druckparameter / Filament‑Wahl Tobias verwendet, ist nicht
dokumentiert; idealerweise bei Gelegenheit hier ergänzen.

## Praktische Hinweise

- **Reichweite:** TSAL6200 + 33 Ω + Linse → real ~5–8 m im Garten,
  abhängig von Umgebungslicht. Bei Sonnenuntergang am besten.
- **WLAN:** Alle Targets + Station müssen im selben Subnetz hängen. Im
  Eigenheim am einfachsten: Heim‑WLAN, alle Geräte fix per Captive‑Portal
  vorgekonfiguriert.
- **Stromversorgung:**
  - Station: 5 V / 2 A Steckernetzteil reicht (ESPs + MAX98357 + Lautsprecher
    bei moderater Lautstärke).
  - Targets: USB‑Powerbank (5 V) oder USB‑Netzteil. Stromaufnahme im
    Idle‑Modus mit 12 RGBW LEDs auf voller Helligkeit kann 600 mA werden.
    Wenn das zuviel ist, im Code `BRIGHTNESS` reduzieren.
- **Wetter:** Targets sind PCB‑mäßig nicht wasserfest. Im 3D‑gedruckten
  Gehäuse aufstellen und nicht direkt im Regen.
- **Einrichtung am Halloween‑Abend:**
  1. Wand Station einstecken, kurz warten (rote LEDs während WLAN‑Connect),
     dann blau = bereit.
  2. Targets einschalten, Regenbogen‑Animation = idle, Verbindung steht.
     Falls gelbliches Glimmen: WLAN‑Portal aktiv, einmal mit Handy konfigurieren.
  3. Wand am Kabel der Station einstecken, Trigger drücken, Laser sichtbar?
  4. Auf das nächste Target zielen – es muss in rote Pulswelle wechseln,
     der Lautsprecher muss den passenden Sound spielen.

## Reset / Notausgang

- Target zurücksetzen: über `InfinitagTargetResetter` (auf einem zweiten
  ESP / Tasterboard) einen System‑Hit per IR senden → Captive‑Portal öffnet,
  man kann neu konfigurieren.
- Wand Station zurücksetzen: kurz vom Strom trennen, oder im Captive‑Portal
  den Reset‑Eintrag wählen.
