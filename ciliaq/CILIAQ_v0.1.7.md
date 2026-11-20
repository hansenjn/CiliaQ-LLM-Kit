# CiliaQ v0.1.7

**Kit version:** 0.0.2 · **Last updated:** November 2026
**Verified against:** git tag `v0.1.7` of https://github.com/hansenjn/CiliaQ (`CiliaQMain.java`)
**Sources:** source code, release notes v0.0.5–v0.2.1, GitHub issues, test runs (November 2026), CiliaQ wiki, CiliaQ SOP, CiliaQ paper

Read `00_START_HERE.md` first. Its rules apply here too. v0.1.7 is the version for **time-lapse images** (v0.2.1 is
not for them). For static 2D and 3D images use `ciliaq/CILIAQ_v0.2.1.md`. The settings that v0.1.7 shares with v0.2.1 are
described in more detail there; this file lists what differs.

---

## 0. Instructions to the assistant

1. **Dialog only.** v0.1.7 has no macro options, no recorder support and no headless mode: give settings as a table
   of dialog field → value. `[code]`
2. **No basal-body detection.** A basal stain can only orient cilia ("basal stain present"); there is no
   segmented-basal-body mode and no basal-body result columns. `[code]`
3. **Calibrate the image first.** An uncalibrated image (unit "pixel", pixel size 1) gives wrong skeleton results (§5);
   set the real pixel size and z step. `[code]` `[measured]`
4. **2D images:** the default exclusion `cilia touching x or y or z borders` excludes every cilium in a 2D image
   (every object touches the z borders). Choose `cilia touching x or y borders`. `[code]` `[wiki]`
5. **Skeleton results of v0.1.7 must never be pooled with versions before v0.1.7** (§4). `[release notes]`

---

## 1. Settings (two dialogs)

First dialog: `process` (active image / multi-task manager / all open images), calibration (`manually calibrate
image`, pixel size, voxel depth, unit), time interval and unit, `Preferences` (enter, or load from a `_CQ.txt` of
v0.0.6 or later), output options (date in file names, per-cilium images, per-cilium 3D, whole-image 3D, number
format). `[code]`

Second dialog: `measure intensity A` / `B`, **`basal stain present`** (a checkbox, not the three-way choice of
v0.2.x), the four channel numbers, `minimum cilium size [voxel]` (10), `Increase range for connecting cilia`, the
four exclusion options (as in v0.2.1), `minimum size of intensity regions`, `Increase range for connecting intensity
regions`, `Determine skeleton-based results`, gauss filter XY / Z sigma before skeletonization, `Reference distance
for tangents and curvature`. `[code]`

Defaults and meaning are as in `ciliaq/CILIAQ_v0.2.1.md` §2.1–2.3 and §3, except:
- **Basal-stain orientation** looks for the stain within **3 µm** of the skeleton ends (v0.2.0 on: 1 µm);
  strictly 3 in the image's unit (§5). For cilia shorter than about 3 µm the base is often assigned to the wrong end. `[release notes]`
- **No automatic 2D switch** of the exclusion option (§0 item 4).

---

## 2. Time-lapse (t > 1) `[code]`

- **Objects are connected across frames by the fill itself:** a cilium voxel connects to the voxel at the same x, y,
  z in the previous and next frame. A cilium is one object as long as it overlaps itself from frame to frame. A
  cilium missing from one frame becomes two objects; touching cilia that move into each other's positions merge.
- **Minimum size per frame:** an object is removed as a whole when **any** of its frames has fewer voxels than the
  minimum cilium size. One small frame (e.g. the cilium going out of focus) removes the whole track. "Minimum size (A/B)" works
  differently: it filters only the first frame (§5).
- **Border exclusion** applies over all frames: a cilium that touches a border in any frame is excluded.
- **Outputs:** `_CQ.txt` / `_CQs.txt` hold **time-averaged** results per cilium ("(time-averaged)" in the column
  names); a subfolder `<name>_CQ…/` holds one kinetics file per cilium (`CQ_C<n>.txt`, `CQ_C<n>s.txt`) with the values
  per frame; `_CQl.txt` the skeleton points with their frame.

---

## 3. Output files (static images)

As in `ciliaq/CILIAQ_v0.2.1.md` §4.2, with these differences: intensity profiles along the skeleton are still columns in
`_CQ.txt` (moved to `_CQl.txt` in v0.2.0); no basal-body columns or counts; 3D overviews of static images as `.png`
(new in v0.1.7). `of total #:` in the header is correct in v0.1.7 (the 0 of v0.2.0/v0.2.1 came with v0.2.0).
`[release notes]` `[code]`

---

## 4. What changed in v0.1.7 and why it matters

`[release notes]`

- **Skeletonization of 2D images is done in 2D**, and the small image made for each cilium gets enough margin for
  the blur. Before v0.1.7, 2D cilia were skeletonized in a 3D image with added empty planes, and the blur could reach
  the edge of that small image. That added small biases: a slightly longer skeleton, extra branches or shortcuts —
  a few percent in long cilia, up to 100–200 % in very short cilia.
- **Consequence:** never pool skeleton-based results (length, bending index, orientation, profiles, curvature) of v0.1.7 or later
  with earlier versions.
- Less frequent garbage collection (faster).

---

## 5. Known issues `[code]`

Found in the v0.1.7 source, the release notes of v0.2.0 and v0.2.1, the GitHub issues and test runs (2026-11).

| issue | effect | status |
|---|---|---|
| **uncalibrated images** (unit "pixel", pixel size 1; GitHub #15) | **skeleton lengths 3× too long**; the first skeleton point at three times its position, so the base mark in `_CQ_SKL.tif` lies away from the skeleton, and orientation, bending index, profiles, arc lengths and the basal-stain orientation are wrong. Volume, surface and intensities are right. **Calibrate the image first** with its real pixel size and z step (any calibration avoids the bug, even 1 µm per pixel) `[measured]` | every version up to v0.2.1; fix planned for v0.2.2 |
| 2D images: the default exclusion excludes every cilium (GitHub #12) | no results at all for 2D images | fixed in v0.2.0 (automatic switch) |
| basal-stain orientation searches **3 µm** around the skeleton ends (GitHub #16) | for cilia shorter than about 3 µm, base and tip are often swapped | 1 µm since v0.2.0 |
| that radius is "3" in the image's unit, not µm | in a nm-calibrated image it is 3 nm | fix planned for v0.2.2 |
| intensity profiles normalised to the reconstruction channel are never computed (a comparison with NaN is always false) | the normalised profile columns are 0 | every version up to v0.2.1; fix planned for v0.2.2 |
| **`Maximum span` is not implemented**: a placeholder | every static cilium −1; the time-averaged value in time-lapse is a negative average (e.g. −1.17) | v0.2.0/v0.2.1 leave the static column empty and unnamed; implementation planned for v0.2.2 |
| undefined values written as a replacement character (&#xFFFD;) or `∞` | e.g. "average of 10 % highest voxels" of objects under 5 voxels (0 / 0) | fix planned for v0.2.2 |
| the time-lapse `_CQ_RP_3D` overview is saved as PNG | only the first frame is kept | fix planned for v0.2.2 |
| an image that was never saved stops the run with an unclear error | — | explained since v0.2.0 |
| 3D visualizations on macOS can stop with a `VolatileOffscreenBuffer` error, at random (GitHub #5) | rerun the image | open |
| **time-lapse: "Minimum size (A/B)" above 1 filters only the first frame** (found 2026-11-19; the same code measured in v0.2.1) | in the later frames the small A/B particles stay, so the A/B results of time-lapses (e.g. colocalized volumes) include them; with the setting at 1 (off) nothing changes | every version up to v0.2.1; fix planned for v0.2.2 |
| a minimum cilium size of 0 | the run stops ("Illegal Capacity: -1"); use 1 or more | every version up to v0.2.1; fix planned for v0.2.2 |
| pixel width ≠ height | CiliaQ notes that it uses the x calibration for both but keeps the height, so skeleton results differ from the same image with square pixels; make the pixels square first | every version up to v0.2.1; fix planned for v0.2.2 |
| 32-bit images | the 3D overviews show no objects (IDs and scale bar only); measurements not affected | every version up to v0.2.1; fix planned for v0.2.2 |
| masks with very many tiny pieces (very noisy masks) | slow: the time grows with the number of pieces times the foreground voxels | every version up to v0.2.1; fix planned for v0.2.2 |
| `_CQ_SKL_info.txt` | names the skeleton image `/<name>_CQ_SKL_Skl.tif`; the file is `<name>_CQ_SKL.tif` | fix planned for v0.2.2 |
| `_CQl.txt` with intensity A or B on, for an image that lacks that channel | 0 in its profile columns instead of empty cells | fix planned for v0.2.2 |
| the multi-file manager | closing its window leaves CiliaQ waiting (start it again); a folder without read access, or `(`, `[`, `+` in a plain-text pattern, stops the pattern search; a file without an extension stops the run; after an image that fails, the progress window lists the next image as done too early | fix planned for v0.2.2 |

Also:
- Without basal-body segmentation there are no basal-body counts; the ciliation rate needs a separate cell count.
- The per-frame minimum size can remove whole tracks (§2).
- No macro or headless use (§0 item 1).
