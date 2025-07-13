# Understanding Homing and Z-Offsets

-----------------------------------------------------------
The setup:
-----------------------------------------------------------
Imagine you are setting up everything for a brand new build, you have T0 mounted, and you have the z_offset set to 0.0 (as well as gcode offsets, but those are always 0 for T0)


-----------------------------------------------------------
Homing for the first time:
-----------------------------------------------------------
Now you run the homing routine to home all three axes, assuming that tap is being used, the routine runs and the printer is homed, but what does that even mean?

- When tap homes on the Z-axis, it is finding zero as the point at which the tap sensor triggers. 
- What that means is that at this point, with all offsets set to 0, commanding Z to 0 will cause the z axis to move down to the point that triggers its sensor.
- Recall that for tap to activate the sensor, the nozzle must be touching the bed and the entire toolhead must slide up on the tap mechanism until it triggers.
- With those points in mind, if we think of this Z position where the trigger occurs as Z = 0, then the nozzle must be located in a position where Z < 0. 
- Recall again that currently we do not have any offsets set, they are all currently still 0.


-----------------------------------------------------------
Finding Z_Offset for T0:
-----------------------------------------------------------
Our objective at this point is now to find the Z_Offset for T0, put differently, we want to find the distance between where Z = 0 currently places the nozzle when the tap sensor triggers, and
where the nozzle actually is. 

- Since we recall the important detail about the Z = 0 position being where the tap sensor triggers, we know that the Z_Offset for T0 will always be a negative value
- Again, this is because when the sensor triggers, the nozzle was already touching the bed and the tap mechanism had to move the toolhead upwards

There are a number of different methods for obtaining the T0 Z_Offset, for simplicity's sake we're going to use the one prescribed by the voron documentation for tap, which is to run the 
`PROBE_CALIBRATE` macro to adjust the Z position until it touches a piece of paper.