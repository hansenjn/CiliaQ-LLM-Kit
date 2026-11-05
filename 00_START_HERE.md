# CiliaQ LLM Kit — START HERE

**Kit version:** 0.0.1 · **Last updated:** November 2026 · **Maintainer:** Jan N. Hansen
**Covers:** CiliaQ Preparator v0.2.0 · CiliaQ Editor v0.0.4 · CiliaQ v0.2.1
**Latest version of this kit:** https://github.com/hansenjn/CiliaQ-LLM-Kit

**For people:** give this file to your AI assistant, together with the file for the tool you need
help with (see §1). You never need to upload your images, but it can be helpful.
More in `README.md`.

> **IMPORTANT — please read before use.** This kit is new and under active development, and it
> may change substantially in the coming weeks. Please report errors or problems (see `README.md`).
>
> - **AI assistants can be wrong.** They may misread these documents, suggest settings that do not
>   suit your data, or describe results they have not seen. Check every suggestion against your own
>   images. You remain responsible for your analysis and for the conclusions you draw from it.
> - **Agents run real commands on your computer.** Let them work only on copies of your data,
>   ideally in a sandbox or a separate folder, and review what they intend to do before they do it.
> - **Sharing data is your decision.** You never need to share images. If you share images or files
>   with an online assistant, that provider's terms apply to them.
> - **No warranty.** This kit is provided "as is", without warranty of any kind, under the
>   [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) license (see `LICENSE`). The authors
>   accept no liability for errors, data loss, or other harm resulting from its use.

---

## 0. Instructions to the assistant

Read this whole file before answering. Then read the tool file that §1 points you to. Follow these
rules for the entire conversation.

1. **Only describe settings that exist in the user's version.** Each tool file lists them verbatim.
   Never invent a parameter, never borrow options from other software, and never describe options
   that exist only in unreleased code (the GitHub `master` branch can contain features that no
   release has).
2. **Establish the versions first** (§2). If the kit has no file for the user's version, say so.
   Give only advice that this file marks as version-independent, and label it as such.
3. **Understand the data and the question before proposing anything.** Ask about: microscope type
   (confocal or widefield/epifluorescence), 2D, 3D or time-lapse, pixel size and z step, which
   channel shows cilia, and **what the user wants to measure** (§4). The best settings depend on it.
4. **Cheap tests before expensive ones.** Much of the segmentation can be explored live in Fiji
   with preview checkboxes in a few minutes. Use full plugin runs to compare finished candidates,
   not to search blindly. Ask the user what they see in the image if you need more visual
   understanding.
5. **Change one thing at a time,** and compare runs on the same image. Propose at most three
   candidates at once, and say what each one tests.
6. **One dataset, one setting.** All images that will be compared must be processed with the same
   settings and the same tool versions. Help the user choose settings that work across the dataset,
   not just on one image, and suggest checking them on several images.
7. **You cannot see the image unless the user shares one.** Never claim a result you have not seen.
   Describe what to look at and let the user judge. Screenshots can help, but sharing one is the
   user's choice; never require it, since the data may be unpublished.
8. **Name trade-offs, then help the user stop.** Segmentation is rarely perfect. Say which bias a
   setting accepts (missing dim parts vs. including background, separating vs. fusing neighbours).
   "Good automatic settings plus a little manual editing" is a legitimate end point, and so is
   "good automatic settings plus post-filtering the result files to remove erroneous cilia". Ask
   the user what level of error is acceptable for their specific research question.
9. **Manual editing changes data.** If the user edits masks, recommend one consistent criterion
   and reporting it in the methods.
10. **Be honest about certainty.** Claims in this kit carry source tags (below). Pass on the
    uncertainty of `[guess]` claims. Point the user to the wiki pages listed in §7 when they want
    to read more.
11. **If you can run commands** (Claude Code or another agent with a shell), read
    `agents/AGENT_GUIDE.md` before running anything.

### Source tags used in this kit

| tag | meaning |
|---|---|
| `[code]` | read from the source code at the release tag of that version |
| `[release notes]` | stated in the official GitHub release notes |
| `[wiki]` | stated in the CiliaQ wiki or SOP |
| `[paper]` | stated in a CiliaQ publication |
| `[developer]` | stated by the CiliaQ developer, not yet written elsewhere |
| `[measured]` | measured on real data by the kit authors |
| `[measured, internal data]` | measured by the kit authors on data that is not public |
| `[guess]` | a reasonable starting point that has not been validated — test it |

---

## 1. Which file to read

| the user wants to… | read |
|---|---|
| segment cilia, choose preprocessing or a threshold method, fix a bad mask automatically | `preparator/PREPARATOR_v0.2.0.md` and `ciliaq/CILIAQ_v0.2.1.md` |
| correct a segmentation by hand | `editor/EDITOR_v0.0.4.md` |
| choose CiliaQ settings, understand or check CiliaQ results | `preparator/PREPARATOR_v0.2.0.md` and `ciliaq/CILIAQ_v0.2.1.md` |
| fix a problem seen in the results without knowing which tool causes it | §5 of this file, then the tool file it names |
| know which versions to use, whether results can be combined, or reproduce an old analysis | `VERSIONS.md` |
| run the tools automatically (agents) | `agents/AGENT_GUIDE.md` |
| set up the microscope or decide how to image | §6 of this file |
| install CiliaQ | the wiki installation guide (§7) |

---

## 2. Versions

### How to find out which version is installed

- **From the menu:** `Plugins ▸ CiliaQ` lists each plugin with its version in the name, e.g.
  `CiliaQ Preparator v0.2.0` or `CiliaQ v0.2.1`. `[developer]`
- **From the output files:** the Preparator's `_CQP.txt` records `Plug-in version` and, since
  v0.2.0, `ImageJ version`. `[release notes]`; CiliaQ's `_CQ.txt` records `Plug-in version`.
  `[developer]`

Note that the user may have installed CiliaQ through the Fiji update manager, which can lag behind
the newest or beta versions. In that case, a manual installation of a newer or beta version can be
advised if it is promising for improving results and suitable for the user's question. `[developer]`

### Current versions

| tool | version this kit covers | status | what to know |
|---|---|---|---|
| CiliaQ Preparator | v0.2.0 (Aug 2026) | stable | With the same settings and ImageJ version, output is byte-identical to v0.1.2. v0.2.0 adds pre-blur, macro/headless use and setup checks. `[release notes]` |
| CiliaQ Editor | v0.0.4 (Oct 2025) | stable | Interactive only. Edits can be combined with data edited in earlier versions only if the option to add pixels with intensity 0 is **off**. `[release notes]` |
| CiliaQ | v0.2.1 (Oct 2024) | use for **static 2D and 3D images** | Not for time-lapse images. For time-lapse, use v0.1.7. `[developer]` |

**Results that must not be combined:** cilium length and other skeleton-based results from CiliaQ
v0.1.7 or later must not be pooled with results from earlier versions, because v0.1.7 changed how
the skeleton is computed. `[release notes]` Full history: `VERSIONS.md`.

---

## 3. How the pipeline fits together

```
raw image (.tif, .czi, .nd2, .lif, … — opened via Fiji/Bio-Formats, no conversion needed)
  │
  ▼  CiliaQ Preparator     decides which voxels are cilium; writes a new image
  │      → <name>_CQP.tif  original channels + segmented channel(s)
  │      → <name>_CQP.txt  settings used, threshold computed, channel map, versions
  │
  ▼  CiliaQ Editor         (optional) inspect the mask, add or remove voxels by hand
  │      → <name>_CQP_ed.tif
  │
  ▼  CiliaQ                builds cilium objects from the mask, measures them
         → <name>_CQ.txt   settings + results table (one row per cilium)
         → <name>_CQs.txt  results table only; its column header is in <name>_CQ.txt
         → <name>_CQl.txt  per-point skeleton data (coordinates, profiles, curvature)
         → <name>_CQ_RP.tif, <name>_CQ_SKL.tif, optional 3D visualizations
```

Facts that connect the steps. Each tool file explains them in detail.

- **The image must be saved on disk** before the Preparator runs. A projection or crop made in
  Fiji has to be saved first. Output files are written next to the input. `[wiki]`
- **The Preparator decides the mask on a preprocessed copy but writes original intensities.**
  Blur and background removal change which voxels count as cilium, not the intensities saved —
  with the default "keep intensities" output. `[code]`
- **Voxels whose original value is exactly 0 stay 0 in "keep intensities" output,** so CiliaQ
  treats them as background even when the mask included them. For images with a true zero
  background, the binary output style avoids this. `[code]` `[wiki]`
- **The Editor needs an unsegmented copy of the segmented channel.** The Preparator adds it only
  if "include an unsegmented copy" is ticked. Without it, the mask cannot be edited later. `[wiki]`
- **Channel numbers change.** In CiliaQ, channels are numbered as they are in `_CQP.tif`, not as in
  the raw file. `_CQP.txt` lists which raw channel ended up where. `[wiki]`
- **CiliaQ settings can compensate for some segmentation problems** — small gaps (connectivity
  option), debris (minimum size), rough masks (blur before skeletonization). Before tuning the
  Preparator further, check §5: the fix may belong in CiliaQ.
- **What can be automated:** the Preparator v0.2.0 by macro or headless; CiliaQ v0.2.x by macro or
  headless `[release notes]`; the Editor not at all — it is only for direct user interaction.

---

## 4. What the user wants to measure decides what "good" means

Ask this early. The same mask can be fine for one question and wrong for another.

| measurement | sensitive to | consequence for settings |
|---|---|---|
| **cilium length** (skeleton-based) | gaps and fused neighbours; much less to mask width | prioritise continuous, separated cilia; a slightly wide mask is acceptable. Length follows the skeleton, which is nearly the same for wide and narrow masks. Check skeletons visually. `[wiki]` |
| **intensity in the cilium** | mask width — a wide mask includes background and dilutes averages | keep the mask close to the signal; prefer centreline-based or top-10 % intensity parameters when masks are wide. Check skeletons visually for centreline accuracy. `[wiki]` |
| **number of cilia, fraction of ciliated cells** | fragmentation (one cilium counted as several) and false positives | prioritise one object per cilium and removing debris. |
| **shape, bending, orientation** | rough mask surfaces, side branches | smoother masks; check skeletons visually. |

---

## 5. Triage: what you see → where to fix it

| symptom | likely cause | where to look |
|---|---|---|
| 2D image: no cilia in the results at all | CiliaQ v0.1.7 and earlier exclude objects touching the z borders by default, and in 2D every object does (v0.2.x switches this automatically) | CiliaQ: change the exclusion setting to x/y borders only `[wiki]` `[code]` |
| some cilia shown with grey numbers and missing from the results | they touch the image borders and were excluded | CiliaQ exclusion setting; or image a wider z range `[wiki]` |
| one cilium reported as several objects | gaps in the staining or in the mask | Preparator (blur, hysteresis); CiliaQ connectivity option; Editor `[wiki]` |
| neighbouring cilia reported as one long cilium | the mask bridges them | Preparator (less blur; Canny 3D fuses more readily); Editor to cut them apart `[wiki]` |
| many skeleton branches | rough mask surface from noise | Preparator (small blur); CiliaQ blur before skeletonization. Branches are harmless if the main skeleton follows the cilium `[wiki]` |
| cilia with no length (0 skeletons) | the object is roughly spherical: a cross-section, a stub, or a cilium cut by the stack edge | usually not an error — see the CiliaQ file for how to handle them `[wiki]` |
| masks much wider than the cilia, skeletons with kinks (widefield images) | out-of-focus blur included in the mask | Preparator: Subtract Background with a small radius `[wiki]` |
| many small debris objects | background kept in the mask | CiliaQ minimum size; Preparator background removal; Editor `[wiki]` |
| most cilia missing | a very bright particle distorting the threshold, an unsuitable method, or weak signal | Preparator; check for aggregates and dirt; sometimes re-imaging, or cropping the image to exclude the bright particle `[wiki]` `[developer]` |
| hollow cilium masks near the top or bottom of the stack (Canny 3D) | edge detection at the z border | CiliaQ blur before skeletonization fills them `[wiki]` |
| nothing detected after Divide by Background | the background is (near) zero | Preparator: use Subtract Background instead `[code]` |

---

## 6. When the image itself is the limit

No setting recovers information the image does not contain. Tell the user when imaging is the
bottleneck. `[wiki]`

- Analysis is easiest and most accurate at pixel sizes of about **0.07–0.15 µm** and z steps of
  about **0.5–0.7 µm**. With large pixels (e.g. 0.5 µm), a cilium may be one pixel wide and breaks
  into fragments wherever it is dim. In this case, upscaling the image with interpolation can be an
  option to create an image that allows better connectivity. `[developer]`
- **3D stacks give better results than maximum projections**, because cilia are traced in 3D.
  Projections are worth considering only when the signal-to-noise ratio is very poor. Consider that
  standing cilia (pointing along z) will be detected wrongly in projections. `[developer]`
- **Widefield (epifluorescence) images** often contain out-of-focus blur that ends up in the mask
  unless it is removed with background subtraction. Strong length changes can still be measured. `[developer]` `[wiki]`

---

## 7. Further reading

- CiliaQ, the main repository (releases, source code, issues): https://github.com/hansenjn/CiliaQ
  - CiliaQ Preparator: https://github.com/hansenjn/CiliaQ_Preparator
  - CiliaQ Editor: https://github.com/hansenjn/CiliaQ_Editor
- CiliaQ wiki (tutorials and Q&A): https://github.com/hansenjn/CiliaQ/wiki
  - Installation: https://github.com/hansenjn/CiliaQ/wiki/Installing-CiliaQ
  - Finding good Preparator settings, worked examples 1–3: linked from the wiki home page
- Paper: Hansen et al., *Eur. Phys. J. E* 44, 18 (2021). https://doi.org/10.1140/epje/s10189-021-00031-y
- Protocol and CiliaQ Explorer (quality control and statistics on CiliaQ output): Burgdorf et al.,
  *STAR Protocols* (2025). https://doi.org/10.1016/j.xpro.2025.104128
- DeleteROI, for removing non-ciliary objects from CiliaQ output: Anuszczyk et al.,
  *microPublication Biology* (2025). https://dx.doi.org/10.17912/micropub.biology.001670
- Questions and discussion: https://github.com/hansenjn/CiliaQ/discussions
