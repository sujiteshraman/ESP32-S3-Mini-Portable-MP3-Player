# THE OSY-1

### ESP32-S3 Portable MP3 Player

THE OSY-1 is a custom ESP32-S3 based portable MP3 player featuring:

* ST7789 TFT Display
* PCM5100A I2S DAC Audio
* MicroSD MP3 Playback
* Physical Button Controls
* Deep Sleep Mode
* Minimal Retro UI

Built using Arduino IDE and optimized for compact handheld designs.

---

# Features

* MP3 playback from MicroSD card
* 240x240 ST7789 TFT interface
* I2S digital audio output
* Play/Pause control
* Next/Previous track control
* Long-press volume adjustment
* Deep sleep power saving
* Dynamic SD card scanning
* Compact ESP32-S3 Mini compatible

---

# Hardware Used

| Component      | Description          |
| -------------- | -------------------- |
| ESP32-S3 Mini  | Main microcontroller |
| ST7789 TFT     | 240x240 SPI display  |
| PCM5100A       | I2S DAC audio output |
| MicroSD Module | MP3 storage          |
| Push Buttons   | Playback controls    |
| LiPo Battery   | Portable power       |

---

# Pin Configuration

## SPI (Display + SD Card)

| Function | GPIO   |
| -------- | ------ |
| SPI MOSI | GPIO11 |
| SPI SCK  | GPIO12 |
| SPI MISO | GPIO13 |
| SD CS    | GPIO8  |

---

## I2S Audio

| Function | GPIO  |
| -------- | ----- |
| I2S BCLK | GPIO1 |
| I2S LRC  | GPIO2 |
| I2S DOUT | GPIO3 |

---

## Buttons

| Button      | GPIO  |
| ----------- | ----- |
| Play/Pause  | GPIO4 |
| Next / Vol+ | GPIO5 |
| Prev / Vol- | GPIO6 |
| Sleep/Wake  | GPIO7 |

---

## Display Pins

| Display Pin | GPIO   |
| ----------- | ------ |
| DC          | GPIO9  |
| RST         | GPIO10 |

---

# Wiring Diagram

```text
ESP32-S3 MINI
│
├── ST7789 TFT
│   ├── MOSI → GPIO11
│   ├── SCK  → GPIO12
│   ├── DC   → GPIO9
│   ├── RST  → GPIO10
│
├── SD CARD
│   ├── MOSI → GPIO11
│   ├── MISO → GPIO13
│   ├── SCK  → GPIO12
│   ├── CS   → GPIO8
│
├── PCM5100A DAC
│   ├── BCLK → GPIO1
│   ├── LRC  → GPIO2
│   ├── DIN  → GPIO3
│
└── BUTTONS
    ├── PLAY → GPIO4
    ├── NEXT → GPIO5
    ├── PREV → GPIO6
    └── SLEEP → GPIO7
```

---

# Required Libraries

Install these libraries using Arduino Library Manager:

## Core Libraries

* ESP32 AudioI2S
* Arduino GFX Library
* SD
* SPI

---

# Installation

## 1. Install ESP32 Board Package

Arduino IDE:

* File → Preferences
* Add ESP32 Board URL:

```text
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```

Then:

* Tools → Board Manager
* Install "ESP32 by Espressif Systems"

---

## 2. Install Required Libraries

Open:

* Sketch → Include Library → Manage Libraries

Install:

* ESP32 AudioI2S
* Arduino_GFX

---

## 3. Select Board

Recommended:

* ESP32S3 Dev Module

---

## 4. Upload Code

* Select correct COM Port
* Upload sketch

---

# SD Card Setup

* Format SD card as FAT32
* Place `.mp3` files in root directory

Example:

```text
/song1.mp3
/song2.mp3
/song3.mp3
```

---

# Controls

| Action           | Function       |
| ---------------- | -------------- |
| PLAY button      | Play/Pause     |
| NEXT short press | Next Track     |
| NEXT long press  | Volume Up      |
| PREV short press | Previous Track |
| PREV long press  | Volume Down    |
| SLEEP button     | Deep Sleep     |

---

# Boot Screen

```text
THE OSY-1
VERSION: 1.0.4-BETA
LOADING KERNEL...
```

---

# Current Limitations

* Maximum 50 MP3 files
* No folder browsing yet
* No playlist support
* No album art support
* UI redraw flicker on updates

---

# Planned Features

* FFT Visualizer
* Album Art
* Smooth UI animations
* Bluetooth Audio
* Battery Indicator
* Playlist Support
* Rotary Encoder Navigation
* Winamp-style Themes

---

# Recommended Hardware Improvements

* Add TFT CS pin for shared SPI stability
* Add 100µF capacitor near DAC
* Add 470µF capacitor near ESP32 power input
* Use short audio traces to reduce noise

---

# Audio DAC Recommendation

Recommended DAC:

* PCM5100A

Best for:

* Headphones
* IEMs
* Portable stereo audio

---

# Deep Sleep

The OSY-1 supports deep sleep mode using:

```cpp
esp_sleep_enable_ext0_wakeup(GPIO_NUM_7, 0);
```

Wake-up occurs using the sleep button connected to GPIO7.

---

# License

MIT License

---

# Credits

Developed by Sujitesh Raman

Powered by:

* ESP32-S3
* Arduino Framework
* ESP32 Audio Library
* Arduino_GFX

---

# Preview

```text
NOW PLAYING:
song.mp3

VOL: 12
```

---

# Future Goal

THE OSY-1 aims to become a fully open-source retro-inspired portable music player platform for ESP32-based handheld devices.
