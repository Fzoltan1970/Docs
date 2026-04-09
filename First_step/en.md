# First Steps

The actual work starts in the **Image Workspace** window.
Processing usually starts by loading an existing image.
It is also possible to create a drawing in the **Sketch** window and send it to the image processor.
Sketch is available from the launcher EXE, you can draw in it, and the finished drawing can be sent to the image processor.
The main window is useful, but it is not the required first step.

## 1. Loading the Image

Open the Image Workspace and load the image.
Every further step starts from here.
Without an image there is no processing.

After that, set the basic machine environment.

## 2. Machine Mode and Profile

First choose the **machine mode**.
In **Diode** mode you work with your own machine profile.
In **Fiber** mode this is not the same kind of blocking step.

In Diode mode, enter or load a **machine profile**.
The profile describes how the machine behaves.
Correct processing and correct G-code are built on it.
If you skip it, processing in Diode mode will not be valid.

You do not need to know the firmware values by heart.
The program can communicate with the machine over USB and read the starting machine data.
If this succeeds, the profile basics are filled in automatically.

The **module value** is also part of the profile.
If you have more than one laser module, it is worth saving a separate profile for each one.
The module is not just a stored value: the system uses it later as well.

Save the profile as soon as it is usable.
This keeps you from entering everything again in later jobs.
The saved profile can be loaded back later.

Once machine mode and profile are ready, move on to geometry.

## 3. Geometry

Enter these together:
- **width**
- **height**
- **DPI**
- **scan axis**

Together they define the raster and the real engraving size.
If any of them is missing or wrong, the result will not be correct either.

Then prepare the image itself.

## 4. Preparing the Image

If you are not using the whole image, set a **crop**.
This is optional.

If the image really needs to be rotated, use **Rotate 90**.
This affects processing as well.

If you only want to view the image in another orientation, use preview rotation.
That is only a display layer.

Now comes the most important decision.

## 5. Processing Mode

Choose a **processing mode**.
This defines the final result and the behavior of the G-code as well.

**Grayscale** is good when you want smoother tone.
**Dither** is good when you want a binary dot image.
**Hybrid** is useful when grayscale is too smooth and dither is too harsh.
**Depth** is for cases where you want to work with two separate processed branches.

After that, check the preview.

## 6. Preview and Process

The preview is for checking.
In some modes it is not a direct 1:1 image of the final engraving.

If the settings are correct, run **Process**.
This is where the processed image is created.
That image becomes the basis for export.

After that, save the control file.

## 7. Export

Save the **G-code**.
This is not running yet.

**Overscan** is an export-side setting.
It controls how the machine runs out past the image edges.

Line shift / bidirectional compensation is a profile-based correction.
It is not a beginner step, and it is not a separate image-processing decision.

When the G-code is ready, move on to running.

## 8. Running

The saved G-code runs in the **Sender** window.
This is where the actual run happens.

Open Sender.
Select the port, then connect to the machine.
If the port does not appear, check the cabling and make sure the machine is connected.

Load the G-code you saved earlier.
Set the position.

If needed, run a separate **frame**.
This is optional, and it is useful before positioning.

Then start the job.
If needed, you can use Pause, Resume and Stop.

## Note

It is worth separating three kinds of saving.

**Profile save** is for reusing the machine.
This lets you quickly load the same machine or module again later.

**Workspace save** stores the full current state.
That includes the actual image and the processing settings.
Use this when you want to return to the same job later.

**DB save** is narrower than that.
It is more of a record- and parameter-level save.
It is not the same as preserving the full workspace state.

You can reload an earlier job from the main window.
Detailed behavior is covered in the Knowledge section.
