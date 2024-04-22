--- 
title: "Wiring"
layout: default
parent: "Electronics"
posted: 2024-04-22
---

# Wiring
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

Once again, the wiring outlined here is my how I wired up my E3NG. It includes additional hardware for Klipper and CANbus. It is provided for my reference and inspiration.

{: .warn }
:warning: Incorrectly wiring components can cause damage to them or at worst personal injury.

Wires should be routed in a clean manner. This means out of the way of motion components and not pinched, kinked, or otherwise damaged. Always use the correct wire gauge. Securely crimp all connections and use electrical tape and/or heat shrink to prevent shorting. Standard color conventions are especially important for power wires. Label wires as needed.

I provided a table below of the gauge wires I used for various components on the build. The bigger AWG, the smaller the wire.

| Component       | AWG   |
|-----------------|-------|
| AC Power        | 16-18 |
| DC Power        | 18    |
| Heaters         | 18-20 |
| Thermistors     | 22    |
| Fans            | 22    |
| Steppers        | 22    |
| Bed Probe       | 22-24 |
| CANbus Comms.   | 22    |

# Mainboard Wiring

The BTT GitHub provides a pinout and schematics of the [SKR Mini E3 v3](https://github.com/bigtreetech/BIGTREETECH-SKR-mini-E3/tree/master). I have included a copy of the pinout for reference. Using this diagram, wiring the board is pretty straight forward. Make sure to use the correct pinout for the mainboard in use.

<img src='/assets/skr_pinout.png'>

{: .tip }
:bulb: Pay attention to the pin names using during wiring. Example: `PC9` on the `HB` terminal for the heated bed. These pin names are used in the firmware to control the components.

## Power

I wired the 24v power supply directly to the `24v` and `GND` terminals labeled `POWER` on the mainboard. I used red and black 18 ga wire and ferrules. Pay close attention to polarity

## Heated Bed

The bed heater was also directly wired to the `HB` terminal using ferrules and the same wire from the stock Ender 3 bed. The polarity of the bed is not important. In the future, I would like to add a quick disconnect system at the bed side.

The bed therminstor was also wired directly to `THB` using the original wiring. I did shorten the cable and crimp a new JST connector to the end. Once again, polarity does not matter and I plan to add a disconnect at the bed side.

## Extruder

The extruder heater and thermistor are wired the same as the heated bed but use `E0` and `TH0`. 

## Fans

Fans can be wired to any of the 3 JST terminals. Make sure the polarity is correct and the pin name is noted. 

{: .tip }
:bulb: If using the stock Ender 3 hotend blower fan with the blue/yellow wires, blue is `GND` and yellow is `24 VIN`.

## Endstops

Endstops can be wired to their respective location using a JST connector. Polarity is not as important as it can be corrected in the firmware.

### Sensorless Homing

If using sensorless homing, no endstop will be plugged in but rather the `DIAG` pins will need to be jumped. 

## Neopixel

LEDs can be plugged into the `Neopixel1` port in accordance to the pinout for control in the firmware.

## Z-Probe

The CR and BL Touch probe should be a drop in fit for the SKR Mini E3 v3. However, verify the pinout for all probes prior to connecting.

## EXP1 Header

The stock Ender 3 display cable can be used here to power the stock digital style display with Klipper. The color LCD displays do not work out of the box.

### CAN Tranciver

The EXP1 header exposes pins `PB8` and `PB9` which are the easiest to access for CANbus. I'm running the SKR Mini E3 in USB to CAN bridge mode so I built a custom 2x3 DuPont connector to utilize `PB8`, `PB9`, `+5V`, & `GND` and connect to a SN65HVD230 CAN tranciver. The table below shows the corrsponding EXP1 pins and CAN tranciever pins.

| EXP1 | CAN  |
|------|------|
| +5V  | 3.3V | 
| GND  | GND  |
| PB8  | RX   |
| PB9  | TX   | 

## USB

It's a MicroUSB port to plug in the Raspberry Pi... The included USB cable should be sufficient.

## Stepper Motors

If using the stock Ender 3 motors and stock wiring harness the motors should pretty much be plug and play. If this is not the case, the motor windings need to be identified. The BTT SKR Mini E3 v3 expects one motor winding on the first two pins of the connector and the other winding to be on the last two pins.

Use a multimeter to find the two motor pins with continuity. This can be identified as winding A which means the remaining two pins make up winding B. Crimp a 4 pin JST connector so the first two pins are one of the windings and the second 2 pins are the other winding. The exact location of each wire/winding is not as important since it can be corrected in the firmware.

### E & Z Motors

The extruder and Z axis motors can be plugged into their respective connectors. `EM` for the extruder and `ZAM` for the Z axis. 

{: .note }
:pencil: I believe `ZBM` would work as well since it is the same motor driver and pinout. My understanding is it allows for a second Z motor to be controlled on the same driver.

### A/B Motors

I plugged the A motor, left side motor if looking at the E3NG head on, into `XM`. That leaves `YM` for the B Motor. I'm not 100% sure if that is the correct way but it works so I have no complaints.

# EBB36 v1.2 Toolhead Wiring

I use BTT's EBB36 Toolhead board. Once again, schematics and pinout can be found on [GitHub](https://github.com/bigtreetech/EBB) and I included a screen shot of the pinout below.

<img src='/assets/ebb_pinout.png'>

Regardless of USB or CAN communication, the EBB36 requires 24v power. I tapped directly off the main power supply and crimped the included 2x2 Molex Mircofit connector. 

{: .warn }
:warning: The pinout diagram and be a bit confusing. When orienting the EBB36 as shown in the diagram, the rear two Molex pins are `GND` and `VIN` respectively. Miswiring the board can send it up in smoke.

## CANbus Wiring

The `CAN-L` and `CAN-H` pins should be wired directly to the `CANH` & `CANL` pins on the CAN tranciever. On the EBB36, these pins can be accessed either via the Molex connector or the two DuPont pins below it.

CANbus wire harness are available from a number of suppliers. The CAN high and CAN low wires should be a twisted pair. The power wires should not be twisted. Since this was a small run, I twisted my own 22 ga wire and secured it intermittently with zip ties then tucked it into a cable loom next to the 18 ga power wires with everything terminating into the 2x2 Molex Microfit connector at the toolhead. 

### Terminating Resistor

The CAN network requires two 120 &#937; resistors, one at each end of the network. My network starts at the CAN tranciever which happens to have a built in terminating resistor. Most of the time the EBB36 will be the last, or only other, node on the network. A jumper must be placed across the `120R` pins to enable the 120 &#937; terminating resistor.

## E Motor

Plug the extruder motor into this connector. Review the information on stepper motor windings in the section [above](/electronics-wiring.html#stepper-motors).

{: .tip }
:bulb: The EBB36 includes a heatsink that should be placed on the TMC2209 driver. That is the chip just below this connector.

## RGB

Any toolhead LEDs can be plugged in here. Pay attention to the power vs. communication pins.

## Endstop

I used this connector for the BL touch. The `Probe` header would have been a drop in fit but required a DuPont connection. I preferred a more secure JST fitting. Just remember to order the pins correctly and this header uses JST 1.5. 

## Hotend

The thermistor plugs right into `TH0` and the heater cartrige used ferrules to screw into `Hotend 0`. The polarity is not important for these components.

## Fans

The part cooling and hotend fans plug into the `FAN1` & `FAN2` ports. It doesn't matter which goes where as this can be addressed in firmware.

## Other Pins/Components

I am not using the additional headers at this time or the MAX31865 temperature sensor. Setting up the integrated ADXL will be covered in the [configuration section](/software_configuration.html).