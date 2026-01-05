# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] - 2025-01-05

### 🎉 Initial Public Release

This is the first stable public release of the ESP32-S3 Voice Satellite project for Home Assistant. The project provides a complete, production-ready voice assistant node with advanced audio processing and visual LED feedback.

---

## ✨ Features

### 🎤 Audio System
- **Dual I2S Bus Architecture**
  - Independent buses for microphone and amplifiers
  - Microphone: GPIO15 (BCLK), GPIO16 (LRCLK), GPIO6 (DATA)
  - Amplifiers: GPIO4 (BCLK), GPIO5 (LRCLK), GPIO11 (DATA)
  - 32-bit microphone @ 16kHz for high-quality capture
  - 16-bit amplifiers @ 48kHz for clear playback

- **Speaker Mixer Platform**
  - Simultaneous announcements and media playback
  - Announcement channel: FLAC 48kHz (voice responses)
  - Media channel: 48kHz (music/audio)
  - Dual MAX98357A amplifiers on parallel DIN line

- **Triple Wake Word Detection**
  - "Alexa" - Amazon-style wake word
  - "Okay Nabu" - Home Assistant native
  - "Hey Jarvis" - Iron Man style
  - Local on-device processing with Micro Wake Word
  - Configurable threshold: 0.7 (adjustable)

- **Media Player Integration**
  - Native ESPHome media player component
  - Announcement pipeline with FLAC support
  - Media pipeline for music/audio streaming
  - Volume multiplier: 4.0 for voice clarity

### 💡 LED Visual Feedback (WS2812B)
All effects implemented as native YAML lambdas:

- **Boot Filling** - Cyan progressive fill (3.6s startup animation)
- **Connection Breathe** - Green breathing on API connection
- **Listening Double** - Blue double-spinner when wake word detected
- **Thinking Pulse** - Blue pulsing during speech processing
- **Speaking Reverse** - Purple reverse spinner during response
- **Mute Indicator** - Red solid @ 30% when microphone muted
- **Error Alert** - Red blinking on pipeline errors

### 🎛️ User Controls
- **Mute Switch** - Microphone mute with visual indicator
- **Push-to-Talk Button** - Manual assistant trigger from HA
- **Auto Re-Arm** - Automatic wake word restart after conversation
- **Smart Boot Sequence** - Intelligent startup with connection detection

### 🔧 System Configuration
- **ESP-IDF Framework** - Production-grade stability
- **240MHz CPU** - ESP32-S3 maximum performance
- **PSRAM** - 16MB Octal @ 80MHz for audio buffering
- **Static IP** - Reliable network addressing (default: 10.0.50.177)
- **Power Save Disabled** - Maximum stability for audio processing

---

## 🛠️ Hardware Requirements

### Core Components
- **ESP32-S3 DevKitC-1 (N8R16)** - 8MB Flash, 16MB PSRAM
- **INMP441** - I2S MEMS microphone
- **2x MAX98357A** - Class D amplifiers (bridged configuration)
- **WS2812B Ring** - 12 addressable LEDs
- **5V 3A Power Supply** - Regulated (minimum 2.5A)

### Critical Power Filtering
- **3x 470µF capacitors** - One per MAX98357A + one for LED
- **Common ground** - All components must share GND

### Amplifier Configuration
- **SD pins → 5V** - Enable + mono mix (L+R)
- **GAIN pins → 5V** - Maximum volume (15dB)

---

## 📋 Complete Pinout

| Component | ESP32 Pin | Device Pin | Function | Voltage |
|-----------|-----------|------------|----------|---------|
| **Microphone (INMP441)** |
| | GPIO15 | SCK | Bit Clock | - |
| | GPIO16 | WS | Word Select | - |
| | GPIO6 | SD | Data In | - |
| | 3.3V | VDD | Power | ⚠️ 3.3V ONLY |
| | GND | GND + L/R | Ground + Left Channel | - |
| **Amplifiers (2x MAX98357A)** |
| | GPIO4 | BCLK | Bit Clock | - |
| | GPIO5 | LRC | Word Select | - |
| | GPIO11 | DIN (both) | Data Out (parallel) | - |
| | 5V | Vin + SD + GAIN | Power + Enable + Max Vol | 5V |
| | GND | GND | Ground | - |
| **LED Ring (WS2812B)** |
| | GPIO21 | DI | Data In | - |
| | 5V | 5V | Power | 5V |
| | GND | GND | Ground | - |

---

## 🚀 Quick Start

### 1. Hardware Assembly
1. Wire according to pinout table above
2. **CRITICAL:** Microphone VDD = 3.3V only (5V will destroy it)
3. Install 470µF capacitors at each amplifier Vin-GND
4. Install 1000µF capacitor at LED 5V-GND
5. Ensure common ground for all components

### 2. ESPHome Setup
1. Create `secrets.yaml` with WiFi credentials and API keys
2. Edit `static_ip` in `voice_assistant.yaml` to match your network
3. Flash firmware via ESPHome Dashboard
4. Device will appear automatically in Home Assistant

### 3. Home Assistant Configuration
1. Go to **Settings → Voice Assistants**
2. Select pipeline (Whisper STT + Piper TTS recommended)
3. Expose entities you want to control via voice

---

## 📚 Documentation

- **README.md** - Complete project overview (Polish)
- **README_EN.md** - English version
- **docs/SCHEMATIC.md** - Detailed wiring diagrams
- **docs/TROUBLESHOOTING.md** - Common issues and solutions

---

## 🔐 Security

- API encryption enabled by default
- OTA password protected
- Recovery AP mode available
- **NEVER commit `secrets.yaml` to git** (included in `.gitignore`)

---

## 📊 Performance Metrics

- **Wake Word Latency:** ~300ms (detection to activation)
- **Audio Latency:** ~200ms (speech to TTS start)
- **Boot Time:** ~6 seconds (full startup)
- **CPU Usage:** ~40% idle, ~80% during conversation
- **Power Draw:** ~2.5A worst case (full LEDs + max audio)

---

## 🤝 Credits

### Hardware Design
- **3D Enclosure:** [Kristopher Mackowiak](https://github.com/KristopherMackowiak/ha_voice_assistant)
- **Schematic:** Circuit design by KRISUUUU

### Software
- **Firmware:** [KRISUUUU](https://github.com/KRISUUUU)
- **Framework:** ESPHome by the ESPHome project
- **Wake Word Models:** [Micro Wake Word](https://github.com/kahrendt/microWakeWord) by Kevin Ahrendt

---

## 📜 License

This project is released under the **MIT License**.

```
MIT License

Copyright (c) 2025 KRISUUUU

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
```

Full text: [LICENSE](LICENSE)

---

## 🌟 Support the Project

If you find this project useful:

- ⭐ **Star on GitHub** - Free and helps with visibility
- 💖 **Become a Sponsor** - [GitHub Sponsors](https://github.com/sponsors/KRISUUUU) (0% fees)
- ☕ **Buy me a coffee** - [BuyCoffee.to](https://buycoffee.to/Krisuuuu) (BLIK/Przelewy24)

---

## 🔮 Roadmap

Future enhancements being considered:

- [ ] Custom wake word training support
- [ ] Volume control via rotary encoder
- [ ] Battery power option with sleep modes
- [ ] Multiple LED ring sizes support
- [ ] Web-based configuration interface
- [ ] Multi-room audio synchronization

---

## 📞 Community

- **Issues:** [GitHub Issues](https://github.com/KRISUUUU/esp32-s3-voice-satellite/issues)
- **Discussions:** [GitHub Discussions](https://github.com/KRISUUUU/esp32-s3-voice-satellite/discussions)
- **Home Assistant Forum:** [Community Thread](https://community.home-assistant.io)

---

*Made with ❤️ for the Home Assistant community*

---

[1.0.0]: https://github.com/KRISUUUU/esp32-s3-voice-satellite/releases/tag/v1.0.0