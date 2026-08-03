<div align="center">
  <img src="https://img.icons8.com/color/96/000000/security-checked.png" alt="Logo">
  <h1>ESP32 RFID Anti-Wall-Jumping System (Student Version)</h1>
  <p><b>Advanced multi-sensor security solution for perimeter protection</b></p>

  [![PlatformIO](https://img.shields.io/badge/PlatformIO-orange?style=for-the-badge&logo=platformio&logoColor=white)](https://platformio.org/)
  [![ESP32](https://img.shields.io/badge/ESP32-blue?style=for-the-badge&logo=espressif&logoColor=white)](https://www.espressif.com/)
  [![C++](https://img.shields.io/badge/C++-00599C?style=for-the-badge&logo=c%2B%2B&logoColor=white)](https://isocpp.org/)
  [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)
</div>

---

## 📌 Overview

The **ESP32 RFID Anti-Wall-Jumping System** is a robust, real-time perimeter security prototype designed to detect unauthorized access (like wall climbing or jumping) and respond instantly. It leverages an Active IR Beam for detection, a long-range UHF RFID reader for authorized access, a DFPlayer Mini for audio alarms, and a SIM800L module for instant SMS notifications to security personnel.

This **Student Version** of the repository contains the core logic (`system_firmware.cpp`) fully commented to help you understand the concurrent hardware integrations, FreeRTOS tasks, and AT-command parsing.

---

## ✨ Key Features

- **🛡️ Perimeter Detection**: Uses a Dual Beam Active IR Sensor for extreme weather-resistant and long-range intrusion detection.
- **📱 Instant SMS Alerts**: Uses SIM800L to send immediate notifications to a registered emergency contact upon intrusion.
- **🔊 Audio Alarms**: Integrates DFPlayer Mini to broadcast custom localized voice alarms or sirens.
- **🪪 Long Range RFID (UHF WG26)**: Allows authorized personnel with UHF tags to bypass the system from up to 20 meters away.
- **💾 Local SD Card Logging**: All events and access attempts are saved to an onboard MicroSD card in FAT32 format.
- **🕒 Real-Time Clock**: Built-in DS3231 RTC for accurate time-stamping of intrusion logs.

---

## 🏗️ System Architecture

![System Architecture](./include/images/SystemArch.png)

---

## 🛠️ Hardware Stack

| Component | Hardware Model / Specification | Communication Interface | Operating Power | Primary Function |
| :--- | :--- | :--- | :--- | :--- |
| **Microcontroller** | ESP32-WROOM-32 (38-Pin Dev Board) | Dual-Core FreeRTOS / Wi-Fi / BT | 5V DC (VIN) | Central processing & state machine |
| **UHF RFID Reader** | RD906M Directional Panel Antenna | Wiegand 26 / 34 Protocol | 12V DC | Long-range RFID scanning (860–960 MHz) |
| **Perimeter Sensor** | Active Single-Beam IR Detector | Photoelectric NC Relay Loop | 12V DC | Infrared perimeter breach detection (Tripwire) |
| **Cellular Modem** | SIM800L EVB GSM / GPRS Module | Hardware UART2 (AT Commands) | 5V DC (2A Peak) | Dispatching instant SMS alerts to security |
| **Audio Synthesizer**| DFPlayer Mini MP3 Synthesizer | Hardware UART1 (Serial MP3) | 5V DC | Local siren and multi-track voice prompts |
| **Real-Time Clock** | DS3231 Precision RTC Module | I2C Bus (`0x68`) | 3.3V DC (CR2032) | Hardware timestamping for incident logging |
| **Data Storage** | MicroSD Card SPI Module | Hardware VSPI Bus | 5V DC | FAT32 logging (`eventlog.csv`, `students.csv`) |

### Realistic Component Layout

![Realistic Hardware Layout](./include/images/realisticComponents.png)

> [!NOTE]
> **Visual Reference Only:** The realistic component layout above is for visual purposes and aesthetic representation. For exact pin mapping and electrical connections, strictly follow the official 2D Schematic Circuit Diagram below.

---

## 🔌 Schematic Circuit Diagram

For full hardware wiring instructions and component mapping, please refer to the official wiring schematic:

![Schematic Circuit Diagram](./include/images/schematic.png)

📄 **[Download Official 2D Schematic Circuit Diagram (PDF)](./include/SchematicDiagram.pdf)**

---

## 🔌 Master Pinout Allocation (ESP32-38P)

All peripherals are assigned to specific GPIO pins to eliminate bus collisions (e.g., isolating Wiegand onto **GPIO 21/22** away from the MicroSD SPI bus on **GPIO 18/19**).

### Left Pin Rail (Sensors & System Inputs)

| GPIO Pin | Peripheral Signal | Connected Module | Voltage / Type | Notes & Logic |
| :--- | :--- | :--- | :--- | :--- |
| `GPIO 34` | **MODE BUTTON** | Tactile Pushbutton | 3.3V Input | Active LOW (Requires 10 kΩ pull-up to 3V3; Hold 3s to enter Config Mode) |
| `GPIO 35` | *(Unassigned)* | Reserved Input | — | High-impedance input available |
| `GPIO 32` | *(Unassigned)* | Reserved GPIO | — | Available for expansion |
| `GPIO 33` | *(Unassigned)* | Reserved GPIO | — | Available for expansion |
| `GPIO 25` | *(Unassigned)* | Reserved GPIO | — | Available for expansion |
| `GPIO 26` | **RTC SCL** | DS3231 RTC (`SCL`) | 3.3V I2C | Hardware I2C Clock (4.7 kΩ pull-up to 3V3) |
| `GPIO 27` | **RTC SDA** | DS3231 RTC (`SDA`) | 3.3V I2C | Hardware I2C Data (4.7 kΩ pull-up to 3V3) |
| `GPIO 14` | **IR BEAM SENSOR**| Active IR Receiver (`NC`) | 3.3V Input | Internal `INPUT_PULLUP` enabled (Fail-safe: HIGH = Beam Broken / Cut) |
| `GPIO 12` | *(Unassigned)* | Reserved GPIO | — | Boot pin MTDI (idles LOW) |
| `GND` | **MASTER GND** | Common Ground Bus | 0.0V Ground | **Tied to ALL component grounds** |
| `GPIO 13` | **STATUS LED** | Blue Indicator LED | 3.3V Output | 220 Ω resistor to GND |

### Right Pin Rail (Communications, SPI & Audio)

| GPIO Pin | Peripheral Signal | Connected Module | Voltage / Type | Notes & Logic |
| :--- | :--- | :--- | :--- | :--- |
| `GPIO 23` | **SPI MOSI** | MicroSD Module (`MOSI`) | 3.3V SPI | Dedicated Hardware VSPI Bus Master-Out |
| `GPIO 22` | **WIEGAND D1** | RD906M Reader (`D1`) | 3.3V/5V Input | Hardware Wiegand Data 1 Interrupt (No SPI conflict) |
| `GPIO 21` | **WIEGAND D0** | RD906M Reader (`D0`) | 3.3V/5V Input | Hardware Wiegand Data 0 Interrupt (No SPI conflict) |
| `GPIO 19` | **SPI MISO** | MicroSD Module (`MISO`) | 3.3V SPI | Dedicated Hardware VSPI Bus Master-In |
| `GPIO 18` | **SPI SCK** | MicroSD Module (`SCK`) | 3.3V SPI | Dedicated Hardware VSPI Bus Clock |
| `GPIO 5` | **SD CARD CS** | MicroSD Module (`CS`) | 3.3V Output | Hardware VSPI Chip Select |
| `GPIO 17` | **GSM TXD** | SIM800L EVB (`RXD`) | 3.3V UART2 | ESP32 `UART2 TX` → SIM800L `RX` |
| `GPIO 16` | **GSM RXD** | SIM800L EVB (`TXD`) | 3.3V UART2 | ESP32 `UART2 RX` ← SIM800L `TX` |
| `GPIO 4` | **ALARM LED** | Red Siren LED | 3.3V Output | 220 Ω resistor to GND (Active HIGH on Alarm trigger) |
| `GPIO 0` | *(Unused)* | System Boot Clock | — | Boot strapping pin (Left floating) |
| `GPIO 2` | **DFPLAYER TXD**| DFPlayer Mini (`RX`) | 3.3V UART1 | ESP32 `UART1 TX` → 1 kΩ series resistor → DFPlayer `RX` |
| `GPIO 15` | **DFPLAYER RXD**| DFPlayer Mini (`TX`) | 3.3V UART1 | ESP32 `UART1 RX` ← DFPlayer `TX` |

---

## 📖 Documentation & Guides

Please refer to the comprehensive manual for detailed instructions on setting up and using the system:

### 👤 [Download the Complete System Manual (PDF)](./System_Manual.pdf)
Practical instructions for setup and usage including:
- Complete Pin Mapping Reference
- Operating Modes (Normal vs Config Mode)
- RFID Registration & Incident Tracking
- Setup for the DFPlayer SD Card (Audio Tracks)

---

## 🚀 How to Study This Code

The provided `system_firmware.cpp` file contains the complete top-level orchestration of the hardware peripherals. You can open `system_firmware.pdf` for a formatted reading experience or open the `.cpp` file in any text editor. Look at how FreeRTOS separates the sensor polling (`sensorTask`) from the SMS dispatching (`commTask`) so that the ESP32 never misses a tripped alarm while waiting for an SMS to send!

<div align="center">
  <i>Developed using C++ and PlatformIO.</i>
</div>
