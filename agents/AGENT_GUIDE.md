# Agent guide — running CiliaQ tools yourself

**Kit version:** 0.0.1 · **Last updated:** November 2026
**For:** Claude Code and other agents that can run shell commands on the user's computer.
**Covers:** CiliaQ Preparator v0.2.0 · CiliaQ v0.2.1 (static images) · CiliaQ Editor v0.0.4 (not runnable)

Read `00_START_HERE.md` and the tool files first. This file adds only what changes when you can
execute things. All rules of the kit still apply.

---

## 0. Rules for agents

1. **What you may run:** CiliaQ Preparator v0.2.0 by macro, in the GUI or headless
   `[release notes]`; CiliaQ v0.2.1 by macro, for static 2D/3D images only `[release notes]`
   `[developer]`. **Never try to run the CiliaQ Editor** — it is interactive only. `[code]`
2. **Never modify, move or delete the user's original data.** Work on copies (§1).
3. **Nothing leaves the machine.** Do not upload images, results or metadata anywhere.
4. **Ask before** installing or updating Fiji, plugins or update sites, and before long batch runs.
5. **The user judges segmentation quality.** You can compute numbers and make overview images;
   you cannot decide on your own that a mask is correct. Show, explain, and let the user choose.
6. **Follow the tool files' workflow:** few candidates, one change at a time, the same settings for
   the whole dataset, exact option spellings.
7. **Keep a run log** (§4) so every result can be traced to the exact command that produced it.

---

## 1. Data safety

Both plugins write their outputs **next to the input file**, and some options overwrite existing
files. `[code]` Therefore:

- Copy the images you work on into a separate working folder (e.g. `ciliaq_work/<date>/`) and run
  everything there. Never point the plugins at the original folder.
- Prefer the date options in output names when comparing settings (`output=[save as filename +
  suffix 'CQP' + date]` in the Preparator, `include-date` in CiliaQ).
- In folder loops, skip files that are already outputs (`_CQP.tif`, `_CQP_ed.tif`, `_CQ_*`), or a
  re-run produces `_CQP_CQP.tif`. `[release notes]`
- Check free disk space before batch runs; `_CQP.tif` files are at least as large as the input.

---

## 2. Setup checks

Do these once and report the result to the user.

- **Find Fiji and its launcher.** There are two kinds of launcher, and they need different
  headless commands (§3). Some installations have both. Ask the user if you cannot find Fiji.
  - new launcher (current Fiji downloads): `Fiji.app/fiji-linux-x64`,
    `Fiji.app/fiji-windows-x64.exe` (run it through `Fiji.app/fiji.bat` so the call waits for the
    run to finish), `Fiji.app/Contents/MacOS/fiji-macos-arm64` or `-x64`;
  - old launcher: `Fiji.app/ImageJ-linux64`, `Fiji.app/ImageJ-win64.exe`,
    `Fiji.app/Contents/MacOS/ImageJ-macosx`.
- **Check the plugins.** Look in `Fiji.app/plugins/` (and subfolders) for the CiliaQ jars and note
  their versions. The menu command names contain the version (`CiliaQ Preparator v0.2.0`,
  `CiliaQ v0.2.1`); a macro naming a different version does not run. `[release notes]` `[code]`
  This macro prints the installed CiliaQ commands, also headless. Two entries for one tool mean two
  versions are installed. `[measured]`

  ```javascript
  List.setCommands;
  keys = split(List.getList, "\n");
  for (i = 0; i < keys.length; i++) {
    k = substring(keys[i], 0, indexOf(keys[i], "="));
    if (indexOf(k, "CiliaQ") >= 0) print(k);
  }
  ```

- **Hysteresis and Canny 3D need the 3D ImageJ Suite** (look for `mcib3d` jars). Without it, use
  only the plain threshold methods. `[release notes]`
- **One version, one run per Fiji.** Do not run a CiliaQ plugin if two versions of it are
  installed. Do not run several plugin instances in the same Fiji, since they can interfere; to
  parallelize, start several Fiji instances with one run each. `[developer]`
- **Read the image metadata** with a small macro before proposing settings:

  ```javascript
  open("/path/to/ciliaq_work/image.tif");
  getDimensions(w, h, channels, slices, frames);
  getPixelSize(unit, pw, ph, pd);
  print("size " + w + "x" + h + "  channels " + channels + "  slices " + slices + "  frames " + frames);
  print("bit depth " + bitDepth() + "  pixel " + pw + " x " + ph + " x " + pd + " " + unit);
  close("*");
  eval("script", "System.exit(0);");
  ```

Frames > 1 means time-lapse: CiliaQ v0.2.1 is not for it. CiliaQ v0.1.7 handles time-lapse but has
no macro support, so the user must run it in the dialog. `[code]` A unit of "pixel" means the image
is not calibrated — ask the user for pixel size and z step.

---

## 3. Running headless

```bash
# new launcher (fiji-linux-x64, fiji-windows-x64.exe, fiji-macos-…)
<Fiji launcher> --headless /path/to/run.ijm
# old launcher (ImageJ-linux64, ImageJ-win64.exe, ImageJ-macosx)
<Fiji launcher> --headless --console -macro /path/to/run.ijm
```

- **Test the command with a one-line macro first** (`print("hello");`) and check that `hello`
  appears in the output. The wrong form fails quietly: the old launcher without `-macro` does
  nothing and exits with code 0; the new launcher with `-macro` stops with "Macro or script not
  found". `[measured]`
- An argument, e.g. a folder, goes after the macro path in both forms; the macro reads it with
  `getArgument()`. `[measured]`
- **End every macro with `eval("script", "System.exit(0);");`** — the Fiji JVM does not reliably
  exit on its own in headless mode. `[release notes]`
- **Judge success by the output files, not the exit code.** Check that the expected
  `_CQP.tif`/`_CQP.txt` or `_CQ.txt` exist after the run. `[release notes]`
- The ImageJ log, `print` output and macro errors arrive on stdout; save them next to the results.
- Use a timeout. Large 3D stacks, Canny 3D and skeletonization can take minutes per image.
- On Windows, use forward slashes in macro paths.
- A Python wrapper and a folder-batch macro for the Preparator are in the Preparator v0.2.0
  release notes: https://github.com/hansenjn/CiliaQ_Preparator/releases/tag/v0.2.0. They use the
  old launcher form; change the command for a new launcher.

### 3.1 Preparator, one image

```javascript
open("/path/to/ciliaq_work/image.tif");
run("CiliaQ Preparator v0.2.0",
    "process=[active image in FIJI] " +
    "output=[save as filename + suffix 'CQP'] " +
    "number-format=[US (0.00...)] " +
    "segmentation-style=[Keep intensities above threshold (creates a background-removed image)] " +
    "channels=1 c1-nr=3 c1-duplicate " +
    "c1-subtract-bg=5.000 " +
    "c1-method=[RenyiEntropy] " +
    "c1-stack=[apply threshold determined in a maximum-intensity-projection] ");
close("*");
eval("script", "System.exit(0);");
```

Write every key (missing keys silently take defaults) and include `c1-duplicate` if the user may
edit later. See `preparator/PREPARATOR_v0.2.0.md` §3 and §5.

### 3.2 CiliaQ on the Preparator output

```javascript
open("/path/to/ciliaq_work/image_CQP.tif");
run("CiliaQ v0.2.1",
    "number-format=[US (0.00...)] " +
    "reconstruction-channel=3 measure-intensity-A channel-A=4 " +
    "minimum-cilium-size=10 increase-range-cilia " +
    "exclude=[cilia touching x or y or z borders] " +
    "minimum-size-intensity=1 increase-range-intensity " +
    "skeleton gauss-xy=2.000000 gauss-z=0.000000 distance-curvature=1.000000 ");
close("*");
eval("script", "System.exit(0);");
```

- Channel numbers refer to `_CQP.tif`: the segmented channel keeps the original position, the
  unsegmented copy follows it, and later channels shift by one. Read the mapping from `_CQP.txt`.
  `[code]`
- **All CiliaQ flags are off unless written**, including those that are on in the dialog. `[code]`
- Add `save-3D-whole-image` for the 3D overviews (`_CQ_RP_3D.png`, `_CQ_SKL_3D.png`). They are
  written headless too. `[measured]`
- **A missing required key stops the run.** Headless, the output shows `Macro Error: Macro options
  missed definition '<key>…'`, no result files are written, and the exit code is still 0. Only the
  first missing key is named. In a Fiji with a window, an error dialog waits for a click and holds
  up a batch. `[measured]` `[code]`
- This macro and the one in `ciliaq/CILIAQ_v0.2.1.md` §4.1 have been run headless against the
  published v0.2.1. `[measured]`
- See `ciliaq/CILIAQ_v0.2.1.md` §2 and §4.

---

## 4. The experiment loop

1. **Ask and look** (Preparator file §4, Steps 1–2). You can compute facts — size, bit depth,
   calibration, whether the background contains exact zeros — but what the image *shows* comes from
   the user.
2. **Propose up to three candidates**, each differing from another in one setting, and say what
   each tests. Get the user's agreement before running.
3. **Run the Preparator** on copies. Collect each `_CQP.txt` and the log; note the computed
   threshold(s) and every warning (inverted hysteresis thresholds, stack-handling mismatch,
   `unknown value … using …`).
4. **Run CiliaQ with one fixed setting** on all candidates, so differences come from the
   segmentation only.
5. **Summarise the numbers per candidate:** foreground fraction of the segmented channel, number of
   cilia and excluded cilia, cilia without length, median length, branches. Without basal-body
   segmentation, count cilia as result rows in `_CQs.txt`: the header's `of total #:` is always 0
   then in v0.2.1 (CiliaQ file §4.2). Compare candidates with each other; there is no correct
   absolute value (Preparator file §6.3).
6. **Make images for the user to judge** — e.g. a maximum projection of the raw cilia channel next
   to the segmented channel for each candidate, saved as PNG in the working folder — and tell them
   which files to open. Point out where candidates differ.
7. **Let the user decide**, change one thing, repeat. When a setting is chosen, test it on several
   further images before a batch run.

**Run log** (e.g. `ciliaq_work/<date>/RUNLOG.md`), one row per run:

| run | tool + version | input | exact option string | thresholds / key numbers | warnings | outputs |
|---|---|---|---|---|---|---|

---

## 5. Known pitfalls

- **Canny 3D `Custom Value` is not applied in Preparator v0.2.0** (fixed in the next version);
  avoid it. `[code]` `[developer]`
- **Version-specific command names:** after an upgrade, update every `run("… vX.Y.Z", …)`.
- **Repeated runs overwrite outputs** unless the date options are used.
- **Time-lapse images:** do not run CiliaQ v0.2.1 on them. v0.1.7 handles them but can only be run
  by the user in the dialog. `[developer]` `[code]`
- **Do not write `_CQP.txt` files by hand** to pass settings; use the macro keys, or
  `settings=[…]` with a file the Preparator wrote. `[release notes]`

---

## 6. Reporting back

After each round, tell the user:

- what ran — the exact option strings and versions;
- what was compared and what differs, with the numbers;
- which files to open and what to look for;
- what you are unsure about, and what you need from them.

Do not present a setting as validated because the numbers look plausible. Only the user's
inspection of the images can confirm it.

---

## 7. Not covered yet

- CiliaQ v0.2.2 (not yet released). It will be added when it is released.
- A packaged skill or an MCP server; this guide is the basis for both.
