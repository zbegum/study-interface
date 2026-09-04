# Research-Grade Multimodal Motion Data Collection Application

You are building a research-grade multimodal data collection application for an HCI study investigating how people perceive, communicate, and externalize dynamic motion using speech and pen input.

The application will run on a Windows computer connected to a Wacom Cintiq 24 pen display.

This is NOT a generic drawing application.

It is an experimental research instrument.

The participant-facing experience should feel closer to a minimal iPad / Apple Pencil drawing surface than to a desktop graphics editor.

The software should be visually simple for participants while producing a rich, synchronized, transparent research dataset underneath.

---

# 1. PRIMARY GOALS

Prioritize:

1. precise timestamped logging,
2. low-latency pen interaction,
3. reliable pen + audio capture,
4. minimal participant-facing interaction,
5. reproducibility,
6. crash-safe local file storage,
7. clean experimental trial sequencing,
8. support for later multimodal alignment,
9. support for later linguistic / HCI / motion coding,
10. transparent human-readable research data.

The system must preserve enough raw information to later reconstruct:

* what stimulus the participant saw,
* when the participant requested playback,
* when playback actually started,
* when playback ended,
* what the participant said,
* what they drew,
* where the pen hovered,
* what they pointed/traced using the Laser tool,
* what they erased,
* what they undid,
* which page they were using,
* when they created pages,
* when they switched pages,
* when they changed ink color,
* when they changed ink thickness,
* when they replayed the stimulus,
* and the timing relationships among all of these behaviors.

The COMPLETE behavioral history matters.

The final sketch alone is NOT the primary research data.

---

# 2. V1 SCOPE: PARTICIPANT EXPERIENCE FIRST

For the first implementation, focus on the PARTICIPANT-FACING EXPERIMENT EXPERIENCE.

Do NOT build a sophisticated experimenter dashboard in V1.

The purpose of V1 is to validate:

* Wacom Cintiq interaction,
* pen coordinate capture,
* pressure,
* hover,
* tilt if supported,
* low-latency ink rendering,
* video trial flow,
* participant-controlled viewing,
* speech recording,
* microphone monitoring,
* synchronized event capture,
* Ink,
* Laser,
* Eraser,
* Undo,
* multiple pages,
* constrained colors,
* constrained stroke thickness,
* Replay,
* reliable local storage.

A minimal pre-session configuration screen is allowed because some information must be configured before participant mode begins.

This screen should remain functional and simple.

Do NOT spend major implementation effort making it into an experiment-management dashboard.

---

# 3. MINIMAL PRE-SESSION SETUP

Before entering participant mode, provide a simple setup screen containing:

* Participant ID
* Session ID, or auto-generated session ID
* Drawing hand:

  * Left
  * Right
* Stimulus manifest/set
* Output directory
* Microphone status
* Pen status
* Start Session

Example:

Research Session Setup

Participant ID:
[P001]

Session ID:
[S001]

Drawing hand:
○ Left
● Right

Stimulus set:
[default]

Output:
[data/participants/...]

Microphone:
✓ Connected

Pen:
✓ Detected

[ START SESSION ]

This screen is researcher-facing only.

It does NOT need:

* analytics,
* participant management,
* past-session browsing,
* trial dashboards,
* transcript editing,
* annotation interfaces,
* statistics,
* charts,
* complex hardware administration.

---

# 4. PARTICIPANT MODE

After Start Session:

* enter fullscreen participant mode on the Cintiq,
* remove setup controls,
* hide desktop chrome as much as practical,
* keep the interface visually quiet.

Participant-facing sequence:

1. Ready for View 1
2. Playing View 1
3. Ready for View 2
4. Playing View 2
5. Ready to Begin Reconstruction
6. Reconstruction Workspace
7. Trial Transition
8. Next Trial
9. Session Complete

---

# 5. CORE TRIAL FLOW

Each trial contains one animation/video stimulus.

Required sequence:

1. Trial starts.
2. Participant explicitly initiates View 1.
3. Video plays once.
4. Participant explicitly initiates View 2.
5. Video plays once.
6. Participant enters reconstruction workspace.
7. Participant can:

   * speak,
   * draw using Ink,
   * use Laser,
   * erase,
   * undo,
   * choose ink color,
   * choose ink thickness,
   * create new pages,
   * switch between pages,
   * optionally replay the stimulus.
8. Participant presses Done.
9. Trial data is finalized.
10. Proceed to next trial.

CRITICAL:

DO NOT autoplay either of the two mandatory stimulus views.

The participant explicitly starts both.

---

# 6. VIEW 1 SCREEN

Conceptually:

VIEW 1 OF 2

[ video area ]

▶ START VIDEO

The participant must initiate playback.

Log separately:

* participant playback request,
* actual playback start,
* actual playback end,
* errors if any.

Do NOT assume click time equals media playback start time.

---

# 7. VIEW 2 SCREEN

After View 1 completes:

View 1 complete ✓

▶ START SECOND VIEW

Then:

VIEW 2 OF 2

[ video ]

▶ START VIDEO

After View 2:

View 2 complete ✓

[ BEGIN RECONSTRUCTION ]

Do not enter reconstruction until both mandatory views are complete.

---

# 8. RECONSTRUCTION WORKSPACE

The reconstruction workspace is the main participant interface.

It should contain:

* very large drawing workspace,
* floating pen-tool palette,
* Ink,
* Laser,
* Eraser,
* Undo,
* New Page,
* Done,
* Replay,
* microphone activity indicator.

Replay and microphone status belong near the TOP-RIGHT corner.

Example concept:

Trial 05 / 16                         ▶ Replay   🎙 waveform

```
             LARGE WORKSPACE


                                             floating tools
```

The workspace should visually dominate the display.

---

# 9. HANDEDNESS LAYOUT

The interface changes according to drawing hand.

## Right-handed participant

One page:

[ WORKSPACE ][ TOOLS ]

Multiple pages:

[ PAGES ][ WORKSPACE ][ TOOLS ]

## Left-handed participant

One page:

[ TOOLS ][ WORKSPACE ]

Multiple pages:

[ TOOLS ][ WORKSPACE ][ PAGES ]

The purpose is to keep navigation away from the primary drawing hand.

Replay and microphone status remain near the upper-right regardless of handedness.

Do NOT mirror Replay/microphone to the left for left-handed users.

---

# 10. NO PAGE BAR INITIALLY

The initial reconstruction workspace contains exactly one page.

Do NOT show a page sidebar/strip while only one page exists.

The first page should simply feel like the workspace.

Only after the participant creates another page should page navigation become visible.

---

# 11. TOOL PALETTE VISUAL LANGUAGE

The palette should feel like a compact tablet drawing palette.

Do NOT implement Ink, Laser, and Eraser as conventional rectangular desktop buttons.

Use a floating vertical palette.

Conceptually:

```
  ╭─────────╮
  │    ✎    │
  │   INK   │
  ╰─────────╯

      ◉
    LASER

      ◇
    ERASER

      ↶
     UNDO

      +
  NEW PAGE
```

Ink, Laser, and Eraser are mutually exclusive MODES.

Undo and New Page are ACTIONS.

Only one of:

* Ink
* Laser
* Eraser

may be active at once.

Default active tool:

INK.

---

# 12. SELECTED TOOL FEEDBACK

The selected tool must be immediately obvious.

Do not rely on color alone.

Use several cues such as:

* rounded background,
* larger icon,
* stronger contrast,
* border/outline,
* heavier label,
* spacing.

Example inactive:

◉
LASER

Example selected:

╭─────────╮
│    ✎    │
│   INK   │
╰─────────╯

The participant should know the selected pen mode without having to inspect the interface carefully.

---

# 13. ACTIVE PEN CURSOR FEEDBACK

Provide subtle feedback near the pen position.

Ink:

* small pen/dot indicator.

Laser:

* subtle glowing target or laser indicator.

Eraser:

* small circle approximately representing eraser area.

Keep these indicators subtle.

Do not make them visually distracting.

---

# 14. INK TOOL

Ink creates persistent visible marks.

A stroke begins on pen contact and ends on pen-up.

Record each stroke separately.

Capture as much raw pen information as the hardware/API reliably provides:

* session monotonic timestamp,
* sample sequence ID,
* screen x/y,
* workspace x/y,
* normalized workspace x/y,
* pressure,
* tilt x,
* tilt y,
* contact state,
* hover state,
* selected tool,
* selected color,
* selected thickness,
* page ID,
* stroke ID.

Do NOT save only SVG/PNG.

Raw samples are authoritative.

---

# 15. INK COLORS

Allow exactly four fixed Ink colors:

* Black
* Blue
* Red
* Green

Do NOT provide an unrestricted color picker.

Do NOT allow custom colors.

Use the same palette for every participant.

Default:

BLACK.

The exact visual values should be defined centrally in configuration/theme code so they remain consistent across sessions.

Log every color change.

Example:

{
"t_ms": 184203.551,
"event": "ink_color_changed",
"from": "black",
"to": "red"
}

Each stroke must also store the color active when the stroke started.

Changing color must not retroactively modify earlier strokes.

---

# 16. INK THICKNESS

Allow exactly three fixed stroke thicknesses:

* Thin
* Medium
* Thick

Do NOT use a continuous thickness slider.

Default:

MEDIUM.

Store the actual rendered width corresponding to each preset in configuration.

Example:

thin = configurable fixed width
medium = configurable fixed width
thick = configurable fixed width

Log thickness changes.

Example:

{
"t_ms": 187932.110,
"event": "ink_thickness_changed",
"from": "medium",
"to": "thick"
}

Every stroke should preserve:

* thickness_id,
* actual width used.

Changing thickness must not modify earlier strokes.

---

# 17. INK SUB-PALETTE

When Ink is active, color and thickness controls may appear as a compact secondary palette.

Conceptually:

╭────────────────╮
│       ✎        │
│      INK       │
│                │
│ ●  ●  ●  ●     │
│                │
│ ─   ━   █      │
╰────────────────╯

The actual UI should be polished and compact.

Avoid making it look like a desktop formatting panel.

---

# 18. DO NOT TURN THIS INTO A FULL DRAWING APPLICATION

Do NOT provide:

* unrestricted color picker,
* unlimited brush sizes,
* text tool,
* geometric shape tool,
* line tool,
* arrow tool,
* lasso,
* selection,
* transform,
* resize,
* object manipulation,
* layers,
* rulers,
* zoom controls,
* formatting menus,
* stickers,
* image insertion.

The participant should have expressive flexibility but a constrained research interface.

---

# 19. HOVER CAPTURE

Capture pen hover whenever technically possible.

Hover is scientifically relevant.

Potential later analyses include:

* pre-stroke planning,
* spatial preparation,
* movement before contact,
* hovering while speaking,
* hesitation before committing to ink.

Do not discard hover data.

Hover should use the same timebase as all other interaction.

---

# 20. LASER TOOL

Laser is an ephemeral digital pointer/trace.

It behaves visually like transient ink or a laser pointer.

The visible trajectory should fade after approximately:

500–1000 ms

Default suggestion:

750 ms

This should be configurable.

CRITICAL:

Visual disappearance must NOT delete research data.

The complete underlying trajectory must remain permanently recorded.

Store:

* laser episode ID,
* page ID,
* start time,
* end time,
* sample sequence range,
* raw trajectory,
* pressure if available,
* contact/hover behavior.

Do NOT automatically classify Laser behavior as:

* pointing,
* tracing,
* gesture,
* path.

Capture observable behavior only.

Semantic classification occurs later.

---

# 21. ERASER TOOL

Eraser removes Ink from the visible page.

The participant should experience natural erasing.

However, historical data must NEVER be destructively deleted.

Example:

stroke_0014 created
→ erase_0003 modifies stroke
→ undo_0003 restores erase

All actions remain in the raw dataset.

Store erase episodes including:

* erase ID,
* page ID,
* start/end timestamps,
* trajectory/sample range,
* affected strokes if technically reliable.

If affected strokes cannot be determined reliably, use null rather than guessing.

---

# 22. UNDO

Undo changes the current visible state.

Undo must NOT delete historical records.

Log:

* undo ID,
* timestamp,
* target event/action type,
* target ID,
* resulting state information where useful.

Example:

{
"undo_id": "undo_0003",
"t_ms": 211244.102,
"target_type": "erase",
"target_id": "erase_0003"
}

---

# 23. NEW PAGE

Initially:

page_01 exists.

No page navigation is visible.

When the participant selects:

*

NEW PAGE

the system should:

1. create a new page,
2. generate the next deterministic page ID,
3. switch immediately to it,
4. reveal the page navigation strip,
5. show Page 1 and Page 2 thumbnails.

Example:

Page 1
[thumbnail]

Page 2
[thumbnail] ← active

---

# 24. PAGE THUMBNAILS

Page thumbnails must visually represent the actual current visible content of each page.

They should update after relevant Ink/erase/undo operations.

Their purpose is navigation and recognition.

Do not show generic blank boxes labeled only "Page 1", "Page 2" when actual content exists.

Participants must be able to recognize previous pages visually.

---

# 25. PAGE SWITCHING

Participants can switch between any created pages.

Switching must preserve page content.

Log:

* page switch timestamp,
* source page,
* destination page.

Every Ink stroke, Laser episode, erase action, etc. must store the page ID active when it occurred.

Do NOT tell participants that pages represent temporal sequence.

Pages are simply additional expressive space.

---

# 26. REPLAY

Replay is available after the two mandatory views.

Place Replay near the TOP-RIGHT corner next to the microphone activity indicator.

Example:

▶ Replay     🎙 ▁▃▆▂

When Replay is requested:

* display the stimulus in a temporary floating video panel,
* preferably upper-right / upper-central,
* do not permanently consume a major portion of the workspace,
* play the video,
* dismiss/hide the replay panel automatically at the end.

CRITICAL:

While Replay is playing:

* pen remains active,
* Ink remains active,
* Laser remains active,
* Eraser remains active,
* speech remains active,
* audio recording remains active.

Do NOT disable the workspace during replay.

Participants may speak and draw while viewing the replay.

---

# 27. REPLAY LOGGING

For every replay record:

* participant request time,
* actual media start,
* actual media end,
* replay index,
* active page at request,
* playback errors.

Example:

{
"t_ms": 234821.004,
"event": "replay_requested",
"replay_index": 2,
"page_id": "page_02"
}

Then:

{
"t_ms": 234848.100,
"event": "replay_start",
"replay_index": 2
}

Then:

{
"t_ms": 238422.711,
"event": "replay_end",
"replay_index": 2
}

Do not impose an arbitrary replay limit unless configuration explicitly requests one.

---

# 28. MICROPHONE ACTIVITY INDICATOR

Place a compact microphone indicator next to Replay.

It should communicate that audio capture is functioning.

Display something like:

🎙 + live input meter/waveform

Possible states:

Normal:
microphone + changing signal

Silence:
microphone + low/flat signal

Error:
microphone + warning indication

Do NOT display live speech transcription.

Do NOT display recognized words.

Do NOT provide participant feedback based on ASR.

The indicator represents AUDIO CAPTURE, not transcription.

---

# 29. MICROPHONE FAILURE

Before participant mode begins:

validate that the configured microphone capture stream works.

If audio recording is enabled but no working microphone is available:

do not silently start the session.

Show the issue on the setup screen.

Require either:

* fixing the microphone,
  or
* an explicit researcher override.

If overridden, log this fact.

---

# 30. AUDIO RECORDING

Record high-quality local audio.

Preferred:

WAV
48 kHz if practical
mono unless there is a reason for stereo.

Do not encode audio as base64 in JSON.

Do not upload audio automatically.

Use local storage.

Audio should be recorded continuously across the relevant session or trial period.

---

# 31. AUDIO TIME ALIGNMENT

Audio timing must be explicitly anchored to the experiment clock.

Store metadata such as:

{
"audio_start_t_ms": 10432.291,
"sample_rate_hz": 48000,
"channels": 1
}

This allows mapping audio sample number n approximately to:

t_ms =
audio_start_t_ms
+
(n / sample_rate_hz) * 1000

Preserve enough metadata for later word-level speech/pen alignment.

---

# 32. ONE AUTHORITATIVE EXPERIMENT CLOCK

This is CRITICAL.

Use ONE high-resolution MONOTONIC SESSION CLOCK.

The authoritative timestamp must NOT reset between trials.

Use a name such as:

t_ms

Definition:

milliseconds since session monotonic clock origin.

Example:

session begins:

t_ms = 0

Trial 1 may begin:

t_ms = 10421.201

Trial 2 may begin:

t_ms = 182401.429

Do NOT reset t_ms to zero when a new trial begins.

---

# 33. OPTIONAL TRIAL-RELATIVE TIME

For convenience, individual events may additionally contain:

trial_t_ms

This is relative to the beginning of the current trial.

Example:

{
"t_ms": 184203.210,
"trial_t_ms": 0.000,
"event": "trial_start"
}

The authoritative time is:

t_ms.

trial_t_ms is only a derived convenience field.

---

# 34. WALL CLOCK

Also record wall-clock time for provenance.

Example:

2026-09-04T14:10:12.201+03:00

But wall-clock timestamps must NOT replace the monotonic clock for behavioral synchronization.

---

# 35. TIMESTAMP UNITS

Use milliseconds consistently.

Field names should make this explicit.

Prefer:

t_ms
start_t_ms
end_t_ms
created_t_ms

Do not use ambiguous fields such as:

"t": 1.237

where it is unclear whether this means seconds or milliseconds.

---

# 36. SAMPLE SEQUENCE IDs

Every raw pen sample should receive a monotonically increasing sample sequence number.

Example:

{
"sample_seq": 10082,
"t_ms": 184991.422,
...
}

Structured stroke, laser, or erase records should reference sequence IDs rather than relying on implicit JSONL line numbers.

Example:

{
"sample_seq_start": 9921,
"sample_seq_end": 10084
}

Define whether these endpoints are inclusive.

Use the same convention throughout the application and document it.

---

# 37. RAW PEN SAMPLE

Example:

{
"sample_seq": 10082,

"t_ms": 184991.422,
"trial_t_ms": 794.201,

"screen_x_px": 1427.1,
"screen_y_px": 690.4,

"workspace_x_px": 812.2,
"workspace_y_px": 451.5,

"workspace_x_norm": 0.637,
"workspace_y_norm": 0.492,

"pressure": 0.37,

"tilt_x": -12,
"tilt_y": 4,

"contact": true,
"hover": false,

"tool": "ink",

"color_id": "black",
"thickness_id": "medium",

"page_id": "page_02",
"stroke_id": "stroke_0014"
}

Use null for unsupported/unavailable values.

Never fabricate device data.

---

# 38. COORDINATE SYSTEMS

Store:

1. screen/display coordinates,
2. workspace-local coordinates,
3. normalized workspace coordinates.

Normalized coordinates:

workspace_x_norm =
workspace_x_px / workspace_width_px

workspace_y_norm =
workspace_y_px / workspace_height_px

Store workspace geometry whenever layout changes.

This matters because the appearance of the page strip may resize the workspace.

---

# 39. LAYOUT CHANGE EVENTS

Whenever page navigation appears or workspace geometry changes, record enough geometry to reconstruct coordinate mapping.

Example event:

{
"t_ms": 244302.120,
"event": "workspace_layout_changed",

"workspace_bounds_before": {
"x": ...,
"y": ...,
"width": ...,
"height": ...
},

"workspace_bounds_after": {
"x": ...,
"y": ...,
"width": ...,
"height": ...
},

"page_strip_bounds": {
...
},

"tool_palette_bounds": {
...
}
}

---

# 40. OBSERVATION VS INTERPRETATION

The live application records OBSERVABLE behavior.

Do NOT automatically annotate:

* Figure,
* Ground,
* Path,
* Manner,
* Talmy categories,
* gesture,
* tracing,
* pointing,
* complementarity,
* redundancy,
* semantic relations,
* motion semantics.

These belong to later analysis.

The collection application should remain theoretically neutral.

---

# 41. STORAGE ARCHITECTURE

DO NOT USE A DATABASE.

Do not use:

* SQLite,
* PostgreSQL,
* MySQL,
* IndexedDB as authoritative storage,
* opaque proprietary database files.

Use a FILE-ONLY LOCAL STORAGE architecture.

The raw dataset must remain understandable without running the application.

Use:

* JSON for metadata,
* JSONL for chronological/high-frequency streams,
* WAV for audio,
* MP4 where needed,
* SVG for vector page snapshots,
* PNG for raster previews,
* later Parquet/CSV for derived analysis.

No cloud upload by default.

---

# 42. RAW VS DERIVED DATA

Maintain a strict separation:

RAW OBSERVATION
↓
PROCESSING
↓
DERIVED DATA
↓
ANNOTATION
↓
ANALYSIS

Raw files must not be destructively modified during transcription, coding, preprocessing, or analysis.

---

# 43. TOP-LEVEL DATA DIRECTORY

Use:

data/
├── participants/
├── stimuli/
├── exports/
└── logs/

Participant data lives under:

data/participants/

---

# 44. ID CONVENTIONS

Participant IDs:

P001
P002
P003

Session IDs:

S001
S002

Trial IDs:

T001
T002

Stimulus IDs:

ST001
ST002

Page IDs:

page_01
page_02

Stroke IDs:

stroke_0001
stroke_0002

Laser IDs:

laser_0001

Erase IDs:

erase_0001

Undo IDs:

undo_0001

Avoid participant names in paths.

---

# 45. COMPLETE SESSION DIRECTORY

Use approximately:

data/
└── participants/
└── P007/
└── S001/
├── session.json
├── session_events.jsonl
│
├── audio/
│   ├── session.wav
│   └── audio_metadata.json
│
├── recordings/
│   ├── screen.mp4
│   └── camera.mp4
│
├── trials/
│   ├── T001/
│   │   ├── trial.json
│   │   ├── events.jsonl
│   │   ├── pen_raw.jsonl
│   │   ├── strokes.jsonl
│   │   ├── laser.jsonl
│   │   ├── eraser.jsonl
│   │   ├── undo.jsonl
│   │   ├── pages.json
│   │   ├── final_state.json
│   │   │
│   │   ├── pages/
│   │   │   ├── page_01.svg
│   │   │   ├── page_01.png
│   │   │   ├── page_02.svg
│   │   │   └── page_02.png
│   │   │
│   │   └── diagnostics/
│   │       └── trial_summary.json
│   │
│   ├── T002/
│   │   └── ...
│   │
│   └── ...
│
└── derived/
├── transcript/
│   ├── transcript_segments.json
│   ├── transcript_words.json
│   └── transcript.txt
│
├── processed/
│   ├── trials.parquet
│   ├── pen_samples.parquet
│   ├── strokes.parquet
│   └── laser_episodes.parquet
│
└── annotations/
└── README.md

Not every derived file needs to be generated by V1.

Reserve this structure for later work.

---

# 46. STIMULUS DIRECTORY

Use:

data/
└── stimuli/
├── stimuli.json
└── videos/
├── ST001.mp4
├── ST002.mp4
└── ST003.mp4

All stimulus paths stored in manifests/metadata should be relative to:

data/

For example:

"stimulus_file": "stimuli/videos/ST021.mp4"

Do NOT use fragile traversal paths such as:

../../../stimuli/videos/ST021.mp4

---

# 47. STIMULUS MANIFEST

Example:

[
{
"stimulus_id": "ST001",
"file": "stimuli/videos/ST001.mp4",
"category": "atomic"
},
{
"stimulus_id": "ST002",
"file": "stimuli/videos/ST002.mp4",
"category": "spatial"
}
]

Do not hard-code filenames into UI components.

---

# 48. TRIAL ORDER

Support an abstraction capable of:

* fixed order,
* randomized order,
* future counterbalancing.

For V1, implement the simplest appropriate version.

Always store the ACTUAL presented order in session.json.

If randomization is used, store the seed.

---

# 49. SESSION.JSON

Example:

{
"schema_version": "1.0",

"participant_id": "P007",
"session_id": "S001",
"handedness": "right",

"status": "in_progress",

"started_at_wallclock": "2026-09-04T14:10:12.201+03:00",
"completed_at_wallclock": null,

"app_version": "0.1.0",
"git_commit": "abc123",

"platform": {
"os": "Windows",
"screen_width_px": 3840,
"screen_height_px": 2160,
"device_pixel_ratio": 1.0
},

"pen_device": {
"name": "...",
"pressure_supported": true,
"tilt_supported": true,
"hover_supported": true
},

"audio_device": {
"name": "...",
"sample_rate_hz": 48000,
"channels": 1
},

"stimulus_order": [
"ST014",
"ST003",
"ST021"
],

"current_trial": "T004",

"completed_trials": [
"T001",
"T002",
"T003"
]
}

Use atomic file writes.

Do not continuously rewrite this file for high-frequency events.

---

# 50. SESSION_EVENTS.JSONL

Use:

session_events.jsonl

Example:

{"t_ms":0.000,"event":"session_start"}
{"t_ms":1120.000,"event":"microphone_check","status":"ok"}
{"t_ms":3402.000,"event":"pen_check","status":"ok"}
{"t_ms":14018.000,"event":"trial_enter","trial_id":"T001"}
{"t_ms":242771.000,"event":"trial_complete","trial_id":"T001"}

Append-only.

---

# 51. TRIAL.JSON

Example:

{
"schema_version": "1.0",

"participant_id": "P007",
"session_id": "S001",
"trial_id": "T004",

"trial_index": 4,

"stimulus_id": "ST021",
"stimulus_file": "stimuli/videos/ST021.mp4",

"status": "in_progress",

"start_t_ms": 184203.210,
"end_t_ms": null,

"mandatory_views_required": 2,
"mandatory_views_completed": 2,

"replay_count": 1,

"page_count": 2,
"active_page_id": "page_02",

"handedness": "right",

"audio_status": "recording",

"errors": []
}

On completion:

"status": "completed"

and set:

end_t_ms.

Use atomic file replacement.

---

# 52. EVENTS.JSONL

Each trial contains:

events.jsonl

This stores discrete trial events.

Example:

{"t_ms":184203.210,"trial_t_ms":0.000,"event":"trial_start","trial_id":"T004"}

{"t_ms":185407.210,"trial_t_ms":1204.000,"event":"stimulus_view_requested","view":1}

{"t_ms":185440.210,"trial_t_ms":1237.000,"event":"stimulus_playback_start","view":1}

{"t_ms":189114.210,"trial_t_ms":4911.000,"event":"stimulus_playback_end","view":1}

{"t_ms":192605.210,"trial_t_ms":8402.000,"event":"stimulus_view_requested","view":2}

{"t_ms":198804.210,"trial_t_ms":14601.000,"event":"workspace_enter"}

{"t_ms":198813.210,"trial_t_ms":14610.000,"event":"tool_selected","tool":"ink"}

{"t_ms":213017.210,"trial_t_ms":28814.000,"event":"tool_selected","tool":"laser"}

{"t_ms":219023.210,"trial_t_ms":34820.000,"event":"replay_requested","replay_index":1}

{"t_ms":226420.210,"trial_t_ms":42217.000,"event":"page_create","page_id":"page_02"}

{"t_ms":226422.210,"trial_t_ms":42219.000,"event":"page_switch","from":"page_01","to":"page_02"}

{"t_ms":247113.210,"trial_t_ms":62910.000,"event":"trial_done_requested"}

Use append-only writing.

---

# 53. PEN_RAW.JSONL

Store:

pen_raw.jsonl

This is the authoritative high-frequency pen stream.

Every sample gets:

sample_seq.

Capture hover and contact.

Use append-only writing.

---

# 54. STROKES.JSONL

Store Ink stroke summaries.

Example:

{
"stroke_id": "stroke_0014",
"page_id": "page_02",

"color_id": "red",
"thickness_id": "medium",
"width_px": 4,

"start_t_ms": 225211.000,
"end_t_ms": 226904.000,

"sample_seq_start": 9921,
"sample_seq_end": 10084
}

Raw pen points remain in pen_raw.jsonl.

---

# 55. LASER.JSONL

Example:

{
"laser_id": "laser_0008",
"page_id": "page_01",

"start_t_ms": 231210.000,
"end_t_ms": 233918.000,

"sample_seq_start": 14320,
"sample_seq_end": 14691
}

---

# 56. ERASER.JSONL

Example:

{
"erase_id": "erase_0003",
"page_id": "page_01",

"start_t_ms": 244021.000,
"end_t_ms": 244881.000,

"sample_seq_start": 18041,
"sample_seq_end": 18192,

"affected_strokes": [
"stroke_0007",
"stroke_0008"
]
}

Use null if affected strokes cannot be reliably determined.

---

# 57. UNDO.JSONL

Example:

{
"undo_id": "undo_0003",
"t_ms": 247411.000,
"target_type": "erase",
"target_id": "erase_0003"
}

---

# 58. PAGES.JSON

Example:

{
"pages": [
{
"page_id": "page_01",
"created_t_ms": 198804.210,
"creation_index": 1
},
{
"page_id": "page_02",
"created_t_ms": 226420.210,
"creation_index": 2
}
]
}

Page switch history stays in events.jsonl.

---

# 59. FINAL PAGE SNAPSHOTS

At trial completion save:

pages/
├── page_01.svg
├── page_01.png
├── page_02.svg
└── page_02.png

SVG preserves vector appearance.

PNG supports:

* quick visual inspection,
* thumbnails,
* later annotation tools,
* reporting.

These are derived representations.

They are not replacements for raw event history.

---

# 60. FINAL_STATE.JSON

Example:

{
"active_page_id": "page_02",
"page_count": 2,

"visible_strokes": {
"page_01": [
"stroke_0001",
"stroke_0002"
],
"page_02": [
"stroke_0010",
"stroke_0012"
]
}
}

This provides the final visible state without requiring complete event replay.

Raw history remains authoritative.

---

# 61. AUDIO FILES

Use:

audio/
├── session.wav
└── audio_metadata.json

Example audio_metadata.json:

{
"audio_start_t_ms": 8321.553,
"sample_rate_hz": 48000,
"channels": 1,
"device": "...",
"format": "wav"
}

If trial-level audio files later prove easier, architecture may support them, but retain explicit synchronization metadata.

---

# 62. SCREEN / CAMERA RECORDINGS

Reserve:

recordings/
├── screen.mp4
└── camera.mp4

These may be produced externally rather than by the application.

Do NOT treat screen video as a replacement for structured logging.

If external recording is used, preserve synchronization metadata where practical.

---

# 63. TRANSCRIPTION

Do not require live ASR.

Transcription happens AFTER data collection.

Reserve:

derived/
└── transcript/
├── transcript_segments.json
├── transcript_words.json
└── transcript.txt

Example:

{
"segment_id": "segment_001",
"start_t_ms": 221320.000,
"end_t_ms": 223310.000,
"text": "and then this moves across"
}

Word-level:

{
"word_id": "word_0042",
"word": "moves",
"start_t_ms": 222420.000,
"end_t_ms": 222830.000
}

---

# 64. ANALYSIS EXPORTS

Later preprocessing may create:

derived/
└── processed/
├── trials.parquet
├── pen_samples.parquet
├── strokes.parquet
├── laser_episodes.parquet
└── transcript_words.parquet

The live collection application does NOT need to produce Parquet.

Raw files remain authoritative.

---

# 65. CRASH-SAFE DATA COLLECTION

Do not wait until the participant presses Done before writing data.

When trial starts:

immediately create:

T004/
├── trial.json
├── events.jsonl
├── pen_raw.jsonl
├── strokes.jsonl
├── laser.jsonl
├── eraser.jsonl
├── undo.jsonl
└── pages.json

Set:

"status": "in_progress"

Write/append during the trial.

On Done:

1. safely finish active interaction,
2. flush buffers,
3. close append streams,
4. generate final page SVG/PNG,
5. write final_state.json,
6. atomically update trial.json,
7. set status to completed,
8. only then proceed.

---

# 66. ATOMIC METADATA WRITES

For mutable JSON files such as:

session.json
trial.json
pages.json
final_state.json

use atomic writing where practical.

Example:

write:

trial.json.tmp

flush/fsync/close

then atomically replace:

trial.json

Chronological behavioral streams should remain append-only.

---

# 67. CRASH RECOVERY

At application startup scan:

data/participants/*/*/trials/*/trial.json

Find trials where:

"status": "in_progress"

Provide researcher recovery handling.

The system should be able to identify incomplete trials without any database.

Do not automatically overwrite unfinished sessions.

---

# 68. COMPLETED DATA PROTECTION

Completed trial directories should not be silently overwritten.

If a participant/session/trial ID already exists:

* warn,
* generate a new session as appropriate,
* or require explicit researcher action.

Protect prior research data.

---

# 69. CINTIQ FULLSCREEN

Participant mode should run fullscreen on the Wacom Cintiq.

Handle:

* Windows scaling,
* high DPI,
* display mapping,
* pointer coordinate consistency,
* pressure,
* hover,
* tilt if available.

Avoid accidental interaction with the Windows desktop where practical.

---

# 70. PEN DIAGNOSTIC

Before session start, perform a lightweight pen capability check.

It should determine whether possible:

* hover,
* contact,
* pressure,
* tilt,
* coordinate mapping.

The V1 setup UI only needs to show something simple such as:

Pen ✓

Developer diagnostics may expose detailed values behind a debug mode.

---

# 71. DEVELOPER DEBUG MODE

A developer-only diagnostic mode may show:

* pen position,
* workspace coordinates,
* normalized coordinates,
* pressure,
* tilt,
* hover/contact,
* sample rate,
* sample sequence,
* current page,
* current stroke,
* selected tool,
* current color,
* current thickness,
* microphone level,
* timestamps.

This mode must never appear during an actual participant session.

---

# 72. PERFORMANCE

Pen rendering must feel immediate.

Do not allow logging/disk writing to produce visible pen lag.

Separate where appropriate:

* input acquisition,
* visual rendering,
* event buffering,
* disk persistence.

Timestamp data as close to actual input receipt as possible.

Do not unnecessarily downsample raw pen input unless technical constraints require it.

If downsampling becomes necessary, document the strategy.

---

# 73. APPLICATION ARCHITECTURE

Keep responsibilities separated.

Suggested conceptual modules:

ExperimentController
TrialStateMachine
StimulusPlayer
CanvasRenderer
PenInputManager
ToolManager
InkStyleManager
PageManager
AudioRecorder
MicLevelMonitor
EventLogger
FileDataStore
SessionManager
RecoveryManager

Do NOT create:

DatabaseManager.

There is no database.

---

# 74. FILE DATA STORE RESPONSIBILITIES

FileDataStore should manage:

* directory creation,
* safe path construction,
* atomic JSON writes,
* JSONL append writers,
* flushing,
* file closing,
* recovery scanning,
* completed-data protection,
* dataset-root-relative paths.

Do not allow UI components to invent arbitrary output paths independently.

---

# 75. TRIAL STATE MACHINE

Use an explicit state machine.

Suggested:

READY_VIEW_1
PLAYING_VIEW_1
READY_VIEW_2
PLAYING_VIEW_2
READY_RECONSTRUCTION
RECONSTRUCTION
FINALIZING
COMPLETE
ERROR

Replay should NOT force the entire trial out of RECONSTRUCTION.

Treat replay playback as a concurrent media activity while reconstruction remains active.

This allows speech/pen interaction during replay.

---

# 76. DONE ACTION

Done should be clearly visible but not located where a normal drawing gesture could accidentally activate it.

One deliberate activation should be sufficient.

Avoid repeated disruptive confirmation dialogs unless testing demonstrates they are necessary.

When Done is activated:

* close active stroke/laser/eraser episode,
* append final event,
* flush logs,
* save current page state,
* generate page snapshots,
* update metadata,
* mark trial completed.

Do not move to next trial before finalization succeeds.

---

# 77. TRIAL TRANSITION

After a completed trial, provide a minimal transition before the next trial.

Do not expose previous research logs or analytics to participants.

Proceed cleanly to the next:

READY_VIEW_1

state.

---

# 78. SESSION COMPLETE

At the end of the last trial, show a minimal participant completion screen.

Example:

Session complete.

Thank you.

Do not show research data, filenames, or internal diagnostics.

---

# 79. CONFIGURATION

Use a human-readable configuration file.

Example:

config/experiment.json

Possible settings:

{
"mandatory_video_views": 2,
"allow_replay": true,
"laser_fade_ms": 750,

"ink": {
"colors": [
"black",
"blue",
"red",
"green"
],
"default_color": "black",

```
"thicknesses": {
  "thin": 2,
  "medium": 4,
  "thick": 7
},

"default_thickness": "medium"
```

},

"audio_enabled": true,
"fullscreen": true
}

The actual width values may be adjusted after testing on the Cintiq.

Do not expose this configuration file to participants.

---

# 80. VERSIONING / PROVENANCE

Each session should record:

* schema_version,
* app_version,
* git commit if available,
* stimulus manifest/version,
* experiment configuration,
* operating system,
* display resolution,
* device pixel ratio,
* detected pen capabilities,
* microphone device,
* audio sample rate,
* handedness,
* actual trial order.

This is important for research reproducibility.

---

# 81. PRIVACY

Use local storage by default.

Do not upload participant data automatically.

Do not include third-party analytics.

Avoid unnecessary personal information.

Use pseudonymous participant IDs.

---

# 82. DATA_SCHEMA.MD

Create:

DATA_SCHEMA.md

Document every output file and field.

For each field include:

* name,
* type,
* unit,
* meaning,
* raw or derived,
* nullable conditions,
* example if useful.

Example:

t_ms

type:
float

unit:
milliseconds

meaning:
milliseconds from the session monotonic clock origin

status:
raw

---

# 83. FILE_STRUCTURE.MD

Create:

FILE_STRUCTURE.md

Document:

* top-level directory,
* participant IDs,
* session IDs,
* trial IDs,
* stimulus IDs,
* page IDs,
* stroke IDs,
* laser IDs,
* erase IDs,
* undo IDs,
* derived-data directories.

Include a complete example directory tree.

---

# 84. README.MD

README should explain:

* prerequisites,
* installation,
* development startup,
* experiment startup,
* Windows requirements,
* Wacom Cintiq setup,
* microphone setup,
* pen capability check,
* adding stimuli,
* choosing output location,
* starting a participant session,
* where data is stored,
* crash recovery,
* validation,
* file meanings.

---

# 85. DATA INTEGRITY VALIDATOR

Create a utility such as:

validate_session

It should scan one session and detect:

* missing files,
* malformed JSON,
* malformed JSONL,
* incomplete trial,
* timestamp regression,
* duplicate sample_seq,
* invalid sample sequence references,
* missing page IDs,
* references to nonexistent strokes,
* missing audio,
* incorrect mandatory-view counts,
* trial status inconsistencies,
* missing final snapshots.

Example:

Session P007/S001

T001  OK
T002  OK
T003  WARNING — page_02.png missing
T004  INCOMPLETE

---

# 86. VALIDATION TESTS

Create automated tests for:

* folder generation,
* ID generation,
* dataset-root-relative stimulus paths,
* valid session.json,
* valid trial.json,
* parseable JSONL,
* monotonically non-decreasing t_ms,
* monotonically increasing sample_seq,
* correct trial_t_ms derivation,
* correct page IDs,
* stroke IDs,
* color stored correctly,
* thickness stored correctly,
* hover capture,
* pressure capture,
* mandatory views exactly 2,
* participant-triggered playback,
* correct replay count,
* drawing remains active during Replay,
* Laser history preserved,
* Eraser history preserved,
* Undo history preserved,
* page strip hidden for one page,
* page strip shown after Page 2,
* handedness layout correct,
* page thumbnail updates,
* microphone failure detected,
* audio anchor metadata saved,
* crash leaves trial in_progress,
* restart discovers incomplete trial,
* completed trial protected from overwrite.

---

# 87. V1 DEVELOPMENT PRIORITIES

Implement incrementally.

## Phase 1 — Foundation

Implement:

* project scaffold,
* file-only persistence,
* session directory generation,
* minimal setup screen,
* handedness,
* participant fullscreen mode,
* trial state machine,
* View 1,
* View 2,
* reconstruction workspace,
* Ink,
* pen raw logging,
* session clock,
* sample_seq,
* events.jsonl,
* session.json,
* trial.json,
* Done.

## Phase 2 — Audio + Replay

Add:

* microphone recording,
* microphone level indicator,
* audio synchronization metadata,
* Replay,
* temporary replay panel,
* drawing while replaying,
* accurate media playback events.

## Phase 3 — Expressive Tools

Add:

* Black/Blue/Red/Green Ink colors,
* Thin/Medium/Thick stroke sizes,
* Laser,
* Eraser,
* Undo.

## Phase 4 — Pages

Add:

* New Page,
* conditional page strip,
* handedness-aware page location,
* live thumbnails,
* page switching,
* SVG/PNG snapshots.

## Phase 5 — Reliability

Add:

* crash recovery,
* atomic file writing,
* validation utility,
* diagnostics,
* automated tests,
* schema documentation.

---

# 88. FIRST FUNCTIONAL VERTICAL SLICE

The first meaningful vertical slice should support:

minimal setup
→ participant ID
→ session ID
→ handedness
→ create session directory
→ fullscreen participant mode
→ Trial 1
→ participant starts View 1
→ video plays
→ participant starts View 2
→ video plays
→ Begin Reconstruction
→ Ink drawing
→ raw pen JSONL logging
→ microphone recording
→ microphone activity indicator
→ optional Replay
→ Done
→ completed trial directory.

The output should already be manually inspectable.

Do not wait for all advanced features before validating the basic acquisition pipeline.

---

# 89. PARTICIPANT VISUAL STYLE

The interface should be:

* minimal,
* calm,
* neutral,
* tablet-like,
* pen-first,
* spacious,
* high-contrast enough for clarity,
* appropriate for a large Cintiq display.

Avoid:

* dashboard styling,
* excessive cards,
* gamification,
* decorative animation,
* unnecessary gradients,
* desktop toolbar chrome,
* dense labels,
* unnecessary borders.

The participant should primarily focus on:

THE MOTION

THEIR SPEECH

THEIR PEN

---

# 90. IMPORTANT UX PRINCIPLE

The tool palette should feel like an instrument attached to the drawing surface, not a collection of application buttons.

In particular, do not make:

INK
LASER
ERASER
UNDO
NEW PAGE

look like five equally sized rectangular buttons.

Use icon-first tablet-tool visual language.

---

# 91. RESEARCH PRINCIPLE

Participant-facing simplicity and research-facing instrumentation must coexist.

Visually:

simple.

Underneath:

rich.

The participant does not need to understand the logging system.

The researcher later needs to reconstruct the session precisely.

---

# 92. WHAT NOT TO BUILD IN V1

Do NOT build:

* full experimenter dashboard,
* participant database,
* web backend,
* authentication,
* cloud synchronization,
* annotation system,
* transcript editor,
* Talmy coding interface,
* gesture coding interface,
* analysis dashboard,
* automated semantic classification,
* sophisticated reporting system.

Those are separate future artifacts if needed.

---

# 93. FUTURE EXPERIMENTER TOOLING

Design architecture so a future experimenter interface could later show:

* session status,
* current trial,
* audio health,
* pen health,
* storage health,
* trial flags,
* emergency save,
* crash recovery,
* participant/session browsing.

But DO NOT build this full interface now.

V1 focuses on the participant experience and data collection reliability.

---

# 94. TECHNICAL DECISIONS BEFORE LARGE IMPLEMENTATION

Before writing large amounts of code:

1. inspect the repository,
2. identify the existing stack,
3. determine whether to preserve or adapt the current stack,
4. investigate the best Windows/Wacom Cintiq pen input approach,
5. determine how pressure will be obtained,
6. determine how hover will be obtained,
7. determine how tilt will be obtained,
8. determine how audio will be captured,
9. determine how the monotonic session clock will be implemented,
10. determine how video playback callbacks/events will be timestamped,
11. determine how pen rendering and logging will remain low latency,
12. propose the module/component architecture,
13. propose exact file-writing strategy,
14. show the expected generated directory tree,
15. identify technical risks.

Do not silently choose a framework simply because it is fashionable.

The important factors are:

* Windows reliability,
* Cintiq pen support,
* low latency,
* local file access,
* reliable audio,
* precise logging.

---

# 95. DO NOT SILENTLY CHANGE CORE REQUIREMENTS

Do not silently introduce:

* database storage,
* cloud storage,
* mandatory autoplay,
* live ASR feedback,
* unrestricted drawing features,
* disabled drawing during Replay,
* default page sidebar,
* same-side tools/pages,
* ambiguous timestamps.

If a technical limitation requires changing a core design decision, explicitly explain the issue before making the architectural change.

---

# 96. EXPECTED EARLY TEST OUTPUT

After one early successful test session:

data/
└── participants/
└── P001/
└── S001/
├── session.json
├── session_events.jsonl
│
├── audio/
│   ├── session.wav
│   └── audio_metadata.json
│
├── recordings/
│
├── trials/
│   └── T001/
│       ├── trial.json
│       ├── events.jsonl
│       ├── pen_raw.jsonl
│       ├── strokes.jsonl
│       ├── laser.jsonl
│       ├── eraser.jsonl
│       ├── undo.jsonl
│       ├── pages.json
│       ├── final_state.json
│       │
│       └── pages/
│           ├── page_01.svg
│           └── page_01.png
│
└── derived/
├── transcript/
├── processed/
└── annotations/

This directory structure is part of the research-data contract, not an incidental implementation detail.

---

# 97. FINAL PRODUCT PRINCIPLE

The participant experience should feel approximately like:

WATCH

→

WATCH AGAIN

→

SPEAK + DRAW NATURALLY

The application should feel almost invisible during the reconstruction task.

The research dataset underneath should be:

* high-resolution,
* synchronized,
* recoverable,
* transparent,
* human-readable,
* reproducible,
* analysis-ready.

---

# 98. STARTING INSTRUCTION

Start by inspecting the existing repository.

Before implementing the full application, report:

1. current repository/technology structure,
2. recommended architecture,
3. recommended Wacom/Cintiq pen input strategy,
4. expected support for hover/pressure/tilt,
5. audio capture strategy,
6. monotonic clock strategy,
7. participant screen/state map,
8. file persistence design,
9. expected generated directory tree,
10. major technical risks.

Then implement the first functional vertical slice.

Prioritize getting the participant-facing interaction and raw synchronized acquisition working correctly before building secondary tooling or visual polish.
