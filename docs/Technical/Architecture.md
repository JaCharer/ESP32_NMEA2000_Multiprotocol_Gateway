# ESP32 NMEA 2000 Gateway: Architecture & Data Flow

This document provides a high-level "helicopter view" of the gateway's architecture. It focuses on the functional design and the continuous lifecycle of data as it travels from physical sensors on the yacht's network to the screens of end-user devices.

---

## 1. The Core Philosophy (Asymmetric Processing)

To guarantee that the gateway never misses a critical message (like a sudden depth drop or a fast AIS target), the system divides its "brain" into two completely independent workers:
*   **The Hardware Worker (Core 1):** Dedicated exclusively to listening to the physical NMEA 2000 bus and decoding data as fast as it arrives.
*   **The Network Worker (Core 0):** Dedicated to handling Wi-Fi clients, translating data into various marine protocols, and serving the Web Dashboard.

This strict separation ensures that a bad Wi-Fi connection or a heavy data download will never slow down the real-time processing of the yacht's instruments.

---

## 2. The Data Pipeline (End-to-End Flow)

Data flows through the system in a strictly controlled, 5-step pipeline:

### Step 1: Physical Ingestion (Sensor -> Gateway)
The gateway sits on the NMEA 2000 CAN bus, listening to everything broadcasted by the yacht's sensors (GPS, Wind, Engines, Tanks). Raw binary frames are captured by the hardware driver and queued for immediate processing.

### Step 2: Decoding & Arbitration (The Traffic Cop)
Before data is accepted, the gateway decodes the raw frame and passes it through an **Arbitration Engine**. 
*   If the yacht has multiple sensors of the same type (e.g., two depth sounders), the engine checks user-defined routing rules to see which sensor is the designated "Primary" (The King). 
*   If the data comes from the Primary sensor, it is allowed through. If it comes from a secondary sensor, it is silently dropped—unless the Primary sensor has failed, in which case the gateway automatically accepts the backup data.

### Step 3: Signal Conditioning (Smoothing)
Marine sensors can be "noisy" (e.g., a wind vane bouncing in waves). Critical, fast-updating metrics like Wind Angle, Heading, and Boat Speed pass through an internal smoothing filter (Exponential Moving Average). This ensures that the data sent to the displays results in smooth, readable needle movements rather than erratic jumping.

### Step 4: The Central Hub & Snapshot (Core 1 -> Core 0)
Once data is decoded, filtered, and approved, it is stored in a central "Yacht State" hub. 
Because the Network Worker (Core 0) operates at a different speed than the Hardware Worker (Core 1), the system uses a **Snapshot Mechanism**. Periodically, the Network Worker takes a lightning-fast "photograph" of the entire yacht's state. This allows the Network Worker to take its time packaging the data for Wi-Fi transmission without forcing the Hardware Worker to pause and wait.

### Step 5: Protocol Dispatch (Gateway -> World)
Using the snapshot, the **Network Dispatcher** translates the data into four distinct languages simultaneously, broadcasting them over the Wi-Fi network:
*   **Web UI (WebSockets):** A highly compressed, real-time JSON stream sent directly to browsers for the built-in dashboard.
*   **Signal K:** The modern, open-source REST/WebSocket format, allowing advanced marine software to consume the data.
*   **NMEA 0183 (TCP/UDP):** Legacy ASCII sentences for older chartplotters and standard navigation apps like Navionics or OpenCPN.
*   **Actisense / Raw N2K (TCP/UDP):** The raw binary stream, passed through untouched, for diagnostic tools or specialized software.

---

## 3. Key Functional Features

Beyond simply passing data, the gateway actively manages the network environment:

*   **Active Network Mapping:** The system constantly builds a live "map" of all connected physical devices. It asks new devices for their Manufacturer and Model info, and quietly removes devices from its routing tables if they are unplugged or lose power.
*   **Intelligent AIS Management:** AIS traffic can overwhelm a wireless network. The gateway intercepts AIS data, separates static ship info (names, call signs) from dynamic info (speed, position), and enforces strict capacity limits. If the network gets crowded, it prioritizes nearby ships and automatically drops data from distant vessels.
*   **Fail-Safe Browser Dashboard:** The user interface relies entirely on a live data stream. If the stream stops (e.g., due to a severed CAN cable or router failure), the dashboard's "Ruthless Watchdog" detects the silence, turns the instruments red to warn the skipper, and safely attempts to reconnect.