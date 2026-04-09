# LaserBase -- Image Processing at Workshop Level

This document describes the LaserBase image-processing model. The goal is
to understand how it works: which representations exist, in what order
the pipeline runs, and how the program gets from the RAW image to G-code.

This is not a developer specification. The focus is a usable, technically
accurate mental model.

------------------------------------------------------------------------

## 1. The Core Picture of the System

LaserBase is not built around binary dithering alone.

The system supports several processing strategies:

- grayscale raster
- binary dither raster
- hybrid tone model
- two-branch depth processing

For that reason, the central question is not only which dither algorithm
to choose, but what image representation the chosen mode produces and what
kind of G-code follows from it.

------------------------------------------------------------------------

## 2. Representations

### 2.1 RAW

RAW is the loaded source image.

It is tied to:

- real 90° rotation
- crop
- source orientation

Crop is not defined on the already processed image, but in RAW space.

### 2.2 Processed Raster

This is the image already:

- bound to the target physical size
- aligned to machine stepping
- carrying the image transforms
- ready for export

### 2.3 BASE

BASE is mode-dependent.

It is not correct to teach it as always binary.

- In **Grayscale**, BASE is grayscale
- In **Hybrid**, BASE is grayscale with patterning
- In binary dither modes, BASE is binary

### 2.4 Depth State

In depth mode, the system keeps two processed rasters:

- one grayscale branch
- one dither branch

The preview can switch between them, and export is built from two passes.

------------------------------------------------------------------------

## 3. Geometry, DPI and Machine Alignment

### 3.1 Requested and Real Raster

The user provides:

- physical size in millimeters
- requested DPI
- scan axis
- machine data

The program does not draw a raster from this directly. It computes a real
raster aligned to what the machine can step.

If lines on the step axis can only be placed at specific step intervals,
the actual pitch and effective DPI are adjusted to that allowed geometry.

### 3.2 Effective DPI

Effective DPI is:

    effective DPI = 25.4 / real_pitch_mm

It can differ from the requested DPI.

### 3.3 Decision Step: BASE or REPAIR

Before processing, the system decides:

- whether the source image is sufficient for the required real raster
- or whether image conditioning / resampling is needed first

If the required number of lines or columns is greater than what the source
image can provide directly, the decision becomes **REPAIR**. Otherwise it
is **BASE**.

------------------------------------------------------------------------

## 4. RAW Space and Orientation

### 4.1 Rotate 90

`rotate_90` is a real geometric transform.

It is not a preview trick. It changes the orientation of the source image.
Because of that, the following steps work on the rotated RAW image:

- crop
- geometry evaluation
- resample when needed
- final raster build

### 4.2 Preview Rotate Back

Preview rotate back is a separate display layer.

It transforms only the view. It does not modify:

- kernel state
- crop definition
- BASE image
- G-code

The two rotations therefore stay separate:

- `rotate_90` = processing geometry
- preview rotate back = visual convenience

------------------------------------------------------------------------

## 5. Crop Model

### 5.1 Why RAW-space Crop?

Crop is defined on the source image so that the cutout happens as early as
possible.

This means:

- further processing already starts from the cropped area
- geometry decisions can use the cropped effective source size
- crop can be saved and restored precisely

### 5.2 Crop Shapes

- rectangle / square
- circle

With circle crop, the geometry must stay square. This is enforced at UI
level as well.

------------------------------------------------------------------------

## 6. Processing Modes

The processing mode is the central choice in the system.

It defines what representation BASE takes, what the preview means, and how
G-code turns raster data into power and motion.

### 6.1 Grayscale

In this mode the raster remains grayscale. The G-code generator maps pixel
tone to PWM values.

This is not on/off engraving, but a more continuous power profile.

### 6.2 Binary Dither

The error-diffusion and ordered modes belong here:

- Floyd--Steinberg
- Atkinson
- JJN
- Stucki
- Bayer
- Halftone clustered

Here the final result is binary. At G-code level, a pixel either receives
full burn or none.

### 6.3 Hybrid

Hybrid is not a classic dither mode.

It starts from a grayscale image and adds a tone-dependent pattern
correction. It is stronger in the midtones and weaker at the extremes.

Its role is:

- not pure grayscale
- not pure binary dither
- more textured transfer of tone fields

In practical terms, Hybrid bridges the gap between overly smooth
grayscale and overly harsh binary dither. It is useful when tone
retention matters but fully smooth grayscale does not provide enough
structure.

### 6.4 Depth

Depth keeps two separate branches:

- grayscale branch
- dither branch

Each branch can store its own control state. Preview switches between
them, and export generates two passes.

Depth is useful when tone and dot structure should not be carried by the
same raster. Compared with plain grayscale, it gives separate working
space to those two roles.

------------------------------------------------------------------------

## 7. The Actual Role of the Image Controls

### 7.1 Negative

In the pipeline, `negative` runs early.

This means the later brightness, contrast, gamma and sharpening steps
already work on the inverted tone space.

### 7.2 Brightness and Contrast

Both operate on the grayscale working image.

- brightness: linear shift
- contrast: scaling around the midpoint

### 7.3 Gamma

Gamma is a nonlinear correction in tone space. The system clamps the
entered value to an allowed range and applies it through a LUT.

### 7.4 Radius and Amount

Sharpening is unsharp-mask based.

### 7.5 Mirror X / Mirror Y

Mirror operations belong to image transforms and run after the tone
modification steps.

### 7.6 Threshold

Threshold is not a universal slider.

It affects only the threshold used by binary error-diffusion modes. In
Bayer, halftone clustered, grayscale and hybrid, its meaning is different
or not materially relevant.

### 7.7 Serpentine Scan

This is not a separate dither algorithm, but an alternation of error
propagation direction from row to row.

Its role:

- it affects only error-diffusion modes
- it does not have the same effect in Bayer or non-binary modes

### 7.8 One Pixel Off

This is a cleanup stage that runs on binary raster output.

It is not a general image filter. Its purpose is to remove isolated single
pixels that stick out from their surroundings.

------------------------------------------------------------------------

## 8. The Actual Pipeline Order

The logical order is:

1.  load RAW image
2.  optional `rotate_90`
3.  RAW-space crop
4.  geometry conditioning / resample if needed
5.  `negative`
6.  `contrast` + `brightness`
7.  `gamma`
8.  sharpening (`radius` + `amount`)
9.  `mirror_x` / `mirror_y`
10. final resize to raster size
11. mode-dependent dithering / hybrid operation
12. optional `one_pixel_off`

The main consequences are:

- crop is an early RAW-side step
- `rotate_90` is applied before crop
- `negative` is not after dithering
- mirror runs after tone controls

This order matters because control behavior follows from it. The same
setting produces different results depending on where it enters the
pipeline.

------------------------------------------------------------------------

## 9. Preview Model

### 9.1 Preview Does Not Have a Single Meaning

The right-hand preview is built from `_right_view_mode` and the active
processed branch.

That preview is not always a direct 1:1 image of the final engraving. In
binary dither and depth mode, it shows processing structure and raster
character more than final material response.

In practice this means:

- there may be no preview
- there may be a processed BASE view
- in depth mode, the active preview may be grayscale or dither

### 9.2 Nearest Preview

This changes only display interpolation. It does not modify the processed
image.

### 9.3 Fullscreen

Fullscreen uses the same active preview branch as the normal right-side
view, but at a larger and more inspectable scale.

------------------------------------------------------------------------

## 10. Auto Recommendation

### 10.1 What It Uses

The inputs are:

- material key
- technique key
- user module wattage

The recommender aggregates records from the database.

### 10.2 Not a Single Linear Formula

Auto is not only a "power-proportional speed" formula.

The logic combines two kinds of recommendation:

1.  module recommendation based on record medians
2.  recommendation based on an energy proxy

The final output can blend both.

### 10.3 Fallback

Fallback is not arbitrary hierarchical climbing.

The system:

- first looks for an exact material hit
- otherwise may fall back to specific safe-stop prefixes
- does not use a general top-level family fallback

This defines the limits of fallback behavior.

### 10.4 UI Behavior

In Auto mode, recommended Speed and Max power are tied to a baseline. If
the user edits one, the other changes around that baseline.

The recommendation is strongest when it comes from an exact hit. With
safe-stop fallback, it remains a more protected starting point, but is
less direct for the given material.

The recommender relies on the records stored in the database.
Those records are not limited to predefined data: the user can add custom entries.

Stored data can be exported and loaded back in another environment.
Imported key files update the centrally prepared database set.

------------------------------------------------------------------------

## 11. Overscan

The overscan formula is:

    overscan ≈ 1.15 × v² / (2a)

where:

- `v` is scan speed in mm/s
- `a` is the acceleration of the active axis in mm/s²

The factor 1.15 acts as a safety margin.

From export point of view there are three states:

- off
- auto
- manual

In manual mode, the entered value overrides the automatic calculation.

------------------------------------------------------------------------

## 12. G-code Strategy

### 12.1 Grayscale and Hybrid

Here the G-code generator maps pixel values to continuous power:

    power = s_min + (s_max - s_min) × tone

For that reason, `min_power` acts as the lower bound of tone mapping.

### 12.2 Binary Dither

In binary modes, black pixels receive the upper power level and white
pixels receive zero.

In this case, `min_power` is not the main control element of binary
evaluation.

### 12.3 Depth Export

In depth mode, the program:

1.  builds the first pass from the dither branch
2.  builds the second pass from the grayscale branch
3.  concatenates both G-code blocks

The second pass also inherits certain export-state values from the first,
so these are not two fully independent exports.

------------------------------------------------------------------------

## 13. Frame

Frame export is more than a simple checkbox.

When frame is active:

- the program opens a separate parameter dialog
- separate speed, power and pass count can be set
- a separate frame file is generated next to the saved G-code

With circle crop, the frame path is circular; otherwise it is rectangular.

------------------------------------------------------------------------

## 14. Saving and Reloading

### 14.1 Save Image

Saves the currently active processed image.

In depth mode this means the saved output is not an abstract "BASE", but
the branch that is active at that moment.

### 14.2 Save DB

The save action stores sidecar-based workspace state as well.

Typical content:

- image reference
- machine mode and machine profile
- geometry fields
- material / technique selection
- base control
- G-code control
- auto control
- crop state
- depth state and branch controls

### 14.3 Reload

Reload reconstructs the workspace and then runs processing again.

For that reason, save/reload is session-like, not only record-like.

------------------------------------------------------------------------

## 15. Sender in Brief, from the Workshop View

Sender is not only raw streaming.

Relevant functions are:

- port and connection handling
- line / byte / auto stream modes
- pause / resume / stop / e-stop
- alarm unlock
- jog
- marker laser
- separate frame run
- machine-start and work-start position handling
- homing-aware start-point handling

At workshop level this matters because frame handling and positioning are
multi-state processes in Sender as well.

------------------------------------------------------------------------

## 16. Correct Mental Model in Summary

In short:

- RAW and preview are not the same
- BASE is not always binary
- preview is not always a single direct image of the final engraving
- `rotate_90` is a real geometric operation
- preview rotate back is display-only
- crop lives in RAW space
- Auto recommendation is database-based and safe-stop limited
- G-code strategy depends on mode
- depth mode has two branches and two passes
- saving carries workspace state

This is a usable and correct picture of how LaserBase works.
