---
title: "Software Installation"
layout: default
nav_order: 7
has_children: true
posted: 2024-04-15
updated: 2024-04-22
---

# Software Installation
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

I already use Klipper with Mainsail on my current printers so it was a natural choice for this project. My installation is running on a Raspberry Pi. Since I prefer to start with a bare bones system and cherry pick what I need, I flashed Raspberry Pi OS Lite then used [KIAUH](https://github.com/dw-0/kiauh) for a "manual" install.

{: .tip }
:bulb: I have a tutorial for [Baking a Raspberry Pi](https://themakermedic.com/posts/Pi-Baking_a_Pi/) in my other blog.

Since so much information already exists between the [KIAUH](https://github.com/dw-0/kiauh) docs along with the [Klipper](https://www.klipper3d.org) and [Mainsail](https://docs.mainsail.xyz) pages I suggest referring to them for more details on the installation process and methods. Here I will outline some basic steps for my general software install and do a deeper dive into things like CANBus and other unique features of my build.


# Klipper Host

The first step is to get a Klipper host up and running. I'm running a bare bones install of Raspberry Pi OS Lite on a Raspberry Pi. Using [KIAUH](https://github.com/dw-0/kiauh), I installed Klipper, Moonraker, and Mainsail. Klipper and Moonraker are mandatory but the frontend, in my case Mainsail, can be different. KIAUH also includes some other software that can be installed at this time if desired. These installations can also be done manually by referencing the individual documentations.

## RPi as an MCU

The Raspberry Pi can also be used as a secondary MCU. This means the GPIO pins on the the Pi can be accessed and controlled by Klipper. Since we're already in the Pi, might as well set that up now.

Open the Klipper directory and set the MCU process to start before Klipper:

```console
cd ~/klipper/
sudo cp ./scripts/klipper-mcu.service /etc/systemd/system/
sudo systemctl enable klipper-mcu.service
```

Build the MCU firmware by running `make menuconfig` and selecting `Linux process`.

<img src='/assets/makemenu_linux.png'>

Once the firmware is built, flash it to the Raspberry Pi:

```console
sudo service klipper stop
make flash
sudo service klipper start
```

## Sonar

I also run [Sonar](https://github.com/mainsail-crew/sonar) on my build. Its a WiFi keepalive daemon and occasionally my Pi likes to drop connections. They also provide a straight forward install script.

{: .tip }
:bulb: I also found running the Pi on a 2.4 GHz only WiFi and with IPv6 disabled seems to be more stable.

# Mainboard (BTT SKR Mini E3 V3)

{: .mod }
:wrench: I am running my personal setup in a USB to CAN configuration. This this is covered [here](/software_install-CANBUS.html). The steps below outline a basic Klipper install.

The Klipper firmware needs to be compiled and flashed to the mainboard. On the host run the following:

```console
cd ~/klipper
make menuconfig
```

This opens the Klipper configuration tool. Make the selections in accordance with the mainboard in use. The Klipper GitHub contains a list of [example configs](https://github.com/Klipper3d/klipper/tree/master/config) for many mainboards along with the required settings. 

I've provided a screenshot of the standard SKR Mini E3 V3 settings using USB communications.

<img src="/assets/klipper_SKR_menu.png">

After setting and saving the required values clear out any previous firmware builds and generate the new firmware file.

```console
make clean
make
```

This will generate the firmware as a `klipper.bin` file in under the standard path of `~/klipper/out/klipper.bin`

The SKR Mini E3 V3 requires the firmware be flashed via SD card. Copy `klipper.bin` to a MicroSD card that has been formatted to FAT32. Rename `klipper.bin` to `firmware.bin`.

{: .tip }
:bulb: Copy the firmware with whatever method works best. I've used SCP, SFTP, Samba, and a direct mount in the past.

>I've had issues with 32GB Samsung MicroSD cards flashing the firmware. The cheap 8GB card that came with my Ender always works fine though.

Power off the printer mainboard, insert the firmware SD card, and power on the mainbaord. Wait several moments for the firmware to flash.

A successful flash can be confirmed in two ways.

1. Assuming the Pi is connected via USB to the SKR run:
```console
ls /dev/serial/by-id
```
An output similar to `usb-Klipper-stm32g0b1xx_#####################-####` should be generated.

     __*OR*__

2. Insert the SD card back into a computer and the `firmware.bin` file should have automatically changed to `firmware.cur`

Use the `ls /dev/serial/by-id` output in the `printer.cfg` file to tell Klipper where it can find the control board.

```console
[mcu]
serial: /usb-Klipper-stm32g0b1xx_#####################-####
```
With Klipper installed on the Pi and flashed to the mainboard. Its time to start the [configuration](/software_configuration.html).