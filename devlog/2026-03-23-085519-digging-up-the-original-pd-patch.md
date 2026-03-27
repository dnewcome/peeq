# Digging up the original PD patch

_2026-03-23_

## What happened

Found the old PEEQ tarball from 2003 and brought it into git. The Pure Data source is remarkably intact — `peeq.pd`, `row.pd`, `rowlogic.pd` — three files, 999 lines total, and the whole architecture is legible. Clock driven by `metro`, pattern data in float arrays, 8 patterns × 8 rows × 16 steps addressed by offset math. The changelog goes back to September 2003.

The thing that still holds up: keyboard-first design. `qwerty` row for steps 1–8, `asdfghjk` for steps 9–16. `[` and `]` move row focus. Macros on keys 1–4 (quarter notes, eighth notes, 2&4, user-defined). Numpad `+`/`-` switch patterns without resetting the clock — which is what Ableton later called legato mode, shipped in 2003 on a Dell laptop in a breakdancing club.

Wrote up a plan doc and README to think through what a port or revival would look like.

## Tweet draft

found my old 2003 Pure Data step sequencer — keyboard-driven, no mouse, legato pattern switching before Ableton had a name for it. thinking about what a browser port would look like

---

_commit: 8cbfd83_
