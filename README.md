# CiliaQ LLM Kit

Documents that let an AI assistant give you **specific, testable help with CiliaQ** — choosing
segmentation settings for your images, correcting masks, choosing CiliaQ settings, and making sense
of the results. It works with any assistant (Claude, ChatGPT, Gemini, a local model, …) and with
coding agents such as Claude Code.

> **Work in progress (November 2026).** This kit is new and will change a lot in the coming weeks.
> AI assistants can be wrong: check their advice against your own images — you remain responsible
> for your analysis. Agents run real commands, so let them work only on copies of your data, ideally
> in a sandbox. The kit comes without warranty and the authors accept no liability for its use; the
> full note is at the top of `00_START_HERE.md`.

## This repository and CiliaQ

This repository holds only the LLM kit. CiliaQ itself — the plugins, their releases, source code
and issue trackers — is developed in its own repositories:

| tool | repository |
|---|---|
| **CiliaQ** (main repository) | https://github.com/hansenjn/CiliaQ |
| CiliaQ Preparator | https://github.com/hansenjn/CiliaQ_Preparator |
| CiliaQ Editor | https://github.com/hansenjn/CiliaQ_Editor |

Tutorials, Q&A and the installation guide are in the
[CiliaQ wiki](https://github.com/hansenjn/CiliaQ/wiki).
The kit has its own version number, given at the top of each file; it is not a plugin version.

## Quick start

**In a normal chat**

1. Attach `00_START_HERE.md` and the file for the tool you need help with (table below). Or link to the kit
   if your LLM or agent can read directly from GitHub.
2. Describe your images and what you want to measure.
3. Answer the assistant's questions, try what it suggests in Fiji, and report back what you see.

**As a persistent assistant** — add the whole folder as knowledge (a Claude Project, a custom GPT,
or similar), so you do not have to attach files every time.

**With an agent that can run commands** (Claude Code or similar) — download or clone this
repository, point the agent at the folder and tell it to start with `00_START_HERE.md`. It will
read `agents/AGENT_GUIDE.md` before running anything.

## Get the files

- **One file:** open it in the file list above and click "Download raw file", or save its raw
  address, e.g. https://raw.githubusercontent.com/hansenjn/CiliaQ-LLM-Kit/main/00_START_HERE.md
- **The whole kit:** green "Code" button → "Download ZIP", or
  `git clone https://github.com/hansenjn/CiliaQ-LLM-Kit.git`.
- **Assistants that read the web:** give them the raw address of `00_START_HERE.md`. The other
  files have the same address with their path, e.g. `…/main/preparator/PREPARATOR_v0.2.0.md`.
- **Use the latest version.** The kit changes often; a copy saved weeks ago may be outdated.

## Which file to attach

| you want to… | attach `00_START_HERE.md` plus |
|---|---|
| segment cilia or improve a segmentation | `preparator/PREPARATOR_v0.2.0.md` and `ciliaq/CILIAQ_v0.2.1.md` |
| correct a segmentation by hand | `editor/EDITOR_v0.0.4.md` |
| choose CiliaQ settings or interpret results | `preparator/PREPARATOR_v0.2.0.md` and `ciliaq/CILIAQ_v0.2.1.md` |
| check which versions to use or whether results can be combined | `VERSIONS.md` |

Not sure? Attach `00_START_HERE.md` alone — it tells the assistant which file it needs.

## Your data stays yours

You never need to upload your images. The assistant works from your description, from what you see
in Fiji, and from the small text files the plugins write (settings and thresholds, no image data).
Sharing a screenshot can help, but that is your choice — keep unpublished data private if you need
to. Agents run CiliaQ on your own computer; nothing has to leave it.

## Versions covered

| tool | version |
|---|---|
| CiliaQ Preparator | v0.2.0 |
| CiliaQ Editor | v0.0.4 |
| CiliaQ | v0.2.1 (static 2D and 3D images; for time-lapse use v0.1.7) |

Check yours in Fiji under `Plugins ▸ CiliaQ`. If your version differs, tell the assistant — the kit
instructs it to say what does and does not apply. `VERSIONS.md` lists what changed between versions
and will be continuously updated and revised in the future with new versions coming in.

## What's in this repository

```
00_START_HERE.md      read first, always: rules for the assistant, pipeline overview, triage
VERSIONS.md           version history, compatibility, which results can be combined
preparator/           CiliaQ Preparator — segmentation settings
editor/               CiliaQ Editor — manual correction
ciliaq/               CiliaQ — analysis settings and results
agents/               for agents that run the tools themselves
LICENSE               the license of this kit
CITATION.cff          citation information (GitHub's "Cite this repository")
```

## Feedback

We especially want to hear from people testing the kit:

- Did the assistant ask about your data before suggesting settings, or did it guess?
- Did it ever mention a setting that does not exist in your version?
- Did its suggestions work in Fiji the first time?
- Did your results improve, and could you tell why?
- Which assistant did you use?

Report errors in the kit in this repository's
[issue tracker](https://github.com/hansenjn/CiliaQ-LLM-Kit/issues). Share questions and
experiences in [CiliaQ Discussions](https://github.com/hansenjn/CiliaQ/discussions). Report bugs in
a plugin in that plugin's repository. Or e-mail jan.hansen (at) scilifelab.se.

## How to cite

If you publish results obtained with CiliaQ, please cite CiliaQ:

Hansen JN, Rassmann S, Stüven B, Jurisch-Yaksi N, Wachten D. CiliaQ: a simple, open-source software
for automated quantification of ciliary morphology and fluorescence in 2D, 3D, and 4D images.
*Eur. Phys. J. E* 44, 18 (2021). https://doi.org/10.1140/epje/s10189-021-00031-y

If this kit helped you choose settings, please also name it in your methods, with its version and
the assistant you used, e.g.: "Settings were chosen with the help of the CiliaQ LLM Kit v0.0.1
(https://github.com/hansenjn/CiliaQ-LLM-Kit) and <assistant and version>." The kit version is at
the top of `00_START_HERE.md`. GitHub's "Cite this repository" button gives the CiliaQ reference.

Once this kit and CiliaQ 0.2.x and CiliaQ Preparator 0.2.x versions with new functions have been fully
built we plan to publish the new implementations as a "CiliaQ 2.0" research paper.

## License

This kit is licensed under CC BY 4.0 (see `LICENSE`). CiliaQ, CiliaQ Preparator, and CiliaQ Editor
have their own license (GPL-3.0), in their own repositories.
