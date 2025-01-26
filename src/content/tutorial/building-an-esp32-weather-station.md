---
title: "Building an ESP32 Weather Station"
meta_title: ""
description: "Building an ESP32 Weather Station: Fetch Online and Room Temperature"
date: 2024-12-11T05:00:00Z
image: "/images/image-placeholder.png"
categories: ["Tutorial"]
author: "Solomon Ankomah"
tags: ["ESP32"]
draft: false
---

The ESP32 is a versatile microcontroller that's perfect for IoT applications. In this tutorial, we'll use it to build a weather station that displays the current weather conditions for a specific city using data from an online weather API. Additionally, we'll incorporate a DHT11 temperature sensor to measure the room temperature.

### Features
1. Display real-time weather information for a chosen city.
2. Measure and display the room temperature using a DHT11 sensor.
3. Use an OLED screen to show the data.

---

### What You Need
#### Hardware
- ESP32 microcontroller
- DHT11 temperature and humidity sensor
- SSD1306 OLED display (I2C)
- Jumper wires and breadboard

#### Software
- Arduino IDE (with the ESP32 board library installed)
- Adafruit_GFX and Adafruit_SSD1306 libraries
- ArduinoJson library

---

### Circuit Setup

1. **Connecting the OLED Display**
   - **VCC** to 3.3V on the ESP32
   - **GND** to GND
   - **SCL** to GPIO 22 (default I2C clock)
   - **SDA** to GPIO 21 (default I2C data)

2. **Connecting the DHT11 Sensor**
   - **VCC** to 3.3V
   - **GND** to GND
   - **DATA** to GPIO 4

---

### Code Explanation
Here's the full Arduino code to run the weather station. Copy and paste it into the Arduino IDE.

```cpp
#include <Wire.h>
#include <WiFi.h>
#include <HTTPClient.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <ArduinoJson.h>
#include "DHT.h"

// Wi-Fi credentials
const char* ssid = "wifi name";
const char* password = "wifi password";

// OLED setup (I2C)
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// Weather API URL (use HTTPS)
const char* weather_url = "https://wttr.in/Dresden?format=j1";

// DHT11 setup
#define DHTPIN 4        // Pin connected to the DHT11 data pin
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);

  // Initialize OLED
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("OLED initialization failed!");
    while (true);
  }
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);

  // Initialize DHT sensor
  dht.begin();

  // Connect to Wi-Fi
  WiFi.begin(ssid, password);
  display.println("Connecting to Wi-Fi...");
  display.display();
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to Wi-Fi...");
  }
  display.clearDisplay();
  display.println("Wi-Fi Connected!");
  display.display();
  delay(2000);
}

void loop() {
  display.clearDisplay();
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    http.begin(weather_url); // Use HTTPS URL
    int httpCode = http.GET();

    Serial.print("HTTP Response Code: ");
    Serial.println(httpCode);

    if (httpCode == 200) { // HTTP OK
      String payload = http.getString();
      Serial.println("HTTP Response:");
      Serial.println(payload);

      // Parse JSON
      DynamicJsonDocument doc(4096); // Increase buffer size for large payloads
      DeserializationError error = deserializeJson(doc, payload);

      if (error) {
        Serial.print("JSON Parsing failed: ");
        Serial.println(error.c_str());
        display.clearDisplay();
        display.setCursor(0, 0);
        display.println("Error parsing");
        display.println("weather data!");
        display.display();
      } else {
        // Extract weather data
        String weather = doc["current_condition"][0]["weatherDesc"][0]["value"].as<String>();
        String temp = doc["current_condition"][0]["temp_C"].as<String>();

        // Truncate long weather descriptions
        if (weather.length() > 20) {
          weather = weather.substring(0, 20) + "...";
        }

        // Read room temperature
        float roomTemp = dht.readTemperature();
        if (isnan(roomTemp)) {
          Serial.println("Failed to read from DHT sensor!");
          roomTemp = -999; // Placeholder for failed reading
        }

        // Display weather and room data on OLED
        display.clearDisplay();
        display.setCursor(0, 0);
        display.println("Dresden Weather:");
        display.print("Temp: ");
        display.print(temp);
        display.println(" C");
        display.print("Condition: ");
        display.println(weather);
        display.println();
        display.println("Room:");
        display.print("Temp: ");
        if (roomTemp != -999) {
          display.print(roomTemp);
          display.println(" C");
        } else {
          display.println("Error");
        }
        display.display();
      }
    } else {
      // Print error
      Serial.print("HTTP Error: ");
      Serial.println(http.errorToString(httpCode).c_str());

      // Display error on OLED
      display.clearDisplay();
      display.setCursor(0, 0);
      display.println("Failed to fetch");
      display.println("weather data.");
      display.println("Error:");
      display.println(http.errorToString(httpCode).c_str());
      display.display();
    }
    http.end();
  } else {
    Serial.println("Wi-Fi disconnected!");
    display.clearDisplay();
    display.setCursor(0, 0);
    display.println("Wi-Fi Disconnected!");
    display.display();
  }
  delay(60000); // Update every 60 seconds
}
```

---

### Explaining the Code

#### Connecting to Wi-Fi
The ESP32 connects to your Wi-Fi network using the `WiFi.begin()` function. Replace the `ssid` and `password` variables with your network credentials.

#### Fetching Weather Data
We use the weather API from [wttr.in](https://wttr.in/), which provides JSON-formatted weather data. Replace the city name in the `weather_url` (e.g., `Dresden`) with your desired city. The API response includes temperature, weather description, and more.

#### Parsing JSON Data
The `ArduinoJson` library extracts relevant data like temperature and weather conditions. Ensure you allocate enough memory to handle the API's JSON response.

#### Reading Room Temperature
The `DHT11` sensor measures room temperature, and its reading is displayed alongside the weather data.

#### Displaying Data
The `Adafruit_SSD1306` library updates the OLED screen with weather and room temperature details.

---

### Customizing for Other Cities
To fetch weather for a different city, change the `weather_url` to include your desired city name:
```cpp
const char* weather_url = "https://wttr.in/London?format=j1";
```
You can also customize the API format and include additional data by exploring the [wttr.in documentation](https://wttr.in/:help).

---

### Conclusion
This project demonstrates how to integrate an ESP32, an online weather API, and a DHT11 sensor to create a functional weather station. You can expand this project by adding humidity readings, a clock, or even storing data for analysis. Happy tinkering!


