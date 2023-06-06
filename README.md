# SoftRF DIY - Stratux compatible fork for T-Beam, T-Beam S3 Supreme and T-Echo

- enable SoftRF to work as a proper GPS and Baro source for Stratux (through USB)
- option to enter Aircraft ID
  - if an Aircraft ID is added, then ADDR_TYPE_ICAO is set for both Legacy and OGN
  - if the SoftRF factory ID remains, then ADDR_TYPE is set according to the selected protocol

**IMPORTANT**: All modifications are provided only in the source code so you need to be familiar with Arduino to compile and flash it for your platform. You need to install the latest version of **Arduino IDE** and add the following two entries into the Additional Board Manager URLs:
- **T-Beam and T-Beam S3 Supreme**: `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`
- **T-Echo**: `https://adafruit.github.io/arduino-board-index/package_adafruit_index.json`

**Arduino IDE** settings for **T-Beam v0.7, T-Beam v1.0, T-Beam v1.1**
- Select Tools -> Board -> ESP32 Dev Module
- Select Tools -> CPU Frequency -> 80MHz
- Select Tools -> Flash Frequency -> 80MHz
- Select Tools -> Flash Mode -> DIO
- Select Tools -> Flash Size -> 4MB
- Select Tools -> Partition Scheme -> Minimal SPIFFS
- Select Tools -> PSRAM -> Enabled
- Select Tools -> Upload Speed -> 921600
- Select Tools -> open serial monitor @ 115200 baud
- connect your T-Beam
- Select Tools -> Port -> (select accodingly)
- compile/upload

**Arduino IDE** settings for **T-Beam S3 Supreme** (**Caution:** the UF2 firmware upload option will no longer work after the following steps)
- Select Tools -> Board -> ESP32S3 Dev Module
- Select Tools -> CPU Frequency -> 80MHz
- Select Tools -> Flash Frequency -> 80MHz
- Select Tools -> Flash Mode -> QIO
- Select Tools -> Flash Size -> 8MB
- Select Tools -> Partition Scheme -> 8MB with spiffs (copy https://github.com/lyusupov/tinyuf2/blob/master/ports/espressif/partitions-8MB-SPIFFS.csv to your local Arduino partitionss path, e.g. `..packages/esp32/hardware/esp32/2.0.9/tools/partitions` and replace `default_8MB.csv`)
- Select Tools -> PSRAM -> QSPI PSRAM
- Select Tools -> Upload Speed -> 921600
- Select Tools -> Upload Mode -> UART0 / Hardware CDC
- Select Tools -> USB Mode -> USB-OTG CDC (TinyUSB)
- Select Tools -> USB Firmware MSC on Boot -> disabled
- Select Tools -> USB DFU on Boot -> disabled
- Select Tools -> USB CDC on Boot -> enabled
- connect your T-Beam S3 Supreme
- put your T-Beam in Espressif Service Mode (press and keep holding RESET, press and keep holding BOOT, release RESET first, thereafter release BOOT)
- compile/upload
- press RESET

1st alternative for **T-Beam S3 Supreme** (and if you want to maintain the UF2 firmware upload option):
- follow all steps above but do **NOT** compile/upload:
  - select **Export Compiled Binary** and then locate `SoftRF.ino.bin`
  - convert `SoftRF.ino.bin` to UF2 using `uf2conv.py SoftRF.ino.bin -c -b 0x00 -f ESP32S3 -o SoftRF.ino.uf2` (from https://github.com/microsoft/uf2/tree/master/utils)
  - connect your T-Beam S3 Supreme and put it in UF2 upload mode (press RESET and shortly thereafter BOOT)
  - upload the `SoftRF.ino.uf2` file to the TBEAMBOOT drive, see also: https://github.com/lyusupov/SoftRF/blob/master/software/firmware/binaries/README.md#esp32-s3

2nd alternative for **T-Beam S3 Supreme** (and if you want to maintain the UF2 firmware upload option):
- follow all steps above but do **NOT** compile/upload:
  - select **Export Compiled Binary** and then locate `SoftRF.ino.bin`
  - connect you PC with the SoftRF WiFi
  - select `Firmware Update``
  - upload `SoftRF.ino.bin`

**Arduino IDE** settings for **T-Echo**
- Select Tools -> Board -> Nordic nRF52840 DK
- connect your T-Echo
- Select Tools -> Port -> (select accodingly)
- compile/upload

In case you want to convert a **T-Beam based OGN Tracker to run SoftRF**, you first need to apply the following GPS reset:
- https://github.com/VirusPilot/LilyGo-T-Beam-GPS-Reset, otherwise the GPS chipset won't work with SoftRF (OGN Tracker uses 57600 baud vs. SoftRF using 9600 baud for the GPS-CPU connection)
- as a alternative you may consider using: https://github.com/Xinyuan-LilyGO/LilyGo-LoRa-Series/tree/master/examples/GPS/UBlox_Recovery, please uncomment the `utility.h` file according to your board model, otherwise compilation will report an error

**T-Beam and T-Beam S3 Supreme modifications:**
- u-blox GPS configuration:
  - enable GSA, GSV, VTG
  - enable GPS, GALILEO, BEIDOU and SBAS
  - enable NMEA extended protocol
- LEGACY NMEA traffic messages are disabled (to relax data rate, Stratux receives LEGACY directly anyhow)
- default connection with Stratux: **USB** (115200 baud), the USB T-Beam connection with Stratux works best if `init_uart_baud=115200` is added to the `/boot/config.txt` file on the Raspberry Pi
- WiFi AP IP changed to `192.168.4.1` to avoid conflicts with Stratux WiFi AP IP

**T-Echo modifications:**
- L76K GPS configuration:
  - enable GSA, GSV, VTG
  - enable GPS, GLONASS and BEIDOU
  - NMEA output through USB (instead of Bluetooth)
- default connection with Stratux: **USB** (115200 baud), the USB T-Echo connection with Stratux works best if `init_uart_baud=115200` is added to the `/boot/config.txt` file on the Raspberry Pi
- LK8EX1 and LEGACY traffic messages over serial connection are disabled (to relax data rate, Stratux receives LEGACY directly anyhow)

**Limitations:**
- GPS update rate is limited to 1 Hz in SoftRF, which is good enough for Stratux except when using GPS as a pseudo AHRS
- the L76K only supports the NMEA "strict" protocol version, therefore some extended satellite information (like elevation, azimut and numbering) is not provided for some satellites and therefore the GPS info page in Stratux is incomplete. Furthermore BEIDOU satellites are not displayed at all but are in fact used and counted for "in solution"
- if your T-Beam or T-Echo has a baro sensor (e.g. BMP280) included, you can omit your Stratux baro module as SoftRF is providing the baro altitude to your Stratux; please note the following limitations when adding a baro module to your T-Beam: https://github.com/lyusupov/SoftRF/issues/32#issuecomment-420242682

**Issues:**
- it appears that on some SX1262 based T-Beams (not the T-Beam S3 Supreme) the modified GPS configuration sometimes reverts to the default GNSS settings, e.g. GLONASS is enabled instead of BEIDOU.

**Recommendations for T-Beam and T-Beam S3 Supreme:**
- modify SoftRF settings (https://github.com/lyusupov/SoftRF/wiki/Settings), using the SoftRF webinterface (192.168.4.1)

**Recommendations for T-Echo:**
- load OGN database: https://github.com/lyusupov/SoftRF/wiki/Badge-Edition.-Aircrafts-database
- modify SoftRF settings (https://github.com/lyusupov/SoftRF/wiki/Settings) by **downloading** the following scripts, **opening** them in a browser to generate the appropriate $PSRFC and $PSKVC sentences and then **sending** these generated sentences to the SoftRF device via a serial USB console (e.g. Arduino IDE comes with a nice built in serial USB console):
  - https://github.com/VirusPilot/SoftRF/blob/master/software/app/Settings/basic.html (e.g. Protocol, Aircraft type, Aircraft ID)
  - https://github.com/VirusPilot/SoftRF/blob/master/software/app/Settings/ui.html (e.g. Units, e-Paper 'ghosts' removal)
