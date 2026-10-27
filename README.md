# ESP32-S3 Internet Radio

A WiFi internet radio built on a **generic ESP32-S3 N16R8** board with a 1.8" colour display, a rotary-encoder knob and a small speaker. It lists stations from the [Radio Browser](https://www.radio-browser.info/) directory and has **Akashvani Kolkata** channels pinned at the top of the India section, so it can play Mahalaya during Durga Puja.

##The Photo of my Internet Radio here:
>![ESP32-S3 Internet Radio](Esp32%20S3%20Internet%20Radio.png)




>
> 
## The Working video of this FM Radio link  :
[View my Internet Radio project post on LinkedIn](https://lnkd.in/p/gAnyhw_Y)


> ## MY all videos during making the device:
> ## Project Media

* 📸 [View LinkedIn Post & Project Showcase](https://lnkd.in)
* 🎥 [Watch ESP32-S3 Internet Radio Experiment Videos (Google Drive)](https://drive.google.com/drive/folders/1tDvZXrfW4PPWzyrY4iObu4WEBJz-WG8D)




## Features

- Streams internet radio over WiFi (MP3 / AAC, plus HLS `.m3u8` streams)
- Station list by **country** and **genre** (editable lists in the code)
- Pinned channels in the India section: AIR Sanchayita, AIR Geetanjali, AIR Maitree, FM Rainbow Kolkata
- Rotary knob: volume, mute, and browse/select menu
- Colour UI on a 160x128 ST7735 display
- Serial Monitor debug output for easy troubleshooting

## Parts

| Part | Notes |
|---|---|
| ESP32-S3 N16R8 dev board | 16MB flash, 8MB OPI PSRAM (PSRAM is required) |
| 1.8" ST7735 TFT, 160x128 | Red module with pins VCC, GND, CLK, SDA, RS, RST, CS |
| MAX98357A I2S amplifier | |
| KY-040 rotary encoder | |
| 8Ω 2W speaker | |
| Breadboard and jumper wires | |

## Wiring

| Display pin | ESP32-S3 GPIO |
|---|---|
| VCC | 3.3V |
| GND | GND |
| CLK (SCK) | 1 |
| SDA (MOSI) | 2 |
| RS (DC) | 3 |
| RST | 4 |
| CS | 5 |

This display module has no BL (backlight) pin; the backlight is always on. Leave the NC pins and the SD card pins unconnected.

| MAX98357A | ESP32-S3 GPIO |
|---|---|
| BCLK | 8 |
| LRC | 9 |
| DIN | 7 |
| VIN | 5V |
| GND | GND |

Speaker goes to the amp's output terminals (+ and −).

| KY-040 | ESP32-S3 GPIO |
|---|---|
| CLK | 10 |
| DT | 11 |
| SW | 6 |
| + | 3.3V |
| GND | GND |

All grounds must be connected together.

## Software setup

### 1. Arduino IDE board package

Install **esp32 by Espressif Systems** from Tools → Board → Boards Manager.

### 2. Libraries

| Library | How to install |
|---|---|
| **ESP32-audioI2S** (schreibfaul1) | Not in Library Manager. Download the ZIP from `github.com/schreibfaul1/ESP32-audioI2S`, then Sketch → Include Library → Add .ZIP Library |
| Adafruit ST7735 and ST7789 Library | Library Manager (also install Adafruit GFX when asked) |
| ArduinoJson (6.x) | Library Manager. The code uses `DynamicJsonDocument` |

### 3. Important Arduino IDE settings (Tools menu)

| Setting | Value |
|---|---|
| Board | ESP32S3 Dev Module |
| Flash Size | 16MB (128Mb) |
| Partition Scheme | 16M Flash (3MB APP / 9.9MB FATFS) or Huge APP (3MB No OTA) |
| **PSRAM** | **OPI PSRAM** (required, see troubleshooting) |
| USB CDC On Boot | Enabled |
| Upload speed | 921600 (use 115200 if uploads fail) |

Serial Monitor baud rate: **115200**.

### 4. WiFi credentials

1. Copy `internet_radio/secrets.example.h` to `internet_radio/secrets.h`.
2. Put your WiFi name and password in it.
3. `secrets.h` is listed in `.gitignore`, so it is never uploaded to GitHub.

### 5. Upload

Open `internet_radio/internet_radio.ino`, select the COM port, and upload. If the upload fails, hold **BOOT**, tap **RST**, release **BOOT**, and upload again.

## How to use

| Action | Result |
|---|---|
| Turn knob (normal screen) | Volume up / down |
| Short press | Mute / unmute |
| Long press | Enter browse mode |
| Browse: turn knob | Move between Now Playing, Country and Genre |
| Browse: short press | Edit the highlighted item |
| Edit: turn knob | Change station / country / genre |
| Edit: short press | Apply and play |
| Long press in browse/edit | Go back |

The menu closes by itself after 10 seconds of no activity.

## Changing pin numbers or stations

- Pins are `#define`s at the top of the sketch.
- Pinned channels are in the `PINNED_IN[]` list. Keep names to 18 characters or less so they fit the screen.
- Countries and genres are in the `COUNTRIES[]` and `GENRES[]` tables.

## Problems I faced and how I solved them

### 1. `fatal error: Audio.h: No such file or directory`

**Cause:** the audio library is not installed. It is not in Arduino Library Manager.
**Fix:** install **ESP32-audioI2S** from GitHub as a ZIP (see Libraries above), then restart the IDE.

### 2. `Sketch too big` / `text section exceeds available space in board`

**Cause:** the default partition scheme gives only about 1.3 MB to the program, and this sketch is about 2 MB.
**Fix:** set Flash Size to 16MB, then choose a larger Partition Scheme (3MB APP).

### 3. Encoder pins from the original design do not work on a generic board

**Cause:** the original design was made for the Seeed XIAO ESP32S3, whose encoder pins D6/D7 are GPIO 43/44. On a generic ESP32-S3 board those two GPIOs are the UART TX/RX pins used for serial and flashing.
**Fix:** move the encoder to free pins (GPIO 10 and 11) and change `ENC_CLK` and `ENC_DT` in the code.

### 4. Display pin names differ from the wiring table

**Cause:** the module I had is labelled CLK, SDA, RS, RST, CS with no BL pin.
**Fix:** CLK = SCK, SDA = MOSI, RS = DC. No backlight wire needed. If colours look wrong, try `INITR_GREENTAB` or `INITR_REDTAB` instead of `INITR_BLACKTAB`.

### 5. Screen works but there is no sound

Display and station list worked, but the speaker was silent. The Serial Monitor showed:

```
RingBuffer_Log: bytesRead(512) > bufferFilled(0)
```

**Cause:** the audio buffer lives in PSRAM and PSRAM was not enabled (a slow, far-away station can also make the buffer run empty).
**Fix:** Tools → PSRAM → **OPI PSRAM**, upload again, and try a closer station (for example one from India).
**Tip:** add `Serial.printf("PSRAM size: %d bytes\n", ESP.getPsramSize());` in `setup()`. It should print about 8,000,000. If it prints 0, PSRAM is off.

### 6. `OOM: failed to allocate 720896 bytes for AudioBuffer`

**Cause:** same root cause as above. PSRAM was disabled again after switching to a new sketch (the Tools settings can reset).
**Fix:** set Tools → PSRAM → **OPI PSRAM** again and re-upload. If it still fails, try QSPI PSRAM, then power-cycle the board.

### 7. Hardware checks when the speaker is silent

- Amp VIN should read about 5V with a multimeter
- Speaker wires must go to the amp output terminals, not the power rails
- BCLK, LRC, DIN must not be swapped
- Breadboard contacts can be loose; press every jumper in
- it's better to use a different power supply of 5v DC for esp32 s3 dev board and another 5V Power supply for I2S amplifier for clear sound and no glitches.

## Akashvani channels

The Akashvani stream links came from the public list at `github.com/codito/akashvani`. They are HLS (`.m3u8`) streams, which can take a few seconds to start. Stream URLs can change over time. If a channel stops playing, check that list for an updated link and replace it in `PINNED_IN[]`.

## Credits

- Audio playback: [ESP32-audioI2S](https://github.com/schreibfaul1/ESP32-audioI2S) by schreibfaul1
- Station directory: [Radio Browser](https://www.radio-browser.info/)
- Display: Adafruit GFX and ST7735 libraries
- Akashvani stream list: [codito/akashvani](https://github.com/codito/akashvani)

