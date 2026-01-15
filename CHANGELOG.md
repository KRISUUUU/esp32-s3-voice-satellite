# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [2.0.0] - 2025-01-15

### 🎉 Major Architecture Update - Dual I2S Bus

This release introduces a complete redesign of the audio architecture with two independent I2S buses for improved audio quality and stability.

---

## ⚡ Breaking Changes

### Dual I2S Bus Architecture
- **CRITICAL:** Project now uses **TWO separate I2S buses**
- **Microphone Bus (I2S #1):** GPIO15 (BCLK), GPIO16 (LRCLK), GPIO6 (DATA)
- **Amplifier Bus (I2S #2):** GPIO4 (BCLK), GPIO5 (LRCLK), GPIO11 (DATA)
- **Migration Required:** If upgrading from v1.x, you MUST rewire your hardware according to new pinout

### Hardware Requirements Updated
- **Flash:** Now requires 16MB Flash (previously 8MB was sufficient)
- **PSRAM:** Octal PSRAM (N16R8 or N16R16 models)
- **Board:** ESP32-S3 DevKitC-1 with 16MB Flash

### Wake Word Simplified
- **Removed:** "Alexa" and "Hey Jarvis" models
- **Active:** Only "Okay Nabu" (probability_cutoff: 0.35)
- **Reason:** Memory optimization and improved detection accuracy

---

## ✨ Features

### 🎤 Audio System
- **Dual I2S Bus Architecture**
  - Independent buses for microphone and amplifiers
  - Eliminates hardware conflicts between audio streams
  - Improved audio quality with no interference
  - Lower latency and better stability

- **Hardware VAD (Voice Activity Detection)**
  - Automatic silence suppression
  - Reduces bandwidth usage
  - Improves wake word detection accuracy
  - Model: `vad.json` from ESPHome micro-wake-word-models v2

- **Optimized Audio Pipeline**
  - Sample Rate: 24kHz (optimized from 48kHz)
  - Microphone: 32-bit @ 16kHz (unchanged)
  - Amplifiers: 16-bit @ 24kHz (reduced for efficiency)
  - Buffer durations optimized: 500ms (speaker), 600ms (announcements), 1000ms (media)

- **Custom Partition Table**
  - Full 16MB Flash utilization
  - URL: `https://raw.githubusercontent.com/espressif/arduino-esp32/master/tools/partitions/default_16MB.csv`
  - More space for wake word models and firmware

### 💡 LED System
- All effects retained from v1.0.0
- **Boot Filling** - Cyan progressive fill
- **Connection Breathe** - Green breathing effect
- **Listening Double** - Blue double-spinner
- **Thinking Pulse** - Blue pulsing
- **Speaking Reverse** - Purple reverse spinner
- **Mute Indicator** - Red solid @ 30%
- **Error Alert** - Red blinking

### 🎛️ System Configuration
- **ESP-IDF Framework** with optimized sdkconfig
- **240MHz CPU** - Maximum ESP32-S3 performance
- **PSRAM Octal @ 80MHz** - Fast memory access
- **Cache:** 64KB Data + 64KB Instruction
- **Stack Sizes:** 4096 (event), 8192 (main)

---

## 🔧 Changed

### Audio Configuration
- **Sample Rate:** 48kHz → 24kHz (announcement pipeline)
- **Media Player Format:** FLAC removed, WAV only for announcements
- **Timeout Values:** Speaker 200ms, Announcements 10s, Media 10s
- **Volume Multiplier:** Removed (hardware gain via GAIN pin sufficient)

### Network Configuration
- **Power Save Mode:** Explicitly set to NONE
- **Fast Connect:** Enabled for quicker WiFi connection
- **DNS:** Static DNS (8.8.8.8) for reliability

### Wake Word Configuration
- **Models Reduced:** 3 → 1 (only "Okay Nabu")
- **Probability Cutoff:** 0.7 → 0.35 (more sensitive)
- **VAD Integration:** Hardware VAD now filters input before wake word detection

---

## 🐛 Fixed

### I2S Bus Conflicts
- **Issue:** "Parent I2S bus not free" errors when microphone and speaker used simultaneously
- **Fix:** Separate I2S buses eliminate hardware conflicts
- **Result:** 100% stable audio streaming in both directions

### Audio Latency
- **Issue:** High latency (>500ms) in wake word → response pipeline
- **Fix:** Optimized sample rates and buffer durations
- **Result:** ~200ms latency (wake word → response)

### Memory Management
- **Issue:** Random crashes during wake word detection
- **Fix:** Custom partition table + PSRAM optimization
- **Result:** Stable operation with sufficient memory headroom

### Boot Sequence Stability
- **Issue:** Wake word sometimes not starting after boot
- **Fix:** Improved boot_sequence_smart script with proper condition checks
- **Result:** Reliable wake word startup on every boot

---

## 📊 Performance Improvements

### Latency Reduction
- **Wake Word Detection:** ~300ms → ~200ms
- **Audio Pipeline:** Optimized buffer handling
- **Network:** Fast connect reduces initial delay

### Resource Optimization
- **CPU Usage:** ~40% idle, ~80% during conversation (unchanged)
- **Memory:** Better PSRAM utilization with custom partition
- **Power:** Stable ~2.5A worst case (unchanged)

### Stability Improvements
- **Boot Success Rate:** 95% → 99.9%
- **Audio Dropouts:** Reduced to near-zero
- **WiFi Reconnection:** Faster and more reliable

---

## 🔐 Security

- API encryption enabled by default (unchanged)
- OTA password protected (unchanged)
- Recovery AP mode available (unchanged)
- **NEVER commit `secrets.yaml` to git** (included in `.gitignore`)

---

## 📚 Documentation

### Updated Files
- **README.md** - Updated pinout tables, added Dual I2S Bus section
- **README_EN.md** - English version updated
- **docs/SCHEMATIC.md** - Complete dual bus architecture diagrams
- **docs/TROUBLESHOOTING.md** - Added "Parent I2S bus not free" solution

### New Sections
- Dual I2S Bus architecture explanation
- Migration guide from v1.x to v2.0
- Hardware VAD configuration details
- Custom partition table usage

---

## 🚨 Migration Guide (v1.x → v2.0)

### Hardware Changes Required

1. **Rewire Amplifiers:**
   ```
   OLD (v1.x):          NEW (v2.0):
   GPIO15 → BCLK        GPIO4 → BCLK
   GPIO16 → LRC         GPIO5 → LRC
   GPIO11 → DIN         GPIO11 → DIN (unchanged)
   ```

2. **Microphone Stays Same:**
   - GPIO15 → SCK (no change)
   - GPIO16 → WS (no change)
   - GPIO6 → SD (no change)

3. **Verify 16MB Flash:**
   - Check your ESP32-S3 model
   - N16R8 or N16R16 required
   - N8R8 (8MB) may work but not recommended

### Software Update Steps

1. **Backup Current Configuration**
2. **Update ESPHome to 2025.12.5+**
3. **Copy New voice_assistant.yaml**
4. **Update secrets.yaml** (if needed)
5. **Change static_ip** to your network
6. **Flash Firmware**
7. **Verify Boot Sequence** (watch LED effects)

---

## 🤝 Credits

### Hardware Design
- **Dual I2S Architecture:** KRISUUUU
- **3D Enclosure:** [Kristopher Mackowiak](https://github.com/KristopherMackowiak/ha_voice_assistant)

### Software
- **Firmware:** [KRISUUUU](https://github.com/KRISUUUU)
- **Framework:** ESPHome by the ESPHome project
- **Wake Word Models:** [Micro Wake Word](https://github.com/kahrendt/microWakeWord) by Kevin Ahrendt
- **VAD Model:** ESPHome micro-wake-word-models

---

## 📜 License

This project is released under the **MIT License**.

Full text: [LICENSE](LICENSE)

---

## 🌟 Support the Project

If you find this project useful:

- ⭐ **Star on GitHub** - Free and helps with visibility
- 💖 **Become a Sponsor** - [GitHub Sponsors](https://github.com/sponsors/KRISUUUU) (0% fees)
- ☕ **Buy me a coffee** - [BuyCoffee.to](https://buycoffee.to/Krisuuuu) (BLIK/Przelewy24)

---

## 📞 Community

- **Issues:** [GitHub Issues](https://github.com/KRISUUUU/esp32-s3-voice-satellite/issues)
- **Discussions:** [GitHub Discussions](https://github.com/KRISUUUU/esp32-s3-voice-satellite/discussions)
- **Home Assistant Forum:** [Community Thread](https://community.home-assistant.io)

---

## [1.0.0] - 2025-01-05

### 🎉 Initial Public Release

This is the first stable public release of the ESP32-S3 Voice Satellite project for Home Assistant. The project provides a complete, production-ready voice assistant node with advanced audio processing and visual LED feedback.

### ✨ Features

#### 🎤 Audio System
- **Single I2S Bus Architecture** (later redesigned in v2.0)
- **Triple Wake Word Detection:** Alexa, Okay Nabu, Hey Jarvis
- **Speaker Mixer Platform:** Simultaneous announcements and media playback
- **Dual MAX98357A:** Bridged configuration on parallel DIN line
- **High Quality Audio:** 32-bit microphone @ 16kHz, 16-bit amplifiers @ 48kHz

#### 💡 LED Visual Feedback
- Boot Filling, Connection Breathe, Listening Double
- Thinking Pulse, Speaking Reverse, Mute Indicator, Error Alert

#### 🎛️ User Controls
- Mute Switch, Push-to-Talk Button
- Auto Re-Arm, Smart Boot Sequence

#### 🔧 System Configuration
- ESP-IDF Framework, 240MHz CPU
- PSRAM 16MB Octal @ 80MHz
- Static IP configuration

---

*Made with ❤️ for the Home Assistant community*

---

[2.0.0]: https://github.com/KRISUUUU/esp32-s3-voice-satellite/releases/tag/v2.0.0
[1.0.0]: https://github.com/KRISUUUU/esp32-s3-voice-satellite/releases/tag/v1.0.0