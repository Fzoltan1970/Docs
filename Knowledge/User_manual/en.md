# LaserBase -- User Manual

This manual describes how LaserBase works. The goal is not to exhaust
every internal detail, but to make the program usable in a correct way
and to make it clear what happens between the image and the G-code.

The text follows the real workflow.

------------------------------------------------------------------------

# 1. What LaserBase Is

LaserBase is a software suite for preparing and running laser engraving
jobs.

Its main parts are:

- **Main window** -- parameter database and record management
- **Image Workspace** -- image processing, preview, G-code export
- **Sender** -- sending G-code to the machine
- **Sketch** -- simple drawing and sketching surface

LaserBase is not a general-purpose image editor. Its purpose is to turn an
image into a machine-aligned raster and a usable control file.

------------------------------------------------------------------------

# 2. The Basis of Engraving

The engraving result is determined by three factors together:

1.  laser power
2.  travel speed
3.  point or line density

For this reason, LaserBase does not process only an image. It also binds
the image to a physical size, a DPI, machine geometry and G-code
parameters.

------------------------------------------------------------------------

# 3. The Full Workflow

A typical job looks like this:

1.  load the image
2.  set size, DPI and machine data
3.  set crop and geometry
4.  choose the processing mode
5.  check the preview
6.  set speed and power, or use the automatic recommendation
7.  save G-code or the processed image
8.  save a record and reload it later if needed

------------------------------------------------------------------------

# 4. Image Workspace

The Image Workspace is the central part of the program.

The source image is shown on the left, the processed view on the right.
That right-hand view does not always mean the same thing: depending on the
selected mode it can be grayscale, binary dither, or one of the processed
branches in depth mode.

For that reason, the preview must always be interpreted together with the
selected processing mode.

------------------------------------------------------------------------

# 5. Loading an Image

Use the **Load image** button to load the picture.

Supported formats:

- PNG
- JPG / JPEG
- BMP

After loading, the program stores the source image and uses it as the
starting point for all further processing.

------------------------------------------------------------------------

# 6. RAW Image, Processed Image and BASE

The program uses several image levels.

**RAW image**

This is the loaded source image. Crop and real rotation belong to this
image.

**Processed image**

This is the raster already aligned to the chosen physical size, DPI and
machine geometry, with image transforms already applied.

**BASE**

BASE is not always binary.

- In **Grayscale** mode, BASE is a grayscale raster
- In **Hybrid** mode, BASE is a patterned grayscale tone image
- In binary dither modes, BASE is a black-and-white dot image

G-code is always generated from the active processed raster.

------------------------------------------------------------------------

# 7. Size and DPI

The engraving size is given in millimeters.

DPI defines line or point density:

    line spacing (mm) = 25.4 / DPI

The requested DPI is not always identical to the DPI physically used by
the machine. LaserBase builds a real raster aligned to the machine step
system, so the processed image size and effective DPI can differ from the
requested values.

------------------------------------------------------------------------

# 8. Machine Profile

The machine profile contains the machine's physical data:

- steps/mm per axis
- max rate
- acceleration
- laser module
- G-code profile data

These values are needed not only for export. The program also uses them
for raster alignment and automatic overscan calculation.

In fiber mode the workspace uses a virtual machine profile; in diode mode
the machine data is part of the actual processing.

------------------------------------------------------------------------

# 9. Crop

Crop is defined in RAW image space. This matters.

It means the cutout is not applied to the already processed image, but to
the source image before further processing.

Available shapes:

- square / rectangle
- circle

For circle crop, width and height must match. If crop is active but
invalid, the Process button does not run.

------------------------------------------------------------------------

# 10. Real Rotation and Preview Rotate Back

The two rotations are not the same.

**Rotate 90**

This is a real geometric transform. It changes the orientation of the
source image, and crop, geometry and processing follow that orientation.

**Preview rotate back**

This is only a display layer. It does not modify processing or G-code. It
only rotates the left and right preview views back.

If the image must be rotated because of the machine, **Rotate 90** is the
relevant control. If the goal is only to look at the image more
comfortably, preview rotate back is the right tool.

------------------------------------------------------------------------

# 11. Processing Modes

The processing mode is the central decision in the workspace.

It does not only define which algorithm runs. It also defines what kind of
BASE is built, what the preview means, and how G-code is generated.

LaserBase is not limited to binary dithering.

**Grayscale**

The processed raster remains grayscale. G-code assigns a PWM value to each
pixel tone.

**Hybrid**

This is a grayscale-based mode that treats tones in a more patterned way.
It is neither classic binary dither nor pure grayscale.

Hybrid is useful when pure grayscale feels too flat and binary dither too
hard. It sits between tone field and dot structure.

**Binary dither modes**

- Floyd--Steinberg
- Atkinson
- JJN
- Stucki
- Bayer
- Halftone clustered

In these modes the final raster is binary: a dot either burns or does not.

**Depth**

This is not a separate dither algorithm but a two-branch processing model.

In depth mode the program keeps a grayscale branch and a dither branch,
and the preview can switch between them. During export the G-code is built
from two passes.

Depth is useful when tone transfer and dot structure should not be carried
by one single raster. Compared to plain grayscale, it gives two separate
processed branches with different roles.

------------------------------------------------------------------------

# 12. Image Controls

**Contrast, Brightness, Gamma, Radius, Amount**

These controls shape the input of the processing stage.

**Negative**

Tone inversion.

**Mirror X / Mirror Y**

Geometric mirroring on the image.

**Threshold**

This is not a general image slider. It affects the threshold of binary
error-diffusion modes.

**Serpentine scan**

In error-diffusion dither modes, it reverses the processing direction of
every other row. It is not a separate dither mode, but an additional
switch.

**1 pixel off**

In binary modes, this is a cleanup step that can remove isolated single
pixels.

------------------------------------------------------------------------

# 13. Auto Recommendation

**Auto** is not a simple ratio calculator.

The recommendation is built from the database using:

- technique
- material
- laser module

It tries to provide Speed / Max power / Min power values from those.

If there is no exact match, the program uses fallback logic. It does not
climb every higher level; it can only fall back to supported safe-stop
groups. When that happens, the program indicates it and may ask for
confirmation.

In Auto mode, Speed and Max power stay linked: if one of them is edited by
hand, the other is adjusted around the current recommendation baseline.

Auto is strongest when it is based on an exact material-and-technique
match. In fallback cases it should be treated more as a safe starting
point than as a final value.

The recommender relies on the records stored in the database.
Those records are not limited to predefined data: the user can add custom entries.

Stored data can be exported and loaded back in another environment.
Imported key files update the centrally prepared database set.

------------------------------------------------------------------------

# 14. Overscan

Overscan is the lead-in and lead-out travel outside the image boundary.

In automatic mode, LaserBase computes it from speed and axis
acceleration. A manual override can also be entered.

If overscan is too small, the image edges can distort because the machine
is still not moving at constant speed there.

------------------------------------------------------------------------

# 15. Processing and the Decision Step

**Process** does not simply run a filter.

Before processing, the program evaluates:

- the requested size
- the requested DPI
- the machine stepping geometry
- the actual resolution of the source image

The result is:

- **BASE** -- processing can be executed directly
- **REPAIR** -- image conditioning / resampling is required for the target raster

The actual processed raster is built after that decision.

------------------------------------------------------------------------

# 16. Preview

The right-hand preview shows the selected processed branch.

That preview is not a direct 1:1 image of the final engraving in every
mode. This is especially true in binary dither and depth mode, where it
shows raster logic more than material response.

In depth mode, you can switch between two previews:

- Grayscale
- Dither

**Nearest preview** changes how the image is displayed, not how it is
processed.

In fullscreen mode it is worth checking:

- tone transitions
- streaking
- over-sharpening
- binary dot density
- the difference between the two branches in depth mode

------------------------------------------------------------------------

# 17. G-code Generation

G-code strategy depends on the mode.

**Grayscale / Hybrid**

The program derives PWM values from pixel tone, so laser power can change
within a line.

**Binary dither**

Black dots burn with the configured maximum power, white dots do not burn.

**Depth**

Two successive passes are produced: the dither branch and the grayscale
branch each get their own G-code block.

------------------------------------------------------------------------

# 18. Save Image, G-code and Frame

**Save image**

Saves the currently active processed image. This can be the grayscale,
hybrid or dither branch.

**G-code**

Saves the engraving control file.

**Frame**

If enabled, the program also creates a separate frame file. Its
parameters are entered in a separate dialog. With circle crop, the frame
is circular; otherwise it is rectangular.

------------------------------------------------------------------------

# 19. Saving to the Database

The main window works as a parameter database, but workspace saving
contains more than that.

When saving, the program stores:

- the source image reference
- machine and geometry state
- crop state
- base and G-code controls
- auto state
- both branch settings in depth mode

For that reason, reload is not only record filling but workspace state
restoration.

------------------------------------------------------------------------

# 20. Reloading

A saved workspace record can be reloaded from the main window.

When that happens, the program:

1.  reloads the source image
2.  restores size, DPI and machine data
3.  restores crop and processing controls
4.  runs processing again

The restored state is therefore session-like rather than a simple list of
parameters.

------------------------------------------------------------------------

# 21. Sender

Sender runs in a separate window.

Basic functions:

1.  select a port
2.  connect
3.  load G-code
4.  start sending

During operation:

- Pause / Resume
- STOP
- E-STOP
- $X Unlock in alarm state
- manual jog movement
- marker laser
- terminal commands

Sender also supports a separate frame run, machine-start and work-start
position storage, and offset-based positioning.

------------------------------------------------------------------------

# 22. Main Window, New Entry, Calculator, Sketch

**Main window**

Stores and manages records.

**New Entry**

Creates a new record.

**Calculator**

Helps compute proportionate new parameters from existing records.

**Sketch**

A simple separate drawing and sketching surface.

------------------------------------------------------------------------

# 23. Common Problems

- **No image** -- load a source image first
- **No machine profile in diode mode** -- enter one or load one
- **Invalid crop** -- fix it or turn it off
- **Circle crop with unequal size** -- width and height must match
- **No auto recommendation** -- there is no suitable data for this material / technique / module combination
- **G-code cannot be saved** -- processing must complete successfully first

------------------------------------------------------------------------

# 24. Useful Practice

- On new materials, check both grayscale and binary dither preview first.
- If the image must be rotated because of the machine, use real Rotate 90.
- If the goal is only a different viewing orientation, use preview rotate back.
- In Auto mode, always check whether the recommendation is exact or fallback-based.
- Use depth mode when the two raster roles should be handled separately.
- Use Frame export before positioning the real job.
- Treat workspace saving as work-state saving, not only as record saving.

------------------------------------------------------------------------

# 25. Quick Reference

    line spacing (mm) = 25.4 / DPI
    effective DPI = 25.4 / actual pitch_mm

Modes:

- Grayscale -> tone-based PWM raster
- Hybrid -> grayscale patterned raster
- Dither -> binary dot raster
- Depth -> two-branch processing and two-pass export

Rotation:

- Rotate 90 -> real geometric change
- Preview rotate back -> display only

Saving:

- Save image -> active processed branch
- Save db -> workspace state + record
- G-code -> control file, optionally with frame file
