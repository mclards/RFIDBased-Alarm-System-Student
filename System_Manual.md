<style>
  @media print {
    h1, h2, h3, h4, h5, h6 { page-break-after: avoid !important; page-break-inside: avoid !important; }
    table, img, pre { page-break-inside: avoid !important; }
    p, li { orphans: 3; widows: 3; }
    .page-break { page-break-before: always !important; }
  }
</style>

# ESP32 RFID Anti-Wall-Jumping System
**Advanced multi-sensor security solution for perimeter protection**

---

## 📌 Overview

The **ESP32 RFID Anti-Wall-Jumping System** is a robust, real-time perimeter security prototype designed to detect unauthorized access (like wall climbing or jumping) and respond instantly. It leverages an Active IR Beam sensor for reliable perimeter detection, a long-range UHF RFID reader for authorized access, a DFPlayer Mini for audio alarms, and a SIM800L module for instant SMS notifications to security personnel.

The system features a **built-in Captive Portal (Web UI)** for easy configuration on the fly, avoiding the need to re-flash the firmware to change settings.

---

## ✨ Key Features

- 🛡️ **Active Perimeter Detection:** Uses an Active Single-Beam IR Detector for extreme weather-resistant and long-range intrusion detection (replacing old ultrasonic sensors).
- 📱 **Instant SMS Alerts:** Uses SIM800L to send immediate notifications to a registered emergency contact upon intrusion.
- 🔊 **Audio Alarms:** Integrates DFPlayer Mini to broadcast custom localized voice alarms or sirens.
- 🪪 **Long Range RFID (UHF WG26):** Allows authorized personnel (using the RD906M Directional Panel Antenna) to bypass the system from a distance.
- 💾 **Local SD Card Logging:** All events and access attempts are saved to an onboard MicroSD card in FAT32 format.
- 🌐 **Web-Based Captive Portal:** Built-in async web server allowing administrators to configure system thresholds, modes, and sensors over Wi-Fi without compiling code.
- 🕒 **Real-Time Clock:** Built-in DS3231 RTC for accurate time-stamping of intrusion logs.
- 🔒 **Hardened Firmware:** Production-grade concurrency protection with dedicated FreeRTOS mutexes, deferred restart handling, validated RTC input, and guarded SD operations.

<div class="page-break"></div>

## 🏗️ System Architecture

![System Architecture](./include/images/SystemArch.png)

---

## 🛠️ Hardware Stack

| Component | Function |
| :--- | :--- |
| **ESP32-38P** | Core processing and Web Server AP |
| **RD906M UHF Panel** | Long-range RFID Scanning |
| **Active IR Beam** | Perimeter breach / motion detection |
| **SIM800L** | GSM Module for SMS alerts |
| **DFPlayer Mini** | MP3 Audio playback |
| **DS3231** | Precision RTC for logging |
| **MicroSD Module** | Local event storage |

<div class="page-break"></div>

## 🖼️ Realistic Component Layout

> **[!NOTE] Visual Reference Only**
> The realistic component layout below is for visual purposes and aesthetic representation. For exact pin mapping and electrical connections, strictly follow the official Schematic Circuit Diagram.

![Realistic Hardware Layout](./include/images/realisticComponents.png)

<div class="page-break"></div>

## 🔌 Schematic Circuit Diagram

For full hardware wiring instructions and component mapping, please refer to the official wiring schematic:

![Schematic Circuit Diagram](./include/images/schematic.png)

---

## 📍 Optimized Pin Assignment (ESP32-38P)

All peripheral pairs are assigned to physically adjacent GPIO pins on the ESP32-38P to minimize wire crossings on a prototype board.

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
GPIO14  ← IR BEAM ┐                       GPIO16  ← SIM800 RX ─┘ UART1
GPIO12  ← (Unassigned)                    GPIO4   ← Alarm LED (Red)
GND                                       GPIO0   ← (Unused, boot clock)
GPIO13  ← Status LED (Blue)               GPIO2   ← DFPlayer TX ─┐ UART2
                                          GPIO15  ← DFPlayer RX ─┘
```

<div class="page-break"></div>

## 🎮 2. How the System Works

The system has two main states or "modes". You can switch between them by holding down the **Mode Button** (GPIO 34).

### 2.1 Normal Mode (Guarding)
- **Blue Light:** Stays ON.
- **What it does:** The system uses the IR beam to watch the wall. If the beam is broken (meaning someone is jumping the wall), it triggers the alarm.
- **When triggered:** It plays a warning sound, flashes the red light, and sends a text message to a security guard.

### 2.2 Setup Mode
- **Blue Light:** Blinking fast.
- **How to enter:** Press and hold the Mode Button for 3 seconds.
- **What it does:** The system stops watching the wall and creates its own Wi-Fi network. You can connect to this Wi-Fi to change settings using your phone or laptop.
- **How to exit:** Press and hold the button for 3 seconds again. *(It will also exit automatically after 1 hour if you forget).*

### 2.3 Remote Control via Text Message (SMS)
You don't always have to press the physical button to change modes! You can send a text message to the system's SIM card to control it remotely:
- `Text AP ON`: Turns on the Wi-Fi Setup Mode so you can connect to the dashboard.
- `Text AP OFF`: Turns off the Wi-Fi Setup Mode and goes back to Normal Guarding mode.

*(Note: For extra security, if "Number Lock" is enabled, the system will only obey text commands sent from the registered security guard's phone number!)*

<div class="page-break"></div>

## ⚙️ 3. Changing Settings on the Dashboard

### 3.1 Connecting to the Setup Page
1. Put the system into **Setup Mode** (hold the button until the blue light blinks).
2. On your phone or laptop, connect to the Wi-Fi network:
   - **Wi-Fi Name:** `AntiWallJump-Config`
   - **Password:** `configure123`
3. A login page should pop up. If it doesn't, open a web browser and go to `http://192.168.4.1`.
4. The default admin password is `admin`.

### 3.2 What's on the Dashboard?
The setup page has different sections and buttons to help you control the system:

**Top Menu Buttons (Quick Actions):**
- **Sync Clock (Clock Icon):** Click this to instantly set the system's time to match your phone or computer.
- **Dark/Light Mode (Moon/Sun Icon):** Switch the screen between dark and light colors.
- **Logout (Door Icon):** Securely sign out of the setup page.
- **Stop Portal (Power Icon):** Quickly exit setup mode and turn the alarm back on right away.

**Bottom Navigation Tabs:**
- **Dashboard:** See the "Node Overview" (system health like the memory card status) and check the overall security status.
- **Registry:** Register student ID cards and see who triggered the alarm.
- **Settings:** Change the phone number for alerts, choose how close someone has to be to trigger the alarm, and pick which sound plays.
- **Records:** View a history of all past alarms and download a record file.

<div class="page-break"></div>

## 🪪 4. Student ID Cards (Registry)

The system keeps track of authorized students using their UHF RFID ID cards.

### 4.1 Adding a New Student Card
1. Go to the **Registry** tab on the dashboard.
2. Click **Scan Card**.
3. Tap a new ID card on the UHF RFID panel. The card's special ID will show up on the screen.
4. Type in the student's name and school ID number.
5. Click **Register** to save them in the system.

### 4.2 What happens when the alarm triggers?
- If the sensors spot someone climbing, it logs an **UNKNOWN INTRUDER**.
- If someone holds a registered student ID card to the reader, the system grants them safe passage.
- If an unregistered card is tapped, it triggers the alarm to keep the area secure.

---

## 📁 5. Alarm Records

The system saves a history of every time the alarm goes off. It saves this inside its internal memory and on a MicroSD card, so nothing gets lost!

### 5.1 Downloading the Records
You can download a spreadsheet file of all the past alarms:
1. Go to the **Records** tab on the dashboard.
2. Click **"Download Logs"**.
3. A file called `Campus_Security_Logs.csv` will download to your device. You can open this with Excel or Google Sheets to see a neat list of everything that happened.
