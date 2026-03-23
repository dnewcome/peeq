# peeq

A keyboard-driven live step sequencer for Pure Data, written in 2003.

Built for live laptop performance — the kind where you are on stage at a breakdancing club with no mouse and no margin for fumbling through menus. Every feature exists to answer the question: how fast can a performer get a musical idea into the sequencer without stopping the music?

## Background

peeq was written during the early livepa.org era, when laptop performance was just becoming a thing and the available hardware was genuinely slow. The direct inspirations were NI Reaktor (for synthesis) and FruityLoops 1.0 (for one specific idea: switching patterns with the numpad while keeping the playhead in place). That playhead-preserving switch is what Ableton would later call legato mode. FruityLoops had it first. peeq had it in Pure Data.

## Requirements

- [Pure Data](https://puredata.info/) (vanilla PD, no externals required)
- A MIDI output device or software synth

## Files

```
peeq.pd        main patch — open this one
row.pd         sequencer row abstraction (16 steps, note, channel, mute)
rowlogic.pd    MIDI note output and pattern array read/write
changelog.txt  version history from v0.01a (Aug 2003) to v0.172b (Dec 2003)
PLAN.md        design notes and forward-looking ideas
```

## How It Works

peeq is an 8-row × 16-step MIDI step sequencer with 8 pattern banks and an 8-bar playlist.

- Each row sends MIDI notes on a configurable channel with a configurable note number
- The clock is driven by a `metro` object at `15000 / BPM` milliseconds per tick
- All 8 rows share the same clock; each row reads independently from its pattern data
- Pattern data is stored as Pure Data float arrays: `$1-seqdata` holds 8 patterns × 16 steps per row
- Rows are modular — adding a 9th row is as simple as adding another `row` instance

## Keyboard Reference

### Transport

| Key | Action |
|-----|--------|
| `Space` | Start / stop (start always resets to step 1) |
| `Numpad +` | Next pattern (legato — playhead stays in position) |
| `Numpad -` | Previous pattern (legato) |

### Row Focus

| Key | Action |
|-----|--------|
| `.` | Move focus down one row |
| `,` | Move focus up one row |

### Step Entry (for the focused row)

The QWERTY and home-row keys are interleaved by column: each keyboard column maps two consecutive steps, Q-row first, A-row second.

```
Q  W  E  R  T  Y  U  I    →  steps  1, 3, 5, 7, 9, 11, 13, 15
A  S  D  F  G  H  J  K    →  steps  2, 4, 6, 8, 10, 12, 14, 16
```

Column pairs: Q+A=1&2, W+S=3&4, E+D=5&6, R+F=7&8, T+G=9&10, Y+H=11&12, U+J=13&14, I+K=15&16.

Each key toggles the corresponding step on or off. The sequencer keeps running while you edit.

### Pattern Macros (for the focused row)

| Key | Pattern loaded |
|-----|---------------|
| `1` | Quarter notes — steps 1, 5, 9, 13 |
| `2` | Eighth notes — steps 1, 3, 5, 7, 9, 11, 13, 15 |
| `3` | 2 & 4 (snare position) — steps 5, 13 |
| `4` | User-configurable (edit the `keycommands` subpatch / `MACROS` in index.html) |

### Per-Row Parameters

| Key | Action |
|-----|--------|
| `'` | Increment MIDI note number |
| `;` | Decrement MIDI note number |
| `]` | Increment MIDI channel |
| `[` | Decrement MIDI channel |
| `C` | Clear pattern (zeroes all 16 steps for all rows of current pattern) |

### Playlist

| Key | Action |
|-----|--------|
| `L` | Toggle playlist on / off |

## Pattern Switching and Legato

The most important behavior: pressing `Numpad +` or `Numpad -` changes the active pattern without resetting the clock. If the sequencer is on step 9, it stays on step 9 in the new pattern. This means pattern changes feel like musical transitions rather than hard cuts.

You can also edit a different pattern than the one currently playing. The `curr_edit_patt` state tracks what you are editing; `pattern` tracks what is playing. Enable chase mode to force the editor to follow the playback position.

## Playlist / Song Mode

The 8-bar playlist assigns one pattern to each bar position. When `L` (playlist) is active, the sequencer advances through bars automatically. Each bar position can be set by clicking directly in the radio button. Playlist length is configurable.

## Architecture Notes

```
peeq.pd
├── transport (metro, BPM, run toggle)
├── pattern bank (8 patterns, numpad switching)
├── playlist (8 bars, L toggle)
├── keycommands subpatch (all keyboard bindings)
└── row 1..8  ← instances of row.pd
      └── rowlogic.pd
            ├── playread  (tabread from $1-seqdata at clock tick)
            ├── writedata (tabwrite on step toggle)
            └── noteout   (makenote → noteout → MIDI)
```

Pattern data layout in `$1-seqdata`:

```
offset = pattern_number * 16 + step_index
```

8 patterns × 16 steps = 128 floats per row. Each row has its own named array.

## Version History (abbreviated)

| Version | Date | Notable addition |
|---------|------|-----------------|
| 0.01a | Aug 2003 | First ideas |
| 0.10b | Sep 2003 | First beta, pattern switching, BPM |
| 0.11b | Sep 2003 | Playlist, spacebar transport, numpad +/- |
| 0.13a | Sep 2003 | Pattern chase on/off, playlist length |
| 0.16b | Dec 2003 | QWERTY step entry, `[`/`]` row focus |
| 0.162b | Dec 2003 | Pattern macros (quarter, eighth, 2&4) |
| 0.17b | Dec 2003 | MIDI note number selection per row |
| 0.171b | Dec 2003 | Key commands for channel, note, mute |
| 0.172b | Dec 2003 | Playlist on/off toggle, spacebar reset |

Full history in `changelog.txt`.

## Known Gaps (from original todo list)

- No load/save — patterns are lost when PD closes
- No MIDI clock sync (send or receive)
- No pattern cut/copy/paste or shift left/right
- No panic / all-notes-off
- No concurrent pattern playback (across rows of different patterns)

## Ideas and Future Directions

See `PLAN.md` for a detailed discussion of:

- **Option A:** Web port — rewrite as a self-contained browser app (Web MIDI, canvas grid, same key bindings)
- **Option B:** WebPD — compile the existing patches to Web Audio API via WebPd
- **Option C:** Live keyboard layer for [tidal-arranger](../tidal-arranger) — bring peeq's keyboard philosophy to a Tidal pattern vocabulary, with bar-quantized slot triggering
- **Option D:** AI-inferred performance templates — analyze live MIDI input in real time to extract rhythmic motifs, harmonic structure, and multi-bar themes; build a template library automatically; let the performer switch to "conductor mode" and arrange the inferred templates with keyboard shortcuts

The core philosophy behind all of these is the same one peeq was built on: **the performer should be able to make musical decisions in real time without ever leaving the keyboard.**
