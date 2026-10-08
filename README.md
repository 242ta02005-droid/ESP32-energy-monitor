# ESP32-energy-monitor
#include <WiFi.h>
#include <WebServer.h>

const char* ssid = "YOUR_WIFI_NAME";
const char* password = "YOUR_WIFI_PASSWORD";

WebServer server(80);

// Example values — replace these with actual sensor readings
float voltage = 230.0;
float current = 1.25;

unsigned long lastTime = 0;
float energy_kWh = 0.0;

void handleData() {
  // Calculate power
  float power = voltage * current;

  // Calculate energy
  unsigned long now = millis();
  float hours = (now - lastTime) / 3600000.0;
  energy_kWh += (power * hours) / 1000.0;
  lastTime = now;

  String json = "{";
  json += "\"voltage\":" + String(voltage, 2) + ",";
  json += "\"current\":" + String(current, 2) + ",";
  json += "\"power\":" + String(power, 2) + ",";
  json += "\"energy\":" + String(energy_kWh, 4);
  json += "}";

  server.send(200, "application/json", json);
}

void setup() {
  Serial.begin(115200);

  WiFi.begin(ssid, password);

  Serial.print("Connecting to WiFi");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.print("ESP32 IP address: ");
  Serial.println(WiFi.localIP());

  lastTime = millis();

  server.on("/data", handleData);
  server.begin();
}

void loop() {
  server.handleClient();
}
Python program
Install the required library:

pip install requests
Then run:

import requests
import time
from datetime import datetime

ESP32_IP = "192.168.1.100"   # Change to your ESP32 IP

while True:
    try:
        url = f"http://{ESP32_IP}/data"

        response = requests.get(url, timeout=5)
        data = response.json()

        voltage = data["voltage"]
        current = data["current"]
        power = data["power"]
        energy = data["energy"]

        print("\n-----------------------------")
        print("       ESP32 ENERGY MONITOR")
        print("-----------------------------")
        print("Time    :", datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
        print(f"Voltage : {voltage:.2f} V")
        print(f"Current : {current:.2f} A")
        print(f"Power   : {power:.2f} W")
        print(f"Energy  : {energy:.4f} kWh")
        print("-----------------------------")

        time.sleep(2)

    except requests.exceptions.RequestException as e:
        print("Connection error:", e)
        time.sleep(2)
