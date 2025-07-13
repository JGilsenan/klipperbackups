# Understanding Homing and Z-Offsets

-----------------------------------------------------------
The setup:
-----------------------------------------------------------
Imagine you are setting up everything for a brand new build, you have T0 mounted, and you have the z_offset set to 0.0 (as well as gcode offsets, but those are always 0 for T0)

Ensure that your printer has heat soaked for 1-2 hours and that the tool being used is heated to 150C for all of these steps.


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

- When you start `PROBE_CALIBRATE` the display should only show current position, if there is a "saved value" listed then you need to ensure all offsets were reset to 0 and restart
- Using the display, step the nozzle down until it just barely produces friction when you try to slide the paper around, it will be a very light amount of friction
- The number displayed on the screen will be positive, since it represents the distance above where the printer currently places Z = 0 where the nozzle almost, but doesn't quite touch the bed
- In my case, the number I arrived at on the display was 0.6mm, which, in terms of absolute value, corresponds to the distance between the nozzle and the build plate when Z = 0 plus the thickness of the paper, or around 0.1mm
- Going back to the number that I arrived at, 0.6mm, that means that if we subtract the 0.1mm for the paper, the nozzle should currently be located 0.5mm below the build surface when Z = 0
- DESPITE MENTIONING PAPER THICKNESS ABOVE, ignore this for now, and stick with the number you arrived at (we account for the paper later on), which for me was 0.6mm, that is my offset

Now we can take our offset and apply it to the configs, the `PROBE_CALIBRATE` menu has an option for automatically saving the value for you, I do not recommend doing this at this point, 
it keeps things easier to understand in the future if you manually set the offset. Before you rush to set a Z_Offset = 0.6, we need to discuss signs. 

- While the offset we reached running `PROBE_CALIBRATE` was a positive value, that is only relative to where the printer currently thinks Z = 0 is right now
- Because the nozzle is below the print surface when Z = 0 now, we need to apply an offset to where the printer thinks Z = 0 is that will place the nozzle right on the build surface when Z = 0
- Therefore, instead of setting Z_Offset = 0.6, we set it to Z_Offset = [offset found using PROBE_CALIBRATE] * -1, or in this case Z_Offset = -0.6


-----------------------------------------------------------
Babystepping, what about the thickness of the paper?:
-----------------------------------------------------------
At this point we have set our T0 Z_Offset to -0.6, saved the configuration and restarted the printer, upon reboot we re-homed it and then ran a QGL in case any corners sagged during the reboot. 
Re-home T0 once again to get the toolhead back in the center of the build area.