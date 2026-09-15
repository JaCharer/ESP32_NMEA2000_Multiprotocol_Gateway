# NMEA 2000 Multiprotocol Gateway User Manual

## Table of Contents

1. [Introduction](#introduction)
2. [System Overview](#system-overview)
3. [Supported Protocols and Features](#supported-protocols-and-features)
4. [Hardware and Network Requirements](#hardware-and-network-requirements)
5. [Installation and Firmware Flashing](#installation-and-firmware-flashing)
   - [Flashing the Firmware](#flashing-the-firmware)
   - [First Connection and Default Credentials](#first-connection-and-default-credentials)
   - [Factory Reset](#factory-reset)
6. [Accessing the Device](#accessing-the-device)
7. [Main Dashboard](#main-dashboard)
   - [Navigation Panel](#navigation-panel)
   - [Wind and Speed Instruments](#wind-and-speed-instruments)
   - [Compass and Heading View](#compass-and-heading-view)
   - [AIS Target List](#ais-target-list)
   - [Weather and Environment](#weather-and-environment)
   - [Engine Instruments](#engine-instruments)
   - [Tanks](#tanks)
   - [Power / DC Sources](#power--dc-sources)
   - [Live Data Cards](#live-data-cards)
8. [System Configuration](#system-configuration)
   - [WiFi Network](#wifi-network)
   - [Client Mode and Access Point Mode](#client-mode-and-access-point-mode)
   - [Advanced Network Settings](#advanced-network-settings)
   - [Signal K and Identity Settings](#signal-k-and-identity-settings)
   - [Services and Routing](#services-and-routing)
   - [AIS Settings](#ais-settings)
   - [Hardware Routing and Labels](#hardware-routing-and-labels)
   - [CAN Hardware and NMEA 2000](#can-hardware-and-nmea-2000)
   - [Advanced and System Settings](#advanced-and-system-settings)
   - [Data Smoothing (Damping)](#data-smoothing-damping)
   - [N2K Generator Mode (Demo)](#n2k-generator-mode-demo)
9. [Device and Network Management](#device-and-network-management)
   - [Detected Devices](#detected-devices)
   - [Virtual Patch Panel and Source Arbitration](#virtual-patch-panel-and-source-arbitration)
10. [Output Streams and Data Distribution](#output-streams-and-data-distribution)
   - [Signal K Stream](#signal-k-stream)
   - [NMEA 0183 Output](#nmea-0183-output)
   - [Actisense Binary Output](#actisense-binary-output)
11. [Connecting Navigation Software](#connecting-navigation-software)
   - [OpenCPN](#opencpn)
   - [Navionics (Boating App)](#navionics-boating-app)
   - [Boat Instrument (Signal K app)](#boat-instrument-signal-k-app)
   - [Other Compatible Software](#other-compatible-software)
12. [Diagnostics and Monitoring](#diagnostics-and-monitoring)
   - [General Diagnostics](#general-diagnostics)
   - [Memory and RTOS Diagnostics](#memory-and-rtos-diagnostics)
   - [Execution Time Statistics](#execution-time-statistics)
13. [Status and Configuration Monitor](#status-and-configuration-monitor)
14. [Troubleshooting](#troubleshooting)
15. [Maintenance and Best Practices](#maintenance-and-best-practices)
16. [Safety and Operational Disclaimer](#safety-and-operational-disclaimer)
   - [License and Legal Notice](#license-and-legal-notice)
   - [Acknowledgments and Open-Source Libraries](#acknowledgments-and-open-source-libraries)
17. [Appendix: Supported NMEA 2000 PGNs and Data Points](#appendix-supported-nmea-2000-pgns-and-data-points)
18. [Glossary](#glossary)

---

## Introduction

This manual describes the operation of the NMEA 2000 multiprotocol gateway, a marine data integration device designed to collect data from the NMEA 2000 CAN bus and distribute it across multiple interfaces and software ecosystems. The gateway is intended for use on recreational and small commercial vessels where it is necessary to share navigation, engine, tank, weather, and AIS information across several onboard applications.

The device can be used as a bridge between the NMEA 2000 network and the wider marine digital environment. It consolidates data from multiple sensors and devices, then transmits that data over standard marine communication protocols to chartplotters, navigation systems, monitoring dashboards, and web-based displays.

The gateway supports multiple protocols and data streams, including:

- NMEA 2000 CAN bus input
- Signal K over WebSockets
- NMEA 0183 output over TCP and UDP
- Actisense Binary output over TCP and UDP
- AIS input reception and parsing
- Local web dashboard for live vessel status monitoring

This system is especially useful when a vessel uses a mix of traditional marine instruments and modern digital software. Instead of creating isolated data silos, the gateway allows the same information to be exposed to different clients and applications without requiring multiple custom integrations.

---

## System Overview

The gateway consists of a compact ESP32-based controller, a marine data interface, a WiFi communication layer, and a browser-based user interface. It is designed to operate as a living bridge between the vessel’s onboard NMEA 2000 network and the user’s computer, tablet, smartphone, or other network-connected marine devices.

The main functions of the system are:

- reading and decoding NMEA 2000 packets from the CAN bus
- identifying connected devices and data streams
- routing preferred sources for specific measurement types
- publishing live data on network interfaces
- visualizing critical navigation and vessel data in the built-in dashboard
- providing system diagnostics for network, memory, and runtime health

The device can run as a standalone marine network node and is typically configured through the built-in web UI, which is served locally by the hardware.

---

## Supported Protocols and Features

### NMEA 2000

NMEA 2000 is the primary vessel network standard used by modern marine electronics. The gateway listens to the CAN bus to receive PGNs and decode vessel data such as:

- GPS position
- heading and course over ground
- speed over ground and through water
- depth
- wind data
- tank levels
- engine information
- battery and DC source status
- AIS targets

For the complete, authoritative list of every NMEA 2000 PGN the decoder currently parses, and exactly which data point each one populates, see [Appendix: Supported NMEA 2000 PGNs and Data Points](#appendix-supported-nmea-2000-pgns-and-data-points).

### Signal K

Signal K is a JSON-based marine data model widely used by modern navigation and monitoring tools. The gateway exposes live vessel data through a local Signal K server and WebSocket stream, making it compatible with software that expects standardized Signal K messages.

### NMEA 0183

NMEA 0183 is a legacy but still common marine protocol used by many chartplotters, instruments, and onboard displays. The gateway can publish NMEA 0183 sentences over TCP and UDP, allowing compatibility with older systems and software that does not support NMEA 2000 directly.

### Actisense Binary

Actisense Binary is a raw binary protocol commonly used in Windows-based navigation software and chartplotters. The gateway can provide a compatible binary stream that emulates a PC gateway interface to applications expecting Actisense-style traffic.

### AIS Processing

The system includes AIS parsing and filtering functions. It can process incoming AIS targets, maintain a local target database, apply refresh logic based on range and priority, and expose tracked ships for display and client integration.

### Web User Interface

The built-in web interface gives access to:

- live dashboard
- configuration pages
- device discovery
- source arbitration
- diagnostics
- system status and JSON monitoring

---

## Hardware and Network Requirements

The gateway is designed for marine environments and must be installed in a protected, dry, and properly ventilated area. The device should be connected to a stable 3.3V/5V power source according to the selected hardware design and to the appropriate NMEA 2000 CAN bus wiring.

Typical requirements include:

- compatible ESP32-based hardware platform
- CAN transceiver and proper NMEA 2000 wiring
- stable power supply for the control unit
- local WiFi access or an access point configuration
- network access for chartplotters, tablets, phones, or PC clients
- a local IP address or mDNS name for browser access
- internet access on the gateway or client device, if the interactive live map or the online weather forecast is required (the dashboard remains fully functional without it, using the offline instrument view and live N2K sensor readings)

For proper operation, the system should be installed in a location that allows reliable WiFi performance and access to the vessel's NMEA 2000 network. Long cable runs, noisy power supplies, and poor grounding may affect system reliability.

---

## Installation and Firmware Flashing

### Flashing the Firmware

The gateway ships as pre-compiled firmware and is flashed directly from a Chromium-based web browser (Chrome, Edge, or Opera) — Safari and Firefox do not support the required Web Serial API.

1. **Identify the correct firmware file** for your hardware:
   - **ESP32-S3 (newer board):** the `factory_esp32-s3...` `.bin` file.
   - **Standard ESP32 / Wemos (older board):** the `factory_esp32dev...` `.bin` file.
2. Open the official Espressif Web Flasher at **https://espressif.github.io/esptool-js/**.
3. Connect the board to the computer with a data-capable USB cable, click **Connect** in the browser, and select the correct USB port from the popup (it may be listed as "USB JTAG/serial debug unit", "CH340", or similar).
   - If the connection fails or gets stuck, press and hold the physical **BOOT** button on the board while clicking **Connect**.
4. In the **Flash Address** field, enter `0x0` (this is correct for both board types). Click **Choose File**, select the downloaded `.bin` file, then click **Program**, and wait until the progress bar reaches 100% without disconnecting the cable.
5. The board does not restart automatically after flashing. Perform a physical reset by pressing the **RST / EN** button, or by unplugging and reconnecting the USB cable.

### First Connection and Default Credentials

After a successful boot, the gateway broadcasts its own WiFi network (Access Point / Router Mode). The default network name (SSID) and password depend on the hardware:

| Board Type | Default WiFi Network (SSID) | Default Password |
|---|---|---|
| ESP32-S3 | `n2k-gateway-s3` | `baltyk2026` |
| Standard ESP32 | `n2k-gateway-generic` | `baltyk2026` |

Connect a phone or computer to this network, then open a browser to the default address `http://192.168.4.1` to reach the dashboard.

To join the vessel's existing WiFi network, go to **System Configuration → WiFi Network** and switch to Client Mode (see [WiFi Network](#wifi-network)). Once the gateway successfully joins the network, it stops broadcasting its own access point, and can be reached instead through its mDNS name:

- ESP32-S3: `http://n2k-gateway-s3.local`
- Standard ESP32: `http://n2k-gateway-generic.local`

Changing the default password immediately after first setup is strongly recommended.

### Factory Reset

If access to the device is lost, or the wrong WiFi settings were entered, the gateway can be restored to its original state:

- Press and hold the physical button connected to **PIN 0 (BOOT button)** for **6 to 8 seconds**.
- The device wipes its stored configuration, reboots, and starts broadcasting its original default Access Point network again, as shown in the table above.

---

## Accessing the Device

To access the gateway, connect to the device using the configured IP address or hostname in a web browser. The interface is served locally by the gateway and can usually be reached through the device’s IP or through an mDNS name if configured.

Common access methods:

- direct IP address in the browser, for example http://192.168.4.1
- mDNS hostname, eg. default 
   + ESP32-S3: `http://n2k-gateway-s3.local`
   + Standard ESP32: `http://n2k-gateway-generic.local`
- connection through a local boat router or marina WiFi

After opening the web interface, the user can navigate between the main dashboard, configuration pages, diagnostics, device manager, and status screens.

The header bar shown on every page includes a row of status indicators — **GUI**, **aBin**, **N2K**, **SK**, and **0183** — that turn green when the corresponding connection or service is active. Each indicator is also a shortcut: tapping or clicking it opens a dedicated quick-access page for that specific service (WebSocket link status, Actisense Binary, detected N2K devices, Signal K, and NMEA 0183, respectively), where the service can be enabled, disabled, and fine-tuned without opening the full System Configuration page.

---

## Main Dashboard

The main dashboard is the primary working interface. It is designed to give the operator a compact overview of the vessel’s status and the most important navigation parameters in one screen.

### Navigation Panel

The navigation section displays the most important data related to vessel movement and position, such as:

- vessel position
- GPS coordinates
- speed over ground
- course over ground
- depth
- satellite count and HDOP status

When the gateway has internet access, the dashboard displays a live, interactive map showing the vessel's own position and heading, its recent track, and any tracked AIS targets. The map supports panning, pinch-to-zoom, and scroll/trackpad zoom, and provides the following on-map controls:

- **Center on Vessel** – recenters and re-locks the map on the vessel's current position. Manually panning or dragging the map disengages this automatic centering.
- **Clear Track** – permanently removes the recorded vessel track from the map and from local storage.
- **TWD Widget** – a compact compass-style indicator in the top-right corner showing true wind direction directly on the map.

Tapping or clicking an AIS target icon on the map opens a popup with the vessel's name, call sign, type, speed over ground (SOG), course over ground (COG), and how long ago it was last heard from.

Because the map, its fonts, and its tile imagery are loaded from external internet sources, a live map requires the gateway (or the connected client device) to have internet access. If no internet access is available, the dashboard automatically falls back to an offline instrument view, still presenting the key numerical values needed for safe navigation support and quick visual checks.

### Wind and Speed Instruments

This panel focuses on wind and speed information. It shows values such as:

- apparent wind speed (AWS)
- apparent wind angle (AWA)
- true wind speed (TWS)
- true wind angle (TWA)
- speed through water (STW)

This information is useful for sailing performance monitoring and tactical situational awareness.

### Compass and Heading View

The compass section gives a graphical representation of heading and course information. It includes:

- heading or course over ground
- directional visual indicators
- variation information
- kinematic and orientation feedback

This display is intended to provide quick orientation without requiring the user to interpret multiple raw values.

### AIS Target List

Below the main instrument cards, the dashboard shows a live table of all currently tracked AIS targets. Targets are automatically grouped into collapsible zones, sorted by distance within each zone:

- **Zone 1: Close Quarters** – targets within 3 nautical miles.
- **Zone 2: Mid Range** – targets between 3 and 10 nautical miles.
- **Zone 3: Far Range** – targets beyond 10 nautical miles.
- **Aids to Navigation (AtoN)** – buoys, beacons, and other fixed AIS aids.
- **No Position Data** – targets for which a valid position has not yet been received.

Each zone header can be tapped or clicked to expand or collapse it. For each target, the list shows the vessel name and MMSI, call sign, distance, speed over ground (SOG), course over ground (COG), and the time elapsed since the last update. Entries that have not been updated in over 3 minutes are visually marked as stale.

When a target has a valid position, clicking or tapping its row scrolls the dashboard to the map and flies directly to that vessel's location.

### Weather and Environment

The Meteo card combines live sensor readings from the NMEA 2000 bus with an online weather forecast:

- **Current (N2K)** – live readings taken directly from onboard sensors: Air Temperature, Atmospheric Pressure, and Water Temperature.
- **Forecast** – once the vessel's GPS position is known, the dashboard automatically downloads an extended weather and sea-state forecast for the current location, refreshed roughly every 30 minutes (or retried every 5 minutes if the previous attempt failed). It shows conditions for **Now**, **+3 Hours**, **+12 Hours**, and **+24 Hours**, each with a sky icon and description, air temperature, wind speed and gusts (in knots) with direction, and wave height and direction. Sunrise and sunset times for the current day are shown below the forecast rows.

Because the forecast is retrieved from an internet weather service, it requires the gateway or client device to have internet access; the "Current (N2K)" readings continue to work offline, from the vessel's own sensors.

### Engine Instruments

The Engines card displays one analog-style RPM gauge per engine (Port and Starboard), scaled from 0 to 4,500 RPM with color-coded performance zones — green (1,000–2,500 RPM), yellow (2,500–3,500 RPM), and red (3,500–4,500 RPM, redline). Next to each gauge, a details panel shows the exact RPM value, coolant temperature, alternator voltage, and accumulated engine hours.

This card — and each individual engine gauge within it — is hidden automatically until that specific engine is actually running (RPM greater than zero), keeping the dashboard uncluttered when the engines are off.

### Tanks

The Tanks card shows a bar-style level indicator for every configured tank, grouped by fluid type: Fuel, Fresh Water, Gray Water, and Black Water. Each tank shows its current level as a percentage and, where a capacity has been configured, the equivalent volume in liters.

Tank level bars turn red and are visually flagged once they cross a safety threshold:

- **Fuel** and **Fresh Water** – flagged when level drops to **20% or below**.
- **Gray Water** – flagged when level rises to **85% or above**.
- **Black Water** – flagged when level rises to **80% or above**.

Tank names can be customized on a per-instance basis; if no custom label has been configured, tanks are shown as "Tank 1", "Tank 2", and so on.

### Power / DC Sources

The Power / DC Sources card shows one entry per detected DC device — batteries, alternators, DC-DC converters, solar controllers, wind generators, or fuel cells, each with a matching icon. Every entry displays live voltage and current (a negative current value indicates the source is discharging), and, when the connected device reports them, additional fields appear automatically:

- **SoC** – State of Charge, as a percentage.
- **Temp** – device temperature.
- **Time remaining** – estimated time until discharge, where supported.

As with tanks, each DC source can be given a custom display name; otherwise it is labeled "DC Source 1", "DC Source 2", and so on.

### Live Data Cards

The dashboard also contains a set of live data cards for parameters such as:

- heading
- course
- speed
- depth
- wind direction and speed
- vessel position

These cards update in real time and allow the operator to monitor the most important navigation values without leaving the dashboard.

---

## System Configuration

> **How to get here:** from the Main Dashboard, tap or click the **CONFIG** button in the footer.

The Configuration page is the main control center for the gateway. It is organized into collapsible cards, each covering a specific area of functionality: network connectivity, identity, output protocols, AIS behavior, hardware mapping, CAN bus settings, system limits, and instrument smoothing. Most settings are saved automatically when changed; network and hardware-related settings may require a device restart, which the interface will indicate.

### WiFi Network

The WiFi Network card controls how the gateway connects to, or creates, a local network. A single **Network Operating Mode** selector switches between the two available modes:

- **Client Mode** – the gateway connects to an existing WiFi network, such as a marina WiFi or the vessel's own router.
- **Router Mode** – the gateway broadcasts its own WiFi network for direct local connection.

Only the fields relevant to the selected mode are shown; the other section is hidden automatically.

**Client Mode fields:**

- **Target SSID** – the name of the network to join.
- **Password** – the WiFi password. Leave blank to keep the currently stored password unchanged.
- **DHCP Mode (Auto IP)** – when enabled, the gateway automatically requests an IP address from the router. When disabled, the static IP fields below become active:
  - **Static IP** – the fixed address the gateway will use on the foreign network.
  - **Gateway** – the IP address of the main router on that network.
  - **Subnet Mask** – typically `255.255.255.0`.

**Router Mode (Access Point) fields:**

- **Broadcast SSID** – the name of the network the gateway will create.
- **AP Password** – the password required to join this network. Leave blank to keep it unchanged.
- **Router IP** – the gateway's own address when acting as a router (e.g. `192.168.4.1`).
- **Gateway** – usually identical to the Router IP.
- **Subnet Mask** – typically `255.255.255.0`.

### Client Mode and Access Point Mode

Client Mode is recommended whenever the vessel already has a suitable WiFi network available. In this mode, the gateway joins the network and becomes reachable on the local LAN using either a DHCP-assigned or manually configured static address.

Router Mode is useful when no existing network is available, or when a dedicated, isolated connection is preferred for setup and troubleshooting. In this mode, the gateway itself becomes the access point that a phone, tablet, or laptop can join directly.

### Advanced Network Settings

This collapsible sub-section (inside the WiFi Network card) exposes parameters shared by both operating modes:

- **mDNS Name** – the local hostname used to reach the device as `http://<name>.local` instead of typing its IP address.
- **NTP Server** – the address of the Network Time Protocol server used to synchronize the system clock over the internet.
- **STA Fallback Timeout (s)** – how long, in seconds, the gateway attempts to join the configured router before automatically falling back to standalone Router Mode.
- **AP WiFi Channel** – the WiFi channel (1–13) used when the gateway is broadcasting in Router Mode. Channels 1, 6, or 11 are recommended to minimize interference with nearby networks.

### Signal K and Identity Settings

The Signal K & Identity card controls the built-in Signal K stream and the identity the gateway presents to marine software:

- **Enable Signal K WebSockets (Port 80)** – activates the Signal K server and discovery engine on the standard network port 80.
- **MMSI Number** – the vessel's 9-digit Maritime Mobile Service Identity.
- **Signal K Token** – reserved for a future security feature used to authorize writes back to a central Signal K node; this field is currently disabled and not yet implemented.
- **Source Label (sLabel)** – the specific identifier for this physical gateway instance inside the Signal K data tree (e.g. `n2k-gateway-s3`).
- **Source Name (sName)** – the general software class/type identifier broadcast in the Signal K discovery header (e.g. `N2K-Gateway`).

### Services and Routing

The Services & Routing card enables and configures the two legacy-compatible output streams:

**NMEA 0183 Stream**

- **NMEA 0183 Stream** – enables ASCII sentence output over the network.
- **0183 Port** – the network port used to broadcast NMEA 0183 strings (default: `10110`).
- **TCP** – enables a connection-oriented TCP server, suitable for charting applications such as OpenCPN or Navionics.
- **UDP** – enables high-speed, connectionless UDP broadcast across the local network.

**Actisense Binary (PC Gateway)**

- **Actisense Binary (PC Gateway)** – enables a raw binary stream that emulates an Actisense NGT-1 gateway, useful for PC-based chartplotters.
- **Actisense Port** – the network port used for the binary stream (default: `10120`).
- **TCP** – enables a reliable TCP socket server for the binary stream.
- **UDP** – enables UDP transmission for low-overhead binary NMEA 2000 packet propagation.

### AIS Settings

The AIS Settings card enables AIS tracking and tunes the performance of the local AIS target database:

- **Enable AIS receiving and parsing** – activates background parsing and logging of transiting AIS targets.
- **Max targets in database** – the absolute memory limit for tracked vessels, preventing memory exhaustion in dense commercial traffic.
- **Near zone (NM)** – radius, in nautical miles, within which targets receive the highest-priority refresh rate.
- **Mid zone (NM)** – radius, in nautical miles, of the medium-priority refresh ring.

An **Advanced AIS Filters & Refresh Rates** sub-section provides finer control:

- **Static data fetch range (NM)** – the maximum range at which heavier static data blocks (such as vessel names) are processed.
- **Near zone (ms)** – transmission interval for close-range dynamic targets (recommended: 2000 ms).
- **Mid zone (ms)** – transmission interval for mid-range dynamic targets (recommended: 10000 ms).
- **Far zone (ms)** – transmission interval for distant dynamic targets (recommended: 60000 ms).
- **Static data (ms)** – minimum interval between static-data updates (callsign, dimensions, ship type, etc.) for the same target (recommended: 18000 ms).
- **Max dynamic msgs / cycle** – the maximum number of AIS targets (dynamic and/or static combined) the gateway will process in a single processing cycle. This is the main safety valve that keeps the CPU from being overloaded when many vessels are in range at once.
- **Max static msgs / cycle** – a tighter, additional cap specifically on how many of those targets may include a static-data update within the same cycle, since static data is heavier to process than a position update.

These settings allow the operator to balance network traffic, CPU load, and target tracking quality. In dense traffic areas, tuning the zone ranges and refresh rates is important for stable operation.

#### How AIS Traffic Throttling Works

In busy waters with many AIS contacts, the gateway cannot update every target every cycle without overloading the CPU and flooding connected clients, so it applies the following logic on every processing pass:

1. **Distance-based refresh zones.** Each target's straight-line distance from the vessel is checked against the configured Near/Mid/Far zone radii. A target is only due for a dynamic (position/speed/course) update once its zone's configured interval has elapsed since it was last sent — close targets refresh far more often than distant ones.
2. **Separately throttled static data.** Static details (name, callsign, dimensions, ship type) are only refreshed for targets within the Static Data Fetch Range, once their own longer interval has elapsed, and only up to the "Max static msgs / cycle" limit — protecting performance, since static data is more expensive to build and send than a position update.
3. **A hard per-cycle ceiling.** No more than "Max dynamic msgs / cycle" targets are processed in total per cycle, regardless of how many are technically due for an update. This keeps worst-case CPU and bandwidth use bounded even with hundreds of contacts nearby.
4. **Fair round-robin scanning.** Rather than always starting from the same point in the target list (which could starve some targets of updates entirely under heavy load), the gateway remembers which target it left off on and resumes scanning from there on the next cycle, wrapping back to the start once it reaches the end. This ensures every tracked target eventually gets serviced in rotation.
5. **Independent Web UI refresh.** The Main Dashboard's AIS Target List is updated independently of the NMEA 0183 and Signal K throttling above — it refreshes only when a target's data has actually changed since the last time it was shown. When a browser first connects (or reconnects), the gateway automatically performs a one-time full dump of the entire AIS database to that client, spread across as many cycles as needed, so a newly opened dashboard shows the complete picture immediately rather than waiting for each target's natural refresh timer.

### Hardware Routing and Labels

The Hardware Routing & Labels card lets the operator assign custom display names and specific NMEA 2000 hardware instances to each display slot on the dashboard. The available dropdown options populate dynamically based on the devices and instances actually detected on the CAN bus, covering:

- engines
- DC sources
- fuel tanks
- fresh water tanks
- gray water tanks
- black water tanks

A **Reset Routing / Labels** button is available to clear all custom mappings and return to automatic assignment.

### CAN Hardware and NMEA 2000

The CAN Hardware & NMEA 2000 card configures how the gateway presents itself on the physical CAN bus:

- **CAN Source ID** – the source node address (0–251) the gateway claims on the NMEA 2000 bus (standard default is `22`).
- **TXD Pin (GPIO)** – the ESP32 GPIO pin wired to the CAN transceiver's Transmit line.
- **RXD Pin (GPIO)** – the ESP32 GPIO pin wired to the CAN transceiver's Receive line.
- **Model ID (mId)** – the hardware name reported to marine multifunction displays (e.g. Garmin or Raymarine device lists).
- **Product Code (pCode)** – the NMEA-certified device class category code (for example, the code identifying a Gateway-class device).

> **Pin entry format:** enter only the bare GPIO number (e.g. `4`, `5`, or `16`) — never a board silkscreen label such as `D4`, `GPIO5`, or `TX`. On boards like Wemos/NodeMCU, a pin printed as `D4` on the board may actually correspond to a different GPIO number; always check the pinout diagram for the specific board before entering a value.
>
> **Wiring note:** unlike a UART serial connection, where TX and RX are normally crossed between two devices, a CAN bus transceiver is wired straight through (1:1) — the GPIO entered as **TX PIN** connects to the transceiver's **TX/TXD** pin, and the GPIO entered as **RX PIN** connects to the transceiver's **RX/RXD** pin. Do not cross them.
>
> **Termination resistor:** make sure the CAN transceiver does not have a built-in 120 Ω termination resistor, unless the gateway is physically positioned at the very end of the NMEA 2000 backbone. A correctly wired NMEA 2000 network has exactly two terminating resistors, one at each end of the main trunk cable.
>
> Changes to WiFi settings or CAN pins require a restart of the gateway to take effect.

### Advanced and System Settings

This card contains general system behavior and connection limits:

- **Log Level (Serial Port)** – the verbosity of diagnostic logs sent to the USB serial console, from `0: Completely Disabled` up to `5: Development Verbose`.

A **Connection Limits (Requires Restart)** sub-section sets the maximum number of simultaneous clients per service:

- **Max Web GUI Clients** – maximum simultaneous browser sessions viewing the dashboard.
- **Max Signal K Clients** – maximum third-party applications connected to the Signal K stream.
- **Max NMEA0183 TCP Clients** – maximum TCP clients on the NMEA 0183 stream.
- **Max Actisense TCP Clients** – maximum TCP clients on the Actisense binary stream.

Because these values affect memory allocation, changes to Connection Limits require a device restart to take effect.

### Data Smoothing (Damping)

The Data Smoothing card controls how quickly the dashboard instruments react to incoming changes. Each parameter uses a slider from `0` (instant response) to `10` (heavily averaged, slow-changing reading, useful in rough seas):

- **Wind Damping (0–10)** – smooths apparent and true wind speed and angle readings (AWS, AWA, TWS, TWA, TWD).
- **Heading Damping (0–10)** – smooths compass heading and course over ground (COG).
- **Speed Damping (0–10)** – smooths speed through water (STW) and speed over ground (SOG).

### N2K Generator Mode (Demo)

This card is only visible on development firmware builds. When present, it allows the operator to enable **data simulation (Demo Mode)**, which generates simulated NMEA 2000 traffic for testing and office demonstrations without a live CAN bus connection.

> **Trying out source arbitration:** the internal generator intentionally simulates two different devices broadcasting GPS position at the same time (a GPSMAP 8412xsv and an AIS700 Transceiver). With both active, the vessel position on the dashboard may fluctuate or "jump" — this is expected, and is a convenient way to test the [Virtual Patch Panel](#virtual-patch-panel-and-source-arbitration): open the **N2K** quick-access page from the header status bar and lock the position stream to your preferred simulated source to see prioritization take effect immediately.
>
> The internal generator simulates standard navigation, engine, tank, and environmental data, but it does **not** generate AIS targets (other vessels).

---

## Device and Network Management

> **How to get here:** from the Main Dashboard, tap or click the **N2K** indicator in the header status bar.

### Detected Devices

The Network Manager page shows every physical device currently detected on the NMEA 2000 bus as an individual card. This screen is useful for confirming that the CAN bus is functioning correctly, that expected instruments are visible to the gateway, and for spotting routing conflicts before they affect the dashboard.

Each device card shows:

- **Device name/model** – the reported product name, or "Unknown Device" if not available. A colored bar to the left of the name indicates whether the device is currently online (green) or has gone silent (red, no data for more than 10 seconds).
- **CAN address** – the device's current source address (0–251) on the bus.
- **ID (NAME)** – the device's unique 64-bit NMEA 2000 NAME. Unlike the CAN address, which can change if the bus re-arbitrates, this identifier stays constant for the physical device and is what the gateway uses internally to track preferred-source rules.
- **Serial number** – reported by the device, when available.
- **Last seen** – how many seconds have passed since the device's last message.
- **Actively Transmitting** – a list of the specific data streams (e.g. Position, Depth, Engine data) this device is currently broadcasting. A device that is only listening on the bus without transmitting shows as "Silent (Listening only)".
- **Preferred Source For** – if the operator has locked this device as the preferred source for one or more data streams (see the Virtual Patch Panel below), those streams are listed here with a `[ LOCKED ]` tag.

A **Refresh Network** button re-queries the bus and rebuilds the device list and patch panel on demand.

### Virtual Patch Panel and Source Arbitration

The Virtual Patch Panel lets the operator control which physical device supplies each *global* NMEA 2000 data stream (for example: position, heading, true or apparent wind, depth, speed through water). Only streams detected as currently active on the bus, or already configured with a preferred source, are listed.

> Engine, Tank, and DC Source readings are **not** managed here — they are routed individually per display slot on the [Hardware Routing and Labels](#hardware-routing-and-labels) page in System Configuration instead.

Each stream can be set to one of two modes, selected from a dropdown:

- **[ AUTO ] Automatic Arbitration** – the gateway accepts an update from **any** device currently broadcasting that stream, with no preference given to whichever device happened to transmit first. If only one device provides a given stream, this works perfectly and needs no attention. However, if **two or more devices broadcast the same stream at the same time** (for example, two GPS receivers both reporting position), the gateway has no way to tell which one is "correct" — each new message simply overwrites the previous one, so the displayed value can flicker or jump erratically as messages from the different devices interleave. This exact scenario can be reproduced deliberately using [Demo Mode](#n2k-generator-mode-demo), which intentionally simulates two competing GPS sources.
- **Lock to: `<device>`** – the operator manually pins one specific physical device as the sole authoritative source for that stream, resolving the ambiguity above.

**How failover and recovery work (locked streams only):** the sticky, self-healing behavior described below applies only once a stream has been manually locked to a device — it does **not** apply while a stream is left in AUTO mode. When a stream is locked, the gateway remembers the chosen device by its unique NAME rather than its CAN address, so the lock survives address changes caused by bus re-arbitration. If the locked device stops transmitting that stream, the gateway keeps rejecting data from other devices for a grace period (approximately 1.5× the normal timeout for that data type) before accepting a backup source, in order to ride out brief dropouts without an unnecessary source switch. As soon as the locked device resumes transmitting, the gateway automatically switches back to it, even if a backup device took over in the meantime. If a locked device disappears from the bus entirely, its entry is shown in the dropdown as "Offline" until it reappears or the operator selects a different source.

This is why explicit source locking, rather than leaving a stream in AUTO, is recommended whenever multiple sensors can provide the same kind of data, such as:

- alternative GPS sources
- multiple wind instruments
- redundant heading or depth sensors

Locking a source is the only way to guarantee a stable, non-flickering reading and to benefit from the automatic failover and recovery behavior described above; AUTO mode should be reserved for streams where only a single device is expected to ever transmit that data.

---

## Output Streams and Data Distribution

Each output protocol has both a general on/off switch and TCP/UDP/port settings on the main System Configuration page (see [Services and Routing](#services-and-routing)), and its own dedicated quick-access page, reachable from the header status bar, where the stream can be enabled and its output content fine-tuned.

### Signal K Stream

The Signal K stream is one of the main distribution channels in the system. It exposes live vessel data to marine software that understands Signal K, typically through WebSocket connectivity on port 80. This stream is commonly used by chartplotting and monitoring software running on onboard computers or tablets.

The dedicated Signal K page (opened from the **SK** header indicator) provides:

- **Enable Signal K server stream** – turns the WebSocket delta stream on or off.
- **Data Transmission Mask** – a set of individual checkboxes controlling exactly which NMEA 2000 parameter groups are converted and published to Signal K, including heading, GPS position, SOG, COG, STW, depth, apparent and true wind, water and air temperature, atmospheric pressure, humidity, rudder angle, distance log, magnetic variation, system date & time, GPS altitude and signal quality, vessel attitude (pitch/roll), all four tank types, both engines, DC power sources, and AIS targets.

Disabling parameters that are not needed by connected Signal K clients reduces unnecessary network and processing load.

> **Known limitation:** as an evolving open-source project, authentication tokens are not yet implemented, so the Signal K stream is currently unsecured for any client on the local network (see also the [Signal K Token](#signal-k-and-identity-settings) field).

AIS targets are fully translated to Signal K deltas, each addressed to its own vessel context (`vessels.urn:mrn:imo:mmsi:<MMSI>`). Depending on what has changed and is due for an update, a target's delta may include dynamic values (position, speed over ground, course over ground) and/or static values (name, MMSI, VHF call sign, length, beam, draft, AIS class A/B, and ship type or Aid-to-Navigation type). AIS targets are subject to the same zone-based throttling described in [AIS Settings](#ais-settings), so update frequency depends on a target's distance from the vessel.

### NMEA 0183 Output

The NMEA 0183 interface publishes ASCII sentence data over the network. NMEA 0183 output is commonly used with older chartplotters and marine software that still relies on sentence-based communication.

The dedicated NMEA 0183 page (opened from the **0183** header indicator) provides:

- **Enable NMEA 0183 output** – turns the sentence stream on or off. Port number and TCP/UDP transport mode are configured on the main System Configuration page.
- **Output Sentences Filter** – individual toggles for each NMEA 0183 sentence type the gateway can generate from the NMEA 2000 bus, including GPRMC/GPGGA (position & fix), GPVTG (SOG/COG), GPZDA (UTC date & time), GPHDG (magnetic heading), WIMWV apparent and true wind, SDDPT (depth), VWVHW (water speed & heading), ERRSA (rudder angle), YXXDR (pitch & roll attitude), VWRPM (engine/shaft RPM), YXMTW (water temperature), VWVLW (distance log/trip), and !AIVDM (AIS targets).

This allows the operator to send only the sentences a specific piece of legacy software expects, avoiding compatibility issues with parsers that misbehave on unrecognized sentence types.

### Actisense Binary Output

Actisense Binary output provides a raw binary feed compatible with software that expects Actisense-like packet formats (for example, OpenCPN in NGT-1 mode, or Coastal Explorer). It transmits full NMEA 2000 frames without converting them to NMEA 0183, and is the recommended format for professional navigation software.

The dedicated Actisense Binary page (opened from the **aBin** header indicator) provides a single **Enable Actisense Binary stream** switch. Port number and TCP/UDP transport mode are configured on the main System Configuration page.

---

## Connecting Navigation Software

The gateway exposes the same underlying vessel data through three parallel protocols (see [Output Streams and Data Distribution](#output-streams-and-data-distribution)), so almost any marine navigation app can connect — the correct choice mainly depends on which protocol that particular app supports, and how much of the data it needs.

### OpenCPN

OpenCPN is the most flexible option, since it can connect using **any** of the gateway's three output protocols. In OpenCPN, go to **Options → Connections → Add Connection**, choose **Network** as the connection type, and set the **Address** field to the gateway's IP address or mDNS hostname. Which protocol and port to select depends on what is needed:

- **Signal K** – protocol: Signal K, port `80`. Recommended if other Signal K tools are also in use, or for a modern JSON-based integration.
- **NMEA 2000 / Actisense format** – protocol matching the gateway's Actisense Binary output, port `10120` (TCP or UDP, matching the setting on the [Actisense Binary](#actisense-binary-output) page). This carries full, unconverted NMEA 2000 data and is the **recommended option for the richest dataset** — including AIS, engine, tank, and DC data that may not be exposed through a limited set of NMEA 0183 sentences.
- **NMEA 0183** – protocol: NMEA 0183, port `10110` (TCP or UDP). The simplest option, sufficient for basic position, heading, and AIS display, but limited to whichever sentences are enabled in the [NMEA 0183 Output Sentences Filter](#nmea-0183-output).

Make sure the corresponding stream is enabled on the gateway (via the header status indicators or System Configuration) before adding the connection in OpenCPN.

### Navionics (Boating App)

The Navionics Boating app connects to external instruments over **NMEA 0183 only** — it does not support Signal K or native NMEA 2000/Actisense data. In the app, open the menu and go to **Paired Devices → Add Device**, then enter:

- **Host** – the gateway's IP address.
- **Port** – `10110` (the gateway's NMEA 0183 port).
- **Protocol** – TCP or UDP; either can work depending on the network and Navionics app version, so it is worth trying both if one does not connect reliably.

Navionics has historically only recognized a limited subset of NMEA 0183 sentences — primarily position/fix (GPRMC/GPGGA), depth (SDDPT), and AIS (!AIVDM/VDO). To ensure reliable compatibility, make sure at least those sentence types are enabled in the gateway's [NMEA 0183 Output Sentences Filter](#nmea-0183-output); enabling additional, unrecognized sentence types generally does no harm, but is unlikely to be used by the app.

### Boat Instrument (Signal K app)

Boat Instrument is a dedicated Signal K client app (available for Android) that displays live data in fully configurable boxes/widgets on one or more pages. It connects directly to the gateway's Signal K server — enter the gateway's IP address or hostname and port `80` when adding the server in the app.

Boat Instrument uses two different parts of the Signal K interface for two different purposes:

- The **Signal K HTTP API** is used only once, when configuring a box, to list the data paths currently available from the gateway (for example, to let the operator pick "Wind: True Speed" from a list rather than typing a raw path manually).
- Live values shown on screen are then delivered continuously via the **Signal K delta** stream over WebSocket — the same efficient, subscription-based update mechanism used internally by the gateway's own dashboard — rather than by repeatedly polling the API.

Each box on each page must be configured individually to display a specific data path, so a bit of initial setup is required before the app shows the desired instruments.

### Other Compatible Software

Because the gateway speaks standard, widely supported marine protocols, most other navigation and instrument apps that accept NMEA 0183 over TCP/UDP, or that support Signal K, should also work, including Aqua Map and WilhelmSK. Configuration generally follows the same pattern as above: enter the gateway's IP address and the port of the desired protocol (see [Output Streams and Data Distribution](#output-streams-and-data-distribution) for the exact ports), and enable the matching stream on the gateway if it is not already active.

---

## Diagnostics and Monitoring

> **How to get here:** from the Main Dashboard, tap or click the **STATUS** button in the footer.

The diagnostics interface provides a live status view of the device and its internal operations. It is useful for troubleshooting, validation, and routine operational checks.

### General Diagnostics

The General Diagnostics card displays the live health of the gateway, refreshed automatically every 2 seconds:

- **System Uptime** – time elapsed since the last boot.
- **CAN Bus Load** – physical bus utilization, calculated against the 250 kbps NMEA 2000 bandwidth. The value turns yellow above 50% and red above 70%.
- **Raw CAN Frames (RX / TX)** – incoming and outgoing frame rates, in frames per second.
- **Processed PGNs** – the rate of NMEA 2000 Parameter Group Numbers successfully parsed, in messages per second.
- **Lost N2K Frames** – a running count of frames the controller failed to receive or process. This value is highlighted red whenever it is greater than zero.
- **GPS Time Sync** – shows `SYNCHRONIZED` (green) once the system clock has been set from GPS data, or `WAITING` (inactive) until then.
- **Active WiFi Clients (AP)** – number of devices currently connected when the gateway is in Router (Access Point) mode.
- **CPU Temperature** – current controller die temperature.
- **Active Service Clients** – a breakdown of how many clients are currently connected to each output service: Actisense Binary, Signal K, NMEA 0183, and the Web GUI.

These values help the operator monitor whether the system is operating normally or whether there are dropouts, bus overloads, or communication issues.

### Memory and RTOS Diagnostics

The RAM Memory (Heap) card shows three separate memory pools:

- **Global heap** – Free Memory, Lowest Level ever recorded since boot (Global Min), and the largest single Max Contiguous Block currently available.
- **Internal SRAM** (used for system and WiFi operation) – Free Memory and Max Contiguous Block.
- **External PSRAM** (used for data buffers) – Free Memory and Max Contiguous Block.

Consistently low free memory, a shrinking Global Min value, or a Max Contiguous Block much smaller than the total free memory (a sign of fragmentation) can indicate resource pressure or an unstable configuration.

The RTOS Tasks card lists every active FreeRTOS task with its **Priority**, **CPU** usage percentage, and **Free Stack** (the minimum stack headroom recorded for that task, in bytes). CPU usage is highlighted yellow above 10% and red above 25%; Free Stack is highlighted red and bold below 512 bytes, since a task running low on stack space is at risk of a crash.

### Execution Time Statistics

The Execution Times card shows approximate processing time for each internal module, measured in microseconds:

- **ACET (Avg)** – the average execution time.
- **WCET (Max)** – the worst-case (maximum) execution time observed, highlighted yellow above 50,000 µs (50 ms).

Monitored modules include the Memory Snapshot routine, the Signal K Generator, the Web UI Serializer, the NMEA 0183 Formatter, the N2K Parser, the Central Data Sync step, and the AIS Output Generator. This data is important when tuning the system for performance, especially when many CAN packets or AIS targets are active.

---

## Status and Configuration Monitor

> **How to get here:** from the Main Dashboard, tap or click the **GUI** indicator in the header status bar.

The Status & Config Monitor is a raw diagnostic view into the exact data structure that drives the entire dashboard. It connects directly to the same internal WebSocket feed (`/wsUI`) used by the Main Dashboard, but instead of rendering instruments, it displays the underlying JSON. This page is intended for advanced troubleshooting, firmware development, and for validating that the device is receiving, parsing, and distributing the expected data — including fields that are not otherwise shown anywhere in the normal user interface.

A status indicator in the header shows **RTOS Live** (connected) or **Disconnected**; the page automatically attempts to reconnect every 2 seconds if the connection drops.

The page presents three separate panels:

- **Current RAM Structure (yachtData)** – the complete, continuously updated in-memory data tree (`yachtData`) held by the browser, built by incrementally deep-merging every incoming delta packet on top of the previous state. This reflects the single source of truth used to drive every instrument, list, and indicator on the dashboard, including navigation and environmental data, engine/tank/DC arrays, AIS targets, and all service status flags (`n2k`, `sk`, `n0183`, `aBin`, `wifi`, `gps`, `demo`, and `aisStatus`).
- **Active Configuration (systemConfig)** – custom labels (such as renamed tanks or engines) detected on the fly whenever a delta packet containing a `labels` field arrives.
- **Last Received Delta (Live Packet)** – the single most recent raw JSON packet received from the gateway, before merging. This is the most useful panel for confirming, in real time, exactly what a given sensor update looks like on the wire.

This screen is particularly useful when:

- inspecting the exact raw field names and data types produced by the firmware
- checking the current live configuration and custom labels
- verifying source routing
- validating a network change
- confirming that a client is receiving data
- inspecting the system state after a reboot

---

## Troubleshooting

If the gateway does not behave as expected, check the following items in order:

1. Verify the WiFi operating mode and network connectivity.
2. Confirm that the device is connected to the correct NMEA 2000 bus and that the CAN bus is healthy.
3. Ensure that the configured service streams are enabled.
4. Review detected devices and preferred source mappings.
5. Check AIS settings if targets are not appearing or the update rate is poor.
6. Inspect the diagnostics page for lost packets, low memory, or abnormal CPU behavior.
7. Confirm that client applications are listening on the expected TCP or UDP ports.

Common issues include:

- no connectivity to the web interface
- no data on the dashboard
- missing AIS target updates
- the live map not loading, or falling back to the offline instrument view, when the gateway or client device has no internet access
- clients not receiving Signal K or NMEA 0183 packets
- duplicate or incorrect source selection
- unstable WiFi after configuration changes

When troubleshooting, it is recommended to reset only one variable at a time and verify the effect of each change.

---

## Maintenance and Best Practices

To keep the system reliable over time:

- regularly check the diagnostics page
- verify that WiFi signal quality is sufficient
- inspect CAN wiring for continuity, corrosion, and secure connections
- confirm that the gateway remains dry and ventilated
- keep the firmware updated according to the manufacturer’s instructions
- validate data output after any configuration change
- document any custom source routing or network settings

In marine environments, power quality, RF interference, and vibration can affect reliability. The gateway should be mounted in a stable, protected location and not exposed to direct moisture or heat buildup.

---

## Safety and Operational Disclaimer

This device is intended as a data acquisition and distribution tool for marine applications. It is not a certified navigational instrument and should not be treated as a standalone replacement for approved marine electronics.

Important safety notes:

- Always keep a human operator monitoring navigation and vessel conditions.
- Do not rely solely on the gateway for navigation decisions.
- Verify the system during installation and before operation at sea.
- Use the gateway as a supporting tool, not as the sole source of navigational safety.
- Treat raw sensor values and protocol streams as advisory information unless verified by approved equipment.

The gateway provides useful operational insight, but marine navigation remains the responsibility of the vessel operator and crew.

### License and Legal Notice

This is a community-driven, open-source project, provided as freeware for personal, non-commercial use on private vessels. It is **not** a certified marine instrument and is **not** intended for critical safety or primary navigation tasks. All software and hardware information is shared "AS IS," without warranties of any kind.

By downloading, installing, or using this firmware, the operator acknowledges and accepts that:

- use is entirely at the operator's own risk;
- the authors and contributors assume no liability for any direct or indirect damages, navigation errors, vessel grounding, equipment damage, or data loss arising from the use of this firmware or hardware;
- maritime navigation requires certified equipment and constant human oversight.

Refer to the project's `LICENSE.md` file for the full End-User License Agreement.

### Acknowledgments and Open-Source Libraries

This project is built on the work of the open-source marine electronics community. In particular, it relies on the following libraries for NMEA 2000 and NMEA 0183 protocol handling:

- Timo Lappalainen – [NMEA2000 Library](https://github.com/ttlappalainen/NMEA2000)
- Timo Lappalainen – [NMEA2000 ESP32 Driver](https://github.com/ttlappalainen/NMEA2000_esp32)
- Jiauka – [NMEA2000 ESP32xx Driver](https://github.com/jiauka/NMEA2000_esp32xx)
- Timo Lappalainen – [NMEA0183 Library](https://github.com/ttlappalainen/NMEA0183)
- Wellenvogel – [NMEA2000-Gateway with ESP32](https://github.com/wellenvogel/esp32-nmea2000)

---

## Appendix: Supported NMEA 2000 PGNs and Data Points

This appendix lists every NMEA 2000 Parameter Group Number (PGN) the gateway's decoder actively parses from the CAN bus, and exactly which internal data point each one populates. Every value listed here also passes through the [source arbitration](#virtual-patch-panel-and-source-arbitration) logic, so it can be locked to a specific physical device if desired.

### Navigation and Environment

| PGN | Description | Data Populated | Notes |
|---|---|---|---|
| 127250 | Vessel Heading | Heading (and Magnetic Variation, if broadcast alongside a magnetic reference) | Smoothed per the Heading Damping setting |
| 127258 | Magnetic Variation | Magnetic Variation | |
| 128259 | Speed, Water Referenced | Speed Through Water (STW) | Smoothed per the Speed Damping setting |
| 128267 | Water Depth | Depth Below Transducer | Includes the configured transducer offset |
| 129025 | GNSS Position, Rapid Update | Latitude / Longitude | Primary, high-frequency (~10 Hz) position source |
| 129026 | COG & SOG, Rapid Update | Course Over Ground (COG), Speed Over Ground (SOG) | Smoothed per the Heading/Speed Damping settings |
| 129029 | GNSS Position Data | Altitude, GPS Time & Date, Satellite Count, HDOP, GNSS Fix Method | Latitude/Longitude from this slower (~1 Hz) frame are only used as a fallback if PGN 129025 has not been received in the last second |
| 130306 | Wind Data | Apparent Wind (AWS/AWA), True Wind (TWS/TWA/TWD) | If the source reports apparent wind, true wind is calculated internally from apparent wind, boat speed, and heading; if the source reports true wind directly, that value is used and smoothed instead |
| 128275 | Distance Log | Log, Trip Log | |
| 127245 | Rudder | Rudder Position | |
| 127257 | Attitude | Pitch, Roll | |
| 130310 | Water Temperature, Outside Air Temperature and Atmospheric Pressure (legacy) | Water Temperature | Legacy combined frame; only used as a fallback when no newer-standard (130312/130316) temperature source is available |
| 130311 | Environmental Parameters (legacy) | Water Temperature, Outside Air Temperature, Atmospheric Pressure, Humidity | Legacy combined frame; each parameter is independently arbitrated |
| 130312 / 130316 | Temperature / Temperature, Extended Range | Water Temperature or Outside Air Temperature | Selected by the reported temperature source field |
| 130313 | Humidity | *(reserved — parsing not yet implemented)* | |
| 130314 | Actual Pressure | Atmospheric Pressure | Only the atmospheric pressure source type is used |

### Engines, Tanks, and DC Sources

| PGN | Description | Data Populated | Notes |
|---|---|---|---|
| 127488 | Engine Parameters, Rapid Update | Engine RPM | ~10 Hz update rate |
| 127489 | Engine Parameters, Dynamic | Engine Coolant Temperature, Alternator Voltage, Fuel Rate, Engine Hours, engine status/alarm flags | ~1 Hz update rate |
| 127505 | Fluid Level | Fuel, Fresh Water, Gray Water, or Black Water tank Level and Capacity | Tank type is read from the message itself; other fluid types (e.g. oil, live well) are ignored |
| 127508 | Battery Status | DC Source Voltage, Current, Temperature | |
| 127506 | DC Detailed Status | DC Source State of Charge, State of Health, Time Remaining, Capacity, DC type | |

Engine, tank, and DC source instances are dynamically bound to internal display slots the first time a given NMEA 2000 instance number is seen, and can be manually pinned to a specific slot on the [Hardware Routing and Labels](#hardware-routing-and-labels) page.

### System and Device Identity

| PGN | Description | Purpose |
|---|---|---|
| 60928 | ISO Address Claim | Identifies each device's unique 64-bit NAME, used to populate the [Detected Devices](#detected-devices) list and to keep source-lock rules valid across bus address changes |
| 126996 | Product Information | Supplies the model, serial number, and software version shown on each device card |

### AIS Targets

| PGN | Description |
|---|---|
| 129038 | AIS Class A Position Report |
| 129039 | AIS Class B Position Report |
| 129040 | AIS Class B Extended Position Report |
| 129794 | AIS Class A Static and Voyage Related Data |
| 129809 | AIS Class B "CS" Static Data Report, Part A |
| 129810 | AIS Class B "CS" Static Data Report, Part B |
| 129041 | AIS Aid to Navigation (AtoN) Report |

AIS frames are read into a dedicated buffer as a priority, ahead of other processing, to avoid missing fast-moving traffic in busy waters.

---

## Glossary

- AIS: Automatic Identification System used to track nearby vessels.
- CAN bus: Controller Area Network used in NMEA 2000 marine systems.
- PGN: Parameter Group Number used in NMEA 2000 data frames.
- Signal K: JSON-based marine data standard used by modern marine software.
- NMEA 0183: Legacy text-based marine protocol.
- Actisense Binary: Binary protocol used for compatibility with specific chartplotter and software ecosystems.
- WiFi AP mode: Access point mode in which the gateway creates its own local network.
- WiFi client mode: Mode in which the gateway connects to an existing local network.
- mDNS: Multicast DNS, used to discover devices by name on a local network.

---

This manual reflects the current functionality of the gateway as implemented in the built-in web interface, including configuration, dashboard monitoring, network management, diagnostics, and protocol output control.
