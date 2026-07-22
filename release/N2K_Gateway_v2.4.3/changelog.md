# Changelog v2.4.3
### Improvements
* **Memory Protection Refactoring (Lock-Free & RAII):** Replaced FreeRTOS spinlocks with modern C++ std::mutex and std::lock_guard for the CAN device list. This ensures memory safety, prevents deadlocks, and allows safe logging directly from within critical sections.
* **N2K Decoder Optimization:** Reduced CPU load by implementing an O(1) lookup table for lightning-fast CAN device tracking and forcing hardware FPU usage (float math) for trigonometry. Additionally, redundant millis() system calls were eliminated by capturing the timestamp only once per frame cycle.
* **Decoder Queue Draining (Zero Dropped Frames):** Implemented an aggressive queue draining mechanism in the N2K Decoder task to process incoming data bursts without artificial delays. This eliminates queue overflows, ensuring minimal dropped messages even under extreme stress-testing with multiple concurrent NMEA2000 data streams.
* **Zero-Cost Logging (Arduino Environment):** Implemented lazy evaluation for the asynchronous logger by moving the log-level checks directly into the preprocessor macros. This ensures that muted logs consume absolutely zero CPU cycles and stack memory, significantly improving performance during heavy CAN bus traffic.
* **TCP Keep-Alive for Data Streams:**Implemented hardware TCP Keep-Alive (10s idle, 3 probes) for both the NMEA0183 and N2K Actisense TCP streams. This enables the network stack to automatically detect and drop unresponsive "zombie" clients, preventing buffer locks and freeing up server slots.
* **TCP Connection Timeout Handling:** Added a timeout callback for all newly connected TCP clients (nmeaClients and n2kClients). This explicitly invokes client->close() upon timeout, ensuring that dead or unresponsive connections are immediately aborted and resources are freed.
* **WiFi Power Management Disabled:** Added WiFi.setSleep(false); for both STA and AP modes immediately after network initialization. This disables WiFi power-saving features to reduce network latency and ensure stable, uninterrupted data streaming.
* **Enhanced WiFi Stability (20MHz Bandwidth):** Restricted the WiFi channel bandwidth from 40MHz down to 20MHz in both AP and STA modes. This drastically reduces susceptibility to RF interference in crowded marina environments and improves overall connection stability, as the high throughput of 40MHz is not required for NMEA telemetry.
* **WebSocket Middleware Architecture:** Completely overhauled connection limits by replacing the unstable "zombie" client workaround with a native ESPAsyncWebServer middleware that safely rejects excess connections at the protocol level before they can consume RAM. To maintain a seamless user experience, the dashboard now performs a lightning-fast pre-flight API check to verify available slots and gracefully display a warning if the server is full.
* **AIS Buffer Optimization:** Transitioned the AIS PGN's parsing  to a flat buffer architecture. Combined with more frequent buffer flushing, this approach minimizes mqueue bottlenecks during high-density target processing.
* **WebSocket Send Pre-Flight Check:** Upgraded JSON payload generation to use defensive C-style memory allocation (malloc wrapped in std::shared_ptr) instead of std::vector, bypassing critical std::bad_alloc exceptions during Wi-Fi glitches. Combined with strict queue capacity validation, this should eliminate vector-related Out-Of-Memory (OOM) panics and shield the system against race conditions caused by congested networks.
* **WebUI AIS Enhancements:** Added a dedicated AIS Class (A/B) indicator badge to the vessel details popup on the dashboard map.

### Bug Fixes
* **AIS/N2K:** Fixed decoder logic bug causing unjustified degradation of Class A vessels (e.g., Inland AIS) to Class B upon receiving static data frames (PGN 129809 / 129810).
* **WebUI/Map:** Resolved DOM rendering issue in the map panel; vessel icons now dynamically update both their color and SVG vector shape on-the-fly when a target's class is corrected or upgraded.

# Changelog v2.4.2
### Bug Fixes
* **WebSocket Stability Fix:** Resolved a critical system hang (indicated by ack timeout errors [AsyncTCP.cpp:1123] _poll(): ack timeout 4) caused by unresponsive Web UI clients. The gateway no longer forces data into full TCP buffers when a connected device goes to sleep or loses Wi-Fi range. Instead, it now uses smart backpressure checks to intelligently drop frames for lagging clients, ensuring the rest of the system operates smoothly without interruption.

### Improvements
* **Overhauled the Wi-Fi:** networking architecture to strictly separate Station (Client) and Access Point (Router) profiles, preventing SSID conflicts and ensuring reliable fallback recovery. The configuration Web UI was also redesigned with smart IP validation and intuitive mode switching.
* **Web Server Stability Fix:** Switched to new, actively maintained forks of the AsyncTCP and ESPAsyncWebServer libraries : mathieucarbou/AsyncTCP and mathieucarbou/ESPAsyncWebServer 

# Changelog v2.4.1

### Bug Fixes
* **N2K Decoder Stability:** Fixed a critical multi-threading race condition (resulting in `LoadProhibited` / Guru Meditation Error crashes) that occurred when the Web UI or background garbage collector modified device routing rules while the CAN bus was actively receiving data.
* **Routing Engine Optimization:** Completely overhauled the preferred source arbitration engine. Heavy, memory-intensive C++ structures (`std::unordered_map`) were replaced with an ultra-fast, pre-allocated micro-cache utilizing hardware spinlocks. This significantly reduces RAM consumption and ensures the CAN decoder never blocks or drops frames, even under heavy network load.
* **Data Integrity Protection:** Resolved a hidden "torn read" issue where 64-bit device names could be read incorrectly by the 32-bit processor if updated mid-cycle. Configuration updates are now safely copied to a local snapshot buffer before processing, guaranteeing 100% thread safety without stalling the main decoder task.


# Changelog v2.4.0
### New Features
* **DC Source Support:** Full support for DC Sources has been added (PGNs 127506, 127508). A dedicated battery/alternator/sloar/generatoe card will automatically appear on the dashboard as soon as the relevant data starts flowing through the network.
* **Smooth Instrument Dials (Damping):** Jumpy readings in rough seas are now a thing of the past. New sliders in the config allow for dynamic adjustments of how quickly the Wind, Heading, and Speed dials react to changes.
* **Network Health Diagnostics:** The Statistics page now displays the true physical load of the NMEA 2000 cable ("Bus Load %"). It automatically turns yellow or red to warn if the boat's network is getting dangerously crowded.
* **Adjustable WiFi Channel:** The WiFi channel can now be manually selected in the advanced WiFi settings. 
* **Emergency WiFi Mode:** If the main boat router fails, the gateway's emergency rescue WiFi will now always appear on a fixed, predictable channel.

### Improvements
* **Multi-Instance Architecture & Labels:** The core architecture has been completely redesigned to fully support and track messages with multiple instances. This provides a much more robust and reliable way of assigning custom names/labels to specific engines, DC sources, and fluid tanks.
* **Remodeled Devices Page:** The network tree page (accessible by clicking the N2K icon) has been completely overhauled. It now features an advanced routing matrix, allowing for the strict, manual assignment of specific physical sensors to dedicated data slots.
* **Smart Speed Fallback:** If the paddlewheel (water speed sensor) gets jammed by algae while the boat is moving, the Wind dial will automatically switch to using GPS speed. The dial's label will turn yellow ("SOG") to clearly indicate a sensor issue.
* **Professional Wind Angles:** The Apparent and True Wind angles now mimic professional marine plotters. Confusing negative numbers have been removed—wind is now displayed strictly from 0° to 180° with a red "P" (Port) or a green "S" (Starboard) suffix.
* **Automatic Device Cleanup:** Disconnected or dead sensors are no longer permanently stuck on the screen. The system will automatically purge them from the device list after 3 minutes of absolute silence.
* **"Unverified" Device Support: Devices that fail to respond to the network introduction request (PGN 60928) are no longer ignored. They will still be displayed as "[UNVERIFIED]" devices, ensuring that specific data slots can be manually linked and routed to them anyway.
* **Cleaner Menus & Settings:** The configuration page is now much easier to navigate. Long lists of engines and tanks, as well as advanced network settings, have been neatly organized into collapsible sections. A quick "Reset" button for hardware assignments was also introduced.
* **Better Statistics View:** The diagnostics page was redesigned to be more compact and easier to read without endless vertical scrolling.

### Bug Fixes
* **Wind Math Fixes:** Fixed mathematical errors that previously caused wind angles to display impossible values (like exceeding 360 degrees) or show visual formatting artifacts.
* **System Stability:** Resolved hidden memory issues that could cause the gateway to unexpectedly freeze or restart when a sensor was suddenly unplugged from the active network.
* **WebSocket Connection Handling:** The method for closing WebSocket connections when the maximum client limit is exceeded has been redesigned. Instead of immediate termination, excess clients are now marked as "zombies" and removed after a short delay. This gives the underlying network library (AsyncTCP) enough time to properly close and clean up the connections in the background, preventing system instability.