# SoftRF fork
- for Stratux use via USB: **T-Beam**, **T-Beam S3 Supreme**
- for Stratux use via USB: **T-Motion** (only up to Stratux 1.6r1-eu032 image)
- for SkyDemon use via BLE: **T-Beam**, **T-Beam S3 Supreme**, **T-Echo** and **Seeed T1000-E**
## ATTENTION: it is strongly recommended to only use the following open standards based protocols (LEGACY protocol is broken and won't be fixed):
- ADS-L (default for all platforms)
- OGN
- FANET
## ATTENTION: only ESP32 (T-Beam, T-Beam S3 Supreme), nRF52 (T-Echo, T1000-E) and STM32 (T-Motion) binaries are provided/tested on a regular basis, they are provided as Github Actions Artifacts - a Github account is required to download them
## ATTENTION: Seeed T1000-E comes with pre-installed Meshtastic and needs to be prepared before flashing SoftRF for the first time (see: https://github.com/lyusupov/SoftRF/wiki/Card-Edition.-Quick-start)
## It is recommended to consider the following Stratux compatible alternatives:
- GXAirCom: FANET+ (FANET+FLARM): https://github.com/gereic/GXAirCom
  - only for **T-Beam** and **T-Beam S3 Supreme**
  - transmission of FANET and FLARM simultaneously
- ADS-L/OGN/FANET/PAW tracker implementation: https://github.com/pjalocha/ogn-tracker
  - only for **T-Beam** and **T-Beam S3 Supreme**
  - transmission of ADS-L/OGN/FANET/PAW simultaneously
- SoftRF fork with a lot of enhancements: https://github.com/moshe-braner/SoftRF
  - only for **T-Beam** and **T-Echo**
### General Features:
- option to enter `AircraftID` (through SoftRF WiFi settings page: http://192.168.1.1/settings or via serial connection, see below)
  - if added (`AircraftID: <ICAO hex Code>`), then ADDR_TYPE_ICAO is set (this is based on the assumtion that your airplane has a transponder)
  - if **not** added, then the SoftRF factory ID remains (`AircraftID: 0`) and ADDR_TYPE_FLARM is set

### T-Beam and T-Beam S3 Supreme Features:
- **USB mode** (default): enables SoftRF to work as a proper GNSS and Baro source for Stratux
- **Bluetooth LE mode**: enables SoftRF to work as a proper traffic rx/tx and GNSS source for SkyDemon (since version 4.0.0), please use the following settings:
  - ![1](https://github.com/user-attachments/assets/93e70aa7-cf88-4eaa-ad6e-fd0072773417)

### T-Echo and Seeed T1000-E Features:
- **Bluetooth LE mode** (enabled by default): enables SoftRF to work as a proper traffic rx/tx and GNSS source for SkyDemon

### T-Motion Features:
- **USB mode**: enables SoftRF to work as a proper GNSS source for Stratux

## UF2 Binaries
UF2 binaries are available for the following platforms and can be downloaded as part of **`SoftRF.zip`** from here: https://github.com/VirusPilot/SoftRF/actions (click on the latest workflow run and download **`SoftRF.zip`** "Artifact"):

**T-Beam S3 Supreme - UF2 update method** (aka. USB Mass Storage method):
- connect your T-Beam S3 Supreme to your PC and put it in UF2 upload mode (press **RESET** and shortly thereafter **BOOT**)
- upload the `SoftRF.zip/esp32.esp32.esp32s3/SoftRF.ino.uf2` file to the TBEAMBOOT drive, see also: https://github.com/lyusupov/SoftRF/blob/master/software/firmware/binaries/README.md#esp32-s3

**T-Echo and T1000-E - UF2 update method** (aka. USB Mass Storage method):
- connect your T-Echo or T1000-E to your PC and put it in UF2 upload mode (T-Echo: double-click RESET, T1000-E: press and hold the device button, then quickly connect the charging cable twice)
- upload the `SoftRF.zip/adafruit.nrf52.pca10056/SoftRF.ino.uf2` file to the **NRF52BOOT**, **TECHBOOT** or **T1000-E** drive that shows up on your PC, see also: https://github.com/lyusupov/SoftRF/blob/master/software/firmware/binaries/README.md#nrf52840

## "classic" T-Beam Binaries
"classic" binaries are available for the **T-Beam up to v1.2** and can be downloaded as part of **`SoftRF.zip`** from here: https://github.com/VirusPilot/SoftRF/actions (click on the latest workflow run and download **`SoftRF.zip`** "Artifact"):
- connect your T-Beam to your PC (via USB)
- open https://espressif.github.io/esp-launchpad/ with your **Chrome** browser, select "DIY" and "connect" your T-Beam
- it may be necessary the execute "Erase Flash" once
- upload all files from the `SoftRF.zip/esp32.esp32.esp32` folder according to the following order and execute "Program":
![Chrome T-Beam Flash Tool](https://github.com/VirusPilot/SoftRF/assets/43483458/bc84d81f-a71f-46e7-a4d8-c7c2ff45bc2e)

## T-Motion Binaries
prior to flashing you need to install STM32CubeProgrammer
- connect your T-Motion to your PC (via USB)
- put your T-Motion in DFU mode: hold **BOOT**, push **RESET**, release **BOOT**
- flash binary as follows:`./Arduino15/packages/STMicroelectronics/tools/STM32Tools/2.2.4/stm32CubeProg.sh -i dfu -f "SoftRF.ino.bin" -o 0x0 -v 0x0483 -p 0xdf11`

## T-Beam and T-Beam S3 Supreme modifications:
- u-blox GNSS configuration:
  - enable GSA, GSV, VTG, GST (in addition to GGA and RMC)
  - enable GPS, GALILEO, BEIDOU and SBAS (u-blox 10 default)
  - enable NMEA extended protocol
- default connection with Stratux: **USB** (115200 baud), the USB T-Beam connection with Stratux works best if `init_uart_baud=115200` is added to the `/boot/config.txt` file on the Raspberry Pi (`/boot/firmware/config.txt` for Bookworm)

## T-Echo and Seeed T1000-E modifications:
- GSA messges enabled (in addition to GGA and RMC)
- LK8EX1 and HEARTBEAT messages are disabled

## Limitations:
- GNSS update rate is limited to 1 Hz in SoftRF, which is good enough for Stratux except when using GNSS as a pseudo AHRS (internally all u-blox based T-Beams use 10Hz measurement rate)
- the L76K only supports the NMEA "strict" protocol version, therefore some extended satellite information (like elevation, azimut and numbering) is not provided for some satellites and therefore the GNSS info page in Stratux is incomplete. Furthermore BEIDOU satellites are not displayed at all but are in fact used and counted for "in solution"
- if your T-Beam or T-Echo has a baro sensor (e.g. BMP280) included, you can omit your Stratux baro module as SoftRF is providing the baro altitude to your Stratux

## Recommendations for T-Beam S3 Supreme:
- load OGN database: https://github.com/lyusupov/SoftRF/wiki/Prime-Edition-MkIII.-Aircrafts-database
- modify SoftRF settings (https://github.com/lyusupov/SoftRF/wiki/Settings), using the SoftRF WiFi settings page: http://192.168.1.1/settings

## Recommendations for T-Beam:
- modify SoftRF settings (https://github.com/lyusupov/SoftRF/wiki/Settings), using the SoftRF WiFi settings page: http://192.168.1.1/settings

## Recommendations for T-Echo, T1000-E and T-Motion:
- modify SoftRF settings (https://github.com/lyusupov/SoftRF/wiki/Settings) by **downloading** the following scripts, **opening** them in a browser to generate the appropriate $PSRFC sentences and then **sending** these generated sentences to the SoftRF device via a serial USB console (e.g. Arduino IDE comes with a nice built in serial USB console) or via a BLE serial terminal:
  - https://github.com/VirusPilot/SoftRF/blob/master/software/app/Settings/basic.html (e.g. Protocol, Aircraft type, Aircraft ID)

## DANGER ZONE - NOT RECOMMENDED FOR UNEXPERIENCED USERS
## Compiling/Flashing from Source Code
You need to be familiar with Arduino to compile and flash it for your platform. You need to install the latest version of **Arduino IDE** and add the following two entries into the Additional Board Manager URLs:
- **T-Beam and T-Beam S3 Supreme**: `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`
- **T-Echo** and **T1000-E**: `https://adafruit.github.io/arduino-board-index/package_adafruit_index.json`

### Arduino IDE settings for T-Beam
- based on https://github.com/Xinyuan-LilyGO/LilyGo-LoRa-Series?tab=readme-ov-file#3%EF%B8%8F⃣arduino-ide-quick-start
- Install "esp32 by Espressif" version 2.0.9 under "BOARDS MANAGER" 
- Select Tools -> Board -> ESP32 Dev Module
- Select Tools -> CPU Frequency -> 240MHz
- Select Tools -> Flash Frequency -> 80MHz
- Select Tools -> Flash Mode -> QIO
- Select Tools -> Flash Size -> 4MB
- Select Tools -> Partition Scheme -> minimal SPIFFS (1.9MB APP with OTA/190KB SPIFFS)
- Select Tools -> PSRAM -> Enabled
- Select Tools -> Upload Speed -> 921600
- Select Tools -> open serial monitor @ 115200 baud
- connect your T-Beam
- Select Tools -> Port -> (select accodingly)
- compile/upload

### Arduino IDE settings for T-Beam S3 Supreme (Caution: the UF2 firmware upload option will no longer work after the following steps)
- based on https://github.com/Xinyuan-LilyGO/LilyGo-LoRa-Series?tab=readme-ov-file#3%EF%B8%8F⃣arduino-ide-quick-start
- Install "esp32 by Espressif" version 2.0.9 under "BOARDS MANAGER" 
- Select Tools -> Board -> ESP32S3 Dev Module
- Select Tools -> USB CDC on Boot -> enabled
- Select Tools -> CPU Frequency -> 240MHz
- Select Tools -> USB DFU on Boot -> disabled
- Select Tools -> Flash Frequency -> 80MHz
- Select Tools -> Flash Mode -> QIO
- Select Tools -> Flash Size -> 8MB
- Select Tools -> USB Firmware MSC on Boot -> disabled
- Select Tools -> Partition Scheme -> 8M with spiffs (3MB APP/1.5MB SPIFFS)
- Select Tools -> PSRAM -> QSPI PSRAM
- Select Tools -> Upload Mode -> USB-OTG CDC (TinyUSB)
- Select Tools -> Upload Speed -> 921600
- Select Tools -> USB Mode -> USB-OTG CDC (TinyUSB)
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

### Arduino IDE settings for T-Echo and T1000-E
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
