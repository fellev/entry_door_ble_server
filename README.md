# ESP32-C3 BLE Door Key System

This repository contains the firmware for an **ESP32-C3** device that performs secure BLE-based proximity authentication and triggers a door-open event via a Tasmota-controlled relay. The project pairs with a companion Android application.

---

## 🚀 Project Overview

The system turns your phone into a secure BLE key:

* When your Android phone gets close to the ESP32-C3, the phone authenticates using an **HMAC-based challenge/response**.
* If authentication succeeds, the ESP32-C3 turns **LED ON** to indicate presence.
* When you press a physical button connected to the ESP32-C3, it signals a secondary ESP32 running **Tasmota**, which triggers a **Home Assistant** door unlock.
* If your phone moves away, the ESP32-C3 detects increasing distance via **RSSI** and turns **LED OFF**.

All proximity and access logic is handled by the ESP32-C3, not the Android app.

---

## 🔧 Hardware Overview

### **Required Components**

* **ESP32-C3 module** running ESP-IDF firmware
* **Second ESP32** running **Tasmota** firmware (acts as a secure MQTT trigger)
* LED + resistor
* Push button
* Wire connection from ESP32-C3 → Tasmota ESP (GPIO pulse)

### **GPIO Mapping (configurable in code)**

* `LED_GPIO` – LED output indicator
* `BUTTON_GPIO` – physical button for door trigger
* `DOOR_GPIO` – output pulse to Tasmota ESP32

---

## 🔐 Authentication Flow (HMAC-based)

1. Android app detects the ESP32-C3 via BLE advertisements.
2. Phone connects to BLE GATT server.
3. ESP32-C3 generates a **random challenge (32 bytes)**.
4. Phone computes:

```
HMAC = HMAC_SHA256(shared_key, challenge || phone_id || timestamp)
```

5. ESP32-C3 validates the HMAC.
6. If valid → LED turns **ON**, authentication state becomes active.
7. ESP32 continuously monitors RSSI to ensure the phone remains close.

---

## 📡 BLE Services & Characteristics

The ESP32-C3 exposes a custom BLE service:

### **Service UUID:**

```
6E400001-B5A3-F393-E0A9-E50E24DCCA9E
```

### **Characteristics**

| Characteristic | UUID                                   | Operation | Description                    |
| -------------- | -------------------------------------- | --------- | ------------------------------ |
| Challenge      | `6E400002-B5A3-F393-E0A9-E50E24DCCA9E` | Read      | Client reads 32-byte challenge |
| Auth           | `6E400003-B5A3-F393-E0A9-E50E24DCCA9E` | Write     | Client writes HMAC auth data   |

---

## 🧠 ESP32-C3 Internal Logic

### Authentication State Machine

* **IDLE** – no valid phone
* **AUTHENTICATED** – LED ON, RSSI monitoring active
* **RSSI_LOST** – LED OFF, session closed

### RSSI Monitoring

Runs every 500ms:

* If RSSI < threshold (default `-70 dBm`) for ~3 seconds → LED OFF, deauthenticate.

### Door Trigger

* Physical button press
* Only active when authenticated
* Triggers **DOOR_GPIO** pulse to Tasmota

---

## 📁 Repository Structure

```
.
├── main/
│   ├── main.c              # Main firmware logic (BLE, HMAC, RSSI, LED, button)
│   └── ...                 # Additional ESP-IDF source files
├── CMakeLists.txt
├── sdkconfig
└── README.md               # This file
```

---

## 📲 Android App (Companion)

The Android phone must:

* scan for the ESP32-C3 BLE beacon
* connect and read challenge
* compute HMAC
* write authentication data

The app does **not** control LED or door actions.
Firmware and app are fully decoupled except for cryptography.

---

## 🏠 Tasmota + Home Assistant Integration

The ESP32-C3 does **not** speak MQTT.
Instead, it pulses a GPIO line to a second ESP32 with **Tasmota**, which sends:

```
MQTT: homeassistant/door/trigger
```

Home Assistant automation handles the actual door lock.

---

## 🔧 Build Instructions

### Requirements

* ESP-IDF v5.x
* CMake
* Python 3

### Build & Flash

```
idf.py build
idf.py flash
idf.py monitor
```

---

## 🔒 Security Notes

* Replace the `SHARED_KEY` placeholder with a securely stored random 32-byte key.
* For production use, consider secure provisioning (QR code onboarding).
* BLE pairing/bonding is optional because HMAC already authenticates sessions.

---

## 📜 License

MIT License

---

## 🤝 Contributions

PRs, suggestions, and improvements are welcome.

---

## 📧 Contact

For questions or integration help, feel free to open an issue.
