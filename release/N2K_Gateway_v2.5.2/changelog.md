# Changelog v2.5.2

### Improvements
**New Hardware Support:** Added a native port for the esp32-s3-touch-lcd-4-can board (Waveshare 4.0" Touch LCD).

**Multiple Build Environments:** Added specific PlatformIO build configurations to generate firmware for various boards (like Wemos, TTGO, and S3 displays), rather than just a single generic build.

**Documentation Updates:** Added a Hardware_profile.md file detailing pinouts and board configs, and a new User Manual in the docs directory.

# Changelog v2.5.1

### Improvements
* **AIS Spatial Filtering & Anti-Thrashing Mechanism:** Overhauled the target admission logic with an FPU-accelerated spatial gatekeeper that evaluates incoming vessels based on real-time physical distance. This completely eliminates cache thrashing (the "revolving door" effect) when the database reaches capacity. While this architecture drastically reduces RAM consumption and CPU overhead on the client device, it introduces a deterministic memory boundary defined by the user-configurable MAX TARGETS IN DATABASE parameter (located in Config/AIS Settings, default: 50). The system strictly prioritizes the immediate spatial threats—ensuring only the absolute closest targets are rendered on the UI and bridged out to downstream NMEA0183 and SignalK multiplexer streams. Once the target limit is saturated, a dynamic triage algorithm seamlessly evicts the most distant vessel from memory to instantly accommodate any newly detected, closer target.
* **WebGL Cartography Engine Upgrade:** Migrated the core mapping framework from Leaflet to MapLibre GL JS. This introduces full GPU-accelerated WebGL rendering, eliminating CPU-bound tile generation bottlenecks. The map now delivers fluid 60fps panning, continuous fractional zooming, and offloads dynamic vector layers (such as the AIS target layer and yacht track) directly to the device's graphics pipeline for massively improved thermal and battery efficiency on mobile clients.
* **Hardware-Accelerated Vector Instruments (SVG):** Architected a complete rewrite of the onboard instrument cluster (Wind, Navigational Compass, and Engine gauges), deprecating legacy HTML5 Canvas in favor of Scalable Vector Graphics (SVG). Gauge animations are now natively handled by the browser's compositor using CSS transform matrices, completely eradicating DOM-blocking JavaScript redraw loops and pixelation on high-DPI (Retina) displays. Additionally, implemented shortest-path angular rotation mathematical logic to ensure flawless, "spin-free" 360-degree rollovers for Heading and COG data.
* **Client-Side Persistent Navigational & AIS State (IndexedDB):** Engineered an asynchronous, browser-native IndexedDB storage layer (YachtDB) to persist critical state directly on the user's viewing device (tablet, smartphone, or PC) across browser sessions or temporary network drops. The web client itself now maintains a local, non-blocking cache of the vessel's track points, active AIS targets, and a comprehensive historical dictionary of AIS static data (Vessel Names, Callsigns, Dimensions). This "Zero-Wait" static resolution instantly populates the UI and map popups with ship details upon boot, entirely bypassing the inherent transmission delays of over-the-air NMEA 2000 static broadcast cycles while keeping the gateway hardware's memory footprint strictly optimized.


# Changelog v2.5.0

### Improvements
* **Web Server Framework Overhaul:** Completely replaced the `AsyncTCP` and `ESPAsyncWebServer` libraries with the native ESP networking framework. This major architectural shift reduces memory fragmentation, eliminates deep-rooted asynchronous socket crashes, and provides much better native integration with the core network stack for superior long-term stability.
* **AIS Aids to Navigation (AtoN) Support:** Added full decoding and WebUI rendering support for AIS Aids to Navigation reports (PGN 129041). Virtual and physical navigational marks (such as buoys and lighthouses) are now properly recognized by the system and displayed on the map with their distinct navigational diamond icons and custom colored tags.
* **Experimental AIS Class B Extended Support (PGN 129040):** Experimental parser for AIS Class B Extended Position Reports. Because the public specification for this PGN remains partially undocumented and lacks out-of-the-box support in standard NMEA 2000 libraries, intoduced dedicated bit-level decoder to extract vessel names, dimensions, and ship types directly from the raw NMEA2000 message.
* **Signal K AIS Data Streaming & Strict Schema Compliance:** Implemented full broadcasting of AIS targets (Class A, Class B, and AtoN) over the Signal K data stream. The integration strictly adheres to the official Signal K specification by dynamically routing contexts: physical ships are mapped to `vessels.*` using `design.aisShipType`, while navigational marks are correctly isolated into the `aton.*` root context using `atonType`. This prevents downstream servers (like OpenCPN or Signal K Node Server) from misclassifying buoys and lighthouses as regular vessels.
* **Signal K Electrical Subsystem Support:** Implemented full dynamic routing and broadcasting of NMEA 2000 DC power sources (PGN 127506/127508) over both the REST API and Delta streams. The system strictly adheres to the Signal K schema by automatically translating raw NMEA 2000 equipment codes into standard electrical categories (such as batteries, solar, alternators, and wind). It features a zero-fragmentation dynamic path generator that seamlessly maps user-defined UI labels—or native NMEA Instance IDs as safe fallbacks—directly into the electrical.* context.
* **Persistent AIS Address Book:** Introduced a long-term, memory-safe local database to persistently store static details (names, dimensions, ship types) of up to 2000 previously encountered vessels. This allows the system to instantly identify and fully render returning ships the moment their first positional data is received, bypassing the wait for delayed NMEA static data packets.
* **AIS State Caching & I/O Optimization:** Implemented asynchronous local caching for active AIS targets to ensure instant map and UI restoration upon page reloads. The data serialization process has been completely decoupled from the main WebSocket loop and offloaded to the garbage collection cycle, eliminating UI freezes and significantly reducing browser I/O strain during high-frequency data streams.
* **AIS Targets Interface Redesign:** Restructured the AIS TARGETS tab to automatically group vessels by distance ranges (<3 NM, 3-10 NM, >10 NM) and introduced a dedicated subgroup for AtoN (Aids to Navigation) objects.
* **Native mDNS & Zero-Conf Discovery:** Migrated mDNS to the native ESP-IDF API for improved memory management and thread safety. The gateway now automatically advertises the Web Panel (HTTP), Signal K (HTTP/WS with required TXT records), NMEA 0183 (TCP/UDP), and aBin (TCP/UDP) services for seamless discovery by navigation software.

### Bug Fixes
* **AIS Target Starvation Fix:** Resolved an issue in the AIS dispatcher logic where distant vessels were being "starved" and dropped from the output queue during high-density traffic. The transmission algorithm has been rebalanced to guarantee that far-range targets receive fair data update cycles without compromising the rapid refresh rates required for close-range collision avoidance.
* **AIS SignalK Timestamps:** Added timestamps to AIS SignalK messages. This ensures proper synchronization and compatibility, allowing third-party marine applications like Boat Instruments to correctly read and parse the data.
* **WebGUI Edge Case Handling:** Applied several minor JavaScript fixes across the web interface. The UI now properly handles edge cases, such as missing sensor data streams or invalid AIS vessel coordinates, preventing rendering errors.

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