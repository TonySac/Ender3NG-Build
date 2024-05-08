---
title: "Additional Configuration"
layout: default
nav_order: 2
parent: "Software Configuration"
has_children: true
posted: 2024-05-07
---

# Additional Configuration
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

This section covers some additional configuration features used on my printer. Although not mandatory for printing, they are quality of life improvements.

# Bed Probe

The whole point of the CR Touch is to create a bed mesh for auto leveling. Klipper also includes a feature to help with adjusting the bed screws.

## Bed Mesh

In order to generate a bed mesh in Klipper, the `[bed_mesh]` configuration must be defined. The example below is for my machine.

```yaml
[bed_mesh]
speed: 300
mesh_min: 0, -9
mesh_max: 223, 190
probe_count: 5,5
algorithm: bicubic
```

{: .note }
:pencil2: The `mesh_min` and `mesh_max` values are where the nozzle is located during the probing process. As such, the probe offset needs to be added into these coordinates and any machine constraints accounted for.

Additionally, I still use [KAMP](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging) for generating an adaptive mesh. More recent versions of Klipper have this feature built in but I have not gotten around to transitioning to it.

Don't forget to call or load the bed mesh when a print is started.

## Bed Screws

Rather than fiddling around with paper, `[screws_tilt_adjust]` can be configured. Then `SCREWS_TILT_CALCULATE` called in Mainsail for the probe to calculate the offset of each bed screw and the recommended adjustment. This configuration parameter is defined on my machine as follows:

```yaml
[screws_tilt_adjust]
screw1: 27, 12
screw1_name: Front Left
screw2: 195, 12
screw2_name: Front Right
screw3: 195, 181
screw3_name: Back Right
screw4: 27, 181
screw4_name: Back Left
screw_thread: CW-M4
```

{: .note }
:pencil2: Once again, the coordinates refer to the nozzle location. Adjust them with the probe offset in mind so the probe is directly over each bed screw.

# Input Shaper

Input shaping helps counteract some of the printer's vibrations to imporve print quailty at higher speeds. See the Klipper documentation on [Resonance Compensation](https://www.klipper3d.org/Resonance_Compensation.html) for more information.

It is defined under the `[input_shaper]` configuration parameter. Manual tests can be performed to calculate the associated values. However, I use an accelerometer, specifically and ADXL345 that is integrated on my EBB36 to do all of the work for me.

First the accelerometer needs to be defined in Klipper. The example below are the pin mappings for the integrated EBB36 ADXL and my defined MCU.

```yaml
[adxl345]
cs_pin: EBB36_Toolhead: PB12
spi_software_sclk_pin: EBB36_Toolhead: PB10
spi_software_mosi_pin: EBB36_Toolhead: PB11
spi_software_miso_pin: EBB36_Toolhead: PB2
```

The `[resonance_tester]` must also be configured. The ideal coordinates are just above the center of the print bed.

```yaml
[resonance_tester]
accel_chip: adxl345
probe_points:
    114.5, 117.5, 10
```

Before running any tests, review Klipper's documentation on [Measuring the Resonance](https://www.klipper3d.org/Measuring_Resonances.html#measuring-the-resonances_1) for a better understanding.

Once everything is set up, `SHAPER_CALIBRATE` will automatically run the tests, measure the resonance, and update the input shaper values in Klipper. It will also give a value that can be used to tune for `max_accel`.