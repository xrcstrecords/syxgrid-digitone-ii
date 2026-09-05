# DN2 SysEx Mapping Reference

**Map version 1.13.0** · generated from `mappings.json` for app **v22.33** (2026-09-01)

> Generated file — do not edit by hand. Edit `mappings.json` and run `node sysexmap/build_map.js`.

> Copyright (c) 2026 XRCST — https://syxgrid.xrcst.com
> Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Reverse-engineered documentation, not vendor material: entries below level A may be wrong. Not affiliated with or endorsed by Elektron.

Every documented offset in the Elektron Digitone II pattern format as this project
understands it, with an explicit confidence level and the evidence behind it.

## Confidence levels

| Level | Meaning | Count |
|---|---|---|
| 🟢 A | Hardware-confirmed. Verified against the physical DN2 (playback or device dump) AND reproduced in the local corpus. | 23 |
| 🔵 B | Corpus-confirmed. Proven by the local fixtures in sysex-visualiser/fixtures/, but not independently checked against hardware behaviour. | 12 |
| 🟡 C | Inherited. Claimed verified against the 419-capture corpus that is NOT in this repository. Decodes plausibly here, but only one value is exercised, so the offset is not discriminated locally. | 9 |
| 🔴 D | Unverified / unknown. Purpose or value undetermined; do not rely on it. | 1 |

**11 of 45 mappings are flagged uncertain** — see the section below.

### What the levels actually mean here

The distinction that matters is **B vs C**. A `B` mapping is proven by files in
this repository: you can re-derive it right now with `node sysexmap/probe_mappings.js`.
A `C` mapping is *inherited* — it was verified against a 419-capture corpus that is
**not** in this repository, and the local fixtures exercise only a single value, so
nothing here would notice if the offset were wrong. Treat `C` as plausible but unproven.

## Device

| | |
|---|---|
| Device | Elektron Digitone II |
| Product byte | `0x15` |
| Geometry | 16 tracks × 128 steps |
| Pattern message | 114118 bytes (F0 … F7) |

## Address spaces

Getting these confused is the most common source of wrong offsets.

| Space | Meaning |
|---|---|
| `raw` | Byte offsets into the F0..F7 SysEx message as transmitted (7-bit packed body). |
| `decoded` | Byte offsets into the payload after unpack7(msg, PACK_START, PACK_END). |

```
raw message  ──unpack7(msg, 10, 114113)──▶  decoded payload
```

## Mapping table

| Field | Space | Offset | Value | Conf | Constant |
|---|---|---|---|---|---|
| **Elektron manufacturer ID** | `raw` | `1..3` | 00 20 3C | 🔵 B | `ELEKTRON_ID` |
| **Product byte — Digitone II** | `raw` | `4` | 0x15 | 🔵 B | `PRODUCT_DN2` |
| **Product byte — Digitone 1 (rejected)** ⚠️ | `raw` | `4` | 0x0D | 🟡 C | `PRODUCT_DN1` |
| **Message type** | `raw` | `6` | 0x50 = PATTERN, 0x54 = project trailer | 🔵 B | `MSG_PATTERN / MSG_TRAILER` |
| **Destination pattern slot on receive** | `raw` | `9` | 0-127 (0 = A01 … 127 = H16) | 🟢 A | `—` |
| **7-bit packed body range** | `raw` | `10 .. 114113` | 8-byte groups: 1 control byte + 7 payload bytes | 🔵 B | `PACK_START / PACK_END` |
| **Checksum (hi, lo)** | `raw` | `114113, 114114` | sum(bytes[10..114113)) % 16384, split into two 7-bit values | 🔵 B | `recomputeChecksum()` |
| **Step-state table geometry** | `decoded` | `4 + 1187*(track-1) + 2*(step-1)` | 2 bytes per step, 16 tracks x 128 steps | 🟢 A | `SS_BASE / SS_TRACK_STRIDE / SS_ENTRY` |
| **Pattern select via Program Change** | `value` | `—` | PC 0-127 selects pattern 1-128 (A01-H16), on the PROGRAM CHG IN channel | 🟢 A | `programChange() / LAB_CHANNEL` |
| **Transport: clock is required for playback** | `value` | `—` | 0xF8 at 24 ppqn = 60000/(bpm*24) ms; 0xFA start, 0xFC stop | 🟢 A | `startClock() / CLOCK_PPQN` |
| **Where a received pattern is stored** | `value` | `—` | the slot ARMED on the device, advancing one per pattern | 🟢 A | `setReceiveBase() / RECV_POS` |
| **Step state — odd-step trig** | `decoded` | `ssOffset(t,s)` | 03 81 | 🟢 A | `trigFor() fallback` |
| **Step state — even-step trig** | `decoded` | `ssOffset(t,s)` | 03 91 | 🟢 A | `trigFor() fallback` |
| **Step state — even-step trig, legacy form** | `decoded` | `ssOffset(t,s)` | 0x03 0x90 / 0x03 0x80 - bit 0 CLEAR, therefore NOT an occupied step | 🟢 A | `isTrig()` |
| **Step state — odd-step idle** | `decoded` | `ssOffset(t,s)` | 00 00 | 🔵 B | `idleFor() fallback` |
| **Step state — even-step idle** | `decoded` | `ssOffset(t,s)` | 00 10 | 🔵 B | `idleFor() fallback` |
| **Trigger record table** | `decoded` | `18996` | 6 bytes per record, up to 8192 records | 🟢 A | `TRIG_BASE / TRIG_SLOT / TRIG_MAX_SLOTS` |
| **Record byte 0 — track** | `decoded` | `TRIG_BASE + 6*i + 0` | 0-based track index | 🟢 A | `—` |
| **Record byte 1 — step** | `decoded` | `TRIG_BASE + 6*i + 1` | 0-based step index | 🟢 A | `—` |
| **Record byte 2 — note** | `decoded` | `TRIG_BASE + 6*i + 2` | MIDI note number | 🟢 A | `—` |
| **Record byte 3 — velocity** | `decoded` | `TRIG_BASE + 6*i + 3` | 0-127, or 0xFF = inherit track default | 🟢 A | `INHERIT` |
| **Record byte 4 — length** | `decoded` | `TRIG_BASE + 6*i + 4` | index into LEN_LABEL/LEN_STEPS, or 0xFF = inherit | 🟢 A | `INHERIT` |
| **Record byte 5 — unknown** | `decoded` | `TRIG_BASE + 6*i + 5` | signed 8-bit two's complement, unit 1/384 note, range -23..+23, centre 0x00 | 🟢 A | `—` |
| **End of record list** | `decoded` | `first unused record slot` | a slot is EMPTY when its IDENTITY fields (track, step, note = bytes 0,1,2) are all 0xFF; skip it and continue scanning | 🟢 A | `—` |
| **Chord record ordering** | `decoded` | `TRIG_BASE` | all primaries first, then all chord extras | 🟢 A | `buildSyxForPattern()` |
| **Pattern name** ⚠️ | `decoded` | `88788` | 16 bytes, null-padded ASCII | 🟡 C | `PAT_NAME_OFFSET / NAME_LEN` |
| **Kit name** | `decoded` | `89096` | 16 bytes, null-padded ASCII | 🔵 B | `KIT_NAME_OFFSET` |
| **Preset name table (16 slots)** | `decoded` | `89160 + 359*i` | 16 bytes each | 🔵 B | `PRESET_NAME_BASE / PRESET_NAME_STRIDE` |
| **MIDI track name table (16 slots)** | `decoded` | `95064 + 268*i` | 16 bytes each | 🔵 B | `MIDI_NAME_BASE / MIDI_NAME_STRIDE` |
| **Track type bitmask (1 = MIDI track, 0 = audio/preset track)** | `decoded` | `99348..99349` | 16-bit big-endian; bit i (LSB=0) = track i+1 | 🟢 A | `TRACK_TYPE_MASK` |
| **SYN machine assigned to a track (enum: 0 FM TONE, 1 WAVETONE, 2 FM DRUM, 3 SWARMER, 4 MIDI)** | `decoded` | `89160 + 359*(track-1) + 232` | 0x00..0x04 | 🟢 A | `TRACK_MACHINE` |
| **Machine-dependent SYN parameter area inside a track's preset block** ⚠️ | `decoded` | `89160 + 359*(track-1) + 82 .. +140` | machine-specific | 🟡 C | `null` |
| **MIDI channel enabled flag (0x01 = a channel is set, 0x00 = CHAN is OFF)** | `decoded` | `95064 + 268*(track-1) + 245` | 0x01 set / 0x00 OFF | 🟢 A | `MIDI_CHAN_ENABLED` |
| **MIDI channel of a track (0-indexed: 0 = channel 1 ... 15 = channel 16); valid only when MIDI_CHAN_ENABLED is 0x01** | `decoded` | `95064 + 268*(track-1) + 82` | 0x00..0x0F | 🟢 A | `MIDI_TRACK_CHANNEL` |
| **Pattern length** | `raw` | `payload 101507, control 101506, mask 0x40` | 1-128 steps | 🟢 A | `PLEN_PAY / PLEN_CTRL / PLEN_MASK` |
| **Tempo (BPM)** ⚠️ | `raw` | `hi 101503, lo 101504, control 101498, masks 0x04/0x02` | BPM = ((hi<<8)\|lo) / 120 | 🔵 B | `TEMPO_HI / TEMPO_LO / TEMPO_CTRL / TEMPO_DIV` |
| **Length mode** ⚠️ | `raw` | `101511` | 0 = pattern-wide length, 1 = per-track lengths | 🟡 C | `PMODE_OFFSET` |
| **Pattern speed multiplier** ⚠️ | `raw` | `101512` | index into ['2x','3/2x','1x','3/4x','1/2x','1/4x','1/8x'] | 🟡 C | `PSPEED_OFFSET / SPEED_LABEL` |
| **Per-track speed (16 offsets)** ⚠️ | `raw` | `1349, 2705, 4062, 5419, 6775, 8132, 9488, 10845, 12201, 13558, 14915, 16271, 17628, 18984, 20341, 21697` | one plain byte per track (NOT a packed pair) | 🟡 C | `TRACK_SPEED_OFFSETS` |
| **Per-track length (16 targets)** ⚠️ | `raw` | `see TRACK_LEN_TARGETS: [track, ctrl, payload, mask]` | 1-128 steps, gated by PATTERN_MODE | 🟡 C | `TRACK_LEN_TARGETS` |
| **Trig length code table (128 entries)** ⚠️ | `value` | `—` | 0x02=1/64, 0x06=1/32, 0x0E=1/16, 0x1E=1/8, 0x2E=1/4, 0x3E=1/2, 0x7F=INF | 🟡 C | `LEN_LABEL / LEN_STEPS` |
| **Note name octave convention** ⚠️ | `value` | `—` | MIDI 60 = C5; octave = floor(n/12) | 🟡 C | `noteName() / parseNoteName()` |
| **Steps per bar** | `value` | `—` | 16 steps = 1 bar (1/16 resolution) | 🔵 B | `—` |
| **byte a of the 2-byte step-state entry** | `decoded` | `ssOffset(t,s)` | 0x03 = ordinary step, 0x78 = TRIGLESS trig (parameter automation, sounds no note) | 🟢 A | `undefined` |
| **message type byte 6 = 0x53 (standalone KIT dump)** ⚠️ | `raw` | `6` | 0x53 = KIT message. NOT PRESENT in this project's dumps (we see only 0x50 pattern and 0x54 trailer). | 🔴 D | `undefined` |

⚠️ = flagged uncertain.

## Evidence, mapping by mapping

### 🟢 A — Hardware-confirmed

#### Destination pattern slot on receive

- **Where:** `raw` offset `9`
- **Value:** 0-127 (0 = A01 … 127 = H16)
- **Evidence:** Send path stamps this byte and the DN2 stores the pattern in the addressed slot; exercised by sequential send and the regression lab.
- **Note:** Must be written BEFORE recomputeChecksum(). buildSyxForPattern() always emits 0 here; the transport overwrites it.

#### Step-state table geometry

- **Where:** `decoded` offset `4 + 1187*(track-1) + 2*(step-1)`
- **Value:** 2 bytes per step, 16 tracks x 128 steps
- **Evidence:** Every trig declared by a device dump resolves to a trig-shaped byte pair at this offset; app-built patterns using the same geometry play correctly on hardware.

#### Pattern select via Program Change

- **Where:** `value` offset `—`
- **Value:** PC 0-127 selects pattern 1-128 (A01-H16), on the PROGRAM CHG IN channel
- **Evidence:** Manual OS1.10D section 8.4 p.24: 'MIDI program change messages 0-127 will select pattern 1-128 (A01-H16) on the Digitone II.' Confirmed by the v22.7 single-trig run: the earlier bank-select scheme passed all 16 bank-A cases and failed all 12 bank-B cases, and every failure was exactly the bank-A pattern at (slot & 15) playing instead.
- **Note:** FLAT program numbering - there is NO Bank Select. Slot 16 (B01) is PC 16, not bank 1 + PC 0. CC 0 / CC 32 appear in the manual only for MIDI TRACKS addressing external gear (Appendix A, SYN PAGE), never for selecting a DN2 pattern. Receive is gated by SETTINGS > MIDI CONFIG > SYNC > PRG CH RECEIVE; the listening channel is PROGRAM CHG IN CH (AUTO = auto channel). A wrong channel fails silently.

#### Transport: clock is required for playback

- **Where:** `value` offset `—`
- **Value:** 0xF8 at 24 ppqn = 60000/(bpm*24) ms; 0xFA start, 0xFC stop
- **Evidence:** Confirmed on the bench 2026-09-01: 0xFA alone left the device silent. Adding a 24 ppqn clock made all four test patterns play. Re-confirmed from the browser — 203 ticks per 4s window at 120 BPM against 192 ideal.
- **Note:** With EXT SYNC on, 0xFA only ARMS the sequencer; 0xF8 ticks advance it. Without a clock captureMidi() returns nothing, which reads as a total encoding failure rather than a transport mistake. Start/Stop/Clock are System Realtime and carry no channel byte, so AUTO CHANNEL does not apply to them. Note setInterval cannot hold a 20.8ms period — schedule against an absolute timeline.

#### Where a received pattern is stored

- **Where:** `value` offset `—`
- **Value:** the slot ARMED on the device, advancing one per pattern
- **Evidence:** Manual OS1.10D 13.5.2 p.77: 'PATTERN will store a received pattern to the SELECTED pattern slot.' The v22.8 single-trig run is proof of the auto-advance: 28 patterns were sent and each played back from a DIFFERENT slot (PC 0..27), scoring 28/28 - only possible if each landed in its own successive slot rather than all piling into the armed one. The parity run confirmed the converse: armed at A01, its 4 patterns landed A01-A04 while the lab selected C01-C04, which were empty (0 captured, 0 unexpected).
- **Note:** Byte 9 of the pattern message does NOT choose the destination - buildSyxForPattern writes 0 there deliberately. Any slot 'plan' in software is fiction unless it matches the armed slot. UNVERIFIED: whether the device stays in RECEIVE for an unbroken run of 67+ patterns, and what happens when the pointer runs past H16.

#### Step state — odd-step trig

- **Where:** `decoded` offset `ssOffset(t,s)`
- **Value:** 03 81
- **Evidence:** Observed 32/32 times on odd trigs in device dumps; app-built patterns using it play on hardware.

#### Step state — even-step trig

- **Where:** `decoded` offset `ssOffset(t,s)`
- **Value:** 03 91
- **Evidence:** Device dumps show 03 91 on even trigs (24 occurrences). Writing 03 90 instead (bit 0 clear) made app-added even steps silently vanish on hardware — the 'only odd steps populate' bug.
- **Note:** Bit 0 is the occupancy bit the DN2 reads. An even step is the odd encoding plus bit 4.

#### Step state — even-step trig, legacy form

- **Where:** `decoded` offset `ssOffset(t,s)`
- **Value:** 0x03 0x90 / 0x03 0x80 - bit 0 CLEAR, therefore NOT an occupied step
- **Evidence:** RESOLVED at v22.13 by real device dumps. microtest1 track 1 steps 9-16 read 03 80 / 03 90 with NO trigger records at all, so the device does not play them; the old 'b===0x90 means trig' fallback invented four phantom trigs on an ordinary edited pattern. Separately, the pre-v16 legacy fixture written by THIS app carries 03 90 on steps that DO have records - the same bytes with the opposite meaning.
- **Note:** Because the same byte pair means both things, neither rule alone is correct. The parser uses bit 0 for occupancy AND treats an existing trigger record as authoritative, so legacy data still loads while device dumps stop producing phantoms. What sets 0x80/0x90 on an untriggered step is still unknown: it is NOT deleted-trig residue (the DN2 has no trig-level undo, per the device owner) and NOT pattern length (a dead-centre control track with identical trigs showed clean 0x00 idles instead).

#### Trigger record table

- **Where:** `decoded` offset `18996`
- **Value:** 6 bytes per record, up to 8192 records
- **Evidence:** Record bytes 0..1 decode to the same (track, step) the step-state table declares, on every fixture. Round-trip preserves the region byte-for-byte.

#### Record byte 0 — track

- **Where:** `decoded` offset `TRIG_BASE + 6*i + 0`
- **Value:** 0-based track index
- **Evidence:** Two independent hardware runs (v22.10 seed 1571352731, v22.11 seed 1572355308), 67/67 each. All 16 tracks confirmed end to end: the multi-track suite places one trig per track and the captured MIDI channel is decoded back to the track number (track N transmits on channel N), never inherited from the case. 274 captured notes, zero track mismatches.
- **Note:** Track is observed from the wire, not assumed. An earlier version took the track from the case under test, which made the report agree with itself and would have hidden the bank-addressing bug.

#### Record byte 1 — step

- **Where:** `decoded` offset `TRIG_BASE + 6*i + 1`
- **Value:** 0-based step index
- **Evidence:** Two independent hardware runs (v22.10 seed 1571352731, v22.11 seed 1572355308), 67/67 each. 37 distinct step positions from 1 to 128 confirmed by playback, including boundaries 1, 16, 17, 32, 63, 64, 65, 127, 128 on both track 1 and track 16, plus the parity suite (all-16, odd-only, even-only, all-32) which pins step alignment against off-by-one and parity errors.
- **Note:** Step is 0-based on the wire and 1-based in the UI. The parity cases are the guard against the historical 0x0390 even-step bug.

#### Record byte 2 — note

- **Where:** `decoded` offset `TRIG_BASE + 6*i + 2`
- **Value:** MIDI note number
- **Evidence:** Two independent hardware runs (v22.10 seed 1571352731, v22.11 seed 1572355308), 67/67 each. Note 60 (C5) confirmed across every suite, and the chord suite confirms 60/64/67/71 sounding simultaneously on one step with all four pitches returned distinctly.
- **Note:** MIDI note 60 is labelled C5 on the DN2. Chord extras share the step and track of their primary.

#### Record byte 3 — velocity

- **Where:** `decoded` offset `TRIG_BASE + 6*i + 3`
- **Value:** 0-127, or 0xFF = inherit track default
- **Evidence:** Two independent hardware runs, 67/67 each, with the comparator actually checking velocity: v22.10 seed 1571352731 returned 1, 21, 39, 107, 127 exactly; v22.11 seed 1572355308 returned 1, 3, 28, 47, 127 exactly. Eight distinct values from independent random draws, both boundaries in each run, zero mismatches.
- **Note:** 0xFF means the device chooses, so it must not be asserted on. Concrete values are compared exactly.

#### Record byte 4 — length

- **Where:** `decoded` offset `TRIG_BASE + 6*i + 4`
- **Value:** index into LEN_LABEL/LEN_STEPS, or 0xFF = inherit
- **Evidence:** Two independent hardware runs, 67/67 each: v22.10 seed 1571352731 and v22.11 seed 1572355308. Six length codes sent and note durations measured note-on to note-off on the wire. 1/128=0.125st, 1/64=0.25st, 1/32=0.5st, 1/16=1st, 1/8=2st, 1/4=4st - a 32x span, worst error 11.2%, tightest tolerance headroom 2.2x. Codes 0, 2, 6, 14, 30, 46 confirmed in BOTH runs.
- **Note:** 0xFF = inherit and is not asserted on. SCOPE: 6 of 127 table entries (4.7%) are hardware-confirmed - the six musical divisions. The rest of LEN_STEPS is fine-grained (adjacent entries differ by as little as 0.06 steps, e.g. 1.94 vs 2.00) and cannot be discriminated by a wire measurement at the current 25% relative tolerance, so those codes stay inferred from the table, not confirmed. Durations above 4 steps (63% of the table) are never exercised because the suite uses 16-step patterns.

#### Record byte 5 — unknown

- **Where:** `decoded` offset `TRIG_BASE + 6*i + 5`
- **Value:** signed 8-bit two's complement, unit 1/384 note, range -23..+23, centre 0x00
- **Evidence:** Three device dumps (microtest1, microtestchords, microtestchords2) with microtiming set by hand on the DN2. All 12 requested values decoded exactly: 0->0x00, +23->0x17, -23->0xE9, +12->0x0C, -12->0xF4, +4->0x04, -4->0xFC, +1->0x01. A dead-centre control track in the same dump read 0x00 on every record, and a full block diff showed byte 6 as the only difference between the offset track and the control track.
- **Note:** PER-RECORD, not per-step: two notes on the SAME step can carry different offsets, which is how chord strum is stored (proven in microtestchords2: step 1 holds C5 at -23 and C6 at +23). The 7x128 per-track lane region stayed entirely 0xFF in every microtiming dump, ruling out a per-step lane. One 16th step = 24 ticks, so +-23 nudges almost a full step without reaching it.

#### End of record list

- **Where:** `decoded` offset `first unused record slot`
- **Value:** a slot is EMPTY when its IDENTITY fields (track, step, note = bytes 0,1,2) are all 0xFF; skip it and continue scanning
- **Evidence:** v22.13. The old rule ('first five bytes 0xFF ends the record list') is WRONG in two ways, both proven by device dumps. (1) It ends the scan too early on a legitimate record: vel and len may BOTH be INHERIT (0xFF), making a live record read 0xFF in fields 3 and 4. (2) The DN2 leaves DELETED slots BETWEEN live records - identity fields blanked but byte 6 (microtiming) left stale - so a rule that BREAKS rather than CONTINUES loses every record after the first gap. In microtestchords the strum dump has deleted slots at positions 4-7 with track 2's records after them; breaking there made track 2 look recordless, which was written up as real device state before the occupancy invariant caught it.
- **Note:** Do NOT break on a blank slot - CONTINUE. When rebuilding an unedited pattern, replay the original slot table verbatim so the gaps survive; compacting them changes bytes in a file that was only read.

#### Chord record ordering

- **Where:** `decoded` offset `TRIG_BASE`
- **Value:** all primaries first, then all chord extras
- **Evidence:** Records are ordered by ONSET TIME (step + microtiming/24). Verified across three dumps, 10/10 multi-note pairs. Pitch ordering is refuted by microtestchords, where C6 at mt 0 is stored BEFORE C5 at mt +23 on steps 2/4/6/8 - higher pitch first because it sounds first. Insertion ordering is refuted by microtestchords2 step 1, where C6 was entered first but stored second because C5 at -23 sounds earlier.
- **Note:** The older 'all primaries before chord extras' description is a consequence of onset ordering in simple cases, not the rule. Sort by (step + micro/24) when rebuilding.

#### Track type bitmask (1 = MIDI track, 0 = audio/preset track)

- **Where:** `decoded` offset `99348..99349`
- **Value:** 16-bit big-endian; bit i (LSB=0) = track i+1
- **Evidence:** Four purpose-built hardware dumps. tracktype1.syx (T2 set to MIDI) reads 0x0002; tracktype2.syx (T2 and T4 MIDI) reads 0x000A -- exactly one bit flips, and it is bit 3 = track 4, as predicted before the dump was taken; tracktypeconfirm.syx (T2, T4, T13, T16 MIDI) reads 0x900A, exercising the HIGH byte for the first time; tracktypeMachines.syx keeps that same mask while changing three AUDIO tracks' machines, confirming the mask tracks MIDI-ness alone and not the machine. Whole-file diff of the first two dumps is 4 bytes and only pattern 0 differs; the other 127 untouched patterns read 0x0000. This mask is a redundant second encoding of TRACK_MACHINE == 4: the invariant holds on all 22,464 track slots across 19 files with 0 mismatches.

#### SYN machine assigned to a track (enum: 0 FM TONE, 1 WAVETONE, 2 FM DRUM, 3 SWARMER, 4 MIDI)

- **Where:** `decoded` offset `89160 + 359*(track-1) + 232`
- **Value:** 0x00..0x04
- **Evidence:** CORRECTION of a v22.23 misreading: this byte was mapped as a BOOLEAN track-type marker (0x04 = MIDI, 0x00 = audio) because every track in every dump then held either FM TONE or MIDI, so the field looked binary. tracktypeMachines.syx sets T1 FM Tone, T3 FM Drum, T5 Wavetone, T6 Swarmer on the device and reads 0x00, 0x02, 0x01, 0x03 -- an enum. Diffing that dump against tracktypeconfirm.syx (identical but for those three tracks) gives 57 differing bytes, ALL inside the preset blocks of exactly T3, T5 and T6. All five SYN machines listed in the manual (APPENDIX A, A.2.1-A.2.5) are accounted for, so the enum is complete. Note the IDs are NOT the manual's section order: the manual lists FM DRUM second and WAVETONE third, the file numbers them the other way round. MIDI is machine 4, i.e. not a separate concept from a machine -- the invariant 'TRACK_TYPE_MASK bit set <=> id == 4' holds on all 22,464 track slots across 19 files with 0 exceptions. Write consequence: deriving this byte from the MIDI mask flattens FM DRUM, WAVETONE and SWARMER to FM TONE on every save; that bug shipped in v22.23 and is fixed in v22.25.

#### MIDI channel enabled flag (0x01 = a channel is set, 0x00 = CHAN is OFF)

- **Where:** `decoded` offset `95064 + 268*(track-1) + 245`
- **Value:** 0x01 set / 0x00 OFF
- **Evidence:** CORRECTION of a v22.23 misreading. This byte was first mapped as a third copy of the track TYPE, because in every dump available then 'is a MIDI track' and 'has a channel set' coincided on all 22,447 track slots. tracktypeconfirm.syx separates them: T16 is a MIDI track by BOTH type encodings (mask bit 15 set, preset marker 0x04) yet reads 0x00 here, and it is the one track the user configured with CHAN unset. It is an enable flag, not a type marker. Manual A.2.5 p.100 gives CHAN the range (OFF, 1-16), and a 0..15 channel byte has no encoding for OFF -- this is where OFF lives. Consequence: deriving this byte from the type mask on write re-enables a deliberately-off channel; that bug shipped in v22.23 and is fixed in v22.24.

#### MIDI channel of a track (0-indexed: 0 = channel 1 ... 15 = channel 16); valid only when MIDI_CHAN_ENABLED is 0x01

- **Where:** `decoded` offset `95064 + 268*(track-1) + 82`
- **Value:** 0x00..0x0F
- **Evidence:** Three dumps. X-MIDI-1-16 (all 16 tracks MIDI on channels 1..16) reads an exact 0..15 ramp, but that test alone is confounded because channel-1 == track-1 there. tracktype2.syx breaks it: MIDI tracks T2 (channel 1) and T4 (channel 4) read 0x00 and 0x03. tracktypeconfirm.syx confirms on a fresh value: T13 set to channel 8 reads 0x07 -- a track index would read 12. Diffing the full 268-byte MIDI block of two MIDI tracks that differ only in channel yields exactly one non-name difference, at +82. Never outside 0..15 in 22,448 track slots across 18 files. OFF is NOT encoded here -- see MIDI_CHAN_ENABLED -- so channel 1 and OFF share the value 0x00 and must be told apart by the enable flag.

#### Pattern length

- **Where:** `raw` offset `payload 101507, control 101506, mask 0x40`
- **Value:** 1-128 steps
- **Evidence:** Hardware-confirmed across TWO independent DN2LAB runs (v22.10 seed 1571352731 and v22.11 seed 1572355308), both 67/67. The pattern-length suite writes 16, 32, 64 and 128 steps, sends each to the device, and captures playback: plen-16, plen-32, plen-64 and plen-128 all pass, with a trig placed on the LAST step of each length so a truncated pattern would drop it. This was also the field behind the v22.6 truncation bug ('plays fine to step 16 then nothing'), fixed by writing raw metadata AFTER repack7 - the regression is locked by test_v22_pattern_length.js plus its _live.js counterpart. Superseding the old note that only one value was exercised: that was true of the static FIXTURES, not of the hardware suite.
- **Note:** Four values proven on hardware (16/32/64/128) out of 128 legal lengths. The control byte at 101506 mask 0x40 must be set for the payload at 101507 to take effect. Writes MUST happen in writeRawMeta() after repack7 or they are silently reverted.

#### byte a of the 2-byte step-state entry

- **Where:** `decoded` offset `ssOffset(t,s)`
- **Value:** 0x03 = ordinary step, 0x78 = TRIGLESS trig (parameter automation, sounds no note)
- **Evidence:** triglesstest1/2, built to spec on the device. T1 steps 2,4,6,8 were set as trigless trigs and encode 78 11; the plain C5 trigs on steps 1,3,5,7 encode 03 81, and the reference track T2 is 03 81/03 91 throughout. triglesstest2 is the discriminator: its steps 9-12 carry a note AND parameter locks yet still encode 03 81/03 91, so 0x78 does not mark p-locks - it marks specifically that the step sounds no note.
- **Note:** A trigless trig stores a COMPLETE ordinary trigger record (note 60, vel/len INHERIT, microtiming 0). Nothing in the record distinguishes it, which is why the editor displayed a phantom C5 on those steps until v22.14. Byte a is the only marker. Do NOT infer it from bit 7 of byte b: bit 7 is SET on empty active steps (03 80 / 03 90, which sound nothing) and CLEAR on trigless steps (78 11), so it cannot mean 'sounds a note'. Preserve byte a verbatim on write and never sample a non-0x03 pair as the template for a newly added trig. Only EVEN trigless steps have been observed (78 11); the odd form is not yet sampled.

### 🔵 B — Corpus-confirmed

#### Elektron manufacturer ID

- **Where:** `raw` offset `1..3`
- **Value:** 00 20 3C
- **Evidence:** 514 fixture messages all match.

#### Product byte — Digitone II

- **Where:** `raw` offset `4`
- **Value:** 0x15
- **Evidence:** 514 fixture messages all read 0x15.

#### Message type

- **Where:** `raw` offset `6`
- **Value:** 0x50 = PATTERN, 0x54 = project trailer
- **Evidence:** Project fixtures split into 128x 0x50 + 1x 0x54; the 128-message fixtures have no trailer.

#### 7-bit packed body range

- **Where:** `raw` offset `10 .. 114113`
- **Value:** 8-byte groups: 1 control byte + 7 payload bytes
- **Evidence:** unpack7 -> repack7 reproduces the source message byte-for-byte on every fixture tested.

#### Checksum (hi, lo)

- **Where:** `raw` offset `114113, 114114`
- **Value:** sum(bytes[10..114113)) % 16384, split into two 7-bit values
- **Evidence:** Recomputed checksum matches the device's own bytes on every fixture tested.

#### Step state — odd-step idle

- **Where:** `decoded` offset `ssOffset(t,s)`
- **Value:** 00 00
- **Evidence:** 1016/1016 non-trig odd steps in the fixtures.

#### Step state — even-step idle

- **Where:** `decoded` offset `ssOffset(t,s)`
- **Value:** 00 10
- **Evidence:** 1016/1016 non-trig even steps in the fixtures. Bit 4 is the even-step structural marker, NOT a trig indicator.
- **Note:** Writing 03 01 for idle turns empty tracks into fully-trigged patterns.

#### Kit name

- **Where:** `decoded` offset `89096`
- **Value:** 16 bytes, null-padded ASCII
- **Evidence:** Decodes to 'KIT 1' while the pattern name is empty, proving it is a distinct field and not a pattern-name mirror.
- **Note:** The digitone-syx-toolkit mislabels this as a pattern-name shadow. It is not. SCOPE: this is the kit name carried INSIDE a 0x50 PATTERN message (decoded space). A 2026-09-02 cross-reference from another project reported kit names at RAW bytes 23-29 and recommended replacing this offset. That report describes the standalone 0x53 KIT message - a different message type with its own layout. Our dumps contain only 0x50 and 0x54, no 0x53 at all, so the two do not conflict and this offset must NOT be replaced. Verified 2026-09-02: decoded 89096 reads 'KIT 1' correctly, while raw bytes 23-29 of the same 0x50 message are binary (17,3,1,42,3,17,3), not ASCII.

#### Preset name table (16 slots)

- **Where:** `decoded` offset `89160 + 359*i`
- **Value:** 16 bytes each
- **Evidence:** All 16 slots decode to the factory defaults PRESET 1..PRESET 16, which proves both base and stride.

#### MIDI track name table (16 slots)

- **Where:** `decoded` offset `95064 + 268*i`
- **Value:** 16 bytes each
- **Evidence:** All 16 slots decode to the factory defaults MIDI 1..MIDI 16, which proves both base and stride.

#### Tempo (BPM) ⚠️

- **Where:** `raw` offset `hi 101503, lo 101504, control 101498, masks 0x04/0x02`
- **Value:** BPM = ((hi<<8)\|lo) / 120
- **Evidence:** Two distinct values are exercised locally: 120.0 BPM (raw 14400) across the device corpus and 180.0 BPM (raw 21600) in the v22 device dump - so the field, its /120 scaling and the hi/lo pair at 101503/101504 are discriminated by more than a single default. Not level A: the DN2LAB suite drives tempo over MIDI clock rather than reading this field back from the device, so no capture proves the DEVICE adopts a written value. A separate project reports 76-127 BPM across 1,280 patterns, consistent with this encoding but not verified here.
- **Note:** To promote to A: write a non-default tempo, dump the project back, and confirm the field round-trips - or capture playback and measure the actual step interval.

#### Steps per bar

- **Where:** `value` offset `—`
- **Value:** 16 steps = 1 bar (1/16 resolution)
- **Evidence:** Consistent with the length table anchor (1/16 == 1 step) and with MIDI export timing tests (step 2 lands at tick 24 at PPQ 96).

### 🟡 C — Inherited

#### Product byte — Digitone 1 (rejected) ⚠️

- **Where:** `raw` offset `4`
- **Value:** 0x0D
- **Evidence:** No Digitone 1 file exists in this repository; the rejection path is unexercised locally.
- **Note:** Rejection logic is implemented but never proven here. Needs a DN1 capture.

#### Pattern name ⚠️

- **Where:** `decoded` offset `88788`
- **Value:** 16 bytes, null-padded ASCII
- **Evidence:** Every fixture pattern is unnamed, so the offset is never exercised by a non-empty value here.
- **Note:** OPEN QUESTION (2026-09-02, from a separate project's cross-reference): they report that in FULL PROJECT dumps the per-pattern names live in the 0x54 TRAILER rather than in each 0x50 pattern message, having found 47 named patterns whose 88788 field was all zeros. We can neither confirm nor refute this: every pattern in our corpus is unnamed, and our 601-byte trailer holds no ASCII run longer than 3 chars. TO SETTLE: name a pattern on the device, dump it, and check whether the name appears at 88788, in the trailer, or both.

#### Machine-dependent SYN parameter area inside a track's preset block ⚠️

- **Where:** `decoded` offset `89160 + 359*(track-1) + 82 .. +140`
- **Value:** machine-specific
- **Evidence:** Changing a track's machine rewrites bytes in +82..+140 of that track's preset block and nowhere else in the pattern (57 bytes across T3/T5/T6 in tracktypeMachines.syx, none outside the preset blocks). Two tracks running different machines differ at 21 offsets in this window. So the pattern DOES carry per-track SYN parameter data, contradicting the earlier research note that a pattern holds only a preset reference. Individual parameters are NOT decoded: no dump yet varies a single parameter at a fixed machine, which is what would be needed to attribute offsets to controls. Level C -- the region is located, its contents are not.

#### Length mode ⚠️

- **Where:** `raw` offset `101511`
- **Value:** 0 = pattern-wide length, 1 = per-track lengths
- **Evidence:** Every local fixture reads 0; per-track mode is never set in this corpus.

#### Pattern speed multiplier ⚠️

- **Where:** `raw` offset `101512`
- **Value:** index into ['2x','3/2x','1x','3/4x','1/2x','1/4x','1/8x']
- **Evidence:** Every local fixture reads 2 (= 1x, the default), so no other code is exercised.

#### Per-track speed (16 offsets) ⚠️

- **Where:** `raw` offset `1349, 2705, 4062, 5419, 6775, 8132, 9488, 10845, 12201, 13558, 14915, 16271, 17628, 18984, 20341, 21697`
- **Value:** one plain byte per track (NOT a packed pair)
- **Evidence:** Claimed isolated by differential on captures not in this repository. Locally all 16 read the default 2, so the individual offsets are not discriminated.
- **Note:** Stride alternates 1356/1357, so the list is explicit rather than computed.

#### Per-track length (16 targets) ⚠️

- **Where:** `raw` offset `see TRACK_LEN_TARGETS: [track, ctrl, payload, mask]`
- **Value:** 1-128 steps, gated by PATTERN_MODE
- **Evidence:** Claimed verified individually via L_MSB_MAP captures not in this repository. Locally all 16 read 16, so the individual offsets are not discriminated.

#### Trig length code table (128 entries) ⚠️

- **Where:** `value` offset `—`
- **Value:** 0x02=1/64, 0x06=1/32, 0x0E=1/16, 0x1E=1/8, 0x2E=1/4, 0x3E=1/2, 0x7F=INF
- **Evidence:** Taken from the toolkit's length_codes.py. Internally consistent (monotonic, division labels land on the right codes), but every fixture note uses INHERIT so no code is exercised against hardware here.
- **Note:** 1/16 == 1 step is the anchor the rest of the table is scaled against.

#### Note name octave convention ⚠️

- **Where:** `value` offset `—`
- **Value:** MIDI 60 = C5; octave = floor(n/12)
- **Evidence:** Display convention matching the DN2's own labelling. No capture in this repository proves it; it rests on prior observation of the device UI.
- **Note:** noteName() and parseNoteName() must always agree, whatever the convention.

### 🔴 D — Unverified / unknown

#### message type byte 6 = 0x53 (standalone KIT dump) ⚠️

- **Where:** `raw` offset `6`
- **Value:** 0x53 = KIT message. NOT PRESENT in this project's dumps (we see only 0x50 pattern and 0x54 trailer).
- **Evidence:** Reported 2026-09-02 by a separate project working from full-project exports that include kit dumps: kit names appear as ASCII at RAW bytes 23-29, header 00 20 3C 15 00 53 01 01 <index>. UNVERIFIED HERE - zero 0x53 messages exist in any of our five device dumps, so we cannot confirm or refute the layout.
- **Note:** Do not merge with KIT_NAME: that mapping addresses the kit name embedded in a 0x50 PATTERN message and is confirmed working. This is a placeholder for a message type we have never captured.

## Uncertain mappings

These 11 entries should not be treated as settled.

| Field | Conf | Why it is uncertain |
|---|---|---|
| Product byte — Digitone 1 (rejected) | 🟡 C | Rejection logic is implemented but never proven here. Needs a DN1 capture. |
| Pattern name | 🟡 C | OPEN QUESTION (2026-09-02, from a separate project's cross-reference): they report that in FULL PROJECT dumps the per-pattern names live in the 0x54 TRAILER rather than in each 0x50 pattern message, having found 47 named patterns whose 88788 field was all zeros. We can neither confirm nor refute this: every pattern in our corpus is unnamed, and our 601-byte trailer holds no ASCII run longer than 3 chars. TO SETTLE: name a pattern on the device, dump it, and check whether the name appears at 88788, in the trailer, or both. |
| Machine-dependent SYN parameter area inside a track's preset block | 🟡 C | Changing a track's machine rewrites bytes in +82..+140 of that track's preset block and nowhere else in the pattern (57 bytes across T3/T5/T6 in tracktypeMachines.syx, none outside the preset blocks). Two tracks running different machines differ at 21 offsets in this window. So the pattern DOES carry per-track SYN parameter data, contradicting the earlier research note that a pattern holds only a preset reference. Individual parameters are NOT decoded: no dump yet varies a single parameter at a fixed machine, which is what would be needed to attribute offsets to controls. Level C -- the region is located, its contents are not. |
| Tempo (BPM) | 🔵 B | To promote to A: write a non-default tempo, dump the project back, and confirm the field round-trips - or capture playback and measure the actual step interval. |
| Length mode | 🟡 C | Every local fixture reads 0; per-track mode is never set in this corpus. |
| Pattern speed multiplier | 🟡 C | Every local fixture reads 2 (= 1x, the default), so no other code is exercised. |
| Per-track speed (16 offsets) | 🟡 C | Stride alternates 1356/1357, so the list is explicit rather than computed. |
| Per-track length (16 targets) | 🟡 C | Claimed verified individually via L_MSB_MAP captures not in this repository. Locally all 16 read 16, so the individual offsets are not discriminated. |
| Trig length code table (128 entries) | 🟡 C | 1/16 == 1 step is the anchor the rest of the table is scaled against. |
| Note name octave convention | 🟡 C | noteName() and parseNoteName() must always agree, whatever the convention. |
| message type byte 6 = 0x53 (standalone KIT dump) | 🔴 D | Do not merge with KIT_NAME: that mapping addresses the kit name embedded in a 0x50 PATTERN message and is confirmed working. This is a placeholder for a message type we have never captured. |

## Closing the gaps

Most `C` entries are unproven for the same reason: the local corpus only ever
exercises the default value. The regression lab (`window.DN2LAB`, v22.3+) has a
suite aimed at each one — run them against real hardware to promote the mapping:

| Gap | Lab suite |
|---|---|
| Velocity field | `velocity` |
| Length field + length table | `length` |
| Chord record ordering | `chord` |
| Pattern length | `pattern-length` |
| Per-track offsets | `multi-track` |
| Step-state across the whole grid | `single-trig`, `parity` |

```js
await page.evaluate(() => DN2LAB.runSuite('all'))
```

## Regenerating and checking

```bash
node sysexmap/probe_mappings.js   # re-derive what the fixtures can prove
node sysexmap/build_map.js        # regenerate this file from mappings.json
node sysexmap/test_map_sync.js    # assert the map matches the app's constants
```

`test_map_sync.js` runs as part of `sysex-visualiser/run_all_tests.sh`, so a
constant changed in the app without updating the map fails the suite.

## Scope

This map covers the **pattern** message (`0x50`) only. Not covered: the project
trailer's contents (`0x54`, skipped as non-pattern), sound/preset parameter data,
kit contents beyond the name, and song/arrangement data. The DN2 pattern format is
not publicly documented; everything here was recovered by differential analysis of
device captures.
