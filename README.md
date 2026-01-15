🇵🇱 Wersja polska | [🇬🇧 English version](README_EN.md)

# ESP32-S3 Voice Satellite dla Home Assistant

![Status Projektu](https://img.shields.io/badge/Status-Stable-green)
![ESPHome](https://img.shields.io/badge/ESPHome-2025.12.5+-blue)
![Framework](https://img.shields.io/badge/Framework-ESP--IDF-orange)
![Licencja](https://img.shields.io/badge/licencja-MIT-grey)
[![BuyCoffee](https://img.shields.io/badge/BuyCoffee.to-Wspieraj-green?logo=buy-me-a-coffee&logoColor=white)](https://buycoffee.to/Krisuuuu)

> ✅ **Status Projektu:** Stabilna wersja produkcyjna 2.0.0 z pełnym wsparciem media player, zaawansowanymi efektami LED i **podwójną magistralą I2S**.

> 💡 **Zaawansowany projekt hobbystyczny stworzony z pasji (Built with passion).**
>
> **Ważna nota od autora:** Nie jestem zawodowym programistą – projekt rozwijam przy wsparciu AI. To wynik moich eksperymentów i pasji do automatyki domowej. Kod jest stabilny i sprawdzony w moim własnym domu, ale jeśli jesteś deweloperem i widzisz pole do poprawek – Twoje Pull Requesty są mile widziane!

"Zaawansowany" satelita głosowy dla Home Assistant oparty na ESP32-S3. Projekt wykorzystuje **dwie niezależne magistrale I2S** dla dźwięku wysokiej jakości oraz pierścień LED WS2812B dla wizualnej informacji zwrotnej.

---

## ✨ Funkcje

### 🎤 Audio
- **Wake Word:** "Okay Nabu" (lokalna detekcja z Hardware VAD)
- **Dual I2S Bus Architecture:** Oddzielne magistrale dla mikrofonu i wzmacniaczy
- **Dual Mono Output:** 2x MAX98357A na wspólnej linii danych (zwiększona głośność)
- **Media Player:** Jednoczesne odtwarzanie ogłoszeń i muzyki (mixer)
- **Wysoka jakość:** 32-bit mikrofon (16kHz), 16-bit wzmacniacze (24kHz)
- **Hardware VAD:** Sprzętowa detekcja aktywności głosowej dla eliminacji ciszy
- **Optymalizacja:** Niskie opóźnienia dzięki zoptymalizowanemu pipeline audio (24kHz WAV)

### 💡 LED (WS2812B)
- **Boot Filling:** Stopniowe zapełnianie przy starcie (cyan)
- **Connection Breathe:** Zielony oddech przy połączeniu
- **Listening Double:** Niebieski wirujący efekt (2 punkty)
- **Thinking Pulse:** Niebieski pulsujący efekt
- **Speaking Reverse:** Fioletowy odwrotny wirujący efekt
- **Error:** Czerwone mruganie
- **Mute:** Czerwony stały 30%

### 🎛️ Sterowanie
- **Mute Switch:** Wyciszenie mikrofonu (czerwony LED)
- **Push-to-Talk:** Przycisk w Home Assistant
- **Auto Re-Arm:** Automatyczne ponowne uruchomienie wake word
- **Smart Boot Sequence:** Inteligentna sekwencja startowa z detekcją połączenia

---

## 🆕 Co nowego w wersji 2.0.0?

### Architektura Dual I2S Bus
Projekt został przeprojektowany na **dwie niezależne magistrale I2S**:
- **Magistrala #1:** Mikrofon INMP441 (GPIO15, GPIO16, GPIO6)
- **Magistrala #2:** Wzmacniacze MAX98357A (GPIO4, GPIO5, GPIO11)

### Inne kluczowe zmiany:
- ✅ Hardware VAD dla eliminacji ciszy
- ✅ Custom partition table dla 16MB Flash
- ✅ Zoptymalizowany pipeline audio (24kHz)
- ✅ Uproszczony wake word (tylko "Okay Nabu")
- ✅ Ulepszone bufory audio dla stabilności

📋 **Pełna lista zmian:** Zobacz [CHANGELOG.md](CHANGELOG.md)

---

## 🛠️ Sprzęt (BOM)

| Element | Model | Ilość | Uwagi |
|---------|-------|-------|-------|
| **MCU** | ESP32-S3 DevKitC-1 N16R8/N16R16 | 1 | 16MB Flash / 8-16MB PSRAM (Octal) |
| **Mikrofon** | INMP441 | 1 | I2S MEMS |
| **Wzmacniacz** | MAX98357A | 2 | Równolegle na GPIO11 |
| **LED** | WS2812B Ring | 1 | 12 diod |
| **Zasilacz** | 5V 3A | 1 | Min. 2.5A |
| **Kondensatory** | 470µF 16V | 3 | Przy każdym wzmacniaczu + LED |
| **Bezpiecznik** | 3A | 1 | Opcjonalnie |

---

## 🔌 Schemat Podłączenia (Pinout)

### Magistrala Mikrofonu (I2S Bus #1)
| Pin ESP32 | → | Mikrofon INMP441 | Funkcja |
|-----------|---|------------------|---------|
| **GPIO15** | → | **SCK** | Bit Clock |
| **GPIO16** | → | **WS** | Word Select |
| **GPIO6** | → | **SD** | Data In |
| **3.3V** | → | **VDD** | Zasilanie ⚠️ |
| **GND** | → | **GND** | Masa |
| **GND** | → | **L/R** | Kanał lewy |

### Magistrala Wzmacniaczy (I2S Bus #2)
| Pin ESP32 | → | 2x MAX98357A | Funkcja |
|-----------|---|--------------|---------|
| **GPIO4** | → | **BCLK** | Bit Clock |
| **GPIO5** | → | **LRC** | Word Select |
| **GPIO11** | → | **DIN** (oba) | Data Out (równolegle) |
| **5V** | → | **Vin** + **SD** + **GAIN** | Zasilanie + Enable |
| **GND** | → | **GND** | Masa |

### LED Ring
| Pin ESP32 | → | WS2812B | Funkcja |
|-----------|---|---------|---------|
| **GPIO21** | → | **DI** | Data In |
| **5V** | → | **5V** | Zasilanie |
| **GND** | → | **GND** | Masa |

> ⚠️ **KRYTYCZNE:**
> - Mikrofon **TYLKO 3.3V** (5V zniszczy układ!)
> - **DWA OSOBNE magistrale I2S** - nie używaj tych samych pinów!
> - Kondensatory 470µF przy **każdym** wzmacniaczu i LED
> - Wspólna masa (GND) dla wszystkich komponentów
> - SD i GAIN wzmacniaczy podłączone do 5V (pełna głośność)

### 📐 Schemat Połączeń

![Schemat Fritzing](docs/wiring_diagram.png)

> 📐 **Szczegóły tekstowe:** Zobacz [docs/SCHEMATIC.md](docs/SCHEMATIC.md)

---

## 🚀 Instalacja (Home Assistant)

### Krok 1: Przygotuj Secrets
Otwórz plik `secrets.yaml` w ESPHome (przycisk **"Secrets"** w prawym górnym rogu) i dodaj (czego brakuje):

```yaml
wifi_ssid: "Twoja_Nazwa_WiFi"
wifi_password: "Twoje_Haslo_WiFi"
api_key: "32_znakowy_klucz_API"
ota_password: "haslo_do_OTA"
ap_password: "haslo_recovery_AP"
```

### Krok 2: Utwórz Urządzenie
1. W ESPHome Dashboard kliknij **+ NEW DEVICE**
2. Nadaj nazwę: `esp32-s3-voice-satellite`
3. Kliknij **SKIP** (pomijamy kreatora)
4. Kliknij **EDIT** na karcie urządzenia

### Krok 3: Wgraj Konfigurację
1. Skopiuj **CAŁY** plik `voice_assistant.yaml` z tego repo
2. Wklej do edytora (nadpisz domyślną konfigurację)
3. **⚠️ ZMIEŃ ADRES IP:**
   ```yaml
   manual_ip:
     static_ip: 10.0.50.177  # ← ZMIEŃ na wolny adres w Twojej sieci!
     gateway: 10.0.50.1      # ← Twój router
   ```
4. Kliknij **SAVE** → **INSTALL**

### Krok 4: Konfiguracja w Home Assistant
1. **Ustawienia** → **Urządzenia i usługi** → urządzenie pojawi się automatycznie
2. **Ustawienia** → **Asystenci głosowi** → wybierz pipeline (Whisper + Piper)
3. Wystaw encje do asystenta (światła, przełączniki, termostaty)

---

## 🎨 Efekty LED - Szczegóły

| Efekt | Kolor | Zachowanie | Wyzwalacz |
|-------|-------|------------|-----------|
| **Boot Filling** | Cyan (0,255,200) | Stopniowe zapełnianie | Start urządzenia |
| **Connection Breathe** | Zielony | Płynny oddech (sin) | Połączenie API |
| **Listening Double** | Niebieski | 2 wirujące punkty | Wykrycie wake word |
| **Thinking Pulse** | Niebieski | Pulsowanie | Koniec mowy (STT) |
| **Speaking Reverse** | Fioletowy | Odwrotny wirujący | Start odpowiedzi (TTS) |
| **Error** | Czerwony | Stały | Błąd pipeline |
| **Mute** | Czerwony | Stały 30% | Mikrofon wyciszony |

---

## 📊 Parametry Techniczne

### Audio
- **Mikrofon:** 16kHz, 32-bit, Mono
- **Wzmacniacze:** 24kHz, 16-bit, Mono
- **Latency:** ~200ms (wake word → odpowiedź)
- **Wake Word:** "Okay Nabu", próg 0.35
- **VAD:** Hardware Voice Activity Detection

### Sieć
- **WiFi:** 2.4GHz, Static IP
- **Power Save:** Wyłączone (stabilność)
- **API:** Szyfrowanie AES

### Wydajność
- **CPU:** 240MHz (ESP32-S3)
- **RAM:** 512KB + 8-16MB PSRAM (Octal 80MHz)
- **Flash:** 16MB z custom partition table
- **Temperatura:** ~45-55°C przy obciążeniu

---

## 🐛 Rozwiązywanie Problemów

### 🔇 Brak dźwięku
- Sprawdź zasilanie wzmacniaczy (5V, min. 2A)
- Upewnij się, że SD i GAIN są podłączone do 5V
- Dodaj kondensatory 470µF przy wzmacniaczach
- **Sprawdź czy używasz dwóch osobnych magistral I2S**

### 🎤 Mikrofon nie działa
- ⚠️ **Sprawdź napięcie:** VDD = 3.3V (NIE 5V!)
- L/R pin mikrofonu musi być na GND
- Zweryfikuj pinout: GPIO15/16/6

### 💡 LED nie świecą
- Sprawdź kierunek: DI (wejście) → DO (wyjście)
- LED wymagają 5V (3.3V nie wystarczy)
- Dodaj kondensator 1000µF przy zasilaniu LED

### 📡 Brak połączenia WiFi
- Zmień `static_ip` na wolny adres w Twojej sieci
- Router musi obsługiwać 2.4GHz (ESP32 nie ma 5GHz)
- Sprawdź czy SSID i hasło są poprawne w `secrets.yaml`

> 📖 **Więcej:** Zobacz [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)

---

## 📂 Struktura Projektu

```
esp32-s3-voice-satellite/
├── voice_assistant.yaml         # ← Główny plik konfiguracji
├── secrets.yaml.example         # Szablon secrets
├── README.md                    # Dokumentacja (PL)
├── README_EN.md                 # Dokumentacja (EN)
├── LICENSE                      # MIT
├── CHANGELOG.md                 # Historia zmian
├── CONTRIBUTING.md              # Wytyczne dla kontrybutorów
│
├── docs/
│   ├── SCHEMATIC.md             # Schemat połączeń
│   └── TROUBLESHOOTING.md       # Rozwiązywanie problemów
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

## 🤝 Podziękowania

### Hardware i Modele 3D
- **Obudowa STL:** [Kristopher Mackowiak](https://github.com/KristopherMackowiak/ha_voice_assistant)
- **Licencja:** MIT

### Firmware
- **Architektura:** Dual I2S Bus, Hardware VAD, Custom Partition Table
- **Optymalizacje:** ESP-IDF, PSRAM Octal, 32-bit audio
- **Autor:** [Krzysztof / @KRISUUUU](https://github.com/KRISUUUU)

---

## 📜 Licencja

MIT License - szczegóły w pliku [LICENSE](LICENSE)

---

## 🌟 Wspieraj Projekt

Rozwój projektu to setki godzin kodowania i testowania sprzętu. Jeśli podoba Ci się to, co robię:

- ⭐ **Daj Gwiazdkę** na GitHubie (to darmowe i pomaga zasięgom!)
- 💖 **Zostań Sponsorem** przez [GitHub Sponsors](https://github.com/sponsors/KRISUUUU) (0% prowizji)
- ☕ **Postaw Kawę** przez [BuyCoffee.to](https://buycoffee.to/Krisuuuu) (BLIK/Przelewy24)

---

## 📊 Statystyki

- **⭐ Stars:** Zobacz aktualny stan na GitHubie
- **🍴 Forks:** Społeczność aktywnie rozwija projekt
- **🐛 Issues:** Średni czas odpowiedzi <24h

---

*Stworzony z ❤️ dla społeczności Home Assistant*