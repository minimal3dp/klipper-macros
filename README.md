# klipper-macros

**Version:** v1.0.2-1

**From Minimal3DP:**

This collection of scripts are the work on [jschuh](https://github.com/jschuh/klipper-macros). As I have been using the scripts, I have started to make some small changes to suit my needs. This branch ([m3dp](https://github.com/minimal3dp/klipper-macros/tree/m3dp))is where I will make my changes.

**From jschuh:**

This is a collection of macros for the
[Klipper 3D printer firmware](https://github.com/Klipper3d/klipper). I
originally created this repo just to have a consistent set of macros shared
between my own 3D printers. But since I've found them useful, I thought other
people might as well.

## What can I do with these?

Most of these macros just improve basic functionality (e.g.
[selectable build sheets](#bed-surface)) and Klipper compatibility
with g-code targeting Marlin printers. However, there are also some nice extras:

- **[Schedule commands at heights and layer changes](#layer-triggers)** -
  This is similar to what your slicer can already do, but I find it simpler, and
  you can schedule these commands while a print is active. As an example of
  usage, I added an [LCD menu item](#lcd-menus) to pause the print at the next
  layer change. This way the pause won't mar the print by e.g. pausing inside
  an external perimeter.
- **Dynamically scale [heaters](#heaters) and [fans](#fans)** - This makes it
  easy to do things like persistently adjust fan settings during a live print,
  or maintain simpler slicer profiles by moving things like a heater bump for a
  hardened steel nozzle into state stored on the printer.
- **Cleaner [LCD menu interface](#lcd-menus)** - I've simplified the menus and
  provided a much easier way to customize materials in the LCD menu (or at least
  I think so). I've also added confirmation dialogs for commands that would
  abort an active print.
- **[Optimized mesh bed leveling](#bed-mesh-improvements)** - Probes only within
  the printed area, which can save a lot of time on smaller prints.
- **[Automated purge lines](#draw_purge_line)** - Set the desired extrusion
  length as `variable_start_purge_length` in your config and a correctly sized
  set of purge lines will be extruded in front of the print area immediately
  before the print starts.

## A few warnings...

- You must have a `heater_bed`, `extruder`, and other [sections listed
  below](#klipper-setup) configured, otherwise the macros will **_force a
  printer shutdown at startup_**. Unfortunately, the Klipper macro
  doesn't have a more graceful way of handling this sort of thing.
- The multi-extruder and chamber heater functionality is very under-tested and
  may have bugs, since I haven't used it much at all. Patches welcome.
- There's probably other stuff I haven't used enough to thoroughly, so use
  at your own risk.

# Troubleshooting

- Double check that you followed the [installation instructions](#installation)
  and are not seeing any console or log errors.
- Ensure that you're running the most current version of stock Klipper, and not
  a fork or otherwise altered or outdated copy.
- Ensure you're using the most current version of these macros and haven't
  made changes to any files in the `klipper-macros` directory.
- Ensure that you've restarted Klipper after any updates or config changes.
- Run `CHECK_KM_CONFIG` in the Klipper console and fix any errors it reports
  to the console and/or logs (it won't output anything if no config errors
  were detected).
- Run `_INIT_SURFACES` in the Klipper console to validate that bed surfaces are
  being initialized without any errors reported to the console and/or logs.
- Verify your slicer settings and review that the gcode output is correct. Pay
  particular attention the initialization portions of the gcode and the
  parameters passed to PRINT_START.

# Reporting Bugs

If you've followed the troubleshooting steps and were unable to resolve the
issue you can [report a bug via Github](https://github.com/jschuh/klipper-macros/issues/new/choose). I will probably
respond within a few days (almost certainly within a week). I probably won't
respond through other channels (e.g. Discord, Twitter), because I don't find
them useful for handling bug reports.

Some important things to remember when reporting bugs:

- **Paste the full text of the command that triggered the error, along with any
  error messages printed to the console** (and relevant sections of the klipper
  logs if appropriate).
- **Attach your config to the bug report.** There's generally no way to diagnose
  anything without the configs.
- **Verify that your issue reproduces on the current, stock installation of
  Klipper and klipper-macros.** Non-stock configurations and outdated versions
  make diagnosis nearly impossible.
- Please don't treat bug reports as a substitute for following the installation
  and troubleshooting instructions.
- If you file a feature request I will most likely close it (unless it's
  something I was already planning on adding). Sorry, but I wrote these macros
  to meet my own needs, so that's what I work on.

> **Note:** Reports that do not follow the above guidelines _**will likely be
> closed without any other action taken.**_

# Contributing

I'm happy to accept bugfix PRs. I'm also potentially open to accepting new
features or additions. However, I may decline the PR if it's something I'm not
interested in or just looks like it would be a hassle for me to maintain.

## Formatting

There's no standard style for Klipper macros, so please just try to follow the
style in the files. That stated, here are a few rules to remember:

- Wrap at 80 characters if at all possible
- Indent 2 spaces, and in line with the logical block when wrapping (no tabs)
- Prefix internal macros with `_` or `_km_`
- Prefix any sort of global state with `_KM_` (e.g. `_KM_SAVE_GCODE_STATE`)

## Commit Messages

These are the rules for commit messages, but you can also just look at the
commit log and follow the observed pattern:

- Use the 50/72 rule for commit messages: No more than 50 characters in the
  title and break lines in the description at 72 characters.
- Begin the title with the module name (usually the main file being modified,
  minus any extension) followed by a colon.
- Title-only commit messages are fine for simple commits, but be sure to
  include a blank line after the title.
- Squash multiple commits if what you're working on makes more sense as a
  single logical commit. _This might require you to do a force push on an open
  PR._

# Installation

To install the macros, first clone this repository inside of your
`printer_data/config` directory with the following command.

```
git clone https://github.com/minimal3dp/klipper-macros
```

Then paste the below sections into your `printer.cfg` to get started. Or even
better, paste all of it into a seperate file in the same path as your config,
and include that file. That will make it easier if you want to remove these
macros in the future.

You may need to customize some settings for your own config. All configurable
settings are in [globals.cfg](globals.cfg#L5), and can be overridden by creating
a corresponding variable with a new value in your `[gcode_macro _km_options]`
section. _**Do not directly modify the variable declarations in globals.cfg.**_
The macro initialization assumes certain default values, and direct
modifications are likely to break things in very unexpected ways.

> **Note:** The paths in this README follow [Moonraker's data folder structure.
> ](https://moonraker.readthedocs.io/en/latest/installation/#data-folder-structure)
> You may need to change them if you are using a different structure.

> **Note:** If you have a `[homing_override]` section you will need to update any
> `G28` commands in that section to use to `G28.6245197` instead (which is the
> renamed version of Klipper's built-in `G28`). Failure to do this will cause
> `G28` commands to error out with the message _"Macro G28 called recursively"_.

# Klipper Setup

```
# All customizations are documented in globals.cfg. Just copy a variable from
# there into the section below, and change the value to meet your needs.

[gcode_macro _km_options]
# These are examples of some likely customizations:
# Any sheets in the below list will be available with a configurable offset.
#variable_bed_surfaces: ['smooth_1','texture_1']
# Length (in mm) of filament to load (bowden tubes will be longer).
variable_load_length: 50.0
# Hide the Octoprint LCD menu since I don't use it.
#variable_menu_show_octoprint: False
# Customize the filament menus (up to 10 entries).
#variable_menu_temperature: [
#  {'name' : 'PLA',  'extruder' : 200.0, 'bed' : 60.0},
#  {'name' : 'PETG', 'extruder' : 230.0, 'bed' : 85.0},
#  {'name' : 'ABS',  'extruder' : 245.0, 'bed' : 110.0, 'chamber' : 60}]
# Length of filament (in millimeters) to purge at print start.
variable_start_purge_length: 30 # This value works for most setups.
gcode: # This line is required by Klipper.
# Any code you put here will run at klipper startup, after the initialization
# for these macros. For example, you could uncomment the following line to
# automatically adjust your bed surface offsets to account for any changes made
# to your Z endstop or probe offset.
#  ADJUST_SURFACE_OFFSETS

# This line includes all the standard macros.
[include klipper-macros/*.cfg]
# Uncomment to include features that require specific hardware support.
# LCD menu support for features like bed surface selection and pause next layer.
#[include klipper-macros/optional/lcd_menus.cfg]
# Optimized bed leveling
[include klipper-macros/optional/bed_mesh.cfg]

# The sections below here are required for the macros to work.
[idle_timeout]
gcode:
  _KM_IDLE_TIMEOUT

[pause_resume]

[respond]

[save_variables]
filename: ~/e3v2_data/config/variables.cfg # UPDATE THIS FOR YOUR PATH!!!

[virtual_sdcard]
path: ~/e3v2_data/gcodes

[display_status]

# Uncomment the sections below if Fluidd complains (because it's confused).
#
#[gcode_macro CANCEL_PRINT]
#rename_existing: CANCEL_PRINT_BASE
#gcode: CANCEL_PRINT_BASE{% for k in params %}{' '~k~'='~params[k]}{% endfor %}
```

## Moonraker Configuration

Paste the following into your `moonraker.conf` if you want the macros to
automatically update directly from this repo.

```
[update_manager klipper-macros]
type: git_repo
origin: https://github.com/minimal3dp/klipper-macros.git
path: ~/e3v2_data/config/klipper-macros # UPDATE THIS FOR YOUR PATH!!!
primary_branch: m3dp
is_system_service: False
managed_services: klipper
```

## Slicer Configuration

From Minimal3DP:

Using a Purge line -

### PrusaSlicer / SuperSlicer

PrusaSlicer and its variants are fairly easy to configure. Just open **Printer
Settings → Custom G-code** for your Klipper printer and paste the below text
into the relevant sections.

#### Start G-code

```
M190 S0
M109 S0
PRINT_START EXTRUDER={first_layer_temperature[initial_tool]} BED=[first_layer_bed_temperature] MESH_MIN={first_layer_print_min[0]},{first_layer_print_min[1]} MESH_MAX={first_layer_print_max[0]},{first_layer_print_max[1]} LAYERS={total_layer_count} NOZZLE_SIZE={nozzle_diameter[0]}

; This is the place to put slicer purge lines if you haven't set a non-zero
; variable_start_purge_length to have START_PRINT automatically purge (e.g. if
; using a Mosaic Palette, which requires the slicer to generate the purge).
```

#### End G-code

```
PRINT_END
```

#### Before layer change G-code

```
;BEFORE_LAYER_CHANGE
;[layer_z]
BEFORE_LAYER_CHANGE HEIGHT=[layer_z] LAYER=[layer_num]
```

#### After layer change G-code

```
;AFTER_LAYER_CHANGE
;[layer_z]
AFTER_LAYER_CHANGE
```

### Ultimaker Cura

Cura is a bit more difficult to configure, and it comes with the following known
issues:

- Cura doesn't have proper placeholders for before and after layer changes, so
  the before triggers all fire and are followed immediately by the after
  triggers, all of which happens inside the layer change. This probably doesn't
  matter, but it does mean that you can't use the before and after triggers to
  avoid running code in the layer change.
- Cura doesn't provide the Z-height of the current layer, so it's inferred from
  the current nozzle position, which will include the Z-hop if the nozzle is
  currently raised. This means height based gcode triggers may fire earlier than
  expected.
- Cura's **Insert at layer change** fires the `After` trigger and then the
  `Before` trigger (i.e before or after the _layer_, versus before or after the
  _layer change_). These macros and PrusaSlicer do the opposite, which is
  something to keep in mind if you're used to how Cura does it. Note that these
  macros do use an **Insert at layer change** script to force `LAYER` comment
  generation, but that doesn't affect the trigger ordering.
- Cura does not provide the first layer bounding rectangle, only the model
  bounding volume. This means the XY bounding box used to speed up mesh probing
  may be larger than it needs to be, resulting in bed probing that's not as fast
  as it could be.

Accepting the caveats, the macros work quite well with Cura if you follow the
configuration steps listed below.

#### Start G-code

```
M190 S0
M109 S0
PRINT_START EXTRUDER={material_print_temperature_layer_0} BED={material_bed_temperature_layer_0} MESH_MIN=%MINX%,%MINY% MESH_MAX=%MAXX%,%MAXY% NOZZLE_SIZE={machine_nozzle_size}

; This is the place to put slicer purge lines if you haven't set a non-zero
; variable_start_purge_length to have START_PRINT automatically purge (e.g. if
; using a Mosaic Palette, which requires the slicer to generate the purge).
```

#### End G-code

```
PRINT_END
```

#### Post Processing Plugin

**From Minimal3DP:**

I found 2 methods that work for the post processing scripts. I believe that the method I am using is slightly easier than the method suggested by **jschuh**. Either should work.

_Minimal3DP Method:_

To make the macro to work in Cura slicer, you need to install the [post process plugin by frankbags](https://gist.github.com/frankbags/c85d37d9faff7bce67b6d18ec4e716ff)

- In cura menu <code>Help</code> -> <code>Show configuration folder</code>.
- Copy the python script from the above link (or [.\cura_scripts](https://github.com/minimal3dp/klipper-macros/tree/m3dp/cura_scripts)) in to Cura <code>scripts</code> folder. In windows, it should be something like: "C:\Users\m3dp\AppData\Roaming\cura\5.2\scripts"
- Restart Cura
- In cura menu <code>Extensions</code> -> <code>Post processing</code> -> <code>Modify G-Code</code> and select <code>Mesh Print Size</code>

_jschuh Method:_

Use the menu item for **Extensions → Post Processing → Modify G-Code** to
open the **Post Processing Plugin** and add the following four scripts. _The
scripts must be run in the order listed below and be sure to copy the strings
exactly, with no leading or trailing spaces._

##### Search and Replace

- Search: `(\n;(MIN|MAX)X:([^\n]+)\n;\2Y:([^\n]+))`
- Replace: `\1\nPRINT_START_SET MESH_\2=\3,\4`
- Use Regular Expressions: ☑️

##### Search and Replace

- Search: `(\n;LAYER_COUNT:([^\n]+))`
- Replace: `\1\nINIT_LAYER_GCODE LAYERS=\2\nPRINT_START_SET LAYERS=\2`
- Use Regular Expressions: ☑️

##### Insert at layer change

- When to insert: `Before`
- G-code to insert: `;BEFORE_LAYER_CHANGE`

##### Search and Replace

- Search: `(\n;LAYER:([^\n]+))`
- Replace: `\1\nBEFORE_LAYER_CHANGE LAYER=\2\nAFTER_LAYER_CHANGE`
- Use Regular Expressions: ☑️

# Command Reference

## Customization

All features are configured by setting `variable_` values in the
`[gcode_macro _km_options]` section. All available variables and their purpose
are listed in [globals.cfg](globals.cfg#L5).

> **Note:** `PRINT_START` specific customizations are [covered in more detail
> below](#print-start-and-end).

### Bed Mesh Improvements

`BED_MESH_CALIBRATE_FAST`

Wraps the Klipper `BED_MESH_CALIBRATE` command to scale and redistribute the
probe points so that only the appropriate area in `MESH_MIN` and `MESH_MAX` is probed. This can dramatically reduce probing times for anything that doesn't
fill the first layer of the bed. `PRINT_START` will automatically use this for
bed mesh calibration if a `[bed_mesh]` section is detected in your config.

The following additional configuration options are available from
[globals.cfg](globals.cfg#L5).

- `variable_probe_mesh_padding` - Extra padding around the rectangle defined by
  `MESH_MIN` and `MESH_MAX`.
- `variable_probe_min_count` - Minimum number of probes for partial probing of a
  bed mesh.
- `variable_probe_count_scale` - Scaling factor to increase probe count for
  partial bed probes.

> **Note:** See the [optional section](#bed-mesh) for additional macros.

> **Note:** The bed mesh optimizations are silently disabled for delta printers
> (because jinja2 lacks the necessary math support) and when the mesh parameters
> include a [`RELATIVE_REFERENCE_INDEX`](https://www.klipper3d.org/Bed_Mesh.html#the-relative-reference-index)
> (which is incompatible with dynamic mesh generation).

`BED_MESH_CHECK`

Checks the `[bed_mesh]` config and warns if `mesh_min` or `mesh_max` could allow
a move out of range during `BED_MESH_CALIBRATE`. This is run implicitly at
Klipper startup and at the start of `BED_MESH_CALIBRATE`.

### Bed Surface

Provides a set of macros for selecting different bed surfaces with
corresponding Z offset adjustments to compensate for their thickness. All
available surfaces must be listed in the `variable_bed_surfaces` array.
Corresponding LCD menus for sheet selection and babystepping will be added to
_Setup_ and _Tune_ if [`lcd_menus.cfg`](#lcd-menus) is included. Any Z offset
adjustments made in the LCD menus, console, or other clients (e.g. Mainsail,
Fluidd) will be applied to the current sheet and persisted across restarts.

Lists all available surfaces.

#### `SET_SURFACE_ACTIVE`

Sets the provided surface active (from one listed in listed in
`variable_bed_surfaces`) and adjusts the current Z offset to match the
offset for the surface. If no `SURFACE` argument is provided the available
surfaces are listed, with active surface preceded by a `*`.

- `SURFACE` - Bed surface with an associated offset.

#### `SET_SURFACE_OFFSET`

Directly sets the the Z offset of `SURFACE` to the value of `OFFSET`. If no
argument for `SURFACE` is provided the current active surface is used. If no
argument for `OFFSET` is provided the current offset is displayed.

- `OFFSET` - New Z offset for the given surface.
- `SURFACE` _(default: current surface)_ - Bed surface.

> **Note:** The `SET_GCODE_OFFSET` macro is overridden to update the
> offset for the active surface. This makes the bed surface work with Z offset
> adjustments made via any interface or client.

#### `ADJUST_SURFACE_OFFSETS`

Adjusts surface offsets to account for changes in the Z endstop position or
probe Z offset. A message to invoke this command will be shown in the console
when a relevant change is made to `printer.cfg`.

- `IGNORE` - Set to 1 to reset the saved offsets without adjusting the surfaces.

### Beep

Implements the M300 command (if a corresponding `[output_pin beeper]` section is
present). This command causes the speaker to emit an audible tone.

#### `M300`

Emits an audible tone.

- `P` _(default: `variable_beep_duration`)_ - Duration of tone.
- `S` _(default: `variable_beep_frequency`)_ - Frequency of tone.

### Draw

Provides convenience methods for extruding along a path and drawing purge lines.

> **Note:** The drawing macros require every `extruder` config(s) to have
> correct `nozzle_diameter` and `filament_diameter` settings.

#### DRAW_LINE_TO

Extrudes a line of filament at the specified height and width from the current
coordinate to the supplied XY coordinate.

- `X` _(default: current X position)_ - Absolute X coordinate to draw to.
- `Y` _(default: current Y position)_ - Absolute Y coordinate to draw to.
- `HEIGHT` _(default: set via `SET_DRAW_PARAMS`)_ - Height (in mm) used to
  calculate extrusion volume.
- `WIDTH` _(default: set via `SET_DRAW_PARAMS`)_ - Extrusion width in mm.
- `FEEDRATE` _(default: set via `SET_DRAW_PARAMS`)_ - Drawing feedrate in mm/m.

> **Note:** The Z axis position must be set prior to calling this macro. The
> `HEIGHT` parameter is used only to calculate the extrusion volume.

#### SET_DRAW_PARAMS

Sets the default parameters used by DRAW_LINE_TO. This is helpful in reducing
`DRAW_LINE_TO` command line lengths (particularly important when debugging in
the console).

- `HEIGHT` _(optional; 0.2mm at startup)_ - Height (in mm) used to
  calculate extrusion volume.
- `WIDTH` _(optional; nozzle diameter at startup)_ - Extrusion width in mm.
- `FEEDRATE` _(optional; 1200mm/m at startup)_ - Drawing feedrate in mm/m.

#### DRAW_PURGE_LINE

Moves to a position at the front edge of the first print layer and purges the
specified length of filament as a line (or rows of lines) in front of the
supplied print area. If no print area is specified the purge lines are drawn at
the front edge of the maximum printable area. If no printable area is set it
defaults to the respective axis limits.

- `PRINT_MIN` _(default: `variable_print_min`)_ - Upper boundary of print.
- `PRINT_MAX` _(default: `variable_print_max`)_ - Lower boundary of print.
- `HEIGHT` _(default: 62.5% of nozzle diameter)_ - Extrusion height in mm.
- `WIDTH` _(default: 125% of nozzle diameter)_ - Extrusion width in mm.
- `LENGTH` _(default: `variable_start_purge_length`)_ - Length of filament
  to purge. _The default in `variable_start_purge_length` is also the amount
  that is automatically purged at print start._

> **Note:** You must set `variable_print_min` and `variable_print_max` if the
> X and Y axis limits in your config allow your toolhead to move outside the
> printable area (e.g. for dockable probes or purge buckets).

> **Note:** If your print touches the front edge of the bed it will overlap with
> with the extrusions from `DRAW_PURGE_LINE`.

### Fans

Implements scaling parameters that alter the behavior of the M106 command. Once
set, these parameters apply to any fan speed until they are cleared.

#### `SET_FAN_SCALING`

Sets scaling parameters for the extruder fan.

- `BOOST` _(default: `0`)_ - Added to the fan speed.
- `SCALE` _(default: `1.0`)_ - The `BOOST` value is added an then the fan
  speed is multiplied by `SCALE`.
- `MAXIMUM` _(default: `255`)_ - The fan speed is clamped to no larger
  than `MAXIMUM`.
- `MINIMUM` _(default: `0`)_ - The fan speed is clamped to no less
  than `MINIMUM`; if this is a non-zero value the fan can be stopped only via
  the M107 command.
- `SPEED` _(optional)_ - This specifies a new speed target, otherwise any new
  adjustments will be applied to the unadjusted value of the last set fan speed.

#### `RESET_FAN_SCALING`

- Clears all existing fan scaling factors.

### Filament

#### `LOAD_FILAMENT` / `UNLOAD_FILAMENT`

Loads or unloads filament to the nozzle.

- `LENGTH` _(default: `variable_load_length`)_ - The length of filament to load
  or unload.
- `SPEED` _(default: `variable_load_speed`)_ - Speed (in mm/m) to feed the
  filament.
- `MINIMUM` _(default: `min_extrude_temp` + 5)_ - Ensures the extruder is heated
  to at least the specified temperature.

#### Marlin Compatibility

- The `M701` and `M702` commands are implemented with a default filament length
  of `variable_load_length`.

### Heaters

Adds scaling parameters that can alter the behavior of the specified heater.
Once set, these parameters apply to any temperature target on that heater until
the scaling parameters are cleared. A zero target temperature will turn the
heater off regardless of scaling parameters.

#### `SET_HEATER_SCALING`

Sets scaling parameters for the specified heater. If run without any arguments
any currently scaled heaters and their scaling parameters will be listed.

- `HEATER` - The name of the heater to scale.
- `BOOST` _(default: `0.0`)_ - Added to a non-zero target temperature.
- `SCALE` _(default: `1.0`)_ - Multiplied with the boosted target
  temperature.
- `MAXIMUM` _(default: `max_temp`)_ - The target temperature is clamped
  to no larger than `MAXIMUM`. This value must be between `min_temp` and
  `min_temp`, inclusive.
- `MINIMUM` _(default: `min_temp`)_ - A non-zero target temperature is
  clamped to no less than `MINIMUM`. This value must be between `min_temp` and
  `min_temp`, inclusive.
- `TARGET` _(optional)_ - This specifies a new target temperature, otherwise any
  new adjustments will be applied to the unadjusted value of the last set target
  temperature.

> **Note:** a zero target temperature will turn the heater off regardless of
> scaling parameters.

#### `RESET_HEATER_SCALING`

Clears current heater scaling.

- `HEATER` _(optional)_ - The name of the heater to reset.

> **Note:** if no HEATER argument is specified scaling parameters will be reset
> for all heaters.

#### `SET_HEATER_TEMPERATURE_SCALED`

The scaled version of Klipper's `SET_HEATER_TEMPERATURE`. All arguments are the
same and the function is identical, except that scaling values are applied.

#### `TEMPERATURE_WAIT_SCALED`

The scaled version of Klipper's `TEMPERATURE_WAIT`. All arguments are the
same and the function is identical, except that scaling values are applied.

#### Marlin Compatibility

- The chamber heating commands `M141` and `M191` are implemented if a
  corresponding `[heater_generic chamber]` section is defined in the config.
- The `R` temperature parameter from Marlin is implemented for the `M109` and
  `M190` commands. This parameter will cause a wait until the target temperature
  stabilizes (i.e. the normal Klipper behavior for `S`).
- The `S` parameter for the `M109` and `M190` commands is altered to behave as
  it does in Marlin. Rather than causing a wait until the temperature
  stabilizes, the wait will complete as soon as the temperature target is
  exceeded.
- The `M109`, `M190`, `M191`, `M104`, `M140`, and `M141` are all overridden to
  implement the heater scaling described above.

> **Note:** Both `SET_HEATER_TEMPERATURE` and `TEMPERATURE_WAIT` are **not**
> overridden and will not scale values. This means that heater scaling
> adjustments made in clients like Mainsail and Fluidd will not be scaled
> (because that seemed like the clearest behavior). The
> [custom LCD menus](#lcd-menus) will also replace the temperature controls
> with non-scaling versions. If you use the stock menus you'll get scaled
> values.

### Kinematics

#### `G28`

Extends the `G28` command to add lazy homing by not re-homing already homed axes
when the `O` argument is included (equivalent to the same argument in Marlin).
See Klipper `G28` documentation for general information and detail on the other
arguments.

- `O` - Omits axes from the homing procedure if they are already homed.

### Layer Triggers

Provides the capability to run user-specified g-code commands at arbitrary layer
changes.

#### `GCODE_AT_LAYER`

Runs arbitrary, user-provided g-code commands at the user-specified layer or
height. If no arguments are specified it will display all currently scheduled
g-code commands along with their associated layer or height.

- `HEIGHT` - Z height (in mm) to run the command. Exactly one of `HEIGHT` or
  `LAYER` must be specified.
- `LAYER` - Layer number (zero indexed) to run the command. Exactly one of
  `HEIGHT` or `LAYER` must be specified. The special value `next` may be
  specified run the command at the next layer change.
- `COMMAND` - The command to run at layer change. Take care to properly quote
  spaces and escape any special characters.
- `BEFORE` _(default: `0`)_ - Set to 1 run the command before the layer
  change (i.e. immediately following completion of the previous layer). By
  default commands run after the layer change (i.e. immediately preceding the
  next layer). In most cases this distinction here doesn't matter, but it can
  be important when dealing with tool changers or other multi-material printing.

#### `CANCEL_ALL_LAYER_GCODE`

Cancels all g-code commands previously scheduled at any layer or height.

#### Convenient Layer Change Macros

- `PAUSE_NEXT_LAYER ...`
  - Schedules the current print to pause at the next layer change. See
    [`PAUSE`](#pause) macro for additional arguments.
- `PAUSE_AT_LAYER  { HEIGHT=<pos> | LAYER=<layer> } ...`
  - Schedules the current print to pause at the specified layer change.
    See [`PAUSE`](#pause) for additional arguments.
- `SPEED_AT_LAYER { HEIGHT=<pos> | LAYER=<layer> } SPEED=<percentage>`
  - Schedules a feedrate adjustment at the specified layer change. (`SPEED`
    parameter behaves the same as the `M220` `S` parameter.)
- `FLOW_AT_LAYER { HEIGHT=<pos> | LAYER=<layer> } FLOW=<percentage>`
  - Schedules a flow-rate adjustment at the specified layer change. (`FLOW`
    parameter behaves the same as the `M221` `S` parameter.)
- `FAN_AT_LAYER { HEIGHT=<pos> | LAYER=<layer> } ...`
  - Schedules a fan adjustment at the specified layer change. See
    [`SET_FAN_SCALING`](#set_fan_scaling) for additional arguments.
- `HEATER_AT_LAYER { HEIGHT=<pos> | LAYER=<layer> } ...`
  - Schedules a heater adjustment at the specified layer change. See
    [`SET_HEATER_SCALING`](#set_heater_scaling) for additional arguments.

> **Note:** If any triggers cause an exception the current print will
> abort. The convenience macros above validate their arguments as much as is
> possible to reduce the chances of an aborted print, but they cannot entirely
> eliminate the risk of a macro doing something that aborts the print.

### Park

Implements toolhead parking.

#### `PARK`

Parks the toolhead.

- `P` _(default: `2`)_ - Parking mode
  - `P=0` - If current Z-pos is lower than Z-park then the nozzle will be raised
    to reach Z-park height
  - `P=1` - No matter the current Z-pos, the nozzle will be raised/lowered to
    reach Z-park height.
  - `P=2` - The nozzle height will be raised by Z-park amount but never going
    over the machine’s Z height limit.
- `X` _(default: `variable_park_x`)_ - Absolute X parking coordinate.
- `Y` _(default: `variable_park_y`)_ - Absolute Y parking coordinate.
- `Z` _(default: `variable_park_z`)_ - Z parking coordinate applied according
  to the `P` parameter.
- `LAZY` _(default: 1)_ - Will home any unhomed axes if needed and will not
  move any axis if already homed and parked (even if `P=2`).

> **Note:** If a print is in progress the larger of the tallest printed layer or
> the current Z position will be used as the current Z position, to avoid
> collisions with already printed objects during a sequential print.

#### Marlin Compatibility

- The `G27` command is implemented with a default `P0` argument.

### Pause, Resume, Cancel

#### `PAUSE`

Pauses the current print.

- `X` _(default: `variable_park_x`)_ - Absolute X parking coordinate.
- `Y` _(default: `variable_park_y`)_ - Absolute Y parking coordinate.
- `Z` _(default: `variable_park_z`)_ - Relative Z parking coordinate
- `E` _(default: `5`)_ - Retraction length to prevent ooze.
- `B` _(default: `10`)_ - Number of beeps to emit (if `M300` is enabled).

#### `RESUME`

- `E` _(default: `5`)_ - Retraction length to prevent ooze.

#### `CANCEL_PRINT`

Cancels the print and performs all the same functions as `PRINT_END`.

#### Marlin Compatibility

- The `M24`, `M25`, `M600`, `M601`, and `M602` commands are all implemented by
  wrapping the above commands.

### Print Start and End

#### `PRINT_START`

Sets up the printer prior to starting a print (called from the slicer's print
start g-code). A target `CHAMBER` temperature may be provided if a
`[heater_generic chamber]` section is present in the klipper config.
If `MESH_MIN` and `MESH_MAX` are provided, then `BED_MESH_CALIBRATE` will probe
only the area within the specified rectangle, and will scale the number of
probes to the appropriate density (this can dramatically reduce probe times for smaller prints).

- `BED` - Bed heater starting temperature.
- `EXTRUDER` - Extruder heater starting temperature.
- `CHAMBER` _(optional)_ - Chamber heater starting temperature.
- `MESH_MIN` _(optional)_ - Minimum x,y coordinate of the first layer.
- `MESH_MAX` _(optional)_ - Maximum x,y coordinate of the first layer.
- `NOZZLE_SIZE` _(default: nozzle_diameter)_ - Nozzle diameter of the primary
  extruder.
- `LAYERS` _(optional)_ - Total number of layers in the print.

These are the customization options you can add to your
`[gcode_macro _km_options]` section to alter `PRINT_START` behavior:

- `variable_start_bed_heat_delay` _(default: 2000)_ - This delay (in
  microseconds) is used to allow the bed to stabilize after it reaches it's
  target temperature. This is present to account for the fact that the
  temperature sensors for most beds are located close to the heating element,
  and thus will register as being at the target temperature before the surface
  of the bed is. For larger or thicker beds you may want to increase this value.
  For smaller or thinner beds you may want to disable this entirely by setting
  it to `0`.

- `variable_start_bed_heat_overshoot` _(default: 2.0)_ - This value (in degrees
  Celsius) is added to the supplied target bed temperature and use as the
  initial target temperature when preheating the bed. After the bed preheats to
  this target it there is a brief delay before the final target is set. This
  allows the bed to stabilize at it's final temperature more quickly. For
  smaller or thinner beds you may want to reduce this value or disable it
  entirely by setting it to `0.0`.

- `variable_start_end_park_y` _(default: `print_max` Y coordinate)_ - The final
  Y position of the toolhead in the `PRINT_END` macro, to ensure that the
  toolhead is out of the way when the bed is presented for print removal.

- `variable_start_extruder_preheat_scale` _(default: 0.5)_ - This value is
  multiplied by the target extruder temperature and the result is used as the
  preheat value for the extruder while the bed is heating. This is done to
  reduce oozing from the extruder while the bed is heating or being probed. Set
  to `1.0` to preheat the extruder to the full target temperature, or to `0.0`
  to not preheat the extruder at all until the bed reaches temperature.

- `variable_start_extruder_set_target_before_level` _(default: True)_ - If
  `True` the extruder is set to its target temperature before bed leveling
  begins. If `False` the target is set after bed level completes. Setting `True`
  warms up the extruder faster and `False` prevents oozing during bed level.
  The extruder preheat is applied independent of this setting.

- `variable_start_gcode_before_print` _(default: None)_ - Optional user-supplied
  gcode run after any leveling operations are complete and the bed, extruder,
  and chamber are all stabilized at their target temperatures. Immediately after
  this gcode executes the purge line will be printed (if specified) and then the
  file from the virtual sdcard will begin printing. This is a useful to add any
  probe docking commands, loading from a multi-material unit, or other
  operations that must occur before any filament is extruded.

- `variable_start_level_bed_at_temp` _(default: True if `bed_mesh` configured
  )_ - If true the `PRINT_START` macro will run [`BED_MESH_CALIBRATE_FAST`](#bed-mesh-improvements) after the bed has stabilized at its target
  temperature.

- `variable_start_home_z_at_temp` _(default: True if `probe:z_virtual_endstop`
  configured)_ - Rehomes the Z axis once the bed reaches its target temperature,
  to account for movement during heating.

- `variable_start_clear_adjustments_at_end` _(default: True)_ - Clears temporary
  adjustments after the print completes or is cancelled (e.g. feedrate,
  flow percentage).

- `variable_start_purge_clearance` _(default: 5.0)_ Distance (in millimeters)
  between the purge lines and the print area (if a `start_purge_length` is
  provided).

- `variable_start_purge_length` _(default: 0.0)_ - Length of filament (in
  millimeters) to purge after the extruder finishes heating and prior to
  starting the print. For most setups `30` is a good starting point.

- `variable_start_purge_prime_length` _(default: 10.0)_ Length of filament (in
  millimeters) to prime the extruder before drawing the purge lines.

- `variable_start_quad_gantry_level_at_temp` _(default: True if
  `quad_gantry_level` configured)_ - If true the `PRINT_START` macro will run
  `QUAD_GANTRY_LEVEL` after the bed has stabilized at its target temperature.

- `variable_start_z_tilt_adjust_at_temp` _(default: True if `z_tilt`
  configured)_ - If true the `PRINT_START` macro will run `Z_TILT_ADJUST` after
  the bed has stabilized at its target temperature.

You can further customize the `PRINT_START` macro by declaring your own override
wrapper. This can be useful for things like loading mesh/skew profiles, or any
other setup that may need to be performed prior to printing.

Here's a skeleton of a `PRINT_START` override wrapper:

```
[gcode_macro PRINT_START]
rename_existing: KM_PRINT_START
gcode:

  # Put macro code here to run before PRINT_START.

  KM_PRINT_START {rawparams}

  # Put macro code here to run after PRINT_START but before the print gcode
```

> **Note:** You can use this same pattern to wrap other macros in order to
> account for customizations specific to your printer. E.g. If you have a
> dockable probe you may choose to wrap `BED_MESH_CALIBRATE` with the
> appropriate docking/undocking commands.

#### `PRINT_END`

Parks the printhead, shuts down heaters, fans, etc., and performs general state
housekeeping at the end of the print (called from the slicer's print end
g-code).

### Velocity

These are some basic wrappers for Klipper's analogs to some of Marlin's velocity
related commands, such as acceleration, jerk, and linear advance.

#### Marlin Compatibility

- The `M201`, `M203`, `M204`, and `M205` commands are implemented by calling
  Klipper's `SET_VELOCITY_LIMIT` command. For calls that set the `ACCEL`
  parameter, the `ACCEL_TO_DECEL` parameter is also set and scaled by
  `variable_velocity_decel_scale` _(default: `0.5`)_.
- The `M900` command is implemented by calling Klipper's `SET_PRESSURE_ADVANCE`
  command. The `K` factor is scaled by `variable_pressure_advance_scale`
  _(default: `-1.0`)_. If the scaling value is negative the `M900` command has no
  effect.

## Optional Configs

### Bed Mesh

`BED_MESH_CALIBRATE` and `G20`

Overrides the default `BED_MESH_CALIBRATE` to use `BED_MESH_CALIBRATE_FAST`
instead, and adds the `G20` command.

**_Configuration:_**

```
[include klipper-macros/optional/bed_mesh.cfg]
```

**\*Requirements:** A properly configured `bed_mesh` section.\*

### LCD Menus

Adds relevant menu items to an LCD display and improves some existing
functionality. See the [customization](#customization) section above for more
information on how to configure specific behaviors.

- Confirmation added for cancelling the print or disabling steppers during a
  print.
- Several temperature menu changes:
  - Up to 10 filaments and their corresponding temperatures can be set via
    `variable_menu_temperature`.
  - Per filament chamber temperature controls are available if a
    `[heater_generic chamber]` section is configured.
  - The cooldown commands are moved to the top level temperature menu.
- The filament loading commands are replaced with macros that use the lengths
  and speeds specified in `variable_load_length` and `variable_load_speed`,
  which includes a priming phase at the end of the load (controlled via
  `variable_load_priming_length` and `variable_load_priming_speed`).
- [Bed surface](#bed-surface) management is integrated into the setup and tuning
  menus.
- The SD card menu has been streamlined for printing and non-printing modes.
- The setup menu includes host shutdown, host restart, speed, and flow controls.
- You can hide the Octoprint or SD card menus if you don't use them
  (via `variable_menu_show_octoprint` and `variable_menu_show_sdcard`,
  respectively).

**_Configuration:_**

```
[include klipper-macros/optional/lcd_menus.cfg]
```

**\*Requirements:** A properly configured `display` section.\*
