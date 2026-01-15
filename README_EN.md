[🇵🇱 Wersja polska](README.md) | 🇬🇧 English version

# ESP32-S3 Voice Satellite for Home Assistant

![Project Status](https://img.shields.io/badge/Status-Stable-green)
![ESPHome](https://img.shields.io/badge/ESPHome-2025.12.5+-blue)
![Framework](https://img.shields.io/badge/Framework-ESP--IDF-orange)
![License](https://img.shields.io/badge/License-MIT-grey)
[![BuyCoffee](https://img.shields.io/badge/BuyCoffee.to-Support-green?logo=buy-me-a-coffee&logoColor=white)](https://buycoffee.to/Krisuuuu)

> ✅ **Project Status:** Stable production release 2.0.0 with full media player support, advanced LED effects, and **dual I2S bus architecture**.

> 💡 **Advanced hobbyist project built with passion.**
>
> **A note from the author:** I am not a professional programmer – I develop this project with the support of AI. It is the result of my experiments and passion for home automation. While the code is stable and field-tested in my own setup, I am always open to improvements. If you are a developer, your Pull Requests are more than welcome!

Advanced voice satellite for Home Assistant based on ESP32-S3. The project uses **two independent I2S buses** for high-quality audio and a WS2812B LED ring for visual feedback.

---

## ✨ Features

### 🎤 Audio
- **Wake Word:** "Okay Nabu" (local detection with Hardware VAD)
- **Dual I2S Bus Architecture:** Separate buses for microphone and amplifiers
- **Dual Mono Output:** 2x MAX98357A on shared data line (increased volume)
- **Media Player:** Simultaneous announcements and music playback (mixer)
- **High Quality:** 32-bit microphone (16kHz), 16-bit amplifiers (24kHz)
- **Hardware VAD:** Hardware Voice Activity Detection for silence suppression
- **Optimization:** Low latency through optimized audio pipeline (24kHz WAV)

### 💡 LED (WS2812B)
- **Boot Filling:** Progressive filling on startup (cyan)
- **Connection Breathe:** Green breathing on API connection
- **Listening Double:** Blue spinning effect (2 points)
- **Thinking Pulse:** Blue pulsing effect
- **Speaking Reverse:** Purple reverse spinning effect
- **Error:** Red blinking
- **Mute:** Red solid 30%

### 🎛️ Controls
- **Mute Switch:** Microphone mute (red LED)
- **Push-to-Talk:** Button in Home Assistant
- **Auto Re-Arm:** Automatic wake word restart
- **Smart Boot Sequence:** Intelligent startup sequence with connection detection

---

## 🆕 What's new in version 2.0.0?

### Dual I2S Bus Architecture
The project has been redesigned with **two independent I2S buses**:
- **Bus #1:** INMP441 Microphone (GPIO15, GPIO16, GPIO6)
- **Bus #2:** MAX98357A Amplifiers (GPIO4, GPIO5, GPIO11)

### Other key changes:
- ✅ Hardware VAD for silence suppression
- ✅ Optimized audio pipeline (24kHz)
- ✅ Simplified wake word (only "Okay Nabu")
- ✅ Improved audio buffers for stability

📋 **Full changelog:** See [CHANGELOG.md](CHANGELOG.md)

---

## 🛠️ Hardware (BOM)

| Component | Model | Qty | Notes |
|-----------|-------|-----|-------|
| **MCU** | ESP32-S3 DevKitC-1 N16R8/N16R16 | 1 | 16MB Flash / 8-16MB PSRAM (Octal) |
| **Microphone** | INMP441 | 1 | I2S MEMS |
| **Amplifier** | MAX98357A | 2 | Parallel on GPIO11 |
| **LED** | WS2812B Ring | 1 | 12 LEDs |
| **PSU** | 5V 3A | 1 | Min. 2.5A |
| **Capacitors** | 470µF 16V | 3 | Each amplifier + LED |
| **Fuse** | 3A | 1 | Optional |

---

## 🔌 Wiring Diagram (Pinout)

### Microphone Bus (I2S Bus #1)
| ESP32 Pin | → | INMP441 Microphone | Function |
|-----------|---|-------------------|----------|
| **GPIO15** | → | **SCK** | Bit Clock |
| **GPIO16** | → | **WS** | Word Select |
| **GPIO6** | → | **SD** | Data In |
| **3.3V** | → | **VDD** | Power ⚠️ |
| **GND** | → | **GND** | Ground |
| **GND** | → | **L/R** | Left channel |

### Amplifier Bus (I2S Bus #2)
| ESP32 Pin | → | 2x MAX98357A | Function |
|-----------|---|--------------|----------|
| **GPIO4** | → | **BCLK** | Bit Clock |
| **GPIO5** | → | **LRC** | Word Select |
| **GPIO11** | → | **DIN** (both) | Data Out (parallel) |
| **5V** | → | **Vin** + **SD** + **GAIN** | Power + Enable |
| **GND** | → | **GND** | Ground |

### LED Ring
| ESP32 Pin | → | WS2812B | Function |
|-----------|---|---------|----------|
| **GPIO21** | → | **DI** | Data In |
| **5V** | → | **5V** | Power |
| **GND** | → | **GND** | Ground |

> ⚠️ **CRITICAL:**
> - Microphone **ONLY 3.3V** (5V will destroy it!)
> - **TWO SEPARATE I2S buses** - do not use the same pins!
> - 470µF capacitors at **each** amplifier and LED
> - Common ground (GND) for all components
> - SD and GAIN of amplifiers connected to 5V (full volume)

### 📐 Wiring Diagram

![Fritzing Schematic](docs/wiring_diagram.png)

> 📐 **Detailed text version:** See [docs/SCHEMATIC.md](docs/SCHEMATIC.md)

---

## 🚀 Installation (Home Assistant)

### Step 1: Prepare Secrets
Open `secrets.yaml` in ESPHome (click **"Secrets"** button in top right corner) and add:

```yaml
wifi_ssid: "Your_WiFi_SSID"
wifi_password: "Your_WiFi_Password"
api_key: "32_character_API_key"
ota_password: "OTA_password"
ap_password: "recovery_AP_password"
```

### Step 2: Create Device
1. In ESPHome Dashboard click **+ NEW DEVICE**
2. Name it: `esp32-s3-voice-satellite`
3. Click **SKIP** (skip the wizard)
4. Click **EDIT** on the device card

### Step 3: Upload Configuration
1. Copy **ENTIRE** `voice_assistant.yaml` file from this repo
2. Paste into the editor (overwrite default config)
3. **⚠️ CHANGE IP ADDRESS:**
   ```yaml
   manual_ip:
     static_ip: 10.0.50.177  # ← CHANGE to free address in your network!
     gateway: 10.0.50.1      # ← Your router
   ```
4. Click **SAVE** → **INSTALL**

### Step 4: Home Assistant Configuration
1. **Settings** → **Devices & Services** → device appears automatically
2. **Settings** → **Voice Assistants** → select pipeline (Whisper + Piper)
3. Expose entities to assistant (lights, switches, thermostats)

---

## 🎨 LED Effects - Details

| Effect | Color | Behavior | Trigger |
|--------|-------|----------|---------|
| **Boot Filling** | Cyan (0,255,200) | Progressive filling | Device startup |
| **Connection Breathe** | Green | Smooth breathing (sin) | API connection |
| **Listening Double** | Blue | 2 spinning points | Wake word detected |
| **Thinking Pulse** | Blue | Pulsing | End of speech (STT) |
| **Speaking Reverse** | Purple | Reverse spinning | Response start (TTS) |
| **Error** | Red | Solid | Pipeline error |
| **Mute** | Red | Solid 30% | Microphone muted |

---

## 📊 Technical Specifications

### Audio
- **Microphone:** 16kHz, 32-bit, Mono
- **Amplifiers:** 24kHz, 16-bit, Mono
- **Latency:** ~200ms (wake word → response)
- **Wake Word:** "Okay Nabu", threshold 0.35
- **VAD:** Hardware Voice Activity Detection

### Network
- **WiFi:** 2.4GHz, Static IP
- **Power Save:** Disabled (stability)
- **API:** AES encryption

### Performance
- **CPU:** 240MHz (ESP32-S3)
- **RAM:** 512KB + 8-16MB PSRAM (Octal 80MHz)
- **Flash:** 16MB with custom partition table
- **Temperature:** ~45-55°C under load

---

## 🐛 Troubleshooting

### 🔇 No sound
- Check amplifier power supply (5V, min. 2A)
- Ensure SD and GAIN are connected to 5V
- Add 470µF capacitors at amplifiers
- **Check if using two separate I2S buses**

### 🎤 Microphone not working
- ⚠️ **Check voltage:** VDD = 3.3V (NOT 5V!)
- L/R pin of microphone must be at GND
- Verify pinout: GPIO15/16/6

### 💡 LEDs not lighting
- Check direction: DI (input) → DO (output)
- LEDs require 5V (3.3V insufficient)
- Add 1000µF capacitor at LED power supply

### 📡 No WiFi connection
- Change `static_ip` to free address in your network
- Router must support 2.4GHz (ESP32 has no 5GHz)
- Check if SSID and password are correct in `secrets.yaml`

> 📖 **More:** See [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)

---

## 📂 Project Structure

```
esp32-s3-voice-satellite/
├── voice_assistant.yaml         # ← Main configuration file
├── secrets.yaml.example         # Secrets template
├── README.md                    # Documentation (PL)
├── README_EN.md                 # Documentation (EN)
├── LICENSE                      # MIT
├── CHANGELOG.md                 # Change history
├── CONTRIBUTING.md              # Contributor guidelines
│
├── docs/
│   ├── SCHEMATIC.md             # Wiring schematic
│   └── TROUBLESHOOTING.md       # Problem solving
│
├── 3d_models/
│   ├── pokrywa.stl
│   ├── maskownica.stl
│   ├── glosnik.stl
│   └── README.md
│
└── .github/
    ├── workflows/
    │   └── esphome-build.yml    # CI/CD
    └── ISSUE_TEMPLATE/
```

---

## 🤝 Credits

### Hardware & 3D Models
- **STL enclosure:** [Kristopher Mackowiak](https://github.com/KristopherMackowiak/ha_voice_assistant)
- **License:** MIT

### Firmware
- **Architecture:** Dual I2S Bus, Hardware VAD, Custom Partition Table
- **Optimizations:** ESP-IDF, PSRAM Octal, 32-bit audio
- **Author:** [Krzysztof / @KRISUUUU](https://github.com/KRISUUUU)

---

## 📜 License

MIT License - see [LICENSE](LICENSE) file for details

---

## 🌟 Support the Project

Developing this project involves hundreds of hours of coding and hardware testing. If you enjoy my work:

- ⭐ **Star the project** on GitHub (it's free and helps visibility!)
- 💖 **Become a Sponsor** via [GitHub Sponsors](https://github.com/sponsors/KRISUUUU) (0% fees)
- ☕ **Buy me a coffee** via [BuyCoffee.to](https://buycoffee.to/Krisuuuu) (BLIK/Przelewy24)

---

## 📊 Statistics

- **⭐ Stars:** Check current count on GitHub
- **🍴 Forks:** Community actively develops the project
- **🐛 Issues:** Average response time <24h

---

*Made with ❤️ for Home Assistant community*