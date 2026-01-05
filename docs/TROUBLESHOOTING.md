# 🛠️ Troubleshooting Guide
ESP32-S3 Voice Satellite

Przewodnik rozwiązywania problemów dla projektu **ESP32-S3 Voice Satellite**. Obejmuje typowe problemy, przyczyny i rozwiązania.

---

## 📡 Problemy z WiFi

### Device won't connect to WiFi / Urządzenie nie łączy się z WiFi

**Objawy:**
- Urządzenie offline w ESPHome Dashboard (czerwony status)
- LED nie pokazuje animacji startowej (Boot Filling)
- Logi pokazują `WiFi connection failed`

**Rozwiązania:**

#### 1. Sprawdź dane logowania
```yaml
# W secrets.yaml:
wifi_ssid: "Twoja_Nazwa_WiFi"      # ← Wielkość liter ma znaczenie!
wifi_password: "Twoje_Haslo_WiFi"  # ← Upewnij się że jest poprawne
```

#### 2. Sprawdź pasmo WiFi
- ESP32-S3 obsługuje **TYLKO 2.4GHz**
- Router nie może być w trybie "5GHz only"
- Sprawdź w ustawieniach routera czy 2.4GHz jest włączone

#### 3. Konflikt adresu IP
**Domyślna konfiguracja:**
```yaml
manual_ip:
  static_ip: 10.0.50.177  # ← TEN ADRES MUSI BYĆ WOLNY!
  gateway: 10.0.50.1
```

**Jak sprawdzić czy adres jest wolny:**
1. Otwórz terminal/CMD
2. Wykonaj ping: `ping 10.0.50.177`
3. Jeśli dostaniesz odpowiedź → adres zajęty, wybierz inny

**Jeśli Twoja sieć to np. 192.168.1.x:**
```yaml
manual_ip:
  static_ip: 192.168.1.177  # ← Zmień na swoją podsieć
  gateway: 192.168.1.1      # ← Twój router
  subnet: 255.255.255.0
  dns1: 8.8.8.8
```

#### 4. Zbyt słaby sygnał
- Urządzenie może być za daleko od routera
- Przenieś bliżej routera lub użyj repeatera WiFi
- W logach szukaj: `RSSI` (powinno być >-70dBm)

---

## 🔊 Problemy z Audio

### No sound from speaker / Brak dźwięku z głośnika

**Objawy:**
- LED działa poprawnie
- Voice Assistant uruchamia się (widać TTS w logach)
- Głośnik milczy kompletnie

**Rozwiązania:**

#### 1. Sprawdź zasilanie wzmacniaczy
```
MAX98357A Pin    | Wymagane Podłączenie
─────────────────┼─────────────────────
Vin              | 5V (WYMAGANE!)
GND              | GND
SD               | 5V (Enable + Mono)
GAIN             | 5V (Max volume 15dB)
```

**KRYTYCZNE:** Vin = 5V (nie 3.3V!) - przy 3.3V głośność będzie niemal niesłyszalna lub zero.

#### 2. Sprawdź pinout I2S wzmacniaczy
```yaml
# W voice_assistant.yaml:
amp_bclk: GPIO4   # ← Musi być GPIO4
amp_lrclk: GPIO5  # ← Musi być GPIO5
amp_data: GPIO11  # ← Musi być GPIO11
```

Sprawdź połączenia:
- GPIO4 → BCLK (oba wzmacniacze)
- GPIO5 → LRC (oba wzmacniacze)
- GPIO11 → DIN (oba wzmacniacze równolegle)

#### 3. Sprawdź konfigurację YAML
```yaml
speaker:
  - platform: i2s_audio
    id: physical_speaker_output
    i2s_audio_id: i2s_amp_bus  # ← Musi być amp_bus, NIE mic_bus!
    dac_type: external
    i2s_dout_pin: GPIO11       # ← Sprawdź GPIO
    channel: mono              # ← WYMAGANE dla MAX98357A
    bits_per_sample: 16bit     # ← NIE zmieniaj na 32bit
    sample_rate: 48000         # ← Standardowy sample rate
```

#### 4. Test diagnostyczny
Dodaj do YAML (tymczasowo):
```yaml
button:
  - platform: template
    name: "Test Dźwięku"
    on_press:
      - logger.log: "Rozpoczynam test audio..."
      - voice_assistant.start:
```

Naciśnij przycisk w HA i sprawdź logi. Jeśli widzisz błędy I2S → problem sprzętowy.

---

### Audio crackling / popping / Trzaski w dźwięku

**Objawy:**
- Słyszalne "popy" gdy LED się włączają
- Zniekształcony lub niestabilny dźwięk
- Trzaski przy każdej zmianie głośności

**Rozwiązania:**

#### 1. Dodaj kondensatory (KRYTYCZNE!)
**Wymagane:**
- 470µF przy **KAŻDYM** wzmacniaczu (Vin → GND)
- 1000µF przy LED Ring (5V → GND)

**Jak podłączyć:**
```
       Vin ───┬─── [Do ESP32 5V]
              │
         [470µF]  ← Kondensator elektrolityczny
              │     (+) do Vin, (-) do GND
       GND ───┴─── [Do ESP32 GND]
```

#### 2. Sprawdź wspólną masę
**WYMAGANE:** Wszystkie GND muszą być połączone:
- ESP32 GND
- Mikrofon GND
- Wzmacniacz #1 GND
- Wzmacniacz #2 GND
- LED GND

**Najlepsze rozwiązanie:** Użyj złączki WAGO lub listwa zaciskowa.

#### 3. Zbyt słabe zasilanie
- Minimalne wymaganie: **5V 2.5A**
- Zalecane: **5V 3A** (stabilizowane)
- **NIE zasilaj z USB komputera** przy włączonych LED i głośnikach

---

### Microphone not picking up voice / Mikrofon nie odbiera głosu

**Objawy:**
- Wake word nigdy się nie aktywuje
- Urządzenie wydaje się "głuche"
- Logi pokazują `No voice detected`

**Rozwiązania:**

#### 1. ⚠️ SPRAWDŹ NAPIĘCIE (KRYTYCZNE!)
```
INMP441 Pin | WYMAGANE Podłączenie
────────────┼─────────────────────
VDD         | 3.3V (TYLKO!)
GND         | GND
```

**UWAGA:** 5V na VDD **nieodwracalnie zniszczy mikrofon!**

#### 2. Sprawdź kanał (L/R)
```
INMP441 Pin | Podłączenie
────────────┼────────────
L/R         | GND (Kanał lewy)
```

W YAML:
```yaml
microphone:
  channel: left  # ← Musi być "left" jeśli L/R = GND
```

#### 3. Sprawdź pinout mikrofonu
```yaml
mic_bclk: GPIO15   # ← Mikrofon SCK
mic_lrclk: GPIO16  # ← Mikrofon WS
mic_data: GPIO6    # ← Mikrofon SD
```

Fizyczne połączenia:
- GPIO15 → SCK (Bit Clock)
- GPIO16 → WS (Word Select)
- GPIO6 → SD (Data)

#### 4. Test czułości mikrofonu
Dodaj do logów:
```yaml
logger:
  level: DEBUG  # ← Zmień na DEBUG
```

Szukaj w logach:
- `[micro_wake_word] Probability: X.XX` ← Powinno być >0.5 gdy mówisz
- Jeśli zawsze 0.00 → mikrofon nie działa

---

## 💡 Problemy z LED

### LEDs not lighting up / LEDy się nie świecą

**Objawy:**
- Pierścień LED całkowicie ciemny
- Brak reakcji na stan urządzenia
- Logi nie pokazują błędów LED

**Rozwiązania:**

#### 1. Sprawdź zasilanie
```
WS2812B Pin | Wymagane
────────────┼─────────
5V          | 5V (NIE 3.3V!)
GND         | GND
```

**Test:** Zmierz multimetrem napięcie na pinach LED:
- Powinno być **5V ± 0.2V**
- Jeśli 3.3V → LED nie zaświecą (za mało)

#### 2. Sprawdź kierunek LED
```
DI (Data In) ───► [LED 0] ───► [LED 1] ───► ... ───► DO (Data Out)
                    ▲
                    │
              Strzałka na PCB
```

**Sprawdź:** Czy GPIO21 jest podłączone do **DI** (nie DO)?

#### 3. Sprawdź konfigurację YAML
```yaml
light:
  - platform: esp32_rmt_led_strip
    id: led_ring
    pin: GPIO21           # ← Sprawdź GPIO
    num_leds: 12          # ← Sprawdź ilość LED
    rgb_order: GRB        # ← WS2812B = GRB (nie RGB!)
    chipset: WS2812       # ← Typ chipsetu
```

#### 4. Test ręczny
Dodaj do YAML:
```yaml
button:
  - platform: template
    name: "Test LED"
    on_press:
      - light.turn_on:
          id: led_ring
          brightness: 100%
          red: 100%
          green: 0%
          blue: 0%
```

Kliknij przycisk w HA. Jeśli LEDy nie świecą → problem sprzętowy.

---

### LEDs flickering randomly / LED migoczą losowo

**Objawy:**
- Losowe mruganie LED
- Niestabilne kolory
- ESP32 się resetuje gdy LED włączają się na full

**Rozwiązania:**

#### 1. Dodaj kondensator (KRYTYCZNE!)
```
5V ───┬─── [Do LED Ring]
      │
 [1000µF]  ← Kondensator elektrolityczny
      │     (+) do 5V, (-) do GND
GND ───┴─── [Do GND]
```

#### 2. Skróć przewód danych
- Maksymalna długość: **50cm**
- Im krótszy przewód → tym stabilniejszy sygnał
- Użyj ekranowanego przewodu jeśli >30cm

#### 3. Zwiększ zasilanie
- 12 LED @ full white = ~700mA
- Jeśli zasilacz <2A → może być za słaby
- Użyj minimum 3A zasiłacza

---

## 🏠 Problemy z Home Assistant

### Device not appearing in Home Assistant / Urządzenie nie pojawia się w HA

**Objawy:**
- Urządzenie online w ESPHome
- Nie widać go w HA → Urządzenia i usługi

**Rozwiązania:**

#### 1. Ręczna integracja
1. **Ustawienia** → **Urządzenia i usługi**
2. Kliknij **+ Dodaj integrację**
3. Wyszukaj **ESPHome**
4. Podaj adres IP urządzenia (np. `10.0.50.177`)
5. Podaj hasło (jeśli wymagane)

#### 2. Sprawdź klucz API
W `secrets.yaml`:
```yaml
api_key: "Twój_32_znakowy_klucz"
```

W Home Assistant → Integration ESPHome → powinien być ten sam klucz.

#### 3. Sprawdź firewall
- ESP32 używa portu **6053** (API)
- Sprawdź czy firewall nie blokuje tego portu
- Sprawdź czy urządzenia są w tej samej podsieci

---

### Wake word not working / Wake word nie działa

**Objawy:**
- Urządzenie działające
- Wake word "Okay Nabu" nie aktywuje asystenta
- Logi pokazują `micro_wake_word: running`

**Rozwiązania:**

#### 1. Sprawdź próg detekcji
```yaml
micro_wake_word:
  models:
    - model: "okay_nabu.json"
      probability_cutoff: 0.7  # ← Zmniejsz do 0.5 jeśli zbyt czuły
```

**Test:** Powiedz głośno "Okay Nabu" kilka razy. Sprawdź logi:
```
[micro_wake_word] Probability: 0.85  ← Wykryto!
[micro_wake_word] Probability: 0.32  ← Zbyt cicho
```

#### 2. Sprawdź czy wake word uruchomione
W logach szukaj:
```
[micro_wake_word] Starting wake word detection
```

Jeśli nie ma → dodaj do boot sequence:
```yaml
on_boot:
  - priority: 600
    then:
      - micro_wake_word.start:
```

#### 3. Problem z mute switch
Sprawdź czy mikrofon nie jest wyciszony:
- W HA → sprawdź encję `switch.wycisz_mikrofon`
- Powinna być **OFF**
- Jeśli **ON** → LED świeci czerwono = mikrofon wyciszony

---

## 🔧 Problemy z Wydajnością

### "Parent I2S bus not free" error

**Objawy:**
- Error w logach przy starcie audio
- Voice assistant nie uruchamia się
- Crash ESP32 przy próbie użycia audio

**Przyczyna:**
- Konflikt konfiguracji magistral I2S
- Mikrofon i wzmacniacz używają tej samej magistrali

**Rozwiązanie:**
Sprawdź czy masz **DWA OSOBNE** definicje I2S:

```yaml
i2s_audio:
  - id: i2s_mic_bus      # ← Magistrala #1 (Mikrofon)
    i2s_lrclk_pin: GPIO16
    i2s_bclk_pin: GPIO15
    
  - id: i2s_amp_bus      # ← Magistrala #2 (Wzmacniacze)
    i2s_lrclk_pin: GPIO5
    i2s_bclk_pin: GPIO4
```

**NIE MOGĄ** mieć tych samych pinów!

---

### "Failed to allocate memory" / Brak pamięci

**Objawy:**
- ESP32 restartuje się losowo
- Error: `Failed to allocate PSRAM`
- Crash podczas wake word detection

**Rozwiązania:**

#### 1. Sprawdź czy PSRAM włączony
```yaml
psram:
  mode: octal  # ← WYMAGANE dla ESP32-S3
  speed: 80MHz
```

#### 2. Zwiększ stack size
```yaml
esp32:
  framework:
    type: esp-idf
    sdkconfig_options:
      CONFIG_ESP_SYSTEM_EVENT_TASK_STACK_SIZE: "4096"  # ← Zwiększ do 8192 jeśli problem
```

#### 3. Zmniejsz ilość wake words
Jeśli masz wszystkie 3 modele (Alexa + Nabu + Jarvis) → zostaw tylko 1:
```yaml
micro_wake_word:
  models:
    - model: "okay_nabu.json"  # ← Tylko jeden model
      probability_cutoff: 0.7
```

---

## 📊 Diagnostyka - Jak zebrać informacje

### Włącz logi DEBUG
```yaml
logger:
  level: DEBUG
  baud_rate: 115200
```

### Podłącz przez USB i czytaj logi
1. Otwórz https://web.esphome.io
2. Podłącz ESP32 przez USB
3. Kliknij **Connect** → wybierz port
4. Oglądaj logi w czasie rzeczywistym

### Logi do załączenia przy zgłoszeniu problemu
```
[I][app:102] ESPHome version ...
[I][wifi:289] WiFi connected ...
[I][micro_wake_word:XXX] ...
[E][voice_assistant:XXX] Error: ...  ← To jest ważne!
```

---

## 🆘 Dalej potrzebujesz pomocy?

### 1. Sprawdź GitHub Issues
https://github.com/KRISUUUU/esp32-s3-voice-satellite/issues

### 2. Otwórz nowy Issue
Załącz:
- [ ] Pełne logi (DEBUG level)
- [ ] Konfigurację YAML (bez secrets!)
- [ ] Zdjęcie połączeń sprzętowych
- [ ] Opis problemu krok po kroku

### 3. Społeczność Home Assistant
- Forum: https://community.home-assistant.io
- Discord: https://discord.gg/home-assistant

---

*Ostatnia aktualizacja: 2025-01-05*