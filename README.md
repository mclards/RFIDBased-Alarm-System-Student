# RFID Anti-Wall-Jumping System

Welcome to the student repository for the RFID Anti-Wall-Jumping System!

This project demonstrates how to integrate multiple hardware components with an ESP32 microcontroller to build a functional, real-time perimeter security system. 

## 📁 Repository Contents

- **`main_student.cpp`**: The core C++ firmware logic running on the ESP32. It handles reading the RFID tags, checking the IR beam sensor for physical intrusion, and dispatching SMS alerts using the SIM800L module.
- **`main_student.pdf`**: A beautifully formatted PDF version of the code, ideal for reading, printing, or reviewing offline.

## 🧠 What You Will Learn

By studying `main_student.cpp`, you will understand:
1. **FreeRTOS Tasks**: How to use ESP32's dual cores to monitor sensors and send SMS messages simultaneously without blocking the main program.
2. **State Machines**: How to reliably send multi-step AT commands to a cellular module (`SIM800L`).
3. **Hardware Integration**: How to safely use hardware interrupts (Wiegand) and I2C/SPI interfaces concurrently.

Feel free to download the code and explore!
