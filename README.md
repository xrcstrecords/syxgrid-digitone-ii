# SYXGRID Digitone II

**Pattern and sequence editor for the Elektron Digitone II.**

A single HTML file. Open it in a browser, load a `.syx` dump, and edit the
pattern grid — or talk to the hardware directly over Web MIDI.

> https://syxgrid.xrcst.com

---

## What it does

Decodes Elektron Digitone II pattern SysEx and renders it as an editable step
grid: **16 tracks × up to 128 steps**, with separate lanes for trigs, notes,
velocity, length and microtiming.

- **Read and write `.syx`** — single-pattern captures and full 128-pattern
  project dumps. Edits are repacked with correct checksums; an unedited file
  saves back **byte-identical**.
- **Send and receive over Web MIDI** — pull a dump off the device, edit, send it
  back.
- **Edit everything on the grid** — notes, chords, velocity, length,
  microtiming, pattern and kit names, pattern length, per-track length and
  speed.
- **Import and export** — `.mid` (Standard MIDI) and `.alc` (Ableton Live Clip),
  per track.
- **Trigless trigs and microtiming** — parameter-automation trigs and per-note
  timing offsets are decoded and editable, not just displayed. Microtiming
  survives a round trip through `.mid` and `.alc` in both directions.
- **Track machines and MIDI channels** — every track's SYN machine (FM TONE,
  FM DRUM, WAVETONE, SWARMER or MIDI) is decoded from the dump and colour-coded,
  and MIDI tracks show the channel they send on — including `CH OFF`, which is
  a real state distinct from channel 1. Track type and channel are editable.
- **One-level undo** — Cmd-Z reverts the last edit. Loading, receiving and
  importing are not undoable; they replace the pattern rather than editing it.

No install, no build step, no server, no dependencies. One file.

---

## Requirements

| | |
|---|---|
| Browser | **Chrome** (or another Chromium browser). Firefox and Safari do not implement Web MIDI. |
| Context | **HTTPS or `localhost`** for MIDI. |
| Hardware | Digitone II. The DN1 (`0x0D`) is a different format and is rejected. |

**Opening the file with `file://` disables MIDI.** Web MIDI requires a secure
context, so send/receive vanish silently. Loading and saving `.syx` files still
work; only device I/O is affected. To use MIDI locally:

```bash
python3 -m http.server 8080
# then open http://localhost:8080/sysex-visualiser.html
```

Sending to the device requires the Digitone II to be in **SysEx receive** mode.

---

## Usage

<!-- HELP:BEGIN -->

### Quick start

### Toolbar

| | |
|---|---|
| ${K('Demo')} | load a real demo pattern, so the editor can be tried without a device. It is a normal pattern once loaded — editable, saveable and sendable, marked **DEMO** in the header with a **reload** link back to the pristine copy. Only offered when this page is served from the web. |
| ${K('Open .syx')} | load a single pattern or a whole-project dump. Drag and drop works too. |
| ${K('Get .syx from device')} | receive a dump live over Web MIDI. Arm the listener, then start SYSEX DUMP on the device. A project dump ends on its own; otherwise press ${K('Dump done')}. |
| ${K('↶ Undo')} | revert the last edit. One level — see *Undo* below. |
| ${K('Save .syx')} | write the dump back to a file, repacked with correct checksums. Offers the whole project or the current pattern alone. |
| ${K('Send to device')} | send the current pattern or the whole project back over Web MIDI. The DN2 must be in SYSEX RECEIVE mode. **The pattern lands in the slot selected on the device** — this app cannot choose it, so select the destination pattern on the DN2 first. A queue lands consecutively from there. The dialog exposes a **GAP** in milliseconds — raise it if a large dump loses trigs in transit. |
| ${K('Import…')} / ${K('Export…')} | .mid and .alc, per track. See *Import and export* below. |
| ${K('PATTERN')} / ${K('TRACK')} | pick the pattern to edit and which track to show. ${K('All active')} draws every track that has trigs. |
| ${K('LANES')} | show or hide the Note, Velocity, Length and Micro lanes. Hiding a lane only affects drawing. |
| ${K('VIEW')} | how many steps to draw. *detected* gives each track exactly its own length. |

### The grid

| | |
|---|---|
| **Click** a trig dot | toggle the trig on or off |
| **Shift-click** a trig dot | restore the trig that was there before it was switched off — chord, velocity, microtiming and all. A plain click gives a fresh default trig instead. |
| **Click** a note dot | open the note editor: note name or MIDI number, velocity, length, and adding or removing notes to make chords |
| **Drag** a velocity cell | up or down to change velocity (2 px = 1 unit). Applies to every note on that step. |
| **Drag** a micro cell | nudge the step earlier or later, ±23 units of 1/384 note — just under one step either way |
| **Click** a length cell | opens the note editor; length is per note |
| ${K('⤢ expand')} | enlarge one track 4x. Sequences over 64 steps wrap into rows of 64. Expanding another track shrinks this one; **Esc** shrinks. |

### Reading the grid

| | |
|---|---|
| ${trigDot} trig | an ordinary note trig. A trig with **parameter locks** that still plays its note looks exactly like this — see the note below. |
| ${triglessD} trigless trig | a trig that plays **no note**, used to carry parameter locks only — the hardware's yellow trig. **Shown, not editable**: the trig is decoded from the step, but the locked values are not. Preserved exactly on save. |
| ${ghostDot} ghost | a trig you switched off is remembered here. **Shift-click** brings it back. |
| ${emptyDot} empty | no trig on this step |
| ${chordDot} ring | the step holds more than one note — a chord |
| ${noteDot} note | a single note on the step |
| **·** dot | the value inherits the track default |
| dimmed cells | past the track's length — they will not play |
| **Inherit defaults** | Note C5 (60), Velocity 100, Length 1/16 step. Leaving a field blank in the note editor uses these. |

### Track machines

| | |
|---|---|
| ${M('FM TONE',0)} | four-operator FM — the device default |
| ${M('FM DRUM',2)} | percussion-oriented FM machine |
| ${M('WAVETONE',1)} | two-oscillator phase-distortion / wavetable engine |
| ${M('SWARMER',3)} | unison swarm machine |
| ${M('MIDI',4)} | controls external gear instead of making sound |
| ${K('PRESET / MIDI CH')} | the preset name on a synth track, the channel on a ${M('MIDI',4)} track. ${K('CH OFF')} means the ${M('MIDI',4)} machine has no channel set, so the track sends nothing — that is **not** the same as channel 1. ${K('EMPTY')} means the slot holds no track at all. |
| **Click** a names row, or ${K('type')} | switch a track between audio and ${M('MIDI',4)} and set its channel. **Which synth machine an audio track runs is shown but not editable** — a machine's parameters live in preset data the app does not decode, so setting one would leave the track claiming a machine it has no sound for. |

### Tracks

| | |
|---|---|
| ${K('+ Add track')} | create one or more empty tracks. Tick every slot you want; ${K('all')} and ${K('none')} select in bulk. For ${M('MIDI',4)} tracks a channel picker appears, with ${K('OFF')} as an explicit choice. |
| ${K('clear')} | remove every trig but keep the track, its length, speed and type. Different from deleting it. |
| ${K('duplicate')} | copy a track's trigs — notes, velocity, length and microtiming — to another track. The picker labels each destination (*inactive*, *active, empty*, or *n trigs — will be REPLACED*) so the warning arrives before you choose. Duplicating an empty track creates an empty one. |
| ${K('🗑')} | delete the track entirely. Switching off the last trig does **not** delete a track — that is what this button is for. |
| ${K('len')} / ${K('speed')} | per-track length and speed multiplier. Setting a per-track length switches the pattern into per-track mode. |

### Pattern

| | |
|---|---|
| ${K('PATTERN NAME')} / ${K('KIT NAME')} | click to rename, 16 characters |
| ${K('TEMPO')} | click to set the BPM. Stored as BPM×120, so 174.5 is exact. |
| ${K('PATTERN LENGTH')} | 1–128 steps, 8 pages of 16 (manual §10.11). **Growing** offers empty new steps or copies the existing trigs into them, which is what the device does. **Shrinking** keeps the trigs past the new end — they simply stop playing, and nothing is deleted unless you pick the cut option, which says so. |
| ${K('Pattern stats')} | counts for the current pattern: active tracks, trigs, notes, chord steps, trigless trigs and microtimed trigs. |

### Import and export

| | |
|---|---|
| ${K('Export…')} | pick **.mid** or **.alc** and which tracks. Files are named slot–pattern–track. |
| ${K('Import…')} | load a .mid or .alc onto a chosen track. Speed and length are auto-detected; ${K('replace')} clears the track first, ${K('merge')} keeps what is there. |
| microtiming | survives both directions. Export offsets each note's onset, and import turns an off-grid note back into a trig plus a microtiming offset rather than quantising it away. |
| speed multiplier | applied to both formats: a 2x track's steps last half as long. |
| overlapping notes | a DN2 trig can outlast the gap to the next trig on the same pitch, which MIDI cannot represent. Notes are trimmed to stop just before the next one — otherwise Ableton merges them and they appear to vanish. |
| notes before the start | a negative microtiming offset on step 1 lands before the file begins, which MIDI cannot express. Those notes are pulled to zero and the count is reported rather than changed silently. |

### Undo

| | |
|---|---|
| **Cmd-Z** / **Ctrl-Z**, ${K('↶ Undo')} | revert the last edit — **one level**. A second press does nothing rather than re-applying. |
| what it covers | trigs, notes, velocity, length, microtiming, track operations, pattern and track length, tempo, names, track type and channel. |
| what it does not | opening a file, receiving a dump and importing are not undoable — they replace the pattern rather than editing it, and the file still exists. Switching pattern also clears the undo slot. |

### Shown but not editable

| | |
|---|---|
| ${triglessD} trigless trigs | the trig is shown and preserved; the parameter locks it carries are not decoded |
| parameter locks | not decoded, and on a trig that still plays its note not even detectable — the step encodes only whether the trig sounds. Locked values sit in a separate sparse table and are carried through untouched. **One consequence worth knowing:** a **retrig** set as a lock makes the device play a step several times, and the editor cannot see it — so ${K('Export…')} writes a single note for that step. Exports of retrigged patterns are not accurate. Nothing is lost in the file itself. |
| synth machine choice | ${M('FM TONE',0)}, ${M('FM DRUM',2)}, ${M('WAVETONE',1)} and ${M('SWARMER',3)} are identified, but their parameters live in preset data. Only audio ↔ ${M('MIDI',4)} can be changed. |
| sound parameters | filter, amp, FX and LFO settings are not decoded |
| presets and kits | names are shown, and preset and MIDI track names are written back when they change; the sounds themselves live in the project's preset pool, which is not part of a pattern dump |

### Keyboard

| | |
|---|---|
| **Cmd-Z** / **Ctrl-Z** | undo the last edit |
| **Esc** | close a dialog; if none is open, shrink an expanded track |
| **?** or **H** | open this help |
| **Shift** + length list | in the note editor, hold Shift while opening the length dropdown to see only the common note divisions |

<!-- HELP:END -->

---

## Format documentation

`sysexmap/` documents the pattern SysEx format itself — **45 mappings**
(map version 1.13.0), each with an explicit confidence level and the
evidence behind it.

| Level | Meaning | Count |
|---|---|---|
| 🟢 **A** | Hardware-confirmed against a physical DN2 | 23 |
| 🔵 **B** | Corpus-confirmed by the fixtures in this repository | 12 |
| 🟡 **C** | Inherited — verified elsewhere, only one value exercised here | 9 |
| 🔴 **D** | Unknown — do not rely on it | 1 |

This is **reverse-engineered, not vendor documentation**. Entries below level A
may be wrong. `SYSEXMAP.md` is generated from `mappings.json`, and a test fails
the build if the map and the app's constants disagree — a mapping document that
drifts from the code is worse than none, because it keeps looking authoritative
after it stops being true.

---

## Licence

| | |
|---|---|
| The application | **MIT** — see [`LICENSE`](LICENSE) |
| `sysexmap/` documentation | **CC BY 4.0** — see [`sysexmap/LICENSE`](sysexmap/LICENSE) |

```
Copyright (c) 2026 XRCST — https://syxgrid.xrcst.com
```

The split is deliberate. Creative Commons
[recommends against using CC licences for software](https://creativecommons.org/faq/#can-i-apply-a-creative-commons-license-to-software)
— they carry no patent grant, no source/binary distinction and no warranty
disclaimer — so the program is MIT. The format map is a documentation work, and
CC BY is the right fit there. Both require attribution.

The editor travels as a single loose HTML file, so the copyright notice lives
**inside the file** as well as here — email it to someone and the attribution
goes with it.

---

## Versions

Public releases are numbered `v0.1`, `v0.2`, … Development happens on a
separate, faster-moving track and the in-app version string carries both, e.g.
`v0.1 (dev v22.34)`, so any copy of the file can be traced back to the
exact source it was cut from.

See [`CHANGELOG.md`](CHANGELOG.md).

---

## Disclaimer

This project is not affiliated with, endorsed by, or supported by Elektron.
Elektron and Digitone are trademarks of Elektron Music Machines MAV AB.

It reads and writes files for hardware it was reverse-engineered against.
It ships with no warranty — **keep backups of patterns you care about.**
