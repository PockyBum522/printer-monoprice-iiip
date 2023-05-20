# printer-monoprice-iiip

Notes and cura settings for setting up a old monoprice iiip in 2023

IMPORTANT: Do not adjust the feedrate speed on the printer while it's printing a part. Ever. It'll bug out and speed things way, way up.


# Before using

* Have a 2mm hex key on hand (Bed levelling screws)

* You can print without a filament dryer, but get it sooner rather than later. 
        Suggested: https://www.amazon.com/gp/product/B08C9RZPMN/

* Wipe the bed down with 90% or higher isopropyl before printing


# Warnings

* If the hotend is hot, you should be in the process of printing, and the fan should be on. No exceptions, you will clog the printhead. 

* If you're not about to print immediately or actively printing, then the fan should be on and the hotend should be cooling down. 

* If there's a problem, the power switch is the e-stop. Just turn it off for a second, then turn it back on, and restart a print so that the fan turns on so you don't violate the above rules. Just cancel the print immediately so that it doesn't print. The fan will remain on.

* Always, always, always, make sure you have enough filament left on the spool for the print you're doing. If you're doing a print with a lot of volume, and there's only a little filament left on the spool, don't chance it. This is similar to the first two rules in that if the filament ends, there will be filament sitting in a hot nozzle but not getting extruded. The filament will get too hot, oxidize, and clog your nozzle. You will hate your life the first time this happens, I promise. 


# Cura machine setup

In Cura, use Monoprice Select Mini V1.

Change the printer name to Monoprice IIIP

When adding the printer, in the Machine Settings windows, replace the "Start G-code" entirely with the following (Sans ``` if you're in plaintext/not markdown)

```
; ----------===== SETUP GCODE BEGIN =====----------
G21                                                          ; Metric values
G90                                                          ; Absolute positioning
M83                                                          ; Relative extrusion mode
M106                                                         ; Start with the fan at 100%. We must do this because the IIIP uses the same fan for part cooling as it does for heat-break cooling
G91                                                          ; Relative positioning to move head up
M211 S0                                                      ; Remove soft stop since machine isn't homed yet. But we don't want to home until things are hot, in case there's cold filament sticking out a little
G1 F1500                                                     ; Set default travel speed (Used when feedrate isn't explicitly specified in subsequent g commands)
G0 Z15                                                       ; Move head up a little in case it's at bed level. So the user can pull any filament that oozes, off. Also not heating the nozzle directly on the bed
M211 S1                                                      ; Re-enable soft stops
G90                                                          ; Back to absolute positioning
M140 S{material_bed_temperature_layer_0}                     ; Set Heat Bed temperature
M190 S{material_bed_temperature_layer_0}                     ; Wait for Heat Bed temperature
M104 S{material_print_temperature_layer_0}                   ; Set Extruder temperature
M109 S{material_print_temperature_layer_0}                   ; Wait for Extruder temperature
G28                                                          ; Home the printer
G92 E0                                                       ; Reset the extruder to 0
G0 X-1 Z0                                                    ; Move outside the printable area
G1 Y60 E4                                                    ; Draw a priming/wiping line to the rear
G1 X-1                                                       ; Move a little closer to the print area
G1 Y10 E4                                                    ; Draw more priming/wiping
; ----------===== SETUP GCODE FINISHED =====----------
```

And replace "End G-Code" entirely with the following (Sans ``` if you're in plaintext/not markdown)

```
; ----------===== END GCODE BEGIN =====----------
M106 255                                                     ; Cura turns off the fan at the end of the print, so uhhhh thanks buddy, but we kinda need that on cause this printer is special
M83                                                          ; Relative extrusion mode
G1 E-4                                                       ; Retract 4mm
M190 S0                                                      ; Turn off heat bed, don't wait
M104 S0                                                      ; Turn off nozzle, don't wait
G91                                                          ; Relative positioning to move head up
G0 Z2                                                        ; Move head up just a little
G90                                                          ; Back to absolute positioning
G0 X0 Y120                                                   ; Stick out the part
G4 S300                                                      ; Delay 5 minutes
M107                                                         ; Turn off part fan (For realsies this time)
M84                                                          ; Turn off stepper motors
; ----------===== END GCODE FINISHED =====----------
```

# Cura printer profile adjustments

In Cura, select printer profile "Finer" because whatever we change below will have the initial values from the selected profile. We'll save it as another profile at the end of this section.

In Cura, under print settings, click "Show Custom" and after that make sure to select expert so it shows you more settings.

Material >
        Build plate temp initial layer: 70c (Helps with crappy PID values monoprice did. We want 60c actual)

Cooling > 
    Set all fan speeds to 100%, yes really, every single one
    Set regular fan speed at height 0.0
        (Most printers have two fans on the hotend, this one is weird and only has one, so we handle turning it on in the start gcode because the heat break in the hotend must be cooled at all times, you should never, ever see the fan off when the hotend is above ~50c)

Speed >
     Print speed:   40mm/s

Build Plate Adhesion >
        Brim (Allows more time to fix any potential levelling problems. Once the bed levelling is dialed in if the part you're printing has a decent bottom surface area on the bed then use skirt instead.)

Special modes >
        Relative extrusion

Z-seam align: Random

Save as new custom profile - Name it My IIIP - Fine or something. I think custom profiles might have metadata to label them in the dropdown as to what they're based off of, but I don't know 100% how that works.


# Misc notes

* It looks like there's a dumb bug with the printer firmware where speed settings get translated into much, much faster actual real-world speeds if you change the speed on the printer display while it's printing. Even if you then change it back to 100%, it will still be going much faster than it should be.

        If you want to change the feedrate percentage, add M220 S90 or something like that in the start gcode. M220 = Override feedrate, S90 = Set it to 90% of what it should be. Remove this line if you ever put a decent FW on the printer.