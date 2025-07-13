# Understanding Homing and Z-Offsets

-----------------------------------------------------------
The setup:
-----------------------------------------------------------
Imagine you are setting up everything for a brand new build, you have T0 mounted, and you have the z_offset set to 0.0 (as well as gcode offsets, but those are always 0 for T0)

Ensure that your printer has heat soaked for 1-2 hours and that the tool being used is heated to 150C for all of these steps.

Ensure that your nozzles are all clean, this only works as well as your nozzles are clean!

-----------------------------------------------------------
Homing for the first time:
-----------------------------------------------------------
Now you run the homing routine to home all three axes, assuming that tap is being used, the routine runs and the printer is homed, but what does that even mean?

- When tap homes on the Z-axis, it is finding zero as the point at which the tap sensor triggers. 
- What that means is that at this point, with all offsets set to 0, commanding Z to 0 will cause the z axis to move down to the point that triggers its sensor.
- Recall that for tap to activate the sensor, the nozzle must be touching the bed and the entire toolhead must slide up on the tap mechanism until it triggers.
- With those points in mind, if we think of this Z position where the trigger occurs as Z = 0, then the nozzle must be located in a position where Z < 0. 
- Recall again that currently we do not have any offsets set, they are all currently still 0.
- NOTE: when homing is complete your printer will move to Z = 10mm


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

- NOTE: previously when homing the printer would move to Z = 10mm, now, with the Z_Offset = -0.6, homing ends at Z = 9.4mm

Next, before continuing, let's confirm everything that has been said before.

- Command the nozzle to move to Z = 0mm
- Insert a piece of paper between the nozzle and the print surface
- The paper should fit with just the slightest friction, exactly as it did when performing the `PROBE_CALIBRATE` procedure before
- If the paper does not fit then a mistake was made and I suggest restarting from the top and retrying until you achieve this result, there is no point in continuing otherwise.

A few quick notes about where the nozzle is positioned for the first layer

- Your slicer assumes that when Z = 0, the nozzle is just touching the print surface
- When computing nozzle positions for the gcode to be generated, the slicer will place the nozzle above the bed by a distance equal to your first layer thickness
- For example, if my first layer is set to a thickness of 0.25mm, then the slicer needs to set the Z position such that the nozzle is exactly 0.25mm above the print surface
- Due to all of this, if your Z_Offset is incorrect at this point, the generated gcode will also not work as expected

Now what about the thickness of the paper?

- You haven't forgotten, and neither have I
- The offset we're currently using still includes the thickness of the paper, which is usually somewhere around 0.1mm
- You may be thinking, well why don't we just get our calipers and measure the paper thickness and then subtract it?
- The short answer is that paper is not uniform in thickness, and compresses easily, so caliper measurements are inaccurate to say the least
- Instead we left that thickness in and will account for it now by babystepping

Methods for babystepping aren't super important here:

- There are a few commonly used methods for babystepping to get your first layer thickness (squish) right, choose whichever works for you
- I tend to use the Ellis method, which relies on using the printer's "fine tuning" menu during a print to adjust the Z_Offset during a print of test patches
- NOTE: if you are using the "fine tuning" menu to babystep, write down the number you arrive at, I DO NOT RECOMMEND AUTO SAVING YOUR CONFIGURATION, it makes things less obvious later on
- While the methods may vary slightly from one another, the important part is that whichever way you did it you got a number out of it
- This part is key because your number may be positive or negative based on the method chosen, ignore the sign and make the number you use negative, so if you got 0.1mm make it -0.1mm
- Doing this will keep things in the same terms as the Z_Offset you arrived at earlier and ease in understanding

Following the above guidance, I arrived at a babystepping offset of -0.1mm, which sounds about right considering that the thickness of the paper I used was around 0.1mm
Now we're left with: what does that number mean and how do I save it?

- As was mentioned above, we're assuming you didn't click on "save z offset" after your babystepping but instead wrote the number down
- This number represents the thickness of the paper when we performed our babystepping
- Consider again our current Z_Offset, -0.6mm, when homing takes place you can imagine that the printer first finds where the tap sensor is triggered (nozzle below bed) and then "subtracts" the 
  Z_Offset value from the current position, which our babystepping has shown to be too high off the print surface by 0.1mm
- Now, to arrive at the correct Z_Offset for our configuration we need to subtract our babystepping offset, -0.1mm, from our current Z_Offset, -0.6mm, -0.6mm - -0.1mm = -0.5mm
- Our new Z_Offset for T0 is -0.5mm

Now, update the Z_Offset for T0 in the configs, then save and restart, don't forget to set your extruder and bed temps again upon reboot, then home, QGL, and home again.

- You should note now that after homing the Z position has changed from 9.4mm before when the Z_Offset was set to -0.6mm to 9.5mm now with a Z_Offset of -0.5mm.

Next, we can verify that our offsets are where we want them for T0

- Command the tool to Z = 0mm
- The nozzle should be just touching the bed
- You should not be able to slide a piece of paper under the nozzle
- If there is a gap visible, or if the paper slides under the nozzle, a mistake was made and I recommend restarting from the top
- Assuming that all looks well at this point, command the Z position to the absolute value of the offset found while babystepping above, in our case ABS(-0.1mm) = 0.1mm
- If everything was done correctly up until this 








