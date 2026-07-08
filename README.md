# ESP32 NMEA2000 Multiprotocol Gateway

This ESP32-based NMEA 2000 gateway reads marine CAN bus data and broadcasts it simultaneously over Wi-Fi as Signal K, NMEA 0183, and Actisense Binary streams. It includes an onboard AIS parser for Class A/B targets. The built-in WebGUI acts as a lightweight chartplotter and instrument display for live navigation, engine, and weather data.

---

##  Key Features

* **Triple-Protocol Streaming:** Simultaneously outputs Signal K (WebSockets), NMEA 0183 (TCP/UDP), and Actisense Binary (UDP).
* **WebGUI Dashboard:** Responsive, browser-based glass-cockpit featuring tactical maps, true Course Over Ground (COG-T), engine gauges, wind instruments, and tank levels.
* **Integrated AIS Parser:** Automatically processes and displays Class A (commercial) and Class B (leisure) targets on the map and data lists.
* **Navigation Software Integration:** Works seamlessly with OpenCPN, Navionics, Avalon Offshore, and standard Signal K ecosystems.
* **Offline Functionality:** The tactical instrument dashboard and basic navigation fully operate without an active internet connection.


## Installation & Quick Start

This repository is a distribution point for pre-compiled binaries (Freeware). Source code is not hosted here.

1. Navigate to the `release/` directory in this repository.
2. Download the `factory_*.bin` file that matches your specific microcontroller architecture.
3. Open the `docs/` directory and follow the **Installation Guide** for detailed wiring diagrams and flashing instructions using standard ESP tools.


## Disclaimer and License

This software is provided as Freeware for personal, non-commercial use on private vessels. 

**CRITICAL MARINE SAFETY NOTICE:** This software is provided "AS IS" and is NOT a certified navigational device. The author assumes no liability for any direct or indirect damages, navigation errors, vessel grounding, or hardware failures arising from the use of this firmware. Maritime navigation requires certified equipment and constant human oversight. 

**By downloading and using this firmware, you acknowledge that you use it entirely at your own risk.**

Please see the `LICENSE.md` file for the full End-User License Agreement.