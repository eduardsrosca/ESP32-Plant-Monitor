# IoT Plant Environment Monitor & Alert System

A robust, fail-safe monitoring system for houseplants that uses an ESP32 to track soil moisture and temperature. The system features a local "offline" alert mode and a web-based "online" dashboard for remote monitoring.

---

## General Description
This system monitors the environmental health of a houseplant by continuously reading soil moisture, temperature, and humidity levels. 

It is designed to be **network-resilient**:
1.  **Local Mode:** It displays live status on an OLED screen and triggers a physical Red LED alarm if moisture drops below a critical threshold.
2.  **Online Mode:** It hosts a lightweight web server (accessible via Smartphone/PC) to view current stats.

[cite_start]If the WiFi connection is lost, the device continues to monitor and alert locally, avoiding the "Invisible Project" antipattern[cite: 423].

## BOM (Bill of Materials)
* [cite_start]**Microcontroller:** ESP32 Development Board (e.g., WROOM-32) [cite: 167]
* **Sensors:**
    * Capacitive Soil Moisture Sensor v1.2
    * DHT22 (Temperature & Humidity Sensor)
* **Actuators/Display:**
    * 0.96" OLED Display (I2C)
    * Red LED + 220Ω Resistor
* **Power:** USB Cable / 5V Power Bank
* **Misc:** Breadboard, Jumper wires

## Tutorial Source
I am using the following tutorial as a starting point for the web server implementation:
* *Reference:* [ESP32 Web Server with BME280/DHT11 - Random Nerd Tutorials](https://randomnerdtutorials.com/)

### Changes from the Tutorial
The original tutorial only displays sensor data on a webpage. I am extending this by:
1.  **Adding State Logic:** Implementing a "Critical" vs "Normal" state machine with hysteresis.
2.  **Local Failsafe:** Integrating an OLED screen and physical LED so the device works without internet.
3.  **Non-blocking Code:** Rewriting the loop to avoid `delay()` so the physical button and network stack remain responsive simultaneously.

---

## Course Requirements (Q&A)

### Q1: What is the system boundary?
The **system** consists of the ESP32, the attached sensors, and the local actuators (LED/OLED). The **outside** world is the user's smartphone. [cite_start]The boundary is the HTTP request; the phone simply requests data, but the "smart" decision to trigger an alarm happens entirely **inside** the ESP32[cite: 341].

### Q2: Where does the intelligence live?
The intelligence lives on the **ESP32**. It allows the device to smooth noisy sensor data and decide if the plant needs water. The phone is just a "dumb" display terminal. [cite_start]This prevents the "Distributed Intelligence" trap[cite: 349].

### Q3: What is the hardest technical problem?
**Concurrency & Timing.** The ESP32 must maintain a WiFi connection and serve web pages (which can be slow) without blocking the sensor reading loop or freezing the physical UI. [cite_start]I will use `millis()` based timers (or FreeRTOS tasks) to ensure the system is responsive[cite: 164, 194].

### Q4: What is the minimum demo?
The **Offline Mode**. If I unplug the WiFi router, the device must still successfully read the soil moisture and light up the Red LED immediately when the sensor is removed from the water. [cite_start]This proves the system works independently of the network[cite: 371].

### Q5: Why is this not just a tutorial?
This project combines two distinct domains (Physical UI via I2C and Network UI via WiFi) that are usually taught separately. [cite_start]Furthermore, I am implementing **Hysteresis logic** for the alarm (e.g., turns ON at <30%, but only turns OFF at >40%) to prevent LED flickering, which requires custom decision logic beyond a simple copy-paste tutorial[cite: 382].

---

## Hardware Requirement
**Do I need an ESP32?**
[cite_start]**Yes.** This project specifically requires WiFi connectivity to allow remote status checks via a smartphone, which is a key feature of the architecture[cite: 318].
