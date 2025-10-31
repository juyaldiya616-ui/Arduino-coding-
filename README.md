# Arduino-coding-
Arduino coding for project 


#include <SoftwareSerial.h>

#define SENSOR_PIN A0
#define LED_PIN 13
#define BT_RX 10
#define BT_TX 11

SoftwareSerial BT(BT_RX, BT_TX);

// ---- CONFIGURE THESE FOR YOUR HARDWARE ----
const float R1 = 0.0f;    // No divider used
const float R2 = 1.0f;
const float VREF = 5.00f; // Arduino reference voltage
const int   SAMPLES = 20; // Number of ADC samples to average
const float V_CALIB_FACTOR = 1.0f; // Calibration factor
// -------------------------------------------

// ---- 3.7V Li-ion Battery Range ----
// 4.20V → 100% (fully charged)
// 3.70V → ~50%
// 3.00V → 0% (empty)
const float BATTERY_MIN_VOLTAGE = 3.0f;  // Empty battery
const float BATTERY_MAX_VOLTAGE = 4.2f;  // Full battery
// ----------------------------------------

void setup() {
  Serial.begin(9600);
  BT.begin(9600);
  pinMode(LED_PIN, OUTPUT);
  Serial.println("🔋 Starting 3.7V Li-ion Battery Monitor...");
}

float readVoltageAtPin_avg() {
  unsigned long sum = 0;
  for (int i = 0; i < SAMPLES; ++i) {
    sum += analogRead(SENSOR_PIN);
    delay(3);
  }
  float avgADC = (float)sum / (float)SAMPLES;
  float voltageAtPin = avgADC * (VREF / 1023.0f);
  return voltageAtPin;
}

float clampf(float x, float lo, float hi) {
  if (x < lo) return lo;
  if (x > hi) return hi;
  return x;
}

void loop() {
  // 1️⃣ Read averaged ADC voltage
  float vAtPin = readVoltageAtPin_avg();

  // 2️⃣ Calculate actual battery voltage
  float battVoltage = vAtPin * ((R1 + R2) / R2);
  battVoltage *= V_CALIB_FACTOR;
  battVoltage = clampf(battVoltage, 0.0f, 5.0f);

  // 3️⃣ Calculate battery percentage
  float percent = ((battVoltage - BATTERY_MIN_VOLTAGE) /
                   (BATTERY_MAX_VOLTAGE - BATTERY_MIN_VOLTAGE)) * 100.0f;
  percent = clampf(percent, 0.0f, 100.0f);

  // 4️⃣ LED indicator — ON if battery > 4V
  if (battVoltage > 4.0) digitalWrite(LED_PIN, HIGH);
  else digitalWrite(LED_PIN, LOW);

  // 5️⃣ Print to Serial Monitor
  Serial.print("ADC Voltage: ");
  Serial.print(vAtPin, 3);
  Serial.print(" V  |  Battery Voltage: ");
  Serial.print(battVoltage, 3);
  Serial.print(" V  |  Battery: ");
  Serial.print(percent, 1);
  Serial.println(" %");

  // 6️⃣ Send to Bluetooth (HC-05)
  BT.print("ADC: ");
  BT.print(vAtPin, 2);
  BT.print(" V | Battery: ");
  BT.print(battVoltage, 2);
  BT.print(" V | ");
  BT.print(percent, 0);
  BT.println("%");

  // 🔁 Update every 10 seconds
  delay(10000);
}


---

📊 Output Example (Serial Monitor / Bluetooth)

ADC Voltage: 3.682 V  |  Battery Voltage: 3.682 V  |  Battery: 56.8 %

Yani:

ADC Voltage → A0 pin pe milne wala voltage

Battery Voltage → Actual battery ka voltage

Battery % → 3.0 V = 0%, 3.7 V = ~50%, 4.2 V = 100%



