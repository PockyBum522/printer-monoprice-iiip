# printer-monoprice-iiip

Notes and cura settings for setting up a old monoprice iiip in 2023

IMPORTANT: Do not adjust the feedrate speed on the printer while it's printing a part. Ever. It'll bug out and speed things way, way up.

# Before using

* Have a 2mm hex key on hand (Bed levelling screws)

* You can print without a filament dryer, but get it sooner rather than later. 
        Suggested: https://www.amazon.com/gp/product/B08C9RZPMN/

* Wipe the bed down with 90% or higher isopropyl before printing

# Cura machine setup

In Cura, use Monoprice Select Mini V1.

Change the printer name to Monoprice IIIP

When adding the printer, in the Machine Settings windows, replace the "Start G-code" entirely with the following (Sans ``` if you're in plaintext/not markdown)

```
; ----------===== SETUP GCODE BEGIN =====----------    NOTE: Remove stupid monoprice FW line if not using stupid monoprice FW
G21                                                          ; Metric values
G90                                                          ; Absolute positioning
M83                                                          ; Relative extrusion mode
M106                                                         ; Start with the fan at 100%. We must do this because the IIIP uses the same fan for part cooling as it does for heat-break cooling
M83                                                          ; Relative positioning to move head up
M220 S25                                                     ; FOR STUPID MONOPRICE FW ONLY: Set feedrate percent to very low because the FW is stupid. Remove this if you put a custom FW on the printer
M211 S0                                                      ; Remove soft stop since machine isn't homed yet. But we don't want to home until things are hot, in case there's cold filament sticking out a little
G1 F2000                                                     ; Set default travel speed (Used when feedrate isn't explicitly specified in subsequent g commands)
G0 Z20                                                       ; Move head up a little in case it's at bed level. This way the user can pull any filament that oozes out off. Also it's not heating the nozzle directly on the bed
M211 S1                                                      ; Re-enable soft stops
G90                                                          ; Back to absolute positioning
M140 S{material_bed_temperature_layer_0}                     ; Set Heat Bed temperature
M190 S{material_bed_temperature_layer_0}                     ; Wait for Heat Bed temperature
M104 S{material_print_temperature_layer_0}                   ; Set Extruder temperature
M109 S{material_print_temperature_layer_0}                   ; Wait for Extruder temperature
G28                                                          ; Home the printer
G28 X0 Y0                                                    ; move X/Y to min endstops
G28 Z0                                                       ; move Z to min endstops
G92 E0                                                       ; Reset the extruder to 0
G0 X-1 Z0                                                    ; Move outside the printable area
G1 Y60 E3                                                    ; Draw a priming/wiping line to the rear
G1 X-1                                                       ; Move a little closer to the print area
G1 Y10 E3                                                    ; Draw more priming/wiping
; ----------===== SETUP GCODE FINISHED =====----------
```

And replace "End G-Code" entirely with the following (Sans ``` if you're in plaintext/not markdown)

```
; ----------===== END GCODE BEGIN =====----------
M106 255                                                          ; Cura turns off the fan at the end of the print, so uhhhh thanks buddy, but we kinda need that on cause this printer is special
G1 E-2                                                       ; Retract 3mm
M190 S0                                                      ; Turn off heat bed, don't wait
M104 S0                                                      ; Turn off nozzle, don't wait
M83                                                          ; Relative positioning to move head up
G0 Z5                                                        ; Move head up just a little
G90                                                          ; Back to absolute positioning
G0 X0 Y120                                                   ; Stick out the part
G92 E10                                                      ; Set extruder to 10
G4 S300                                                      ; Delay 5 minutes
M107                                                         ; Turn off part fan (For realsies this time)
M84                                                          ; Turn off stepper motors
; ----------===== END GCODE FINISHED =====----------
```


# Cura printer profile adjustments

In Cura, select printer profile "Finer" because whatever we change below will have the initial values from the selected profile. We'll save it as another profile at the end of this section.

In Cura, under print settings, click "Show Custom" and after that make sure to select expert so it shows you more settings.

Material >
        Build plate temp initial layer: 70c (Helps with crappy PID values monoprice did. We're aiming for 60c(ish))

Cooling > 
    Set all fan speeds to 100%, yes really, every single one
    Set regular fan speed at height 0.0
        (Most printers have two fans on the hotend, this one is weird and only has one, so we handle turning it on in the start gcode because the heat break in the hotend must be cooled at all times, you should never, ever see the fan off when the hotend is above ~50c)

Speed >
     Print speed:   40mm/s

Build Plate Adhesion >
        Brim (Allows more time to fix any potential levelling problems. You don't have to use this once the bed levelling is dialed in if the part you're printing has a decent bottom surface area on the bed.)

Special modes >
        Relative extrusion

Save as new custom profile - Name it My IIIP - Fine or something


# Misc notes

* It looks like there's a dumb bug with the printer firmware where speed settings get translated into much, much faster actual real-world speeds. 

        This is fixed with the M220 S20 in the start gcode. M220 = Override feedrate, S20 = Set it to 20% of what it should be. Remove this line if you ever put a decent FW on the printer or it'll print way slower than what you have for print speed in Cura.


# Calibration cube notes

Estimated time:
        1h 15m

25%

Print started at:
        14:04
