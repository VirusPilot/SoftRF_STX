# SoftRF DIY - Stratux compatible fork

- enable SoftRF to work as a proper GPS and Baro source for Stratux (through USB or WiFi)
- option to enter Aircraft ID
  - if an Aircraft ID is added, then ADDR_TYPE_ICAO is set for both Legacy and OGN
  - if the SoftRF factory ID remains, then ADDR_TYPE is set according to the selected protocol

**IMPORTANT**: All modifications are provided only in the source code so you need to be familiar with Arduino to compile and flash it for your platform. You need to install Arduino IDE (v1.8 or later) and add the following two entries into the Additional Board Manager URLs:
- T-Beam: `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`
- T-Echo: `https://adafruit.github.io/arduino-board-index/package_adafruit_index.json`

For the T-Beam you need to select the `ESP32 Dev Module` board, for the T-Echo the `Nordic nRF52840 DK` board. For more details please read the following related upstream WiKi sections: https://github.com/lyusupov/SoftRF/tree/master/software/firmware/source#esp32 and https://github.com/lyusupov/SoftRF/tree/master/software/firmware/source#nrf52840

In case you want to convert a T-Beam based OGN Tracker to run SoftRF, you first need to apply the following reset: https://github.com/VirusPilot/LilyGo-T-Beam-GPS-Reset, otherwise the GPS chipset won't work with SoftRF (OGN Tracker uses 57600 baud vs. SoftRF using 9600 baud for the GPS-CPU connection).

**T-Beam modifications:**
- u-blox GPS configuration:
  - enable GSA, GSV, VTG
  - enable GPS, GALILEO, BEIDOU and SBAS
  - enable NMEA extended protocol
- LEGACY NMEA traffic messages are disabled (to relax data rate, Stratux receives LEGACY directly anyhow)
- default connection with Stratux: **USB** (115200 baud), the USB T-Beam connection with Stratux works best if `init_uart_baud=115200` is added to the `/boot/config.txt` file on the Raspberry Pi
- WiFi AP IP changed to `192.168.4.1` to avoid conflicts with Stratux WiFi AP IP
- alternative connection with Stratux (under development): **WiFi**
  - Stratux needs to be in `AP+Client mode` to connect to the SoftRF WiFi AP (please add your SoftRF's SSID and password to the list of `WiFi Client Networks` in the Stratux WiFi settings)
  - T-Beam `NMEA output` needs to be set to `TCP`, using the SoftRF webinterface
  - to be implemented on Stratux: automatic port fowarding in Stratux, for now `socat TCP:192.168.4.1:2000 TCP:localhost:30011` is doing the trick but not reliable yet

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
- it appears that on SX1262 based T-Beams the modified GPS configuration sometimes reverts to the default GNSS settings, e.g. GLONASS is enabled instead of BEIDOU. It is totally unclear why this happens, therefore SX1276 based T-Beams are recommended for the time being

**Recommendations for T-Beam:**
- modify SoftRF settings (https://github.com/lyusupov/SoftRF/wiki/Settings), using the SoftRF webinterface

**Recommendations for T-Echo:**
- load OGN database: https://github.com/lyusupov/SoftRF/wiki/Badge-Edition.-Aircrafts-database
- modify SoftRF settings (https://github.com/lyusupov/SoftRF/wiki/Settings) by **downloading** the following scripts, **opening** them in a browser to generate the appropriate $PSRFC and $PSKVC sentences and then **sending** these generated sentences to the SoftRF device via a serial USB console (e.g. Arduino IDE comes with a nice built in serial USB console):
  - https://github.com/VirusPilot/SoftRF/blob/master/software/app/Settings/basic.html (e.g. Protocol, Aircraft type, Aircraft ID)
  - https://github.com/VirusPilot/SoftRF/blob/master/software/app/Settings/ui.html (e.g. Units, e-Paper 'ghosts' removal)
