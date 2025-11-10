# Transformer Oil Monitoring System
## Focused on Oil Temperature & Level

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Hardware](https://img.shields.io/badge/Hardware-ESP32-blue.svg)](https://www.espressif.com/en/products/socs/esp32)
[![Focus](https://img.shields.io/badge/Focus-Oil_Monitoring-orange.svg)]()

## 📋 Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Monitored Parameters](#monitored-parameters)
- [Hardware Components](#hardware-components)
- [Pin Configuration](#pin-configuration)
- [Installation](#installation)
- [Blynk Dashboard Setup](#blynk-dashboard-setup)
- [Protection Logic](#protection-logic)
- [Usage](#usage)
- [Troubleshooting](#troubleshooting)

## 🔍 Overview

This project implements a **simplified, focused IoT monitoring system** for distribution transformers that tracks the **two most critical parameters: oil temperature and oil level**. The system continuously monitors these parameters and transmits data to a cloud-based IoT dashboard via ESP32 WiFi module.

This version removes unnecessary sensors and focuses entirely on:
- **Oil Temperature Monitoring** - Preventing transformer overheating
- **Oil Level Monitoring** - Preventing oil starvation and damage

## ✨ Key Features

- **Real-time Oil Temperature Monitoring** - Tracks temperature continuously (40-85°C range)
- **Real-time Oil Level Monitoring** - Measures oil level accurately (0-50cm)
- **IoT Integration** - Cloud-based dashboard via Blynk platform
- **Mobile Dashboard** - Access via smartphone app
- **Local LCD Display** - 16x2 display for on-site viewing
- **Automated Protection** - Two-stage temperature protection with fan control
- **Alert System** - Instant notifications for abnormal conditions
- **Cost-Effective** - Total hardware cost: $40-60 USD
- **Simple & Reliable** - Focused on essential parameters only

## 📊 Monitored Parameters

### 1. ⚡ Oil Temperature (Primary)
- **Sensor**: DHT11/DHT22
- **Range**: 40-85°C
- **Normal Range**: < 65°C
- **Fan Activation**: ≥ 70°C
- **Supply Trip**: ≥ 85°C
- **Update Rate**: Every 2 seconds

### 2. 💧 Oil Level (Primary)
- **Sensor**: HC-SR04 Ultrasonic
- **Range**: 0-50cm
- **Normal Level**: > 35cm
- **Warning Level**: 20-35cm
- **Critical Level**: < 10cm
- **Update Rate**: Every 2 seconds

## 🔧 Hardware Components

| Component | Specification | Qty | Cost | Purpose |
|-----------|--------------|-----|------|---------|
| **ESP32** | ESP32-WROOM-32 | 1 | $8-12 | Main microcontroller with WiFi |
| **Temperature Sensor** | DHT11 or DHT22 | 1 | $2-5 | Oil temperature measurement |
| **Ultrasonic Sensor** | HC-SR04 | 1 | $2-3 | Oil level measurement |
| **LCD Display** | 16x2 I2C | 1 | $4-6 | Local parameter display |
| **Relay Module** | 2-Channel 5V | 1 | $3-4 | Fan and supply control |
| **Cooling Fan** | 12V DC | 1 | $5-8 | Emergency cooling |
| **Power Supply** | 5V/3.3V 2A | 1 | $5-8 | Circuit power |
| **Miscellaneous** | Wires, Resistors | - | $5-10 | Connections, Components |
| **Enclosure** | IP65 Plastic | 1 | $8-12 | Weather-proof housing |

**Total Estimated Cost: $42-58 USD**

## 📍 Pin Configuration

### ESP32 GPIO Mapping

```
ESP32 GPIO    →    Component              Function
─────────────────────────────────────────────────────
GPIO 4        →    DHT11/DHT22            Oil Temperature Sensing
GPIO 18       →    HC-SR04 Trigger        Oil Level Measurement
GPIO 19       →    HC-SR04 Echo           Oil Level Echo
GPIO 21       →    LCD SDA                Display Data (I2C)
GPIO 22       →    LCD SCL                Display Clock (I2C)
GPIO 25       →    Relay Channel 1        Cooling Fan Control
GPIO 26       →    Relay Channel 2        Main Supply Control
5V/3.3V       →    Power Rails            Component Power
GND           →    Common Ground          Ground Reference
```

### Sensor Connections

#### DHT11/DHT22 Temperature Sensor
```
DHT Pin    →    ESP32
─────────────────────────────
VCC        →    3.3V
DATA       →    GPIO 4 (with 10kΩ pull-up to 3.3V)
GND        →    GND
```

#### HC-SR04 Ultrasonic Sensor (Oil Level)
```
HC-SR04    →    ESP32
─────────────────────────────
VCC        →    5V
TRIG       →    GPIO 18
ECHO       →    GPIO 19 (through voltage divider: 2kΩ + 1kΩ)
GND        →    GND
```

#### 16x2 I2C LCD Display
```
LCD        →    ESP32
─────────────────────────────
VCC        →    5V
GND        →    GND
SDA        →    GPIO 21
SCL        →    GPIO 22
```

#### Relay Module
```
Relay      →    ESP32
─────────────────────────────
VCC        →    5V
GND        →    GND
IN1        →    GPIO 25 (Cooling Fan)
IN2        →    GPIO 26 (Main Supply)
```

## 🚀 Installation

### Step 1: Hardware Setup
1. Purchase all components from the list above
2. Connect DHT11/DHT22 to GPIO 4 with 10kΩ pull-up resistor
3. Connect HC-SR04 ultrasonic sensor (with voltage divider for ECHO pin)
4. Connect LCD display on I2C pins (GPIO 21, 22)
5. Connect relays to GPIO 25 and 26
6. Mount sensors on transformer:
   - Temperature sensor near oil tank
   - Ultrasonic sensor above oil surface

### Step 2: Software Installation
1. **Install Arduino IDE** (v1.8.19+)
2. **Add ESP32 Board Support**:
   - File → Preferences
   - Add: `https://dl.espressif.com/dl/package_esp32_index.json`
   - Tools → Board Manager → Install ESP32

3. **Install Libraries**:
   - Sketch → Include Library → Manage Libraries
   - Install: **DHT sensor library** (Adafruit)
   - Install: **LiquidCrystal_I2C**
   - Install: **Blynk** (Volodymyr Shymanskyy)

4. **Upload Code**:
   - Copy the provided main code
   - Replace WiFi credentials and Blynk token
   - Connect ESP32 via USB
   - Tools → Board: ESP32 Dev Module
   - Tools → Port: (select your COM port)
   - Click Upload

### Step 3: Calibration

#### Temperature Calibration
```cpp
// Read sensor value with thermometer reference
// If ESP32 reads 50°C but thermometer shows 52°C:
#define TEMPERATURE_OFFSET 2.0  // Add 2°C offset
```

#### Oil Level Calibration
```cpp
// Measure tank height (from sensor to bottom when empty)
#define TANK_HEIGHT 50.0  // Adjust to your tank height
```

## 🌐 Blynk IoT Dashboard Setup

### Create Blynk Account & Device
1. Sign up at https://blynk.io
2. Create new template: "Oil Monitoring"
3. Select Hardware: ESP32, Connection: WiFi
4. Create device from template

### Configure Datastreams

| Virtual Pin | Parameter | Type | Min | Max | Unit |
|------------|-----------|------|-----|-----|------|
| V0 | Oil Temperature | Double | 40 | 100 | °C |
| V3 | Oil Level | Double | 0 | 50 | cm |
| V1 | Fan Status | Integer | 0 | 1 | - |
| V2 | Supply Status | Integer | 0 | 1 | - |
| V4 | Manual Fan Control | Integer | 0 | 1 | - |
| V5 | Manual Supply Control | Integer | 0 | 1 | - |

### Create Dashboard Widgets

1. **Gauge Widget for Temperature (V0)**
   - Min: 40°C, Max: 100°C
   - Color: Blue to Red gradient
   - Display: Real-time value

2. **Level Widget for Oil Level (V3)**
   - Min: 0cm, Max: 50cm
   - Color: Green to Red gradient
   - Display: Visual bar chart

3. **LED Indicators**
   - V1: Fan Status (shows when fan is active)
   - V2: Supply Status (shows if supply is connected)

4. **Button Switches (Manual Control)**
   - V4: Manual Fan Control
   - V5: Manual Supply Control

5. **Chart Widget (Historical Data)**
   - Add V0 and V3 for trend analysis
   - View 1 hour, 1 day, 1 week history

6. **Notification Widget**
   - Setup alerts for temperature alarms
   - Setup alerts for low oil level

### Get Your Auth Token
1. Go to Devices → Your Device → Edit
2. Copy the **Auth Token**
3. Paste into code: `#define BLYNK_AUTH_TOKEN "Your_Token_Here"`

## 🛡️ Protection Logic

### Temperature-Based Protection

```
Oil Temperature Status:
├─ < 65°C       → ✓ Normal Operation
│  └─ Action: None
│
├─ 65-70°C      → ⚠️  Warning Zone
│  └─ Action: Monitor (no action)
│
├─ 70-85°C      → ⚠️  Fan Activation
│  └─ Action: TURN ON COOLING FAN
│
└─ ≥ 85°C       → 🚨 CRITICAL
   └─ Action: DISCONNECT MAIN SUPPLY (Emergency shutdown)
```

### Oil Level-Based Protection

```
Oil Level Status:
├─ > 35cm       → ✓ Normal Level
│  └─ Action: None
│
├─ 20-35cm      → ⚠️  Warning
│  └─ Action: Send Alert (maintenance needed)
│
└─ < 10cm       → 🚨 CRITICAL
   └─ Action: DISCONNECT SUPPLY + Alert Operator
```

## 💻 Code Configuration

Edit these parameters in the code:

```cpp
// WiFi Credentials
#define WIFI_SSID "YourWiFiName"
#define WIFI_PASSWORD "YourPassword"

// Blynk Token
#define BLYNK_AUTH_TOKEN "Your_Auth_Token_Here"

// Temperature Thresholds (°C)
#define MAX_OIL_TEMPERATURE 85.0        // Supply trip
#define FAN_TRIGGER_TEMPERATURE 70.0    // Fan activation
#define NORMAL_TEMP_MAX 65.0            // Normal range

// Oil Level Thresholds (cm)
#define TANK_HEIGHT 50.0                // Total tank height
#define NORMAL_OIL_LEVEL 35.0           // Normal operating level
#define WARNING_OIL_LEVEL 20.0          // Warning threshold
#define CRITICAL_OIL_LEVEL 10.0         // Critical threshold

// Calibration
#define TEMPERATURE_OFFSET 0.0          // Temperature correction
```

## 📖 Usage

### Normal Operation
1. Power on the ESP32 system
2. LCD will show "System Ready"
3. System automatically connects to WiFi
4. Data uploads to Blynk dashboard every 5 seconds

### Viewing Data

**Local Display (LCD)**
```
Line 1: T:62.5C      FAN
Line 2: L:38.2cm     OK
```

**Mobile App (Blynk)**
- Temperature gauge showing real-time value
- Oil level bar chart
- Status indicators for fan and supply
- Historical graphs

**Serial Monitor** (115200 baud)
- Real-time sensor readings
- Protection logic status
- WiFi and Blynk connection status
- Alert messages

### Alert Conditions

The system will send alerts when:
- Oil temperature exceeds 70°C (fan activation)
- Oil temperature reaches 85°C (supply disconnection)
- Oil level falls below 20cm (low warning)
- Oil level falls below 10cm (critical - supply disconnection)

### Manual Controls

Use Blynk app buttons to manually:
- Activate/deactivate cooling fan
- Connect/disconnect main supply

⚠️ **Use manual controls carefully! Disconnecting supply without cause may damage equipment.**

## 📊 Serial Monitor Output Example

```
========================================
Oil Temperature & Level Monitoring
Transformer Monitoring System v2.0
========================================

[WiFi] Connecting to: MyWiFi
.....
[WiFi] ✓ Connected successfully!
[WiFi] IP Address: 192.168.1.100

[Sensors] Initializing Oil Monitoring Sensors...
[Sensors] ✓ Temperature sensor initialized
[Sensors] ✓ Ultrasonic sensor initialized

[Temp] Oil Temperature: 62.5 °C
[Level] Oil Level: 38.2 cm
[Blynk] Dashboard updated | Temp: 62.5°C | Level: 38.2cm

[Protection] Temperature normal - Fan off
```

## 🔧 Troubleshooting

### Temperature Sensor Not Reading
- Check DHT11/DHT22 connections
- Verify GPIO 4 connection
- Add 10kΩ pull-up resistor if missing
- Check sensor power (3.3V)

### Oil Level Showing Zero
- Check ultrasonic sensor power (5V)
- Verify GPIO 18 (trigger) and 19 (echo) connections
- Add voltage divider (2kΩ + 1kΩ) on ECHO pin
- Ensure clear path from sensor to oil surface

### WiFi Disconnects
- Check WiFi signal strength
- Verify SSID is 2.4GHz (not 5GHz)
- Ensure password is correct
- Check ESP32 power supply stability

### Blynk Dashboard Not Updating
- Verify Auth Token is correct and active
- Check internet connection on ESP32
- Monitor serial output for errors
- Ensure Blynk account has active subscription

### LCD Display Blank
- Adjust contrast potentiometer on backpack
- Verify I2C address (0x27 or 0x3F)
- Check GPIO 21 (SDA) and 22 (SCL) connections
- Verify 5V power to LCD

### Relays Not Switching
- Check relay module 5V power
- Verify GPIO 25 and 26 connections
- Test relay coil with multimeter (should be ~70Ω)
- Listen for relay click when triggered

## 📈 Maintenance Schedule

### Weekly
- Check Blynk dashboard for consistency
- Review historical trends
- Verify WiFi connectivity

### Monthly
- Visual inspection of sensors
- Check LCD display functionality
- Test relay operation manually
- Clean sensor surfaces

### Quarterly
- Recalibrate temperature sensor
- Verify oil level accuracy
- Test protection mechanisms
- Update firmware if available

### Annually
- Full system functional test
- Replace degraded components
- Review and update threshold settings

## 🎯 Specifications

- **Microcontroller**: ESP32 (240 MHz, 520KB SRAM)
- **WiFi**: 802.11 b/g/n (2.4 GHz only)
- **Sampling Rate**: 2 seconds
- **Dashboard Update**: 5 seconds
- **Sensor Accuracy**: ±1-2% (after calibration)
- **Operating Temp**: -10°C to 80°C (system)
- **Temperature Range**: 40-85°C (monitoring)
- **Oil Level Range**: 0-50cm (monitoring)
- **Power Supply**: 5V 2A minimum
- **Enclosure Rating**: IP65 (dust/water resistant)

## 📄 License

MIT License - Free to use, modify, and distribute.

## 🤝 Contributing

Contributions welcome! Please:
- Test thoroughly with hardware
- Update documentation
- Follow code style guidelines
- Submit pull requests with clear descriptions

## 📞 Support

- Check troubleshooting section above
- Review code comments for detailed explanations
- Open GitHub issues for bugs
- Check Blynk documentation for dashboard setup

## 🚀 Future Enhancements

- Add SD card data logging
- Implement predictive maintenance using ML
- Add email alerts
- Support multiple transformers
- OTA firmware updates
- Battery backup for continuous monitoring
- Advanced analytics dashboard

---

**Made with ❤️ for Electrical Engineers**

**This simplified version focuses on the two most critical parameters: Oil Temperature and Oil Level**