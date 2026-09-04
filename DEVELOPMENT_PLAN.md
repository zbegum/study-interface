# Development Plan — Research-Grade Multimodal Motion Data Collection App

**Status:** planning complete, implementation not started
**Target machine:** Windows + Wacom Cintiq 24
**Specification:** [`prompt.md`](prompt.md) — section references below (§n) point into it

---

## 0. Context

`prompt.md` specifies a research instrument for an HCI study on how people perceive,
communicate, and externalize dynamic motion using speech and pen input. It is not a drawing
application. The participant sees something close to a minimal iPad / Apple Pencil surface:
watch a motion stimulus twice, then speak and draw a reconstruction. Underneath, the app must
produce a file-only, human-readable, crash-safe dataset rich enough to later reconstruct the
**complete behavioral history** — not just the final sketch.

The repository was empty at the time of planning (`prompt.md` + `.gitattributes`). This is
greenfield: there is no existing stack to preserve or adapt.

Implementation happens **on the Windows machine with the Cintiq attached**, hardware-first.
See [`docs/PEN_CAPABILITY_CHECKLIST.md`](docs/PEN_CAPABILITY_CHECKLIST.md) — that probe runs
*before* any input code is written, because its answers determine what `PenInputManager` can
honestly claim to capture.

Locked decisions and their rationale live in [`DECISIONS.md`](DECISIONS.md). §95 forbids
silently changing them.

---

## 1. Pre-Implementation Report (§98)

### 1.1 Current repository / technology structure

Empty. `prompt.md` and `.gitattributes` only, single "Initial commit". Nothing to adapt.

### 1.2 Recommended architecture

**Electron + TypeScript + React**, two-process split:

```
main (Node)                        renderer (React/TS)
─────────────────────────          ────────────────────────────────────
FileDataStore                      PenInputManager   (pointerrawupdate)
  JSONL appenders (append-only)    CanvasRenderer    (desynchronized:true)
  atomic JSON writes               StimulusPlayer    (requestVideoFrameCallback)
SessionManager                     AudioRecorder     (AudioWorklet, 48 kHz mono)
RecoveryManager                    ToolManager / InkStyleManager / PageManager
WavWriter                          EventLogger
        ▲                                   │
        └────────── IPC batches ────────────┘
              (250 ms flush, never on the input path)
```

The **renderer** owns everything latency-sensitive and everything that carries a true
high-resolution timestamp: pen input, canvas rendering, video playback callbacks, audio capture.
The **main process** owns all disk I/O. Nothing that touches the pen input path ever touches the
filesystem synchronously.

Why this stack, given §94's instruction not to pick a framework because it is fashionable:

- Chromium Pointer Events on Windows Ink deliver pressure, `tiltX`/`tiltY`, hover, and
  sub-millisecond monotonic timestamps in the *same* timebase as `performance.now()`.
- `pointerrawupdate` / `getCoalescedEvents()` recover the full device-rate Wacom sample stream
  rather than the 60 Hz compositor rate.
- HTML5 `<video>` + `requestVideoFrameCallback` gives a precise, separately observable
  playback-start moment — which §6 explicitly requires be distinguished from the click.
- Node's `fs` gives append-only streams and atomic rename, satisfying §41/§66 with no database.
- SVG/PNG page snapshots, live thumbnails, and the polished tablet-like palette the spec
  demands (§11, §17, §24, §89, §90) are dramatically cheaper in this stack than in Qt or WPF.

The trade-off accepted: Qt (`QTabletEvent`) and WPF (`StylusPlugIn`) offer marginally more
direct pen access and, for WPF, lower theoretical ink latency. The pen probe in §1.4 exists to
confirm the Chromium path is good enough on the actual hardware before we commit further.

### 1.3 Wacom / Cintiq pen input strategy

Pointer Events bound to the workspace element:

- **`pointerrawupdate`** for uncoalesced device-rate samples (Wacom ≈ 133–200 Hz), with
  `pointermove` + `getCoalescedEvents()` as fallback where `pointerrawupdate` is unavailable.
  Whichever path is used is recorded per session; §72 forbids silent downsampling.
- Filter on `event.pointerType === 'pen'`. Mouse and touch input are still recorded but tagged
  with `pointer_type`, so a mouse-driven development session can never be mistaken for pen data.
- `setPointerCapture` on pen-down, so a stroke survives the pen leaving the element bounds.
- `touch-action: none` and `user-select: none` on the workspace to stop gesture interception.

### 1.4 Expected hover / pressure / tilt support

| Capability | Source | Expectation |
|---|---|---|
| Pressure | `event.pressure`, 0..1 | Reliable for pen. Chromium reports `0.5` for mouse-down and `0` on hover — for non-pen pointers we store `null`, never a fabricated `0.5`. |
| Tilt | `tiltX` / `tiltY`, degrees | Available on Windows **when Windows Ink is enabled** in the Wacom driver. If the probe finds tilt is always 0, we record `tilt_supported: false` and write `null` (§37: never fabricate device data). |
| Hover | `pointermove`/`pointerrawupdate` with `buttons === 0`; `pointerenter`/`pointerleave` for proximity | Well supported for Wacom pens. Hover is scientifically relevant (§19) and is never discarded. |

A **pen capability probe** on the setup screen samples a few seconds of real input and writes
what it actually observed into `session.json.pen_device`. The participant-facing setup UI shows
only `Pen ✓` (§70); details stay in the data.

### 1.5 Audio capture strategy

```js
getUserMedia({ audio: {
  channelCount: 1, sampleRate: 48000,
  echoCancellation: false, noiseSuppression: false, autoGainControl: false
}})
```

→ `AudioWorklet` → Float32 → Int16 PCM → batched IPC → main appends to `audio/session.wav`.

Raw PCM rather than `MediaRecorder`, because §30 requires WAV and §31 requires sample-accurate
anchoring to the experiment clock. Browser DSP is switched off so the recording stays
research-grade. The same worklet emits RMS values that drive the microphone activity indicator
(§28) — this is an **audio-capture** indicator, not transcription. No live ASR anywhere (§63).

### 1.6 Monotonic clock strategy (§32 — critical)

**One session clock. It never resets between trials.**

- `t0 = performance.now()` captured at `session_start`; `t_ms = performance.now() - t0`.
- Because `PointerEvent.timeStamp` is in the *same* `performance.now()` timebase, every pen
  sample is stamped `event.timeStamp - t0` — the actual input-receipt time, not the time our
  handler happened to run (§72). Coalesced events each carry their own `timeStamp`.
- Main-process-originated events convert through a `clock_correlation` pair
  (`process.hrtime.bigint()` ↔ renderer `performance.now()`) captured once at session start and
  stored in `session.json`.
- `trial_t_ms` is always **derived** and never authoritative (§33).
- Wall-clock time is recorded for provenance only (§34).
- All time fields are milliseconds with explicit `_ms` names (§35).

### 1.7 Participant screen / state map (§75)

```
READY_VIEW_1 → PLAYING_VIEW_1 → READY_VIEW_2 → PLAYING_VIEW_2
             → READY_RECONSTRUCTION → RECONSTRUCTION → FINALIZING → COMPLETE
                                                                  (+ ERROR)
```

**Replay is not a state.** It is a concurrent media activity inside `RECONSTRUCTION`. Pen, ink,
laser, eraser, speech, and audio recording all stay live while a replay plays (§26).

### 1.8 File persistence design

`FileDataStore` in the main process is the single writer. No UI component ever constructs a path
(§74).

- **Append-only** `JsonlAppender`: one `fs.createWriteStream` per stream, `\n`-delimited,
  periodic `fsync` — for all chronological/high-frequency streams.
- **Atomic** `writeJsonAtomic`: write `.tmp` → `fsync` → `rename` — for mutable metadata
  (`session.json`, `trial.json`, `pages.json`, `final_state.json`) (§66).
- No database of any kind. No `DatabaseManager` module exists (§41, §73).

### 1.9 Expected generated directory tree

Exactly §96 — see section 3 below.

### 1.10 Major technical risks

See section 7.

---

## 2. Repository Layout

```
package.json  tsconfig*.json  electron.vite.config.ts  README.md
DEVELOPMENT_PLAN.md  DECISIONS.md
config/experiment.json                 # §79, never shown to participants
docs/PEN_CAPABILITY_CHECKLIST.md
data/{participants,stimuli,exports,logs}/
  stimuli/stimuli.json  stimuli/videos/ST00*.mp4
scripts/make-placeholder-stimuli.sh    # ffmpeg-generated dev stimuli
src/
  shared/      types.ts  ids.ts  config.ts  events.ts     # shared by both processes
  main/        index.ts  windows.ts  clock.ts  ipc.ts
               FileDataStore.ts  JsonlAppender.ts  atomicJson.ts
               SessionManager.ts  RecoveryManager.ts  WavWriter.ts
  preload/     index.ts                                   # contextBridge, typed API only
  renderer/
    experiment/ ExperimentController.ts  TrialStateMachine.ts  clock.ts
    input/      PenInputManager.ts  coords.ts
    render/     CanvasRenderer.ts
    tools/      ToolManager.ts  InkStyleManager.ts
    pages/      PageManager.ts
    audio/      AudioRecorder.ts  MicLevelMonitor.ts  pcm-worklet.ts
    media/      StimulusPlayer.tsx
    logging/    EventLogger.ts
    screens/    SetupScreen  ViewScreen  ReconstructionScreen
                TransitionScreen  CompleteScreen
    ui/         ToolPalette  InkSubPalette  TopRightBar  PenCursor
    theme/      tokens.ts                                 # §15/§16 values live here only
```

Module names deliberately mirror the §73 conceptual list.

---

## 3. Data Contract

What a V1 session writes (§96):

```
data/participants/P001/S001/
├── session.json            # atomic; §49 + provenance §80 + clock_correlation
├── session_events.jsonl    # append-only; §50
├── audio/
│   ├── session.wav
│   └── audio_metadata.json
├── recordings/             # created empty; filled externally (§62)
├── trials/
│   └── T001/
│       ├── trial.json      # created at trial start, status "in_progress" (§65)
│       ├── events.jsonl
│       ├── pen_raw.jsonl
│       ├── strokes.jsonl
│       ├── laser.jsonl     # created empty in the V1 slice
│       ├── eraser.jsonl    # created empty in the V1 slice
│       ├── undo.jsonl      # created empty in the V1 slice
│       ├── pages.json
│       ├── final_state.json
│       └── pages/
│           ├── page_01.svg
│           └── page_01.png
└── derived/
    ├── transcript/         # reserved, empty
    ├── processed/          # reserved, empty
    └── annotations/        # reserved, empty
```

**Conventions fixed now, to be documented in `DATA_SCHEMA.md`:**

- `t_ms` — float, milliseconds from the session monotonic clock origin. Authoritative.
- `sample_seq` — **session-wide** strictly increasing integer.
- All `sample_seq_start` / `sample_seq_end` ranges are **inclusive** (§36 requires this be
  defined and applied consistently).
- Unavailable device values are `null`. Never invented (§37).
- Stimulus paths are stored relative to `data/`, e.g. `stimuli/videos/ST001.mp4`. Never
  traversal paths like `../../../` (§46).
- ID formats: `P001`, `S001`, `T001`, `ST001`, `page_01`, `stroke_0001`, `laser_0001`,
  `erase_0001`, `undo_0001` (§44). No participant names in paths (§81).

---

## 4. Build Order

### Scope of the first pass

Spec **Phase 1** (foundation) + **Phase 2** (audio + replay) — the §88 vertical slice. This is
deliberate: validate real Cintiq acquisition before building tools, pages, and recovery on top
of unvalidated input assumptions.

### Step 0 — Pen probe on real hardware

Run [`docs/PEN_CAPABILITY_CHECKLIST.md`](docs/PEN_CAPABILITY_CHECKLIST.md) **before writing
input code.** Its answers determine what `PenInputManager` can honestly claim.

### Step 1 — Scaffold

`electron-vite` + React + TypeScript. `config/experiment.json` matching §79 verbatim:
`mandatory_video_views: 2`, `allow_replay: true`, `laser_fade_ms: 750`, ink colours
black/blue/red/green with `default_color: "black"`, thicknesses thin/medium/thick with
`default_thickness: "medium"`, `audio_enabled: true`, `fullscreen: true`.
`scripts/make-placeholder-stimuli.sh` generates short ffmpeg motion clips plus `stimuli.json`
so the flow is runnable immediately. No filenames are hard-coded into UI components (§47).

### Step 2 — Persistence layer (main), before any UI

`atomicJson.ts`, `JsonlAppender.ts`, `FileDataStore.ts`. `SessionManager` creates the full tree
and **pre-creates every per-trial file at trial start** with `status: "in_progress"` (§65) —
data is written during the trial, never held until Done. `RecoveryManager` scans
`data/participants/*/*/trials/*/trial.json` at startup for in-progress trials, and refuses to
silently overwrite an existing participant/session (§67, §68). Both surface on the setup screen.

### Step 3 — Clock + EventLogger

`renderer/experiment/clock.ts` owns `t0`, `t_ms()`, `trialT0`, and the `sample_seq` counter.
`EventLogger` sends **discrete events immediately**, and **pen samples via a ring buffer flushed
every 250 ms** — and force-flushed at stroke end, page change, replay boundaries, and Done. Disk
I/O never touches the input path (§72).

### Step 4 — Setup screen (§3)

Participant ID, Session ID (auto `S00N`), handedness radio, stimulus set, output directory,
microphone status, pen status, Start Session. Researcher-facing and deliberately plain — this is
explicitly **not** an experiment-management dashboard (§2, §92).

Microphone validation per §29: capture ~1 s and check for a live signal. On failure, block start
and require either a fix or an explicit researcher override, which is logged as
`microphone_override`. Never silently start a session with a dead microphone.

### Step 5 — Participant mode + trial state machine

Kiosk `BrowserWindow` on the chosen display (`screen.getAllDisplays()`, configurable index).
`devicePixelRatio` and display bounds recorded into `session.json`. Explicit `TrialStateMachine`
implementing §75; illegal transitions throw rather than silently self-correct.

### Step 6 — View 1 / View 2 (§5–§7)

`StimulusPlayer` with `preload="auto"` and **no autoplay, ever** (§5 marks this CRITICAL).
Three distinct timestamps:

| Event | Source |
|---|---|
| `stimulus_view_requested` | the click |
| `stimulus_playback_start` | first `requestVideoFrameCallback` (carries `media_time_s`) |
| `stimulus_playback_end` | `ended` |
| playback error | `error` |

Click time is never treated as playback start (§6). Reconstruction is unreachable until both
mandatory views have completed.

### Step 7 — Reconstruction workspace (§8–§13)

Handedness-driven layout (§9): right-handed → `[WORKSPACE][TOOLS]`, left-handed →
`[TOOLS][WORKSPACE]`. Replay and the microphone indicator stay **top-right for both**
handedness settings — §9 explicitly forbids mirroring them. No page strip in V1: a single page
should simply feel like the workspace (§10).

Every layout/geometry change emits `workspace_layout_changed` with before/after bounds (§39),
so workspace and normalized coordinates stay reconstructable across resizes.

Tool palette (§11, §12, §90): floating, vertical, icon-first — explicitly **not** five equally
sized rectangular buttons. Ink / Laser / Eraser are mutually exclusive modes with Ink as
default; Undo and New Page are actions. In the V1 slice the not-yet-wired tools are rendered
and honestly disabled rather than faked. Selected-tool feedback uses several simultaneous cues
— rounded background, larger icon, stronger contrast, outline, heavier label — never colour
alone (§12). A subtle `PenCursor` follows the pen position (§13).

`CanvasRenderer` uses two canvases for latency: a **wet** canvas that draws the in-progress
stroke on every input sample, and a **dry** canvas holding committed strokes — both created with
`getContext('2d', { desynchronized: true })`. On pen-up the stroke commits to dry and wet clears.

### Step 8 — Ink + raw capture (§14, §37, §38)

Each stroke gets a `stroke_0001`-style ID and records the colour and thickness active **at
stroke start**; later changes never retroactively modify earlier strokes (§15, §16).
`strokes.jsonl` stores the summary and an inclusive `sample_seq` range; the raw points stay in
`pen_raw.jsonl`, which is authoritative (§14: do not save only SVG/PNG).

Every raw sample carries the full §37 field set: `sample_seq`, `t_ms`, `trial_t_ms`, screen px,
workspace px, normalized workspace, `pressure`, `tilt_x`, `tilt_y`, `contact`, `hover`, `tool`,
`color_id`, `thickness_id`, `page_id`, `stroke_id`. Hover samples are captured and kept (§19).

### Step 9 — Audio (§28–§31)

`AudioRecorder` streams Int16 PCM to main. `WavWriter` writes a 48 kHz mono WAV header with
placeholder sizes and patches them at finalize; after a crash the true size is still recoverable
from the file length. `audio_metadata.json` stores `audio_start_t_ms`, `sample_rate_hz`,
`channels`, `device`, **plus the latency components used to derive the anchor** — so the
estimate is auditable rather than presented as exact. `MicLevelMonitor` drives the top-right
meter. No ASR, no transcript display, no participant feedback based on recognition (§28).

### Step 10 — Replay (§26, §27)

A small floating panel, upper-right / upper-central, auto-dismissed on `ended`. The workspace
stays fully live underneath — pen, ink, and audio all continue. Logs `replay_requested` (with
`replay_index` and the active page), `replay_start`, `replay_end`, and any playback error. No
replay limit unless configuration sets one.

### Step 11 — Done + finalization (§65, §76)

Done is prominent but placed away from natural drawing reach; one deliberate activation, no
nagging confirmation dialogue. On activation, in order:

1. close any open stroke / laser / erase episode
2. append `trial_done_requested`
3. flush and close all append streams
4. render `page_01.svg` and `page_01.png`
5. write `final_state.json`
6. atomically rewrite `trial.json` with `status: "completed"` and `end_t_ms`

The next trial does not begin until finalization resolves.

### Step 12 — Transition + session complete (§77, §78)

Minimal transition screen between trials. A bare "Session complete. Thank you." at the end. No
filenames, no diagnostics, no research data shown to the participant.

---

## 5. Explicitly Deferred

Not in the first pass. The architecture reserves their seams — empty JSONL files, reserved
directories, module boundaries — so they drop in cleanly:

- **Phase 3** — ink colour/thickness UI wiring, Laser (`laser.jsonl`, 750 ms fade, full
  trajectory always retained regardless of visual fade, §20), stroke-level Eraser, Undo.
- **Phase 4** — New Page, conditional page strip, handedness-aware strip placement, live
  content thumbnails, page switching, multi-page SVG/PNG snapshots.
- **Phase 5** — `validate_session` (§85), the §86 automated test suite, `DATA_SCHEMA.md` (§82),
  `FILE_STRUCTURE.md` (§83), developer debug mode (§71, never visible in a real session).

Permanently out of scope for V1 (§92): experimenter dashboard, participant database, web
backend, authentication, cloud sync, annotation system, transcript editor, Talmy/gesture coding
interfaces, analysis dashboard, automated semantic classification. Per §40 the collector stays
theoretically neutral — it records observable behaviour and annotates nothing.

---

## 6. Verification

Run on the Windows machine with the Cintiq attached, checking each item as it is built rather
than all at the end.

0. **Pen probe on real hardware** — see `docs/PEN_CAPABILITY_CHECKLIST.md`. Record the answers
   in that file.
1. `npm run dev` → setup screen shows microphone and pen status. Try Start with the microphone
   denied: confirm it blocks and offers a logged override.
2. Complete one trial: View 1 → View 2 → reconstruct with several ink strokes → Replay
   mid-drawing (confirm you can keep drawing while it plays) → Done.
3. Tree matches §96: `find data/participants/P001/S001 -type f | sort`
4. Timing invariants:
   - `jq -s 'map(.t_ms) | . == sort' trials/T001/pen_raw.jsonl` → `true`
   - `jq -s '[.[].sample_seq] | . == (unique | sort)' trials/T001/pen_raw.jsonl` → `true`
     (strictly increasing, no duplicates)
   - `t_ms` does **not** reset at trial 2 — the first `trial_start` of T002 must exceed the last
     event of T001
   - spot-check `trial_t_ms == t_ms - trial_start.t_ms` on a few rows
5. Playback separation: `stimulus_view_requested` and `stimulus_playback_start` must have
   **different** `t_ms` values in `events.jsonl`.
6. Audio: `session.wav` opens in any player, duration ≈ session length, and
   `audio_metadata.json` carries the anchor.
7. Crash safety: kill the app mid-trial, restart, and confirm `trial.json` is still
   `"status": "in_progress"`, that the partial JSONL is still valid line-by-line, and that
   `RecoveryManager` reports the incomplete trial without overwriting it.
8. Handedness: run one session left, one right. Tools swap sides; Replay and the microphone
   indicator stay top-right in both.

---

## 7. Risks

1. **Tilt on Windows / Wacom** depends on Windows Ink being enabled in the driver. The
   capability probe records what was actually observed, and we write `null` rather than `0`.
   Must be confirmed on the real Cintiq before data collection begins.
2. **True sample rate.** If `pointerrawupdate` under-delivers on the target machine we fall back
   to `getCoalescedEvents()`. Either way the achieved rate is measured and recorded per session.
   §72 forbids silent downsampling, so any downsampling would be documented.
3. **Audio anchor precision.** `audio_start_t_ms` is an estimate (worklet block size + input
   latency). We store the components so it can be re-derived, and never claim exactness.
4. **Windows DPI / display mapping** is isolated in `main/windows.ts`. Because the build happens
   on the target machine this gets validated directly rather than stubbed — but it remains the
   area most likely to need iteration against the Cintiq's scaling settings.
5. **Renderer buffer loss on crash.** Up to ~250 ms of pen samples could be lost in a hard
   crash. Discrete events are sent immediately and buffers force-flush at every episode
   boundary, so structural records stay intact. Documented rather than hidden.
6. **Stroke-level erase** is a deliberate research trade-off: reliable `affected_strokes` and
   clean vector SVG at the cost of a chunkier erase feel. See `DECISIONS.md`. Revisit only if
   Cintiq testing shows it feels wrong.
