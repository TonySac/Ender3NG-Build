---
title: "Startup Tuning"
layout: default
nav_order: 2
parent: "The printer.cfg File"
grand_parent: "Software Configuration"
posted: 2024-05-07
---

# Startup Checks and Tuning
{: .no_toc}

<details closed markdown="block">
  <summary>
  Table of Contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

Since the E3NG is a newer design, this is a scratch configuration build, and a new firmware install, some basic startup checks and minor tuning is required. These checks make sure the motors and endstops function appropriately, the print area is correctly defined, and the heaters work as expected.

{: .warn }
:warning: All of these checks and steps should be performed with the axis well out of the way of each other and an emergency stop/shutdown procedure ready, available, and working. If the printer fails one of these checks it can result in damage.

# Motion System

The motion system checks first verify the operation of all axis motors and endstops then is followed by tuning of the print area and homing fucntions.

## Verifying Steppers

Start by checking each stepper with the `STEPPER_BUZZ` command. With the bed/nozzle well out of the way use the Kipper console to run: 

```yaml
STEPPER_BUZZ STEPPER=stepper_x
```

This should rotate the `A` motor 1mm clockwise then back 1mm counterclockwise for a total of 10 times. If the desired behavior is not observed verify the `enable_pin`, `step_pin`, `dir_pin`, and `rotation_distance` config settings. Also double check the physical motor connections and locations.

Repeat these checks for `stepper_y` and `stepper_z`

## Verify Physical Endstops

Start by making sure no endstops are depressed. Then run `QUERY_ENDSTOPS` in the Klipper terminal. If everything is correct, you should see the following output:

```yaml
x:open y:open z:open
```

Now manually depress an endstop and run `QUERY_ENDSTOP` again. This time the depressed endstop should report `TRIGGERED`. Repeat this for all physical endstop switches.

> In the event the opposite behavior occurs, invert the pin definition by adding or removing a prepended `!`. If the wrong endstop reports and/or nothing happens verify the wiring, pin assignments, and pullup resistor settings. The pullup resistor is indicated with `^` prepended to the pin name.

Once the physical endstops are verified, we can check the motors home and move correctly.

## Homing Checks

{: .warn }
:warning: Be prepared to quickly stop the printer if the axis is moving in the wrong direction.

### Physical Endstops

Send a home command for a single axis that is equipped with a physical endstop. For the sake of example and because this is about my config. I will use the Y axis. A home Y command `G28 Y` will begin to home the Y axis. The expected behavior is the Y axis moves back until it triggers the endstop, completes a second homing move, then reports a location of about Y 235.

> Based on the `stepper_y` config, the endstop is located at 235mm (which is also `position_max`) so this axis wants to home in the positive direction.

In the event the Y axis moves in the opposite direction, **IMMEDIATELY STOP** the printer. Evaluate the `dir_pin` as it may need to be inverted with `!`. If the X axis moves instead of the Y **IMMEDIATELY STOP** the printer. The A/B cables likely need to be swapped or the pins can be changed in the config file.

Repeat this for any axis with a physical endstop.

### Tune Sensorless Homing

Sensorless homing needs to be tuned and motor direction confirmed for the equipped axis. As long as the max StallGaurd or `driver_SGTHRS` was set in the config it is safe to begin this. Since I'm only using sensorless homing on the X axis, I will be using that example.

Start by attempting to home the X axis `G28 X`. The X axis should begin to move in the positive direction but almost immediately stop. If it does not stop or the wrong axis moves, **IMMEDIATELY STOP** the printer. If the axis moves in the wrong direction, verify the `dir_pin` and invert it with `!` if needed. Take time to double check the motor plugs and other pins as needed.

> Per the `stepper_x` config above, Klipper expects to home in the positive direction, this can be changed per preference.

If everything is moving properly, manually lower the StallGaurd threshold using `SET_TMC_FIELD FIELD=SGTHRS STEPPER=stepper_x VALUE=###`. Where `VALUE=###` is max at 255 and min at 0. Manually lower this threshold and continue to attempt to home the axis until it successfully homes without banging into the rail. Record the max StallGaurd that successfully homes the axis. My StallGaurd_Max was 90.

Continue to lower the StallGaurd value until the axis homes with a noticeable bang or skipped step. Increase the value +1 and confirm the axis homes without issue. Record this value as the minium StallGaurd for a successful home. On my printer StallGaurd 58 resulted in a bang but StallGaurd 59 homed fine. Therefore, my StallGaurd_Min was 59.

Use the following formula to calculate the new `driver_SGTHRS` value:

```
driver_SGTHRS = [(StallGaurd_Max - StallGaurd_Min)/3] + StallGaurd_Min
```

Example:

```
[(90-59)/3]+59 = 69.33 =~ 70 {I opted to round up}
```

Update `printer.cfg` and repeat the process as needed.

{: .notice }
:loudspeaker: If changes are made to the printer hardware or motor configuration, including belt tensions, motor currents, speeds, etc. then sensorless homing may require additional tuning.

### Z Endstop/Probe

Since this build is equipped with a CR Touch, it will be used as the Z endstop. If installed and wired correctly, the CR touch should self test on startup then have a steady blue LED. Confirm the probe functionality with `BLTOUCH_DEBUG COMMAND=pin_down` to lower the probe pin, followed by `BLTOUCH_DEBUG COMMAND=pin_up` to raise the pin. 

Check the sensor by lowering the pin `BLTOUCH_DEBUG COMMAND=pin_down` and entering touch mode with `BLTOUCH_DEBUG COMMAND=touch_mode`. Run `QUERY_PROBE` and verify the output os `open`. Gently push the pin and `QUERY_PROBE` the result should now show `TRIGGERED`.

If these tests fail, verify the probe wiring and double check the configuration definitions.

It goes without saying the probe should also be slightly above the nozzle with it is stowed. Additionally, take the time now to measure the distance between the probe and nozzle. Update the config with the correct `x_offest` and `y_offset` as needed. Remember these can be negative values.

Once the probe functions as expected, center the print bed and prepare to test the Z homing. Run `G28 Z`. The expected behaviro is hte probe will deploy then the bed will begin moving upwards towards the probe. If this does not happen, **IMMEDATLY STOP** the printer.

  > If the probe failed to deploy, check the wiring, config, and rerun the above tests. If the bed moves in the wrong direction, check the `dir_pin` in `[stepper_z].

As the bed is moving towards the probe, manually trigger the probe to confirm the bed stops. **IMMEDIATELY STOP** the printer if the bed does not stop. 

If the manually triggered probe stopped the bed, it is time to run real Z home test. Assuming the X/Y endstops have also been configured, this can be done alongside a complete machine home. Before this is done, it is important to add a `[safe_z_home]` section to the config. This tells the machine how to position the toolhead prior to homing. A good starting point is as follows:

```yaml
[safe_z_home]
home_xy_position: 117.5, 117.5 
z_hop: 10                
```

This will move the bed 10 mm out of the way prior to homing, then position the toolhead at the specified position before homing the Z axis. The `home_xy_position` value should be calibrated to the true center of the print bed.

If the process is successful, allow the Z axis to run a homing routine with the probe actually touching the bed.


## X/Y Motion Range

The motion range can be defined in Klipper by using a combination of the `postion_min`, `position_max`, and `position_endstop` values in the stepper motor configurations. Defining these values prevents the printer from making a move out of range or a crash from occurring.

For the sake of this example, I will be using the Y axis. The procedure is the same for X.

{: .warn }
:warning: The `position_min` value will need to be made negative for this procedure. This increases the possibility of a toolhead crash and machine damage. Use caution. 

Start by setting the `postion_min` to a negative number. -25 is usually sufficient. With this complete, home the Y axis. Since my build homes towards the maximum travel, I will record the reported value as Y_Max. Slowly step the axis over until the nozzle is directly over the maximum point (or back) of the bed and record this value as Y_Bed_Max. Continue to step the axis until it is just over the minimum point (or front) of the bed and record this as Y_Bed_Min. Finally, continue to move the axis until it can no longer physically move and record this as Y_Min. 

We want Klipper to use the front of the bed as the 0 or origin point for the Y axis. Use the following formulas to update *printer.cfg*. Pay close attention to any negative values.

```
position_max = Y_Max -  Y_Bed_Min 
position_min = Y_Min -  Y_Bed_Min 
position_endstop = position_max
```

> The difference between Y_Max and Y_Min `(Y_Max - Y_Min)` can be used to determine the true print volume.

Input these changes and home the axis. Verify the printer reports the location of 0 when the nozzle is just over the front of the bed. Then verify the carriage cannot move beyond the set limits or bump the rails.

Repeat this as needed for the X axis. Be sure to input the print volume and origin location into the slicer.

> I'm using a Stealthburner toolhead. This limits some of the printable area. Rather than lose about 35 mm off the Y axis, I took about 10 mm off the X axis (5 on each side). This means the origin point is just to the right of the bed edge on my build. It also means I can still crash my toolhead on the front right frame if I'm not careful. Most of this can be addressed in the slicer and with some macros.

## Z Motion Range

With the use of the CR touch, I usually just leave this set to the theoretical maximum. Unless I plan to print parts that are close to the Z limit. I do recommend changing the `position_min` to around -2 to -5 to facilitate later tuning of the Z offset.

# Extruder Checks

## Fans

Before checking the extruder heater, make sure the fans work. The Mainsail fan control should have the part cooling fan controllable on the front end. Increase its speed to 100% and make sure it is indeed the part cooling fan that turns on. In the event the hotend fan turns on, simply update the pin definitions in *printer.cfg*. If nothing happens, then verify both the wiring and pin definitions are correct.

## Heater

Next, check the extruder heats up. Set the extruder temp to 200&deg;C. The expected behavior is for the hotend fan to engage and the extruder to start to heat. Return the extruder temp to 0&deg;C. The extruder should begin to cool and the fan will turn off once below the minimum set temperature.

## Motor

Finally, make sure the extruder motor is working as expected. On a direct drive, this is often easier to do at temperature and with filament loaded. Heat the extruder to the appropriate extrude temperature and run an extrude command. The motor should turn in the correct direction. If not, verify the pins and connections.

{: .note }
:pencil2: If so inclined, e steps can also be calibrated at this time.

# Verify Heated Bed

Similar to the extruder, check the heated bed. Set the bed temp to 50&deg;C. The bed should slowly start to heat up. Return the bed temp to 0&deg;C and it should begin to cool down.

# PID Tune

Although not immediately required, it is helpful to tune the PID controller prior to attempting a print. This avoids heater out of range errors. Make sure to set the printer up and tune for as close to normal printing conditions as possible.

Run `PID_CALIBRATE HEATER=heater_bed TARGET=##` to calibrate the bed. Where `TARGET=##` should be the bed temperature planned to be used. Once this is complete save the values and run it again but this time for `HEATER=extruder` along with the appropriate temperature. I also recommend setting the print fan to the expected speed for this calibration.

---

# Time to Print!

With the *printer.cfg* file completed and some startup checks/tuning done it is time to run a first print. A Benchy or Calibration Cube are always good choices. These will also help with identifying any remaining issues. For perfect prints, additional calibration will still be needed. I recommend [Ellis' Guide](https://ellis3dp.com/Print-Tuning-Guide/) along with [Teaching Tech](https://teachingtechyt.github.io/calibration.html) and some of [OrcaSlicer's built in calibration tools](https://github.com/SoftFever/OrcaSlicer/wiki/Calibration).

