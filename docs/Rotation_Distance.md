Stepper motor drivers on Klipper require a `rotation_distance`
parameter in each
[stepper config section](Config_Reference.md#stepper). The
`rotation_distance` is the amount of distance that the axis moves with
one full revolution of the stepper motor. This document describes how
one can configure this value.

# Obtaining rotation_distance from steps_per_mm (or step_distance)

The designers of your 3d printer originally calculated `steps_per_mm`
from a rotation distance. If you know the steps_per_mm then it is
possible to use this general formula to obtain that original rotation
distance:
```
rotation_distance = <full_steps_per_rotation> * <microsteps> / <steps_per_mm>
```

Or, if you have an older Klipper configuration and know the
`step_distance` parameter you can use this formula:
```
rotation_distance = <full_steps_per_rotation> * <microsteps> * <step_distance>
```

The `<full_steps_per_rotation>` setting is determined from the type of
stepper motor. Most stepper motors are "1.8 degree steppers" and
therefore have 200 full steps per rotation (360 divided by 1.8 is
200). Some stepper motors are "0.9 degree steppers" and thus have 400
full steps per rotation. Other stepper motors are rare. If unsure, do
not set full_steps_per_rotation in the config file and use 200 in the
formula above.

The `<microsteps>` setting is determined by the stepper motor driver.
Most drivers use 16 microsteps. If unsure, set `microsteps: 16` in the
config and use 16 in the formula above.

Almost all printers should have a whole number for `rotation_distance`
on x, y, z, and similar axes. If the above formula results in a
rotation_distance that is within .01 of a whole number then round the
final value to that whole_number.

# Obtaining rotation_distance by inspecting the hardware

It's also possible to calculate rotation_distance with knowledge of
the stepper motors and printer kinematics. This may be useful if the
steps_per_mm is not known or if designing a new printer.

## Belt driven axes

It is easy to calculate rotation_distance for a linear axis that uses
a belt and pulley.

First determine the type of belt. Most printers use a 2mm belt pitch
(that is, each tooth on the belt is 2mm apart). Then count the number
of teeth on the stepper motor pulley. The rotation_distance is then
calculated as:
```
rotation_distance = <belt_pitch> * <number_of_teeth_on_pulley>
```

For example, if a printer has a 2mm belt and uses a pulley with 16
teeth, then the rotation distance is 32.

## Axes with a lead screw

It is easy to calculate the rotation_distance for common lead screws
using the following formula:
```
rotation_distance = <screw_pitch> * <number_of_separate_threads>
```

For example, the common "T8 leadscrew" has a rotation distance of 8
(it has a pitch of 2mm and has 4 separate threads).

Older printers with "threaded rods" have only one "thread" on the lead
screw and thus the rotation distance is the pitch of the screw. (So,
for example, an M6 metric rod has a rotation distance of 1, and an M8
rod has a rotation distance of 1.25.)

# Calibrating rotation_distance on extruders

On an extruder, the `rotation_distance` is the amount of distance the
filament travels for one full rotation of the stepper motor. The best
way to get an accurate value for this setting is to use a "measure and
trim" procedure.

First start with an initial guess for the rotation distance. If you
don't have any starting point at all, then one can estimate the
rotation distance by measuring the diameter of the hobbed gear that
pushes the filament and use `rotation_distance = <diameter> * 3.14`.

Then use the following procedure to "measure and trim":
1. Make sure the extruder has filament in it, the hotend is heated to
   an appropriate temperature, and the printer is ready to extrude.
2. Use a marker to place a mark on the filament around 70mm from the
   intake of the extruder body. Then use a digital calipers to measure
   the actual distance of that mark as precisely as one can. Note this
   as `<initial_mark_distance>`.
3. Extrude 50mm of filament with the following command sequence: `G91`
   followed by `G1 E50 F60`. Note 50mm as
   `<requested_extrude_distance>`.
4. Use the digital calipers to measure the new distance between the
   extruder body and the mark on the filament. Note this as
   `<subsequent_mark_distance>`.
5. Calculate `actual_extrude_distance = <initial_mark_distance> -
   <subsequent_mark_distance>`. Then calculate rotation_distance as
   `rotation_distance = <previous_rotation_distance> *
   <actual_extrude_distance> / <requested_extrude_distance>`. Round
   the new rotation_distance to three decimal places.

If the actual_extrude_distance differs from requested_extrude_distance
by more than about 2mm then it is a good idea to perform the steps
above a second time.

Note: Do *not* use a "measure and trim" type of method to calibrate x,
y, z, or other kinematic axis. The "measure and trim" method is not
accurate enough for those axes and will likely lead to a worse
configuration. Instead, if needed, those axes can be determined by
measuring the belts, pulleys, and lead screw hardware as discussed in
the sections above.

# Using a gear_ratio

Setting a `gear_ratio` can make it easier to configure the
`rotation_distance` on steppers that have a gear box (or similar)
attached to it. Most steppers do not have a gear box - if unsure then
do not set `gear_ratio`.

When `gear_ratio` is set, the `rotation_distance` represents the
distance the axis moves with one full rotation of the final gear on
the gear box. If, for example, one is using a gearbox with a "5:1"
ratio, then one could calculate the rotation_distance from
[knowledge of the hardware](#obtaining-rotation_distance-by-inspecting-the-hardware)
and then add `gear_ratio: 5:1` to the config. Note that the common
"5.18:1 planetary gearbox" is more accurately configured with
`gear_ratio: 57:11`.

For gearing implemented with belts and pulleys, it is possible to
determine the gear_ratio by counting the teeth on the pulleys. For
example, if a stepper with a 16 toothed pulley drives the next pulley
with 80 teeth then one would use `gear_ratio: 80:16`. Indeed, one
could open a common off the shelf "gear boxes" and count the teeth in
them to confirm their gear ratio.

If several gears are used on an axis then it is possible to provide a
comma separate list to gear_ratio. For example, a "5:1" gear box
driving a 16 toothed to 80 toothed pulley could use `gear_ratio: 5:1,
80:16`.

In most cases, gear_ratio should be defined with whole numbers as
common gears and pulleys have a whole number of teeth on them.
However, in cases where a belt drives a pulley using friction instead
of teeth, it may make sense to use a floating point number in the gear
ratio (eg, `gear_ratio: 107.237:16`).
