# Transformer Health Monitoring System using IoT

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Hardware](https://img.shields.io/badge/Hardware-ESP32-blue.svg)](https://www.espressif.com/en/products/socs/esp32)
[![Platform](https://img.shields.io/badge/Platform-Arduino-green.svg)](https://www.arduino.cc/)

## 📋 Table of Contents
- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [Software Requirements](#software-requirements)
- [Circuit Diagram](#circuit-diagram)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Monitoring Parameters](#monitoring-parameters)
- [Protection Mechanisms](#protection-mechanisms)
- [IoT Dashboard](#iot-dashboard)
- [Project Structure](#project-structure)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgments](#acknowledgments)

## 🔍 Overview

This project implements a real-time condition monitoring system for distribution transformers using Internet of Things (IoT) technology. The system continuously monitors critical transformer health parameters and transmits data to a cloud-based IoT dashboard through an ESP32 microcontroller, enabling remote monitoring and predictive maintenance.

Distribution transformers are critical components in power distribution networks, and their failure can lead to significant power outages and economic losses. This monitoring system provides early warning of potential failures by tracking key operational parameters in real-time.

## ✨ Features

- **Real-time Monitoring**: Continuous monitoring of transformer health parameters
- **IoT Integration**: Cloud-based data logging and remote access via ESP32 Wi-Fi module
- **Local Display**: LCD display for on-site parameter visualization
- **Multi-parameter Sensing**: Monitors 5 critical transformer parameters
- **Automated Protection**: Intelligent protection mechanisms for transformer safety
- **Alert System**: Immediate notifications for abnormal operating conditions
- **Data Logging**: Historical data storage for trend analysis
- **Low Cost**: Affordable implementation using readily available components
- **Scalable**: Can be deployed across multiple transformers in a distribution network

## 🏗️ System Architecture

The system consists of three main layers:

1. **Sensing Layer**: Multiple sensors measure transformer parameters
2. **Processing Layer**: ESP32 microcontroller processes sensor data and implements control logic
3. **Application Layer**: IoT dashboard (Blynk/ThingSpeak) for remote monitoring and data visualization

```
[Sensors] → [ESP32 + Control Logic] → [Local LCD Display]
                      ↓
            [Wi-Fi Communication]
                      ↓
            [IoT Cloud Dashboard]
```

## 🔧 Hardware Components

| Component | Specification | Quantity | Purpose |
|-----------|--------------|----------|---------|
| **Microcontroller** | ESP32 Development Board | 1 | Main processing unit with built-in Wi-Fi |
| **Temperature Sensor** | DHT11/DHT22 or DS18B20 | 1 | Monitors transformer oil temperature |
| **Voltage Sensor** | ZMPT101B AC Voltage Sensor | 1 | Measures transformer voltage levels |
| **Current Sensor** | ACS712 (20A/30A) | 1 | Monitors load current |
| **Ultrasonic Sensor** | HC-SR04 | 1 | Measures transformer oil level |
| **Color Sensor** | TCS3200 or Manual Indicator | 1 | Monitors silica gel color (moisture indication) |
| **LCD Display** | 16x2 or 20x4 I2C LCD | 1 | Local parameter display |
| **Relay Module** | 5V Relay (2-Channel) | 1 | Controls cooling fan and main supply |
| **Cooling Fan** | 12V DC Fan | 1 | Emergency cooling system |
| **Circuit Breaker/Contactor** | As per transformer rating | 1 | Main supply disconnection |
| **Power Supply** | 5V/3.3V Power Adapter | 1 | Powers the monitoring circuit |
| **PCB/Breadboard** | General Purpose | 1 | Circuit assembly |
| **Connecting Wires** | Jumper Wires | As needed | Electrical connections |
| **Enclosure** | IP65 Rated Box | 1 | Weather-proof housing |

### Optional Components
- Battery backup (for continuous monitoring during power failures)
- SD card module (for local data logging)
- Buzzer (for local alarms)

## 💻 Software Requirements

### Development Environment
- **Arduino IDE** (v1.8.19 or later) or **PlatformIO**
- **ESP32 Board Support Package** for Arduino
- **USB to UART Driver** (CP210x or CH340)

### Required Libraries
```cpp
// Core Libraries
#include <WiFi.h>
#include <Wire.h>

// Sensor Libraries
#include <DHT.h>              // DHT11/DHT22 temperature sensor
#include <LiquidCrystal_I2C.h> // I2C LCD display

// IoT Platform Libraries (choose one)
#include <BlynkSimpleEsp32.h>  // For Blynk IoT
// OR
#include <ThingSpeak.h>        // For ThingSpeak IoT
```

### IoT Platform Setup
- **Blynk IoT** (Recommended): Create account at [blynk.io](https://blynk.io)
- **ThingSpeak**: Create account at [thingspeak.com](https://thingspeak.com)
- **MQTT Broker** (Optional): Mosquitto or other MQTT server

## 📊 Circuit Diagram

### Pin Configuration

#### ESP32 Pin Mapping
```
ESP32 GPIO    →    Component
----------------------------------------
GPIO 4        →    DHT11/DHT22 Data Pin
GPIO 18       →    Ultrasonic Trigger
GPIO 19       →    Ultrasonic Echo
GPIO 21       →    LCD SDA (I2C)
GPIO 22       →    LCD SCL (I2C)
GPIO 25       →    Relay 1 (Cooling Fan)
GPIO 26       →    Relay 2 (Main Supply)
GPIO 32       →    Voltage Sensor Output
GPIO 33       →    Current Sensor Output
GPIO 34       →    Color Sensor Output
GND           →    Common Ground
5V/3.3V       →    Power Supply
```

### Sensor Connections

#### DHT11/DHT22 Temperature Sensor
```
DHT Pin    →    ESP32
VCC        →    3.3V
DATA       →    GPIO 4 (with 10kΩ pull-up resistor)
GND        →    GND
```

#### HC-SR04 Ultrasonic Sensor (Oil Level)
```
HC-SR04    →    ESP32
VCC        →    5V
TRIG       →    GPIO 18
ECHO       →    GPIO 19 (use voltage divider 5V to 3.3V)
GND        →    GND
```

#### ZMPT101B Voltage Sensor
```
ZMPT101B   →    ESP32
VCC        →    5V
OUT        →    GPIO 32 (ADC1 Channel)
GND        →    GND
```

#### ACS712 Current Sensor
```
ACS712     →    ESP32
VCC        →    5V
OUT        →    GPIO 33 (ADC1 Channel)
GND        →    GND
```

#### 16x2 I2C LCD Display
```
LCD        →    ESP32
VCC        →    5V
GND        →    GND
SDA        →    GPIO 21
SCL        →    GPIO 22
```

#### Relay Module
```
Relay      →    ESP32
VCC        →    5V
GND        →    GND
IN1        →    GPIO 25 (Fan Control)
IN2        →    GPIO 26 (Supply Control)
```

## 🚀 Installation

### Step 1: Hardware Assembly
1. Connect all sensors to the ESP32 according to the pin configuration table
2. Ensure proper power supply ratings for all components
3. Use appropriate voltage dividers for sensors requiring 3.3V input
4. Mount the circuit in a weather-proof enclosure
5. Install sensors at appropriate locations on the transformer

### Step 2: Software Setup
1. **Install Arduino IDE**
   ```bash
   # Download from: https://www.arduino.cc/en/software
   ```

2. **Install ESP32 Board Support**
   - Open Arduino IDE
   - Go to File → Preferences
   - Add to "Additional Board Manager URLs":
   ```
   https://dl.espressif.com/dl/package_esp32_index.json
   ```
   - Go to Tools → Board → Board Manager
   - Search for "ESP32" and install

3. **Install Required Libraries**
   - Go to Sketch → Include Library → Manage Libraries
   - Install the following:
     - DHT sensor library by Adafruit
     - LiquidCrystal I2C
     - Blynk library (or ThingSpeak)
     - WiFi (included with ESP32)

4. **Clone this Repository**
   ```bash
   git clone https://github.com/yourusername/transformer-health-monitoring.git
   cd transformer-health-monitoring
   ```

### Step 3: Configuration
1. Open `config.h` and update the following:
   ```cpp
   // WiFi Credentials
   #define WIFI_SSID "Your_WiFi_SSID"
   #define WIFI_PASSWORD "Your_WiFi_Password"
   
   // Blynk Auth Token
   #define BLYNK_AUTH "Your_Blynk_Auth_Token"
   
   // Sensor Calibration Values
   #define VOLTAGE_CALIBRATION 1.0
   #define CURRENT_CALIBRATION 1.0
   ```

2. Set threshold values for alarms:
   ```cpp
   // Protection Thresholds
   #define MAX_TEMPERATURE 75.0      // °C
   #define FAN_TRIGGER_TEMP 65.0     // °C
   #define MAX_VOLTAGE 260.0         // V
   #define MIN_VOLTAGE 200.0         // V
   #define MAX_CURRENT 15.0          // A
   #define MIN_OIL_LEVEL 10.0        // cm
   ```

### Step 4: Upload Code
1. Connect ESP32 to computer via USB
2. Select correct board: Tools → Board → ESP32 Dev Module
3. Select correct port: Tools → Port → (your COM port)
4. Click Upload button
5. Monitor serial output to verify successful connection

## ⚙️ Configuration

### WiFi Setup
The system automatically connects to the configured WiFi network. If connection fails, it will retry every 5 seconds.

### IoT Dashboard Setup

#### Blynk Configuration
1. Create a new template in Blynk Console
2. Add the following datastreams:
   - V0: Temperature (Virtual Pin)
   - V1: Voltage (Virtual Pin)
   - V2: Current (Virtual Pin)
   - V3: Oil Level (Virtual Pin)
   - V4: Silica Gel Status (Virtual Pin)
   - V5: Fan Status (Virtual Pin)
   - V6: System Status (Virtual Pin)

3. Create dashboard widgets:
   - Gauge widgets for temperature, voltage, current
   - Level widget for oil level
   - LED indicators for status
   - Chart widgets for historical data

4. Setup notifications for alerts

## 📖 Usage

### Starting the System
1. Power on the ESP32 module
2. System will connect to WiFi automatically
3. LCD display will show system status
4. Data will start uploading to IoT platform

### Monitoring Data
- **Local Display**: View real-time parameters on LCD
- **IoT Dashboard**: Access remote dashboard via mobile app or web browser
- **Serial Monitor**: Debug and detailed logs (115200 baud rate)

### Normal Operation
```
LCD Display Format:
Line 1: Temp:45°C V:230V
Line 2: I:8.5A Oil:OK
```

## 📊 Monitoring Parameters

### 1. Oil Temperature
- **Normal Range**: 40-70°C
- **Action at 65°C**: Activate cooling fan
- **Action at 75°C**: Disconnect main supply
- **Sensor**: DHT11/DHT22 or DS18B20

### 2. Voltage Level
- **Normal Range**: 220V ± 10% (198-242V for 230V system)
- **Action**: Disconnect supply if out of range
- **Sensor**: ZMPT101B AC voltage sensor

### 3. Current Level
- **Normal Range**: 0 - Rated Current
- **Action**: Disconnect supply if overcurrent detected
- **Sensor**: ACS712 Hall-effect current sensor

### 4. Oil Level
- **Normal Range**: > 20cm (depends on transformer size)
- **Action**: Alert if below minimum level
- **Sensor**: HC-SR04 ultrasonic sensor

### 5. Silica Gel Color
- **Blue**: Good (dry, functional)
- **Pink/White**: Saturated (needs replacement)
- **Action**: Alert for maintenance
- **Sensor**: TCS3200 color sensor or visual indicator

## 🛡️ Protection Mechanisms

### Two-Stage Temperature Protection

#### Stage 1: Cooling Activation
```cpp
if (temperature >= FAN_TRIGGER_TEMP && temperature < MAX_TEMPERATURE) {
    activateCoolingFan();
    sendAlert("Cooling fan activated");
}
```

#### Stage 2: Emergency Shutdown
```cpp
if (temperature >= MAX_TEMPERATURE) {
    disconnectMainSupply();
    sendAlert("CRITICAL: Transformer overheating - Supply disconnected");
}
```

## 📁 Project Structure

```
transformer-health-monitoring/
│
├── src/
│   ├── main.cpp                 # Main program file
│   ├── config.h                 # Configuration settings
│   ├── sensors.cpp              # Sensor reading functions
│   └── protection.cpp           # Protection logic
│
├── hardware/
│   ├── circuit_diagram.png      # Complete circuit schematic
│   ├── pcb_layout.png           # PCB layout design
│   └── bom.xlsx                 # Bill of Materials
│
├── docs/
│   ├── installation_guide.md    # Detailed installation manual
│   ├── user_manual.md           # User operation manual
│   └── troubleshooting.md       # Common issues and solutions
│
├── examples/
│   ├── sensor_test/             # Individual sensor test codes
│   └── wifi_test/               # WiFi connection test
│
├── .gitignore
├── LICENSE
└── README.md
```

## 🔧 Troubleshooting

### WiFi Connection Issues
**Problem**: ESP32 not connecting to WiFi
**Solution**:
- Verify SSID and password in config.h
- Check WiFi signal strength
- Ensure 2.4GHz WiFi (ESP32 doesn't support 5GHz)
- Restart router and ESP32

### Sensor Reading Errors
**Problem**: Incorrect or no sensor readings
**Solution**:
- Check sensor connections and power supply
- Verify pin assignments in code
- Test sensors individually using example codes
- Calibrate sensors using known reference values

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Thanks to the ESP32 community for excellent documentation
- Blynk team for the IoT platform
- All contributors and testers
- Open-source sensor library developers

## 📞 Support

For questions and support:
- **Issues**: [GitHub Issues](https://github.com/yourusername/transformer-health-monitoring/issues)
- **Discussions**: [GitHub Discussions](https://github.com/yourusername/transformer-health-monitoring/discussions)

## 📈 Future Enhancements

- [ ] Add dissolved gas analysis (DGA) capability
- [ ] Implement machine learning for predictive maintenance
- [ ] Add support for multiple transformers
- [ ] Develop mobile app for iOS and Android
- [ ] Integrate with SCADA systems
- [ ] Implement OTA (Over-The-Air) firmware updates

---

**Star ⭐ this repository if you find it useful!**

**Made with ❤️ for the Electrical Engineering Community**