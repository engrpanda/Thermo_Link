<div align="center">

<img src="https://img.shields.io/badge/Platform-Android%206.0%2B-3DDC84?style=for-the-badge&logo=android&logoColor=white"/>
<img src="https://img.shields.io/badge/Connection-WiFi%20%7C%20Bluetooth-00BCD4?style=for-the-badge&logo=bluetooth&logoColor=white"/>
<img src="https://img.shields.io/badge/Sensors-10%20Supported-FF6F00?style=for-the-badge&logo=arduino&logoColor=white"/>
<img src="https://img.shields.io/badge/License-MIT-9C27B0?style=for-the-badge"/>

# 🌡 ThermoLink

**Wireless IR temperature scanning for Android — powered by ESP8266 / ESP32**

Professional-grade contactless forehead scanning with live thermal imaging, guided face detection, a real-time web dashboard, and scan history export. Built for kiosk deployments, clinics, and any high-throughput screening environment.

[**⬇ Download APK**](https://github.com/engrpanda/Thermo_Link/releases) · [**ESP Sketch**](#esp-sketches) · [**Web Dashboard**](#web-dashboard)

---

</div>

## Origin

ThermoLink is the full Android successor to the original **Wireless IR Temperature Scanner** — a project that started on Instructables and Hackster.io. The original used a simple serial-over-Bluetooth link to a desktop app. ThermoLink rebuilds that idea from the ground up: native Android, Wi-Fi TCP, 10 sensors, thermal imaging, face detection, and a live web dashboard — while keeping the same ESP8266 hardware most builders already have.

| | Original | ThermoLink |
|---|---|---|
| Platform | Desktop (serial monitor) | Android app |
| Connection | Bluetooth serial | Wi-Fi TCP · Bluetooth |
| Sensors | MLX90614 | 10 sensors incl. thermal arrays |
| Output | Serial text | Live screen · web dashboard · XLSX export |
| Face guidance | — | ML Kit on-device detection |

**Original project:**

- 📄 [Arduino sketch](https://github.com/engrpanda/Wireless-IR-Temperature-Scanner/blob/master/Wireless_IR_Temperature_Scanner.ino)
- 🔧 [Instructables tutorial](https://www.instructables.com/id/Wireless-IR-Temperature-Scanner/)
- 🔧 [Hackster.io tutorial](https://www.hackster.io/engrpandaece/wireless-ir-temperature-scanner-acbfd9)

---

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

## Manual Capture

If auto-capture stalls or you prefer full control, three buttons are always available at the bottom of the scan screen:

| Button | What it does |
|---|---|
| 📸 **Take Photo** | Snaps a photo and caches it. Shows **📸 Photo ✅** when ready. Nothing is saved yet. |
| 🌡 **Capture Temp** | Caches the current temperature reading. Shows **🌡 Temp ✅** when ready. Nothing is saved yet. |
| 💾 **Save Scan** | Commits everything — cached photo + temperature — to the database and web dashboard in one shot. |

You can tap them in any order. **Save Scan** dims until at least one of the two is staged, then brightens to signal it's ready to commit. Both labels reset after a successful save.

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
| **Omron D6T-44L** | Thermal array | 4 × 4 | Raw I²C |
| **Omron D6T-8L** | Thermal array | 1 × 8 | Raw I²C |

Array sensors render a **live iron-palette heatmap** on the scan screen and web dashboard in real time.

---

## Getting Started

### 1 · Connect the App

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

### 2 · Scan

1. Tap **▶ Start Scan**
2. Step in front of the camera — the app finds your face automatically
3. Hold the sensor **3–5 cm from the forehead**, above the eyebrows
4. Tap **Done** — temperature is captured and saved automatically

If auto-capture doesn't trigger, use the manual buttons: tap **📸 Take Photo**, then **🌡 Capture Temp**, then **Save Scan**.

> **Tips:** Scan indoors. Avoid hair, hats, sweat, and direct sunlight. Use a consistent distance every time for best accuracy.

---

## ESP Sketches

Each sensor has its own ready-to-flash sketch below. Open in Arduino IDE, set your Wi-Fi credentials, select your board, and flash.

**Board selection:**
- ESP8266 → `Tools → Board → NodeMCU 1.0` or `Wemos D1 Mini`
- ESP32 → `Tools → Board → ESP32 Dev Module` — and replace `#include <ESP8266WiFi.h>` with `#include <WiFi.h>`

**I²C wiring (all I²C sensors):**
```
ESP8266: SDA = D2 (GPIO4)  ·  SCL = D1 (GPIO5)
ESP32:   SDA = GPIO21      ·  SCL = GPIO22
```

---

### MLX90614

**Library:** `Adafruit MLX90614 Library` (Arduino Library Manager)

```cpp
#include <ESP8266WiFi.h>   // use <WiFi.h> for ESP32
#include <Wire.h>
#include <Adafruit_MLX90614.h>

const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASS = "YOUR_WIFI_PASSWORD";
const int   TCP_PORT  = 8888;

Adafruit_MLX90614 mlx;
WiFiServer server(TCP_PORT);

void setup() {
  Serial.begin(115200);
  Wire.begin();
  if (!mlx.begin()) { Serial.println("MLX90614 not found!"); while(1); }
  Serial.println("MLX90614 ready");
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }
  Serial.println("\nIP: " + WiFi.localIP().toString());
  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (!client) return;
  while (client.connected()) {
    float t = mlx.readObjectTempC();
    client.println("T:" + String(t, 2));
    delay(500);
  }
  client.stop();
}
```

---

### MLX90615

**Library:** `Adafruit MLX90614 Library` (same library as MLX90614)

```cpp
#include <ESP8266WiFi.h>   // use <WiFi.h> for ESP32
#include <Wire.h>
#include <Adafruit_MLX90614.h>

const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASS = "YOUR_WIFI_PASSWORD";
const int   TCP_PORT  = 8888;

Adafruit_MLX90614 mlx;
WiFiServer server(TCP_PORT);

void setup() {
  Serial.begin(115200);
  Wire.begin();
  if (!mlx.begin()) { Serial.println("MLX90615 not found!"); while(1); }
  Serial.println("MLX90615 ready");
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }
  Serial.println("\nIP: " + WiFi.localIP().toString());
  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (!client) return;
  while (client.connected()) {
    float t = mlx.readObjectTempC();
    client.println("T:" + String(t, 2));
    delay(500);
  }
  client.stop();
}
```

---

### MLX90632

**Library:** `Adafruit MLX90614 Library` (compatible)

```cpp
#include <ESP8266WiFi.h>   // use <WiFi.h> for ESP32
#include <Wire.h>
#include <Adafruit_MLX90614.h>

const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASS = "YOUR_WIFI_PASSWORD";
const int   TCP_PORT  = 8888;

Adafruit_MLX90614 mlx;
WiFiServer server(TCP_PORT);

void setup() {
  Serial.begin(115200);
  Wire.begin();
  if (!mlx.begin()) { Serial.println("MLX90632 not found!"); while(1); }
  Serial.println("MLX90632 ready");
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }
  Serial.println("\nIP: " + WiFi.localIP().toString());
  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (!client) return;
  while (client.connected()) {
    float t = mlx.readObjectTempC();
    client.println("T:" + String(t, 2));
    delay(500);
  }
  client.stop();
}
```

---

### TMP007

**Library:** `Adafruit TMP007 Library` (Arduino Library Manager)

```cpp
#include <ESP8266WiFi.h>   // use <WiFi.h> for ESP32
#include <Wire.h>
#include <Adafruit_TMP007.h>

const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASS = "YOUR_WIFI_PASSWORD";
const int   TCP_PORT  = 8888;

Adafruit_TMP007 tmp;
WiFiServer server(TCP_PORT);

void setup() {
  Serial.begin(115200);
  Wire.begin();
  if (!tmp.begin()) { Serial.println("TMP007 not found!"); while(1); }
  Serial.println("TMP007 ready");
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }
  Serial.println("\nIP: " + WiFi.localIP().toString());
  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (!client) return;
  while (client.connected()) {
    float t = tmp.readObjTempC();
    client.println("T:" + String(t, 2));
    delay(500);
  }
  client.stop();
}
```

---

### AMG8833 / Panasonic Grid-EYE

**Library:** `Adafruit AMG88xx Library` (Arduino Library Manager)

```cpp
#include <ESP8266WiFi.h>   // use <WiFi.h> for ESP32
#include <Wire.h>
#include <Adafruit_AMG88xx.h>

const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASS = "YOUR_WIFI_PASSWORD";
const int   TCP_PORT  = 8888;

Adafruit_AMG88xx amg;
WiFiServer server(TCP_PORT);

void setup() {
  Serial.begin(115200);
  Wire.begin();
  if (!amg.begin()) { Serial.println("AMG8833 not found!"); while(1); }
  Serial.println("AMG8833 (8x8) ready");
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }
  Serial.println("\nIP: " + WiFi.localIP().toString());
  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (!client) return;
  while (client.connected()) {
    float pixels[64];
    amg.readPixels(pixels);
    String frame = "GRID:";
    for (int i = 0; i < 64; i++) {
      frame += String(pixels[i], 1);
      if (i < 63) frame += ",";
    }
    client.println(frame);
    delay(500);
  }
  client.stop();
}
```

---

### MLX90640

**Library:** `Adafruit MLX90640 Library` (Arduino Library Manager)

```cpp
#include <ESP8266WiFi.h>   // use <WiFi.h> for ESP32
#include <Wire.h>
#include <Adafruit_MLX90640.h>

const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASS = "YOUR_WIFI_PASSWORD";
const int   TCP_PORT  = 8888;

Adafruit_MLX90640 mlx640;
WiFiServer server(TCP_PORT);

void setup() {
  Serial.begin(115200);
  Wire.begin();
  if (!mlx640.begin(MLX90640_I2CADDR_DEFAULT, &Wire)) {
    Serial.println("MLX90640 not found!"); while(1);
  }
  mlx640.setMode(MLX90640_CHESS);
  mlx640.setResolution(MLX90640_ADC_18BIT);
  mlx640.setRefreshRate(MLX90640_8_HZ);
  Serial.println("MLX90640 (32x24) ready");
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }
  Serial.println("\nIP: " + WiFi.localIP().toString());
  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (!client) return;
  while (client.connected()) {
    float frame[768];
    if (mlx640.getFrame(frame) == 0) {
      String out = "MLX640:";
      for (int i = 0; i < 768; i++) {
        out += String(frame[i], 1);
        if (i < 767) out += ",";
      }
      client.println(out);
    }
    delay(250);
  }
  client.stop();
}
```

---

### MLX90621

**Library:** [MLX90621 Arduino Library](https://github.com/longjos/MLX90621_Arduino_Library) — install manually from GitHub

```cpp
#include <ESP8266WiFi.h>   // use <WiFi.h> for ESP32
#include <Wire.h>
#include <MLX90621.h>

const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASS = "YOUR_WIFI_PASSWORD";
const int   TCP_PORT  = 8888;

MLX90621 mlx621;
WiFiServer server(TCP_PORT);

void setup() {
  Serial.begin(115200);
  Wire.begin();
  mlx621.initialise(4);   // 4 Hz refresh rate
  Serial.println("MLX90621 (4x16) ready");
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }
  Serial.println("\nIP: " + WiFi.localIP().toString());
  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (!client) return;
  while (client.connected()) {
    mlx621.measure(true);
    String frame = "MLX621:";
    for (int i = 0; i < 64; i++) {
      frame += String(mlx621.getTemperature(i), 1);
      if (i < 63) frame += ",";
    }
    client.println(frame);
    delay(500);
  }
  client.stop();
}
```

---

### Omron D6T-44L

**Library:** None — uses raw I²C. **Wiring:** SDA/SCL same as other I²C sensors.

```cpp
#include <ESP8266WiFi.h>   // use <WiFi.h> for ESP32
#include <Wire.h>

const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASS = "YOUR_WIFI_PASSWORD";
const int   TCP_PORT  = 8888;

#define D6T_ADDR 0x0A
WiFiServer server(TCP_PORT);

void setup() {
  Serial.begin(115200);
  Wire.begin();
  Serial.println("D6T-44L ready");
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }
  Serial.println("\nIP: " + WiFi.localIP().toString());
  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (!client) return;
  while (client.connected()) {
    Wire.requestFrom(D6T_ADDR, 35);
    float pixels[16];
    int idx = 0;
    Wire.read(); Wire.read();  // skip PTAT bytes
    for (int i = 0; i < 16 && Wire.available() >= 2; i++) {
      int lo = Wire.read(), hi = Wire.read();
      pixels[idx++] = ((hi << 8) | lo) / 10.0f;
    }
    String frame = "D6T44:";
    for (int i = 0; i < 16; i++) {
      frame += String(pixels[i], 1);
      if (i < 15) frame += ",";
    }
    client.println(frame);
    delay(500);
  }
  client.stop();
}
```

---

### Omron D6T-8L

**Library:** None — uses raw I²C. **Wiring:** SDA/SCL same as other I²C sensors.

```cpp
#include <ESP8266WiFi.h>   // use <WiFi.h> for ESP32
#include <Wire.h>

const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASS = "YOUR_WIFI_PASSWORD";
const int   TCP_PORT  = 8888;

#define D6T_ADDR 0x0A
WiFiServer server(TCP_PORT);

void setup() {
  Serial.begin(115200);
  Wire.begin();
  Serial.println("D6T-8L ready");
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }
  Serial.println("\nIP: " + WiFi.localIP().toString());
  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (!client) return;
  while (client.connected()) {
    Wire.requestFrom(D6T_ADDR, 19);
    float pixels[8];
    int idx = 0;
    Wire.read(); Wire.read();  // skip PTAT bytes
    for (int i = 0; i < 8 && Wire.available() >= 2; i++) {
      int lo = Wire.read(), hi = Wire.read();
      pixels[idx++] = ((hi << 8) | lo) / 10.0f;
    }
    String frame = "D6T8:";
    for (int i = 0; i < 8; i++) {
      frame += String(pixels[i], 1);
      if (i < 7) frame += ",";
    }
    client.println(frame);
    delay(500);
  }
  client.stop();
}
```

---

## Sensor Wiring (I²C)

```
ESP8266          Sensor
─────────        ──────
3.3V      →      VCC
GND       →      GND
D2 (SDA)  →      SDA
D1 (SCL)  →      SCL
```

For ESP32: use GPIO 21 (SDA) and GPIO 22 (SCL).
D6T sensors use I²C at 3.3V — no additional wiring needed beyond SDA/SCL.

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

Built with ❤️ by [engrpanda](https://github.com/engrpanda)

Successor to [Wireless IR Temperature Scanner](https://github.com/engrpanda/Wireless-IR-Temperature-Scanner) · [Instructables](https://www.instructables.com/id/Wireless-IR-Temperature-Scanner/) · [Hackster.io](https://www.hackster.io/engrpandaece/wireless-ir-temperature-scanner-acbfd9)

</div>
