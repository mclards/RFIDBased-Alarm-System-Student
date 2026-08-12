# ESP32 RFID Anti-Wall-Jumping System
**Advanced multi-sensor security solution for perimeter protection**

---

## 📌 Overview

The **ESP32 RFID Anti-Wall-Jumping System** is a robust, real-time perimeter security prototype designed to detect unauthorized boundary breaches (such as wall climbing or jumping) and respond instantly. It leverages an Active IR Beam sensor for reliable perimeter tripwire detection, a long-range RD906M UHF RFID reader (Wiegand protocol) for student tag identification, a DFPlayer Mini for localized audio sirens and voice prompts, and a SIM800L module for sending instant SMS alerts to security personnel.

The system features a **built-in Captive Portal (Web UI)** accessible via Wi-Fi for easy on-the-fly configuration of phone numbers, student registries, voice tracks, and log downloads without needing to re-flash the firmware.

---

## ✨ Key Features

- 🛡️ **Active Perimeter Detection:** Uses an Active Single-Beam IR Detector for extreme weather-resistant and long-range intrusion detection.
- 📱 **Instant SMS Alerts:** Integrates SIM800L module to dispatch immediate SMS notifications (including student names and IDs) to the guard's phone.
- 🔊 **Audio Alarms & Voice Prompts:** Uses DFPlayer Mini to broadcast custom localized voice warnings or siren tracks.
- 🪪 **Long Range UHF RFID (WG26/34):** Detects RD906M UHF RFID tags from a distance to track individuals near the perimeter wall.
- 💾 **Dual-Storage Logging (SD Card & EEPROM):** Saves master event logs (`/eventlog.csv`), individual student incident files (`/logs/<tagID>.csv`), and student registries (`/students.csv`) in FAT32 format on a MicroSD card.
- 🌐 **Web-Based Captive Portal:** Built-in Async Web Server allowing administrators to configure system thresholds, manage student cards, sync RTC time, and view logs over Wi-Fi.
- 🕒 **Precision Real-Time Clock:** Built-in DS3231 RTC module for accurate timestamping of all security incidents.
- 🔒 **Hardened FreeRTOS Firmware:** Multi-threaded task separation (Core 0 for Sensors, Core 1 for SMS/Web) protected with FreeRTOS mutexes to prevent memory conflicts.

<div class="page-break"></div>

## 🏗️ System Architecture

![System Architecture](./include/images/SystemArch.png)

---

## 🛠️ Hardware Stack

| Component | Specification / Model | Interface | Power | Primary Function |
| :--- | :--- | :--- | :--- | :--- |
| **Microcontroller** | ESP32-WROOM-32 (38-Pin) | Dual-Core FreeRTOS | 5V DC (VIN) | Core processing & state machine |
| **UHF RFID Reader** | RD906M Directional Panel | Wiegand 26 / 34 | 12V DC | Long-range RFID tag scanning |
| **Perimeter Sensor** | Active Single-Beam IR Detector | Photoelectric Relay (`NC`) | 12V DC | Boundary breach tripwire detection |
| **Cellular Modem** | SIM800L EVB GSM/GPRS | Hardware UART2 | 5V DC (2A Peak) | Sending SMS alerts to security |
| **Audio Synthesizer**| DFPlayer Mini MP3 Player | Hardware UART1 | 5V DC | Local siren and voice track playback |
| **Real-Time Clock** | DS3231 Precision RTC | I2C Bus (`0x68`) | 3.3V DC | Hardware timestamping for incident logs |
| **Data Storage** | MicroSD SPI Card Reader | Hardware VSPI Bus | 5V DC | FAT32 log & student database storage |

<div class="page-break"></div>

## 🖼️ Realistic Component Layout

> **[!NOTE] Visual Reference Only**
> The realistic component layout below is for visual representation. For exact pin mapping and electrical connections, strictly follow the official 2D Schematic Circuit Diagram.

![Realistic Hardware Layout](./include/images/realisticComponents.png)

<div class="page-break"></div>

## 🔌 Schematic Circuit Diagram

For full hardware wiring instructions and component mapping, please refer to the official wiring schematic:

![Schematic Circuit Diagram](./include/images/schematic.png)

---

## 📍 Optimized Pin Assignment (ESP32-38P)

All peripheral pairs are assigned to physically adjacent GPIO pins on the ESP32-38P to prevent bus collisions and minimize wire crossings on a prototype board.

```text
LEFT SIDE  (top → bottom)                 RIGHT SIDE
──────────────────────────────────────────────────────────────────
GPIO34  ← MODE BUTTON                     GPIO23  ← SPI MOSI ─┐
GPIO35  ← (Unassigned)                    GPIO22  ← WIEGAND D1│ RD906M
GPIO32  ← (Unassigned)                    GPIO21  ← WIEGAND D0│ Reader
GPIO33  ← (Unassigned)                    GPIO19  ← SPI MISO ─┐
GPIO25  ← (Unassigned)                    GPIO18  ← SPI SCK   │ SD Card
GPIO26  ← RTC SCL ┐                       GPIO5   ← SD CS    ─┘
GPIO27  ← RTC SDA ┘ I2C                   GPIO17  ← SIM800 TX ─┐
GPIO14  ← IR BEAM ┐                       GPIO16  ← SIM800 RX ─┘ UART2
GPIO12  ← (Unassigned)                    GPIO4   ← Alarm LED (Red)
GND     ← Master Ground                   GPIO0   ← (Unused, boot clock)
GPIO13  ← Status LED (Blue)               GPIO2   ← DFPlayer TX ─┐ UART1
                                          GPIO15  ← DFPlayer RX ─┘
```

<div class="page-break"></div>

## 🎮 2. System Operations & State Machine

The system operates under two distinct system modes controlled via the physical Mode Button (GPIO 34) or remote SMS commands.

### 2.1 Normal Mode (Guarding)
- **Blue Status LED:** Solid **ON**.
- **System Operation:** Core 0 actively monitors the IR Beam sensor and the Wiegand RFID reader.
- **Alarm Sequence:** When the IR beam is broken while an RFID tag is detected:
  1. The DFPlayer Mini immediately plays the configured voice siren track.
  2. The Red Alarm LED (GPIO 4) illuminates.
  3. The system logs the incident to the MicroSD card (`/eventlog.csv` and `/logs/<tagID>.csv`).
  4. Core 1 formats and queues an SMS alert containing the student's Name, Student ID, and Location, then dispatches it via the SIM800L modem to the registered Guard phone number.

### 2.2 Setup Mode (Configuration)
- **Blue Status LED:** **Blinking Fast** (every 400ms).
- **How to Enter:** Press and hold the Mode Button for 3 seconds, or send SMS text command `AP ON`.
- **System Operation:** Wall breach monitoring is temporarily paused to prevent false alarms. The ESP32 starts its Wi-Fi Access Point (`AntiWallJump-Setup`) and launches the Web Captive Portal.
- **How to Exit:** Hold the Mode Button for 3 seconds, send `AP OFF`, click "Stop Portal" on the web dashboard, or wait 1 hour for automatic idle timeout.

### 2.3 Remote Control via SMS Commands
Security personnel can control the system remotely by sending SMS messages to the SIM800L SIM card:
- `AP ON` — Activates Wi-Fi Setup Mode and turns on the configuration web portal.
- `AP OFF` — Deactivates Setup Mode and returns the system to Normal Guarding Mode.

*(Note: If **Number Lock** is enabled in the configuration, the system strictly validates the sender's phone number and will only execute commands sent from the registered `guardPhone` number.)*

<div class="page-break"></div>

## ⚙️ 3. Web Dashboard & Configuration

### 3.1 Connecting to the Setup Page
1. Switch the system to **Setup Mode** (hold Mode Button until Blue LED blinks fast).
2. Connect your smartphone or laptop to the Wi-Fi network:
   - **SSID:** `AntiWallJump-Setup`
   - **Password:** `configure123`
3. The Captive Portal page will automatically open. If it does not auto-open, navigate your web browser to `http://192.168.4.1`.
4. Enter the default Admin Password (`admin`) to log in.

### 3.2 Dashboard Overview & Features
- **Top Quick Action Bar:**
  - 🕒 **Sync Clock:** Instantly synchronizes the DS3231 RTC chip with your phone/computer system time.
  - 🌓 **Theme Switcher:** Toggles the interface between Dark Mode and Light Mode.
  - 🚪 **Logout:** Ends the current admin session.
  - ⚡ **Stop Portal:** Immediately exits Setup Mode and re-arms the perimeter security system.
- **Navigation Tabs:**
  - 📊 **Dashboard:** Displays node health, memory card status, SIM signal status, and live sensor readings.
  - 🪪 **Registry:** Manage authorized student RFID cards (Add, Scan, Delete).
  - ⚙️ **Settings:** Configure Guard Phone Number, SMS Recipient Role, Wall Label, Alarm Voice Track, Alarm Duration, Cooldown, and Admin Passwords.
  - 📁 **Records:** View live intrusion logs and download complete CSV security reports.

### 3.3 Optimal Configuration Defaults
For large physical perimeters, the following time-based settings are recommended for best performance:
- **RFID Memory / Tag Retention:** `30000 ms` (30s) — Time window a student has to jump the wall after scanning their ID.
- **Bucketing Window:** `10000 ms` (10s) — Wait time after an alarm starts to gather any other students jumping into the same SMS alert.
- **Breach Memory / Sensor Hold:** `5000 ms` (5s) — How long the system remembers a motion sensor trigger before clearing it.
- **Cooldown Time:** `60000 ms` (60s) — Wait time after an alarm before the system can trigger again (prevents SMS spam).
- **Alarm Duration:** `30000 ms` (30s) — How long the alarm siren will sound when someone jumps the wall.

<div class="page-break"></div>

## 🪪 4. Student ID Cards & Registry Management

The system keeps track of authorized students using their UHF RFID tag IDs linked to their student records.

### 4.1 Registering a New Student Card
1. Navigate to the **Registry** tab on the web dashboard.
2. Click **Scan Card** to place the web portal into tag scanning mode.
3. Tap or hold the student's UHF RFID card near the RD906M panel. The reader will capture the unique Wiegand tag ID and auto-fill the **RFID Tag ID** field on your screen.
4. Enter the student's **Full Name** and **Student ID Number**.
5. Click **Register Student** to save the record to internal EEPROM and the MicroSD card (`/students.csv`).

### 4.2 Intrusion Detection Logic
- **Registered Student:** If a registered student's tag is present when the perimeter beam is breached, the alarm triggers and dispatches an SMS specifically naming the student and their ID number to ensure accountability.
- **Unregistered Tag / Unknown Intruder:** If an unregistered card or no tag is detected during a breach, the system logs the incident as `"UNREGISTERED"` and alerts security to an unknown intruder.

---

## 📁 5. Alarm Records & Data Logging

Every security event is timestamped by the DS3231 RTC and saved to the onboard MicroSD card across two log layers:
1. **Master Event Log (`/eventlog.csv`):** Stores a chronological list of all perimeter breach events.
2. **Student Incident Log (`/logs/<tagID>.csv`):** Stores individual incident histories dedicated to each specific student card.

### 5.1 Downloading Complete CSV Reports
1. Navigate to the **Records** tab on the web dashboard.
2. Click **"Download Logs"**.
3. The system generates and streams a comprehensive report file called **`Campus_Security_Logs.csv`** to your device.
4. The downloaded file contains both the Master Security Log and individual Student Incident Trackers, which can be directly opened in Microsoft Excel or Google Sheets.
