[🇵🇱 Wersja polska](README.md) | 🇬🇧 English version

# ESP32-S3 Voice Satellite for Home Assistant

![Project Status](https://img.shields.io/badge/Status-Beta-yellow)
![ESPHome](https://img.shields.io/badge/ESPHome-2024.12.0+-blue)
![C++](https://img.shields.io/badge/Native_Logic-C++-orange)
![License](https://img.shields.io/badge/License-MIT-grey)
[![BuyCoffee](https://img.shields.io/badge/BuyCoffee.to-Support-green?logo=buy-coffee&logoColor=white)](https://buycoffee.to/Krisuuuu)

> ⚠️ **Project Status:** This is an early working version under active development. Core features are stable, but new functionality is being added frequently.

Advanced voice satellite for Home Assistant based on ESP32-S3. The project utilizes the I2S protocol for high-quality audio (Dual Mono - high volume) and a WS2812B LED ring for visual feedback.

## ✨ Features
- **Local Wake Word:** Supports "Okay Nabu" (Micro Wake Word) directly on the device.
- **Dual Mono Audio:** Supports two MAX98357A amplifiers on a single data line.
- **Stability:** Optimized I2S configuration (32-bit) and power filtering.
- **Visual Feedback:** LED animations for states: Listening, Thinking, Speaking, Error.
- **Push-to-Talk:** Home Assistant button for manual assistant triggering.

---

## 🛠️ Hardware (BOM)

| Component | Description |
| :--- | :--- |
| **MCU** | ESP32-S3 DevKitC-1 (N8R16 - 8MB Flash / 16MB PSRAM) |
| **Microphone** | INMP441 (I2S MEMS) |
| **Amplifier** | 2x MAX98357A (Bridged DIN) |
| **LED** | WS2812B Ring (12 LEDs) |
| **Power** | 5V 3A PSU (min.) + Fuse |
| **Filtering** | 470µF Capacitors on power rails (LED, Amps) |

---

## 🔌 Pinout / Wiring

All audio devices operate on a Shared Bus.

| ESP32 Pin | Device | Device Pin | Function |
| :--- | :--- | :--- | :--- |
| **GPIO 15** | Mic + 2x Amp | SCK / BCLK | **Bit Clock (Shared)** |
| **GPIO 16** | Mic + 2x Amp | WS / LRC | **Word Select (Shared)** |
| **GPIO 6** | Mic | SD | Mic Data In |
| **GPIO 11** | 2x Amp | DIN | Amp Data Out (Shared) |
| **GPIO 21** | LED | DI | LED Data (Direct) |

> **NOTE:** Ensure a common Ground (GND) connection for all components!

> Full schematic available in [docs/schematic.md](docs/SCHEMATIC.md%20)

---

## 🚀 Installation (Home Assistant)

This project is designed to be used directly with the **ESPHome** add-on in Home Assistant.

### Step 1: Verify Secrets
Your `secrets.yaml` file in ESPHome might already contain configuration. Open it (click **"Secrets"** in the top right corner of the dashboard) and ensure the following variables are defined.

**Required variables for this project:**
(If missing, add them. If they already exist, do not duplicate them).

wifi_ssid: "Your_WiFi_SSID"
wifi_password: "Your_WiFi_Password"
api_key: "Generate_Long_Password_Here"
ota_password: "Generate_Another_Password_Here"
ap_password: "StrongPasswordForRecoveryAP"

### Step 2: Create Device
1. In ESPHome Dashboard click **+ NEW DEVICE**.
2. Name it: `esp32-s3-voice-satellite`.
3. Click **SKIP** to skip the wizard.
4. Click **EDIT** on the newly created device card.

### Step 3: Flash Code
1. Copy the entire content of `voice_assistant.yaml` from this repository.
2. Paste it into the ESPHome editor (overwrite default config).
3. **IMPORTANT:** Find the `manual_ip` section and enter an IP address that is **free** in your local network (check your router!).
4. Click **SAVE** and **INSTALL**.

### Step 4: Home Assistant Configuration
1. **Settings** → **Devices & Services** → device appears automatically
2. **Settings** → **Voice Assistants** → select pipeline
3. Expose entities to assistant (lights, switches, etc.)

---

## ⚠️ Important Notes

* **IP Address:** The code uses a static IP for connection stability. Adjust it to match your subnet (e.g., `192.168.1.xxx` or `10.0.x.x`).
* **Power Supply:** Stable 5V (min 2A-3A) is required. Do not power via PC USB when using LEDs and speakers.
* **Capacitors:** Electrolytic capacitors (e.g., 470µF) are recommended at the amplifier and LED power pins to prevent audio glitches and reboots.

---

## 📂 Project Structure

```
ha-voice-satellite/
├── README.md                    # Polish version
├── README_EN.md                 # This file
├── LICENSE                      # MIT License
├── .gitignore                   
├── secrets.yaml.example         # Template
│
├── voice_assistant.yaml         # ← Main installation file
│
├── docs/
│   ├── schematic.svg            # Graphical schematic
│   ├── SCHEMATIC.md             # Text schematic + tables
│   └── TROUBLESHOOTING.md       # Problem solving
│
└── 3d_models/
    ├── pokrywa.stl              # 3D printable models
    ├── maskownica.stl
    ├── glosnik.stl
    └── README.md                # Models info
```
---


## 🎨 LED Effects

| Effect | Trigger | Implementation |
|--------|---------|----------------|
| **Listening** | Wake word detected | Cyan, breathing via `sin()` |
| **Thinking** | End of speech | Blue spinner |
| **Speaking** | TTS start | Green simulation |
| **Error** | Error | Red blinking |

---


## 🐛 Problems?

Common issues and solutions in [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)

---

## 🤝 Credits

### Hardware & 3D Models
- **STL enclosure:** [Kristopher Mackowiak](https://github.com/KristopherMackowiak/ha_voice_assistant)
- **License:** MIT

### Firmware
- **Architecture:** Clean Code, Separation of Concerns, HAL
- **Optimizations:** Native C++, I2S configuration
- **Author:** [Krzysztof / @KRISUUUU](https://github.com/KRISUUUU)

---

## 📜 License

MIT License - see [LICENSE](LICENSE) file for details

---

## 🌟 Support the Project

Developing this project involves hundreds of hours of coding and hardware testing. If you enjoy my work, please consider supporting me:

- ⭐ **Star the project** on GitHub (it's free and helps visibility!)
- 💖 **Become a Sponsor** via [GitHub Sponsors](https://github.com/sponsors/KRISUUUU) (0% fees, monthly support)
- ☕ **Buy me a coffee** via [BuyCoffee.to](https://buycoffee.to/Krisuuuu) (one-time support via BLIK/Przelewy24)

---

*Made for Home Assistant community*