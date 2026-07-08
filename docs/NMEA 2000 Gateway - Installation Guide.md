# **NMEA 2000 Gateway: Firmware Installation & Setup Guide**

### **⚠️ IMPORTANT LEGAL NOTICE & DISCLAIMER**

Please keep in mind that this is a community-driven, open-source endeavor and **NOT** a certified marine instrument. This gateway is **NOT** meant for critical safety or primary navigation tasks. All software and hardware details are shared "AS IS," with no warranties provided. Use of this firmware is done strictly at your own discretion and risk; the authors and developers are not responsible for any vessel incidents, equipment damage, or data loss that may occur.

Don't worry if you're not a programmer – installing the firmware is very easy and works directly from your web browser\! Just follow these simple steps to flash the device and configure it for your boat's network.

## **Part 1: Flashing the Firmware**

### **Step 1: Know your board and get the right file**

There are two different versions of the firmware depending on the hardware you have. Make sure you download the correct .bin file:

* **ESP32-S3 (Newer board):** Download the factory\_esp32-s3... .bin file.  
* **Standard ESP32 / Wemos (Older board):** Download the factory\_esp32dev... .bin file.

### **Step 2: Open the Web Flasher**

Go to the official Espressif Web Flasher page (use a Chromium-based browser like Chrome, Edge, or Opera; Safari and Firefox do not support Web Serial API):

https://espressif.github.io/esptool-js/

### **Step 3: Connect to your computer**

1. Connect your ESP board to your computer using a data-capable USB cable.  
2. On the website, click the **Connect** button.  
3. A small browser popup will appear. Select the USB port your board is connected to (it might be called USB JTAG/serial debug unit, CH340, or similar) and click **Connect**.

*Troubleshooting tip: If it fails to connect or gets stuck, press and hold the physical **BOOT** button on your board while clicking "Connect" on the screen.*

### **Step 4: Flash the Firmware**

Once connected, you will see the flashing interface.

1. In the **Flash Address** field, type exactly: 0x0 (Yes, it is 0x0 for BOTH board types\!).  
2. Click the **Choose File** button and select the .bin file you downloaded in Step 1\.  
3. Hit the **Program** button.  
4. Wait for the progress bar to reach 100%. Do not unplug the cable\!

### **Step 5: The Physical Reset**

When the screen says the flashing is complete, the board will not start automatically. You need to do a hard reset to boot the new system:

* Simply press the physical **RST / EN** button on the board.  
* Alternatively, just unplug the USB cable and plug it back in.

## **Part 2: Initial Setup & Configuration**

### **Step 6: Connecting to Wi-Fi**

After a successful boot, the device will act as an Access Point and broadcast its own Wi-Fi network. The network name (SSID) depends on your hardware:

| Board Type | Wi-Fi Network Name (SSID) | Default Password   |
| :---- | :---- | :---- |
| ESP32-S3 | n2k-gateway-s3 | baltyk2026 |
| Standard ESP32 | n2k-gateway-generic | baltyk2026 |

Connect to this network using your phone or computer. Once connected, open your web browser and go to the default IP address to access the configuration dashboard:  http://192.168.4.1.   
Here you can connect the gateway to your existing boat Wi-Fi network by navigating to the **Config \> WIFI network** tab.  
Once the gateway successfully connects to your existing network, it will stop broadcasting its own signal. You can then access it from any device on your boat's network using its mDNS address:

* For ESP32-S3: http://n2k-gateway-s3.local  
* For Standard ESP32: http://n2k-gateway-generic.local

### **Step 7: CAN Bus Hardware Configuration**

To ensure proper communication with your NMEA 2000 network, you need to define which GPIO pins are connected to your CAN transceiver. This can be configured dynamically:

1. Go to the **Config \> CAN hardware configuration** tab in the web interface.  
2. Enter the correct pin numbers for your RX (Receive) and TX (Transmit) lines.
   FORMATTING RULE: Type ONLY the pure digit (e.g., 4, 5, or 16). Do NOT type letters like D4, GPIO5, or TX. Check your board documentation for free available pins.

*CRITICAL WIRING NOTE FOR BEGINNERS:* Unlike standard serial connections (UART) where you cross the TX and RX wires, CAN bus transceivers are wired straight through (1:1).
- Use GPIO numbers, not board labels: The gateway needs the actual ESP32 GPIO number. Be careful with boards like Wemos/NodeMCU where a pin labeled D4 on the physical board might actually be GPIO 2. Always check the pinout diagram for your specific board!
- The GPIO you enter for TX PIN must connect to the TX (or TXD) pin on your CAN transceiver., The GPIO you enter for RX PIN must connect to the RX (or RXD) pin on your CAN transceiver.
Do not cross them!

*Important Hardware Note:* Make sure your CAN transceiver does **not** have a built-in termination resistor (120 Ohm), unless your gateway is positioned at the very end of the physical NMEA 2000 backbone. The NMEA 2000 network should only have two terminators, one at each end of the main trunk.  
*Note: Changing Wi-Fi settings or CAN pins requires a restart of the gateway to take effect.*

### **Step 8: Factory Reset**

If you ever lose access to the device or configure the wrong Wi-Fi settings, you can easily restore the gateway to its original state.

* Press and hold the physical button connected to **PIN 0 (BOOT button)** for **6 to 8 seconds**.  
* The device will wipe its configuration, reboot, and broadcast its original Access Point network again.

Step 9: Demo Mode (N2K Generator)  
The gateway features a built-in Demo Mode that allows you to test the interface without being connected to a physical vessel network. You can enable it by navigating to the Config \> N2K GENERATOR MODE (DEMO) tab. Once activated, an internal generator will simulate NMEA 2000 network traffic.

Important Note on GPS Sources & Data Selection: The generator intentionally simulates two different devices broadcasting GPS positions at the same time (a GPSMAP 8412xsv and an AIS700 Transceiver). This is designed specifically to let you test the gateway's source selection feature.

Because there are two active GPS sources, the position on your screen might fluctuate or "jump". To fix this and see how data prioritization works:

Click the N2K icon at the top of the web interface.

Select your preferred GPS data source from the list.

Note: The internal generator simulates standard navigation data but does NOT generate AIS targets (other vessels).

### **Step 9: Demo Mode (N2K Generator)**

The gateway features a built-in Demo Mode that allows you to test the interface without being connected to a physical vessel network. You can enable it by navigating to the **Config \> N2K GENERATOR MODE (DEMO)** tab. Once activated, an internal generator will simulate NMEA 2000 network traffic.

*Important Note on GPS Sources & Data Selection:* The generator intentionally simulates two different devices broadcasting GPS positions at the same time (a GPSMAP 8412xsv and an AIS700 Transceiver). This is designed specifically to let you test the gateway's source selection feature.

Because there are two active GPS sources, the position on your screen might fluctuate or "jump". To fix this and see how data prioritization works:

1. Click the **N2K** icon at the top of the web interface.  
2. Select your preferred GPS data source from the list.

*Note: The internal generator simulates standard navigation data but does NOT generate AIS targets (other vessels).*

### **Step 10: Supported Protocols & Services**

The gateway utilizes various communication methods to transmit boat data to popular navigation software like OpenCPN, Navionics, or Aqua Map. These options can be toggled and adjusted within the **Config \> SERVICES & PORTS** menu.

The following services are available:

* **Signal K WebSockets:** A contemporary marine data format based on JSON.  
* **NMEA 0183 (TCP/UDP Stream):** Support for the industry-standard legacy protocol. Users can select TCP, UDP, or dual-stream transmission, with a default port of 10110\.  
* **Actisense Binary (PC Gateway):** Designed for sending raw network packets to computer software. The default port is 10120\.

*Current Limitations / Known Issues:* As an evolving open-source project, please be aware of the following developmental constraints:

* **Signal K Security:** Authentication tokens are pending. The stream is currently unsecured for all local network users.  
* **Signal K AIS:** Translation of AIS data is not yet functional in Signal K. *(Note: AIS remains fully functional via the Actisense Binary or NMEA 0183 output\!)*.

## 

## **Acknowledgments & Open-Source Libraries**

This project wouldn't be possible without the incredible work of the open-source community. A massive thank you to the creators of these fantastic libraries for handling NMEA 2000 and NMEA 0183 protocols seamlessly:

* Timo Lappalainen \- [NMEA2000 Library](https://github.com/ttlappalainen/NMEA2000)  
* Timo Lappalainen \- [NMEA2000 ESP32 Driver](https://github.com/ttlappalainen/NMEA2000_esp32)  
* Jiauka \- [NMEA2000 ESP32xx Driver](https://github.com/jiauka/NMEA2000_esp32xx)  
* Timo Lappalainen \- [NMEA0183 Library](https://github.com/ttlappalainen/NMEA0183)  
* Wellenvogel \- [NMEA2000-Gateway with ESP32](https://github.com/wellenvogel/esp32-nmea2000)

