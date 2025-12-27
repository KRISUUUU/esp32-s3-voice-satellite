# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-12-27

### 🚀 Initial Stable Release
This is the first production-ready release, completely rewritten for stability and performance.

### ✨ New Features
- **Architecture:** Transitioned to a robust single-file YAML configuration (Monolithic) to eliminate compilation errors.
- **Audio Engine:**
    - Implemented **Dual Mono** support (2x MAX98357A on bridged DIN line).
    - Enforced **32-bit I2S** configuration to eliminate audio crackling/pops.
    - **Shared Bus** topology (Microphone and Amplifiers share BCLK/WS lines).
- **Voice Assistant Logic:**
    - **Force Re-Arm:** Added logic to automatically restart the Wake Word engine after a conversation ends (fixing the "one-shot" bug).
    - **Push-to-Talk:** Added a manual trigger button in Home Assistant.
    - **Visual Feedback:** Integrated native YAML Lambdas for LED effects (Listening, Thinking, Speaking, Error) - removed dependency on external C++ headers.
- **Hardware Config:**
    - **Simplified Wiring:** Removed requirement for Logic Level Shifter (WS2812B connected directly to GPIO).
    - **Network:** Implemented Static IP configuration for connection reliability.

### 🐛 Fixed
- Fixed `rmt_channel` compilation error in ESPHome 2025.12.0+.
- Fixed audio synchronization issues by switching from 16-bit to 32-bit samples.
- Fixed LED flickering caused by incorrect logic level converter usage.
- Fixed "Parent I2S bus not free" errors by using a unified I2S configuration.

### 📚 Documentation
- Complete rewrite of README (PL & EN) focused on Home Assistant Dashboard installation.
- Updated wiring diagrams (text-based) and BOM to reflect the removal of the Level Shifter.
- Added "Secrets" verification step to installation guide.

---
[1.0.0]: https://github.com/KRISUUUU/esp32-s3-voice-satellite/releases/tag/v1.0.0