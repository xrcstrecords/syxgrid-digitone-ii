# Changelog

Public releases of SYXGRID Digitone II.

This is a user-facing changelog: it describes what changed for someone using
the editor. Development happens on a separate, faster-moving track with its own
detailed changelog; the in-app version string carries both, e.g.
`v0.1 (dev v22.18)`, so any copy of the file can be traced to the exact source
it was built from.

---

## v1.0 — first stable release

Everything in v0.1, plus the work that made it worth calling stable: track
editing, undo, a demo you can try without hardware, and a rewritten help page.

### Track editing

- **Change a track's type** — switch any track between an audio machine and
  MIDI, and set its MIDI channel, from the track editor or by clicking a row in
  the track names table. Both open the same dialog.
- **MIDI channel is remembered.** Turning a channel off and back on restores the
  number you had, because that is what the hardware does.
- **Add, clear, duplicate and delete tracks**, with a destination picker that
  labels every slot so you know what you are about to overwrite.
- Adding MIDI tracks asks for the channel up front, with **OFF** as an explicit
  choice.

### Undo

- **One level of undo** on every action — `Cmd-Z` / `Ctrl-Z`, or the toolbar
  button, which is greyed out when there is nothing to undo.
- Covers everything: trigs, notes, track operations, pattern length, tempo,
  type and channel changes.
- Switching patterns clears the undo slot, so undo can never restore one
  pattern's state onto another.

### Try it without a Digitone II

- A **Demo** button loads a real pattern — 128 steps, eight tracks at different
  lengths and speed multipliers, chords, microtiming and trigless trigs.
- It is a normal pattern once loaded: editable, saveable, sendable. Marked
  **DEMO** with a link back to the pristine copy.

### Help, rewritten

- A **quick start** in four steps, and every dialog explained.
- Colour-coded: machine names carry their machine colour, UI labels are quoted
  exactly as they appear on the control.
- The trig legend draws **real dots** rather than naming colours.
- A **"shown but not editable"** section that says plainly what the editor
  cannot author, and why — and that none of it is lost by loading and saving.

### Display

- The track names table shows each track's **machine** (FM TONE, FM DRUM,
  WAVETONE, SWARMER, MIDI), colour-coded, with MIDI tracks showing their
  channel instead of a preset name.
- Per-track headers share the same colours.
- Microtiming shows as `<` early, `>` late, `<>` strum.

### Fixed

- A track is no longer deleted when its last trig is switched off. A track
  existing and a track having trigs are different things.
- Duplicating an empty track works, and offers to add a track when the
  destination slot is empty.
- Newly added MIDI tracks show their channel immediately.
- MIDI and Ableton clip export now carries **microtiming**; before, exported
  clips were quantised to the step grid.
- Importing a clip and scaling its speed or length no longer discards
  microtiming.

### Known limitations

- **Parameter locks are not decoded.** A trigless trig is shown (it is the
  yellow one); a p-locked trig that still plays its note is indistinguishable
  from an ordinary trig. Locked values are carried through untouched either way.
- **Sound parameters and presets are not decoded** — this edits the sequencer,
  not the synth.
- **Preset and track names cannot be saved.** They are shown, and the fields
  accept edits, but the change is not written to the file.

---

## v0.1 — first public release

The editor as it stands after an extended reverse-engineering effort against
real Digitone II hardware.

### Reading and writing patterns

- Load Digitone II pattern SysEx — single-pattern captures or full
  128-pattern project dumps.
- Save edits back to `.syx` with correct checksums. **An unedited file saves
  back byte-identical**, verified against a corpus of hardware dumps.
- Send and receive over Web MIDI, so you can pull a pattern off the device,
  edit it, and send it back without files at all.

### Editing

- 16 tracks × up to 128 steps, with separate lanes for trigs, notes, velocity,
  length and microtiming.
- Notes and chords: add, remove, retune, per-note velocity and length.
- **Group editing** — select several notes and move them together. Pitch and
  microtiming shift by the same amount; velocity is additive, so notes at 100
  and 1 dragged up by 27 become 127 and 28, keeping the shape of the chord.
- **Drag to edit** — any numeric field changes by dragging up or down; click to
  type an exact value instead.
- Pattern and kit names, pattern length, per-track length and speed.

### Understanding what you are looking at

- **Trigless trigs** (parameter automation with no note) are decoded and shown
  in yellow, matching the hardware, rather than being mistaken for ordinary
  trigs with a phantom note.
- **Microtiming** is decoded per note and editable — `<` plays early, `>` late,
  `<>` for a chord strummed in both directions.
- Chords, inherited values, out-of-length steps and remembered trigs each have
  their own indicator. Press `?` in the app for the full legend.

### Import and export

- Import `.mid` or `.alc` onto a chosen track, with automatic speed and length
  detection, replacing or merging.
- Export `.mid` or `.alc` per track.

### Format documentation

`sysexmap/` publishes what this project has worked out about the pattern SysEx
format — 40 documented mappings, each with an explicit confidence level and the
evidence behind it. Published under CC BY 4.0 so others can build on it.

### Known limits

- **Web MIDI needs Chrome and a secure context.** Opening the file with
  `file://` silently disables device send/receive; loading and saving files
  still work. Serve over HTTPS or `localhost` for MIDI.
- Note length is confirmed against hardware for a subset of the 127 possible
  length codes; the rest are decoded but not individually hardware-verified.
- A few bytes in the pattern format remain unidentified and are preserved
  untouched rather than guessed at.
