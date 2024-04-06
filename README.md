# motorSync

This repository contains scripts to do motor synchronization for AWD 3D printers. It has been prepared to be used on the VZbot

## Prerequisite

Install https://github.com/droans/klipper_extras. The synchronization procedure makes use of python scripts from within klipper Macros.
At the moment, there is a small bug within the klipper_extras installer. If prompted to update your printer_data/config/moonraker.conf for updating the
repository, prompt with no to complete. Add the update section to your printer_data/config/moonraker.conf manually.

## Installation

The folder structure of the repository represents the home directory. Copy the provided files accordingly.

Copy the python files that evaluate motor synchronization and ADXL noise.
```
cd
mkdir klipper_functions/
cp motorSync/klipper_functions/* klipper_functions/
```
Copy the yaml file for letting klipper know where to find the python files.
```
cd
mkdir printer_data/functions
cp motorSync/printer_data/functions/* printer_data/functions/
```
Copy the config file containing the motorSync Macros.
```
cd
cp motorSync/printer_data/config/motorSync.cfg printer_data/config/
```
Take a look at `motorSync/printer_data/config/printer.cfg` and add the changes to your `printer.cfg` file.

### Changes to printer_data/config/motorSync.cfg
- adapt the `ACCELEROMETER_MEASURE` calls if your chip is not named `adxl345`
- on top of the file
  - adjust the number of microsteps you are using
  - adjust the distance of a fullstep of your printer
  - adjust the number of optimization runs, depending on the accuracy you wish
 
## Usage

The synchronization macro is executed with `MOTSYNC_SYNC`. Call this before starting a print. There is no need to PAUSE or RESUME your print.

## Some background

The Macro does not use any delayed gcode. This is achieved by having a constant number of vibration runs required to obtain the best synchronization at the expense of a somewhat larger runtime.

The Macro determines static noise of your accelerometer (and gravity) and removes it from the measurement, which makes the approach more reliable.

To bring down the quality of the synchronization to a single number, the peak vibration is determined and then the area of the chart is determined within an (arbitrary) 12ms timespan. The data that was cleaned from noise looks like this:

<img src="Vibrations.png" width="500">

Now the magnitude is calculated and the area between the two markers is calculated via integration, representing the quality of the synchronization.

<img src="Magnitude.png" width="500">

## Kudos
Kudos goes to Altzbox with his initial script: https://github.com/altzbox/motors_sync/wiki/Motor-synchronization-on-printers-with-QuadXY-kinematics-(AWD)
I tried to improve on some of the limitations.

Differences between the two scripts
|                   | motors_sync                                                                           | motorSync                                                 |
|-------------------|---------------------------------------------------------------------------------------|-----------------------------------------------------------|
| master            | python script invoced via system call. uses klippy.serial to invoke klipper movements | klipper Macro. uses klipper_extras to call python scripts |
| timing            | hard coded manually                                                                   | klipper times Macro automatically                         |
| motor sync factor | determined by highest peaks of magnitude and moving average                           | integrate to determine area of magnitude                  |

motors_sync has a big potential of speedup using the BUZZ Macro of motorSync. In the end, motorSync is faster, but more iterations are performed. Getting a result accurate to a 1/32 fullstep requires 72sec.

## Example output
```
21:33:04; $ MOTSYNC_SYNC
21:33:04; echo: MOTSYNC: Starting AWD motor synchronization for
21:33:05; echo: MOTSYNC: microstepping of 32 and a fullstep distance of 0.1 mm.
21:33:05; echo: MOTSYNC: Home and move to center.
21:33:18; echo: MOTSYNC: Measure noise.
21:33:20; echo: MOTSYNC: Determined noise as (x,y,z) (335.7700145872341, 359.90188497872344, 9211.62671228936).
21:33:20; echo: MOTSYNC: Optimize for best X position.
21:33:20; echo: MOTSYNC: Evaluate synchronization at microstep position -16.0
21:33:22; echo: MOTSYNC: Synchronization factor is 106.05982502482075.
21:33:22; echo: MOTSYNC: Evaluate synchronization at microstep position -8.0
21:33:25; echo: MOTSYNC: Synchronization factor is 10.991023219601232.
21:33:25; echo: MOTSYNC: Evaluate synchronization at microstep position 0.0
21:33:27; echo: MOTSYNC: Synchronization factor is 29.322061996377503.
21:33:28; echo: MOTSYNC: Evaluate synchronization at microstep position 8.0
21:33:30; echo: MOTSYNC: Synchronization factor is 45.7907787160798.
21:33:30; echo: MOTSYNC: Evaluate synchronization at microstep position 16.0
21:33:32; echo: MOTSYNC: Synchronization factor is 57.36115538388715.
21:33:33; echo: MOTSYNC: Evaluate synchronization at microstep position -4.0
21:33:35; echo: MOTSYNC: Synchronization factor is 12.799137946132689.
21:33:35; echo: MOTSYNC: Evaluate synchronization at microstep position -12.0
21:33:37; echo: MOTSYNC: Synchronization factor is 88.57909000023558.
21:33:37; echo: MOTSYNC: Evaluate synchronization at microstep position -6.0
21:33:40; echo: MOTSYNC: Synchronization factor is 2.939645008131497.
21:33:40; echo: MOTSYNC: Evaluate synchronization at microstep position -10.0
21:33:43; echo: MOTSYNC: Synchronization factor is 7.684410890565739.
21:33:43; echo: MOTSYNC: Evaluate synchronization at microstep position -7.0
21:33:45; echo: MOTSYNC: Synchronization factor is 17.0378368829733.
21:33:45; echo: MOTSYNC: Evaluate synchronization at microstep position -9.0
21:33:48; echo: MOTSYNC: Synchronization factor is 16.62006898968306.
21:33:48; echo: MOTSYNC: Final x synchronization factor is 2.939645008131497
21:33:48; echo: MOTSYNC: Final position is -6 microstep(s)
21:33:48; echo: MOTSYNC: Optimize for best Y position.
21:33:48; echo: MOTSYNC: Evaluate synchronization at microstep position -16.0
21:33:50; echo: MOTSYNC: Synchronization factor is 2.531243225865282.
21:33:50; echo: MOTSYNC: Evaluate synchronization at microstep position -8.0
21:33:53; echo: MOTSYNC: Synchronization factor is 27.604444991219808.
21:33:53; echo: MOTSYNC: Evaluate synchronization at microstep position 0.0
21:33:55; echo: MOTSYNC: Synchronization factor is 40.12537643932361.
21:33:55; echo: MOTSYNC: Evaluate synchronization at microstep position 8.0
21:33:58; echo: MOTSYNC: Synchronization factor is 57.386973112811056.
21:33:58; echo: MOTSYNC: Evaluate synchronization at microstep position 16.0
21:34:00; echo: MOTSYNC: Synchronization factor is 65.47827744697699.
21:34:00; echo: MOTSYNC: Evaluate synchronization at microstep position -12.0
21:34:03; echo: MOTSYNC: Synchronization factor is 21.68640305355946.
21:34:03; echo: MOTSYNC: Evaluate synchronization at microstep position -20.0
21:34:05; echo: MOTSYNC: Synchronization factor is 13.77146702969177.
21:34:05; echo: MOTSYNC: Evaluate synchronization at microstep position -14.0
21:34:08; echo: MOTSYNC: Synchronization factor is 25.25641141375112.
21:34:08; echo: MOTSYNC: Evaluate synchronization at microstep position -18.0
21:34:10; echo: MOTSYNC: Synchronization factor is 22.7587027743155.
21:34:11; echo: MOTSYNC: Evaluate synchronization at microstep position -15.0
21:34:13; echo: MOTSYNC: Synchronization factor is 27.249913676231312.
21:34:13; echo: MOTSYNC: Evaluate synchronization at microstep position -17.0
21:34:16; echo: MOTSYNC: Synchronization factor is 25.2214910769776.
21:34:16; echo: MOTSYNC: Final y synchronization factor is 2.531243225865282
21:34:16; echo: MOTSYNC: Final position is -16 microstep(s)
21:34:16; echo: MOTSYNC: Done!
```

I still think I have an issue with my Y axis, so jeah it looks slightly off...
