# Pen Capability Checklist — run this FIRST

**Run this on the Windows machine with the Wacom Cintiq 24 attached, before writing any input
code.** These answers determine what `PenInputManager` can honestly claim to capture. §37 of
[`prompt.md`](../prompt.md) forbids fabricating device data, so anything the hardware does not
report must be written as `null` — and we need to know which fields those are before we build.

## How to run

Open [`pen-probe.html`](pen-probe.html) directly in Chrome or Edge on the Cintiq (double-click
the file; no server or install needed). Then, on the Cintiq surface:

1. **Hover** the pen ~1 cm above the surface and move it around slowly.
2. **Draw** several strokes with varying pressure, from feather-light to hard.
3. **Tilt** the pen markedly while drawing — lay it over toward each edge in turn.
4. Read the measurements off the panel and fill in the table below.

Also check first: **Wacom Tablet Properties → Mapping → "Use Windows Ink"**. Note whether it is
on or off, and if tilt reads zero, retry with it toggled.

## Record the answers here

| Question | Expected | Measured | Notes |
|---|---|---|---|
| `pointerType` reported for the pen | `"pen"` | | if `"mouse"`, driver/Windows Ink is wrong |
| `pointerrawupdate` fires at all | yes | | if no, fall back to `getCoalescedEvents()` |
| Sample rate while drawing (Hz) | ≈133–200 | | drives what "no downsampling" means (§72) |
| Coalesced events per `pointermove` | >1 | | confirms the fallback path also works |
| `pressure` range observed | ~0.0 → 1.0 | | note min/max actually reachable |
| `pressure` on hover | 0 | | must not be logged as contact |
| `tiltX` / `tiltY` ever non-zero | yes | | **if always 0 → `tilt_supported: false`** |
| `tiltX` / `tiltY` range (degrees) | ≈ ±60 | | |
| Hover events arrive with `buttons === 0` | yes | | hover capture is required (§19) |
| Hover proximity height | ~1 cm | | how far above the glass hover still reports |
| `pointerenter` / `pointerleave` fire on proximity | yes | | proximity in/out boundaries |
| `twist` / barrel rotation reported | maybe | | not required; record if present |
| `event.timeStamp` in `performance.now()` timebase | yes | | **critical for §32** |
| `devicePixelRatio` | | | record into `session.json` |
| `screen.width` × `screen.height` (CSS px) | | | |
| Windows display scaling % | | | 150%/200% is common on a 4K Cintiq |
| Windows Ink enabled in driver | | | |
| Wacom driver version | | | provenance (§80) |
| Cursor offset between pen tip and drawn point | none | | parallax/calibration check |

## What each answer changes

- **No `pointerrawupdate`** → `PenInputManager` uses `pointermove` + `getCoalescedEvents()`.
  Record which path was used in `session.json`.
- **Tilt always zero** → `pen_device.tilt_supported: false`, and `tilt_x`/`tilt_y` are written
  as `null` in `pen_raw.jsonl`. Never write `0` for "unknown".
- **Sample rate well below ~100 Hz** → investigate before proceeding; the raw pen stream is the
  authoritative research data and §72 forbids silently accepting downsampling.
- **`event.timeStamp` not matching `performance.now()`** → the clock design in
  `DEVELOPMENT_PLAN.md` §1.6 needs revisiting; this would be a significant finding.
- **DPI scaling** → feeds the coordinate mapping in `renderer/input/coords.ts` and the
  `platform` block of `session.json`.

## Also worth confirming while you are here

- [ ] Fullscreen/kiosk Electron window lands on the **Cintiq**, not the primary monitor
- [ ] Palm rejection behaves acceptably when resting a hand on the surface
- [ ] The pen's barrel buttons do not trigger anything unwanted (context menus, back navigation)
- [ ] Microphone is visible to `navigator.mediaDevices.enumerateDevices()` and its label is
      readable after permission is granted (needed for `audio_metadata.json`)
- [ ] `getUserMedia` actually yields 48 kHz — check `AudioContext.sampleRate`
