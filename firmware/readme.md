# AURA Firmware

Firmware for the AURA wearable AI pendant running on the **Seeed XIAO ESP32-S3 Sense**.

Handles audio capture, image capture, BLE streaming to the companion app, and OTA updates.

---

## Architecture

```
firmware/src/
├── app.cpp              ← main application loop & task orchestration
├── app.h
├── mic.cpp              ← PDM microphone (I2S, 16kHz)
├── mic.h
├── opus_encoder.cpp     ← Opus audio compression
├── opus_encoder.h
├── ota.cpp              ← OTA firmware update over BLE+WiFi
├── ota.h
├── config.h             ← all tunable settings
├── camera_pins.h        ← ESP32-S3 Sense camera pin map
└── camera_index.h       ← camera driver
```

Key settings are in `config.h`. You should not need to edit anything else for a standard build.

---

## Configuration (`src/config.h`)

All important values are in one file:

```cpp
// Device identity
#define BLE_DEVICE_NAME         "AURA"
#define FIRMWARE_VERSION_STRING "2.3.2"

// Camera
#define PHOTO_CAPTURE_INTERVAL_MS  30000   // 30s between captures
#define CAMERA_FRAME_SIZE          FRAMESIZE_VGA  // 640×480

// Audio
#define MIC_SAMPLE_RATE  16000   // 16kHz
#define OPUS_BITRATE     32000   // 32kbps

// Battery (2× 250mAh = 500mAh total)
#define BATTERY_MAX_VOLTAGE  4.2f
#define BATTERY_MIN_VOLTAGE  3.2f
```

Increasing `PHOTO_CAPTURE_INTERVAL_MS` extends battery life significantly.

---

## Method 1 — UF2 Flash (Easiest)

No toolchain needed. Just drag and drop.

### Step 1 — Enter bootloader mode

1. Hold the **BOOT** button on the ESP32-S3
2. While holding BOOT, press and release **RESET**
3. Release BOOT
4. The device appears as a USB drive named **"ESP32S3"**

### Step 2 — Flash

Copy the `.uf2` file from `firmware/releases/` to the ESP32S3 drive.  
The device flashes automatically and reboots.

### Step 3 — Monitor (optional)

```bash
pio device monitor --baud 115200
```

---

## Method 2 — PlatformIO

More control over build and upload.

**Install PlatformIO:**
```bash
pip install platformio
```

**Build and upload:**
```bash
cd firmware

# Standard build
platformio run -e seeed_xiao_esp32s3 --target upload

# If upload fails, try slower mode
platformio run -e seeed_xiao_esp32s3_slow --target upload
```

**Monitor serial:**
```bash
platformio device monitor --baud 115200
```

**Build environments:**

| Environment | Description | Use Case |
|:---|:---|:---|
| `seeed_xiao_esp32s3` | Standard | Development |
| `seeed_xiao_esp32s3_slow` | Slower upload | Connection issues |
| `uf2_release` | Optimized release | Production / best battery |

**Build UF2 file:**
```bash
./scripts/build_uf2.sh -e uf2_release
```

---

## Method 3 — Arduino IDE

**Prerequisites:**

1. Install [Arduino IDE 2.x](https://www.arduino.cc/en/software)
2. Add ESP32 board package URL in `File → Preferences → Additional boards manager URLs`:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
3. `Tools → Board → Boards Manager` → search `esp32` → Install

**Board settings — get these right or the camera won't work:**

```
Tools → Board  →  XIAO_ESP32S3
Tools → PSRAM  →  OPI PSRAM        ← required. Camera fails without this.
Tools → Port   →  your COM port
```

> **Windows:** If COM port doesn't appear, install the [CH340 driver](https://www.wch-ic.com/downloads/CH341SER_EXE.html) and restart.

**Upload:**
```
1. Connect ESP32-S3 via USB-C
2. Hold BOOT → press RESET → release BOOT
3. Click Upload in Arduino IDE
4. Open Serial Monitor @ 115200 baud
```

**Expected serial output:**
```
[AURA] Camera initialized
[AURA] Microphone initialized
[AURA] BLE advertising
[AURA] Ready
```

---

## Method 4 — Arduino CLI

**Install board:**
```bash
arduino-cli config add board_manager.additional_urls \
  https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json

arduino-cli core install esp32:esp32@2.0.17
```

**Check board ID:**
```bash
arduino-cli board list
```
On Windows 11 should show as `esp32:esp32:XIAO_ESP32S3`.

**Compile and upload** (replace `COM5` with your port):
```bash
arduino-cli compile --build-path build --output-dir dist \
  -e -u -p COM5 -b esp32:esp32:XIAO_ESP32S3:PSRAM=opi
```

**Opus library support:**

Find your libraries folder:
```bash
arduino-cli config get directories.user
# add /libraries to the path
```

Clone the required libraries:
```bash
git clone https://github.com/pschatzmann/arduino-libopus.git
git clone https://github.com/pschatzmann/arduino-audio-tools.git
```

---

## Battery & Power

### Hardware (AURA pendant)

- **Batteries:** 2× 250mAh Li-ion in parallel = 500mAh total, 3.7V nominal
- **Charging:** USB-C on the ESP32-S3 board
- **ADC pin:** `GPIO2` (A1) — voltage divider for battery monitoring
- **Voltage divider:** R1=169kΩ, R2=110kΩ (2.536:1 ratio)

```
Battery + ──[R1: 169kΩ]──+──[R2: 110kΩ]── Battery -
                          |
                        GPIO2 (A1)
```

### Expected runtime

| Usage Pattern | Runtime |
|:---|:---|
| Heavy — continuous capture | 6–7 hours |
| Normal — mixed active/standby | 8–10 hours |
| Light — occasional captures | 12–15 hours |

### Current draw

| Mode | Current |
|:---|:---|
| Active (Camera + BLE) | ~80mA |
| Standby (BLE only) | ~40mA |
| Deep sleep | ~2mA |

### Charging

| LED | Status |
|:---|:---|
| Red | Charging |
| Green | Fully charged |

Typical charge time: 1–1.5 hours from USB 2.0.

---

## Serial Battery Commands

Connect at 115200 baud and type:

| Command | What it shows |
|:---|:---|
| `status` | Voltage, battery %, BLE state |
| `charging` | 10 readings over 20s with charge status |
| `runtime` | Estimated hours remaining |
| `chargetime` | Time to 80% / 90% / 100% |
| `monitor` | Continuous 5s interval updates (any key stops) |

**Voltage reference:**

| Voltage | Battery % | Status |
|:---|:---|:---|
| 4.2–4.3V | 100% | Fully charged |
| 4.0–4.2V | 80–100% | Good |
| 3.8–4.0V | 20–80% | Moderate |
| 3.7–3.8V | 0–20% | Low |
| < 3.5V | Critical | Check hardware |

---

## OTA Firmware Updates

AURA supports over-the-air firmware updates via BLE + WiFi.

WiFi credentials are sent from the app over BLE. The device connects to WiFi only for the update, then returns to BLE-only mode.

**OTA flow:**
1. App sends WiFi credentials via `OTA_CONTROL_UUID`
2. Device connects to WiFi
3. App sends firmware URL
4. Device downloads and installs firmware
5. Device reboots with new firmware

---

## Troubleshooting

| Problem | Fix |
|:---|:---|
| Camera init failed | `Tools → PSRAM → OPI PSRAM` → re-upload |
| COM port missing (Windows) | Install CH340 driver, restart |
| Device not appearing as USB drive | Re-enter bootloader mode, try different USB cable |
| Build fails (PlatformIO) | `pip install platformio`, then `pio run --target clean` |
| Always shows 0% or 100% battery | Check voltage divider wiring on GPIO2 |
| BLE not pairing | Power cycle device, forget in phone Bluetooth settings |

---

## Best Practices

- **Charge when below 20%** (3.8V)
- **Don't let voltage drop below 3.5V**
- **Charge at 10°C–40°C**
- **Store at ~50% charge** if unused for weeks
- **Increase `PHOTO_CAPTURE_INTERVAL_MS`** for all-day use
