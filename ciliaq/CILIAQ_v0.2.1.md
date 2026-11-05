# CiliaQ v0.2.1

**Kit version:** 0.0.1 · **Last updated:** November 2026
**Verified against:** git tag `v0.2.1` of https://github.com/hansenjn/CiliaQ (`CiliaQMain.java`)
**Sources:** source code, release notes v0.0.5–v0.2.1, CiliaQ wiki (Step 3 tutorial, Output
Parameters, Q&As), CiliaQ SOP, CiliaQ paper

Read `00_START_HERE.md` first. Its rules apply here too.

---

## 0. Instructions to the assistant (CiliaQ-specific)

1. **Static 2D and 3D images only.** v0.2.1 is not for time-lapse images; for those, CiliaQ v0.1.7
   should be used, which this file does not cover. `[developer]`
2. **Expect a beta warning.** When started from the menu, v0.2.1 shows a dialog calling itself a
   beta ("Version v0.2.0") and recommending v0.1.7; press OK, since Cancel stops CiliaQ. Macro runs
   skip it. For static images, the developer considers v0.2.1 suitable. `[code]` `[developer]`
3. **Use only the settings in §2, spelled exactly.** In a macro, a missing flag means "off", even
   where the dialog's default is "on" (§2.4).
4. **Ask for the `_CQ.txt` header** (settings, version, counts) **and a few result rows** when
   judging a run. They contain no image data.
5. **Keep skeleton settings and versions constant across a dataset.** Length and other skeleton
   results depend on the blur before skeletonization and on the CiliaQ version (§5.4).

---

## 1. What CiliaQ does

`[code]` `[wiki]` `[paper]`

1. Reads the **reconstruction channel** — the segmented channel of `_CQP.tif` or `_CQP_ed.tif`.
   Every voxel above 0 counts as cilium.
2. Connects adjacent cilium voxels into **objects** (flood fill). With "Increase range", diagonal
   neighbours are connected too.
3. Removes objects smaller than the **minimum cilium size** (in voxels) as noise.
4. Marks objects touching the image borders as **excluded**, according to the exclusion setting.
5. Measures each object: position, volume, surface, shape, and intensities in up to two further
   channels (A and B).
6. Optionally computes a **skeleton** for each object (after upscaling and a Gaussian blur) and from
   it the length, orientation, bending, intensity profiles and curvature.
7. Optionally uses a **basal-body channel** to orient cilia, or to detect basal bodies and match
   them to cilia (3D static images only).

---

## 2. Settings reference

Command: `Plugins ▸ CiliaQ ▸ CiliaQ v0.2.1`; in a macro `run("CiliaQ v0.2.1", "…")`. `[code]`

### 2.1 First dialog (general)

| dialog field | macro key | default |
|---|---|---|
| `process` | — (a macro always processes the front-most image) | multi-task manager |
| `manually calibrate image` | `re-calibrate` (flag) | off |
| `-> calibration [unit/px]:` | `length=` | 0.21 |
| `-> voxel depth [unit/voxel]:` | `depth=` | 1.0 |
| `-> calibration unit:` | `calibration-unit=` | µm |
| `Time interval: [time unit]:` | `time-interval=` | 0.5 |
| `Time unit:` | `time-unit=` (`sec`, `min`, `hr`, `d`) | min |
| `Preferences:` | — | `manually enter preferences`; or `load preferences from existing CiliaQ results file ('...CQ.txt')` |
| `save date in output file names` | `include-date` (flag) | off |
| `save result image for each individual cilium` | `save-single-cilia-result-images` (flag) | off |
| `save 3D visualizations for each individual cilium` | `save-3D-individual-cilia` (flag) | off |
| `save 3D visualizations for whole image` | `save-3D-whole-image` (flag) | **on** |
| `output number format` | `number-format=` `[US (0.00...)]` or `[Germany (0,00...)]` | US |

Calibration keys are read only together with `re-calibrate`. `[code]`

### 2.2 Second dialog (detection)

| dialog field | macro key | default |
|---|---|---|
| `measure intensity A:` | `measure-intensity-A` (flag) + `channel-A=` | on, channel 2 |
| `measure intensity B:` | `measure-intensity-B` (flag) + `channel-B=` | on, channel 3 |
| `channel with basal (body) stain ...` | see below | not available |
| `channels (Reconstruction / (opt. intensity A) / (opt. intensity B) / (basal stain)):` | `reconstruction-channel=`, `channel-A=`, `channel-B=`, `channel-basal-stain=` | 1 / 2 / 3 / 4 |
| `minimum cilium size [voxel]:` | `minimum-cilium-size=` | 10 |
| `Increase range for connecting cilia` | `increase-range-cilia` (flag) | **on** |
| `additionally exclude...` | `exclude=[…]` (§2.3) | x or y or z borders |
| `minimum size of intensity regions (for A and B) [voxel]:` | `minimum-size-intensity=` | 1 |
| `Increase range for connecting intensity regions` | `increase-range-intensity` (flag) | **on** |
| `Determine skeleton-based results (e.g. length)` | `skeleton` (flag) | **on** |
| `before skeletonization: gauss filter XY and Z sigma:` | `gauss-xy=`, `gauss-z=` | 2.0, 0.0 |
| `Reference distance for tangents and curvature [calibration unit (e.g. µm)]:` | `distance-curvature=` | 1.0 |

**All channel numbers refer to `_CQP.tif` / `_CQP_ed.tif`, not to the raw file.** `_CQP.txt`
lists the mapping. `[wiki]`

**Basal stain choice** (macro):

| dialog choice | macro |
|---|---|
| `not available - ignore channel ID specified below` | neither flag |
| `available - use channel to orient cilia` | `basal-stain-present channel-basal-stain=<n>` |
| `segmented - detect and quantify BBs from channel (set-up detection in next dialog)` | `basal-stain-segmented channel-basal-stain=<n>` + the keys in §2.5 |

### 2.3 Exclusion options

`exclude=` takes exactly one of:

```
[nothing]
[cilia touching x or y borders]
[cilia touching x or y or z borders]
[cilia touching x or y borders or whose thickest part is located at the z border]
```

The last option excludes objects touching the x/y borders, plus objects whose largest number of
voxels lies in the first or last slice. For **2D images**, v0.2.1 automatically switches the two
z-border options to `cilia touching x or y borders` and reports it; older versions excluded every
cilium in 2D. `[code]`

### 2.4 How macro options are read

`[code]`

- **Flags are off unless written** — including `increase-range-cilia`, `increase-range-intensity`,
  `skeleton` and `save-3D-whole-image`, which are *on* by default in the dialog.
- **Missing required values stop the run with an error message** that names the first missing key:
  `reconstruction-channel`, `minimum-cilium-size`, `exclude`, `minimum-size-intensity`,
  `distance-curvature`, `gauss-xy` and `gauss-z` (with `skeleton`), `channel-A`/`-B` (with the
  measure flags). No result files are written. `[measured]`
- `exclude`, `assign` and `remove` must match the listed values exactly.
- Use a decimal point in numbers. End the option string with a space.
- `showgui` shows the progress window during a macro run.
- **Recording** (`Plugins ▸ Macros ▸ Record…`) while using the dialogs produces a replayable
  `run("CiliaQ v0.2.1", "…")` line — the easiest way to hand a working setting to the assistant.

### 2.5 Basal-body dialog (only with "segmented")

| dialog field | macro key | default |
|---|---|---|
| `minimum basal body object size [voxel]:` | `minimum-basal-body-size=` | 1 |
| `Increase range for connecting pixels to form a basal body object` | `increase-range-basal-body` (flag) | on |
| `Assign` | `assign=[each BB to closest cilium]` (only option) | — |
| `Max allowed BB-center-to-cilium-skeleton-end distance (calibrated units):` | `max-bb-to-cilium-distance=` | 2.0 |
| `Remove` | `remove=[…]`: `[nothing based on BB-cilia-linking]`, `[BBs without cilia]`, `[cilia without BBs]`, `[cilia without BBs and BBs without cilia]` | **cilia without BBs and BBs without cilia** |

Requires a segmented basal-body channel from the Preparator. Not available for time-lapse images:
the analysis is cancelled. `[code]` `[wiki]`

---

## 3. Choosing the settings

### 3.1 Calibration

Lengths, volumes and distances are in the image's calibration. Check `Image ▸ Properties…` on the
input: if the unit is "pixel", either fix the calibration or tick `manually calibrate image` and
enter pixel size and z step. A wrong voxel depth silently distorts every 3D measurement. `[wiki]`

### 3.2 Channels

- **Reconstruction:** the segmented cilia channel.
- **Intensity A / B:** the channels to quantify in the cilium — typically the unsegmented copy of
  the cilia channel and/or a protein of interest. Absolute intensity parameters need *unsegmented*
  channels. If a *segmented* channel is given as A or B, use only the "colocalized volume"
  parameters from it. `[wiki]`
- **Basal stain:** "available" uses the stain to decide which skeleton end is the base (within
  1 µm of the skeleton ends since v0.2.0); without it, base and tip are assigned arbitrarily, so
  orientation vectors and profile directions are not meaningful. "Segmented" detects basal bodies
  (§3.6). `[release notes]` `[wiki]`

### 3.3 Object detection

- **Minimum cilium size** removes small objects as noise. It counts voxels, so the same number
  means different physical sizes at different pixel sizes and z steps. Choose it below the
  smallest real cilium in the data; compare the `_CQ_RP.tif` against the original to see what was
  removed. `[wiki]`
- **Increase range** connects diagonal neighbours. It helps with low signal-to-noise and small
  gaps, but can attach noise voxels to cilia and distort length. `[wiki]`
- **Exclusion:** for 3D, `x or y or z borders` (default) keeps only cilia fully inside the stack.
  Cilia that touch the first or last slice are reported only with a less strict option; whether to
  include them depends on whether partly captured cilia would bias the measurement. In 2D, see
  §2.3. `[wiki]` `[code]`
- Leave **minimum size of intensity regions** at 1 unless channel A or B is segmented. `[wiki]`

### 3.4 Skeleton

- **Gauss filter before skeletonization:** smooths the object so the skeleton follows the
  centreline without side branches or kinks. XY sigma 1–2 is usual; Z usually 0. Larger values
  give fewer branches but slightly shorter lengths. `[wiki]` `[paper]`
- **Use the same values for the whole dataset.** `[wiki]`
- The blur can also fill hollow masks, e.g. from Canny 3D at the z borders. `[wiki]`
- It affects only skeleton-based results (length, orientation, bending, profiles, curvature), not
  volume or intensities. `[wiki]`
- **Reference distance for tangents and curvature:** leave the default unless tangents or
  curvature are analysed. `[wiki]`

### 3.5 Output options

3D visualizations for the whole image are useful for eye-ball-checking (`_CQ_RP_3D`, `_CQ_SKL_3D`);
per-cilium 3D visualizations are slow with many cilia. "Save date in output file names" prevents
overwriting results of an earlier run on the same image — useful when comparing settings.
`[wiki]`

### 3.6 Basal bodies (segmented)

Each basal body is reduced to its centre. Basal bodies and cilia are then matched one-to-one by an
optimal assignment (Hungarian algorithm on squared distances, which favours close matches) on the
distance from the basal-body centre to the nearest skeleton point of each cilium —
or to the cilium centre if it has no skeleton. Matches farther apart than the maximum distance are
rejected. `[code]` `[developer]` The skeleton is then extended to the basal-body centre, which
improves length for very short cilia; a cilium without a skeleton is connected to its basal body.
`[release notes]`

If each cell shows a single basal-body spot — or the segmentation fuses the two dots of mother and
daughter centriole into one object — the basal-body count can estimate the number of cells and
thus the ciliation rate (from the header counts, §4.2). If the two centrioles are segmented as
separate objects, the count is roughly twice the number of cells. `[developer]` If this varies
between basal bodies, the cell count is unreliable — say so. Object sizes might help tell single
from fused pairs, but CiliaQ does not output basal-body sizes, so this would need a separate
measurement on the segmented channel. `[guess]`

- The default `Remove` setting drops **both** cilia without a basal body and basal bodies without
  a cilium. That changes cilium counts; choose deliberately.
- With `nothing…` or `cilia without BBs`, basal bodies without a cilium appear as extra result
  rows with empty cilium values. **Do not count rows to count cilia** here; with segmented basal
  bodies, the header counts are correct (§4.2). `[wiki]` `[measured]`

---

## 4. Running and reading the output

### 4.1 Macro template

`[code]`

```javascript
open("/full/path/to/image_CQP.tif");
run("CiliaQ v0.2.1",
    "number-format=[US (0.00...)] " +
    "reconstruction-channel=3 " +
    "measure-intensity-A channel-A=4 " +
    "minimum-cilium-size=10 " +
    "increase-range-cilia " +
    "exclude=[cilia touching x or y or z borders] " +
    "minimum-size-intensity=1 " +
    "increase-range-intensity " +
    "skeleton gauss-xy=2.000000 gauss-z=0.000000 " +
    "distance-curvature=1.000000 " +
    "save-3D-whole-image ");
```

This exact line has been run headless against the published v0.2.1. `[measured]`

### 4.2 Files

All next to the input, named `<input name>_CQ…` (with a date if chosen). `[code]` `[wiki]`

| file | content |
|---|---|
| `_CQ.txt` | settings, counts, results table (one row per cilium), `Plug-in version` |
| `_CQs.txt` | results rows only; column header in `_CQ.txt` |
| `_CQl.txt` | per skeleton point: coordinates, arc length, intensities, tangent, curvature — intensity profiles are here since v0.2.0, no longer in `_CQ.txt` `[release notes]` |
| `_CQ_RP.tif` | reconstruction after noise filtering, labelled with cilium IDs |
| `_CQ_SKL.tif` | skeletons, labelled with IDs |
| `_CQ_SKL_info.txt` | channel legend of `_CQ_SKL.tif` (skeleton, detected base), `Plug-in version` |
| `_CQ_RP_3D`, `_CQ_SKL_3D` (`.png` for static images) | 3D overviews, if chosen |

**Header counts.** `# excluded:` and `of total #:` give excluded and total cilia; with segmented
basal bodies also `#BBs excluded:` and `of total #BB:`. With a nuclear stain as basal channel,
the BB count corresponds to cells. Fraction of ciliated cells: see the wiki Output Parameters page.
`[code]` `[wiki]`

- **Known issue in v0.2.1:** without basal-body segmentation, `of total #:` is always 0. Count the
  cilia as result rows in `_CQs.txt` (one row per included cilium); total = rows + `# excluded:`.
  With segmented basal bodies, the header counts are correct. `[code]` `[measured]`
- `# excluded:` counts cilia excluded for any reason, including the basal-body `Remove` filter,
  although the header line names only the border option. `[measured]`

**IDs** are assigned in scan order (x, then y, then z) and appear in the result images; excluded
cilia are labelled in a different colour and have no result rows. `[wiki]` `[code]`

### 4.3 Checking a run

- Compare `_CQ_RP.tif` and `_CQ_SKL.tif` (or the 3D overviews) with the original: one object per
  cilium? Skeleton along the centre of each cilium?
- **Many branches** are harmless if the main skeleton follows the cilium; otherwise smooth more
  (here, or in the Preparator). `[wiki]`
- **No length / 0 skeletons:** the object is roughly spherical — a cross-section in 2D, a stub, a
  cilium standing along z, or one cut by the stack edge. Usually include these with length 0, or
  add a constant below one pixel length to all cilia; do not drop them silently. `[wiki]`
- **Unusually long cilia** often are fused neighbours; **many tiny objects** are debris (raise the
  minimum size or fix the segmentation). See `00_START_HERE.md` §5.
- For quality control and statistics across many files, CiliaQ Explorer reads the outputs directly
  and warns when results from different CiliaQ versions are pooled. `[wiki]`

---

## 5. Interpreting results

The full parameter list with definitions is on the wiki Output Parameters page (§7). Points an
assistant must get right: `[wiki]`

### 5.1 Morphology

- **Volume and surface** depend strongly on the segmentation; compare only within one setting.
- **Length** is the longest shortest path through the skeleton; it is much less sensitive to mask
  width than volume. **Tree length** includes side branches.
- **Orientation** needs a basal stain to know which end is the base.
- **Bending index** relates length to the base-to-tip distance; it is not the mathematical
  curvature (that is in `_CQl.txt`).

### 5.2 Intensity

- **Average intensity** uses all voxels of the object, so a wide mask dilutes it.
- **Average intensity on centreline** and **average of 10 % highest voxels** are more robust to
  mask width.
- **Integrated intensity on centreline** reflects total amount and grows with length.
- **"Compared to background"** parameters use an automatic background threshold per image: the
  image is divided into 25 cuboids, the brightest 10 % of non-cilium voxels of each are collected,
  and threshold = mean + 1.5 × SD. `[wiki]` `[code]` For channels that may contain no real signal,
  segment them with a fixed threshold in the Preparator instead (see the Preparator file).
  `[developer]`

### 5.3 Basal-body parameters

Written only with a segmented basal-body channel in static images. Empty fields mean "no basal
body assigned", not zero. The 1 µm and 1–2 µm intensity shells are fixed radii in the calibration
unit. `[wiki]`

### 5.4 Combining results

- Same CiliaQ version, same settings, same Preparator setting for everything that is compared.
- Never pool skeleton-based results from v0.1.7 or later with earlier versions. `[release notes]`
- Changing the skeleton blur changes lengths slightly; never mix sigmas. `[wiki]`

---

## 6. If the run fails or results look wrong

| what happens | likely reason | fix |
|---|---|---|
| beta-version warning at start | expected in v0.2.1 when started from the menu | press OK and continue for static images (§0); Cancel stops CiliaQ |
| error "Macro options missed definition …" | a required key is missing; only the first is named | add it and run again until no error remains (§2.4) |
| macro run ignores connectivity / skeleton / 3D output | flags not written | add `increase-range-cilia`, `skeleton`, … |
| no or few cilia in the results | border exclusion, a too-high minimum size, or the wrong reconstruction channel | check `# excluded:` in the header, the exclusion option and the channel mapping in `_CQP.txt` |
| lengths in pixels or implausible | image not calibrated / wrong voxel depth | check `Image ▸ Properties…`; manual calibration |
| analysis cancelled with basal-body detection | time-lapse image | not supported; use v0.1.7 without basal-body detection |
| numbers unreadable in Excel | number format | choose the format matching your spreadsheet locale |

---

## 7. Further reading and not covered

- Step 3 tutorial (based on older version, v0.1.4 `[developer]`): https://github.com/hansenjn/CiliaQ/wiki/Step-3---CiliaQ---Detecting-and-quantifying-cilia-from-the-segmented-image
- Output parameters: https://github.com/hansenjn/CiliaQ/wiki/CiliaQ-Output-Parameters
- Not covered: time-lapse analysis, CiliaQ v0.1.7 and earlier, segmentation
  (`preparator/PREPARATOR_v0.2.0.md`), manual editing (`editor/EDITOR_v0.0.4.md`).
