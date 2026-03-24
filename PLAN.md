# PEEQ — Origins, Insights, and Forward Plan

## Background

PEEQ is a Pure Data step sequencer written circa 2003, built for live performance on underpowered hardware. The name likely derives from the author's handle. It was written at a time when live laptop performance was a nascent scene (livepa.org era) and consumer hardware was genuinely limited.

The key design pressures that shaped it:
- Had to run on a very slow laptop
- Needed to work for live performance in high-energy settings (breakdancing club)
- Inspired by FruityLoops 1.0's numpad-driven pattern switching, which kept the playhead in the right position relative to the new pattern — an early form of what Ableton would later call "legato mode"
- Built around the principle that the performer should never need to touch a mouse

## What PEEQ Got Right

### Keyboard-First Pattern Entry

The most distinctive feature is using the QWERTY keyboard as a step grid:

```
Q W E R T Y U I   → steps 1–8
A S D F G H J K   → steps 9–16
```

With `[` / `]` to move row focus, you can compose an entire pattern without touching a mouse. This is rare even now. Most software grid sequencers are entirely mouse-driven.

### Macro Pattern Shortcuts

Keys 1–4 instantly load common rhythmic patterns:
- `1` → quarter notes (steps 1, 5, 9, 13)
- `2` → eighth notes (all odd steps)
- `3` → 2 & 4 snare (steps 5, 13)
- `4` → user-definable

These aren't cosmetic — they mean you can lay down a kick, snare, and hi-hat groove in under five keystrokes while the sequencer is running.

### Legato-Mode Pattern Switching

Numpad `+` / `-` switch patterns without resetting the clock. The new pattern picks up at whatever beat position the old one left off. This is the core live-performance insight: pattern changes feel musical, not mechanical. FruityLoops had this; most other tools of the era did not.

### Dual Edit/Play Model

`curr_edit_patt` vs. `pattern` — you can edit pattern 3 while pattern 1 is playing. Chase mode optionally forces the editor to follow playback. This asymmetry is powerful: you prepare your next section while the audience hears the current one.

## Architecture Summary (Pure Data)

```
peeq.pd          — top-level UI, transport, pattern bank
  row.pd         — single sequencer row (16 steps, note, channel, mute)
    rowlogic.pd  — MIDI note output, array read/write
```

Pattern data stored as float arrays (`$1-seqdata`): 8 patterns × 8 rows × 16 steps = 1024 values total, addressed by offset math. Clock driven by `metro` at `15000 / BPM` ms. All rows share the same clock tick.

## Options for Moving Forward

### Option A: Port to Web (Vanilla JS + Web MIDI)

Rewrite PEEQ as a browser app using the same architecture:
- Web MIDI API for note output (same as tidal-arranger)
- HTML canvas or CSS grid for the step buttons
- `AudioContext` + `setTimeout`-based clock (or `SharedArrayBuffer` worker for tighter timing)
- Same keyboard bindings, same macro system

**Pros:** Modern, shareable, runs anywhere, no Pure Data dependency
**Cons:** Web timing is less reliable than PD's metro; requires more code

### Option B: WebPD / Run As-Is

[WebPD](https://github.com/sebpiq/WebPd) can compile Pure Data patches to Web Audio API. PEEQ's patches are old but use only core PD objects, so compatibility is plausible.

**Pros:** Zero rewrite, preserves exact behavior
**Cons:** WebPD maturity is uncertain; debugging compiled patches is hard; no easy integration with other web projects

### Option C: Live Keyboard Layer for Tidal Arranger (Preferred Near-Term)

The most immediately valuable path: bring PEEQ's keyboard-driven philosophy into `tidal-arranger`. Rather than rewriting a drum machine, add a **live performance layer** to the existing arranger that lets you trigger, mute, and compose Tidal patterns using keyboard shortcuts while the transport runs.

This is the spiritual successor to PEEQ's core insight applied to a more expressive language.

---

## Option C: Tidal Arranger Live Keyboard Layer

### Core Concept

While `tidal-arranger` plays back a pre-authored arrangement, a parallel **live mode** lets you:
- Instantly trigger named patterns with single keypresses
- Mute/unmute tracks in real time
- Queue pattern changes to land on the next bar boundary (legato)
- Build up a loop live, then optionally save it as a section

Think of it as PEEQ's numpad philosophy applied to a Tidal pattern vocabulary.

### Keyboard Layout

Mirror PEEQ's layout philosophy — mnemonic, clustered, never needing the mouse:

```
Pattern triggers (top row):
  1 2 3 4 5 6 7 8   → trigger pattern slots A–H (kick, snare, hh, bass, lead, etc.)

Mute toggles (home row with modifier):
  Shift+1..8        → toggle mute on track 1–8

Transport:
  Space             → play/stop (resets to bar 0 on stop, resumes in place on play)
  Enter             → advance to next section (queued, lands on bar boundary)
  Backspace         → go back one section

Pattern slot editing (when in edit mode):
  Tab               → enter/exit edit mode for currently focused slot
  Q W E R T Y U I  → toggle steps 1–8 (if slot is a step pattern)
  A S D F G H J K  → toggle steps 9–16

Pattern macros (when in edit mode):
  F1                → quarter note grid
  F2                → eighth note grid
  F3                → offbeat (2 & 4)
  F4                → clear
```

### Pattern Slot Model

Each slot holds a named Tidal pattern (or hard-coded step pattern). Slots are pre-loaded from the current arrangement or typed inline:

```
slot A: s "bd ~ ~ ~  bd ~ ~ ~  bd ~ ~ ~  bd ~ ~ ~"
slot B: s "~ ~ cp ~  ~ ~ cp ~  ~ ~ cp ~  ~ ~ cp ~"
slot C: s "hh*8"
slot D: n "0 7 0 5" # "bass" | sustain 0.5
```

When you press `1`, slot A fires (or queues to next bar boundary). When you press it again, it stops. This is exactly PEEQ's numpad behavior but for Tidal syntax.

### Legato / Quantized Switching

All pattern changes are **bar-quantized** by default — a keypress queues the change, which fires at the next bar boundary. This is the FruityLoops insight that peeq preserved:

```
[currently on bar 3 of 4]
→ press 2 (slot B)
→ slot B queued (indicator lights up)
→ bar 4 plays out
→ bar 5: slot B activates, playhead continues from position 0 of new pattern
```

An "instant" mode (hold Alt) can bypass this for deliberate fills or one-shots.

### Integration with Existing Arranger

The live layer sits alongside the existing arrangement editor, not replacing it:

- **Arrange mode:** current behavior — edit and play back a written song
- **Live mode:** keyboard layer active, pattern slots displayed, arrangement auto-follows or pauses
- A "capture" function records live-triggered patterns back into the arrangement as a new section

### UI Sketch

```
┌─────────────────────────────────────────────────────────┐
│  LIVE  [●]  BPM: 120   BAR: 7   BEAT: 3                │
├──────┬──────┬──────┬──────┬──────┬──────┬──────┬───────┤
│  A   │  B   │  C   │  D   │  E   │  F   │  G   │  H    │
│ [1]  │ [2]  │ [3]  │ [4]  │ [5]  │ [6]  │ [7]  │ [8]   │
│ bd   │ cp   │ hh   │ bass │      │      │      │       │
│ ████ │      │ ████ │ ▓▓▓▓ │      │      │      │       │
└──────┴──────┴──────┴──────┴──────┴──────┴──────┴───────┘
```

Active slots highlighted, queued slots blinking, muted slots dimmed.

### Implementation Plan

#### Phase 1: Slot Engine
- [ ] Define `PatternSlot` data structure: `{ name, tidalSource, active, muted, queued }`
- [ ] Add 8 slot definitions to arranger state
- [ ] Keyboard handler: number keys 1–8 toggle slot active/queued
- [ ] Bar-boundary scheduler: on each bar tick, apply queued state changes
- [ ] Render slot strip as a fixed UI element (below or above existing editor)

#### Phase 2: Playback Integration
- [ ] Wire active slots into the MIDI scheduler (same path as existing pattern playback)
- [ ] Handle slot deactivation cleanly (let current note durations expire)
- [ ] Per-slot channel assignment (reuse tidal-arranger's existing MIDI channel model)

#### Phase 3: Live Polish
- [ ] Mute toggle (Shift+1–8)
- [ ] Alt+key for instant (non-quantized) trigger
- [ ] Visual beat indicator per slot (which step is playing)
- [ ] Slot name editing inline

#### Phase 4: Capture
- [ ] Record active slots over time as a sequence of events
- [ ] "Commit to arrangement" function: turn the live take into a `section` block in the existing song

---

## Option D: AI-Inferred Performance Templates (The Bigger Idea)

This is the most ambitious direction and deserves its own framing. It extends the abstraction axis that has been moving through the whole history of live music tools — from playing every note, to triggering clips, to conducting arrangements — and adds a new dimension: **the machine infers the structure from your playing**.

### The Abstraction Axis in Live Performance

Tools over time have let performers choose their level of abstraction:

```
← more note-level                    more structural →

  Live instrument  →  DAW arrange  →  Ableton session  →  Live coding  →  [AI conductor]
  (every note)        (pre-written)    (clip triggers)     (text macros)   (inferred templates)
```

The AI conductor mode is not replacing the others — it's a new position further right on the same axis, and crucially, **you can slide back left at any time**.

There's also a second axis that has emerged: the visual/spatial dimension (VJing, live stage control, video reactive to music) and performance style axes (live coder as performer, typing as gesture). A full live AI system would operate on all of these simultaneously, but the musical structure inference is the foundation.

### Phase 1: Live MIDI Input Analysis

The system takes a MIDI keyboard (or any instrument with MIDI out) as its primary input and runs continuous analysis in the background:

**Rhythmic extraction:**
- Quantize incoming note events (configurable tolerance, e.g. ±15ms to nearest 16th)
- Detect repeating rhythmic cells (motifs) over a rolling window of N bars
- Distinguish intent from human error: a motif played 4 times is a template candidate; 4 times then 3 times on the next cycle could be either a deliberate variation or a dropped note — confidence scoring, not hard decisions

**Harmonic extraction:**
- Detect key and mode from note distribution over time (chromagram analysis)
- Identify chord changes: cluster simultaneous or near-simultaneous notes into chord events
- Track chord progressions over multi-bar windows: `I → IV → V → I` type sequences
- Flag key changes (deliberate modulation vs. accidental out-of-key note)

**Structural extraction:**
- Multi-bar theme detection: does bar 3-4 repeat bar 1-2? Is there a verse/chorus alternation?
- Detect phrase boundaries (rests, downbeat emphasis, dynamic drops)
- Identify sections: `intro`, `A section`, `B section`, `breakdown`, `return`

**Expressive extraction:**
- Velocity patterns: accent profiles across a bar (strong 1, softer 2, accented 3, etc.)
- Timing micro-variation: does the performer consistently push the beat? Drag it? Only on certain notes?
- Dynamic arcs: crescendo/decrescendo over multi-bar phrases
- These become performance expression metadata attached to each template

### Phase 2: Template Library Accumulation

As inference runs, the system populates a template library — essentially an auto-generated Tidal/pattern vocabulary derived from what the performer actually played:

```
[template 003]  confidence: 0.91
type: melodic motif
length: 2 bars
tidal: n "0 ~ 3 ~ 5 7 ~ 3" # "piano"
key context: C minor
velocity profile: [90 ~ 70 ~ 85 95 ~ 75]
timing nudge: +8ms on beat 3 (performer characteristic)
occurrences: 7  (bars 4, 8, 12, 16, 20, 24, 28)
variations: 1  (bar 20: last note 5 instead of 3, possible intent)

[template 007]  confidence: 0.78
type: chord progression
length: 4 bars
chords: Cm → Ab → Eb → Bb
occurrences: 3
```

The Tidal representation is the natural output format since it's already the working language of the arranger. The AI's job is essentially to be a very good transcriptionist and pattern recognizer, outputting to the same notation system the performer would use manually.

### Phase 3: Conductor Mode

Once a meaningful template library exists (could be mid-performance, after 30-60 seconds of playing), the performer switches to **conductor mode**:

- The keyboard shortcuts from Option C become active
- Templates are assigned to slots automatically by type and confidence (highest-confidence melodic motif in slot 1, best rhythmic cell in slot 2, etc.)
- The performer can now trigger, mute, layer, and re-arrange the inferred templates using PEEQ-style keys
- The original live input stream continues to feed the inference engine — new templates accumulate, existing ones get refined, the library evolves during the performance

This creates a feedback loop: you play → machine learns → you conduct what the machine learned → the conducting itself generates new patterns → machine learns from those too.

### The Error/Intent Problem

This is the hardest and most interesting inference problem: a human playing 4 bars of a pattern and then accidentally playing 3 is subjectively different from deliberately playing 3 as a variation. But to a naive pattern matcher they look similar.

Some signals that help:
- **Repetition count:** 3× repeated exactly = high template confidence. 2× = plausible. 1× = exploratory/mistake
- **Position in phrase:** errors cluster near phrase starts (coming in late) and phrase ends (rushing)
- **Recovery behavior:** after a mistake, performers typically repeat the correct version; after a variation, they continue
- **Velocity consistency:** mistakes often have lower velocity (hesitation) or higher (overcompensation)
- **Context:** if the performer is in "steady state" (same material for several bars), a variation is probably intentional; if they're still establishing a pattern, it's more likely an error

The system should not suppress variation — variation IS the performance. The goal is confidence scoring, not error correction. A template with variations is richer than one without: `n "0 ~ 3 ~ 5 7 ~ 3" | sometimes (# "ghost_note")` might capture the template + its real variation together.

### The Key/Chord Change Signal

This is valuable not just for transcription but as a **macro performance gesture**. If the AI detects a deliberate key change (e.g., a ii-V-I pivot into a new key), it can:
- Mark the transition point in the live session timeline
- Transpose all active templates to the new key automatically (if the performer wants)
- Or hold templates in the old key while new-key templates accumulate in the library
- Surface the key change as a section boundary in the arrangement

The performer can choose: follow the key change automatically, or control it explicitly (a keyboard shortcut to "commit key change now").

### Implementation Architecture

This system needs components that don't yet exist in tidal-arranger:

```
[MIDI keyboard input]
        ↓
[Web MIDI input handler]  ← already exists in tidal-arranger
        ↓
[Event buffer]            ← rolling window, N bars at current BPM
        ↓
[Analysis engine]
  ├── Quantizer           ← snap to grid with confidence
  ├── MotifDetector       ← sliding window repeat detection
  ├── HarmonicAnalyzer    ← chromagram → key/chord
  ├── StructureDetector   ← phrase/section boundary
  └── ExpressionCapture   ← velocity/timing profiles
        ↓
[Template library]        ← confidence-scored, Tidal-serializable
        ↓
[Conductor UI]            ← Option C keyboard layer, fed by inferred templates
        ↓
[Arranger / MIDI out]     ← existing tidal-arranger engine
```

The analysis engine is the new piece. Everything else either exists or is described in Option C.

**AI role:** The analysis engine could be:
- **Rule-based:** pattern matching, windowed autocorrelation, chromagram statistics — fast, deterministic, no model needed, good for rhythm and harmony
- **ML-assisted:** a small transformer or LSTM watching the event stream for longer-range structure, good for verse/chorus detection and phrase segmentation
- **LLM-in-the-loop (async):** periodically send the accumulated template library to an LLM and ask it to annotate, name, relate, or suggest variations — not in the real-time path, but as a background enrichment pass

The real-time path (quantizer → motif detector → harmonic analyzer) should be rule-based for latency reasons. The ML/LLM layers operate on already-detected templates, not on the raw event stream.

### Performance Model

The performer's journey through a show with this system:

```
0:00  Start playing freely. System is listening.
      Templates accumulate silently. Confidence scores climb.
      Performer is in "musician" mode — no AI layer yet active.

2:00  [Tab] → switch to Conductor mode.
      8 slots auto-populated from top-confidence templates.
      Performer can now trigger the templates they just played.
      Still playing live over the top.

4:00  Key change. AI detects it.
      New templates begin accumulating in new key.
      Old templates still available in conductor slots.
      [K] → "commit key change, transpose active slots"

6:00  Performer stops playing directly.
      Now conducting purely via keyboard: triggering, muting, re-arranging.
      Live input channel still open — they can drop back in at any time.

8:00  [Capture] → the last 2 minutes of conductor actions recorded as a section.
      Exported to tidal-arranger arrangement for later refinement.
```

---

## Relationship to PEEQ

PEEQ was a complete sequencer. The tidal-arranger live layer is one component of a larger system. But the lineage is direct:

| PEEQ | Tidal Live Layer |
|------|-----------------|
| Numpad +/- to switch patterns | Number keys 1–8 to trigger slots |
| Legato (playhead-preserving) switch | Bar-quantized switch |
| QWERTY step entry | QWERTY step entry (optional, for step-pattern slots) |
| Macros (quarter/eighth/snare) | Macros (F1–F4) |
| 8 rows × 8 patterns | 8 slots × configurable patterns |
| MIDI out via PD noteout | Web MIDI API |

The underlying philosophy is identical: the performer should be able to make musical decisions in real time without ever leaving the keyboard.

---

## Vim-Style Keyboard Evolution

### Why Vim Now

The original peeq keyboard layout was designed in 2003 by someone who was not yet a heavy vim user. The choices made sense at the time: comma/period for row navigation, QWERTY interleaved with home-row for steps. They were pragmatic, not philosophically grounded.

The current thinking: lean into vim-style bindings throughout. `j`/`k` for row navigation is already second nature from years of daily vim use. The question is how far to extend the idiom, and where the sequencer's domain model diverges from a text editor's in ways that make vim semantics awkward or misleading.

### The h/l Tension

`h`/`l` in vim move left and right by character. In peeq that maps to two different ideas:

**Option 1: Pattern navigation (current implementation)**
`h` = previous pattern, `l` = next pattern. This is clean and consistent with the transport metaphor. Patterns are like "pages" and you're moving between them.

**Option 2: Column/step selection**
`h`/`l` move a cursor left and right across the 16 steps of the focused row. This requires introducing the concept of a *selected step* — which currently does not exist. Right now the keyboard toggles steps directly; there is no selection cursor.

The tradeoff: Option 1 is simpler and already works. Option 2 enables a lot more — trig locks, accents, per-step editing — but it introduces selection state that has to be displayed, managed, and escaped from.

A possible resolution: `h`/`l` for patterns, and a separate mechanism (maybe `Tab` or entering a step-edit mode) to move a column cursor. This keeps the main navigation modal and avoids cluttering the always-on keyboard state.

### Trig Locks and Accents

The question of whether accents are per-trig or per-column is a design decision with musical implications:

**Per-column accent:** every time step 5 fires across any row, it fires accented. Simple, no additional data structure, maps cleanly to a velocity column. Less expressive: you can't have a kick accent without also accenting everything else on beat 2.

**Per-trig accent:** each individual enabled step cell can independently be accented, normal, or ghost. This is how the Elektron boxes do it (trig locks). More expressive, but requires the step data model to grow from `uint1` (on/off) to at least `uint2` or a richer struct.

The current data model stores steps as 0/1 floats. Adding a second bit of information (accent) would mean either:
- A parallel accent array alongside the step array (same shape, easy to add)
- Encoding accent into the step value: 0=off, 1=on, 2=accented (simple, backward-compatible as long as >0 = on)

If we go per-trig, the keyboard question becomes: how do you accent a step you're toggling? Options:
- Shift+stepkey toggles accent on an already-enabled step
- A separate accent mode (hold a modifier, then tap step keys) — similar to Elektron's function key
- `v` enters visual/accent mode for the focused step, then you mark accents, then escape

### Save/Load and Pattern Banks

The original peeq had no save/load — patterns were lost when PD closed. This was a known gap even in 2003. For the web version the natural solutions are:

- `localStorage` for auto-save on every change (no user action required, survives page refresh)
- JSON export/import for named saves — a "save slot" model where you can snapshot and restore entire sessions
- Multiple named banks: a bank is a set of 8 patterns + per-row settings (note, channel). Switching banks mid-performance is a macro-level pattern change.

The keyboard interface for this is not obvious. `s` is already used as a step key. One approach: a command prefix key (`:` in vim, or a dedicated `Escape`-then-letter sequence) for session operations that aren't needed during performance.

### Copy/Paste: yy and p

Vim's `yy` (yank line) / `p` (put) maps naturally onto row copy/paste:

- `yy` yanks the focused row's 16-step pattern into a register
- `p` pastes it onto the focused row (replacing existing steps)
- `P` could paste onto the row above (vim convention for paste-before)

The interesting extension: **the yank register doubles as the macro source**. Right now macros are hardcoded (quarter notes, eighth notes, 2&4). If `yy` writes into a named register and macros are just named registers that happen to be bound to number keys, then:
- You play a fill, yank it with `yy`, and it becomes available as macro `4` (user slot)
- Or you build a library: `"ayy` yanks into register `a`, and pressing some prefix+`a` replays it
- This unifies the macro system and the clipboard — they're the same thing

This is a significant design move. It means the macro keys (1–4) become register slots, and `yy` / `"a yy` are the way to populate them live.

### Visual Line Mode

Vim's `V` (visual line) selects whole lines; you can then yank, delete, or operate on the selection. For peeq this would mean:

- `V` enters visual row mode from the currently focused row
- `j`/`k` extend the selection down/up
- `y` yanks all selected rows as a multi-row block
- `p` pastes the block starting at the focused row

Use cases: copy a 3-row groove (kick, snare, hh) and paste it to a different set of rows, or into a different pattern. This is the "copy section" operation that currently has no keyboard equivalent.

Open questions:
- Does visual mode select within a pattern or across patterns?
- What does `d` (delete) do in visual row mode — clear all selected rows? That's a destructive operation worth a confirm.
- Does the yanked block preserve row settings (note, channel) or just the step data?

The complexity cost is real. Visual mode adds a modal layer that has to be visible in the UI and escapable. It may be worth implementing basic `yy`/`p` first and seeing whether multi-row selection actually comes up in practice before building the full visual mode machinery.

---

## Decision: Web Rewrite vs. WebPD

Unless there is a specific reason to run peeq.pd as-is (e.g., for archival demonstration), the web rewrite (Option A) or the tidal-arranger integration (Option C) are both more productive paths. Option C is recommended as the first move because:

1. `tidal-arranger` already has working Web MIDI, a parser, and a timing engine
2. Adding a live layer delivers immediate value without rebuilding what already exists
3. The Tidal pattern language is more expressive than PEEQ's binary step grid
4. The work directly advances the arranger project which is already active

A standalone web port of PEEQ (Option A) remains worth doing as a self-contained pedagogical or archival project — a clean, dependency-free step sequencer in a single HTML file.
