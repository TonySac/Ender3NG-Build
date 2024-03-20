---
title: Printing Parts
layout: default
nav_order: 2
parent: Sourcing Parts
---

# Printing Parts

I printed most of my parts on my Voron 0.2. However, since the build volume is so small, some of the larger parts didn't fit. I used my Ender 3 Neo to crank out some of the larger parts.

{: .alert }
Make sure to print everything prior to tearing down your only working printer.

A specialized calibration cube is included in the project files to confirm tolerances. I printed one off on both my Voron and Ender 3 prior to printing the functional parts.

## Materials

<img src="/assets/voron0.png" width="200" />
{: .float-right}

I chose to print all of my parts in ABS. ABS along with ASA are the recommended materials. I didn't have any experience with ASA. I was pleased with the ABS parts I received for my previous Voron build and I figured if ABS is good enough for Voron, it will be good enough here.

Specifically, I used *Sunlu Black ABS* for the majority of the primary parts and *Fusion Filaments Seismic Red ABS 1.5* for the accent parts. These are the same colors and filaments used on my Voron 0.2 and I wanted to keep with the color scheme.

## Print Settings

I made some slight modifications to the suggested print settings. Below is a comparison of the E3NG settings and mine:

|                  |E3NG Settings      | My Settings   |
|------------------|-------------------|---------------|
Layer Height:      | 0.20 mm           | 0.20 mm       |
Infill:            | 30%               | 40% (Cubic)   |
Walls:             | 4                 | 4             |
Top/Bottom Layers: | 5                 | 5             |
Extrusion Width:   | Not specified     | 0.40 mm       |
Supports:          | :x:               | :x:           |
Brim:              | Not specified     | As needed     |

All of the parts were sliced in [OrcaSlicer](https://github.com/SoftFever/OrcaSlicer). My printers and filaments were tuned in accordance to [Ellis' guide](https://ellis3dp.com/Print-Tuning-Guide/).

<img src="/assets/main-parts.png" />

For reference, the below 3MF files include the printer, filament, and process settings used on my Voron 0.2.

* [Voron 0.2 Main Part Project File](https://github.com/TonySac/Ender3NG-Build/blob/main/Orca/E3NG_Main_Parts-Voron.3mf)
* [Voron 0.2 Accent Part Project File](https://github.com/TonySac/Ender3NG-Build/blob/main/Orca/E3NG_Accent_Parts-Voron.3mf)

### Printing ABS on the Ender

For the parts unable to fit in the Voron 0.2, I used my Ender 3 Neo.

Full disclosure, my Ender 3 Neo is not stock. Some of the notable changes are:

* BTT SKR Mini E3 V3 Mainboard
* Klipper Firmware
    * Input Shaping
* Upgraded NEMA 17 Steppers
* Stealthbuner Toolhead
    * Clockwork 2 Direct Drive Extruder
    * Phaetus Dragonfly BMO Hotend
* Textured PEI Plate

Below are the 3MF files the include the settings used on my Ender 3. 

* [Ender 3 Main Part Project File]()
* [Ender 3 Accent Part Project File](https://github.com/TonySac/Ender3NG-Build/blob/main/Orca/E3NG_Accent_Parts-Ender.3mf)

I managed to print these parts without an enclosure. Hairspray and a brim was used to aid with bed adhesion. I did have some mild warping of larger electronics panel parts. I don't foresee this causing issues during assembly as they do not appear to be structural in nature. I will likely reprint them down the road.