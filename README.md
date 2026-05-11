#include "Arduino.h"
#include "Audio.h"
#include "SD.h"
#include "SPI.h"
#include <Arduino_GFX_Library.h>

// --- COLOR DEFINITIONS (Fixes Compilation Errors) ---
#define BLACK   0x0000
#define WHITE   0xFFFF
#define RED     0xF800
#define GREEN   0x07E0
#define BLUE    0x001F
#define CYAN    0x07FF
#define MAGENTA 0xF81F
#define YELLOW  0xFFE0
#define ORANGE  0xFD20

// --- PIN DEFINITIONS (Super Mini Layout) ---
#define SD_CS          8
#define SPI_MOSI      11
#define SPI_SCK       12
#define SPI_MISO      13

#define I2S_BCLK       1
#define I2S_LRC        2
#define I2S_DOUT       3

#define BTN_PLAY       4  // Play/Pause
#define BTN_NEXT       5  // Next (Short) / Vol Up (Long)
#define BTN_PREV       6  // Prev (Short) / Vol Down (Long)
#define BTN_SLEEP      7  // Sleep / Wake (RTC Pin)

// --- OBJECTS ---
Audio audio;

// Display Setup for ST7789 240x240
// Pins: DC=9, RST=10, MOSI=11, SCK=12
Arduino_DataBus *bus = new Arduino_ESP32SPI(9 /* DC */, -1 /* CS */, 12 /* SCK */, 11 /* MOSI */, 13 /* MISO */);
Arduino_GFX *gfx = new Arduino_ST7789(bus, 10 /* RST */, 0 /* rot */, true, 240, 240);

// --- VARIABLES ---
String audioFiles[50]; 
int fileCount = 0;
int currentSong = 0;
int volume = 12; 

void updateUI() {
  gfx->fillScreen(BLACK);
  gfx->setCursor(0, 40);
  gfx->setTextColor(GREEN);
  gfx->setTextSize(2);
  gfx->println("NOW PLAYING:");
  
  gfx->setCursor(0, 80);
  gfx->setTextColor(WHITE);
  if(fileCount > 0) {
    gfx->println(audioFiles[currentSong].substring(1)); 
  } else {
    gfx->println("No Files Found");
  }

  gfx->setCursor(0, 200);
  gfx->setTextColor(YELLOW);
  gfx->print("VOL: "); 
  gfx->println(volume);
}

void setup() {
  // Enable Serial for debugging
  Serial.begin(115200);

  // 1. Initialize Display
  gfx->begin();
  gfx->fillScreen(BLACK);
  gfx->setCursor(0, 20);
  gfx->setTextSize(2);
  
  gfx->setTextColor(CYAN);
  gfx->println("THE OSY-1"); // Your new name!
  delay(300);
  
  gfx->setTextColor(WHITE);
  gfx->println("VERSION: 1.0.4-BETA");
  delay(300);
  
  gfx->setTextColor(GREEN);
  gfx->println("LOADING KERNEL...");
  delay(500);

  // 2. Initialize SD Card with manual SPI mapping
  // This tells the S3 exactly where your enamel wires are soldered
  SPI.begin(SPI_SCK, SPI_MISO, SPI_MOSI, SD_CS); 
  
  if (!SD.begin(SD_CS)) {
    Serial.println("SD Card Mount Failed");
    gfx->setTextColor(RED);
    gfx->setCursor(0, 80);
    gfx->println("SD FAIL!");
    while(1); // Stop here if SD fails
  }
  Serial.println("SD Card OK!");

  // 3. Scan SD for MP3s
  File root = SD.open("/");
  File file = root.openNextFile();
  while(file && fileCount < 50) {
    String name = file.name();
    if(name.endsWith(".mp3") || name.endsWith(".MP3")) {
      audioFiles[fileCount] = "/" + name;
      fileCount++;
    }
    file = root.openNextFile();
  }
  gfx->println("FILES: " + String(fileCount));
  delay(1000);

  // 4. Setup Audio (I2S DAC)
  audio.setPinout(I2S_BCLK, I2S_LRC, I2S_DOUT);
  audio.setVolume(volume); 
  
  if(fileCount > 0) {
    audio.connecttoFS(SD, audioFiles[currentSong].c_str());
  }

  // 5. Setup Buttons
  pinMode(BTN_PLAY, INPUT_PULLUP);
  pinMode(BTN_NEXT, INPUT_PULLUP);
  pinMode(BTN_PREV, INPUT_PULLUP);
  pinMode(BTN_SLEEP, INPUT_PULLUP);

  updateUI();
}

void loop() {
  // Keep the audio buffer running
  audio.loop();

  // Button 1: Play / Pause
  if (digitalRead(BTN_PLAY) == LOW) {
    audio.pauseResume();
    delay(300); // Simple debounce
  }

  // Button 2: Next (Short) / Vol Up (Long)
  if (digitalRead(BTN_NEXT) == LOW) {
    unsigned long start = millis();
    while(digitalRead(BTN_NEXT) == LOW); 
    if (millis() - start > 500) {
      volume = min(volume + 1, 21);
      audio.setVolume(volume);
    } else {
      if(fileCount > 0) {
        currentSong = (currentSong + 1) % fileCount;
        audio.connecttoFS(SD, audioFiles[currentSong].c_str());
      }
    }
    updateUI();
  }

  // Button 3: Prev (Short) / Vol Down (Long)
  if (digitalRead(BTN_PREV) == LOW) {
    unsigned long start = millis();
    while(digitalRead(BTN_PREV) == LOW);
    if (millis() - start > 500) {
      volume = max(volume - 1, 0);
      audio.setVolume(volume);
    } else {
      if(fileCount > 0) {
        currentSong = (currentSong - 1 + fileCount) % fileCount;
        audio.connecttoFS(SD, audioFiles[currentSong].c_str());
      }
    }
    updateUI();
  }

  // Button 4: Sleep Mode
  if (digitalRead(BTN_SLEEP) == LOW) {
    gfx->fillScreen(BLACK);
    gfx->setTextColor(RED);
    gfx->setCursor(60, 100);
    gfx->println("SLEEPING...");
    delay(1000);
    // Wake up using Pin 7 (Sleep Button)
    esp_sleep_enable_ext0_wakeup(GPIO_NUM_7, 0); 
    esp_deep_sleep_start();
  }
}

// Optional Audio callbacks for Serial debugging
void audio_info(const char *info){ Serial.println(info); }


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
