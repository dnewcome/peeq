# Keyboard overhaul: vim navigation and a new step layout

_2026-03-24_

## What happened

Reworked the keyboard scheme significantly. The original qwerty-row layout mapped keys linearly to steps 1–16, which is intuitive but puts beats 1, 5, 9, 13 scattered across two rows. The new layout groups by beat position: `asdf`/`ASDF` hit beats 1,5,9,13 and 3,7,11,15; `qwer`/`QWER` hit beats 2,6,10,14 and 4,8,12,16. You can now drop a kick pattern (every beat) with four adjacent keys on one row.

Added vim-style navigation: `h`/`l` for pattern left/right, `j`/`k` for row up/down. `x` clears the focused row, `X` clears the full pattern.

Also bumped the UI: 13px→16px font, wider max-width, brighter text colors for contrast. It's readable in a dark room now.

## Tweet draft

reworked the keyboard layout on the browser port of my old step sequencer — beats 1/5/9/13 now land on adjacent keys, vim hjkl for navigation, x/X to clear. starting to feel right for live use

---

_commit: 7971a36_
