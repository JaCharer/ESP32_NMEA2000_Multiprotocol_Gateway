This section helps you choose the right gateway hardware profile for your build. The simplest and most reliable starting point is the ESP32-S3 generic profile. If you want a display and a ready-made marine enclosure layout, the RS485-CAN profile is the next step and uses the same core firmware with a different pin mapping and OLED.

The project currently defines several PlatformIO profiles for different board layouts. The table below reflects the active firmware configuration used in [platformio.ini](../platformio.ini).

> **Recommended starting point:** the `esp32-s3-generic` build is the cleanest and most stable profile to begin with. It is the best baseline for first-time setup and validation, and it runs on the same firmware stack as the `esp32-s3-rs485-can` variant. The RS485-CAN board is essentially the same ESP32-S3 platform with an added display and adjusted CAN pins.

## **1. ESP32-S3 with mini OLED**

### [esp32-s3-rs485-can](https://www.waveshare.com/wiki/ESP32-S3-RS485-CAN?srsltid=AU7gw4UpSBSYJJDARLye0Bpbc1NREzqWy1Cv9nwe4-2apnAH_INsF1R2)

A practical S3 build for users who want more headroom, more client capacity, and a small OLED display.

* **Board:** ESP32-S3-DevKitC-1 (16MB Flash, 8MB OPI PSRAM)  
* **Display:** [External 0.91" I2C OLED](https://www.waveshare.com/wiki/0.91inch_OLED_Module?srsltid=AU7gw4Wlur9xmsmnFvwcvC_xG93_MThsybgiMf0S1lfpP9_5JsjG_-U_) (128x32 resolution)   
* **mDNS / SSID:** n2k-gateway-s3.local / N2K\_Gateway\_S3  
* **Capacity:** Constrained (Max 8 Web GUI clients, 2 SignalK clients, 4 NMEA0183 TCP clients, 4 TCP Actisense Binary stream clients)  
* **Pin Mapping:**

| Function | Pin | Notes |
| :---- | :---- | :---- |
| **CAN TX** | GPIO 15 | Connects to CAN Transceiver TX |
| **CAN RX** | GPIO 16 | Connects to CAN Transceiver RX |
| **I2C SDA** | GPIO 1 | Display Data |
| **I2C SCL** | GPIO 2 | Display Clock |
| **Reset** | GPIO 0 | Hardware BOOT button (Hold 5s for factory reset) |

## **2. ESP32-S3 with 4-inch touch display**

### [esp32-s3-touch-lcd-4-can](https://www.waveshare.com/wiki/ESP32-S3-Touch-LCD-4?srsltid=AU7gw4Vo-_x9LnKvxe95QhYLbenJ52WpGhiPa7VkT3rCwzjQ5b90Oavq)

A premium UI-focused variant for users who want a full-color touch panel and the extra processing headroom of the ESP32-S3.

* **Board:** ESP32-S3 (Specifically tuned for Waveshare ESP32-S3-Touch-LCD-4)  
* **Display:** 4.0" RGB Touch LCD  
* **mDNS / SSID:** n2k-s3-lcd.local / N2K\_S3\_lcd  
* **Capacity:** Constrained (Max 8 Web GUI clients, 2 SignalK clients, 4 NMEA0183 TCP clients, 4 TCP Actisense Binary stream clients)  
* **Pin Mapping:**

| Function | Pin | Notes |
| :---- | :---- | :---- |
| **CAN TX** | GPIO 6 | *Note: Changed from standard S3 build* |
| **CAN RX** | GPIO 0 | *Note: Shares pin with BOOT/Reset* |

*(Note: No hardware reset button implemented).*

## **3. Classic Wemos with built-in OLED**

### [esp32dev-wemos-oled](https://github.com/pjpmarques/LOLIN-ESP32-OLED)

A compact and inexpensive ESP32 build for users who want a small display without the PSRAM-heavy S3 platform.

* **Board:** Wemos/Lolin32 OLED (Classic ESP32, No PSRAM)  
* **Display:** Integrated 0.96" I2C OLED (128x64 resolution \- twice as tall as the S3 OLED)  
* **mDNS / SSID:** n2k-gateway-Wemos.local / N2K\_Gateway\_Wemos  
* **Capacity:** Constrained (Max 2 Web GUI clients, 2 SignalK clients, 2 NMEA0183 TCP clients, 2 TCP Actisense Binary stream clients)  
* **Pin Mapping:**

| Function | Pin | Notes |
| :---- | :---- | :---- |
| **CAN TX** | GPIO 25 | Default ESP32 CAN TX |
| **CAN RX** | GPIO 26 | Default ESP32 CAN RX |
| **I2C SDA** | GPIO 5 | Display Data |
| **I2C SCL** | GPIO 4 | Display Clock |
| **Reset** | GPIO 0 | FLASH button next to USB port |

## **4. LilyGO TTGO T-Display**

### [esp32dev-ttgo](https://wiki.lilygo.cc/products/t-display-series/t-display/)

A color-screen variant for users who prefer a bright TFT display and a classic ESP32 platform. It is a compact and capable build, but it is still a classic ESP32 profile with tighter memory limits than the S3 variants.

* **Board:** LilyGO TTGO T-Display (Classic ESP32)  
* **Display:** Integrated 1.14" IPS Color TFT (135x240 resolution, ST7789 driver)  
* **mDNS / SSID:** n2k-gateway-TTGO.local / N2K\_Gateway\_TTGO  
* **Capacity:** Constrained (Max 2 Web GUI clients, 2 SignalK clients, 2 NMEA0183 TCP clients, 2 TCP Actisense Binary stream clients)  
* **Pin Mapping:**

| Function | Pin | Notes |
| :---- | :---- | :---- |
| **CAN TX** | GPIO 25 | External Transceiver required |
| **CAN RX** | GPIO 26 | External Transceiver required |
| **TFT MOSI** | GPIO 19 | Display Data |
| **TFT SCLK** | GPIO 18 | Display Clock |
| **TFT CS** | GPIO 5 | Chip Select |
| **TFT DC** | GPIO 16 | Data/Command |
| **TFT RST** | GPIO 23 | Reset |
| **TFT BL** | GPIO 4 | Backlight control |

## **5. ESP32-S3 generic**

### esp32-s3-generic

This is the best starting profile for the first build and for everyday use without a display. It uses the full ESP32-S3 with PSRAM, gives the best headroom for clients and data traffic, and is the cleanest reference configuration for testing and stable operation.

The same firmware also runs correctly on the `esp32-s3-rs485-can` board, which adds an OLED and different pin mapping without changing the core functionality.

* **Board:** ESP32-S3-DevKitC-1 (16MB Flash, 8MB PSRAM)  
* **Display:** **None (Headless)**  
* **mDNS / SSID:** n2k-gateway-s3.local / N2K\_Gateway\_S3  
* **Capacity:** Constrained (Max 8 Web GUI clients, 2 SignalK clients, 4 NMEA0183 TCP clients, 4 TCP Actisense Binary stream clients)  
* **Pin Mapping:**

| Function | Pin | Notes |
| :---- | :---- | :---- |
| **CAN TX** | GPIO 15 | Connects to CAN Transceiver TX |
| **CAN RX** | GPIO 16 | Connects to CAN Transceiver RX |
| **Reset** | GPIO 0 | Hardware BOOT button |

## **6. Classic ESP32 generic**

### esp32dev-generic

The simplest headless ESP32 build for a minimal setup. It is suitable when you want a lightweight background gateway without a display and without the extra S3 memory headroom.

* **Board:** Generic ESP32 Dev Board (No PSRAM)  
* **Display:** **None (Headless)**  
* **mDNS / SSID:** n2k-gateway-generic.local / N2K\_Gateway\_Generic  
* **Capacity:** Constrained (Max 2 Web GUI clients, 2 SignalK clients, 2 NMEA0183 TCP clients, 2 TCP Actisense Binary stream clients)  
* **Pin Mapping:**

| Function | Pin | Notes |
| :---- | :---- | :---- |
| **CAN TX** | GPIO 25 | Connects to CAN Transceiver TX |
| **CAN RX** | GPIO 26 | Connects to CAN Transceiver RX |
| **Reset** | GPIO 0 | Hardware BOOT button |

