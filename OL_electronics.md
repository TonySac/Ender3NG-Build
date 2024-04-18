---
title: "Electronics :building_construction:"
layout: default
nav_order: 6
has_children: true
---

:construction: 
I haven't got this far yet...
:building_construction: 

# Electronics

---

The official documentation does not include information on wiring or installing electronics. The following pages cover the electronics and wiring of my personal E3NG. It should not be considered a standard. It is provided for personal reference and inspiration only.

In addition to the standard printer electronics (steppers, thermistors, & heaters) my build makes use of the following specific components:

* BTT SKR Mini E3 V3 Mainboard
* Raspberry Pi 3A+
* SN65HVD230 CAN Tranciver
* BTT EBB36 CAN Toolhead Board
* LEDs

{: .mod }
:wrench: I skipped the Klicky probe in favor for the CR Touch I already have on hand. I also did not install any endstops as I'm using sensorless homing.

# Mounting Electronics

<img src="/assets/electronics.png" width="300">{: .float-right}{: .ml-3}

I don't feel like there is any "right" way to mount the electronics in the rear of the printer. I used a combination of the included DIN clips and some leftover clips from my Voron build.

The large power supply only fit on the lower DIN rail. I was unsuccessful using `PSU3` and ended up using the standard `DIN1` clip with enlarged holes. In the future I would like to add a buck converter to this lower rail to power the Raspberry Pi. I also need to figure out a more stable way to secure the DIN rail itself.

The upper DIN rail is home to the Raspberry Pi, mainboard, and a CAN transceiver.  I used the DIN clips and brackets from my Voron build, simply because they were already printed and on hand.

# Toolhead Electronics

As previously stated, I'm running a Voron Stealthburner for the toolhead. In addition to the usual comonents (fans, thermistor, and heater) it has LEDs. I'm also reusing the CR Touch from the Ender 3. Eventually I will upgrade to a newer bed probe and maybe even change the toolhead. The big thing to remember with toolhead electronics is to route the wiring an a way that it does not jam up any motion components.

