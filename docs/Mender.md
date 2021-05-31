# ChargePi

ChargePi is an open-source Raspberry Pi 4 based OCPP client/Charging Point, which supports multiple EVSEs and
connectors. A charging connector consists of a WS2811 LED strip, a relay and a power meter.

ChargePi can be deployed/run in multiple ways:

- standalone
- Docker by building the image and running the container
- Docker-compose to be deployed with SteVe Central System and Watchtower
- Docker-compose by running the client service

### Charging station specifications

| Protocol implementation | Core functionalities | Offline charging | Local authorization | Charging profiles |
| :---:    | :---:    | :---:    |:---:    | :---:    |
| OCPP 1.6 JSON/WS | Yes | Yes | Yes | No |
| OCPP 2.0.1 JSON/WS | Will be implemented | Will be implemented | Will be implemented | No |

## Prerequisites

## Hardware support and schematics

This client supports the following hardware:

- Raspberry Pi 3B/4B
- Relay(s) 230V 10A
- MFRC522 or PN532 RFID/NFC readers
- Power meter (CS5460A chip)
- LCD (optionally with PCF8574 I2C module)
- WS281x LED strip

Hardware must be configured through _settings.json_ and _connetors.json_ files.

#### RFID/NFC reader pinout

##### MFRC522 RFID reader:

RFID reader must be on SPI bus 0.

| RPI PIN|  MFRC522 PIN    |  RPI PIN|   MFRC522 PIN    |
| :---:	| :---:	| :---:	| :---:	|
|   1    |   3.3v    | 19    |   MOSI    |
|   6    |   GND    |   21    |   MISO    |
|   24    |   SDA    |   22    |   RST    |
|   23    |   SCK    |   /    |  /    |

##### PN532 NFC/RFID reader:

PN532 reader shares the I2C bus 1 with the LCD.

| RPI PIN |   PN532 PIN    | 
| :---:	| :---:	|
|   2 or any 5V pin    |  VCC  |
|   14 or any ground pin    |  GND    | 
|   3 (GPIO 2)    |  SDA    |
|   5 (GPIO 3)    |  SCL    | 

#### LCD I2C pinout

LCD should be on I2C bus 1 with address 0x27.

To find your I2C address, follow these steps:

1. Download i2c tools:

   > sudo apt-get install -y python-smbus  i2c-tools

2. If needed, reboot.

3. Run the following command to get the I2C address:

   > sudo i2cdetect -y 1

| RPI PIN |   PCF8574 PIN    | 
| :---:	| :---:	|
|   2 or any 5V pin    |  VCC  |
|   14 or any ground pin    |  GND    | 
|   3 (GPIO 2)    |  SDA    |
|   5 (GPIO 3)    |  SCL    | 

#### Relay pinout

It is highly recommended splitting both GND and VCC between relays or using a relay module.

| RPI PIN |  RELAY PIN    | 
| ---	| :---:	|
|   4 or any 5V pin    |   VCC    | 
|   20 or any ground pin    |   GND    |  
|  37 (GPIO 26) or any free GPIO pin    |   S/Enable    |  

#### Power meter pinout

It is highly recommended splitting both GND and VCC between power meters.

| RPI PIN|  CS5460A PIN    |  RPI PIN |   CS5460A PIN    |
| :---:	| :---:	| :---:	| :---:	|
|   4 or 2    |   VCC    |  38 (GPIO 20)    |   MOSI    |
|   25 or any ground pin    |   GND    |   35 (GPIO 19)    |   MISO    |
|   Any free pin    |   CE/SDA    |   /    |   /    |
|   40 (GPIO 21)    |   SCK    |   /    |  /    |

#### WS281x LED strip pinout

| RPI PIN|  WS281x PIN    |  RPI PIN |   WS281x PIN    |
| :---:	| :---:	| :---:	| :---:	|
|   External 12V    |   VCC    |  32 (GPIO 12)    |   Data |
|   External GND   |   GND    |   /    |  / |

### Configuration and settings

To configure the ChargePi client, check out the **configuration.md** guide under __/docs__. Client comes with predefined
settings which require minimal configuration.

### Bill of materials

#### Enclosure and wiring

| Item| # | 
| :---:    | :---:    | 
| Schneider Electric Kaedra with 4 openings (or more) | 1x | 
| Schneider Electric Schuko outlet for Kaedra | 4x (depends on the electrical box)| 
| Terminal/crimp connectors | a lot | 
| 1,5 mm2 or any 10A rated wire | around 20m | 
| Schuko plug | 1 |

#### Electronics

| Item| # | 
| :---:    | :---:    | 
| ETIMat6 C10 | 1 | 
| ETIMat6 B6 | 1 | 
| Raspberry Pi 4 2GB | 1 | 
| 4-Relay module 230V 10A | 1 | 
| Huawei LTE modem | 1 | 
| CS5460 power meter | 4x (for each outlet) | 
| PN532 NFC/RFID reader | 1 | 
| WS281x LED strip | a few meters | 

#### Misc

| Item| # | 
| :---:    | :---:    | 
| M2,5 20mm screws | 4 |
| M2 or M2,5 6mm screws | 2 | 
| __optionally__ 3D printed DIN mounts | 2 |

### Graylog logging server

ChargePi uses [Graylog](https://www.graylog.org/) logging server for storing logs, so a logging server should be
up and running if you want the logs to be stored. Logs are sent through GELF UDP protocol to the logging server
at port 12201. The library used for logging is graypy.

Configure the **"log_server"** property in the _settings.json_ file with your logging server IP.

## Initial setup

1. Make sure you have this directory structure:

   > ChargePi/client/

2. If you wish to run SteVe on the same machine:

   > ChargePi/steve/

   *_When cloning Steve from GitHub, steve directory should be automatically generated._
   Replace SteVe's default Dockerfile with Dockerfile provided in ChargePi/steve/Dockerfile


3. Make sure the client folder has all the files and that the settings are correct.

4. Wire your hardware correctly according to the provided schematics.
   *[Useful reference for Raspberry Pi](https://pinout.xyz/)*
   
5. Be sure you have Docker
   installed. [Installing Docker on Pi](https://www.docker.com/blog/happy-pi-day-docker-raspberry-pi/)

6. After installing, restart the system or start Docker daemon:

   > sudo service start Docker

7. Be sure your Docker is installed and running:

   > docker info

## Complete charging station experience

For the complete charging station experience with Over-the-air (*OTA*) updates, minimal configuration and minimal
hassle, follow this guide.

We will use [Mender](https://mender.io/) as OTA service for updating Linux, Docker and OCPP Client. If you do not wish
to use Mender, skip the first 3 steps. For the OCPP server/central system, we will
use **[SteVe](https://github.com/RWTH-i5-IDSG/steve)**.

1. Sign up on Mender. If you want, host a Mender server yourself.
2. Follow this guide
   for [installing Mender to the Pi](https://docs.mender.io/get-started/preparation/prepare-a-raspberry-pi-device).
3. Add the Raspberry to the Devices list and test if Mender works.
5. Clone the SteVe repository to ChargePi/. It should automatically create steve folder.
   *Skip this step if you do not want the server to be hosted on the same machine. Mostly used for testing purposes.*
6. If you do not wish to automatically update the OCPP client, change this label to false in docker-compose.yaml:

   ```
     chargepi:
        label:
           - com.centurylinklabs.watchtower.enable="false"
   ```

   **Not recommended when using Mender, since you can update the images with Mender.**

7. Run docker-compose:

   > docker-compose up -d

* Optionally, you can run only the client service:

  > docker-compose up -d chargepi

## Helpful references

### WS281X LED library & tutorial:

- [LED library](https://github.com/jgarff/rpi_ws281x)
- [tutorial](https://tutorials-raspberrypi.com/connect-control-raspberry-pi-ws2812-rgb-led-strips/)

### Python Unsync library:

- [Unsync](https://asherman.io/projects/unsync.html)

### CS5460A library:

- [library](https://github.com/cbm80amiga/ST7789_power_meter_cs5460a_display/)
- [Datasheet](https://statics.cirrus.com/pubs/proDatasheet/CS5460A_F5.pdf)

### OCPP:

- [Steve](https://github.com/RWTH-i5-IDSG/steve)
- [Python library](https://github.com/mobilityhouse/ocpp)
- [1.6 specification](https://www.oasis-open.org/committees/download.php/58944/ocpp-1.6.pdf)
- [2.0.1 specification](https://github.com/mobilityhouse/ocpp/tree/master/docs/v201)

### Docker:

- [Installing Docker on Pi](https://www.docker.com/blog/happy-pi-day-docker-raspberry-pi/)
- [Watchtower](https://github.com/containrrr/watchtower)
- [Docker](https://docs.docker.com/)

### RPi:

- [Pinout](https://pinout.xyz/)

### Mender:

- [Docs](https://docs.mender.io/get-started/preparation/prepare-a-raspberry-pi-device)