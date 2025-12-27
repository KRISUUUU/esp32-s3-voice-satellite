# 🛠️ Troubleshooting Guide
ESP32-S3 Voice Satellite

This guide covers common issues and their solutions for the **ESP32-S3 Voice Satellite** project.

---

## 📡 WiFi Connection Issues

### Device won't connect to WiFi

**Symptoms:**
- Device appears offline in ESPHome Dashboard (red status)
- LED ring does not show the startup flash

**Solutions:**

- **Check credentials in `secrets.yaml`**  
  Ensure `wifi_ssid` and `wifi_password` are correct (**case-sensitive**).

- **Check WiFi band**  
  ESP32-S3 supports **2.4GHz only**.  
  Ensure your router is not locked to 5GHz-only mode.

- **Static IP conflict**  
  Default configuration uses:
  ```
  10.0.50.177
  ```
  If your network uses a different subnet (e.g. `192.168.1.x`), the device will not connect.

  ✅ **Fix:**  
  Update the `manual_ip` section in `voice_assistant.yaml` to match your network.

---

## 🔊 Audio Problems

### No sound from speaker

**Symptoms:**
- LED effects work correctly
- Voice Assistant runs (TTS visible in logs)
- Speaker is silent

**Solutions:**

- **Check Shared I2S wiring**
  - GPIO 15 → Amp BCLK **and** Mic SCK
  - GPIO 16 → Amp LRC **and** Mic WS
  - GPIO 11 → Amp DIN (both amplifiers)

- **Check Power Supply**
  - Amplifiers **require 5V**
  - Powering from 3.3V will result in silence or very low volume
  - PSU should provide **≥2A**

---

### Audio crackling / popping

**Symptoms:**
- Audible pops when LEDs turn on
- Distorted or unstable voice output

**Solutions:**

- **Install capacitors (CRUCIAL)**
  - 470µF directly across VIN/GND on:
    - Each amplifier
    - LED ring

- **Check shared ground**
  - ESP32, Mic, Amps, and LEDs **must share common GND**

---

### Microphone not picking up voice

**Symptoms:**
- Wake word never triggers
- Device appears deaf

**Solutions:**

- ⚠️ **CRITICAL – Check Voltage**
  - INMP441 **VDD must be 3.3V**
  - **5V will permanently destroy the microphone**

- **Check channel selection**
  - L/R pin on INMP441 **must be connected to GND**
  - (Left channel as expected by YAML config)

---

## 💡 LED Issues

### LEDs not lighting up

**Solutions:**

- **Check wiring**
  - GPIO 21 → DI (Data In)
  - Verify arrow direction on LED PCB (IN → OUT)

- **Check voltage**
  - WS2812B LEDs require **5V**
  - 3.3V is insufficient

---

### LEDs flickering randomly

**Solution:**

- Power instability detected:
  - Add **1000µF capacitor** across LED 5V/GND
  - Keep data line (GPIO 21) **shorter than 50cm**

---

## 🏠 Home Assistant Integration

### Device not appearing in Home Assistant

**Solutions:**

- **Manual Integration**
  1. Settings → Devices & Services
  2. Click **+ Add Integration**
  3. Select **ESPHome**
  4. Enter device IP (e.g. `10.0.50.177`)

- **API Encryption Key**
  - Ensure HA key matches the one in `secrets.yaml`

---

## 🔧 Performance Issues

### "Parent I2S bus not free" error

**Cause:**
- Audio configuration mismatch on shared I2S bus

**Solution:**
- Do **NOT** change `bits_per_sample`
- Must remain **32-bit** for both microphone and speaker
- Ensures proper clock synchronization

---

## 🛟 Still Need Help?

### Collect Logs
1. Connect device via USB
2. Open https://web.esphome.io
3. View serial logs

### Open an Issue
- Post logs and description in the GitHub repository
