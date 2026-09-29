## Skipper's Quick Start: NMEA 2000 Gateway Guide (4-Wire Version)

This short guide will show you how to set up your wireless NMEA 2000 gateway yourself and keep the installation neat. It will send data from your boat's network (wind, AIS, engines, depth) straight to your tablet running Navionics, OpenCPN, or Aqua Map. No soldering, no external chargers, and no cable clutter.

Step 1: What exactly should you buy? (Shopping List)
Gateway board: Buy the Waveshare ESP32-S3 RS485-CAN. 
<p>
  <img src="images/esp32-s3-rs485-can-1.png" alt="Waveshare ESP32-S3 RS485-CAN" height="400">
</p>
It is an excellent choice for sailors. Why? It has a built-in CAN interface and an integrated power converter (7-36V). This means you can power it directly from the NMEA 2000 cable!

Boat network cable: A ready-made drop cable with one end cut off (an NMEA 2000 drop cable with loose wires). The other end should have a plug compatible with your system (a standard T-connector).

Step 2: Installing the system (Do it comfortably at home)
Before taking the equipment to the boat, install the system at your desk first. This lets you test it without risking any of the boat's electronics.

Connect the Waveshare board to your computer with a standard USB cable.

Open a browser (Chrome or Edge is recommended) and go to: [https://espressif.github.io/esptool-js/](https://espressif.github.io/esptool-js/)

Click Connect and select your board's USB port in the dialog box.

In the "Flash Address" field, enter: 0x0

Click "Choose File", select our firmware file (factory_esp32-s3... .bin), and then click Program.

Wait until the progress bar reaches 100%. Done! The gateway is programmed, and you can disconnect the USB cable.

Step 3: Connecting it on the boat (Only 4 wires)

<p>
  <img src="images/esp32-s3-rs485-can-2.png" alt="NMEA2000" height="400">
</p>

It is time to connect the gateway to the boat's backbone. You will do this using the screw terminals on the board; a small screwdriver is all you need.

Strip a section of the black insulation from the cut end of the NMEA 2000 cable. You will find colored wires inside. Connect them to the terminals on the board as follows:

Red (+12V NMEA power): connect it to the power input (marked VIN or VCC in the power section).

Black (NMEA ground / GND): connect it to the ground input (marked GND).

White (NET-H data): connect it to the CAN_H (or H) terminal.

Blue (NET-L data): connect it to the CAN_L (or L) terminal.

(Fold back the cable's silver shield and insulate it with tape.)

Now simply plug the cable into an available NMEA 2000 T-connector on the boat. The device will start automatically together with the boat's navigation system!

⚠️ Important safety note: The gateway is now powered directly from the boat's batteries (12V). Do not connect it to a laptop with a USB cable at the same time, for example for an update, if the laptop is charging from the boat's electrical system. This can create a ground loop and damage the computer's ports. Use only the WiFi connection for configuration changes.

Step 4: Disabling the Gateway's Termination Resistor

<p>
  <img src="images/ESP32-S3-CAN-2CH-details-intro.png" alt="Waveshare ESP32-S3 RS485-CAN" height="400">
</p>

If the gateway is connected to the NMEA 2000 backbone through a drop cable and a T-connector, so it must not add a third termination resistor to the network. Locate the termination-resistor jumper, marked as item 8 in the accompanying board diagram, and set it to disconnect the board's built-in 120 Ω resistor. The NMEA 2000 backbone must have exactly two 120 Ω terminators, one at each physical end. Do not add a separate terminator at the gateway. 

Step 5: Configuring WiFi
The gateway is powered and is receiving data. It is time to connect to it. By default, the gateway works as an access point:

Take your phone or tablet, open the WiFi settings, and connect to the new network named: n2k-gateway-s3. (Default password: baltyk2026).

Open a browser and enter: [http://192.168.4.1](http://192.168.4.1) (or simply [http://n2k-gateway-s3.local](http://n2k-gateway-s3.local)).

You will see a dashboard with your boat's instruments.

Optional: If you have an internet-connected router on the boat (such as Starlink), you can switch the gateway to Client Mode in the "Config -> WIFI network" tab by entering the password for the main router. The gateway will then join the boat's local network.

Step 6: Displaying data in navigation apps
Would you like to see AIS targets and wind data overlaid directly on the map in your favorite app? Make sure your tablet is connected to the gateway's WiFi network (or the boat's network, if you configured it that way).

For Navionics users (Boating App):

Go to Menu -> Paired Devices -> Add Device.

Enter the Host (gateway IP address): 192.168.4.1 (or the address assigned by your boat's router).

Enter port: 10110 and select the TCP protocol.

For OpenCPN users:

Go to Options -> Connections -> Add Network Connection.

Best choice (all NMEA 2000 data): Select the type "NMEA 2000 / Actisense Protocol", enter the gateway's IP address, and use port 10120 (TCP/UDP).

Alternative (Signal K): Select "Signal K", enter the gateway's IP address, and use port 80.

That's all! You can tuck the gateway behind the navigation panel and enjoy modern, wireless data transfer.