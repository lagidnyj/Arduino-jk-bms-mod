# FORK DIFF
* new functions marked as // --- new function name ---

* added average calculation of time to full discharge/charge of the battery

* text formatting reduced, unnecessary elements removed to save space on the screen

* added moving BMS to the second column, for simultaneous display next to each other (in css)

* added fix for mounting file system for the first time

# Arduino script with NimBLE to monitor JK-BMS 

This project is a modified version of [syssi/esphome-jk-bms](https://github.com/syssi/esphome-jk-bms/), rewritten for Arduino using NimBLE.
Usefull for v11 / v15 BMS and also for v19 but there seem to be differences depending on [which ESP32](https://github.com/peff74/Arduino-jk-bms/issues/7#issuecomment-3515887170) you use.

## Features
*   **Real-time Monitoring**:
    
    *   All data that are also available via the app
        
    *   Cell voltages, resistances, temperatures, battery voltage, current, state of charge and more
        
    *   BMS settings and status flags.
        
*   **Control**:
    
    *   Turn **Charging**, **Discharging**, and **Balancing** ON/OFF for each BMS.
 
    *   I plan to add more.
        
*   **System Information**:
    
    *   Uptime 
        
    *   Free heap memory 
        
    *   Sketch information 
        
*   **Multi-BMS Support**:
    
    *   Monitor and control multiple BMS devices simultaneously.
        
*   **Web Interface**:
    
    *   Simple but not pretty web interface with all data and some switches.
 
    *   A small file manager is included to handle web server files
        
    *   Automatic refresh 


## Session Notes

### 2026-10-19

* Added a system summary above the BMS columns in `sketch/index_lite.html`. It averages connected-BMS voltage, sums power/current/capacities, calculates capacity-weighted state of charge, and estimates system charge/discharge time from aggregate capacity and smoothed current.
* The summary excludes disconnected BMS devices using the `connected` flag added to the `/data` response. Update the firmware as well as the LittleFS HTML/CSS files to use this feature.

### 2026-10-18

* Documented the latest stable toolchain versions available by 2025-02-28, ESP32 platform requirements, and firmware-size troubleshooting. These are historical recommendations, not a verified record of the exact versions used for the original build.
* Added `sketch/index_lite.html`, which keeps the original page intact and hides the BMS Settings and Wire Resistances sections.
* Reduced serial debug output in `sketch/jk-bms.ino`. `DEBUG_ENABLED` remains enabled; verbose notification, raw-frame, register, cell, and settings dumps were removed. Concise BMS discovery/RSSI, connection, device info, and telemetry messages remain. Wi-Fi connection and IP messages are unchanged.
* Added `sketch/test-data.json` with one nine-cell BMS sample based on the README serial log. It is a standalone fixture for experiments; the attempted HTML test-mode integration was reverted.
* Planned, not implemented: remove web controls, reduce the JSON payload to the required fields, and test remote reporting/control. Multi-BMS behavior and operation on the target ESP32 hardware still need field testing.


Installation
------------

### Recommended toolchain for a February 2025 build

The versions below were the latest stable releases available by **2025-02-28**. Use this set to reproduce a build from that period; it is a baseline, not a guarantee of the smallest binary.

| Component | Version | Release |
| --- | --- | --- |
| Arduino IDE | 2.3.4 | [Release](https://github.com/arduino/arduino-ide/releases/tag/2.3.4) |
| `esp32` boards by Espressif Systems | 3.1.3 | [Release](https://github.com/espressif/arduino-esp32/releases/tag/3.1.3) |
| NimBLE-Arduino | 2.2.3 | [Release](https://github.com/h2zero/NimBLE-Arduino/releases/tag/2.2.3) |
| ArduinoJson | 7.3.1 | [Release](https://github.com/bblanchon/ArduinoJson/releases/tag/v7.3.1) |

The sketch targets an **Espressif ESP32 board with both Wi-Fi and BLE**. The original serial log is from a classic ESP32. ESP32-S2 has no BLE, and ESP32-H2 has no Wi-Fi, so neither can run this complete sketch. `Arduino.h`, `WiFi`, `WebServer`, and `LittleFS` are provided by the ESP32 board package; do not install separate libraries for them. Install only NimBLE-Arduino and ArduinoJson as libraries.

For a classic ESP32 DevKit/WROOM, select the matching board in Arduino IDE (usually **ESP32 Dev Module**), set Flash Size to match the physical module, and choose a partition scheme with a sufficiently large app slot and a filesystem partition for LittleFS. OTA partition schemes allocate less space to each app image than non-OTA schemes.

### If the firmware does not fit

* Set `DEBUG_ENABLED` to `false` in `jk-bms.ino`. This project routes its debug messages through macros, so disabling them also removes the log strings and formatting calls from the build.
* Keep the board package, libraries, board selection, and partition scheme fixed while comparing build sizes. A larger app image can be caused by the selected core or compiler, but an app partition that is too small is a separate issue.
* The sketch's ArduinoJson calls are compatible with the 6.x API. As a size-constrained alternative, try **ArduinoJson 6.21.5** ([release](https://github.com/bblanchon/ArduinoJson/releases/tag/v6.21.5)), the latest 6.x release available at the time, and compare the compiled sizes. The size impact depends on the actual build, so verify rather than assume this version is smaller.

*   **Get startet**:

    *   copy index.html + style.css + jk-bms.ino local
    
*   **Open the Project in Arduino IDE**:
    
    *   Open the jk-bms.ino file in the Arduino IDE.
        
*   **Install Required Libraries**:
    
    *   Install **NimBLE-Arduino 2.2.3** and **ArduinoJson 7.3.1** using Library Manager (or use the pinned versions in the table above).
 
    *   `NimBLEDevice.h` is provided by NimBLE-Arduino. Wi-Fi, WebServer, and LittleFS come from the Espressif ESP32 boards package; other included headers are part of the Arduino core/toolchain.
 
*   **Edit settings**:

    *   WiFi SSID / Password + MAC from the BMS
        
*   **Upload the Sketch**:
    
    *   Connect your ESP32 to your computer.
        
    *   Select the correct board and port in the Arduino IDE.
        
    *   Upload the sketch to the ESP32.
        
*   **Access the Web Interface**:
    
    *   After the upload, the ESP32 will connect to your Wi-Fi and the IP will be displayed in the serial monitor.
        
    *   Open a web browser and navigate to the IP address of the ESP32.
 
    *   A small error message appears that no Index.html was found
 
    *   Follow the link or use http://IP/fs to upload index.html + style.css to the ESP32
 
## Examples

   *   Serial interface

<details>
<summary>📁 Click to show</summary>

```
rst:0x1 (POWERON_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
configsip: 0, SPIWP:0xee
clk_drv:0x00,q_drv:0x00,d_drv:0x00,cs0_drv:0x00,hd_drv:0x00,wp_drv:0x00
mode:DIO, clock div:1
load:0x3fff0030,len:4916
load:0x40078000,len:16436
load:0x40080400,len:4
ho 8 tail 4 room 4
load:0x40080404,len:3524
entry 0x400805b8
..
WiFi connected
IP address: 
192.168.0.102
LittleFS Mounted Successfully
Initializing NimBLE Client...
Starting scan...
BLE Device found: Name: , Address: 27:74:79:c1:9a:a6, manufacturer data: 0600010f200283b89eafdb644a971466f6f372ef262acdf0eb0c92be0f
BLE Device found: Name: JK_BD4A20S4P, Address: 20:22:08:25:26:8b, manufacturer data: 4a4b0001, serviceUUID: 0xffe0
Found target device: 20:22:08:25:26:8b
Attempting to connect to 20:22:08:25:26:8b...
New client created.
Connected to 20:22:08:25:26:8b
Connected to: 20:22:08:25:26:8b RSSI: -41
Subscribed to notifications for 0xffe1
Writing register: address=0x97, value=0x00000000, length=0
Frame to be sent: AA 55 90 EB 97 00 00 00 00 00 00 00 00 00 00 00 00 00 00 11 
Notification received...
Handling notification...
Start of data frame detected.
Notification received...
Handling notification...
Continuing data frame...
New data available for parsing.
Device info frame detected.
Processing device info...
Raw data received:
55 AA EB 90 03 77 4A 4B 5F 42 44 34 41 32 30 53 
34 50 00 00 00 00 31 31 2E 58 57 00 00 00 31 31 
2E 32 35 00 00 00 90 C4 05 00 3A 00 00 00 4A 4B 
5F 42 44 34 41 32 30 53 34 50 00 00 00 00 31 32 
33 34 00 00 00 00 00 00 00 00 00 00 00 00 32 34 
30 39 32 37 00 00 32 30 38 32 33 32 31 34 33 39 
00 30 30 30 30 00 49 6E 70 75 74 20 55 73 65 72 
64 61 74 61 00 00 31 32 33 34 00 00 00 00 00 00 
00 00 00 00 00 00 49 6E 70 75 74 20 55 73 65 72 
64 61 74 61 00 00 7C F8 FF FF 1F 0D 00 00 00 00 
00 00 90 0F 00 00 00 00 C0 D8 03 00 00 00 00 01 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 FE 
0F 00 00 00 00 00 00 00 00 00 00 53 
  Vendor ID: JK_BD4A20S4P
  Hardware version: 11.XW
  Software version: 11.25
  Uptime: 378000 s
  Power on count: 58
  Device name: JK_BD4A20S4P
  Device passcode: 1234
  Manufacturing date: 240927
  Serial number: 2082321439
  Passcode: 0000
  User data: Input Userdata
  Setup passcode: 1234
Notification received...
Handling notification...
Writing register: address=0x96, value=0x00000000, length=0
Frame to be sent: AA 55 90 EB 96 00 00 00 00 00 00 00 00 00 00 00 00 00 00 10 
20:22:08:25:26:8b connected successfully
Notification received...
Handling notification...
Start of data frame detected.
Notification received...
Handling notification...
Continuing data frame...
New data available for parsing.
BMS Settings frame detected.
Processing BMS settings...
Cell voltage undervoltage protection: 2.82V
Cell voltage undervoltage recovery: 2.85V
Cell voltage overvoltage protection: 4.20V
Cell voltage overvoltage recovery: 4.18V
Balance trigger voltage: 0.01V
Power off voltage: 2.80V
Max charge current: 25.00A
Charge overcurrent protection delay: 30.00s
Charge overcurrent protection recovery time: 60.00s
Max discharge current: 40.00A
Discharge overcurrent protection delay: 300.00s
Discharge overcurrent protection recovery time: 60.00s
Short circuit protection recovery time: 60.00s
Max balance current: 0.40A
Charge overtemperature protection: 70.00C
Charge overtemperature protection recovery: 60.00C
Discharge overtemperature protection: 70.00C
Discharge overtemperature protection recovery: 60.00C
Charge undertemperature protection: -20.00C
Charge undertemperature protection recovery: -10.00C
Power tube overtemperature protection: 100.00C
Power tube overtemperature protection recovery: 80.00C
Cell count: 9
Total battery capacity: 40.00Ah
Short circuit protection delay: 1500.00us
Balance starting voltage: 3.00V
Notification received...
Handling notification...
Notification received...
Handling notification...
Start of data frame detected.
Notification received...
Handling notification...
Continuing data frame...
New data available for parsing.
Cell data frame detected.
Parsing data...

--- Data from 20:22:08:25:26:8b ---
Cell Voltages:
  Cell 01: 3.695 V
  Cell 02: 3.696 V
  Cell 03: 3.696 V
  Cell 04: 3.698 V
  Cell 05: 3.697 V
  Cell 06: 3.701 V
  Cell 07: 3.700 V
  Cell 08: 3.700 V
  Cell 09: 3.696 V
  Cell 10: 0.000 V
  Cell 11: 0.000 V
  Cell 12: 0.000 V
  Cell 13: 0.000 V
  Cell 14: 0.000 V
  Cell 15: 0.000 V
  Cell 16: 0.000 V
wire Resist:
  Cell 01: 0.321 Ohm
  Cell 02: 0.326 Ohm
  Cell 03: 0.332 Ohm
  Cell 04: 0.325 Ohm
  Cell 05: 0.310 Ohm
  Cell 06: 0.314 Ohm
  Cell 07: 0.314 Ohm
  Cell 08: 0.312 Ohm
  Cell 09: 0.313 Ohm
  Cell 10: 0.000 Ohm
  Cell 11: 0.000 Ohm
  Cell 12: 0.000 Ohm
  Cell 13: 0.000 Ohm
  Cell 14: 0.000 Ohm
  Cell 15: 0.000 Ohm
  Cell 16: 0.000 Ohm
Average Cell Voltage: 3.70V
Delta Cell Voltage: 0.01V
Balance Curr: 0.00A
Battery Voltage: 33.28V
Battery Power: 0.00W
Charge Current: 0.00A
Charge: 77%
Capacity Remain: 30.81Ah
Nominal Capacity: 40.00Ah
Cycle Count: 0.00
Cycle Capacity: 0.05Ah
Temperature T1: 22.0C
Temperature T2: 22.9C
Temperature MOS: 28.1C
Uptime: 4d 9h 4m
Charge: 1
Discharge: 1
Balance: 0
Balancing Action: 0
Notification received...
Handling notification...
Ignoring notification. Remaining: 9
Notification received...
Handling notification...
Ignoring notification. Remaining: 8
Notification received...
Handling notification...
Ignoring notification. Remaining: 7
Notification received...
Handling notification...
Ignoring notification. Remaining: 6
Notification received...
Handling notification...
Ignoring notification. Remaining: 5
Notification received...
Handling notification...
Ignoring notification. Remaining: 4
Notification received...
Handling notification...
Ignoring notification. Remaining: 3
Notification received...
Handling notification...
```
</details>


 *   Web interface

![Exampel](https://github.com/peff74/Arduino-jk-bms/blob/main/jkbms_website.png)

## Disclaimer
This is an experimental project. Use it at your own risk.

## Credits
Based on the work of [syssi](https://github.com/syssi).

![Badge](https://hitscounter.dev/api/hit?url=https%3A%2F%2Fgithub.com%2Fpeff74%2FArduino-jk-bms&label=Hits&icon=github&color=%23198754&message=&style=flat&tz=UTC)
