# Multimodal Motion Data Collection — Study Interface

A research instrument for an HCI study on how people perceive, communicate, and externalize
dynamic motion using speech and pen input. Participants watch a motion stimulus twice, then
speak and draw a reconstruction on a Wacom Cintiq 24.

This is **not** a drawing application. It is an experimental data-collection instrument:
visually simple for the participant, richly instrumented underneath.

> **Status: pre-implementation.** Planning is complete; no application code exists yet.

## Start here

| Document | What it is |
|---|---|
| [`prompt.md`](prompt.md) | The full specification. Authoritative. Sections are referenced as §n throughout. |
| [`DEVELOPMENT_PLAN.md`](DEVELOPMENT_PLAN.md) | Architecture, module map, data contract, build order, verification, risks. |
| [`DECISIONS.md`](DECISIONS.md) | Locked decisions with rationale. §95 forbids changing these silently. |
| [`docs/PEN_CAPABILITY_CHECKLIST.md`](docs/PEN_CAPABILITY_CHECKLIST.md) | **Run this first**, on the Cintiq, before writing input code. |

## Next action

On the Windows machine with the Cintiq attached, open
[`docs/pen-probe.html`](docs/pen-probe.html) in Chrome or Edge and fill in the checklist. What
the hardware actually reports — sample rate, tilt, hover, pressure range, DPI — determines what
the pen logger can honestly record. §37 of the spec forbids fabricating device data, so this
comes before the scaffold.

Then build the §88 vertical slice in the order given in `DEVELOPMENT_PLAN.md` §4.

## Planned stack

Electron + TypeScript + React. Renderer owns pen input, rendering, video and audio capture;
main process owns all disk I/O. File-only storage — JSON, JSONL, WAV, SVG, PNG. No database,
no cloud, no live speech recognition. See `DECISIONS.md` D1 for why.

## Data

Sessions are written to `data/participants/<PID>/<SID>/`, human-readable without running the
application. `DEVELOPMENT_PLAN.md` §3 gives the full tree and the field conventions;
`DATA_SCHEMA.md` and `FILE_STRUCTURE.md` follow in a later pass (§82, §83).

Participant data is pseudonymous and stays local. Nothing is uploaded.
