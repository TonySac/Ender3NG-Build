---
title: "Software Configuration"
layout: default
nav_order: 8
has_children: true
posted: 2024-05-07
---

# Software Configuration
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

{: .notice }
:loudspeaker: These steps assume the Klipper host is running and accessible from the local network.

This section covers configuring Klipper for the Ender 3 NG. Specifically, it covers the configuration of a BTT SKR Mini E3 v3 in USB to CAN bridge and a EBB 36 Toolhead board.

The Klipper GitHub contains [configuration examples](https://github.com/Klipper3d/klipper/tree/master/config) for not only these boards but also some generic configs and resources for other control boards. These provide a good starting point for the configuration. Additionally, my [personal Ender 3 NG config files](https://github.com/TonySac/Ender3NG-Config) are hosted on my GitHub.

{: .note }
:pencil2: Neither my config nor the example configs are plug and play. Some validation and updating will be required for any machine.

After experimenting with Klipper on my Ender 3 Neo and building a Voron 0.2, I pulled from those configuration files and the [Klipper Configuration Reference](https://www.klipper3d.org/Config_Reference.html) to generate my own E3NG config.

> My old [Ender 3 Neo config](https://github.com/TonySac/Ender-3-Neo-Config) and current [Voron 0.2 config](https://github.com/TonySac/Voron0-Config) can also be found on my GitHub.

# Editing the Config Files

The Klipper config files are stored on the host system in `~/printer_data/confg`. The files can be accessed from the Mainsail web interface under the `Machine` tab. They can also be edited directly on the host or remotely via SSH and tools like VSCode. A simple `Save & Restart` immediately applies changes to the firmware.

Comments in the config code are superseded with `#`. Additional files can be placed in the `~/printer_data/confg` directory with a *.cfg* extension and included with an `[include additional.cfg]` line in *printer.cfg*. This is useful for larger custom configurations and macro files. When pins are defined special characters can be used. Prepending a pin with `!` inverts the pin polarity. If so equipped `^` enables a hardware pull-up resistor and `~` a hardware pull-down resistor. See some examples below.

```yaml
#This is a comment followed by some examples

[include macros.cfg] #This file is included as if it were part of the config

dir_pin: PB2 #This tells the motor to spin in the default direction
dir_pin: !PB2 #This reverses the motor's default direction

endstop_pin: ^PC1 #This enables the hardware pull-up resistor
```

# Getting Started

The Klipper web interface can be accessed in a web browser using the host's IP address. In this case, I'm using Mainsail.

{: .tip }
:bulb: It's a good idea to set the Klipper host up with a static IP.

Assuming Klipper was properly flashed and and the MCU path updated during [software installation](/software_install.html) the Mainsail interface should load up with a basic screen and no way to control the printer.

<img src="/assets/new-mainsail.png"/>

In the result an error is reported regarding connection to the MCU, it's likely the MCU path is incorrect.

Navigate to `Machine` then `printer.cfg`.

If the printer is connected to the host via USB, the `mcu` line should read:

```yaml
[mcu]
serial: /dev/serial/by-id/usb-Klipper_Klipper_firmware_12345-if00
```

Where the correct `/serial/by-id/` path is found by running `ls /dev/serial/by-id/*` on the host machine.

If USB to CAN bridge mode is used, then the `mcu` line should be similar to

```yaml
[mcu]
canbus_uuid: 027a9447a345
```

The `canbus_uuid` can be found by running `~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0` on the host machine.

If the problem persists, troubleshoot the wiring and Klipper installation.

Once the interface is accessible, building the configuration can begin.