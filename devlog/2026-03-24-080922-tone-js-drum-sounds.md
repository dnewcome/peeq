# Tone.js drum sounds per row

_2026-03-24_

## What happened

Each row now has a sound selector: `midi` (default) or one of seven built-in drum voices — kick, snare, closed hi-hat, open hi-hat, clap, tom, rim. Drum rows trigger Tone.js synths (MembraneSynth for kick/tom, NoiseSynth for snare/chh/clap, MetalSynth for ohh/rim) at the scheduler's precise Web Audio timestamps. Tone.js syncs to the existing AudioContext on first play.

MIDI rows behave exactly as before. Mixed rows work — you can run a kick on the browser's built-in sounds while sending snare and hi-hat to an external MIDI device.

The original PD version obviously had no audio output, just MIDI. This makes it self-contained: you can use it without any hardware connected.

## Tweet draft

added built-in drum sounds to the browser sequencer — each row picks midi or kick/snare/chh/ohh/clap/tom/rim. Tone.js MembraneSynth, NoiseSynth, MetalSynth. mix browser audio and MIDI out on separate rows

---

_commit: d80defb_
