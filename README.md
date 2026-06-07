![Logo](/readme_images/Flag.png)
### Canadian-designed open source hardware. For everyone.

## Flexi-Pi

Flexi-Pi is a custom configured LinuxCNC image for use with the [Flexi-HAL](https://github.com/Expatria-Technologies/Flexi-HAL) and [FlexiHAL 2350](https://github.com/Expatria-Technologies/FlexiHAL_2350) CNC controllers. The LinuxCNC components required for both boards are pre-installed along with all of their dependencies. Reference configurations are included to get you started. **You will need to edit these configurations for your specific machine before attempting motion.** The base configuration runs QtDragon_hd as the default supported UI, which requires a 1080p display. 

Both the Raspberry Pi 5 and Raspberry Pi 4 are currently supported. Performance is significantly better with a Pi 5, however the Pi 4 image here performs better than the previous releases. 

**To get started:**
* Download the release for your specific Pi board (Pi 4 or Pi 5)
  https://github.com/Expatria-Technologies/Flexi-Pi/releases
* Extract the image archive
* Flash the image to an an 8GB or larger SD card using dd, balenaEtcher, or the tool of your choice. The filesystem will be automatically resized on first boot to fill the entirety of the SD card.
* Install the Pi onto the Flexi-HAL headers and insert the SD card into the Pi. You will need to use an extension header if you are using a cooler. 
* If you are using a Flexi-HAL, ensure the jumpers are set as shown in the Flexi-HAL Configuration section of this README.
* Connect your display, keyboard, etc. to the Pi.
* Connect power to the Pi/Flexi-HAL and boot the OS.
* Read the contents of the README present on the desktop for instructions specific to the image used. This includes details on flashing firmware for your board and starting LinuxCNC.

The default username and password are both `expatria`.

A high-endurance SD card from a reputable manufacturer is recommended. The Sandisk U3 High Endurance cards have performed well in our testing. Select a card with a high read/write speed for the best performance. 


While not directly used by our boards, also pre-installed in this image are the standard Remora-spi and Remora-eth-3.0 components. These are included on an as-is basis, but are updated with each image build. If we have missed an update and you'd like to use this image, please let us know and we will update them as needed.

### Booting the Pi 5 from an NVMe Drive

If you wish to use an NVMe drive instead of an SD card, you will need to update the boot order in the Pi's EEPROM. You can do this from the Flexi-Pi image running on an SD card. To update the boot order in EEPROM:

* Flash the Flexi-Pi image to an SD card
* Insert the SD card into the Pi and boot
* **Ensure the Pi has a stable source of power for this operation.**
* Open a terminal and enter the command `sudo rpi-eeprom-config --edit`
* Change the 'BOOT_ORDER' line to 'BOOT_ORDER=0xf416' to boot from NVMe first, and fall back to SD if not present.
* Add 'PCIE_PROBE=1' to a new line below the 'BOOT_ORDER' line above
* Exit the editor/save the file. The EEPROM will then program automatically.

After editing your EEPROM config should contain:

```
[all]
BOOT_UART=1
POWER_OFF_ON_HALT=0
BOOT_ORDER=0xf416
PCIE_PROBE=1
```

You can then write the Flexi-Pi image to an NVMe drive using a USB NVMe Dock and your PC, or by downloading the image to the Pi while it is running from the SD card and writing it to your NVMe drive using `dd` or similar. 

We suggest avoiding NVMe hats that install on or otherwise interfere with the GPIO pins. The [Pimoroni NVMe Base](https://shop.pimoroni.com/products/nvme-base) has been tested to work, and mounts to the underside of the Pi which works well with the Pi mounted on the Flexi-HAL.

## Flexi-HAL Configuration
Jumpers need to be in place in the marked locations on the Flexi-HAL for the Pi to communicate with the MCU for firmware flashing:

<img src="https://github.com/Expatria-Technologies/remora-flexi-hal/raw/master/Images/Jumper_locations.png" width="500">

These are shipped in place by default from the Expatria shop, but if they have been moved or removed they will need to be replaced. 

To provide sufficient clearance for a cooler on a Pi 5, a 40-pin female header can be used as a riser between the Flexi-HAL and the Pi. 

 
## Getting started with LinuxCNC:

Connect 24V power to your board and a suitable USB-C to your Raspberry Pi. Do not connect anything to the USB-C port on the board while flashing over the GPIO header. Doing so may prevent the bootloader from functioning.

### Flashing Firmware

Use the `flash_firmware` script to flash firmware on either board. It will detect your hardware and guide you through the process. The script will check for updated releases and prompt you to download them if desired.

For **FlexiHAL 2350** (Flexi-Ninja firmware):
  - Flashes the RP2350 main firmware over SWD via the Pi GPIO header
  - Optionally flashes the RP2040 FlexGPIO bootloader over SWD
  - Optionally nukes the flash on both the RP2350 and RP2040 prior to programming. If you had a previous configuration in flash, this may be helpful in ensuring you start from scratch.

For **Flexi-HAL** (STM32F4):
  - Flashes the STM32F4 over UART via the Pi GPIO header

---

### Reference Configurations

#### Flexi-HAL (SPI)

  `~/linuxcnc/configs/flexi-hal/`

  A base PrintNC configuration is provided. Edit the INI with your machine configuration prior to attempting motion. This base configuration includes QtDragon_hd and vfdmod; vfdmod is configured for the Durapulse GS10 VFD, but can be changed to suit any Modbus VFD. A sample HAL configuration for the HY VFD is included, commented out.

#### FlexiHAL 2350 (SPI)

  `~/linuxcnc/configs/flexi-ninja/`

  This is a QtDragon HD based configuration, similar to the PrintNC configuration provided for the Flexi-HAL. Edit `flexi-ninja.ini` for your machine configuration before attempting motion. This base configuration includes QtDragon_hd and vfdmod; vfdmod is configured for the Durapulse GS10 VFD, but can be changed to suit any Modbus VFD. A sample HAL configuration for the HY VFD is included, commented out.

  New options vs. the Flexi-HAL firmware:
  - Stepgen pulse width is configurable via `flexi-ninja.stepgen.pulse-width`
  - DIR setup timing is configurable per axis via `flexi-ninja.stepgen.{N}.dir-setup`
  - Two encoders: ENC1 (high-speed PIO-based) and ENC2 (IRQ-based, can be disabled to free its pins as AUXIN0-2)
  - FlexGPIO I/O expander firmware is loaded to RAM at startup; no configuration needed
  - Full README: https://github.com/Expatria-Technologies/flexi-ninja

#### FlexiHAL 2350 (Ethernet)

  `~/linuxcnc/configs/flexi-ninja-eth/`

  Same configuration as SPI but communicates over Ethernet. The board is reachable at `192.168.5.1` as configured in the reference config.


#### uFlexiNet (Ethernet for Flexi-HAL)

  `~/linuxcnc/configs/uFlexiNet/`

  The remora-eth component is pre-installed. Upload your configuration file to the FlexiHAL using `upload_config.py` from your config folder:

    python3 upload_config.py FlexiHAL-config.txt

  To use the RS485 interface on the FlexiHAL via uFlexiNet, set up a virtual serial port with socat:

    socat pty, link=/tmp/virtualcom0, raw udp:10.10.10.10:27183 &

  More information is available at:
  https://github.com/Expatria-Technologies/Remora-STM32F4xx-W5500

---

### Ethernet Connectivity

The image is configured for DHCP on eth0 by default, with secondary static IP addresses of `192.168.5.2/24` for FlexiHAL 2350 Ethernet communication and `10.10.10.2/24` for Remora uFlexiNet. These can remain present regardless of which board you are using, and won't need changing unless you need a different default network config. Network configurations (including wifi) can be managed with `menu-config`.

---

### Autostart LinuxCNC

If you'd like to autostart LinuxCNC on boot after configuring your machine, add `linuxcnc -l` to 'Application Autostart' in the 'Session and Startup' menu to automatically load the configuration you last selected in the LinuxCNC selection window.

---

## Support

Support is provided primarily through our Discord Server: https://discord.gg/bSEXy7GhcZ. If you need help getting started or with configuration, please feel free to reach out there. 

---

## Original Sources

This is based on the [rpi-img-builder](https://github.com/pyavitz/rpi-img-builder), which was then customized for [LinuxCNC](https://github.com/LinuxCNC/rpi-img-builder-lcnc), and finally modified for Flexi-HAL and FlexiHAL 2350. 

---  

## System Menu: `menu-config`

<img src="https://i.imgur.com/vwFVBzF.png" alt="Main Menu" />  


## Re-configuring and Building on Ubuntu 24.04:


### Currently supported boards

* **Raspberry Pi 5** bcm2712 / ARM64

* **Raspberry Pi 4/400** bcm2711 / ARM64


#### Install dependencies

  

```sh
make ccompile # Install x86-64 dependencies
make ncompile # Install Aarch64 dependencies
```

#### Building existing configuration

```sh
make commit board=bcm2712 # build RT patched kernel for Pi 5
make rootfs board=bcm2712 # build rootfs archive
make image board=bcm2712 # build image for Pi 5
```


#### Menu interface

  

```sh

make config # Create user data file
make menu # Open menu interface
make dialogrc # Set builder theme (optional)

```

  

#### Command list

  

```sh

make list # List boards
make all board=xxx # Kernel > rootfs > image
make kernel board=xxx # Builds linux kernel package
make commit board=xxx # Builds linux kernel package
make rootfs board=xxx # Create rootfs tarball
make image board=xxx # Make bootable image

```
  

#### Miscellaneous

  

```sh
make clean # Clean up rootfs and image errors
make purge # Remove source directory
make purge-all # Remove source and output directory
make commands # List more commands
make check # Shows latest revision of selected branch

```

---
