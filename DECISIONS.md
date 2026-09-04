# Architectural Decisions

§95 of [`prompt.md`](prompt.md) forbids silently changing core requirements. This file records
the decisions that were made explicitly, with the reasoning, so they are not quietly
re-litigated later or on another machine.

Add new entries here rather than changing code assumptions in place.

---

## D1 — Stack: Electron + TypeScript + React

**Date:** 2026-09-04 · **Status:** accepted

### Context

§94 requires the stack be chosen on Windows reliability, Cintiq pen support, low latency, local
file access, reliable audio, and precise logging — explicitly *not* on fashion. Candidates
considered: Electron + TS + React, Python + PySide6/Qt, C# + WPF.

### Decision

Electron + TypeScript + React.

### Why

- Chromium Pointer Events on Windows Ink expose pressure, `tiltX`/`tiltY`, hover, and
  sub-millisecond timestamps in the **same timebase as `performance.now()`** — which makes the
  §32 single monotonic clock nearly free and lets pen samples be stamped at input receipt
  rather than at handler execution (§72).
- `pointerrawupdate` and `getCoalescedEvents()` recover the full device-rate Wacom stream
  (≈133–200 Hz), not just the 60 Hz compositor rate.
- `requestVideoFrameCallback` on `<video>` gives an independently observable playback-start
  moment, which §6 requires be distinguished from the click.
- Node `fs` provides append-only streams and atomic rename, satisfying §41 and §66 with no
  database.
- The polished tablet-like UI the spec repeatedly demands (§11, §17, §24, §89, §90), plus SVG
  and PNG page snapshots and live thumbnails, is far cheaper here than in Qt or WPF.

### Rejected alternatives

- **Python + PySide6/Qt.** `QTabletEvent` is arguably the most direct Wacom path on Windows —
  native-rate samples, pressure, tilt, true proximity events, independent of Windows Ink
  quirks. Rejected because the required UI polish, live thumbnails, SVG export, and
  frame-accurate video callbacks are all substantially more work, and `QMediaPlayer`
  playback-start timing is less precise than `requestVideoFrameCallback`.
- **C# + WPF + Windows Ink.** `StylusPlugIn` on the real-time stylus thread is the
  lowest-latency ink path available on Windows. Rejected because every non-ink part (video,
  audio, SVG, thumbnails, UI) costs considerably more, for a latency gain that has not been
  shown to matter for this task.

### Consequences

- Tilt availability is contingent on Windows Ink being enabled in the Wacom driver. This is the
  first thing the pen probe checks; see `docs/PEN_CAPABILITY_CHECKLIST.md`.
- If the probe shows Chromium cannot deliver acceptable sample rate or fidelity on the Cintiq,
  revisit D1 before building further. That is precisely why the probe runs first.

---

## D2 — Eraser: stroke-level hit-testing

**Date:** 2026-09-04 · **Status:** accepted

### Context

§21 requires that erasing feel natural to the participant while never destructively deleting
historical data, and that `eraser.jsonl` record `affected_strokes` — or `null` if they cannot be
determined reliably (§56). Pixel-level erasing would force `null`.

### Decision

The eraser hit-tests its path against stroke polylines and removes whole strokes it touches.

### Why

- `affected_strokes` is then **always** reliably known, which is the analytically valuable
  outcome. `null` is permitted by the spec but is a strictly worse dataset.
- Undo restores exactly, with no raster snapshot stack (§22).
- Page SVG stays clean vector geometry with no mask compositing (§59).
- Strokes are never left partially destroyed, so `final_state.json`'s `visible_strokes` stays
  meaningful (§60).

### Consequences

- Participants experience a slightly "chunky" eraser — a whole stroke disappears rather than
  part of one. Accepted deliberately as a research trade-off.
- Revisit only if testing on the Cintiq shows it feels wrong in practice.

---

## D3 — First-pass scope: the §88 vertical slice

**Date:** 2026-09-04 · **Status:** accepted

### Decision

Build spec **Phase 1** (foundation) + **Phase 2** (audio + replay) first, then stop and validate
against real hardware before Phases 3–5.

### Why

§87 and §88 both ask for incremental implementation with an early inspectable output, and warn
against waiting for advanced features before validating the acquisition pipeline. Building
tools, pages, and recovery on top of unvalidated pen-input assumptions would bake any wrong
assumption into far more code.

### Consequences

Laser, Eraser, Undo, pages, thumbnails, crash recovery, `validate_session`, the test suite, and
the schema docs land in later passes. Their seams are reserved now: empty `laser.jsonl`,
`eraser.jsonl`, `undo.jsonl`, reserved `derived/` directories, and matching module boundaries.

---

## D4 — Development happens on the Windows + Cintiq machine

**Date:** 2026-09-04 · **Status:** accepted

### Decision

Build on the target hardware, testing pen behaviour as we go, rather than developing elsewhere
and porting.

### Why

The highest-risk unknowns in this project are all hardware-contingent: actual sample rate, tilt
availability, hover proximity behaviour, pressure curve, and Windows DPI/display mapping. §37
forbids fabricating device data, so the code cannot honestly be written until we know what the
device reports.

### Consequences

- `docs/PEN_CAPABILITY_CHECKLIST.md` runs **before** any input code is written.
- Windows-specific concerns (DPI scaling, display targeting, kiosk mode, Windows Ink) get
  validated directly instead of stubbed, though they remain the area most likely to need
  iteration.

---

## Standing constraints (§95 — do not silently introduce)

These are not decisions to revisit casually. Changing any of them requires an explicit entry
above explaining the technical limitation that forced it:

- no database storage of any kind — files only
- no cloud storage or automatic upload
- no mandatory autoplay of either stimulus view
- no live ASR feedback to the participant
- no unrestricted drawing features (no colour picker, no shapes, no text, no layers, no zoom)
- drawing is never disabled during Replay
- no page sidebar while only one page exists
- tools and pages never on the same side
- no ambiguous timestamp units — always explicit `_ms`
