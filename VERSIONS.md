# CiliaQ versions and compatibility

**Kit version:** 0.0.2 · **Last updated:** November 2026
**Sources:** GitHub release notes and issues of all three repositories, source code at the release tags,
the CiliaQ wiki, test runs (November 2026), statements by the developer.

Read `00_START_HERE.md` first. This file answers: which version should be used, what changed in a
way that affects results, and which results can be combined.

---

## 1. Current recommendation

| tool | use | covered by this kit | scriptable |
|---|---|---|---|
| CiliaQ Preparator | **v0.2.0** | yes | macro and headless |
| CiliaQ Editor | **v0.0.4** | yes | no (interactive only) |
| CiliaQ, static 2D/3D images | **v0.2.1** | yes | macro and headless `[release notes]` |
| CiliaQ, time-lapse images | **v0.1.7** | yes | no (dialog only) `[code]` |

`[developer]`

- CiliaQ v0.2.0 and v0.2.1 are labelled beta on their release pages and in a dialog when started
  from the menu. For static images the developer considers v0.2.1 suitable. `[release notes]`
  `[code]` `[developer]`
- The Fiji update site can lag behind the newest releases. A specific version can be installed by
  hand: https://github.com/hansenjn/CiliaQ/wiki/Q&A:-How-to-install-a-specific-version-of-a-CiliaQ-plugin%3F
- Static images (v0.2.1) and time-lapse images (v0.1.7) are analysed with different CiliaQ versions, so their
  results must not be pooled or compared directly (rule 1; for basal-stain orientation see also rule 3).
- For a version this kit does not cover, the assistant should say so and give only
  version-independent advice (`00_START_HERE.md` §0 rule 2).

---

## 2. Rules for combining results

1. **Everything that is compared is processed with the same versions and the same settings** —
   Preparator, Editor and CiliaQ.
2. **CiliaQ skeleton-based results** (length, bending, orientation, profiles, curvature) from
   **v0.1.7 or later must not be pooled with earlier versions.** `[release notes]`
3. **CiliaQ v0.2.0+ with a basal stain** decides the ciliary base within 1 µm (strictly: 1 in the image's unit,
   see the known issues in §5) of the skeleton ends instead of 3 µm. For cilia shorter than about 3 µm,
   base and tip can therefore be assigned differently than in v0.1.7. Check before pooling orientation- or
   profile-direction results across this boundary. `[release notes]`
4. **CiliaQ Editor v0.0.4 edits** can be combined with edits from earlier Editor versions only if
   "Add pixels to mask even if intensity is 0.0…" was **off**. `[release notes]`
5. **Preparator v0.2.0 and v0.1.2** give byte-identical output with the same settings, pre-blur off,
   no Canny 3D threshold set to `Custom Value` (not applied in v0.2.0, §3), settings not loaded
   from a file that names `IJ_IsoData` (v0.1.2 read it as `IsoData`), and the same ImageJ version. `[release notes]`
6. **ImageJ version:** auto-thresholds can differ slightly between ImageJ versions. To reproduce a
   Preparator run exactly, use the ImageJ version recorded in `_CQP.txt` (recorded since v0.2.0).
   `[release notes]`
7. **Same skeleton blur (gauss filter before skeletonization)** for everything compared. `[wiki]`
8. **Uncalibrated images (GitHub #15; all CiliaQ versions up to v0.2.1).** Affected only when `Image ▸ Properties…`
   shows the unit "pixel" with pixel width, height and voxel depth all 1; the `_CQ.txt` header then shows
   `Calibration [pixel/px] 1.000000`. Skeleton lengths are 3× too long (v0.0.5–v0.2.1), and the first skeleton point
   lies at three times its true position (v0.0.6–v0.2.1). This distorts orientation vectors, bending index, profiles,
   arc lengths and the basal-stain orientation, and from v0.2.0 also the basal-body matching. Volume, surface and
   intensities are correct. Never pool such skeleton results with anything: set the real pixel size and z step and
   rerun. Any calibration avoids the bug, even 1 µm per pixel, but with such a placeholder every setting and result in
   calibrated units (lengths, the reference distance, the basal-stain radius, the basal-body distance) refers to
   pixels. A fix is planned for v0.2.2. `[code]` `[measured]` `[developer]`
9. **Intensity profiles normalised to the reconstruction channel are 0 in every CiliaQ version so far** (up to
   v0.2.1). Use only the raw profiles. A fix is planned for v0.2.2. `[code]` `[measured]`

---

## 3. CiliaQ Preparator

| version | date | changes that matter for results or use |
|---|---|---|
| **v0.2.0** | 2026-08 | macro options, recorder and headless mode; optional **pre-blur** before background removal; setup checks and warnings (e.g. a reminder whenever Divide by Background is used, inverted hysteresis thresholds, stack-handling mismatch); `_CQP.txt` records the ImageJ version and pre-blur; loading `IJ_IsoData` from settings files fixed (was read as `IsoData`); loading files with more than 3 channels fixed. **Output identical to v0.1.2 under the conditions of §2 rule 5.**|
| v0.1.2 | 2023-09 | warning for unsaved images; virtual stacks handled; Canny 3D low/high threshold swap fixed — affected only runs combining a custom value for one Canny threshold with an algorithm for the other; loading such old settings triggers a warning |
| v0.1.1 | 2021-03 | 2D hole filling for Canny 3D; "black background" forced on (previously Canny 3D could fail with that ImageJ preference off); stay-awake mode |
| v0.1.0 | 2020-09 | loading settings from `_CQP.txt`; Divide by Background; Gaussian blur; hysteresis threshold; custom threshold |
| v0.0.6 | 2020-06 | all series of multi-series microscopy files processed (before: only the first); settings files from this version onward can be loaded |
| v0.0.3–v0.0.5 | 2020-05 | Canny 3D added (v0.0.3); bug fixes |
| v0.0.2 | 2020-05 | first release |

All entries `[release notes]` unless tagged otherwise.

**Known issues in v0.2.0** (to be fixed in the next version; details in `preparator/PREPARATOR_v0.2.0.md` §10):
`[code]` `[developer]`
- Canny 3D `Custom Value` thresholds are not applied: "No enum constant ij.process.AutoThresholder.Method.Custom",
  the whole image becomes one object, on large stacks "Applying Canny 3D failed" `[wiki]` `[measured]`; the recorder
  does not record `canny-low-thr` / `canny-high-thr`.
- Macro flags written `key=true` are read as off; choice values are case-sensitive.
- `settings=` ignores the other segmentation keys on the line without a message.
- A macro always processes the front-most image, whatever `process=` says.

---

## 4. CiliaQ Editor

| version | date | changes |
|---|---|---|
| **v0.0.4** | 2025-10 | add/remove a selection in all slices (F3/F4); option to add pixels that are 0 in the unsegmented copy; works on binary masks; automatic display set-up (magenta/green) and sliders; checks that channels exist and match pixel-wise. **Mixing with older edits only with the zero-pixel option off.** |
| v0.0.3 | 2020-06 | saved ROI file names include slice, frame and add/remove |
| v0.0.2 | 2020-05 | undo fixed |
| v0.0.1 | 2020-05 | first release |

`[release notes]`

---

## 5. CiliaQ

| version | date | changes that matter for results or use |
|---|---|---|
| **v0.2.1** | 2024-10 | fixes a crash when there are more cilia than basal bodies |
| v0.2.0 | 2024-10 | macro options, recorder and headless mode; basal-stain orientation radius 3 → 1 (in the image's unit, normally µm); basal-body detection from a segmented channel, matching to cilia and skeleton extension to the basal body (static 2D and 3D images) `[developer]`; basal-body result columns and counts; intensity profiles moved from `_CQ.txt` to `_CQl.txt`; for 2D images, z-border exclusion switched automatically to x/y borders `[code]`; beta warning dialog |
| **v0.1.7** | 2023-05 | **improved skeletonization and length**, especially for small cilia in 2D: 2D cilia skeletonized in 2D (before: in a 3D image with added empty planes), enough margin for the blur in each cilium's image (before: edge effects); biases before v0.1.7 were a few percent in long cilia and up to 100–200 % in very short ones — **do not mix skeleton results with earlier versions**; 3D overviews of static images saved as `.png`; faster. Dialog only (no macro or headless use) `[code]`; basal stain for orientation only, within 3 µm; time-lapse: frames connected by overlap in the fill, an object removed when any of its frames is below the minimum cilium size, time-averaged `_CQ.txt` plus per-cilium kinetics files `[code]`. File: `ciliaq/CILIAQ_v0.1.7.md`. |
| v0.1.4 | 2020-12 | `_CQl.txt` output with tangent vectors and curvature (new setting: reference distance); loading settings from `_CQ.txt`; two-step dialog; results compatible with earlier versions |
| v0.1.0–v0.1.3 | 2020-05 to 2020-07 | "colocalized on centreline" parameters (v0.1.0); menu entry `Plugins ▸ CiliaQ` (v0.1.1); 3D visualization fixes |
| v0.0.5–v0.0.10 | 2019-12 to 2020-04 | first release (v0.0.5); "increase range" connectivity, integrated and centreline intensity, thresholds in the output (v0.0.6); pattern-based file selection (v0.0.10); results compatible within v0.0.6–v0.0.10 |

All entries `[release notes]` unless tagged otherwise.

**Known issues in v0.2.1** (details in `ciliaq/CILIAQ_v0.2.1.md` §8; fixes planned for v0.2.2): `[code]`
- **Uncalibrated images:** skeleton results wrong (GitHub #15). `[measured]`
- Without basal-body segmentation, the `_CQ.txt` header shows `of total #: 0`. Count the result rows instead
  (`ciliaq/CILIAQ_v0.2.1.md` §4.2). `[measured]`
- `minimum-basal-body-size` has no effect, so basal bodies smaller than `minimum-cilium-size` are dropped without a
  warning (`ciliaq/CILIAQ_v0.2.1.md` §2 rule 5). `[measured]` `[developer]`
- Basal-body z coordinates are converted with the pixel width instead of the voxel depth: basal-body positions,
  distances to cilia and the matching can be wrong in z, and in z-stacks the intensities around the basal body are
  measured at the wrong height; do not use those columns from z-stacks (`ciliaq/CILIAQ_v0.2.1.md` §5.3).
  `[measured]` `[developer]`
- The basal-body intensity shells (1 µm, 1–2 µm) and the basal-stain orientation radius (1 µm) are 1 and 2 in the
  image's unit, whatever it is (e.g. nm).
- Intensity profiles normalised to the reconstruction channel are 0 (rule 9).
- `Maximum span` is not computed (an empty, unnamed column).
- Undefined values (e.g. averages over no voxels) are written as a replacement character (&#xFFFD;) or `∞`.
- The recorder can write the CiliaQ line twice.
- The run stops for an image without any basal body when basal bodies are segmented, and for a minimum size of 0.
  `[measured]`
- With skeleton results off, the basal-body columns stay empty. `[measured]`
- `# excluded:` also counts the cilia the basal-body filter removes.
- Pixel width ≠ height gives other skeleton results than square pixels; a basal-body channel beyond the channel
  count is read from the last channel; 32-bit images give empty 3D overviews. `[measured]`
- Smaller: very noisy masks are slow, `_CQ_SKL_info.txt` names the wrong file, `_CQl.txt` writes 0 for a missing
  channel, the multi-file manager (closing it, unreadable folders, special characters in patterns).

**Known issues in v0.1.7** (details in `ciliaq/CILIAQ_v0.1.7.md` §5): uncalibrated images (rule 8); every cilium
excluded in 2D with the default exclusion (fixed in v0.2.0); basal-stain radius 3 µm (1 µm since v0.2.0) and in the
image's unit; normalised profiles 0 (rule 9); `Maximum span` a −1 placeholder (negative averages in time-lapse);
&#xFFFD; / `∞` for undefined values; the time-lapse `_CQ_RP_3D` overview keeps only the first frame; GitHub #5 (3D
visualization on macOS) open; **in time-lapses, "Minimum size (A/B)" above 1 filters only the first frame**; a
minimum size of 0 stops the run; non-square pixels, 32-bit 3D overviews, very noisy masks, the multi-file manager
(as in v0.2.1). `[code]`

---

## 6. Keeping this file current

When a new version of any tool is released:

1. add a row to its table, listing only changes that affect results, settings or use;
2. update the recommendation in §1 and in `00_START_HERE.md` §2;
3. add a rule to §2 if results can no longer be combined across the new version;
4. add a new tool file for the version;
5. add one line under the title of the previous tool file: "Superseded by `<new file>` — use
   this file only for <tool> <old version>.";
6. remove fixed issues from the known-issues lists, and update every "so far", "up to vX" and "planned for vX" in
   §2 (rules 8 and 9), here and in the tool files and `agents/AGENT_GUIDE.md`;
7. update the kit version and "Last updated" at the top of every changed file.