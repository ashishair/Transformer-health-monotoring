/*
 * SIMPLIFIED CODE - Oil Temperature & Level Monitoring
 * 
 * This is the main Arduino/ESP32 code for measuring:
 * 1. Oil Temperature (DHT11/DHT22)
 * 2. Oil Level (HC-SR04 Ultrasonic)
 * 
 * Features: Blynk IoT, Two-stage temperature protection, LCD display
 * 
 * COPY THIS CODE TO: src/main.cpp or main.ino
 * Then update the WiFi and Blynk credentials below
 */

#include <WiFi.h>
#include <Wire.h>
#include <DHT.h>
#include <LiquidCrystal_I2C.h>
#include <BlynkSimpleEsp32.h>

// ==================== CONFIGURATION - EDIT THESE ====================
#define WIFI_SSID "Your_WiFi_SSID"
#define WIFI_PASSWORD "Your_WiFi_Password"
#define BLYNK_AUTH_TOKEN "Your_Blynk_Auth_Token"

// ==================== PIN DEFINITIONS ====================
#define TEMPERATURE_SENSOR_PIN 4
#define ULTRASONIC_TRIG_PIN 18
#define ULTRASONIC_ECHO_PIN 19
#define RELAY_PIN_FAN 25
#define RELAY_PIN_SUPPLY 26

// ==================== I2C PINS ====================
#define I2C_SDA 21
#define I2C_SCL 22

// ==================== SENSOR SETUP ====================
#define DHT_TYPE DHT11
DHT dht(TEMPERATURE_SENSOR_PIN, DHT_TYPE);
LiquidCrystal_I2C lcd(0x27, 16, 2);

// ==================== THRESHOLDS ====================
#define MAX_OIL_TEMPERATURE 85.0
#define FAN_TRIGGER_TEMPERATURE 70.0
#define TANK_HEIGHT 50.0
#define CRITICAL_OIL_LEVEL 10.0
#define WARNING_OIL_LEVEL 20.0

// ==================== GLOBAL VARIABLES ====================
float oilTemperature = 0.0;
float oilLevel = 0.0;
bool fanActive = false;
bool supplyConnected = true;
unsigned long lastSensorRead = 0;
unsigned long lastBlynkUpdate = 0;
unsigned long lastDisplayUpdate = 0;

// ==================== SETUP ====================
void setup() {
  Serial.begin(115200);
  delay(100);
  
  Serial.println("\n=== Oil Temperature & Level Monitoring ===\n");
  
  // Setup I2C and LCD
  Wire.begin(I2C_SDA, I2C_SCL);
  lcd.init();
  lcd.backlight();
  lcd.print("Oil Monitor");
  lcd.setCursor(0, 1);
  lcd.print("Starting...");
  
  // Setup WiFi
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  int attempts = 0;
  while (WiFi.status() != WL_CONNECTED && attempts < 20) {
    delay(500);
    Serial.print(".");
    attempts++;
  }
  
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\n[WiFi] Connected!");
    Serial.print("[IP] ");
    Serial.println(WiFi.localIP());
  }
  
  // Setup sensors
  dht.begin();
  pinMode(ULTRASONIC_TRIG_PIN, OUTPUT);
  pinMode(ULTRASONIC_ECHO_PIN, INPUT);
  
  // Setup relays
  pinMode(RELAY_PIN_FAN, OUTPUT);
  pinMode(RELAY_PIN_SUPPLY, OUTPUT);
  digitalWrite(RELAY_PIN_FAN, LOW);
  digitalWrite(RELAY_PIN_SUPPLY, HIGH);
  
  // Blynk connection
  Blynk.begin(BLYNK_AUTH_TOKEN, WIFI_SSID, WIFI_PASSWORD);
  
  Serial.println("[INIT] System Ready!\n");
  lcd.clear();
  lcd.print("System Ready");
  delay(2000);
  lcd.clear();
}

// ==================== MAIN LOOP ====================
void loop() {
  Blynk.run();
  
  unsigned long currentMillis = millis();
  
  // Read sensors every 2 seconds
  if (currentMillis - lastSensorRead >= 2000) {
    lastSensorRead = currentMillis;
    
    readOilTemperature();
    readOilLevel();
    checkTemperatureProtection();
    checkOilLevelStatus();
  }
  
  // Update display every 1 second
  if (currentMillis - lastDisplayUpdate >= 1000) {
    lastDisplayUpdate = currentMillis;
    updateDisplay();
  }
  
  // Update Blynk every 5 seconds
  if (currentMillis - lastBlynkUpdate >= 5000) {
    lastBlynkUpdate = currentMillis;
    Blynk.virtualWrite(V0, oilTemperature);
    Blynk.virtualWrite(V3, oilLevel);
  }
}

// ==================== TEMPERATURE READING ====================
void readOilTemperature() {
  oilTemperature = dht.readTemperature();
  
  if (isnan(oilTemperature)) {
    Serial.println("[ERROR] Temperature sensor failed!");
    oilTemperature = -1.0;
    return;
  }
  
  Serial.print("[Temp] ");
  Serial.print(oilTemperature);
  Serial.println(" °C");
}

// ==================== OIL LEVEL READING ====================
void readOilLevel() {
  // Send pulse
  digitalWrite(ULTRASONIC_TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(ULTRASONIC_TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(ULTRASONIC_TRIG_PIN, LOW);
  
  // Read duration
  long duration = pulseIn(ULTRASONIC_ECHO_PIN, HIGH, 30000);
  float distance = duration * 0.034 / 2.0;
  
  // Calculate oil level
  oilLevel = TANK_HEIGHT - distance;
  if (oilLevel < 0) oilLevel = 0;
  if (oilLevel > TANK_HEIGHT) oilLevel = TANK_HEIGHT;
  
  Serial.print("[Level] ");
  Serial.print(oilLevel);
  Serial.println(" cm");
}

// ==================== TEMPERATURE PROTECTION ====================
void checkTemperatureProtection() {
  // Normal: disable fan
  if (oilTemperature < 65.0) {
    if (fanActive) {
      digitalWrite(RELAY_PIN_FAN, LOW);
      fanActive = false;
      Serial.println("[Fan] OFF");
    }
    return;
  }
  
  // Warning: activate fan
  if (oilTemperature >= FAN_TRIGGER_TEMPERATURE && oilTemperature < MAX_OIL_TEMPERATURE) {
    if (!fanActive) {
      digitalWrite(RELAY_PIN_FAN, HIGH);
      fanActive = true;
      Serial.println("[Fan] ON - Temperature high!");
      Blynk.virtualWrite(V1, 1);
    }
  }
  
  // Critical: disconnect supply
  if (oilTemperature >= MAX_OIL_TEMPERATURE) {
    if (supplyConnected) {
      digitalWrite(RELAY_PIN_SUPPLY, LOW);
      supplyConnected = false;
      Serial.println("[CRITICAL] Supply disconnected!");
      Blynk.virtualWrite(V2, 0);
    }
  }
}

// ==================== OIL LEVEL STATUS ====================
void checkOilLevelStatus() {
  if (oilLevel > WARNING_OIL_LEVEL) {
    return; // Normal
  }
  
  if (oilLevel > CRITICAL_OIL_LEVEL) {
    Serial.println("[WARNING] Low oil level!");
    return;
  }
  
  // Critical level
  if (oilLevel <= CRITICAL_OIL_LEVEL && supplyConnected) {
    digitalWrite(RELAY_PIN_SUPPLY, LOW);
    supplyConnected = false;
    Serial.println("[CRITICAL] Oil level low - Supply disconnected!");
    Blynk.virtualWrite(V2, 0);
  }
}

// ==================== LCD DISPLAY UPDATE ====================
void updateDisplay() {
  lcd.clear();
  
  // Line 1: Temperature
  lcd.setCursor(0, 0);
  lcd.print("T:");
  if (oilTemperature < 0) {
    lcd.print("ERR");
  } else {
    lcd.print(oilTemperature, 1);
    lcd.print("C");
  }
  
  // Status indicator
  if (oilTemperature >= MAX_OIL_TEMPERATURE) {
    lcd.setCursor(9, 0);
    lcd.print("CRIT!");
  } else if (fanActive) {
    lcd.setCursor(11, 0);
    lcd.print("FAN");
  }
  
  // Line 2: Oil Level
  lcd.setCursor(0, 1);
  lcd.print("L:");
  lcd.print(oilLevel, 1);
  lcd.print("cm");
  
  // Level status
  if (oilLevel <= CRITICAL_OIL_LEVEL) {
    lcd.setCursor(9, 1);
    lcd.print("CRIT!");
  } else if (oilLevel <= WARNING_OIL_LEVEL) {
    lcd.setCursor(10, 1);
    lcd.print("LOW");
  } else {
    lcd.setCursor(11, 1);
    lcd.print("OK");
  }
}

// ==================== BLYNK CONTROLS ====================

// Manual fan control
BLYNK_WRITE(V4) {
  int value = param.asInt();
  digitalWrite(RELAY_PIN_FAN, value ? HIGH : LOW);
  fanActive = value;
}

// Manual supply control
BLYNK_WRITE(V5) {
  int value = param.asInt();
  digitalWrite(RELAY_PIN_SUPPLY, value ? HIGH : LOW);
  supplyConnected = value;
}

// ==================== END ====================