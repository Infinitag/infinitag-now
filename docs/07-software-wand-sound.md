# 07 – Software: Sound‑ESP der Station

Auf dem Sound‑Slot des Wand‑Station Dev Board V1 sitzt ein ESP32‑S3 mit der
Aufgabe, einen sauberen Audio‑Datenpfad bereitzustellen, ohne dass IR‑Senden /
WLAN‑Empfang auf demselben Chip stören. Das war historisch der Auslöser für
den Split in zwei ESPs auf einem PCB.

Es gibt zwei Sketches, die beide den Sound‑ESP betreffen, aber **funktional
nicht zueinander passen** – nur einer von beiden ist der produktive Stand:

| Sketch | Pfad | Stand | Verwendet | Zustand |
| --- | --- | --- | --- | --- |
| `ESP32I2S_S3_MMC_Test.ino` | `/Volumes/basteln/Arduino/ESP32I2S_S3_MMC_Test/` | 2023‑10‑01 | MAX98357 + SD_MMC + I²C‑Slave | **produktiv (bestätigt von Tobias, 2026‑05‑07)** |
| `InfinitagWandStation.ino` | `/Volumes/basteln/Arduino/InfinitagWandStation/` | 2024‑10‑05 | AudioKit (LyraT V4.3) + Touch | Parallel‑Experiment, nicht produktiv |

> **Bestätigt:** Auf dem Sound‑Slot sitzt ein ESP32‑S3‑DevKit mit
> **eingebautem SD‑Karten‑Slot** auf der Modul‑Platine selbst – damit ist
> der `SD_MMC.setPins(39, 38, 40)`‑Zugriff im Sketch direkt verkabelt, ohne
> dass das Wand‑Station‑PCB ein extra SD‑Modul tragen muss.

## ESP32I2S_S3_MMC_Test (produktiv)

Der Sketch heißt zwar nicht „InfinitagWandStation“, ist aber inhaltlich
exakt das, was die Wand‑Station‑Hardware verlangt: I²C‑Slave auf Adresse 1,
beim Empfang eines Byte wird der entsprechende Sound aus einer SD‑Karte über
den MAX98357 abgespielt.

### Wichtige Konstanten

| Symbol | Wert | Bemerkung |
| --- | --- | --- |
| `I2C_DEV_ADDR` | 1 | Slave‑Adresse, passt zu `Wire.beginTransmission(1)` im Logic‑ESP |
| `I2C_SDA` | 19 | I²C SDA am S3 |
| `I2C_SCL` | 20 | I²C SCL am S3 |
| `I2S_DOUT` | 47 | DIN am MAX98357 (Schaltplan: `ESP_S3_IO47`) |
| `I2S_BCLK` | 2 | BCLK am MAX98357 (Schaltplan: `ESP_S3_IO2`) |
| `I2S_LRC` | 1 | LRC am MAX98357 (Schaltplan: `ESP_S3_IO1`) |
| `SD_MMC_CMD` | 38 | SD‑Karte Kommandoleitung |
| `SD_MMC_CLK` | 39 | SD‑Karte Takt |
| `SD_MMC_D0` | 40 | SD‑Karte Daten |

### Setup

```cpp
SD_MMC.setPins(SD_MMC_CLK, SD_MMC_CMD, SD_MMC_D0);
SD_MMC.begin("/sdcard", true, true, SDMMC_FREQ_DEFAULT, 5);

audio.setPinout(I2S_BCLK, I2S_LRC, I2S_DOUT);
audio.forceMono(true);
audio.setVolumeSteps(100);
audio.setVolume(100);

setupWireLoop();   // erstellt Task auf Core 0
```

### I²C‑Empfang

Eine eigene Task läuft auf Core 0, hängt sich als I²C‑Slave an den Bus und
registriert `onReceive(int len)`:

```cpp
Wire.onReceive(onReceive);
Wire.begin((uint8_t)I2C_DEV_ADDR, I2C_SDA, I2C_SCL, 0);
```

Im Callback wird jedes empfangene Byte als Sound‑ID interpretiert und an
`chooseSound()` weitergegeben:

```cpp
void onReceive(int len){
  while(Wire.available()){
    int x = Wire.read();
    chooseSound(x);
  }
}
```

### Sound‑Map (im Code)

`chooseSound()` ist ein 14‑Way‑Switch von ID 0..13 auf konkrete Dateipfade
auf der SD‑Karte:

| ID | Dateipfad im Sketch |
| --- | --- |
| 0 | `/sound-effects/1_booAndLaugh.mp3` |
| 1 | `/sound-effects/2_bubbles.mp3` |
| 2 | `/sound-effects/3_catMeow.mp3` |
| 3 | `/sound-effects/4_daemonIchLiebeKinder.mp3` |
| 4 | `/sound-effects/5_doorBang.mp3` |
| 5 | `/sound-effects/6_gears.mp3` |
| 6 | `/sound-effects/7_littleGirl.mp3` |
| 7 | `/sound-effects/8_owlHooting.mp3` |
| 8 | `/sound-effects/9_psychoSound.mp3` |
| 9 | `/sound-effects/10_scaryClock.mp3` |
| 10 | `/sound-effects/11_spookyScarySkeleton.mp3` |
| 11 | `/sound-effects/12_werewolf.mp3` |
| 12 | `/sound-effects/13_werewolfGrowl.mp3` |
| 13 | `/sound-effects/14_witch.mp3` |

> ⚠️ **Diskrepanz zur Datei‑Numerierung im Halloween‑Ordner.** Auf
> `/Volumes/basteln/Halloween/Lasertag/StationSounds/sound-effects/` heißen
> die Dateien `0_booAndLaugh.mp3` bis `13_witch.mp3` (also 0‑basiert). Im
> Code ist die Numerierung 1‑basiert (`1_booAndLaugh.mp3` bis `14_witch.mp3`).
> Entweder werden die Dateien beim Bespielen der SD‑Karte umbenannt, oder
> der Sketch erwartet andere Dateinamen als auf dem NAS gepflegt.
> Eintrag in [`11-offene-punkte.md`](11-offene-punkte.md).

### Bibliotheken

- `Audio.h` aus **schreibfaul1/ESP32-audioI2S** (nicht zu verwechseln mit
  `arduino-audio-tools`)
- `SD_MMC.h` aus dem ESP32‑Arduino‑Core
- `Wire.h` (I²C)

### Hauptloop

```cpp
void loop() {
  audio.loop();
}
```

Sehr klein – die ganze Audio‑Pipeline (lesen, decodieren, in I²S schieben)
liegt in `audio.loop()`. Die I²C‑Empfänger‑Task läuft auf dem zweiten Core.

### SD‑Karte – am Sound‑ESP‑Modul selbst

Der Sketch greift mit `SD_MMC.setPins(39, 38, 40)` auf eine SD‑Karte zu.
Im Schaltplan des Wand Station Dev Board V1 ist kein SD‑Slot eingezeichnet,
das ist auch nicht nötig: **Tobias hat ein ESP32‑S3‑DevKit gewählt, das
einen MicroSD‑Slot direkt auf der Modul‑Platine trägt** (gibt es z. B. von
Waveshare als „ESP32‑S3‑Zero w/ SD“ oder als „ESP32‑S3‑DevKitC‑1 mit
SDIO‑Karte“). Damit liegen GPIO38/39/40 als CMD/CLK/D0 schon auf der
Karte – der Sound‑PCB‑Slot reicht die Pins nur durch.

## InfinitagWandStation.ino (Experiment, nicht produktiv)

Quelldatei: `/Volumes/basteln/Arduino/InfinitagWandStation/InfinitagWandStation.ino`
(Stand 2024‑10‑05).

Verwendet **AudioKit (LyraT V4.3 Profil)** und liest Sound‑Effekte von einer
SD‑Karte. Trigger‑Auslöser ist `touchRead(32)` – ein **Touch‑Eingang**, der
auf der Wand‑Station‑Hardware gar nicht vorgesehen ist (dort kommt die
Sound‑ID per I²C vom Logic‑ESP).

Indikatoren, dass der Sketch nicht der produktive Stand ist:

- AudioKit konfiguriert das ES8388‑Codec eines AI‑Thinker‑LyraT‑Boards –
  am Wand Station Dev Board V1 ist aber ein **MAX98357** (I²S‑Class‑D) verbaut.
- Kein `Wire.begin()`, kein I²C‑Slave – der Logic‑ESP würde also ins Leere
  reden.
- Touch‑Eingang an Pin 32 – im Schaltplan kein Touch verbaut.
- Adafruit NeoPixel werden permanent grün gehalten – Sound‑ESP hat aber
  keine NeoPixel‑Anschluss zur Wand.

Wahrscheinlich war der Sketch ein Versuch von Tobias, vor Halloween 2024
einen alternativen Sound‑Stack auf einem AI‑Thinker‑LyraT‑Board zu fahren
(Vorgänger‑Sketch `LyratAudioTest.ino` macht fast dasselbe). In der Wand
Station landete am Ende doch wieder die MAX98357‑Variante
(`ESP32I2S_S3_MMC_Test`).

## Verwandte Hilfssketche

| Sketch | Zweck |
| --- | --- |
| `LyratAudioTest.ino` | Vorgänger / Vorlage von `InfinitagWandStation.ino`, nutzt AudioKit + AudioSourceSDMMC. Standalone‑Test auf einem AI‑Thinker‑LyraT. |
| `LyratMiniAudioTest.ino` | Lyrat‑Mini‑V1.1‑Variante des selben Tests. |
| `ESP32I2S_S3_Test.ino` (2023‑06‑13) | Vorlage / Vorversion von `ESP32I2S_S3_MMC_Test.ino` ohne I²C, mit SPI‑SD statt SD_MMC. |
| `ESP32I2S_Test.ino` (2023‑06‑13) | Gleiche Logik auf einem klassischen ESP32 (kein S3). |
