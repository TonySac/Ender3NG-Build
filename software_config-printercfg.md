---
title: "The printer.cfg File"
layout: default
nav_order: 1
parent: "Software Configuration"
has_children: true
posted: 2024-05-07
---

# The *printer.cfg* File
{: .no_toc }
---

I chose to generate my own initial configuration from previous knowledge while also referring to my other printers and [Klipper's configuration reference](https://www.klipper3d.org/Config_Reference.html).

This document covers the step by step generation of my *printer.cfg* file. Subsequent pages will cover some of the `[include]` files along with startup and tuning checks.

{: .note }
:pencil2: I found it easier to do some of the startup checks as the configuration was generated. For brevity of this document, I broke them out into a separate write ups and included a link when nessicary.

My configuration is broken down as follows:

<details open markdown="block">
  <summary>
  printer.cfg
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>
---

{: .notice }
:loudspeaker: Remember this is all specific to my particular machine. All pin definitions should be verified and changes made in accordance with the hardware in use.


A fresh Klipper & Mainsail installation should have generated a bare bones *printer.cfg* file.

```yaml
[include mainsail.cfg]
[mcu]
serial: /dev/serial/by-id/<your-mcu-id>

[virtual_sdcard]
path: /home/E3NG/printer_data/gcodes
on_error_gcode: CANCEL_PRINT

[printer]
kinematics: none
max_velocity: 1000
max_accel: 1000
```

Make sure to keep the `[include mainsail.cfg]` and the information in the `[printer]` section. The entire `[virtual_sdcard]` section can be removed as it is duplicated in *mainsail.cfg*. 

# MCU Settings & Features

As the title suggests, this block holds the MCU definitions and enables Klipper features.

My primary mcu communicates via USB to CAN bridge and is defined with the following code. The `canbus_uuid` is unique per machine.

```yaml
[mcu]
canbus_uuid: 027a94e7a31e
```

I also added a temperature reading to the frontend for this board by using the `[temperature_sensor]` option. Where "SKR" is my chosen name for the controller.

```yaml
[temperature_sensor SKR]
sensor_type: temperature_mcu
min_temp: 0
max_temp: 100
```

The EBB36 CAN toolhead board was then added to the configuration.

```yaml
[mcu EBB36_Toolhead]
canbus_uuid: cf152fa74cfc
```

> Since this is an additional mcu, I gave it the name `EBB36_Toolhead`. This will be referred to in later sections.

Once again, I added a temperature sensor to the fronted. This time I needed to define `sensor_mcu` to tell Klipper what MCU the sensor was located on.

```yaml
[temperature_sensor EBB36_Toolhead]
sensor_type: temperature_mcu
sensor_mcu: EBB36_Toolhead
min_temp: 0
max_temp: 100
```

I perviously set up the Raspberry Pi host as an MCU, that needed to be defined as well along with a temperature sensor for it as the host. The host serial path should be the same regardless of machine or installation.

```yaml
[mcu Raspberry_Pi]
serial: /tmp/klipper_host_mcu

[temperature_sensor Raspberry_Pi]
sensor_type: temperature_host
min_temp: 0
max_temp: 100
```

{: .note }
:pencil2: The Pi has an integrated temperature sensor but not all host MCUs do.

Finally, I added the `[exlcude_object]` feature. This is also where I place any included config files.

```yaml
[exclude_object]
[include mainsail.cfg]
```

A `Save & Restart` of the firmware should bring Mainsail back online with the above defined temperature sensors now visible on the frontend.

<img src="/assets/mainsail-board-temps.png"/>


# Motion

This section contains the printer kinematics, stepper motor configurations, and other motion related settings.

To start, the `[printer]` section needs to be updated. The Ender 3 NG uses corexy kinematics. The accelerations and velocities should also be set but can be tuned later. I started with the following settings:

```yaml
[printer]
kinematics: corexy
max_velocity: 200
max_accel: 2000
max_z_velocity: 5
max_z_accel: 100
```

If a firmware restart is done at this point, an error regarding "step_pin" will appear. This is expected as we have yet to define any stepper motors. Therefore, I next defined the stepper motor configurations.

## Stepper X/Y/Z
{: .no_toc }

The stepper X/Y/Z settings can be pulled off of the [generic BTT SKR Mini E3 v3 config](https://github.com/Klipper3d/klipper/blob/master/config/generic-bigtreetech-skr-mini-e3-v3.0.cfg) with some minimal modifications. Remember, this board uses TMC2209 drivers, so both sections are needed for each motor.

The `step`, `dir`, `enable`, `uart`, and `tx` pins should all be the same. I added comments in the code below to denote what was changed from the generic config.

### Stepper X (A Motor)
{: .no_toc }

Additional configuration settings have been added for sensorless homing.

```yaml
[stepper_x]
step_pin: PB13
dir_pin: !PB12
enable_pin: !PB14
microsteps: 32 #Increased from 16
rotation_distance: 40
endstop_pin: tmc2209_stepper_x: virtual_endstop #For sensorless homing. Also requires DIAG jumper. Hardware pin was ^PC0 
position_endstop: 235 #Set to theoretical max and will be tuned.
position_min: 0 #Added for visibility and will be tuned.
position_max: 235 # Theoretical max. Will be tuned.
homing_speed: 25 #Decreased from 50.
homing_retract_dist: 0 #Disables second homing attempt. Must be disabled for sensorless homing.

[tmc2209 stepper_x]
uart_pin: PC11
tx_pin: PC10
uart_address: 0
interpolate: False #Disable interpolation
run_current: 0.7 # Default 0.580 | Calculated as 70% max_run_current where max_run_current=(rated_current * 0.707). For StepperOnline 17HS15-1504S-X1 motor ((1.50 * 0.707)*0.70=0.742).
stealthchop_threshold: 0 #Use spreadCycle. Default is 999999 for stealthChop
diag_pin: ^PC0 #Jumped endstop pin required for sensorless homing
driver_SGTHRS: 255 #Added at max sensitivity. Must be tuned for sensorless homing
```

### Stepper Y (B Motor)
{: .no_toc }

The `stepper_y` includes most of the same changes as `stepper_x` however I elected to use a physical endstop for this axis.

```yaml
[stepper_y]
step_pin: PB10
dir_pin: !PB2
enable_pin: !PB11
microsteps: 32 #Increased from 16
rotation_distance: 40
endstop_pin: ^PC1 
position_endstop: 235 #Set to theoretical max and will be tuned.
position_min: 0 #Added for visibility and will be tuned.
position_max: 235
homing_speed: 25 #Decreased from 50.

[tmc2209 stepper_y]
uart_pin: PC11
tx_pin: PC10
uart_address: 2
interpolate: False #Disable interpolation
run_current: 0.7 # Default 0.580 | Calculated as 70% max_run_current where max_run_current=(rated_current * 0.707). For StepperOnline 17HS15-1504S-X1 motor ((1.50 * 0.707)*0.70=0.742).
stealthchop_threshold: 0 #Use spreadCycle. Default is 999999 for stealthChop
```

### Stepper Z
{: .no_toc }

The Z axis does not and should not use sensorless homing. My build currently uses a CR Touch and as such this section is immediately followed up by a `[bltouch]` configuration. The base config is outlined below with changes in the comments.

```yaml
[stepper_z]
step_pin: PB0
dir_pin: PC5
enable_pin: !PB1
microsteps: 16
rotation_distance: 8
gear_ratio: 42:20 #Add the 42 tooth printed pulley to 20T motor pulley ratio
endstop_pin: probe: z_virtual_endstop # For CR/BL Touch. Hardware endstop pin was ^PC2
#position_endstop: 0.0 Remove this since a probe is used
position_max: 245 #Set to theoretical max and will be tuned.
position_min: 0 #Added for visibility and will be tuned.

[tmc2209 stepper_z]
uart_pin: PC11
tx_pin: PC10
uart_address: 1
run_current: 0.7 # Default 0.580 | Calculated as 70% max_run_current where max_run_current=(rated_current * 0.707). For StepperOnline 17HS15-1504S-X1 motor ((1.50 * 0.707)*0.70=0.742).
stealthchop_threshold: 0 #Use spreadCycle. Default is 999999 for stealthChop
```

If the firmware is saved and restarted now an error regarding `probe` should appear. Again, this is an expected error as the probe still needs to be configured.

#### Probe
{: .no_toc }

I've included the probe or `[bltouch]` configuration next. It is required in the case a probe is being used as the Z endstop.

{: .notice }
:loudspeaker: Verify the probe and board pins. My installation has the CR Touch connected to the endstop connector on the EBB36 and **NOT** the standard probe pins.

```yaml
[bltouch]
sensor_pin: ^EBB36_Toolhead: PB7
control_pin: EBB36_Toolhead: PB6
y_offset: -18 #Measured per printer/mount. `x_offset` can also be specified.
z_offset: 0 # Required initially but will be tuned.
```

At this point, all of the motion components have been defined. A `Save & Restart` should bring Klipper back online with no errors and some motion control available on the front end.

> I suggest completing some [basic motion checks and tuning](/software_config-printercfg-start_tune.html#motion-system) before continuing to build the configuration.

# Extruder

This section defines the extruder motor, heater, and limits. Recall, my build uses an EBB36 toolhead board so all of the pin definitions are in relation to that. Additionally, some configurations were modified for use with the Stealthbuner. Things like PID control and rotation distance will need to be tuned on a per machine basis. Also, rather than specify pressure advance, I use slicer macros to push the value on a per filament basis.

```yaml
[extruder]
step_pin: EBB36_Toolhead: PD0
dir_pin: EBB36_Toolhead: PD1
enable_pin: !EBB36_Toolhead: PD2                                 
rotation_distance: 22.23 #Default value. Will be tuned.
gear_ratio: 50:10 #Stealthburner/CW2 ratio
microsteps: 32
nozzle_diameter: 0.4
filament_diameter: 1.750
heater_pin: EBB36_Toolhead: PB13
sensor_type: Generic 3950
sensor_pin: EBB36_Toolhead: PA3
control: pid # Generic PID settings. Will be tuned.
pid_Kp: 21.527
pid_Ki: 1.063
pid_Kd: 108.982
min_temp: 0
max_temp: 285
max_extrude_only_distance: 101 

[tmc2209 extruder]
uart_pin: EBB36_Toolhead: PA15
interpolate: False
run_current: 0.6
sense_resistor: 0.110
stealthchop_threshold: 0 #Set to 0 for spreadcycle, avoid using stealthchop on extruder
```

If everything is correctly defined, the extruder temperature should now be visible on the Mainsail frontend.

{: .notice }
:loudspeaker: Make sure the extruder temperature is holding steady and not heating. If it is heating, then shut down the printer and start troubleshooting the wiring and/or config definitions.

Other error messages are usually fairly detailed and can assist if issues arise.

I've also included the configuration for the toolhead fans in this section. It made sense to me as they are needed for proper extruder operation.

```yaml
[fan]
pin: EBB36_Toolhead: PA1

[heater_fan hotend_fan]
pin: EBB36_Toolhead: PA0
heater: extruder
heater_temp: 50.0
```

<img src="/assets/fans.png" width="375"/>{: .float-right}{: .ml-3}

Once the fans are defined they should also appear on Mainsail. 

> I recommended [verifying the extruder](/software_config-printercfg-start_tune.html#extruder-checks) functions at this stage.

# Bed Heater

Similar to the extruder section the heated bed needs to be defined.

```yaml
[heater_bed]
heater_pin: PC9
sensor_type: ATC Semitec 104GT-2
sensor_pin: PC4
min_temp: 0
max_temp: 130
control: pid # Generic PID settings. Will be tuned.
pid_Kp: 21.527
pid_Ki: 1.063
pid_Kd: 108.982
```

Once again, heater bed should now be visible on the front end with a steady temperature.

> The [heated bed function should be verified](/software_config-printercfg-start_tune.html#verify-heated-bed).

<br>

The basic *printer.cfg* has been built. Before starting to print, some [minimal checks and startup tuning](/software_config-printercfg-start_tune.html#startup-checks-and-tuning) is required if it has not already been done. 