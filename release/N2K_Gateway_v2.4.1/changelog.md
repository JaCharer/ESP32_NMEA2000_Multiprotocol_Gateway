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