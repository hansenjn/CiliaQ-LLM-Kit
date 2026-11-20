# CiliaQ Preparator v0.2.0

**Kit version:** 0.0.2 · **Last updated:** November 2026
**Verified against:** git tag `v0.2.0` of https://github.com/hansenjn/CiliaQ_Preparator
(`CiliaQPrepMain.java`, `canny3d_thresholder/`)
**Sources:** source code, release notes v0.0.2–v0.2.0, CiliaQ wiki tutorials and Q&As (including the method pages
on hysteresis and Canny 3D as of November 2026, kept in §3.4, §3.5 and §9 because the wiki will move on to newer
versions), test runs (November 2026), CiliaQ SOP, CiliaQ paper

Read `00_START_HERE.md` first. Its rules apply here too.

---

## 0. Instructions to the assistant (Preparator-specific)

1. **Use only the settings in §3, spelled exactly as written there.** In a macro, a misspelt value
   does not fail — it silently falls back to a default (§3.6).
2. **Check the version.** This file describes v0.2.0. For **v0.1.2**, the segmentation guidance
   applies unchanged, because v0.2.0 produces byte-identical output with the same settings and
   ImageJ version, with the exceptions in `VERSIONS.md` §2 rule 5 `[release notes]`; but v0.1.2
   has no pre-blur, no macro options and no headless mode, so give settings as a dialog table,
   not a macro line. For versions older than v0.1.2, say that this file does not cover them.
3. **Follow the workflow in §4.** Look at the image first, test preprocessing and thresholds live
   in Fiji, and only then run the plugin to compare a few finished candidates.
4. **Give settings in a form the user can apply directly:** a table of dialog field → value, or a
   macro line (§5), or both. In a macro, write every key explicitly, and flags as bare words.
5. **After every run, ask for the `_CQP.txt` file and any messages in the Log window** (§6). The
   `_CQP.txt` records what actually ran and the threshold that was computed; the Log window shows
   warnings and silently replaced values.
6. **Judge by comparing runs on the same image, never by one absolute number** (§6.3).
7. **Do not use `Custom Value` with Canny 3D in v0.2.0** (§3.5, §10).

---

## 1. What the Preparator does

The Preparator decides, for the channel you choose, which voxels belong to cilia, and writes a new
image in which all other voxels of that channel are 0. CiliaQ then builds cilium objects from that
image.

- **The mask is decided on a preprocessed copy; the output keeps the original intensities.**
  Blurring and background removal change *which* voxels are kept, not the intensity values written
  to the output (with the default "keep intensities" style). `[code]` `[release notes]`
- **But preprocessing changes the mask,** and with it object shape, width, volume and connectivity.
- **In "keep intensities" output, a voxel whose original value is exactly 0 stays 0** even if the
  mask includes it, and CiliaQ treats it as background. This matters for images with a true zero
  background (noise-free detectors such as Leica HyD, an offset, or rescaled exports): a blur that
  bridges a zero-valued gap bridges it in the mask but not in the output. Use the **binary** style
  for such images. `[code]` `[wiki]`
- **Keep intensities vs. binary.** "Keep intensities" lets CiliaQ measure intensities in the
  segmented (reconstruction) channel and is the default recommendation. "Binary" makes the mask
  easier to inspect and is the right choice for zero-background images; intensity values *of the
  segmented channel* are then meaningless, while all other channels are unaffected. `[code]` `[wiki]`
- **Output layout.** The segmented channel replaces the original channel at the same position. If
  "Include also an unsegmented copy of the channel" is ticked, the raw copy is inserted directly
  after it, and all later channels move up by one. `_CQP.txt` lists the mapping. Slice labels of
  segmented images start with `segm`. `[code]` `[wiki]`
- **Files** are written next to the input image: `<name>_CQP.tif` and `<name>_CQP.txt` (or with a
  date suffix, §3.1). The input image must be saved on disk. `[wiki]` `[release notes]`
- Microscopy files containing several series are processed series by series using BioFormats
  (since v0.0.6) `[release notes]` but it is worth considering that BioFormats importing can have its own bugs
  and it is not maintained with CiliaQ. `[developer]` Virtual stacks are copied into memory automatically
  (since v0.1.2). `[release notes]`

---

## 2. The processing order is fixed

Each step is optional; the order is not. The dialog numbers the steps in this order. `[code]`

```
copy of the chosen channel
  → 1. Gaussian blur ("pre-blur")          2D, per slice; radius = sigma in pixels
  → 2. Subtract Background                 ImageJ rolling ball, per slice; radius in pixels
  → 3. Divide by Background                divides by a Gaussian-blurred copy (radius = sigma
                                           in pixels); result is a 32-bit ratio image
  → 4. Gaussian blur ("additional blur")   2D, per slice; radius = sigma in pixels
  → 5. Segmentation method                 → mask → applied to the ORIGINAL channel
```

- **Pre-blur (1) acts before background removal, the additional blur (4) after it.** Use 1 when
  noise disturbs the background estimate; use 4 to smooth the image the threshold sees.
  `[code]` `[release notes]`
- **All blurs are 2D.** There is no smoothing along z. `[code]`
- **Subtract Background** and **Divide by Background** both use a "radius", but they mean different
  things: a rolling-ball radius versus the sigma of a Gaussian. Do not transfer values between them.
  `[code]`

---

## 3. Settings reference

Values in `[brackets]` are literal strings: copy them exactly, including spaces and punctuation.

### 3.1 First dialog (general)

| dialog field | macro key | values | default |
|---|---|---|---|
| `process` | `process` | `[active image in FIJI]`, `[multiple images (open multi-task manager)]`, `[all images open in FIJI]` | multi-task manager |
| `Preferences:` | (route, see §3.6) | `[manually enter preferences]`, `[load preferences from existing CiliaQ Preparator metadata file]` | manually |
| `Output image name:` | `output` | `[save as filename + suffix 'CQP']`, `[save as filename + suffix 'CQP' + date]` | without date |
| `output number format` | `number-format` | `[US (0.00...)]`, `[Germany (0,00...)]` | US |
| `Keep computer awake during processing` | `keep-awake` (flag) | present / absent | off |

**In a macro, the plugin always processes the front-most image,** whatever `process` says.
`[code]` To process several files by macro, loop over them (§5.3).

Preferences can be loaded from `_CQP.txt` files written by v0.0.6 or later. `[code]`

### 3.2 Second dialog (per segmented channel)

In a macro, keys carry the prefix `c1-`, `c2-`, … for the first, second, … channel to segment. **`<n>` counts the
segmented channels; it is not the channel number in the image**: segmenting image channels 2 and 4 means
`channels=2 c1-nr=2 … c2-nr=4 …`. The settings dialog shows both numbers in its title ("Insert processing settings
for channel 1 with channel nr 2"). `[wiki]`

| dialog field | macro key | value | default |
|---|---|---|---|
| `Channel Nr (>= 1 & <= nr of channels)` | `c1-nr` | integer, channel number **in the input image** | 1 |
| `Include also an unsegmented copy of the channel` | `c1-duplicate` (flag) | present / absent | **ticked in the dialog, off in a macro unless the flag is given** |
| `1. Smooth with Gaussian blur - radius` | `c1-preblur` | number, e.g. `2.000` | off; field pre-filled 2.0 |
| `2. Subtract Background before segmentation - radius` | `c1-subtract-bg` | number, e.g. `10.000` | off; field pre-filled 10.0 |
| `3. Divide by Background before segmentation - radius` | `c1-divide-bg` | number, e.g. `3.000` | off; field pre-filled 3.0 |
| `4. Smooth with Gaussian blur - radius` | `c1-blur` | number, e.g. `0.500` | off; field pre-filled 0.5 |
| `5. Segmentation method` | `c1-method` | see §3.3 | `RenyiEntropy` |
| `If 'CUSTOM threshold' was selected, specify threshold here` | `c1-custom-thr` | number | 0 |
| `Stack handling (obsolete for segmentation method 'CANNY 3D'):` | `c1-stack` | see §3.3 | MIP |
| `Threshold every time step independently (obsolete for segmentation method 'CANNY 3D')` | `c1-separate-timesteps` (flag) | present / absent | off |
| `Segment more channels:` | `channels` | integer; in the dialog the number of **additional** channels, in a macro the **total** | 0 / 1 |
| `Segmentation style:` | `segmentation-style` | `[Keep intensities above threshold (creates a background-removed image)]`, `[Set intensities above thresholds to maximum possible intensity value (creates a binary image)]` | keep intensities |

**In a macro, a preprocessing step is switched on by giving its key** and off by leaving it out.
`[code]`

**The pre-filled radii are not recommendations.** They are in pixels, so they only fit images at
one particular pixel size. Choose them with §4.

### 3.3 Allowed values

**Segmentation method** (`c1-method`) — 17 ImageJ auto-threshold algorithms and 3 special methods:

```
Default  IJ_IsoData  Huang  Intermodes  IsoData  Li  MaxEntropy  Mean  MinError  Minimum
Moments  Otsu  Percentile  RenyiEntropy  Shanbhag  Triangle  Yen
CANNY 3D  CUSTOM threshold  HYSTERESIS threshold
```

**Hysteresis low/high methods** (`c1-hyst-low`, `c1-hyst-high`) — 18 values:

```
CUSTOM threshold  Default  IJ_IsoData  Huang  Intermodes  IsoData  Li  MaxEntropy  Mean  MinError
Minimum  Moments  Otsu  Percentile  RenyiEntropy  Shanbhag  Triangle  Yen
```

**Canny 3D low/high methods** (`c1-canny-low`, `c1-canny-high`) — 17 values, a **different list**:
there is no `Default`, and the custom option is called `Custom Value`, not `CUSTOM threshold`.
`[code]`

```
Custom Value  Huang  Intermodes  IsoData  IJ_IsoData  Li  MaxEntropy  Mean  MinError  Minimum
Moments  Otsu  Percentile  RenyiEntropy  Shanbhag  Triangle  Yen
```

**Stack handling** (`c1-stack`):

```
[apply threshold determined in the stack histogram]
[apply threshold determined in a maximum-intensity-projection]
[input image is no stack]
```

### 3.4 Hysteresis dialog (only with `HYSTERESIS threshold`)

| dialog field | macro key | default |
|---|---|---|
| `Select method for low threshold` | `c1-hyst-low` | `Triangle` |
| `Low threshold (if 'CUSTOM threshold' is chosen)` | `c1-hyst-low-thr` | 0 |
| `Select method for high threshold` | `c1-hyst-high` | `Otsu` |
| `High threshold (if 'CUSTOM threshold' is chosen)` | `c1-hyst-high-thr` | 0 |

Both thresholds are determined on the **preprocessed** image, with the chosen **stack handling** (MIP or stack
histogram) and, if set, per time step. Then the 3D Hysteresis Thresholding of the **3D ImageJ Suite** keeps every
region above the **low** threshold that contains at least one voxel above the **high** threshold; voxels above the
low threshold that touch, also across planes, form one region (connected **in 3D**). Requires the 3D ImageJ Suite
update site. `[code]` `[wiki]`

So **the low threshold sets how wide the objects are; the high threshold decides which objects are kept.** A kept
object keeps its full outline; nothing is thinned. Use hysteresis when an inclusive single threshold keeps the cilia
whole but also keeps background specks, and a strict one drops the specks but thins the cilia or breaks dim ones.
`[wiki]`

- **Choosing the pair:** an *inclusive* method low (it outlines even dim cilia completely; specks do not matter) and
  a *strict* method high (it finds a bright part of every real cilium but nothing in the background). Triangle / Otsu
  (the default) and Li / Otsu are common pairs. `[wiki]`
- **Low too low:** masks wider than the cilia, close objects join. **Low too high:** cilia thin, broken into pieces.
- **High too low:** background specks kept. **High too high:** whole dim cilia disappear (an object is kept whole or
  dropped).
- **Custom values** (`CUSTOM threshold`) are applied unchanged to every image; brightness often differs between images
  of a batch, so an automatic pair is usually more robust. Test a few pairs on several images. `[wiki]`
- **The low threshold must be below the high one.** If it is not, v0.2.0 shows a note for each affected image (in
  the Log and the progress window), and the result is no hysteresis: simply everything above the low threshold (e.g.
  Otsu low / Triangle high gave 302 objects, Triangle alone 304, §9 Example 5). `[code]` `[wiki]` `[measured]`
- **The same method for both** works like a plain threshold; v0.2.0 warns before it starts. `[code]`
- **Mostly-zero backgrounds:** the high method can return 0, so the pair inverts and everything above the low
  threshold is kept; the same note appears. Use other methods or custom values (§7). `[code]` `[wiki]`
- The computed values are in `_CQP.txt` ("Hysteresis threshold(s) for channel …"), for numeric comparison of pairs.
- Measured examples: §9, Example 5.

### 3.5 Canny 3D dialog (only with `CANNY 3D`)

| dialog field | macro key | default |
|---|---|---|
| `Sigma for Gaussian blur (pixels)` | `c1-canny-sigma` | 1.0 |
| `Alpha (sensitivity for edge detection)` | `c1-canny-alpha` | 5.0 |
| `Select method for low threshold` | `c1-canny-low` | `Triangle` |
| `Value (if custom value is chosen)` | `c1-canny-low-thr` | 0 |
| `Select method for high threshold` | `c1-canny-high` | `Otsu` |
| `Value (if custom value is chosen)` | `c1-canny-high-thr` | 0 |

Canny 3D finds objects by their **edges** (where the intensity changes steeply, e.g. at the border between a cilium
and the background), not by their brightness. (1) Each plane is blurred in 2D (sigma); (2) a 3D Canny-Deriche filter
(Deriche's fast version of Canny's edge detector, applied along x, y and z; from the 3D ImageJ Suite, via Sebastian
Rassmann's Canny 3D Thresholder) computes the **edge strength** of every voxel (alpha); (3) a low/high hysteresis
threshold is applied **to the edge strength**; (4) the objects enclosed by the edges are filled. A dim cilium with a
steep border is found like a bright one, so Canny 3D can keep dim parts that an intensity threshold loses.
`[code]` `[paper]` `[wiki]` Consequences:

- **When to use it:** unevenly labelled cilia (bright and dim parts) that even hysteresis on the intensity breaks
  into pieces. **The cost:** wider masks, especially in z, and close cilia join more easily. For evenly stained cilia
  an intensity threshold outlines the cilium more precisely. `[wiki]` `[paper]`
- **The threshold pair changes the result most.** An inclusive method low, a strict one high; Triangle / Otsu (the
  default) is a good start. **Pairs that work for hysteresis on the intensity need not work on the edge strength:
  Li / Otsu fails with Canny 3D** (one object holding a quarter of the stack in the measured example). Low too low:
  background edges kept, large parts of the stack become one object. Too strict: dim cilia lost or broken. `[wiki]`
- **Alpha and sigma change mainly how wide the masks are**, not which objects are found. **Lower alpha** = a wider
  filter reach and wider masks; above the default 5 little changes. The filter works in voxel steps, so where the
  planes are further apart than the pixels its reach in µm is larger in z — likely why the masks widen mostly in z.
  **Higher sigma** = wider masks, and close objects join. Usually keep **alpha ≥ 5 and sigma ≤ 1**. `[wiki]`
- Canny 3D has its own blur. An additional blur (step 4) is usually not needed; with a pre-blur already applied,
  sigma 0.5 and 1 give nearly the same masks. `[wiki]`
- Its thresholds are computed from the edge strength of the **whole stack**, per time point. Stack handling and
  "threshold every time step independently" do not apply (the macro has no `c1-stack` key for it). `[code]`
- Custom values would be edge strengths, not image intensities — but see the bug below.
- **Hollow objects at the first and last plane** are expected: the edges enclose an object there without filling it.
  CiliaQ's blur before skeletonization fills them. `[wiki]`
- Requires the **3D ImageJ Suite** update site.
- Measured examples: §9, Example 5.

> **Known bug in v0.2.0, to be fixed in the next version:** `Processing.java` tests `pS.lowThrAlgorithm == "Custom"`,
> but the list value is `"Custom Value"` (and `==` compares references), so Canny custom values are never applied.
> **Symptoms:** the message "No enum constant ij.process.AutoThresholder.Method.Custom", and **the whole image becomes
> one object**; on large stacks Fiji can run out of memory and stop with "Applying Canny 3D failed". `[code]` `[wiki]`
> `[measured]` Also, `createRecordString()` does not record `canny-low-thr` / `canny-high-thr`. **With v0.2.0, use
> only the automatic methods for Canny 3D.**

### 3.6 How macro options are read

`[code]`

- **Missing keys take defaults silently** (the defaults in the tables above). No dialog appears.
  This is how a run quietly becomes something other than what was intended — so write every key.
- **Flags must be written as bare words, exactly** (`c1-duplicate`, `keep-awake`, `c1-separate-timesteps`).
  `c1-duplicate=true` is read as **off**, and so is a different case.
- **Values are matched exactly first, then as a unique substring** (e.g. `MIP` would not work
  because it is not a substring, but `maximum-intensity` would). **Matching is case-sensitive** (`otsu` is not
  `Otsu`). If nothing matches, or the
  substring is ambiguous, the Log window shows
  `unknown value for '<key>': '<value>', using '<default>'` and the default is used.
- **Two routes:** either the per-channel keys above, or `settings=[/path/to/previous_CQP.txt]`,
  which loads all segmentation settings from an earlier metadata file (`output`, `number-format`
  and `keep-awake` still come from the macro line). **Every other key on the same line is ignored without a
  message** (e.g. a changed `c1-method=`), so do not mix the two routes. If a `_CQP.tif` is given by mistake, the
  matching `_CQP.txt` next to it is used and the swap is logged. `[release notes]`
- **The command name contains the version:** `run("CiliaQ Preparator v0.2.0", ...)`. Macros must
  be updated after an upgrade. `[release notes]`
- **Recording:** with `Plugins ▸ Macros ▸ Record…` active, configuring the plugin in the dialog
  records one replayable `run(...)` line. This is the easiest way for a user to hand a working
  setting to the assistant. `[release notes]`

---

## 4. Finding good settings — the workflow

This follows how the CiliaQ developer optimises settings in the wiki tutorials (§9). The aim is to
settle most choices live in Fiji in minutes, and to use plugin runs only to compare a few finished
candidates.

### Step 1 — Ask

Ask the questions in `00_START_HERE.md` §0 rule 3: microscope type, 2D/3D/time-lapse, pixel size and
z step, cilia channel, other stains, bit depth, and **what will be measured**. Also ask:

- Does the image background sit at exactly 0 anywhere? (Fiji: `Analyze ▸ Histogram`, or hover over
  background and read the value.) This decides the output style (§1) and rules out Divide by
  Background.
- How wide is a sharp cilium, in pixels? A line across one cilium with `Analyze ▸ Plot Profile`,
  width at half height. This gives a scale for the radii. Measure it by hand: deriving it from an
  automatic threshold mask was badly wrong on a test image where the threshold kept background.
  `[measured]`
- **Low resolution?** At about 1 µm per pixel (e.g. tissue scans) a cilium is only 1–3 px wide and 2–15 px long and
  often breaks into pieces. **Upscaling 2–3× before the Preparator can help** (`Image ▸ Scale…`, bilinear or
  bicubic, all planes): it adds no information but gives the methods more pixels across each cilium. Then adjust
  every pixel-based setting (radii, minimum sizes) in the Preparator and CiliaQ, and check in
  `Image ▸ Properties…` that the pixel size was scaled too. Memory and run time grow with the pixel count (9× in
  2D at 3×). `[wiki]`

### Step 2 — Look at the image

Have the user extract the cilia channel (`Image ▸ Duplicate…`, choose the channel), make a maximum
projection (`Image ▸ Stacks ▸ Z Project…`, Max Intensity), and lower the displayed maximum in
`Image ▸ Adjust ▸ Brightness/Contrast` until dim cilia are visible. Then ask what they see. `[wiki]`

| what the user sees | challenge | usually helps |
|---|---|---|
| cell bodies, diffuse or out-of-focus background, debris in the channel | background structures | Subtract Background (Step 3a) |
| uneven brightness across the field | uneven background | Subtract Background; Divide by Background |
| some cilia much brighter than others, or bright bases and dim tips | heterogeneous intensity | small-radius Subtract Background; hysteresis; Canny 3D |
| gaps, dotted staining along a cilium | discontinuous staining | blur; hysteresis |
| grainy, noisy image | noise | blur (pre-blur if background removal is used) |
| cilia lying very close together | fusion risk | avoid large blurs; avoid Canny 3D; plan for editing |
| one or a few extremely bright spots (aggregates, dirt) | threshold distortion (§7) | re-image, crop them out, or use a fixed threshold |

### Step 3 — Test preprocessing live, in pipeline order

Work on the projection, zoomed into the most difficult region, and use each Fiji command's
**Preview** checkbox. Step through values from too little to clearly too much, and describe what
each does. Apply the chosen step before testing the next one. `[wiki]`

The Preparator applies these steps to each slice and then thresholds a projection of the
*processed* stack, so testing on a projection is an approximation.

**3a. Subtract Background** — `Process ▸ Subtract Background…`, Preview on.

- Objects wider than about twice the radius are removed; narrower ones stay. Start a little above
  the width of a sharp cilium. `[wiki]`
- **Decrease the radius** to remove more background, out-of-focus blur, or to even out brightness
  *between* cilia. In the tutorials, 10 → 5 px removed more background while keeping all cilia,
  and 1 px left essentially only cilia in a tissue image. `[wiki]`
- **Increase the radius** if cilia become thinner, dimmer or break up.
- Values in practice range widely: 1–10 px in the wiki examples, 50 px in a published protocol at
  0.046 µm/px. Always choose by preview on the user's image. `[wiki]` `[paper]`
- Smaller radii raise the risk that a patchy cilium falls apart into fragments; a small blur after
  background removal reconnects it (3b). `[wiki]`
- If the answer to "are there other structures in the channel?" is no, background removal may not
  be needed at all.

**3b. Blur** — `Process ▸ Filters ▸ Gaussian Blur…`, Preview on; "Sigma (Radius)" corresponds to
the Preparator radius.

- Try 0.5, 1, 2, then something clearly too large (e.g. 5) to see the range. In one tutorial,
  0.5 px hardly closed gaps, 1–2 px connected the cilium and reduced noise, and 5 px blurred cilia
  away. `[wiki]`
- Blur widens the mask a little. That barely matters for length or for centreline and top-10 %
  intensity parameters; it slightly dilutes whole-cilium average intensity. `[wiki]`
- Blur also fuses cilia that lie close together, and it dims thin, dim cilia — exactly when
  threshold headroom is smallest.
- **Pre-blur (step 1) or additional blur (step 4)?** If background subtraction is used and the
  image is noisy, a pre-blur lets the background estimate work on a smoother image; a pre-blur of
  about a quarter of the rolling-ball radius is a reasonable start, and a pre-blur as large as the
  radius largely cancels the background subtraction. Otherwise use the additional blur. Using both
  smooths twice; the plugin warns about it. `[code]` `[release notes]`

**3c. Divide by Background** — consider only if brightness varies strongly across the field or
along cilia and Subtract Background is not enough. Never on images whose background is (near) 0:
the ratio becomes infinite or undefined and nothing is detected. Poor on 8-bit images. `[code]`

### Step 4 — Test thresholds live

On the preprocessed projection, open `Image ▸ Adjust ▸ Threshold…`, tick **Dark background**, and
step through the algorithms. These are the same algorithms the Preparator offers. `[wiki]`
Sort each into one of three groups:

- **inclusive** — catches every cilium completely, but also some background;
- **strict** — catches only cilia, but misses dim parts or dim cilia;
- **unusable** — e.g. makes bright cilia into fat blobs, or misses most cilia. `[wiki]`

Then choose the method:

| situation | method |
|---|---|
| one algorithm separates cilia and background well everywhere | that algorithm |
| no single algorithm works, but one is inclusive and one is strict | `HYSTERESIS threshold`, **low = the inclusive one, high = the strict one** (e.g. Triangle/Otsu or Li/Otsu in the tutorials; §3.4) `[wiki]` |
| intensities too heterogeneous even for hysteresis; unevenly labelled cilia | `CANNY 3D` with defaults first (Triangle / Otsu, sigma 1, alpha 5). It detects unevenly labelled cilia more completely but produces wider masks (mostly in z) and fuses close neighbours more readily; for homogeneously stained cilia, intensity thresholds outline the cilium more precisely. Change the threshold pair first; alpha and sigma mainly change the mask width (§3.5). Do not carry a hysteresis pair over unchecked: Li / Otsu fails with Canny 3D. `[paper]` `[wiki]` `[developer]`|
| a fixed threshold must apply to the whole dataset, e.g. a protein-of-interest channel that may be empty in controls (automatic methods always find "foreground", even in empty images) | `CUSTOM threshold`, from background measurements: draw ROIs in background on several images, measure mean and SD, e.g. threshold = mean of means + 3 × mean SD. Only valid if acquisition was identical for all images. Least recommended since requires a lot of user supervision. `[wiki]` `[developer]`|

Hysteresis and Canny 3D cannot easily be previewed in Fiji; they are tested by running the Preparator. For
hysteresis, the live test still helps: the inclusive and the strict method found here are the low and the high.

### Step 5 — Run 2–3 candidates

Run the Preparator on the **original, saved image** with the preprocessing from Step 3 and the
method(s) from Step 4. Each candidate should differ from another in **one** setting, and the
assistant should say what each one tests (e.g. blur 1 vs. 2; hysteresis vs. Canny 3D).

**Stack handling:** for z-stacks use the maximum-intensity projection — it gives a better
foreground/background ratio for small structures like cilia, and is the recommended default.
Use the stack histogram if the signal is uniform through z. Use `[input image is no stack]` only
for single planes: on a stack it takes the threshold from the **first slice only**. `[paper]`
`[wiki]` `[code]` The stack histogram gives lower thresholds, because most voxels are background (§9, Example 5).

**Time-lapse:** tick "threshold every time step independently" only if brightness changes over
time (e.g. bleaching). It makes masks of different time points less comparable. `[wiki]`

**Editing later?** Tick "Include also an unsegmented copy of the channel" (`c1-duplicate`) — the
Editor needs it.

### Step 6 — Inspect

Viewing recipe for `_CQP.tif`: `Image ▸ Color ▸ Channels Tool…`, Composite, show only the
segmented channel and its unsegmented copy; colour the segmented channel cyan and the copy red
(detected voxels then appear white, missed signal red); lower the maximum of the segmented channel
in Brightness/Contrast so all detected voxels are clearly visible. A maximum projection of the
output gives an overview; check single slices too, and **the whole image**, not only the region
used for tuning. **Check side views (x–z) too** (`Image ▸ Stacks ▸ Orthogonal Views`): Canny 3D and a low
hysteresis low threshold widen masks mostly in z. `[wiki]`

| what the user sees | meaning | what to change |
|---|---|---|
| cilia thinner than in the original, or broken | too strict | more inclusive method; hysteresis (a more inclusive **low**); larger subtract-background radius; small blur |
| background structures kept | too inclusive, or no background removal | stricter method or hysteresis (a stricter **high**); Subtract Background |
| dim cilia missing completely (hysteresis) | high threshold too strict | a more inclusive **high** |
| one cilium split into several pieces | gaps | blur; hysteresis; or leave it to CiliaQ's connectivity option (see `ciliaq/CILIAQ_v0.2.1.md`) |
| neighbouring cilia merged | over-bridged | smaller blur; hysteresis: a stricter **low**; Canny 3D: lower sigma, alpha ≥ 5, or not Canny 3D; Editor |
| bright cilia as fat blobs | inclusive threshold on very bright cilia | stricter method; hysteresis |
| almost nothing kept | threshold near the maximum | check for bright particles (§7); more inclusive method |
| everything kept | threshold at or below the background | check for a zero background and the hysteresis trap (§7) |
| one object fills most of the stack (Canny 3D) | low threshold too low on the edge strength, or `Custom Value` in v0.2.0 | a stricter low method (e.g. Triangle instead of Li); never `Custom Value` in v0.2.0 |
| hollow cilia near the top/bottom of the stack (Canny 3D) | edge detection at the z border | usually fine: CiliaQ's blur before skeletonization fills them `[wiki]` |

### Step 7 — Decide, check on more images, stop

- Name the remaining trade-off and let the user choose the bias that suits the measurement
  (`00_START_HERE.md` §4). Example from the tutorials: hysteresis plus *adding* missed voxels in
  the Editor, or Canny 3D plus *removing* false positives. `[wiki]`
- Structures that even the user cannot classify can be kept and removed later — in the Editor, or
  by excluding their IDs from CiliaQ results. `[wiki]`
- **Check the chosen setting on several images of the dataset** before processing all of it, and
  then use one setting for the whole dataset. `[wiki]`
- Very heterogeneous regions in one image can be cropped into separate images and processed
  separately. `[wiki]`

---

## 5. Running the plugin

### 5.1 Dialog

`Plugins ▸ CiliaQ ▸ CiliaQ Preparator v0.2.0`. Every macro key in §3 corresponds to a dialog field.
The assistant can give a table of field → value instead of a macro.

### 5.2 Macro line

Open `File ▸ New ▸ Script…`, set the language to **IJ1 Macro**, paste, and press Run.
Template — **write every key**; add or remove only the preprocessing keys:

```javascript
open("/full/path/to/image.tif");
run("CiliaQ Preparator v0.2.0",
    "process=[active image in FIJI] " +
    "output=[save as filename + suffix 'CQP'] " +
    "number-format=[US (0.00...)] " +
    "segmentation-style=[Keep intensities above threshold (creates a background-removed image)] " +
    "channels=1 " +
    "c1-nr=3 " +
    "c1-duplicate " +
    "c1-preblur=2.000 " +
    "c1-subtract-bg=10.000 " +
    "c1-method=[RenyiEntropy] " +
    "c1-stack=[apply threshold determined in a maximum-intensity-projection]");
```

This exact line has been run against the published v0.2.0. `[measured]`

Method-specific keys to add:

```
hysteresis:  "c1-method=[HYSTERESIS threshold] c1-hyst-low=[Triangle] c1-hyst-high=[Otsu] "
             custom values: "c1-hyst-low=[CUSTOM threshold] c1-hyst-low-thr=862 c1-hyst-high=[CUSTOM threshold] c1-hyst-high-thr=4680 "
custom:      "c1-method=[CUSTOM threshold] c1-custom-thr=1500 "
Canny 3D:    "c1-method=[CANNY 3D] c1-canny-sigma=1.0 c1-canny-alpha=5.0 c1-canny-low=[Triangle] c1-canny-high=[Otsu] "
             (no c1-stack / c1-separate-timesteps needed for Canny 3D; no Custom Value in v0.2.0)
```

For a second segmented channel, set `channels=2` and add the same keys with the prefix `c2-`, e.g. image channels 2
and 4 with hysteresis for both:
`channels=2 c1-nr=2 c1-method=[HYSTERESIS threshold] c1-hyst-low=[Triangle] c1-hyst-high=[Otsu] c2-nr=4 c2-method=[HYSTERESIS threshold] c2-hyst-low=[Li] c2-hyst-high=[Otsu]`.
`[wiki]`

### 5.3 Reusing a setting

- `settings=[/path/to/previous_CQP.txt]` applies a setting that worked to another image, or to a
  whole folder in a loop. Do not write such a file by hand unless you are really sure what to do —
  it is tab-delimited with meaningful blank lines. `[release notes]` Other keys on the same line are ignored (§3.6).
- Older settings files: before v0.2.0, `IJ_IsoData` was read back as `IsoData`. Check the method in
  the Log after loading an old file. `[release notes]`
- For batch processing a folder by macro, and for running headless, see `agents/AGENT_GUIDE.md`.
  Skip `_CQP.tif` files in loops, or a re-run produces `_CQP_CQP.tif`. `[release notes]`

### 5.4 If the run fails

| what happens | likely reason | fix |
|---|---|---|
| `Unrecognized command: "CiliaQ Preparator v0.2.0"` | plugin not installed, or a different version | check the exact entry under `Plugins ▸ CiliaQ` and use that string |
| the wrong file opens, or none (Windows) | `\` is an escape character in macro strings | use forward slashes `C:/data/img.tif`, or double every backslash |
| a setting seems ignored | missing or misspelt key → default used; a flag written `=true`; other keys next to `settings=` | compare with §3; check the Log for `unknown value`; check `_CQP.txt` |
| warning about an unsaved image | the image exists only in Fiji (e.g. a projection or crop) | save it first, then run `[release notes]` |
| hysteresis or Canny 3D fails | 3D ImageJ Suite not installed | enable the "3D ImageJ Suite" update site, restart Fiji `[release notes]` |
| "No enum constant ij.process.AutoThresholder.Method.Custom", whole image one object, or "Applying Canny 3D failed" | `Custom Value` with Canny 3D (v0.2.0 bug, §3.5) | use an automatic method |
| image skipped: channel does not exist | `c1-nr` higher than the number of channels | fix the channel number `[code]` |
| nothing saved | the image was not the active one | `open(...)` directly before `run(...)`, as in the template |

---

## 6. Reading the result

### 6.1 `_CQP.txt`

About 1 KB, no image data. It records the settings that ran, the threshold(s) the plugin computed,
the channel mapping of the output, `Plug-in version` and `ImageJ version`. `[code]` `[release notes]`
Examples of the threshold lines (numbers illustrative):

```
Threshold(s) for channel 3
	All timesteps:	7470.000

Hysteresis threshold(s) for channel 3
	Time step #	Low Threshold	High Threshold
	1	2402.000	5130.000
```

### 6.2 What the threshold tells you

- **Compare it with the image's values:** a threshold at or below the background level keeps
  nearly everything; one close to the maximum keeps almost nothing.
- **ImageJ computes auto-thresholds from a 256-bin histogram** spanning the image's minimum to
  maximum. A few very bright voxels make the bins wide, so the threshold moves in large steps and
  can land far above the cilia. This is why a single aggregate can ruin segmentation, and why
  different ImageJ versions can shift a threshold noticeably on sparse images. `[release notes]`
- Reproducing an analysis exactly requires the same ImageJ version as recorded in `_CQP.txt`.
  `[release notes]`
- **Read the Log window too.** v0.2.0 writes warnings there: inverted hysteresis thresholds, stack
  handling that does not match the image, a reminder whenever Divide by Background is used (it
  fails on a zero background), both blurs switched on, a pre-blur not smaller than the
  rolling-ball radius, a custom threshold left at 0, and replaced macro values. In dialog runs the
  warnings also pop up as messages. `[code]`

### 6.3 Compare runs; do not judge one run by an absolute number

There is no correct foreground fraction. Across 98 segmentations an expert rated as good, the
fraction of the field that was cilium varied by a factor of about 500, because fields differ in how
many and how large their cilia are. What does transfer is the **comparison of two runs on the same
image**: against a good segmentation of the same field, an over-segmented mask had more foreground
in about 90 % of cases (median 2.5×), and an under-segmented mask less in about 90 % (median 0.2×).
Object count alone is unreliable, because over-segmentation both fuses neighbours and adds specks.
`[measured, internal data]`

So: compare candidates, reason about their differences, and let the user's eyes decide which end
is right. CiliaQ's results (object counts, volumes, objects without length, branches; count objects
as result rows, `ciliaq/CILIAQ_v0.2.1.md` §4.2) can then provide also a quantitative check — see
`ciliaq/CILIAQ_v0.2.1.md`.

---

## 7. Traps

- **True zero background + "keep intensities"** — mask voxels with original value 0 are lost in
  the output; gaps bridged by blur reopen. Use the binary style. `[code]` `[wiki]`
- **Rescaled intensity scales** — in some exports (e.g. from image databases) most voxels are
  exactly 0 and the next value is far above 0. Every threshold between 0 and that value gives the
  same mask, so trying more threshold algorithms changes nothing; preprocessing and method choice
  matter instead. Spot it as a huge histogram spike at 0 with an empty gap above it. `[measured]`
- **Hysteresis on mostly-zero images** — the high-threshold algorithm can return 0, the pair
  inverts, and the result becomes everything above the low threshold. The Log warns per image.
  Use custom values or different algorithms. `[release notes]`
- **Bright particles** — aggregates or dirt stretch the histogram (§6.2) and most cilia vanish.
  Re-image, crop them out, or use a fixed threshold. `[wiki]` `[developer]`
- **Divide by Background on a (near-)zero background** — nothing is detected. `[code]`
- **Radii are in pixels.** A setting tuned at one pixel size is wrong at another; convert when
  reusing settings across acquisitions.
- **Blurs are 2D only** — no smoothing along z but that is usually not a problem since most
  microscopes have a larger point spread function along z so microscopes naturally blur sub-
  diffraction-limit objects like cilia in the z direction. `[code]` `[developer]`
- **`[input image is no stack]` on a z-stack** — threshold from the first slice only. The Log
  warns. `[code]`
- **`c1-duplicate` in macros** — off unless the flag is written (as a bare word; `c1-duplicate=true` is off too),
  although the dialog ticks it by default. Without it the Editor cannot be used later. `[code]`
- **`c<n>` is not the image channel** — `c1-` is the first segmented channel, whatever its number (§3.2). `[wiki]`
- **Canny 3D thresholds work on edge strength**, not intensity; stack handling does not apply; a pair that works for
  hysteresis may fail (Li / Otsu). `Custom Value` does not work in v0.2.0. `[code]` `[wiki]`

---

## 8. Further segmented channels

The Preparator can segment more than one channel (`channels=2`, keys `c2-…`). Common uses:

- **Basal bodies** — for CiliaQ v0.2.x basal-body detection in static 2D and 3D images
  (see `ciliaq/CILIAQ_v0.2.1.md` §3.6; in v0.2.1, basal bodies smaller than CiliaQ's minimum cilium
  size are dropped, §2.5). A basal body is a dot, not a line: give it its own settings, typically less blur and no
  gap bridging; do not copy the cilia settings with only the channel number changed. `[developer]`
- **A protein of interest** — to measure its overlap with cilia in CiliaQ. If some images may have
  no real signal (controls), use a `CUSTOM threshold` determined from background to threshold the
  channel rather than using CiliaQ's compared to background parameters (§4 Step 4). `[wiki]`

---

## 9. Worked cases from the wiki

These are condensed from the developer's tutorials. Each links to the full page with images.

**Example 1 — cilia on cultured cells.** Blurred background structures and large intensity
differences within cilia. Subtract Background 5 px removed the blurred background. In live
thresholding, Huang left cilia discontinuous, Triangle made bright cilia into blobs, and Li worked
best but still kept some background. Canny 3D with default settings plus Subtract Background 5 px
detected all cilia evenly; some masks were hollow at the z borders, which CiliaQ's blur before
skeletonization (2.0) fixed; a few touching cilia had to be separated in the Editor. `[wiki]`
https://github.com/hansenjn/CiliaQ/wiki/Q&A:-How-to-find-an-appropriate-segmentation-method-for-my-data-sets%3F

**Example 2 — confocal stack of cilia in tissue.** Cilia brightness varied widely and overlapped
with background staining; Otsu on the raw projection kept background and missed cilia. Subtract
Background with a decreasing radius (5 → 1 px) removed cell labelling and evened out cilia; a blur
bridged gaps. Triangle was inclusive and Otsu strict, so hysteresis Triangle/Otsu detected most
cilia, some only sparsely. Canny 3D (with Subtract Background only, since Canny blurs itself)
detected all cilia plus some background, and fused close cilia more. Conclusion: choose the bias —
hysteresis plus adding voxels in the Editor, or Canny 3D plus removing structures. `[wiki]`
https://github.com/hansenjn/CiliaQ/wiki/Q&A:-How-to-find-an-appropriate-segmentation-method-for-my-data-sets-(2)%3F

**Example 3 — 3D culture with very variable intensity.** The user's setting (Subtract Background
10 px, blur 0.5 px, RenyiEntropy on the MIP) missed thin, dim cilia. Subtract Background 5 px
removed more background while keeping all cilia; blur 0.5 px hardly closed gaps, 1–2 px did, 5 px
was far too much. Li was inclusive and Otsu strict → hysteresis Li/Otsu. Runs with blur 1 and 2 px
were compared: 2 px lost one cilium, so 1 px was chosen. Structures that could not be classified
were kept for later removal. `[wiki]`
https://github.com/hansenjn/CiliaQ/wiki/Q&A:-Finding-better-segmentation-settings-(Example-3)

**Example 4 — widefield (non-confocal) 3D image.** Default settings with Canny 3D included
out-of-focus blur, so masks were wide and skeletons kinked. Subtract Background 5 px removed the
out-of-focus signal (it removes objects wider than about 10 px) and kept the sharp cilia; a 1 px
blur after it reconnected patchy cilia. In CiliaQ, a larger minimum object size removed small
spots. Skeletons then followed the cilia. `[wiki]`
https://github.com/hansenjn/CiliaQ/wiki/Q&A:-The-detected-ciliary-objects-(_RP_3D.tif-file)-are-much-wider-and-deformed-compared-to-how-I-see-the-cilia-in-the-image%3F-Why%3F

**Example 5 — one stack, every threshold setting varied (method pages, November 2026).** An ICC IF image of cilia,
2048 × 2048 px, 14 planes, 0.07 µm per pixel, 0.70 µm between planes; preprocessing in all runs: pre-blur 2,
Subtract Background 15; MIP stack handling unless stated; binary output; objects counted 26-connected, as CiliaQ
does, for the whole field. Measured with v0.2.0 for the wiki pages and rerun with a build of the tag v0.2.0
(November 2026): every number reproduced. `[wiki]` `[measured]`

| run | thresholds | objects | what it looked like |
|---|---|---|---|
| Triangle alone | 862 | 304 | cilia at full width plus almost 300 specks |
| Otsu alone | 4680 | 21 | no specks; cilia thin; the cilium with a dim middle in two pieces |
| hysteresis Triangle / Otsu (default) | 862 / 4680 | 15 | full width, no specks |
| hysteresis, low 400 (too low) | 400 / 4680 | 14 | masks much wider than the cilia, most visibly in x–z; a speck merges with a cilium |
| hysteresis, low 2000 (too high) | 2000 / 4680 | 16 | masks shrink to the brightest parts; the dim-middle cilium breaks in two |
| hysteresis, high 1500 (too low) | 862 / 1500 | 52 | background specks kept |
| hysteresis, high 10000 (too high) | 862 / 10000 | 8 | dim cilia gone completely |
| hysteresis, high 20000 | 862 / 20000 | 3 | only the brightest objects; kept objects unchanged in size |
| hysteresis, stack histogram | 739 / 3571 | 19 | lower thresholds (most voxels are background) |
| hysteresis Otsu / Triangle (inverted) | 4680 / 862 | 302 | the note "not below the high threshold"; about Triangle alone (304) |
| Canny 3D defaults (Triangle / Otsu, sigma 1, alpha 5) | edge strength | 18 | cilia whole plus a few specks; **1.7× the volume of the hysteresis masks, mostly in z** |
| Canny 3D Li / Otsu | edge strength | 1 | one object holding 23 % of all voxels of the stack |
| Canny 3D Otsu / MaxEntropy (too strict) | edge strength | 13 | a dim cilium lost; the dim-middle cilium in two |
| Canny 3D alpha 1 / 2 / 5 / 10 | – | 17 / 17 / 18 / 18 | alpha 1: 1.75× the volume of alpha 5, mostly in z; 2: 1.2×; 10 = 5 |
| Canny 3D sigma 0.5 / 1 / 2 / 4 | – | 20 / 18 / 17 / 14 | 0.5 ≈ 1 (the pre-blur had already smoothed); 2 wider; 4 much wider, a speck joins a cilium |
| Canny 3D `Custom Value` | – | – | stops: "No enum constant ij.process.AutoThresholder.Method.Custom" (§3.5) |

What transfers: the direction of each change, not the numbers. Hysteresis: low = width, high = which objects; the
high never thins a kept object. Canny 3D: the threshold pair decides the objects, alpha and sigma the width.
Pages (they describe the newest version and may since have changed):
https://github.com/hansenjn/CiliaQ/wiki/Hysteresis-threshold-in-CiliaQ-Preparator,
https://github.com/hansenjn/CiliaQ/wiki/Canny-3D-in-CiliaQ-Preparator


---

## 10. Known issues in v0.2.0 (to be fixed in the next version) `[code]`

- **Canny 3D `Custom Value`** is never applied: "No enum constant …Method.Custom", the whole image becomes one
  object, on large stacks "Applying Canny 3D failed". The recorder also omits `canny-low-thr` / `canny-high-thr`.
  Use automatic methods. (§3.5) `[wiki]` `[measured]`
- **Macro flags written `key=true`** (or in another case) are read as off. Write bare flags. (§3.6)
- **Choice values are case-sensitive** (`otsu` falls back to the default with an `unknown value` line). (§3.6)
- **`settings=` ignores every other segmentation key** on the line without a message (only `output`,
  `number-format` and `keep-awake` are read). (§3.6)
- **A macro always processes the front-most image**, whatever `process=` says. (§3.1)

---

## 11. Not covered here

- CiliaQ's own settings and results: `ciliaq/CILIAQ_v0.2.1.md`.
- Manual correction: `editor/EDITOR_v0.0.4.md`.
- Running headless or in batch by an agent: `agents/AGENT_GUIDE.md`.
- Preparator versions other than v0.2.0 (and v0.1.2 as described in §0).
