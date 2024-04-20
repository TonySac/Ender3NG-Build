---
title: "Electronics"
layout: default
nav_order: 6
has_children: true
posted: 2024-04-17
---

# Electronics

---

The official documentation does not include information on wiring or installing electronics. The following pages cover the electronics and wiring of my personal E3NG. It should not be considered a standard. It is provided for personal reference and inspiration only.

In addition to the standard printer electronics (steppers, thermistors, & heaters) my build makes use of the following specific components:

* BTT SKR Mini E3 V3 Mainboard
* Raspberry Pi 3A+ :pencil:
* SN65HVD230 CAN Tranciver
* BTT EBB36 CAN Toolhead Board
* LEDs

{: .mod }
:wrench: I skipped the Klicky probe in favor for the CR Touch I already have on hand. I also did not install any endstops as I'm using sensorless homing.

{: .note }
:pencil: I upgraded to a Pi 4 that I had on hand. I ran into some bottlenecks with dev environments. The 3A should be fine for just a printer.

# Main Electronics

<img src="/assets/electronics.png" width="300">{: .float-right}{: .ml-3}

I don't feel like there is any "right" way to mount the electronics in the rear of the printer. I used a combination of the included DIN clips and some leftover clips from my Voron build.

The large power supply only fit on the lower DIN rail. I was unsuccessful using `PSU3` and ended up using the standard `DIN1` clip with enlarged holes. In the future I would like to add a buck converter to this lower rail to power the Raspberry Pi. I also need to figure out a more stable way to secure the DIN rail itself.

The upper DIN rail is home to the Raspberry Pi, mainboard, and a CAN transceiver.  I used the DIN clips and brackets from my Voron build, simply because they were already printed and on hand.

# Toolhead Electronics

As previously stated, I'm running a Voron Stealthburner for the toolhead. In addition to the usual components (fans, thermistor, and heater) it has LEDs. I'm also reusing the CR Touch from the Ender 3 and running a BTT EBB36 board. Eventually I will upgrade to a newer bed probe and maybe even change the toolhead. 

<img src="/assets/cr_mount.png" width="150">{: .float-right}{: .ml-3}

I made some modification to the CR Touch hosuing and used [this](https://www.printables.com/model/709806-ender-3-ng-vocano-voron-bltouch-stealthburner) mount to attach the Stealthburner to my E3NG. The mount was originally designed for a BL Touch so I needed to shave down and remove some of the CR Touch housing. I also had to add some spacers to the mount.

{: .note }
:pencil: A more elegant solution would be to redesign a mount that fits the probe correctly. With my currently subpar CAD skills and plans to upgrade the probe, I went with my modification instead. 

The big thing to remember with toolhead electronics is to route the wiring an a way that it does not jam up any motion components. The EBB36 CANbus toolhead board helps in this regard. I used one of the many EBB36 mounts found online for the Stealthburner/Clockwork2 then routed and secured any wires.

# Bed Electronics

I reused the stock Ender 3 bed including heater and thermistor. This just required routing the existing cables. Future plans include adding some sort of quick disconnect, maybe WAGOs, to make the bed easier to remove and a cable chain.

# Switches and LEDs

In lieu of limit switches, I used sensorless homing. 

I'm still mapping out my plans for LEDs.