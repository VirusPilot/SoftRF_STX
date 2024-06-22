## ATTENTION: it is strongly recommended to only use the following open standards based protocols: ADS-L, OGN or FANET
## ATTENTION: only T-Beam S3 Supreme and T-Echo UF2 binaries are provided/tested on a regular basis
### It is recommended to consider the following alternatives:
- SoftRF fork with a lot of enhancements: https://github.com/moshe-braner/SoftRF
- ADS-L/OGN/FANET tracker implementation: https://github.com/pjalocha/ogn-tracker (WIP)

## Features
- enable SoftRF to work as a proper GPS and Baro source for Stratux (through USB)
- option to enter `AircraftID` (through SoftRF WiFi settings page: http://192.168.1.1/settings)
  - if added (`AircraftID: <ICAO hex Code>`), then ADDR_TYPE_ICAO is set (this is based on the assumtion that your airplane has a transponder)
  - if **not** added, then the SoftRF factory ID remains (`AircraftID: 0`) and ADDR_TYPE_FLARM is set

## UF2 Binaries
UF2 binaries are available for the following platforms and can be downloaded as part of **`SoftRF.zip`** from here: https://github.com/VirusPilot/SoftRF/actions (click on the latest workflow run and download **`SoftRF.zip`** "Artifact"):

**T-Beam S3 Supreme - UF2 update method** (aka. USB Mass Storage method):
- connect your T-Beam S3 Supreme to your PC and put it in UF2 upload mode (press **RESET** and shortly thereafter **BOOT**)
- upload the `SoftRF.zip/esp32.esp32.esp32s3/SoftRF.ino.uf2` file to the TBEAMBOOT drive, see also: https://github.com/lyusupov/SoftRF/blob/master/software/firmware/binaries/README.md#esp32-s3

**T-Echo - UF2 update method** (aka. USB Mass Storage method):
- connect your T-Echo to your PC and put it in UF2 upload mode (double-click **RESET**)
- upload the `SoftRF.zip/adafruit.nrf52.pca10056/SoftRF.ino.uf2` file to the **NRF52BOOT** or **TECHBOOT** drive that shows up on your PC, see also: https://github.com/lyusupov/SoftRF/blob/master/software/firmware/binaries/README.md#nrf52840

**Please be aware that flashing these binaries on your SoftRF device may render it unusable**

## T-Beam S3 Supreme modifications:
- u-blox GPS configuration:
  - enable GSA, GSV, VTG, GST
  - enable GPS, GALILEO, BEIDOU and SBAS (u-blox 10 default)
  - enable NMEA extended protocol
- LEGACY NMEA traffic messages are disabled (to relax data rate, Stratux receives LEGACY directly anyhow)
- default connection with Stratux: **USB** (115200 baud), the USB T-Beam connection with Stratux works best if `init_uart_baud=115200` is added to the `/boot/config.txt` file on the Raspberry Pi (`/boot/firmware/config.txt` for Bookworm)

## T-Echo modifications:
- L76K GPS configuration:
  - enable GSA, GSV, VTG
  - enable GPS, GLONASS and BEIDOU
  - NMEA output through USB (instead of Bluetooth)
- default connection with Stratux: **USB** (115200 baud), the USB T-Echo connection with Stratux works best if `init_uart_baud=115200` is added to the `/boot/config.txt` file on the Raspberry Pi (`/boot/firmware/config.txt` for Bookworm)
- LK8EX1 and LEGACY traffic messages over serial connection are disabled (to relax data rate, Stratux receives LEGACY directly anyhow)

## Limitations:
- GPS update rate is limited to 1 Hz in SoftRF, which is good enough for Stratux except when using GPS as a pseudo AHRS (internally all u-blox based T-Beams use 10Hz measurement rate)
- the L76K only supports the NMEA "strict" protocol version, therefore some extended satellite information (like elevation, azimut and numbering) is not provided for some satellites and therefore the GPS info page in Stratux is incomplete. Furthermore BEIDOU satellites are not displayed at all but are in fact used and counted for "in solution"
- if your T-Beam S3 Supreme or T-Echo has a baro sensor (e.g. BMP280) included, you can omit your Stratux baro module as SoftRF is providing the baro altitude to your Stratux

## Recommendations for T-Beam S3 Supreme:
- load OGN database: https://github.com/lyusupov/SoftRF/wiki/Prime-Edition-MkIII#aircrafts-database
- modify SoftRF settings (https://github.com/lyusupov/SoftRF/wiki/Settings), using the SoftRF WiFi settings page: http://192.168.1.1/settings

## Recommendations for T-Echo:
- load OGN database: https://github.com/lyusupov/SoftRF/wiki/Badge-Edition.-Aircrafts-database
- modify SoftRF settings (https://github.com/lyusupov/SoftRF/wiki/Settings) by **downloading** the following scripts, **opening** them in a browser to generate the appropriate $PSRFC and $PSKVC sentences and then **sending** these generated sentences to the SoftRF device via a serial USB console (e.g. Arduino IDE comes with a nice built in serial USB console):
  - https://github.com/VirusPilot/SoftRF/blob/master/software/app/Settings/basic.html (e.g. Protocol, Aircraft type, Aircraft ID)
  - https://github.com/VirusPilot/SoftRF/blob/master/software/app/Settings/ui.html (e.g. Units, e-Paper 'ghosts' removal)

## DANGER ZONE - NOT RECOMMENDED FOR UNEXPERIENCED USERS
## Compiling/Flashing from Source Code
You need to be familiar with Arduino to compile and flash it for your platform. You need to install the latest version of **Arduino IDE** and add the following two entries into the Additional Board Manager URLs:
- **T-Beam and T-Beam S3 Supreme**: `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`
- **T-Echo**: `https://adafruit.github.io/arduino-board-index/package_adafruit_index.json`

### Arduino IDE settings for T-Beam
- Install "esp32 by Espressif" version 2.0.9 under "BOARDS MANAGER" 
- Select Tools -> Board -> ESP32 Dev Module
- Select Tools -> CPU Frequency -> 240MHz
- Select Tools -> Flash Frequency -> 80MHz
- Select Tools -> Flash Mode -> DIO
- Select Tools -> Flash Size -> 4MB
- Select Tools -> Partition Scheme -> minimal SPIFFS (1.9MB APP with OTA/190KB SPIFFS)
- Select Tools -> PSRAM -> Enabled
- Select Tools -> Upload Speed -> 921600
- Select Tools -> open serial monitor @ 115200 baud
- connect your T-Beam
- Select Tools -> Port -> (select accodingly)
- compile/upload

### Arduino IDE settings for T-Beam S3 Supreme (Caution: the UF2 firmware upload option will no longer work after the following steps)
- Install "esp32 by Espressif" version 2.0.9 under "BOARDS MANAGER" 
- Select Tools -> Board -> ESP32S3 Dev Module
- Select Tools -> CPU Frequency -> 240MHz
- Select Tools -> Flash Frequency -> 80MHz
- Select Tools -> Flash Mode -> QIO
- Select Tools -> Flash Size -> 8MB
- Select Tools -> Partition Scheme -> 8M with spiffs (3MB APP/1.5MB SPIFFS)
- Select Tools -> PSRAM -> QSPI PSRAM
- Select Tools -> Upload Speed -> 921600
- Select Tools -> Upload Mode -> USB-OTG CDC (TinyUSB)
- Select Tools -> USB Mode -> USB-OTG CDC (TinyUSB)
- Select Tools -> USB Firmware MSC on Boot -> disabled
- Select Tools -> USB DFU on Boot -> disabled
- Select Tools -> USB CDC on Boot -> enabled
- connect your T-Beam S3 Supreme
- put your T-Beam in Espressif Service Mode (press and keep holding **BOOT**, press and release **RESET**, thereafter release **BOOT**)
- compile/upload
- press **RESET**

alternative for **T-Beam S3 Supreme** (and if you want to maintain the UF2 firmware upload option):
- follow all steps above but do **NOT** compile/upload
- select **Export Compiled Binary** and then locate `SoftRF.ino.bin`
- convert `SoftRF.ino.bin` to UF2 using `uf2conv.py SoftRF.ino.bin -c -b 0x00 -f 0xc47e5767 -o SoftRF.ino.uf2` (from https://github.com/microsoft/uf2/tree/master/utils)
- connect your T-Beam S3 Supreme and put it in UF2 upload mode (press **RESET** and shortly thereafter **BOOT**)
- upload the `SoftRF.ino.uf2` file to the TBEAMBOOT drive, see also: https://github.com/lyusupov/SoftRF/blob/master/software/firmware/binaries/README.md#esp32-s3

### Arduino IDE settings for T-Echo
- Install "Adafruit nRF52 by Adafruit" under "BOARDS MANAGER" 
- Select Tools -> Board -> Nordic nRF52840 DK
- connect your T-Echo
- Select Tools -> Port -> (select accodingly)
- compile/upload

alternative for **T-Echo**: 
- follow all steps above but without compile/upload
- select **Export Compiled Binary** and then locate `SoftRF.ino.bin`
- convert `SoftRF.ino.bin` to UF2 using `uf2conv.py SoftRF.ino.bin -c -b 0x00 -f 0xADA52840 -o SoftRF.ino.uf2` (from https://github.com/microsoft/uf2/tree/master/utils)
- connect your T-Echo and put it in UF2 upload mode (double-click **RESET**)
- upload the `SoftRF.ino.uf2` file to the NRF52BOOT or TECHBOOT drive, see also: https://github.com/lyusupov/SoftRF/blob/master/software/firmware/binaries/README.md#nrf52840
