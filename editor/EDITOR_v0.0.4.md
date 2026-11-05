# CiliaQ Editor v0.0.4

**Kit version:** 0.0.1 · **Last updated:** November 2026
**Verified against:** git tag `v0.0.4` of https://github.com/hansenjn/CiliaQ_Editor
(`CiliaQEdMain.java`, `EditingDialog.java`)
**Sources:** source code, release notes v0.0.1–v0.0.4, CiliaQ wiki and SOP

Read `00_START_HERE.md` first. Its rules apply here too.

---

## 0. Instructions to the assistant (Editor-specific)

1. **The Editor is interactive only.** It has no macro options, no recorder support and no headless
   mode. Never offer a macro line or a batch run for it, and agents must never try to run it. Guide
   the person at the screen step by step. `[code]`
2. **Check that editing is the right fix.** If the same error appears in many cilia or many images,
   improve the segmentation instead (`preparator/PREPARATOR_v0.2.0.md`), or use a CiliaQ setting
   (`ciliaq/CILIAQ_v0.2.1.md`). Editing is for the remaining, occasional errors.
3. **Editing changes data.** Help the user define one criterion for what is edited, apply it to all
   images of the dataset, and report it in the methods (§6).

---

## 1. What the Editor does

The Editor lets the user correct the segmented channel of a `_CQP.tif` by hand: draw a selection,
then add the voxels inside it to the mask or remove them. Added voxels take their values from the
unsegmented copy of the same channel. CiliaQ then analyses the edited image. `[code]`

**Requirements** `[code]` `[wiki]`

- An open image that was **opened from disk** (normally `<name>_CQP.tif`). Outputs are written
  next to it.
- It must contain the segmented channel **and an unsegmented copy of the same channel** — the
  Preparator adds that copy only if "Include also an unsegmented copy of the channel" was ticked
  (macro: `c1-duplicate`). Without it, the image cannot be edited; re-run the Preparator.
- For "keep intensities" masks, every non-zero voxel of the segmented channel must equal the
  unsegmented channel at the same position; otherwise the Editor stops with a message that the
  intensities do not match. Binary masks (only 0 and the maximum value) are recognised
  automatically and skip this check. `[code]` `[release notes]`
- The unsegmented copy is also what lets the user see where cilia actually are: which cilia were
  missed, and which structures were falsely detected as cilia. `[developer]`

---

## 2. Settings

`Plugins ▸ CiliaQ ▸ CiliaQ Editor v0.0.4` opens one dialog. `[code]`

| dialog field | default | what to enter |
|---|---|---|
| `Channel Nr of channel that is segmented / semi-binarized / binarized` | 2 | the segmented channel **in `_CQP.tif`** |
| `Channel Nr of an unmodified/unsegmented copy of the same channel` | 3 | its unsegmented copy **in `_CQP.tif`** |
| `Add pixels to mask even if intensity is 0.0 in the unsegmented copy of the channel` | off | see below |
| `Output image name:` | `save as filename + suffix '_ed'` | also `save as filename + suffix 'ed' + date`, `overwrite input file` |

**Finding the channel numbers.** The defaults (2 and 3) rarely match. The channel list at the end
of `_CQP.txt` says which output channel is the segmented one and which is the copy. The Preparator
puts the segmented channel at the original position and the copy directly after it — e.g. cilia in
raw channel 3 → segmented channel 3, copy in channel 4. `[code]`

**"Add pixels to mask even if intensity is 0.0…"** `[code]` `[release notes]`

- **Off (default):** voxels whose value in the unsegmented copy is 0 are never added, in binary
  and in keep-intensities masks alike.
- **On:** such voxels are added too — useful for images with a true zero background (noise-free
  detectors, an offset). In a keep-intensities mask they are written as **1.0**, although their
  real value was 0, so CiliaQ's intensity parameters for the segmented (reconstruction) channel
  become minimally falsified; intensity parameters of other channels are unaffected.
- **Leave it off if the data must be combined with data edited by earlier Editor versions.**
  `[release notes]`

**Output name.** If a file with the output name already exists, it is **overwritten**. The date
option prevents that. Do not use `overwrite input file` unless the original `_CQP.tif` is backed
up. `[code]`

---

## 3. Editing

### 3.1 Display

When editing starts, the segmented channel is shown in **magenta** with display range 0–5, so
every detected voxel appears bright; the unsegmented copy is shown in **green** with display range
0 to its 99.9th percentile. Other channels are hidden. Detected signal therefore appears white,
missed signal green, and mask without signal magenta. Sliders in the Editor window adjust the
display; original colours are restored when editing ends. `[code]` `[release notes]`

### 3.2 Actions

Draw a selection with any ImageJ selection tool (freehand, polygon, rectangle, …), then:

| button | key | effect |
|---|---|---|
| `add selection` | **F1** | add the voxels inside the selection to the mask, **in the current slice** |
| `remove selection` | **F2** | remove them from the mask, in the current slice |
| `add sel. to all slices` | **F3** | add inside the selection in **every** z-slice of the current time point — most likely never appropriate, since it creates a block-shaped object extending across all planes, which is not what a cilium looks like `[developer]` |
| `remove sel. from all slices` | **F4** | remove inside the selection in every z-slice — the quick way to delete a false-positive object |
| `undo last editing` | **Ctrl+Z** | undo the last action (an all-slices action is undone as one step) |
| `finish analysis & save editings` | | save and close (§5) |
| `cancel analysis & discard editings` | | close without saving (asks for confirmation if edits were made) |

Edits apply to the current time point only. `[code]`

### 3.3 What "add" really does

**Add copies every voxel inside the selection** from the unsegmented copy into the mask — not just
the cilium. A generous selection around a gap therefore also adds the dim background around it.
`[code]`

- Draw **tight** selections that follow the missing part of the cilium.
- **Avoid F3** for cilia: it adds the whole selection in *every* slice, including slices where
  that area contains only background (§3.2).

### 3.4 Typical corrections

| problem | how | watch out for |
|---|---|---|
| a false-positive object (debris, cell body, mitotic spindle, …) | selection around it, **F4** | other cilia under the selection in other slices would be removed too; check the slices `[code]` |
| two touching cilia fused into one object | remove voxels along the contact, **F2**, in **every slice** where they touch `[wiki]` | CiliaQ connects voxels in 3D, and with its "increase range" option also diagonally; a cut that leaves voxels touching at a corner keeps the objects joined. Make the cut at least 2 px wide `[developer]` |
| a cilium broken into pieces | tight selection over the gap, **F1**, in the slices where the gap is | gap voxels with value 0 are only added with the zero-pixel option on |
| a dim part missing | tight selection over the missing part, **F1** | do not add surrounding background |

After editing, look through all slices once more before saving.

---

## 4. If editing seems wrong

| what happens | likely reason | fix |
|---|---|---|
| "No image open." | no image, or the Editor was started without one | open `_CQP.tif` first |
| channel number "out of range" | number larger than the channel count | read the channel list in `_CQP.txt` |
| "Select different channels…" | both fields have the same number | enter the segmented channel and its copy |
| editing stops: intensities do not match | wrong pair of channels, or the copy was not produced by the Preparator, or the segmented channel was modified | choose the right pair; re-run the Preparator with the unsegmented copy `[code]` |
| an error right after the dialog | the image was not opened from a file (e.g. a duplicate or projection made in Fiji) | save the image, reopen it from disk, start again `[code]` |
| nothing is added in a gap | the gap voxels are 0 in the copy | switch on the zero-pixel option (and note the intensity caveat in §2) |
| the image window was closed by accident | — | the Editor offers to restore the image with all edits |

---

## 5. Output

On `finish analysis & save editings`, next to the input image: `[code]`

- `<name>_ed.tif` — the edited image (e.g. `image_CQP_ed.tif`), which goes to CiliaQ;
- `<name>_ed.txt` — date, image name, the channel numbers, the zero-pixel setting, the number of
  edits, and for each edit whether it added or removed voxels, its slice and frame, plus
  `Plug-in version`;
- a folder `<name>_ed/` with every selection as an ImageJ ROI file, named
  `<nr>_s<slice>_t<frame>_add.roi` or `…_rem.roi`.

(With the date option, `_ed_<yyMMdd_HHmmss>` replaces `_ed`.)

Together these document every manual change. Keep them with the data. `[release notes]`

---

## 6. Editing as part of the method

- **Use one written criterion** for what counts as an error and how it is corrected, and apply it
  to all images and conditions alike.
- **Edit without knowing the condition** where possible, to avoid unconscious bias.
- **Report it:** that masks were corrected manually, the criterion, and how much was edited — the
  `_ed.txt` files give the number of edits per image.
- **Prefer reproducible alternatives where they work:** better Preparator settings; CiliaQ's
  minimum object size and border exclusion; excluding cilium IDs from the results afterwards
  `[wiki]`; or **DeleteROI**, a plugin from the Pazour lab for removing non-ciliary objects from
  CiliaQ output (`00_START_HERE.md` §7). `[wiki]`

---

## 7. Not covered here

- Choosing segmentation settings: `preparator/PREPARATOR_v0.2.0.md`.
- CiliaQ settings and results: `ciliaq/CILIAQ_v0.2.1.md`.
- Editor versions other than v0.0.4. For v0.0.3 and earlier: no F3/F4, no zero-pixel option,
  no automatic display setup. `[release notes]`
