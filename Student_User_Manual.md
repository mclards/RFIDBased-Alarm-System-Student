# 🎓 Student User Manual: RFID Anti-Wall-Jumping System

Welcome to the **Student User Manual**! This guide will walk you through how to assemble, install, and test your ESP32-based perimeter security system in the real world. 

---

## 1. System Overview
This project detects if someone tries to climb over a designated boundary (like a wall or fence). 
- If the **Active IR Beam** is broken, the alarm is triggered.
- If an authorized **UHF RFID Tag** is read before crossing the beam, the system grants safe passage and ignores the beam break for a few seconds.
- When an alarm is triggered, the **DFPlayer** plays a loud warning sound, and the **SIM800L** sends an emergency SMS text message!

---

## 2. Hardware Assembly & Wiring

Before taking the system outside, make sure your components are wired according to the main layout diagram. 

**Critical Connections:**
- **ESP32 (Core)**: The brain of the operation.
- **RD906M (UHF RFID)**: Connect the Wiegand pins (D0 to GPIO 21, D1 to GPIO 22).
- **SIM800L (GSM)**: Connect TX/RX to GPIO 16 and 17. **Important**: The SIM800L requires a strong 5V power supply (up to 2A) to send texts successfully. 
- **Active IR Beam**: Connect the relay signal wire to GPIO 14. 
- **DFPlayer Mini**: Connect TX/RX to GPIO 2 and 15. Make sure your MicroSD card is inserted with your `.mp3` alarm files!

---

## 3. Installation for Actual Testing

When you are ready to test the system in a real environment (like a school courtyard or field), follow these steps:

### Step 1: Align the Active IR Beam
The IR Beam has two parts: a **Transmitter (T)** and a **Receiver (R)**. 
1. Mount them on opposite sides of the wall/boundary you want to protect. 
2. Ensure they are perfectly aligned facing each other. (Most beam sensors have a small alignment LED inside that lights up when they are correctly pointed at each other).
3. If the beam is broken, the Receiver will trigger the ESP32.

### Step 2: Mount the UHF RFID Panel
1. Mount the large white RD906M panel near the entrance of the boundary.
2. Angle the panel so it faces the direction people will be walking from.
3. Ensure it is firmly powered by the 12V battery.

### Step 3: Insert the SIM Card & SD Card
1. Insert an active, unlocked Micro-SIM card into the SIM800L module. 
2. Insert a FAT32-formatted MicroSD card into the SD Card Module (for logging).
3. Insert the MicroSD card with your MP3 files into the DFPlayer Mini.

### Step 4: Power Up
1. Connect your 12V battery.
2. The power converters will step the 12V down to 5V and 3.3V to safely power the ESP32 and modules.
3. Wait about 30 seconds for the SIM800L to connect to the cellular network (the blinking LED on the SIM module will slow down to once every 3 seconds when connected).

---

## 4. Flashing the Firmware

If you haven't uploaded the code yet:
1. Open the project folder in **VS Code** with the **PlatformIO** extension.
2. Open `system_firmware.cpp`.
3. Connect the ESP32 to your laptop via USB.
4. Click the **Upload** button in PlatformIO.

---

## 5. Live Testing!

Now that everything is powered and running outside, let's test it:

1. **Test the Alarm (Intrusion)**: 
   - Walk between the IR Transmitter and Receiver to break the beam.
   - The DFPlayer should instantly play the alarm sound.
   - Within 5 seconds, your designated emergency phone should receive a text message alert!
2. **Test the Safe Passage (RFID)**:
   - Hold your UHF RFID tag and walk past the white panel. 
   - The system should register your ID (you might hear a small beep or see the status LED flash).
   - Walk through the IR Beam. 
   - The alarm **should NOT** trigger, and no text should be sent!

**Troubleshooting:**
- **No SMS received?** Check if the SIM800L LED is blinking fast (searching for signal) or slow (connected). Ensure the battery is fully charged!
- **Alarm randomly goes off?** The IR beams might be misaligned, or sunlight/wind might be interfering. Try realigning them perfectly.
