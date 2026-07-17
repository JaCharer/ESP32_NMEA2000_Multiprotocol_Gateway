# N2K Data Decoder & Arbitration Engine

This document outlines the architecture and internal workings of the N2K Data Decoder. The system is designed for high-performance, real-time NMEA 2000 data processing on an ESP32 microcontroller, utilizing a dual-core architecture, lock-free micro-caching, and dynamic arbitration.

---

## 1. Core Architecture & Arbitration (The "King" Concept)

In a typical NMEA 2000 network, multiple devices might broadcast the same data (e.g., two GPS antennas or two wind sensors). The gateway must decide which data to process and which to ignore. This is handled by the **Arbitration Engine**.

### The "King" (Preferred Source)
* **Definition:** The "King" is the specific device (identified by its unique 64-bit `NAME`) that the user has selected in the Web UI as the primary source for a specific data stream (e.g., `DataType::WIND_APPARENT`).
* **The Rule:** If the King is actively transmitting, all data from competing devices for that specific stream is instantly dropped.
* **Failover (The Fallback):** If the King stops transmitting and exceeds the predefined `DATA_TIMEOUT`, the system automatically accepts data from the next available device (a "Candidate").
* **Reclaiming the Throne:** If the King comes back online, the system instantly detects the King's CAN address and overrides the Candidate, giving the throne back to the King.
* **Auto Mode:** If no King is defined for a stream, the first device that broadcasts the data claims the stream until it times out.

---

## 2. Handling Instances (Engines, Tanks, Batteries)

Some data types, like fluid levels or engine parameters, utilize **Instances** to differentiate between multiple identical devices (e.g., Port Engine vs. Starboard Engine). 

The decoder uses a **Virtual Slot Mapping** system (`GetOrBindSlot`):
1. **Hard Binding:** If the user explicitly assigned a specific N2K Instance (e.g., Instance `1`) to a specific Web UI/Signal K slot (e.g., `Tank 2`) and saved it in the configuration, the system strictly follows this map.
2. **Soft Binding (Auto):** If the slot map is empty (`255`), the system dynamically binds the first incoming instance to the first available internal slot. 
3. **Stream Keys:** Arbitration rules are tracked using a 16-bit `streamKey`, which combines the `DataType` (upper 8 bits) and the `Instance` (lower 8 bits). Global data (like Wind or Position) uses `255` as the instance byte.

---

## 3. The Lock-Free Micro-Cache (High-Performance Hot Path)

The most critical part of the decoder is the `IsPreferredSource()` function, which is called hundreds of times per second on **Core 1** (the CAN reader core). 

To prevent multi-threading crashes (such as `LoadProhibited` errors) when **Core 0** (Web UI / Garbage Collector) modifies routing rules, the system uses an ultra-fast, pre-allocated **Micro-Cache**.

### How it Works:
* **No Dynamic Memory:** It completely avoids heavy C++ containers like `std::unordered_map`. Instead, it uses a flat array (`tRouteCacheEntry`) limited to 64 active streams. This takes less than 1 KB of static RAM.
* **Hardware Spinlocks:** Instead of FreeRTOS Mutexes (which can put the core to sleep and cause dropped CAN frames), it uses `portMUX_TYPE` (spinlocks). These pause hardware interrupts for a fraction of a microsecond—just long enough to read or write the array.
* **Torn Read Protection:** When the user updates the configuration on Core 0, the decoder safely copies the new rules into a local snapshot (`localPrefSources`) using `memcpy` before processing. This guarantees that Core 1 never reads a partially updated 64-bit device `NAME`.

---

## 4. AIS Processing Engine

The Automatic Identification System (AIS) generates a massive amount of data. Processing it synchronously could block the main N2K decoding loop.

### Buffered Processing
When an AIS PGN (e.g., 129038, 129039) arrives, the decoder does not parse it immediately. Instead, it pushes the raw `tN2kMsg` into a lightweight `aisRawBuffer` (capped at 20 frames to protect RAM). The buffer is safely flushed and parsed sequentially during the `Update()` phase.

### Capacity Management & Eviction (`EnsureAISTargetCapacity`)
The gateway stores a maximum number of AIS targets (e.g., 300) to prevent memory exhaustion. If a new ship appears and the database is full, the system executes an eviction algorithm:
1. **Invalid Targets First:** Ships missing critical coordinate data are immediately deleted.
2. **Distance Penalty:** The system calculates a pseudo-distance (squared) between the user's vessel and the AIS targets.
3. **Timeout Penalty:** Ships that haven't sent dynamic data in the last 60 seconds have their distance artificially multiplied.
4. **Eviction:** The furthest/quietest ship is permanently deleted to make room for the new target.

### Independent Garbage Collector
A dedicated AIS Garbage Collector runs every 10 seconds. It utilizes a zero-wait try-lock (`xSemaphoreTake(aisMutex, 0)`). If the Web UI is currently reading the AIS table, the GC simply aborts and tries again later. When successful, it sweeps the database and removes any ship that has been completely silent for over 5 minutes.

## 5. Data Parsing & Smoothing (EMA Filter)

Raw PGNs are intercepted by dedicated handlers (e.g., `HandleWind`, `HandleHeading`). Before being stored, rapidly changing metrics—such as wind angle, boat speed, and heading—pass through a custom Exponential Moving Average (EMA) filter via the `SmoothScalar` and `SmoothAngle` functions. 

The damping level (alpha) is dynamically applied based on user preferences (levels 0-10 mapped via the `DAMPING_ALPHA` array). This mathematical smoothing ensures UI stability and prevents needle jitter on displays without sacrificing overall system responsiveness.

## 6. Central Storage & Synchronization

Data is not written directly to public structures that could be accessed concurrently by network protocols. Instead, the decoder follows a localized buffering strategy:
1. **Internal Buffer:** Parsed and smoothed data lands in a local, protected `internalBuf` (of type `tYachtData`).
2. **Update Cycle:** During the main `Update()` loop, the system compares the values in `internalBuf` with the globally accessible `YachtData` structure.
3. **Dirty Flags:** If a value has changed, it updates the global structure and flips specific bits in a `tDataGroupMask` union (`detectedChanges.flag`). These flags instantly notify the Web UI, Signal K, and NMEA 0183 dispatchers that fresh data is ready to be sent over the network.
4. **Timeouts & Data Expiration:** A tracking mechanism (`LastUpdate` arrays) monitors the exact age of every data point. If a sensor drops offline and exceeds its specific `DATA_TIMEOUTS` threshold, the `InvalidateData()` function automatically clears those specific fields (setting them to `N2kDoubleNA`) to prevent the gateway from broadcasting stale or "frozen" information[cite: 1].

## 7. Device Discovery & "Ghosts"

In a dynamic NMEA 2000 network, devices can be plugged in while the gateway is already running. If the gateway receives data from a CAN address that is not currently in the `knownDevices` map (meaning the gateway missed its initial Address Claim), it creates a temporary "Ghost" device. 

The system assigns this Ghost a synthetic 64-bit NAME (using a `0xFFFFFFFFFF000000ULL` mask combined with its CAN address) and immediately allows its data to flow into the arbitration engine. Simultaneously, the gateway transmits an ISO Request (PGN 60928) targeted at that address, demanding the mysterious device to identify itself. Once the true NAME is received, the Ghost is exorcised (erased) and replaced by the verified device record, seamlessly rebuilding the routing cache.

## 8. CAN Network Topology & Device Management

The gateway doesn't just passively read data; it actively builds and maintains a live "image" of the physical NMEA 2000 network. This device list (`knownDevices` map) is the foundation of the Web UI's routing matrix and the arbitration engine.

### Building the Network Image
1. **ISO Address Claim (PGN 60928):** When a device connects to the network or boots up, it broadcasts its address claim. The gateway intercepts this via `HandleAddressClaim()` and extracts the device's unique 64-bit `NAME`, Manufacturer Code, Device Function, and Device Class.
2. **Product Information (PGN 126996):** If a newly discovered device lacks detailed metadata, the gateway proactively sends an ISO Request asking for its Product Information. The `HandleProductInfo()` function parses the response to extract the Model ID, Software Version, Serial Number, and Load Equivalency, creating a complete profile of the sensor.

### Network Garbage Collection (The "Sweeper")
To ensure the network image reflects reality (e.g., when a user physically unplugs a chartplotter or a sensor loses power), the gateway employs a background Garbage Collector. 
* **The Process:** Every 10 seconds, the system scans the `LastSeen` timestamp of every known device under the protection of a mutex (`deviceListMux`).
* **Eviction:** If a device has been completely silent for over 3 minutes (180000 ms), it is permanently deleted from the active network map.
* **Auto-Recovery:** If the deleted device was currently acting as a "King" (preferred source) in the arbitration cache, the system immediately triggers `RebuildCaches()`[cite: 1]. This safely flushes the micro-cache and forces the system to automatically fall back to the next available backup sensor without dropping a single frame.