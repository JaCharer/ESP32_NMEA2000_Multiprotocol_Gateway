## Troubleshooting Boat Instrument Gauges: Fixing the Configuration File

If the gauges (indicators) in your Boat Instrument app are not displaying data—such as engine RPM or tank levels—it is likely due to an unspecified configuration error within the app.

When a gauge is added to the layout without a designated "Instance ID" (like `port`, `starboard`, or `main`), the app generates an invalid, broken Signal K path containing double dots (e.g., `propulsion..revolutions`). As a result, the application incorrectly listens for this malformed path. Even though your Signal K gateway is broadcasting the correct and valid data (e.g., `propulsion.port.revolutions`), the app ignores it because it is stubbornly waiting for the broken path. This mismatch results in empty gauges on your screen.

To fix this, you must manually edit the app's configuration file: **`boatinstrument.json`**.

### How to Fix the Configuration

Open the `boatinstrument.json` file in any standard text editor. Locate the `"pages"` section and look for the specific boxes (gauges) that are failing. You need to assign a valid `"id"` inside their `"settings"` objects.

**Example 1: Engine RPM (Propulsion)**

* **Broken Configuration:** The settings object is completely empty, causing the app to listen for `propulsion..revolutions`.
```json
"id": "propulsion-rpm",
"settings": {}

```


* **Fixed Configuration:** Add the `"id"` property (e.g., `"port"`).
```json
"id": "propulsion-rpm",
"settings": {
    "id": "port"
}

```



**Example 2: Freshwater Tank**

* **Broken Configuration:** The `"id"` field exists but is left blank (`""`), causing the app to listen for `tanks.freshWater..currentLevel`.
```json
"id": "tank-freshwater",
"settings": {
    "id": "",
    "capacity": 0.5
}

```


* **Fixed Configuration:** Enter the correct instance name (e.g., `"port"`).
```json
"id": "tank-freshwater",
"settings": {
    "id": "port",
    "capacity": 0.5
}

```



### Where to Find `boatinstrument.json`

The location of the configuration file depends on the operating system you are using to run the Boat Instrument app. Make sure the app is fully closed before saving changes to the file.

* **Android:**
`<storage>/Android/data/name.seeley.phil.boatinstrument/files/boatinstrument.json`
* **Linux:**
`$HOME/Documents/boatinstrument.json`
* **MacOS:**
`$HOME/Library/Containers/name.seeley.phil.boatinstrument/Data/Documents/boatinstrument.json`
* **Windows:**
`%UserProfile%/<My Documents>/boatinstrument.json`
* **Flutter-pi:**
`/boatinstrument.json`

Once the file is saved, simply restart the Boat Instrument app. The gauges will now expect the correct Signal K paths and display your data properly.

### Important: Connection Limits and the "Two-Socket" Behavior
If your gauges are configured correctly but the app completely fails to connect or periodically drops data, you might be hitting a strict connection limit on your gateway. By design, the Boat Instruments application inherently opens two simultaneous WebSocket connections to the Signal K server. When connecting to embedded micro-gateways, this behavior can quickly exhaust the available network slots. To ensure stable operation, you must manually allocate enough slots for the app on your gateway:

Open your gateway's WebUI in a browser.
Navigate to config -> ADVANCED & SYSTEM SETTINGS -> CONNECTION LIMITS (REQUIRES RESTART).

Ensure that the maximum number of Signal K clients is set high enough to accommodate the app. You must reserve at least 2 slots for every single device running the Boat Instruments app. Save the configuration and restart the gateway.

---
