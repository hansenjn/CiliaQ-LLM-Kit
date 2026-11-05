# CiliaQ versions and compatibility

**Kit version:** 0.0.1 · **Last updated:** November 2026
**Sources:** GitHub release notes of all three repositories, source code at the release tags,
statements by the developer.

Read `00_START_HERE.md` first. This file answers: which version should be used, what changed in a
way that affects results, and which results can be combined.

---

## 1. Current recommendation

| tool | use | covered by this kit | scriptable |
|---|---|---|---|
| CiliaQ Preparator | **v0.2.0** | yes | macro and headless |
| CiliaQ Editor | **v0.0.4** | yes | no (interactive only) |
| CiliaQ, static 2D/3D images | **v0.2.1** | yes | macro and headless `[release notes]` |
| CiliaQ, time-lapse images | **v0.1.7** | no | no (dialog only) `[code]` |

`[developer]`

- CiliaQ v0.2.0 and v0.2.1 are labelled beta on their release pages and in a dialog when started
  from the menu. For static images the developer considers v0.2.1 suitable. `[release notes]`
  `[code]` `[developer]`
- The Fiji update site can lag behind the newest releases. A specific version can be installed by
  hand: https://github.com/hansenjn/CiliaQ/wiki/Q&A:-How-to-install-a-specific-version-of-a-CiliaQ-plugin%3F
- For a version this kit does not cover, the assistant should say so and give only
  version-independent advice (`00_START_HERE.md` rule 2).

---

## 2. Rules for combining results

1. **Everything that is compared is processed with the same versions and the same settings** —
   Preparator, Editor and CiliaQ.
2. **CiliaQ skeleton-based results** (length, bending, orientation, profiles, curvature) from
   **v0.1.7 or later must not be pooled with earlier versions.** `[release notes]`
3. **CiliaQ v0.2.0+ with a basal stain** decides the ciliary base within 1 µm of the skeleton ends
   instead of 3 µm. For cilia shorter than about 3 µm, base and tip can therefore be assigned
   differently than in v0.1.7. Check before pooling orientation- or profile-direction results
   across this boundary. `[release notes]`
4. **CiliaQ Editor v0.0.4 edits** can be combined with edits from earlier Editor versions only if
   "Add pixels to mask even if intensity is 0.0…" was **off**. `[release notes]`
5. **Preparator v0.2.0 and v0.1.2** give byte-identical output with the same settings and the same
   ImageJ version. `[release notes]`
6. **ImageJ version:** auto-thresholds can differ slightly between ImageJ versions. To reproduce a
   Preparator run exactly, use the ImageJ version recorded in `_CQP.txt` (recorded since v0.2.0).
   `[release notes]`
7. **Same skeleton blur (gauss filter before skeletonization)** for everything compared. `[wiki]`

---

## 3. CiliaQ Preparator

| version | date | changes that matter for results or use |
|---|---|---|
| **v0.2.0** | 2026-08 | macro options, recorder and headless mode; optional **pre-blur** before background removal; setup checks and warnings (e.g. a reminder whenever Divide by Background is used, inverted hysteresis thresholds, stack-handling mismatch); `_CQP.txt` records the ImageJ version and pre-blur; loading `IJ_IsoData` from settings files fixed (was read as `IsoData`); loading files with more than 3 channels fixed. **Output identical to v0.1.2.** |
| v0.1.2 | 2023-09 | warning for unsaved images; virtual stacks handled; Canny 3D low/high threshold swap fixed — affected only runs combining a custom value for one Canny threshold with an algorithm for the other; loading such old settings triggers a warning |
| v0.1.1 | 2021-03 | 2D hole filling for Canny 3D; "black background" forced on (previously Canny 3D could fail with that ImageJ preference off); stay-awake mode |
| v0.1.0 | 2020-09 | loading settings from `_CQP.txt`; Divide by Background; Gaussian blur; hysteresis threshold; custom threshold |
| v0.0.6 | 2020-06 | all series of multi-series microscopy files processed (before: only the first); settings files from this version onward can be loaded |
| v0.0.3–v0.0.5 | 2020-05 | Canny 3D added (v0.0.3); bug fixes |
| v0.0.2 | 2020-05 | first release |

**Known issues in v0.2.0:** Canny 3D `Custom Value` thresholds are not applied, and the recorder
does not record `canny-low-thr` / `canny-high-thr`. To be fixed in the next version. `[code]`
`[developer]`

All entries `[release notes]` unless tagged otherwise.

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
| v0.2.0 | 2024-10 | macro options, recorder and headless mode; basal-body orientation radius 3 µm → 1 µm; basal-body detection from a segmented channel, matching to cilia and skeleton extension to the basal body (3D static images only); basal-body result columns and counts; intensity profiles moved from `_CQ.txt` to `_CQl.txt`; for 2D images, z-border exclusion switched automatically to x/y borders `[code]`; beta warning dialog |
| **v0.1.7** | 2023-05 | **improved skeletonization and length**, especially for small cilia in 2D — do not mix skeleton results with earlier versions; 3D overviews of static images saved as `.png` |
| v0.1.4 | 2020-12 | `_CQl.txt` output with tangent vectors and curvature (new setting: reference distance); loading settings from `_CQ.txt`; two-step dialog; results compatible with earlier versions |
| v0.1.0–v0.1.3 | 2020-05 to 2020-07 | "colocalized on centreline" parameters (v0.1.0); menu entry `Plugins ▸ CiliaQ` (v0.1.1); 3D visualization fixes |
| v0.0.5–v0.0.10 | 2019-12 to 2020-04 | first release (v0.0.5); "increase range" connectivity, integrated and centreline intensity, thresholds in the output (v0.0.6); pattern-based file selection (v0.0.10); results compatible within v0.0.6–v0.0.10 |

All entries `[release notes]` unless tagged otherwise.

**Known issue in v0.2.1:** without basal-body segmentation, the `_CQ.txt` header shows
`of total #: 0`. Count the result rows instead (`ciliaq/CILIAQ_v0.2.1.md` §4.2). `[code]`
`[measured]`

---

## 6. Keeping this file current

When a new version of any tool is released:

1. add a row to its table, listing only changes that affect results, settings or use;
2. update the recommendation in §1 and in `00_START_HERE.md` §2;
3. add a rule to §2 if results can no longer be combined across the new version;
4. add a new tool file for the version.
5. add one line under the title of the previous tool file: "Superseded by `<new file>` — use
   this file only for <tool> <old version>."