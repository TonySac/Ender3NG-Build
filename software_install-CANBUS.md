---
title: "CANbus Software Installation :building_construction:"
layout: default
nav_order: 2
parent: "Software Installation"
posted: 2024-04-15
#updated: 2024-04-10
---

:construction: 
I haven't got this far yet...
:building_construction: 

# CANbus Software Installation
{: .no_toc }

<details closed markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

<img src="/assets/ebb36.png" width="200" />{: .float-right}{: .ml-3}

CANbus is communication protocol that only uses 2 wires. Most commonly in 3D printers it is used to connect a second MCU, often a toolhead board, to the main controller. This is exactly the use case I have set up. I am running a [BTT EBB36](https://github.com/bigtreetech/EBB) mounted on my toolhead. All of the toolhead electronics (hotend, thermistor, fans, LEDs, extruder motor, bed probe, etc.) are connected to that board. The board is then connected to my main MCU using 2 wires and a CAN transceiver. The board does also require 2 wires for power. So now all of my toolhead electronics and managed using only 4 wires back to my electronics case.

This is considered an advanced configuration. It is worth noting the EBB36 can be connected in USB mode and does not require the use of CANbus. Additionally, other boards are available and there are some other ways to run CANbus on a 3D printer. This covers using a BTT SKR Mini E3 V3 as a USB to CAN bridge along with a cheap CAN transceiver to communicate with the EBB36. Klipper then connects to all of these devices using CAN rather than USB.

> Special thanks to [Esoterical's CANBUS Guide](https://canbus.esoterical.online) and [Zippy's CANBus Your Pico](https://github.com/rootiest/zippy_guides/blob/main/guides/pico_can.md).

# Build the CAN Network

For Raspberry Pi OS set up the CAN interface with:

```console
sudo nano /etc/network/interfaces.d/can0
```

Then populate the file with the following block:

```yaml
allow-hotplug can0
iface can0 can static
  bitrate 1000000
  up ip link set can0 txqueuelen 1024
```

This sets up the CAN network with a bit rate of 1,000,000 as recommended in the Klipper docs and a transmit queue length of 1024. The Klipper docs recommended 128. Other guides recommended higher. I found 1024 works well.

Once these are saved, reboot the Pi.

# Flash the BTT SKR Mini E3 V3

{: .note }
:pencil: These steps are specific to the BTT SKR Mini E3 V3 mainboard. It will need to be adapted for any other device.

Make sure some python dependencies are installed:

```console
sudo apt install python3 python3-pip python3-can
```

## Build & Flash Katapult

I elected to install [Katapult](https://github.com/Arksine/katapult) on the SKR. This allows the CANbus network to be used to flash Klipper firmware. Normally, the SKR Mini E3 V3 must be flashed directly from the SD card slot. Katapult replaces the default bootloader to make this possible. However Katapult is not mandatory for CANbus. Skip ahead to [building and flashing Klipper](/software_install-CANBUS.html#build--flash-klipper).

{: .warn }
:warning: Overwriting the bootloader on the SKR Mini E3 V3 has the potential to brick the mainboard if done incorrectly. Proceed with caution.

Start by cloning [Katapult](https://github.com/Arksine/katapult) to the Pi.

```console
cd ~
git clone https://github.com/Arksine/katapult
```

Open the configuration tool and begin to configure the firmware.

```console
cd ~/katapult
make menuconfig
```

I have included a screenshot of the relevant settings. Once again, remember this is specific to the **BTT SKR Mini E3 V3**.

<img src='/assets/skr_katapult.png'>

Build save the configuration and build the new firmware/bootloader.

```console
make clean
make
```

The new bootloader will have been generated as `deployer.bin` under the path `~/katapult/out/deployer.bin`.

The bootloader must be installed from the SD card slot. Copy `deployer.bin` to a MicroSD card that has been formatted to FAT32. Rename `deployer.bin` to `firmware.bin`.

{: .tip }
:bulb: Copy the firmware with whatever method works best. I've used SCP, SFTP, Samba, and a direct mount in the past.

{: .note }
:pencil: I've had issues with 32GB Samsung MicroSD cards flashing the firmware. The cheap 8GB card that came with my Ender always works fine though.

Power off the printer mainboard, insert the firmware SD card, and power on the mainbaord. Wait several moments for the new bootloader to flash.

Assuming it was all done in accordance with the above screenshot, the status LED should start to slowly blink indicating "Katapult mode".

Additionally, running `ls /dev/serial/by-id` should show a Katapult device connected via USB.

```console
usb-katapult_stm36g0b1_########-####
```

Make note of the above device name as it will be needed to flash Klipper later.

## Build & Flash Klipper

Klipper needs to be built for the SKR Mini E3 V3 to communicate via CANbus. Open the Klipper configuration tool.

```console
cd ~/klipper
make menuconfig
```

Build the configuration for the BTT SKR Mini E3 V3 to communicate via `USB to CAN`. Make sure the bootloader offset is agrees with the Katapult install. Select a CANbus interface, `PB8/PB9` are pins easily accessible on the EXP1 header. Finally make sure the bit rate is the same as the previously configured CAN network. Once again, I have attached a screenshot of my configuration for a BTT SKR Mini E3 V3 for CANbus in bridge mode with Katapult.

<img src='/assets/skr_klipper-can.png'>

Save the configuration and generate the firmware.

```console
make clean
make
```

This will generate the firmware as a `klipper.bin` file in under the standard path of `~/klipper/out/klipper.bin`. It is now ready to be flashed [with Katapult](//software_install-CANBUS.html#flash-with-katapult) or [without Katapult](/software_install-CANBUS.html#flash-without-katapult).

### Flash without Katapult

If not using Katapult, the new firmware must be flashed via SD card. Copy `klipper.bin` to a MicroSD card that has been formatted to FAT32. Rename `klipper.bin` to `firmware.bin`.

Power off the printer mainboard, insert the firmware SD card, and power on the mainbaord. Wait several moments for the firmware to flash.

### Flashing with Katapult

If Katapult is installed on the mainboard, the Klipper firmware can be flashed directly without the need for and SD card. 

First stop the Klipper service.

```console
sudo service klipper stop
```

Navigate to and run the Katapult flash script:

```console
cd ~/katapult/scripts
python3 flashtool.py -d /dev/serial/by-id/usb-katapult_stm36g0b1_########-####
```

{: .tip }
:bulb: Make sure to use the correct `/dev/serial/by-id/` path for the Katapult mainboard as found when it was [previously flashed](/software_install-CANBUS.html#build--flash-katapult).

## Connect via CAN

The CAN network should be running. Assuming the network is `can0`, This can be verified with:

```console
ip -s -d link show can0
```

The output should show the details of the CAN network, including the bit rate and transmit queue length set previously.

The mainboard should also be visible as a CAN device with `lsusb`.

The CANbus query tool will discover the CANbus UUID of the mainboard.

```console
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```

This UUID will be used in the `printer.cfg` file to tell Klipper to use CAN for communication. Remove the `serial` line and add the `canbus_uuid` as it corresponds to your machine.

```diff
[mcu]
- serial: /dev/serial/by-id/usb-Klipper_Klipper_firmware_12345-if00
+ canbus_uuid: 027a9447a345
```


