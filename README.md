<div align="center">

<img src="https://img.shields.io/badge/Platform-Android%206.0%2B-3DDC84?style=for-the-badge&logo=android&logoColor=white"/>
<img src="https://img.shields.io/badge/Connection-WiFi%20%7C%20Bluetooth-00BCD4?style=for-the-badge&logo=bluetooth&logoColor=white"/>
<img src="https://img.shields.io/badge/Sensors-10%20Supported-FF6F00?style=for-the-badge&logo=arduino&logoColor=white"/>
<img src="https://img.shields.io/badge/License-MIT-9C27B0?style=for-the-badge"/>

# 🌡 ThermoLink

**Wireless IR temperature scanning for Android — powered by ESP8266 / ESP32**

Professional-grade contactless forehead scanning with live thermal imaging, guided face detection, a real-time web dashboard, and scan history export. Built for kiosk deployments, clinics, and any high-throughput screening environment.

[**Download APK**](#) · [**ESP Sketch**](#getting-started) · [**Web Dashboard**](#web-dashboard)

---

</div>

## What It Does

ThermoLink turns an Android phone into a contactless temperature screening station. It connects to an ESP8266 or ESP32 microcontroller over **Wi-Fi (TCP)** or **Bluetooth**, reads temperature data from your IR sensor, and guides the subject through the scan with on-screen and voice instructions.

Every scan is logged with a timestamp, temperature, label (Normal / Elevated / Fever), optional photo, and device name. Results are immediately visible on the phone and on a **live web dashboard** accessible from any browser on the same network.

---

## Guided Scan Flow

The scan screen walks through a defined sequence — no accidental triggers, no background scanning.

```
Tap "Start Scan"
      ↓
Camera activates · Face detection runs
      ↓
Face found → "Get closer to sensor · aim above eyebrows"
      ↓
Tap "Done" when positioned
      ↓
Temperature captured · Result displayed
      ↓
Auto-resets for next person (3 s)
```

While idle, the screen shows scanning tips. The moment a face is detected, the tips hide and guidance takes over. The camera only runs during an active scan — not continuously in the background.

---

## Supported Sensors

| Sensor | Type | Resolution | Notes |
|---|---|---|---|
| **MLX90614** | Single-point IR | — | Default, most common |
| **MLX90615** | Single-point IR | — | Compact form factor |
| **MLX90632** | Single-point IR | — | Medical-grade accuracy |
| **TMP007** | Single-point IR | — | TI sensor |
| **AMG8833** | Thermal array | 8 × 8 | Live heatmap |
| **Panasonic Grid-EYE** | Thermal array | 8 × 8 | Live heatmap |
| **MLX90640** | Thermal camera | 32 × 24 | High-res heatmap |
| **MLX90621** | Thermal array | 16 × 4 | Wide-angle |
| **Omron D6T-44L** | Thermal array | 4 × 4 | UART |
| **Omron D6T-8L** | Thermal array | 1 × 8 | UART |

Array sensors render a **live iron-palette heatmap** on the scan screen and web dashboard in real time.

---

## Getting Started

### 1 · Flash the ESP

1. Open `esp_sketch.ino` (bundled inside the app under **Settings → ESP Sketch**) in Arduino IDE
2. Set your Wi-Fi credentials at the top:
   ```cpp
   const char* WIFI_SSID = "your_network";
   const char* WIFI_PASS = "your_password";
   ```
3. Uncomment the block for **your sensor** (one block only — all others stay commented)
4. Select your board:
   - ESP8266 → **NodeMCU 1.0** or **Wemos D1 Mini**
   - ESP32 → **ESP32 Dev Module** (swap `<ESP8266WiFi.h>` → `<WiFi.h>`)
5. Flash and open Serial Monitor at **115200 baud** — note the IP address printed on boot

**Required Arduino libraries** (install via Library Manager):

| Library | Used for |
|---|---|
| Adafruit MLX90614 | MLX90614, MLX90615, MLX90632 |
| Adafruit AMG88xx | AMG8833, Grid-EYE |
| Adafruit MLX90640 | MLX90640 |
| Adafruit TMP007 | TMP007 |

---

### 2 · Connect the App

**Wi-Fi (recommended)**
1. On the Scan screen, tap **Wi-Fi ESP**
2. Enter the ESP's IP address and port (`8888` default)
3. Select your sensor model from the list
4. Tap **Connect**

**Bluetooth**
1. Pair your HC-05/HC-06 module to the phone in Android Settings
2. On the Scan screen, tap **Bluetooth**
3. Select the device from the list

The app auto-reconnects to the last used device on launch.

---

### 3 · Scan

1. Tap **▶ Start Scan**
2. Step in front of the camera — the app finds your face automatically
3. Hold the sensor **3–5 cm from the forehead**, above the eyebrows
4. Tap **Done** — the temperature is captured and saved

> **Tips:** Scan indoors. Avoid hair, hats, sweat, and direct sunlight. Use a consistent distance every time for best accuracy.

---

## Web Dashboard

While connected, ThermoLink hosts a live dashboard accessible from any browser on the same Wi-Fi network.

**How to open it:**
1. On the Scan screen, tap the **⋮ menu → Web Dashboard**
2. Copy the URL shown (e.g. `http://192.168.1.42:8080`)
3. Open it on a tablet, monitor, or second device

**What it shows:**
- Live temperature and label, updated every 1.5 seconds
- Live thermal heatmap (array sensors)
- Room temperature, heat index, and humidity
- Session scan counter
- Temperature trend chart (last 50 scans)
- Label distribution chart
- Full scan history table with photos, filterable and sortable
- Browser voice readout (toggleable)
- One-click CSV export

The dashboard keeps running even with the phone screen off, as long as the app is in the foreground service.

---

## Scan History & Export

Every scan is saved locally with:
- Timestamp
- Temperature (°C or °F)
- Label (Normal / Elevated / Fever / custom)
- Photo (if camera is enabled)
- Device name

**Export options:**
- **XLSX** — native Excel format, no external library, one tap from the History screen
- **CSV** — from the web dashboard Export button

---

## Settings

| Setting | What it does |
|---|---|
| Temperature unit | °C or °F |
| Calibration offset | Fine-tune sensor readings (±°) |
| Custom thresholds | Set your own Normal / Elevated / Fever cutoffs |
| Camera | Enable photo capture per scan |
| Face detection | ML Kit on-device face guidance |
| Auto-scan when ready | Trigger scan automatically when face is positioned |
| Voice guidance | Spoken prompts during scan |
| Voice results | Read out temperature + label after each scan |
| Alert: beep / vibrate / flash | Configurable alert on fever detection |
| Background scanning | Keep scanning with screen off (foreground service) |
| Web server port | Default 8080 |
| Scan delay | Minimum milliseconds between scans |
| Weather | Room temperature and heat index overlay |
| Clock | Live clock on scan screen |

---

## Sensor Wiring (I²C — all single-point and most array sensors)

```
ESP8266          Sensor
─────────        ──────
3.3V      →      VCC
GND       →      GND
D2 (SDA)  →      SDA
D1 (SCL)  →      SCL
```

For ESP32: use GPIO 21 (SDA) and GPIO 22 (SCL).  
D6T sensors use UART — see comments in the sketch for pin mapping.

---

## Requirements

- Android 6.0 (API 23) or higher
- Camera permission (for photo capture and face detection)
- Location permission (for weather widget — optional)
- ESP8266 or ESP32 with supported IR sensor
- Both devices on the same Wi-Fi network (for Wi-Fi mode)

---

## Privacy

All processing is on-device. No data leaves your network. Face detection runs entirely offline via ML Kit. No analytics, no accounts, no cloud. Scan photos are stored only in the app's private external directory.

---

<div align="center">

Built with ❤️ by [engrpanda](https://github.com/engrpanda) · Originally from [Instructables: Wireless IR Temperature Scanner](https://www.instructables.com)

</div>
