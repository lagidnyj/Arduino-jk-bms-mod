# Arduino JK-BMS Monitor

ESP32 firmware for monitoring and controlling JK-BMS devices over BLE, with a browser-based dashboard served over Wi-Fi.

The project is derived from [syssi/esphome-jk-bms](https://github.com/syssi/esphome-jk-bms/). Compatibility can vary between BMS firmware versions and ESP32 boards; verify operation with your hardware before relying on it.

## Features

- Monitor multiple JK-BMS devices by their BLE MAC addresses.
- View battery and cell measurements, temperatures, capacity, status flags, and BMS settings.
- Control charging, discharging, and balancing from the web interface.
- View device uptime, ESP32 uptime, free heap, and firmware information.
- Browse and manage files stored in LittleFS through the built-in file manager.
- Optionally upload telemetry to Micro IoT Cloud.
- The dashboard refreshes telemetry automatically. `index.html` is the full interface; `index_lite.html` is an alternative, reduced interface.

## Requirements

- An ESP32 board that supports both Wi-Fi and BLE.
- The Espressif ESP32 board package for Arduino.
- The **NimBLE-Arduino** and **ArduinoJson** libraries.
- JK-BMS devices supported by the firmware's BLE protocol implementation.

ESP32-S2 has no BLE, and ESP32-H2 has no Wi-Fi, so neither supports this project. Compatibility with BMS models and firmware versions may vary.

## Configure and build

1. Install the ESP32 board package, **NimBLE-Arduino**, and **ArduinoJson** in Arduino IDE.
2. Open `sketch/sketch.ino`. The sketch folder and `.ino` file intentionally share the name `sketch`.
3. Copy `sketch/secret.h.example` to `sketch/secret.h`:
   - Set `ssid` and `password` to your Wi-Fi credentials.
   - Add each BMS MAC address to `jkBmsMacAddresses`; the device count is derived from this list.
4. Select your ESP32 board and port. Choose a partition scheme that provides a LittleFS-compatible filesystem partition and enough space for the firmware.
5. Upload the sketch.

`sketch/partitions.csv` is a custom 4 MB flash layout with OTA app slots and a 448 KiB filesystem partition. Use it only if it matches your board's flash size and partition configuration.

The example values in `secret.h` allow the project to compile, but replace them with your own settings before use. Do not publish real Wi-Fi credentials.

## Install the web interface

The firmware stores the web files in LittleFS; uploading the firmware alone does not install them.

1. Connect the ESP32 to Wi-Fi and note the IP address printed to Serial Monitor at **115200 baud**.
2. Open `http://<device-ip>/fs` in a browser.
3. Upload `sketch/index.html` and `sketch/style.css`.
4. To use the reduced interface, make a local copy of `sketch/index_lite.html` named `index.html`, then upload that file and `sketch/style.css`.
5. To enable cloud reporting, copy `sketch/config.json.example` to `config.json`, set `cloud_enabled` to `true`, and fill in the cloud API URL, device ID, and device token. Upload it as `/config.json` using the file manager, then restart the ESP32.
6. Open `http://<device-ip>/` to view the dashboard.

Cloud reporting is disabled if `/config.json` is missing or `cloud_enabled` is not `true`. The config format matches the `example2` configuration in [esp32-daly-bms-monitor](https://github.com/lagidnyj/esp32-daly-bms-monitor/tree/main/example2); upload intervals from 60 to 180 seconds are accepted, with 60 seconds as the default. Keep the device token private. The HTTPS client currently skips certificate verification, matching the reference implementation.

If `index.html` is not present in LittleFS, the root page links to the file manager. The file manager can upload, view, and delete files, and format the filesystem. Formatting erases all files stored in LittleFS.

If LittleFS fails to mount, the firmware attempts to format it. Check the selected partition scheme if this happens; formatting can erase the dashboard files.

## HTTP endpoints

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/data` | Read telemetry and settings for configured BMS devices. |
| `POST` | `/control` | Send a charge, discharge, or balancing command. |
| `GET` | `/uptime` | Read ESP32 uptime. |
| `GET` | `/freeheap` | Read free-heap information. |
| `GET` | `/sketchinfo` | Read firmware and build information. |
| `GET` | `/fs` | Open the LittleFS file manager. |

`POST /control` accepts a JSON body with `mac`, `action`, and `state` fields. `action` is `charging`, `discharging`, or `balancing`; `state` is `on` or `off`.

The web server does not require authentication. Use it only on a trusted network; do not expose it directly to the internet.

## Project files

- `sketch/sketch.ino` — firmware.
- `sketch/secret.h` — Wi-Fi credentials and configured BMS MAC addresses.
- `sketch/CloudReporter.cpp` and `sketch/CloudReporter.h` — optional cloud reporting module.
- `sketch/config.json.example` — Micro IoT Cloud config template; upload a configured copy as `/config.json`.
- `sketch/index.html` — full web interface.
- `sketch/index_lite.html` — reduced web interface.
- `sketch/style.css` — shared dashboard styles.
- `sketch/partitions.csv` — optional custom flash partition table.
- `sketch/test-data.json` — standalone sample data; it is not loaded by the firmware.

## Disclaimer

This is an experimental project. Use it at your own risk, and verify any control action with your specific BMS before relying on it.
