# ESP32 Campus Security Gateway: Complete Web Portal Guide
**Comprehensive Reference for Web Dashboard Features, Form Inputs, Tooltips & Administration**

---

## 📌 1. Access & Authentication

### 1.1 Connecting to the Configuration Hotspot
1. Place the ESP32 into **Setup Mode** by holding the physical Mode Button (GPIO 34) for 3 seconds or by sending an SMS with `AP ON` to the device.
2. The Blue Status LED on the unit will begin **blinking rapidly** (400ms interval).
3. Connect your smartphone, tablet, or laptop to the broadcasted Wi-Fi network:
   - **Default SSID:** `AntiWallJump-Setup`
   - **Default Wi-Fi Password:** `configure123`
4. The captive portal login screen will launch automatically. If it does not, open any browser and navigate to `http://192.168.4.1`.

### 1.2 Web Authentication
- Enter the default administrative password: `admin`
- Click **Log In** to receive an authenticated Bearer token stored in your browser session.

<div class="page-break"></div>

---

## 🌐 2. Persistent Header Bar & Global Actions

The top navigation header remains visible across all pages and provides real-time system status and quick-access controls.

### 2.1 Real-Time Status Badges

| Badge Name | HTML Identifier | Source | Description |
| :--- | :--- | :--- | :--- |
| **System Clock** | `#hdrTime` | DS3231 RTC | Displays the live local time formatted as `HH:MM:SS AM/PM`. Updates every second. |
| **Node Identifier** | `#hdrNode` | `sysConfig.nodeID` | Shows the configured alphanumeric name of the alarm unit (e.g., `NORTH_WALL_01`). |
| **Cellular Signal** | `#hdrSimStatus` | SIM800L Modem | Displays GSM network registration status: **`Registered`** (Green) when online, or **`Searching`** / **`No SIM`** (Red) when offline. |
| **SIM Number** | `#hdrSimNumber` | SIM800L Modem | Displays the mobile subscriber number of the installed SIM card. |

### 2.2 Global Control Buttons

- **Sync Clock (🔄):** Immediately reads your device's local Unix timestamp and updates the hardware DS3231 RTC chip.
- **Theme Switcher (🌓):** Toggles between Dark Mode and Light Mode. Your preference is saved in local browser storage.
- **Logout (🚪):** Clears the active authentication session and redirects back to the login screen.
- **Stop Portal (⚡):** Displays a confirmation dialog, shuts down the Wi-Fi Access Point, re-arms the perimeter tripwire, and returns the device to Normal Guarding Mode.

<div class="page-break"></div>

---

## 📊 3. Tab 1: Dashboard (`#page-dashboard`)

The Dashboard provides live visibility into perimeter security status, hardware health, and recent infraction records.

### 3.1 Security Status Banner
- **`SYSTEM SECURED` (Green):** The infrared beam is intact and the system is actively guarding the perimeter.
- **`SECURITY COMPROMISED - ALARM ACTIVE` (Pulsing Red):** The infrared beam tripwire has been interrupted and an active alarm/bucketing cycle is running.

### 3.2 "Test SMS" Button (`#btnTestSms`)
- Clicking **Test SMS** sends an immediate test text message to the configured emergency phone number.
- Verifies SIM prepaid balance, antenna signal strength, and network dispatch without tripping the physical alarm.

### 3.3 Node Overview Stat Cards

| Card Label | HTML ID | Data Source | Tooltip & Explanation |
| :--- | :--- | :--- | :--- |
| **Node ID** | `#statNodeID` | `sysConfig.nodeID` | **Tooltip:** *"The unique name and software version of this alarm unit."*<br>Identifies which physical security unit is reporting across campus. |
| **Storage Media** | `#statSDStatus` | SPI SD Driver | **Tooltip:** *"Shows if the memory card is working to save logs."*<br>Displays `Mounted & Active` (Green) or `Missing / Failed` (Red). |
| **Hardware Clock** | `#statTime` | DS3231 RTC | **Tooltip:** *"The current time, automatically updated over Wi-Fi."*<br>Displays the high-precision hardware time used for all audit logging. |
| **Alert Recipient** | `#statPhone` & `#statRole` | `guardPhone` | **Tooltip:** *"The phone number that will receive the text messages."*<br>Displays the active emergency contact number and recipient role (`GUARD` / `TEACHER`). |
| **Sensor Config** | `#statSensors` | System Core | **Tooltip:** *"The physical setup of the infrared beam sensors."*<br>Confirms active sensor topology (`Active IR Beam`). |
| **Audio Module** | `#statAudio` | UART1 Driver | **Tooltip:** *"Shows if the alarm speaker is working properly."*<br>Displays `Ready` (Green) or `Missing / Error` (Red) for the DFPlayer Mini MP3 decoder. |
| **Zone Assigned** | `#statWall` | `wallLabel` | **Tooltip:** *"The name of the wall or fence being protected."*<br>Short boundary sector label (e.g., `North Wall Fence`). |
| **Exact Location** | `#statLocation` | `locationDesc` | **Tooltip:** *"The specific location description for the guards."*<br>Detailed physical description to guide security personnel to the exact site. |
| **Sensor Status** | `#statSensor` | GPIO 14 / Wiegand | **Tooltip:** *"Shows if the jump sensors and ID scanner are active."*<br>Live dual indicator: <br>• **Port 14 (IR Beam):** `CLEAR` (Green) or `TRIGGERED` (Red)<br>• **D0/D1 (RFID):** `STANDBY` or `READING...` (Blue) |

### 3.4 Immediate Incident Flag Table
- Displays the most recent security infractions registered during the current run cycle (`Timestamp`, `Status`, `Entity Name`, `Tag ID`).

<div class="page-break"></div>

---

## ⚙️ 4. Tab 2: Settings (`#page-settings`)

The Settings tab contains three sub-sections. Any modified field automatically turns the respective **Save Settings** button orange with an asterisk (`Save Changes *`) until saved.

---

### 4.1 Sub-Tab: Device Config (`#cfg-device`)

Controls hardware identification, emergency contact routing, and SMS permission filters.

| Field Label | HTML ID | Default | Constraints | Tooltip & Explanation |
| :--- | :--- | :--- | :--- | :--- |
| **Device ID** | `#cfgNodeID` | `NODE_01` | 1–31 Characters | **Tooltip:** *"Name of this alarm device (e.g., Gate 1)."*<br>Unique alphanumeric identifier for this security unit. |
| **Wall Label** | `#cfgWallLabel` | `Wall Section` | 1–31 Characters | **Tooltip:** *"Name of the physical wall or boundary being protected (e.g., Section A)."*<br>Primary location headline placed inside SMS alert text messages. |
| **Location Description** | `#cfgLocationDesc` | `Campus Perimeter` | 1–127 Characters | **Tooltip:** *"A clear description of where this wall is located to help guards find it."*<br>Full geographic description (e.g., *"Behind Chemistry Laboratory Complex Building B"*). |
| **Emergency Contact Number** | `#cfgPhone` | `+639XXXXXXXXX` | Max 31 Characters | **Tooltip:** *"The phone number that will receive the text message alerts."*<br>Supports standard local formats (`09XXXXXXXXX`), international formats (`+639XXXXXXXXX`), or 10-digit mobile numbers (`9XXXXXXXXX`). Automatically normalized on save. |
| **Number Lock** | `#cfgNumberLock` | `OFF` (0) | Toggle (`ON`/`OFF`) | **Tooltip:** *"If checked, only the emergency phone number is allowed to send text commands to the alarm."*<br>When enabled, incoming SMS commands (`AP ON`, `AP OFF`) are ignored unless sent from the configured emergency phone number. |
| **Recipient Role** | `#cfgRecipientRole` | `Security Guard` | `Guard` / `Teacher` | **Tooltip:** *"Choose the recipient role for SMS alerts (e.g., Guard or Teacher)."*<br>Sets the SMS header: `[CAMPUS ALERT - SECURITY]` for Guard, or `[CAMPUS ALERT - FACULTY]` for Teacher. |

<div class="page-break"></div>

---

### 4.2 Sub-Tab: Alert Triggers & Timers (`#cfg-alert`)

Controls the FreeRTOS event collection engine, audio siren playback, and sensor hold windows.

| Field Label | HTML ID | Default | Range | Tooltip & Explanation |
| :--- | :--- | :--- | :--- | :--- |
| **Alarm Duration (ms)** | `#cfgDuration` | `30000` | 1,000–300,000 ms | **Tooltip:** *"How long the alarm siren will sound when someone jumps the wall. (e.g., 30000 = 30 seconds)."*<br>Controls the runtime of the DFPlayer Mini siren and the Red Alarm LED (GPIO 4). |
| **Cooldown Time (ms)** | `#cfgCooldown` | `60000` | 5,000–600,000 ms | **Tooltip:** *"Wait time after an alarm before the system can trigger again. Prevents SMS and siren spam. (e.g., 60000 = 60 seconds)."*<br>Mandatory quiet period after an alarm to avoid redundant alerts while guards respond. |
| **Breach Memory (ms)** | `#cfgBreachMemoryMs` | `5000` | 1,000–60,000 ms | **Tooltip:** *"How long the system holds a motion sensor trigger before clearing it. (e.g., 5000 = 5 seconds)."*<br>Duration a beam-break pulse is remembered in memory while waiting to pair with an RFID card read. |
| **RFID Memory (ms)** | `#cfgTagMemoryMs` | `30000` | 1,000–600,000 ms | **Tooltip:** *"Time window a student has to cross the beam after scanning their ID. If they cross within this time, they are caught. (e.g., 30000 = 30 seconds)."*<br>Retention window for buffered RFID cards approaching the perimeter. |
| **Bucketing Window (ms)**| `#cfgBucketWindowMs`| `10000` | 1,000–60,000 ms | **Tooltip:** *"Time window after an alarm triggers to group other jumping students into the same SMS alert. (e.g., 10000 = 10 seconds)."*<br>Aggregation window to group accomplices jumping together into a single SMS report. |
| **Unsent SMS Expiry (min)**| `#cfgSmsAlertExpiryMinutes`| `60` | 1–1,440 min | **Tooltip:** *"Failed SMS alerts are retried until this time limit is reached. Valid range: 1 to 1440 minutes."*<br>Time queued SMS alerts will be retried during cellular network outages before expiration. |
| **Beam Alert Delay (min)**| `#cfgPersistentBeamAlertMinutes`| `15` | 1–1,440 min | **Tooltip:** *"Send one SMS when the IR beam stays broken for this long. Another SMS is allowed only after the beam recovers. Valid range: 1 to 1440 minutes."*<br>Dispatches a persistent obstruction warning if the IR beam remains continuously broken. |

#### Audio Track Controls & Voice Testing
- **Alarm Voice Track Selector (`#cfgAlarmVoice`):** Selects from 8 MP3 tracks stored on the DFPlayer Mini SD card (`/mp3/0001.mp3` through `/mp3/0008.mp3`).
  - *Track 1:* "Please return to class..."
  - *Track 2:* "Security alert. You are in a restricted zone..."
  - *Track 3:* "Warning. Unauthorized access detected..."
  - *Track 4:* "Attention. Climbing the fence is prohibited..."
  - *Track 5:* "This is a restricted area. CCTV recording in progress..."
  - *Track 6:* "Emergency alert. Perimeter breach detected..."
  - *Track 7:* High-Decibel Siren 1
  - *Track 8:* High-Decibel Siren 2
- **Play Button:** Auditions the currently selected track through the connected speaker.
- **Stop Button:** Immediately halts audio playback.
- **System Voice Prompt Tests:**
  - *Save Conf:* Plays Track 12 ("Configuration Saved").
  - *Load Avail:* Plays Track 13 ("SIM Load Available").
  - *No Load:* Plays Track 14 ("SIM Card Insufficient Balance").

<div class="page-break"></div>

---

### 4.3 Sub-Tab: System Security (`#cfg-security`)

Controls Wi-Fi credentials, gateway IP addressing, administrative passwords, and system resets.

| Field Label | HTML ID | Default | Constraints | Tooltip & Explanation |
| :--- | :--- | :--- | :--- | :--- |
| **New AP Name** | `#cfgApSSID` | `AntiWallJump-Setup` | 1–31 Characters | **Tooltip:** *"The new Wi-Fi name this alarm will broadcast."*<br>Configures the SoftAP network SSID broadcast during configuration mode. |
| **New AP Password** | `#cfgApPassword` | `configure123` | 8–63 Characters | **Tooltip:** *"The new password to connect to this alarm's Wi-Fi."*<br>WPA2 Wi-Fi passphrase. Must be at least 8 characters. |
| **Confirm AP Password** | `#cfgApPasswordConfirm` | — | Must match AP Pass | **Tooltip:** *"Type the new Wi-Fi password again to confirm."*<br>Verifies no typos were entered in the AP password field. |
| **Device IP Address** | `#cfgDeviceIP` | `192.168.4.1` | Valid IPv4 | **Tooltip:** *"The web address used to access this page. Leave as default unless needed."*<br>Gateway IP address of the Access Point. |
| **New Admin Password** | `#cfgAdminPassword` | `admin` | Min 4 Characters | **Tooltip:** *"The new password to log into this webpage."*<br>Sets the password required to log into the web dashboard. |
| **Confirm Admin Password**| `#cfgAdminPasswordConfirm`| — | Must match Admin Pass | **Tooltip:** *"Type the new webpage password again to confirm."*<br>Verifies no typos were entered in the admin password field. |
| **Reboot Hardware** | `/api/reboot` | — | Action Button | Safely reboots the ESP32 microcontroller without deleting any saved configurations or logs. |
| **Factory Reset** | `/api/factory_reset` | — | Action Button | Opens a confirmation modal requiring the user to type `CONFIRM`. Wipes EEPROM, deletes SD card logs and student registries, restores factory defaults, and restarts. |

<div class="page-break"></div>

---

## 🪪 5. Tab 3: Student Registry (`#page-registry`)

The Registry tab is used to enroll authorized student RFID cards, link them to student names and ID numbers, and review individual violation histories.

### 5.1 Enrolling a Student Card (Step-by-Step)
1. Click **Scan Card** (`#btnScan`). The scanner enters a 30-second live listening mode with an animated spinner.
2. Tap or hold the student's UHF RFID card in front of the RD906M panel:
   - **Success:** The reader decodes the tag and auto-populates the **RFID Tag ID** field (`#regTagID`).
   - **Collision Warning:** If multiple cards are detected simultaneously, the system displays: *"⚠️ Multiple cards detected! Clear the area and scan only ONE card."*
   - **Duplicate Tag Warning:** If the card is already in the database, the system identifies the currently assigned student name.
3. Enter the student's **Complete Name** in `#regName` (Tooltip: *"The full name of the student or staff member."*).
4. Enter the student's **Official Student ID** in `#regStudentID` (Tooltip: *"The official student ID number."*).
5. Click **Register Student**. The record is saved to `/students.csv` on the MicroSD card.
   - If a conflict occurs, a confirmation dialog offers to reassign the tag or update the existing student profile.

### 5.2 Registered Students Table

| Table Column | Data Content | Functional Action |
| :--- | :--- | :--- |
| **TAG ID** | Hexadecimal card ID (e.g., `1A2B3C4D`) | Unique hardware identifier. |
| **COMPLETE NAME** | Student Full Name (e.g., `Juan Dela Cruz`) | Used in SMS alerts and event logs. |
| **OFFICIAL STUDENT ID** | Institutional ID (e.g., `2026-0413`) | Clickable blue link that opens the **Student Incident History Modal**. |
| **FREQUENCY** | Total breach count (e.g., `3`) | Highlighted in red if the student has triggered the alarm. |
| **ACTION** | **Drop** button | Displays a confirmation dialog to permanently remove the student record. |

### 5.3 Student Incident History Modal (`#history-modal`)
- Clicking any student's blue ID number opens their dedicated violation log parsed directly from `/logs/<tagID>.csv`.
- Displays an infraction table containing:
  - **Timestamp:** Exact RTC date and time of the violation.
  - **Wall:** Boundary zone breached.
  - **Severity:** Infraction classification badge (`CRITICAL` or `WARNING`).

<div class="page-break"></div>

---

## 📁 6. Tab 4: Records & Data Logging (`#page-logs`)

The Records tab provides access to master event logs and data export tools.

### 6.1 Event Logs Table
- Chronological list of all perimeter incidents parsed from `/eventlog.csv`.
- Columns:
  - **TIMESTAMP:** Date and time of incident (e.g., `2026-08-17 14:30:15`).
  - **STATUS:** Severity badge: `CRITICAL` (Red) for student breaches and unidentified trips, or `WARNING` (Yellow) for unregistered cards.
  - **NAME:** Full student name, or `Unregistered` / `Unidentified`.
  - **TAG ID:** Hexadecimal card ID, or `NONE`.

### 6.2 Data Management Buttons

- **Download Logs (`#downloadLogs`):**
  - Generates and downloads a multi-sheet spreadsheet named **`Campus_Security_Logs.csv`**.
  - Contains three distinct sections:
    1. *Master Chronological Event Log*
    2. *Student Registry Summary & Infraction Counts*
    3. *Individual Student Incident Breakdowns*
  - **Mobile Browser Note:** If your phone's captive portal browser suppresses downloads, open Google Chrome or Apple Safari, type `http://192.168.4.1`, and click Download Logs.
- **Clear Logs (`#clearLogs`):**
  - Displays a confirmation prompt before permanently deleting all event logs from the MicroSD card.

<div class="page-break"></div>

---

## 🔧 7. Web Portal Troubleshooting Matrix

| Issue | Probable Cause | Corrective Action |
| :--- | :--- | :--- |
| **Cannot open `192.168.4.1`** | Wi-Fi disconnected or incorrect IP address. | Verify Wi-Fi is connected to `AntiWallJump-Setup`. Ensure mobile data is temporarily disabled if your phone tries to route traffic over cellular. |
| **Login fails with "Unauthorized"** | Incorrect password entered. | Default password is `admin`. If changed and forgotten, send SMS command `RESTORE FACTORY SETTINGS` from the Guard phone to reset. |
| **Save Settings shows "Invalid phone number"** | Number does not match carrier rules. | Enter a valid 11-digit local (`09123456789`) or 13-digit E.164 (`+639123456789`) number. |
| **Scan Card times out after 30 seconds** | Reader disconnected, unpowered, or card out of range. | Ensure RD906M reader has 12V DC power and Wiegand D0/D1 are connected to GPIO 21 and 22. |
| **Log download does not start on mobile** | Captive portal browser sandbox restriction. | Open the native mobile browser (Chrome/Safari), enter `http://192.168.4.1`, log in, and tap Download Logs. |
