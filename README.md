# ESP32-S3 Voice Satellite dla Home Assistant

![Status Projektu](https://img.shields.io/badge/status-produkcyjny-green)
![ESPHome](https://img.shields.io/badge/ESPHome-2024.12.0+-blue)
![C++](https://img.shields.io/badge/Logika_Natywna-C++-orange)
![Licencja](https://img.shields.io/badge/licencja-MIT-grey)

Zaawansowany satelita głosowy dla Home Assistant oparty na ESP32-S3. Projekt wykorzystuje protokół I2S dla dźwięku wysokiej jakości (Dual Mono - wysoka głośność) oraz pierścień LED WS2812B dla wizualnej informacji zwrotnej.

## ✨ Funkcje
- **Lokalne Wake Word:** Obsługa "Okay Nabu" (Micro Wake Word) bezpośrednio na urządzeniu.
- **Dual Mono Audio:** Obsługa dwóch wzmacniaczy MAX98357A na jednej linii danych (mostek).
- **Stabilność:** Zoptymalizowana konfiguracja I2S (32-bit) i filtrowanie zasilania.
- **Visual Feedback:** Animacje LED dla stanów: Słuchanie, Myślenie, Mówienie, Błąd.
- **Push-to-Talk:** Przycisk w Home Assistant do ręcznego wywoływania asystenta.

---

## 🛠️ Sprzęt (BOM)

| Element | Opis |
| :--- | :--- |
| **MCU** | ESP32-S3 DevKitC-1 (N8R16 - 8MB Flash / 16MB PSRAM) |
| **Mikrofon** | INMP441 (I2S MEMS) |
| **Wzmacniacz** | 2x MAX98357A (Klasa D - Zmostkowane kanały DIN) |
| **LED** | WS2812B Ring (12 diod) |
| **Zasilanie** | Zasilacz 5V 3A (min.) + Bezpiecznik |
| **Filtrowanie** | Kondensatory 470µF przy zasilaniu wzmacniaczy i LED |

---

## 🔌 Schemat Podłączenia (Pinout)

Wszystkie urządzenia audio pracują na wspólnej magistrali (Shared Bus).

| Pin ESP32 | Urządzenie | Pin Urządzenia | Funkcja |
| :--- | :--- | :--- | :--- |
| **GPIO 15** | Mic + 2x Amp | SCK / BCLK | **Bit Clock (Mostek)** |
| **GPIO 16** | Mic + 2x Amp | WS / LRC | **Word Select (Mostek)** |
| **GPIO 6** | Mic | SD | Dane Mikrofonu |
| **GPIO 11** | 2x Amp | DIN | Dane Wzmacniaczy (Mostek) |
| **GPIO 21** | LED | DI | Dane LED (Bezpośrednio) |

> **UWAGA:** Pamiętaj o solidnym połączeniu wspólnej masy (GND) dla wszystkich komponentów!

## Diagram Połączeń

> Pełny schemat dostępny w [docs/schematic.md](docs/SCHEMATIC.md%20)

---

## 🚀 Instalacja (Home Assistant)

Ten projekt jest przeznaczony do użycia bezpośrednio w dodatku **ESPHome** w Home Assistant.

### Krok 1: Weryfikacja Sekretów
Twój plik `secrets.yaml` w ESPHome może już zawierać konfigurację. Otwórz go (przycisk **"Secrets"** w prawym górnym rogu dashboardu) i upewnij się, że posiadasz zdefiniowane poniższe zmienne.

**Wymagane zmienne dla tego projektu:**
(Jeśli ich brakuje – dopisz je na końcu pliku. Jeśli już są – nie musisz ich dublować).

wifi_ssid: "Twoja_Nazwa_WiFi"
wifi_password: "Twoje_Haslo_WiFi"
api_key: "Wygeneruj_Dlugie_Haslo_Tutaj"
ota_password: "Wygeneruj_Inne_Haslo_Tutaj"
ap_password: "SuperTajneHasloDoHotspota"

### Krok 2: Utworzenie Urządzenia
1. W ESPHome Dashboard kliknij **+ NEW DEVICE**.
2. Nadaj nazwę: `esp32-s3-voice-satellite`.
3. Pomiń instalację (kliknij **SKIP**).
4. Wejdź w edycję nowo powstałego urządzenia (**EDIT**).

### Krok 3: Wgranie Kodu
1. Skopiuj całą zawartość pliku `voice_assistant.yaml` z tego repozytorium.
2. Wklej do edytora w ESPHome (nadpisz domyślną konfigurację).
3. **WAŻNE:** Znajdź sekcję `manual_ip` i wpisz adres IP, który jest **wolny** w Twojej sieci (sprawdź w routerze, czy adres nie jest zajęty!).
4. Kliknij **SAVE** i **INSTALL**.

### Krok 4: Konfiguracja w Home Assistant
1. **Ustawienia** → **Urządzenia i usługi** → urządzenie pojawi się automatycznie
2. **Ustawienia** → **Asystenci głosowi** → wybierz pipeline
3. Wystawianie encji do asystenta (światła, przełączniki itp.)

---

## ⚠️ Ważne uwagi

* **Adres IP:** W kodzie zdefiniowany jest statyczny adres IP dla stabilności połączenia. Musisz go dostosować do swojej podsieci (np. `192.168.1.xxx` lub `10.0.x.x`).
* **Zasilanie:** Projekt wymaga wydajnego zasilania (min. 2A-3A). Zasilanie z portu USB komputera może powodować niestabilność przy włączonych LEDach i głośnikach.
* **Kondensatory:** Zaleca się przylutowanie kondensatorów elektrolitycznych (np. 470µF) bezpośrednio przy pinach zasilania wzmacniaczy i pierścienia LED, aby wyeliminować trzaski.

---

## 🎨 Efekty LED

| Efekt | Wyzwalacz | Implementacja |
|-------|-----------|---------------|
| **Listening** | Wykrycie wake word | Cyan, oddychanie przez `sin()` |
| **Thinking** | Koniec mowy | Niebieski spinner |
| **Speaking** | Start TTS | Zielona symulacja |
| **Error** | Błąd | Czerwone mruganie |

---

## 📂 Struktura Projektu

```
ha-voice-satellite/
├── README.md                    # Dokumentacja (ten plik)
├── README_EN.md                 # English version
├── LICENSE                      # Licencja MIT
├── .gitignore                   
├── secrets.yaml.example         # Szablon
│
├── voice_assistant.yaml         # ← Główny plik instalacyjny
│
├── docs/
│   ├── schematic.svg            # Schemat graficzny
│   ├── SCHEMATIC.md             # Schemat tekstowy + tabele
│   └── TROUBLESHOOTING.md       # Rozwiązywanie problemów
│
└── 3d_models/
    ├── pokrywa.stl              # Modele do druku 3D
    ├── maskownica.stl
    ├── glosnik.stl
    └── README.md                # Info o modelach
```
---

## 🐛 Problemy?

Najczęstsze problemy i rozwiązania w [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)

---

## 🤝 Podziękowania

### Hardware i Modele 3D
- **Obudowa STL:** [Kristopher Mackowiak](https://github.com/KristopherMackowiak/ha_voice_assistant)
- **Licencja:** MIT

### Firmware
- **Architektura:** Clean Code, Separation of Concerns, HAL
- **Optymalizacje:** Natywny C++, konfiguracja I2S
- **Autor:** [Krzysztof / @KRISUUUU](https://github.com/KRISUUUU)

---

## 📜 Licencja

MIT License - szczegóły w [LICENSE](LICENSE)

---

## 🌟 Wspieraj Projekt

- ⭐ Gwiazdka na GitHubie
- 🐛 Zgłaszanie błędów
- 💡 Propozycje funkcji
- 🔀 Pull Requests

---

*Stworzony dla społeczności Home Assistant*