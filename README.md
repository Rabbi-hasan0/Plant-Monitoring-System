## Plant-Monitoring-System - Iot Project
### Project Documents
1. [Project Report](https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Papers%20of%20project/IoT%20Report.pdf)
2. [Project Presentation Slide](https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Papers%20of%20project/Presentation.pdf)


<p align="center">
      <strong>Our project Final Look</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/WhatsApp%20Image%202025-05-21%20at%201.27.38%20PM.jpeg" width="300" height="300">
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/Full-setup.jpeg" width="500" height="300">
</p>


<h3 align="left">All Components with Name</h3>
<table>
  <tr>
    <td align="center">
      <strong>1. Soil Moisture Sensor</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/soil-sensor.jpg" width="200" height="200">
    </td>
    <td align="center">
      <strong>2. DHT11</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/dht11-digital-temperature-and-humidity-sensor.jpg" width="200" height="200">
    </td>
    <td align="center">
      <strong>3. ESP8266-NodeMCU</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/ESP8266-NodeMCU.jpg" width="200" height="200">
    </td>
  </tr>
  <tr>
    <td align="center">
      <strong>4. Relay</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/1-channel-5v-relay-board-module-robotics-bangladesh.jpg" width="200" height="200">
    </td>
    <td align="center">
      <strong>5. Pump</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/mini-pump.jpg" width="200" height="200">
    </td>
    <td align="center">
      <strong>6. LCD-Display-(16X2)</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/LCD-Display-(16X2).jpg" width="200" height="200">
    </td>
  </tr>
  <tr>
    <td align="center">
      <strong>7. 5v Battery</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/5v-battery.jpg" width="200" height="200">
    </td>
    <td align="center">
      <strong>8. Battery Box</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/battery-box.jpg" width="200" height="200">
    </td>
    <td align="center">
      <strong>9. Battery Charger</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/charger.png" width="200" height="200">
    </td>
  </tr>
  <tr>
    <td align="center">
      <strong>10. Bread-Board</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/board.jpg" width="200" height="200">
    </td>
    <td align="center">
      <strong>11. Wire</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/Male-to-Female-Jumper-Wire.jpg" width="200" height="200">
    </td>
  </tr>
</table>

<div align="center">
  <h2 align="left"> Circuit Diagram of Our project </h2> 
  <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/Circuit.png" width="500" height="300">
</div>


<div>
  <h2 align="left"> Code for Our project </h2> 
  
  ```c++
  #define BLYNK_TEMPLATE_ID "TMPL6qkgDyA5i"
#define BLYNK_TEMPLATE_NAME "Nature of Plant"
#define BLYNK_AUTH_TOKEN "ctpOlzHZP-4hDF_RLyLbRNJtUvvZI3_i"


#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <DHT.h>
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>

// WiFi credentials
char ssid[] = "IOT LAB";
char pass[] = "iotlab@507";

// DHT & LCD Setup
// Humidity D4
#define DHTPIN D4   
// Temperature DHT11
#define DHTTYPE DHT11  
// Making object "dht"
DHT dht(DHTPIN, DHTTYPE);

// I2C LCD display address 0x27 and size of display 16x2 define by column line
LiquidCrystal_I2C lcd(0x27, 16, 2); 

// Pin Setup
// Relay for pump(on/off) D0
const int relayPin = D0;
// soil sensor pin A0
const int soilSensorPin = A0;

// Moisture calibration
const int dryValue = 1023;// means soil situation is dry 
const int wetValue = 300; // means soil situation is wet
const int threshold = 40; // for auto run

bool pumpRunning = false;
bool manualControl = false;
unsigned long pumpStartTime = 0;
const long pumpMaxDuration = 30000; // 30 sec max

BlynkTimer timer;  // For work after certain time

// Blynk button to control pump (V3)
// when signal give from virtual pin v3 then call this function for pump (on-off)
BLYNK_WRITE(V3) {
  int pinValue = param.asInt();
  manualControl = pinValue == 1;

  if (manualControl) {
    digitalWrite(relayPin, LOW);
    pumpRunning = true;
    pumpStartTime = millis();
    Serial.println("Pump ON via App");
  } else {
    digitalWrite(relayPin, HIGH);
    pumpRunning = false;
    Serial.println("Pump OFF via App");
  }
}
/* 
সেন্সরের ডেটা (মাটি, তাপমাত্রা, আর্দ্রতা) পাঠানো হয় 
Blynk-এ এবং LCD তেও দেখানো হয়। পাম্প চালু থাকলে, ৩০ 
সেকেন্ড পর অটো বন্ধ করে।
*/
void sendSensorData() {
  int soilValue = analogRead(soilSensorPin); //analog value(0-1023)
  int moisturePercent = map(soilValue, dryValue, wetValue, 0, 100);
  moisturePercent = constrain(moisturePercent, 0, 100);

  float t = dht.readTemperature();
  float h = dht.readHumidity();

  /* Blynk virtual pins
  V0 = মাটি আর্দ্রতা (%)
  V1 = তাপমাত্রা
  V2 = বাতাসের আর্দ্রতা
  V3 = পাম্প চালু (1) না বন্ধ (0)
  */
  Blynk.virtualWrite(V0, moisturePercent);
  Blynk.virtualWrite(V1, t);
  Blynk.virtualWrite(V2, h);
  Blynk.virtualWrite(V3, pumpRunning ? 1 : 0);

  // LCD Display
  lcd.setCursor(0, 0);
  lcd.print("Moist:");
  lcd.print(moisturePercent);
  lcd.print("% ");
  lcd.print(pumpRunning ? "P:ON " : "P:OFF");

  lcd.setCursor(0, 1);
  if (isnan(t) || isnan(h)) {
    lcd.print("Temp/Humid Error");
  } else {
    lcd.print("T:");
    lcd.print(t, 1);
    lcd.print("C H:");
    lcd.print(h, 0);
    lcd.print("% ");
  }

  // Auto OFF after 30 seconds
  if (pumpRunning && (millis() - pumpStartTime >= pumpMaxDuration)) {
    digitalWrite(relayPin, HIGH);
    pumpRunning = false;
    Blynk.virtualWrite(V3, 0);
    Serial.println("Pump Auto OFF");
  }
}

void setup() {
  Serial.begin(115200);  //সিরিয়াল মনিটর চালু করে 115200 baud rate-এ। ডিবাগ মেসেজ দেখার জন্য।
  /*  
  Blynk অ্যাপের সাথে সংযোগ স্থাপন করে:
  BLYNK_AUTH_TOKEN এর মাধ্যমে Blynk অ্যাকাউন্ট চিনে,
  ssid ও pass এর মাধ্যমে WiFi-তে কানেক্ট হয়।
  */
  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  dht.begin();
  pinMode(relayPin, OUTPUT);
  digitalWrite(relayPin, HIGH); // Pump OFF

  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Plant Monitoring Sytem");
  lcd.setCursor(0, 1);
  lcd.print("IOT project...");
  delay(2000);
  lcd.clear();

  timer.setInterval(1000L, sendSensorData);
}

/*
মেইন লুপ — Blynk ও টাইমার দুইটাই চালু রাখা হয় যাতে ডেটা পাঠানো এবং ইনপুট নেওয়া যায়।
*/
void loop() {
  Blynk.run();
  timer.run();
}
```
</div>


<div>
  <h2 align="left"> Final Look of out project </h2> 
  <table> 
  <tr>
    <td align="center">
      <strong>Garden Setup</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/WhatsApp%20Image%202025-05-21%20at%201.27.54%20PM.jpeg" width="300" height="250">
    </td>
    <td align="center">
      <strong>Circuit Setup</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/WhatsApp%20Image%202025-05-21%20at%201.27.38%20PM.jpeg" width="300" height="250">
    </td>
    <td align="center">
      <strong>Full Setup</strong><br>
      <img src="https://github.com/Rabbi-hasan0/Plant-Monitoring-System/blob/main/Image/Full-setup.jpeg" width="400" height="250">
    </td>
  </tr>
 </table>
</div>
