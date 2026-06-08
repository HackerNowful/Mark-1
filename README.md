# Mark-1 — Portable Wireless Security Research Platform

<p align="center">
  <img src="https://img.shields.io/badge/Platform-ESP32-blue?style=for-the-badge&logo=espressif" />
  <img src="https://img.shields.io/badge/Display-TFT%20240x320-cyan?style=for-the-badge" />
  <img src="https://img.shields.io/badge/License-Educational-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Framework-Arduino-red?style=for-the-badge&logo=arduino" />
</p>

> ⚠️ **Legal Notice:** Mark-1 is designed exclusively for **authorized security research**, educational use, and testing on hardware/networks you own or have explicit written permission to test. Unauthorized use against third-party devices or networks may violate local laws. The authors accept no liability for misuse.

---

## What is Mark-1?

Mark-1 is a custom ESP32-based handheld wireless security research tool with a 240×320 TFT touchscreen. It combines WiFi, Bluetooth/BLE, Sub-GHz RF, IR, and NFC capabilities into a single portable device — with a cyberpunk sci-fi UI built entirely from scratch in C++/Arduino.

Think of it as a fully open-source, self-built platform for RF research, protocol analysis, and educational pentesting on your own devices.

---

## Hardware Specifications

| Component | Details |
|---|---|
| **MCU** | ESP32 (dual-core 240MHz) |
| **Display** | TFT 240×320, TFT_eSPI library |
| **Touchscreen** | XPT2046 (HSPI, separate from display) |
| **Sub-GHz Radio** | CC1101 (300–928 MHz) via VSPI |
| **NFC** | PN532 via I2C (address 0x24) |
| **IR** | Transmitter (GPIO5) + Receiver (GPIO4), IRremoteESP8266 |
| **GPIO Expander** | PCF8574 at I2C 0x20 (6-button matrix) |
| **Storage** | MicroSD card (VSPI, CS GPIO5) |
| **Optional** | NRF24L01+ modules ×3 |
| **Power** | Battery ADC on GPIO34 |


### Button Matrix (via PCF8574)

| Button | PCF Pin |
|---|---|
| UP | 6 |
| DOWN | 3 |
| LEFT | 4 |
| RIGHT | 5 |
| SELECT | 7 |
| BACK | 2 |

---

## Feature Overview

### 📶 WiFi Tools

| Tool | Description |
|---|---|
| **AP Scanner** | Scans nearby access points with SSID, BSSID, RSSI, channel. SELECT on any result opens HUD with SAVE / DEAUTH options. |
| **WiFi Analyzer** | Live 14-channel bar graph showing signal strength per channel, rotated to landscape. |
| **Probe Logger** | Promiscuous sniffer that captures probe requests — shows source MAC, vendor OUI lookup, target SSID, and RSSI. Hops channels every 250ms. |
| **Management Frame Analyzer** | Counts beacon, probe, assoc, deauth, disassoc frames across all 13 channels with live per-type counters. |
| **Channel Meter** | Measures packets/second on each of 13 channels, draws live RSSI bar chart. Dwells 400ms per channel. |
| **Hidden SSID Hunter** | Passively sniffs beacon/probe-response frames to detect and reveal hidden SSIDs when a device connects. |
| **PCAP Sniffer** | Full packet capture to SD card in Wireshark-compatible `.pcap` format, with EAPOL/WPA handshake detection alert. Files saved to `/PCAP/`. |
| **Beacon Spam** | Floods the air with randomly named SSIDs using raw 802.11 beacon frames via `esp_wifi_80211_tx`. |
| **Deauther** | Sends 802.11 deauthentication frames to a selected target AP. For authorized testing only. |
| **Deauth Detector** | Monitors for deauth frames on all 13 channels and alerts with attacker MAC and RSSI. |
| **WiFi Connect** | Save and reconnect to known WiFi networks; credentials stored to SD. |

### 🔵 Bluetooth / BLE Tools

| Tool | Description |
|---|---|
| **BLE Scanner** | Active scan, shows device name, RSSI, address. SELECT opens detail HUD. |
| **BLE Tracker** | Continuous passive scan tracking new vs known devices with vendor OUI lookup. |
| **BLE Inspector** | Connects to a selectable device and enumerates all GATT services + characteristics, reads readable values. |
| **BLE Sniffer** | Passive advertisement capture and hex display. |
| **BLE Spoofer** | Emulates Apple AirPods, Beats, AirPods Pro, AirTV, Homepod, and more using BLE advertisement spoofing. Random MAC per advertisement. |
| **AirTag Detector** | Identifies Apple AirTag, Apple FindMy, Samsung SmartTag, Tile, and Chipolo trackers by manufacturer data and service UUIDs. |
| **Flipper Zero Detector** | Detects Flipper Zero devices via BLE payload signature (0x81/0x82/0x83 + 0x30). Shows color variant. |
| **Skimmer Detect** | Looks for known card-skimmer BLE module signatures: HC-05, HC-06, RNBT, SPP UUID, unknown manufacturer ID. |
| **Android BLE Spam** | Sends Google Fast Pair, Samsung SmartTag, and Microsoft SwiftPair advertisement payloads. Cycles through subtypes. |
| **Sour Apple** | Apple-specific BLE advertisement flooding with TV Setup, New Phone, AirPods, Homepod, AppleTV, Vision Pro payloads. |
| **BLE Jammer (NRF24)** | Uses 1–3 NRF24L01+ modules to jam BLE and Bluetooth channels using constant carrier mode. Requires SD card removal. |

### 🔴 IR (Infrared)

| Tool | Description |
|---|---|
| **Learn Remote** | Captures IR signals using IRremoteESP8266. Decodes protocol, code, address, command, raw timing (header, mark, space, gap, average pulse). SEND and SAVE buttons in HUD. Saves to `/Remote/<folder>/<name>.ir`. |
| **Universal Remote** | Pre-built grid UI for TV, AC, LED, Projector, Fan, Audio. Each device has up to 8 action buttons with custom 25×25 XBM icons. Reads `.ir` files from SD; supports both Flipper `type: raw` and `type: parsed` formats. |
| **Select Remote** | File browser for `/Remote/` directory. Browse saved `.ir` files, open detail HUD with SEND / ADD button. |

### 📡 Sub-GHz (CC1101)

| Tool | Description |
|---|---|
| **Read (Hopper)** | Multi-frequency capture using RCSwitch. Decodes KEY, YEK (reversed bits), SN, BTN, TE, protocol. FFT waterfall display. Captured signals listed in paged blocks with HUD for SEND / SAVE. |
| **Read Raw** | ISR-based raw OOK/ASK capture to signed pulse buffer. Dual-core replay (TX on core 1, waveform animation on core 0). Saves Flipper-compatible `.sub` files to `/Sub-Ghz/RAW/`. Live RSSI wave while recording. |
| **Hopping** | Continuous frequency hopper across 17 frequencies from 300–925 MHz. |
| **Signal Meter** | Live RSSI bar + 80-sample scrolling history graph. Squelch line, peak hold, freeze mode. Left/Right steps frequency by 100 kHz. |
| **Spectrum Analyzer** | Sweeps configurable bands (300–928 MHz), draws live per-column RSSI chart with peak hold. 8 band presets including EU key fob (430–440 MHz) and full 300–928 MHz sweep. |
| **OOK Generator** | Generates OOK test signals: Single Pulse, Repeating, Carrier Only, Custom Code (via RCSwitch). Configurable frequency, pulse width, repeat interval, repeat count. |
| **Saved Files** | File browser for `/RAW/` directory. SELECT sends file via CC1101 async TX with Flipper-format parsing. |
| **Config** | 7-parameter CC1101 configuration: Frequency, Modulation (ASK/OOK, 2-FSK, GFSK, MSK), Protocol, RSSI Threshold, Data Rate, Bandwidth, Deviation. Saved to EEPROM. |

### 🏷️ NFC (PN532)

| Tool | Description |
|---|---|
| **UID Scan** | Reads UID, type (MIFARE Classic / NFC Type2/4), length, manufacturer byte. Full sector scan with authentication attempt. Saves dump to `/NFC/TAG_<uid>.txt`. |
| **Card Reader** | ISO14443A passive read with card type detection (Classic, Ultralight, NTAG, DESFire). Shows sample memory. |
| **Read Data** | Authenticates with default keys and reads block 4 data, displays hex and ASCII. |
| **Write Data** | Writes a test string to block 4 of MIFARE Classic with Key A authentication and progress bar. |
| **Erase** | Zero-fills all writable blocks across 16 sectors with sector-by-sector progress display. |
| **Dump Tag** | Full memory dump — Classic (64 blocks) or UL/NTAG (pages 4–36). Output to Serial Monitor and SD card. |
| **Clone Tag** | Two-tag clone flow: read source → confirm swap → write to blank. Supports Gen2 magic card UID block write. Handles Classic (16 sectors × 3 data blocks) and Ultralight/NTAG (user pages). |
| **Decode Access Bits** | Reads MIFARE Classic sector 1 trailer (block 7), extracts and displays C1/C2/C3 access condition bits. |
| **Jam Reader** | PN532 target mode: sends rapid responses toward external readers. For authorized testing only. |
| **Disrupt Emulate** | Cycles FF/00/random payloads in target mode. For authorized testing only. |
| **Tag Disrupt** | Writes malicious sector trailers to MIFARE Classic. Can brick the tag. Own tags only. |

### 📻 NRF24 Analyser (5 tools, NRF24L01+ required)

| Tool | Description |
|---|---|
| **Channel Scanner** | Sweeps channels 0–80, counts carrier-detected hits per channel, draws live heatmap. RIGHT clears peaks. |
| **Device Finder** | Promiscuous 2-byte address sniffing across 16 key channels. Shows address, channel, packet count, last-seen time, payload hex preview. |
| **Packet Sniffer** | Raw 32-byte payload capture on a fixed channel. LEFT/RIGHT changes channel. Hex dump scrolls in log. |
| **Signal Analyser** | Per-channel activity percentage with exponential moving average. Top-5 active channels shown with protocol guesses (BLE, WiFi, Bluetooth, USB, ZigBee, etc.). |
| **Hidden Device Scan** | Scans all **128** RF channels (0–127), including the 81–127 "hidden zone" not covered by standard tools. |

### 🛠️ Apps

| App | Description |
|---|---|
| **Firmware Vault** | Full SD card file manager with OTA `.bin` installation. Confirm popup, segmented progress bar, 3-second countdown launch. |
| **Serial Monitor** | UART terminal on GPIO16/17. Configurable baud (9600–115200). UP/DOWN scrolls history buffer (200 lines). Touch swipe scrolls. |
| **I2C Scanner** | Scans all 127 I2C addresses, names common chips (PN532, PCF8574, OLED, BME280, ADS1115, etc.). |
| **Calculator** | Scientific calculator with sin, cos, tan, sqrt, log, exp. Touch or D-pad navigation across 6×4 button grid. |
| **Paint** | Full touchscreen freehand drawing app. |
| **Spy-Cam** | Creates WiFi AP (`Mark1_CAM` / `Mark1_CAM`), accepts WebSocket JPEG stream from an ESP32-CAM companion. Displays live video via TJpg_Decoder. |

### 🎮 Games

- **Snake** — Classic snake game with increasing speed. D-pad controls.
- **Tetris** — Full Tetris with sprite buffer (TFT_eSprite), score, speed scaling, next-piece preview.

### ⚙️ Settings

| Setting | Description |
|---|---|
| **Theme** | 8 themes: BlackWhite, SciFi, Matrix, Hacker, Flipper, RedAlert, Custom (RGB input), CyberPunk. Custom themes save per-profile to EEPROM (5 profiles). |
| **WiFi Connect** | Save/load/delete WiFi credentials. Credentials stored on SD card at `/wifi.cfg`. |
| **SD Info** | SD card type, total/used/free space, storage usage bar, file system info. |
| **Device Info** | Chip model, core count, CPU frequency, flash size, free/total RAM, firmware size, OTA free space, SDK version, MAC address. Typewriter animation. |
| **GPIO Info** | Pin reference table for the Mark-1 hardware. |

---

## SD Card File Structure

```
/ (root)
├── wifi.cfg              ← saved WiFi credentials
├── PCAP/
│   └── capture_N.pcap    ← Wireshark captures
├── NFC/
│   └── TAG_<uid>.txt     ← NFC dumps
├── Sub-Ghz/
│   ├── <name>.sub        ← decoded sub-GHz keys (Flipper format)
│   └── RAW/
│       └── RAW_NNN.sub   ← raw OOK captures (Flipper format)
├── RAW/
│   └── RAW_NNN.sub       ← alternate raw path
├── Remote/
│   └── <folder>/
│       └── <button>.ir   ← IR button files
└── *.bin                 ← OTA app binaries
```

### Flipper-Compatible `.sub` Format

```
Filetype: SubGhz Key File
Version: 1
Frequency: 433920000
Protocol: 1
Bit: 24
Key: A1B2C3
TE: 300
BTN: F
SN: 0A1B2C
```

### IR File Format

```
name: Power
type: parsed
protocol: NEC
address: 0x04
command: 0x08
```

or raw:

```
name: CoolDown
type: raw
frequency: 38000
data: 9000 4500 560 560 560 1690 ...
```

---

## Building

### Requirements

- Arduino IDE 2.x or PlatformIO
- ESP32 Arduino Core 2.x
- Board: **ESP32 Dev Module**, Flash: 16MB, Partition: `default_8MB` or custom with OTA

### Libraries

```
TFT_eSPI
XPT2046_Touchscreen
ELECHOUSE_CC1101_ESP32DIV   (custom fork, shared SPI)
IRremoteESP8266
RF24
PN532 + PN532_I2C
NdefMessage + EmulateTag
BLEDevice (ESP32 BLE Arduino)
ArduinoFFT
RCSwitch
PCF8574
SD (ESP32)
EEPROM
ArduinoWebsockets
TJpg_Decoder
TimeLib
PCAP
arduinoFFT
TFT_eSprite (part of TFT_eSPI)
```

### TFT_eSPI Configuration

You must configure `User_Setup.h` in TFT_eSPI for your specific Mark-1 display driver and pins. Set VSPI pins to match: SCK=18, MISO=19, MOSI=23, TFT CS and DC to your display's wiring.

---

## Navigation Guide

### Main Menu
The home screen is a 3×3 icon grid showing: WiFi, Bluetooth, IR, Sub-GHz, NFC, NRF, APPS, Games, Settings.

- **UP/DOWN** — move cursor row by row
- **LEFT/RIGHT** — move cursor column by column
- **SELECT** — enter highlighted menu
- **Touch** — tap icon to select, swipe left/right to change page

### Submenus
All submenus are vertical lists with up to 8 visible items per page.

- **UP/DOWN** — move selection
- **SELECT** — execute highlighted item
- **BACK** — return to previous menu
- **Touch** — tap item to execute, swipe up/down to page through long lists

### HUD Detail Screens
Most tools open a HUD overlay when you press SELECT on a captured item.

- **LEFT/RIGHT** — navigate BACK / SEND / SAVE buttons
- **SELECT** — execute highlighted button
- **BACK** — close HUD
- **Touch** — tap button directly

---

## Boot Sequence

1. Power on → `INITIALIZING...` typewriter text
2. Mark-1 splash image displayed for 2.5 seconds
3. Boot warning screen (warranty/legal notice) — hold any key to skip, or wait for progress bar
4. Hardware init: CC1101 → SD mount → RCSwitch → status tile check
5. Main menu displayed with hardware status tiles (SD / CC1101 / IR / NFC / BLE)

---

## Status Tiles

At the bottom of the main menu, 5 status tiles show hardware availability detected at boot:

`[SD]  [SUB-GHz]  [IR]  [NFC]  [BLE]`

Active (green) = detected. Inactive (dim) = not found or failed init.

---

## Theme System

8 built-in themes, switchable in Settings → Theme:

| # | Name | Colors |
|---|---|---|
| 0 | BlackWhite | Pure black/white |
| 1 | SciFi | Cyan on dark blue (default) |
| 2 | Matrix | Green on black |
| 3 | Hacker | Bright green on black |
| 4 | Flipper | Orange/amber on black |
| 5 | RedAlert | Red on black |
| 6 | Custom | User-defined RGB (5 saved profiles) |
| 7 | CyberPunk | Cyan/magenta/yellow on black |

Themes are saved to EEPROM and persist across reboots.

---

## SPI Bus Architecture

Mark-1 shares a single VSPI bus (SCK=18, MISO=19, MOSI=23) between the CC1101 radio and the SD card. A software CS switching mechanism (`spiSwitch.h`) ensures only one device is selected at a time:

- `switchToCC1101()` —  deselects SD
- `switchToSD()` —  deselects CC1101
- `spiDeselectAll()` — both CS high

The NRF24L01+ modules use **different CS pins** (27, 17, 5) and **conflict with CC1101 and SD**. A warning popup is shown before launching the NRF jammer, prompting the user to physically remove the SD card.

---

## OTA App System

Mark-1 supports installing `.bin` firmware files from SD:

1. Place a compiled `.bin` in the SD root
2. Navigate to Apps → Firmware Vault
3. Select the file, confirm the install dialog
4. Mark-1 writes to OTA partition 


---

## Project Status

| Feature | Status |
|---|---|
| WiFi tools | ✅ Complete |
| BLE tools | ✅ Complete |
| Sub-GHz tools | ✅ Complete |
| IR tools | ✅ Complete |
| NFC tools | ✅ Complete |
| NRF24 analyser | ✅ Complete |
| Apps | ✅ Complete |
| Games | ✅ Complete |
| WarDrive (GPS) | 🔧 Planned |
| Evil Twin Portal | 🔧 Planned |
| Mark-2 hardware | 🔧 In progress |

---

## Support

**Email:** contact@hackgears.in

---

*Mark-1 is an open IoT research platform. Use responsibly. Test only on hardware and networks you own or have explicit permission to test.*
