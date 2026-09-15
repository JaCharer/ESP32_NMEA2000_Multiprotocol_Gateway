Below is the detailed breakdown of each hardware profile.

## **1.ESP32 S3 with external mini OLED ([esp32-s3-rs485-can](https://www.waveshare.com/wiki/ESP32-S3-RS485-CAN?srsltid=AU7gw4UpSBSYJJDARLye0Bpbc1NREzqWy1Cv9nwe4-2apnAH_INsF1R2))**

Designed for high-performance processing, leveraging the ESP32-S3's extra memory to handle more concurrent network clients and larger data buffers.

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

## **2\. ESP32 S3 with 4-inch Touch Screen ([esp32-s3-touch-lcd-4-can](https://www.waveshare.com/wiki/ESP32-S3-Touch-LCD-4?srsltid=AU7gw4Vo-_x9LnKvxe95QhYLbenJ52WpGhiPa7VkT3rCwzjQ5b90Oavq))**

A premium UI-focused build utilizing a large Waveshare smart display. It shares the same high-performance networking limits as the standard S3 build but routes the CAN bus to different pins and uses a dedicated graphics library (moononournation/GFX).

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

## **3\. Classic Wemos with Built-in OLED ([esp32dev-wemos-oled](https://github.com/pjpmarques/LOLIN-ESP32-OLED))**

A cost-effective, classic ESP32 build tailored for the popular Wemos/Lolin boards that feature an integrated screen. Because the classic ESP32 lacks external PSRAM, memory limits (stack sizes, client connections) are heavily optimized to prevent crashes.

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

## **4\. LilyGO TTGO T-Display ([esp32dev-ttgo](https://wiki.lilygo.cc/products/t-display-series/t-display/))**

Tailored for the popular TTGO board featuring a vibrant color TFT display. Like the Wemos, it runs on a classic ESP32 and features strict memory management, but it heavily customizes the SPI pins to drive the onboard ST7789 screen.

* **Board:** LilyGO TTGO T-Display (Classic ESP32)  
* **Display:** Integrated 1.14" IPS Color TFT (135x240 resolution, ST7789 driver)  
* **mDNS / SSID:** n2k-gateway-TTGO.local / N2K\_Gateway\_TTGO  
* **Capacity:** Constrained (Max 8 Web GUI clients, 2 SignalK clients, 4 NMEA0183 TCP clients, 4 TCP Actisense Binary stream clients)  
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

## **5\. ESP32 S3 (esp32-s3-generic)**

A pure "black box" gateway. It utilizes the powerful ESP32-S3 with PSRAM for maximum network throughput and client capacity, but completely strips out all UI/Display rendering logic to save CPU cycles.

* **Board:** ESP32-S3-DevKitC-1 (16MB Flash, 8MB PSRAM)  
* **Display:** **None (Headless)**  
* **mDNS / SSID:** n2k-gateway-s3.local / N2K\_Gateway\_S3  
* **Pin Mapping:**

| Function | Pin | Notes |
| :---- | :---- | :---- |
| **CAN TX** | GPIO 15 | Connects to CAN Transceiver TX |
| **CAN RX** | GPIO 16 | Connects to CAN Transceiver RX |
| **Reset** | GPIO 0 | Hardware BOOT button |

## **6\. Classic ESP32 (esp32dev-generic)**

The most basic "black box" configuration. Standard ESP32 without a screen. Highly optimized memory usage to function reliably as a background bridge without running out of RAM.

* **Board:** Generic ESP32 Dev Board (No PSRAM)  
* **Display:** **None (Headless)**  
* **mDNS / SSID:** n2k-gateway-generic.local / N2K\_Gateway\_Generic  
*   
* **Capacity:** Constrained (Max 2 Web GUI clients, 2 SignalK clients, 2 NMEA0183 TCP clients, 2 TCP Actisense Binary stream clients)  
* **Pin Mapping:**

| Function | Pin | Notes |
| :---- | :---- | :---- |
| **CAN TX** | GPIO 25 | Connects to CAN Transceiver TX |
| **CAN RX** | GPIO 26 | Connects to CAN Transceiver RX |
| **Reset** | GPIO 0 | Hardware BOOT button |

