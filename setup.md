## PHD2 Configuration Notes and Required Dependencies

This driver requires the **ASCOM Platform** to be installed before use. 

The installer registers the SV205 Synthetic Mono Camera driver, but it does not replace the ASCOM runtime, ASCOM Chooser, or ASCOM Profile infrastructure used by PHD2. 

Install the ASCOM Platform first from the official ASCOM downloads page: https://ascom-standards.org/Downloads/Index.htm. 

PHD2 should be installed from the official PHD2 site: https://openphdguiding.org/downloads/. 

### Recommended PHD2 configuration

For the first validation, keep PHD2 as simple as possible. Use **ASCOM Camera → SV205 Synthetic Mono Camera**, set the driver mode to **Guide**, and start with a fixed **2.0 second exposure**. 

Disable **Auto Exposure** during testing so PHD2 does not change the exposure while the driver is being evaluated. Disable **Subframes** in the advanced menu.  

Disable PHD2 **Noise Reduction** at first, because it can mask whether the driver itself is delivering a clean luma frame. After stable guiding is confirmed, these options can be re-tested one at a time, **except for subframes**!!

Recommended initial values:

```text
Camera: ASCOM Camera
Driver: SV205 Synthetic Mono Camera
PHD2 Exposure: 2.0s fixed
Auto Exposure: Off
Subframes: Off Always!
Noise Reduction: Off initially
Pixel Size: read from driver Pixel Size Metadata, default 1.4 µm
Guide Scope Focal Length: configure in PHD2
```
---

## Driver setup dialog

Open the driver setup dialog from the ASCOM chooser/properties screen.

Recommended initial settings:

```text
Camera Device: SVBONY SV205
Require SV205 Device Match: On
Operating Mode: Guide
Resolution: 1280x720
Native Exposure Level: -1 or 0
Gain: 45 to 64
Stack Mode: Average
Hot Pixel Rejection: On
Dark Frame Subtraction: Off
Pixel Size Metadata: 1.4 µm
Save Debug Frames: Off
```

---

## Pixel size and focal length

PHD2 may disable the pixel-size field when using an ASCOM camera because it reads pixel size from the driver.

The driver provides:

```text
Pixel Size Metadata (µm)
```

Default:

```text
1.4 µm
```

If you need a different pixel size, set it in the ASCOM driver setup dialog.

Set the guide scope focal length in PHD2.

Example:

```text
Pixel Size Metadata: 1.4 µm
Guide scope focal length: configure in PHD2
```

---

## Recommended PHD2 exposure values

The driver supports PHD2 exposures from approximately:

```text
0.5s to 5s
```

Recommended use:

```text
0.5s  → focus, framing, bright objects
1s    → bright guide stars and faster guiding
2s    → recommended starting point for guiding
3s    → weak stars or unstable seeing
5s    → acquisition / faint stars, not first-choice guiding
```

For most guiding tests, start with:

```text
2s
```

If the star is bright and stable, try:

```text
1s
```

If stars are weak, try:

```text
3s
```

---

## Understanding native exposure level

The SV205 behaves approximately like this:

```text
Native Exposure Level  0  ≈ 1 second per frame
Native Exposure Level -1  ≈ 0.5 second per frame
Native Exposure Level -2  ≈ 0.25 second per frame
Native Exposure Level -3  ≈ 0.125 second per frame
Native Exposure Level -6  ≈ fast / video-like
```

The driver uses **time-based exposure capture**.

If PHD2 asks for:

```text
StartExposure(3.0)
```

the driver captures frames for about 3 seconds, then stacks all frames captured in that time.

This means:

```text
Native Exposure Level 0  → about 3 frames in 3 seconds
Native Exposure Level -6 → many frames in 3 seconds
```
**PHD2 receives one completed image either way.** 

---

## Luma / grayscale processing

The driver uses luma/grayscale output.

The SV205 is a color camera physically, but for guiding, PHD2 mainly needs a stable brightness signal for star centroid detection.

Why luma is preferred:

```text
PHD2 guides from brightness, not color.
Luma reduces unnecessary chroma information.
Luma is simpler and more stable for star centroid detection.
It avoids color-channel decisions for guiding.
```

---

## Stacking methods

The driver can combine multiple frames captured during the requested PHD2 exposure.

### Average

Recommended default for guiding.

```text
Smooths random noise
Preserves brightness scale
Lower risk of saturation
Good centroid stability
```

Use for:

```text
Guide mode
Focus mode
Planetary mode
Normal PHD2 operation
```

### Sum

Adds frames together.

```text
Makes faint signal brighter
Can help with acquisition
Can increase noise
Can saturate stars
Can make hot pixels more visible
```

Use for:

```text
Deep Space / Acquisition
finding faint guide stars
testing sensitivity
```

Avoid for saturated stars.

### Median

Uses the median pixel value across frames.

```text
Rejects outliers
Helps with hot pixels
Can reduce faint signal
More computationally expensive
```

Use for:

```text
noise rejection
hot-pixel-heavy situations
```

### Max

Uses the brightest pixel value across frames.

```text
Highlights bright events
Can help detection experiments
Strongly emphasizes hot pixels
Not recommended for stable guiding
```

Use for:

```text
experiments only
acquisition tests
```

### Sigma-clipped average

Rejects statistical outliers before averaging.

```text
Cleaner than simple average
Useful when enough frames are captured
Good advanced guiding/acquisition option
```

Use when:

```text
many frames are captured
hot pixels or transient noise affect the stack
```

### Weighted average

Gives more influence to frames estimated to be cleaner or better.

```text
Experimental
Useful if frame quality varies
May fall back to Average if quality cannot be estimated reliably
```

### Dark-subtracted average

Subtracts a compatible dark frame before averaging.

```text
Reduces fixed-pattern noise
Can reduce thermal artifacts
Requires matching dark frame
```

Use only when a compatible dark frame exists.

---

## Calibration options

### Hot Pixel Rejection

Recommended default for guiding:

```text
On
```

Purpose:

```text
prevents isolated hot pixels from being mistaken for guide stars
reduces false lock risk
helps PHD2 avoid bad stars
```

### Dark Frame Subtraction

Recommended default:

```text
Off initially
```

Turn on only when you have a compatible dark frame.

Purpose:

```text
subtracts fixed-pattern noise
reduces thermal artifacts
improves cleaner image calibration
```

If dark subtraction is enabled but no compatible dark frame exists, the driver should continue without crashing and log a warning.

---

## Recommended profile settings

### Guide

Best default for normal PHD2 guiding.

```text
Native Exposure Level: -1 or 0
Gain: 45 to 64
Stack Mode: Average
Hot Pixel Rejection: On
Dark Frame Subtraction: Off
PHD2 Exposure: 2s
Resolution: 1280x720
```

If stars are saturated:

```text
lower gain
use Native Exposure Level -1 or -2
```

If stars are weak:

```text
increase gain
try PHD2 Exposure 3s
```

### Focus / Framing

For fast updates.

```text
Native Exposure Level: -3 or -4
Gain: about 45 to 64
Stack Mode: Average
Hot Pixel Rejection: Off or On
Dark Frame Subtraction: Off
PHD2 Exposure: 0.5s
Resolution: 1280x720 or 640x480
```

Use this to focus and frame, not as the default guiding mode.

### Planetary

For bright objects.

```text
Native Exposure Level: -4 to -6
Gain: 30 to 64
Stack Mode: Average
Hot Pixel Rejection: Off or On
Dark Frame Subtraction: Off
PHD2 Exposure: 0.5s
Resolution: 640x480 or 1280x720
```

Avoid Sum because bright targets saturate easily.

### Deep Space / Acquisition

For finding faint stars.

```text
Native Exposure Level: 0
Gain: 80 to 100
Stack Mode: Sum or Sigma-clipped average
Hot Pixel Rejection: On
Dark Frame Subtraction: Off unless dark frame exists
PHD2 Exposure: 3s
Resolution: 1280x720
```

Use 5 seconds only for acquisition or faint star search, not as the first guiding default.

### Custom

For advanced testing.

```text
User controls all fields.
```

---

## How to interpret PHD2 star quality

PHD2 log values to watch:

```text
Mass
SNR
Peak
HFD
```

Good general range:

```text
SNR > 10
Peak below saturation
HFD around 2 to 8
Mass stable
```

### Saturated star

If the log shows:

```text
Peak=255
Saturated!!
```

the star is too bright for accurate centroiding.

Fix:

```text
reduce gain
reduce native exposure
avoid Sum stacking
use shorter PHD2 exposure
```

### Low mass / low SNR

If the log shows:

```text
Lost Star - Low mass
Lost Star  - low SNR
Mass=0
SNR=0
HFD=0
```

the star is not being detected reliably.

Fix:

```text
increase gain
use Native Exposure Level 0
try PHD2 exposure 2s or 3s
use Deep Space / Acquisition mode
ensure focus is correct
disable PHD2 noise reduction during testing
```
---

## Troubleshooting

### PHD2 shows a striped or corrupted image

Check the debug frames:

```text
%LOCALAPPDATA%\SV205SyntheticMono\logs\last_internal_frame.png
%LOCALAPPDATA%\SV205SyntheticMono\logs\last_ascom_frame.png
```

If `last_internal_frame.png` is clean but PHD2 is corrupted, the problem is likely ASCOM ImageArray orientation or dimensions (check if you have not inputed in reverse in config!)

If both are corrupted, the problem is likely capture or grayscale conversion.

### PHD2 detects star, then loses it

If the star is saturated:

```text
reduce gain or native exposure
```

If subframes are enabled and unstable:

```text
disable subframes
```

If noise reduction is enabled:

```text
disable noise reduction for validation
```

### Pixel size cannot be edited in PHD2

This is normal for many ASCOM cameras.

Set:

```text
Pixel Size Metadata (µm)
```

inside the driver setup dialog.

### Camera opens the wrong device

Open driver setup and confirm:

```text
Camera Device: SVBONY SV205
Require SV205 Device Match: On
```

If the SV205 is not listed:

```text
close other camera apps
unplug/replug SV205
refresh camera list
```

---

## Monitoring PHD2 logs

PHD2 logs can be used to calibrate driver profiles.

Important patterns:

```text
Star::Find returns
Mass
SNR
Peak
HFD
Saturado
Estrela perdida
UpdateImageDisplay
ScheduleExposure
Exposure complete
```
---

## Restoring or checking the SV205 native camera settings

If the SV205 behaves unexpectedly after using the ASCOM driver, open the native Windows / DirectShow camera properties dialog with FFmpeg and confirm the camera settings manually.

Open PowerShell and run:

```powershell
ffmpeg -f dshow -show_video_device_dialog true -i video="SVBONY SV205"
```
In the camera properties window, select on both tabs "Reset" and all set!

## Final notes

Use the driver conservatively at first. Avoid saturated guide stars. Start with Average stacking. Enable more advanced stacking only after verifying the camera is stable in PHD2.

The most important signs of a good setup are:

```text
stable star lock
SNR above 10
Peak below saturation
HFD above 0
few or no lost-star events
consistent exposure cadence
```
