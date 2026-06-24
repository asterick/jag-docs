<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ CD-ROM Subsystem ▸ **CD-ROM Development Tools**
<!-- /nav:top -->

# CD-ROM Development Tools

The tools used to author, emulate, and master Jaguar CDs: the Falcon030-based
**CD-ROM Emulator / Authoring Tool**, the Windows **Jaguar CD Track Creator**,
and **CD mastering** notes.

> **Source:** *Jaguar CD-ROM* developer manual (scanned), © Atari Corp. 1995,
> §3 (Emulator Setup), §4 (Authoring Tool), §5 (Emulator Q&A), §7 (Mastering &
> Track Creator), pp. 15–25, 33–40 (printed).

## Jaguar CD Track Creator (Windows)

The most directly useful tool for a programmer: it merges your data files into a
single track file with the correct [Atari header/tailer and partition
markers](data-format.md#track-headers-and-tailers), so you don't hand-build them.
Runs under Windows 3.1 / WfW 3.11 / Windows 95 beta.

### Inputs

- **Your data files** — code, graphics, music, sound effects, etc.
- **A batch file** — ASCII, one line per data file: `filename <TAB> MARK`, where
  `MARK` is a **4-letter code** repeated 16× to form the 64-byte
  [partition sync marker](data-format.md#partition-markers) that precedes that
  file in the track. At runtime your code searches for this 64-byte block; the
  data follows immediately.

```
G:\JAGUAR\PROJECT\BOOTCODE.BIN	CODE
G:\JAGUAR\PROJECT\TITLESCR.RGB	SCRN
G:\JAGUAR\PROJECT\MUSIC.DAT	MUSC
```

### Outputs

| Field | File | Contents |
|-------|------|----------|
| Track Filename | `*.CDI` | Raw track file ready for CD mastering (header + tailer + markers + data) |
| Header Filename | `*.H` / `*.INC` | `#define` (C) or `equ` (Madmac) of `FILE_<name>` → index, in file order |
| Structure Filename | `*.C` / `*.S` | Array of `FILEDATA` records describing each file in the track |
| Log Filename | `*.LOG` | Human-readable log of the track-creation run |

The generated structure (C form):

```c
typedef struct {
    int   track;          /* track number where the file lives        */
    long  block_offset;   /* offset in CD blocks from track start      */
    long  length;         /* file data length in bytes                 */
    long  marker;         /* the 4-byte partition sync marker          */
} FILEDATA;
```

### Track options

- **Track Number** — written into the header/tailer. **Track #0 = the boot
  track**: the first batch file is treated as boot code, the boot-code
  load/exec address and size fields appear, and the output follows the
  [boot track structure](data-format.md#the-boot-track) (no marker before the
  boot code, plus the load-address/size longwords). Any other track number uses
  the plain [track structure](data-format.md#track-layout-diagram) and the first
  file gets its batch-file marker.
- **Boot Code Load/Exec Address** (hex) — only for track #0.
- **Boot Code Size** (hex) — only for track #0; written into the boot header.
- **End of Track Padding** (hex) — extra padding appended to the track.

### Menus

- **File → Do Batch** (Ctrl+B) processes the batch and writes the selected
  outputs; **Exit** (Ctrl+Q) quits.
- **Options** toggles: *Output Track Data*, *Output Header & Structure File*,
  *Output Log File*, and the mutually-exclusive *C Language Output* /
  *Assembly Output*.

## CD Mastering

To burn a disc you need a CD-Recordable writer, mastering software, and your data.

- Atari used a **Philips CDD-522** recorder on a 486 PC with an Adaptec SCSI host
  adapter, and the mastering packages **CeQuadrat WinOnCD Pro** and **InCat
  Systems Easy CD Pro v3.0**. These make CD-DA, ISO 9660, and CD-XA discs.
- A Jaguar CD is essentially a **multi-session audio-like CD** — choose **"Audio"
  or "Raw"** track type. Some cheaper recorders struggle with multi-session
  (a Jaguar requirement).
- **If your software can't use raw binary files** (e.g. Corel CD Creator wants
  WAV/AIFF): wrap the Track Creator's output with **MKAIFF** (from the Jaguar
  Sound & Music tools) to add an AIFF/WAV wrapper, which mastering software
  strips before writing. The deprecated `FilmToAIFF` option of the Cinepak
  Utilities only handled Cinepak films — don't use it.
- **Watch for inserted silence:** some software adds 2 seconds of silence (150
  blocks × 2352 = 352,800 bytes) at the start of each audio track. Turn this off;
  if you can't, account for it when reading.

## Falcon030 CD-ROM Emulator & Authoring Tool

The emulator (`CDROM.PRG`) runs on an **Atari Falcon030** and emulates the Jaguar
CD by serving data from a large MS-DOS-formatted SCSI hard drive to the Jaguar
console, so you can test without burning discs.

### Hardware setup (§3)

Required: Falcon030 (+ mouse, VGA adapter, VGA monitor), a Jaguar Development
System, a Jaguar Developer CD, the three-header connector, a Falcon030→Jaguar
adapter card with ribbon cable, and a **SCSI hard drive you supply**. Atari
recommends the **Conner CFP1060S / CFP1080S** (~1 GB) — other drives may not work
due to speed/buffer/caching differences.

The three-header PCB connects the Falcon030→Jaguar cable to the gray connector;
the CD-ROM unit connects to the **outer** black connector for normal operation
(emulation disabled) or the **inner** connector to emulate (onboard mechanism
disabled). Launch with **F1** / double-click `CDROM.PRG`.

> **SCSI drive prep (critical):** format the drive on an **Adaptec 1542** in an
> MS-DOS PC (MS-DOS 5.0+), single partition. **Do not use DoubleSpace or any
> real-time compression.** Then move it to the Falcon030.
>
> **Never access the PC-formatted drive from the Falcon030 desktop** — only
> through the CD Emulator's File Selector. The partition scheme is close to but
> not identical to MS-DOS; reading it corrupts the Falcon030 OS (leading to
> crashes / `Internal Error Number -3000`), and writing it corrupts the disk
> (requiring reformat). `Internal Error Number -4000` usually means the wrong
> SCSI ID is set.

### Authoring Tool (§4)

Edits a `.TOC` document as a hierarchy of **Sessions → Tracks → Files**, showing
each item's size, start time-stamp, length, and a comment (≤ 64 chars).

- **File:** New, Open, Save / Save As, **Emulate CD-ROM**, Log File Name¹.
- **Edit:** Cut/Copy/Paste/Delete, Undo, Insert Session, Insert Track (F3),
  Insert File, Select All, Add Comments (F5), Edit File Name. Empty sessions/
  tracks are auto-filled; inserting renumbers following tracks; time-stamps
  recompute after every edit.
- **Search:** Goto Session, Goto Track, Find / Find Next.
- **Options → Preferences:** session/track lead-in/lead-out, **SCSI ID** (must be
  set or the emulator won't initialize), and the **CD-ROM Latency** table.
  Preload Buffer Size¹.

¹ *Log File Name* and *Preload Buffers* are **non-functional** in v2.0 (ignored).

**Latency table:** the defaults are *very* worst-case and don't represent a
production disc — **set them all to 0**. For timing-critical work, burn a real
disc instead. (Operations include initial spin-up, stop/park, pause/unpause,
short seeks, and long seeks of various spans, each with per-session/per-track
adders.)

### Emulating

**File → Emulate CD-ROM** installs drivers and serves data to the Jaguar; press
**Esc** to stop. Restrictions: the data rate is ~95% of double-speed and **no CD
errors are simulated** — so the emulator can't validate your error handling
(burn a disc for that).

**Use one file per track in practice.** The emulator pads each file with zeros to
a multiple of 16 KB and then to a multiple of 2352 bytes (on the fly, not on
disk) to get good Falcon030 performance, which wastes emulated space. Your real
layout shouldn't waste that space — so concatenate your files into one big file
and specify it as a single track. v2.0 reads v1.0 `.TOC` files.

> **Emulator bug (through v2.02):** reading from before the first track or after
> the last track crashes the Falcon030 — add padding tracks to avoid it. Also,
> the classic "CD_read stops after 5–20 KB" symptom is a too-slow 68000 VBL
> handler (see [Programming Guide](programming-guide.md#the-latency-rule)).
>
> A memory dump showing `$DEAD` is the emulator's fill for virtual-disc regions
> with no data (lead-in/out, gaps before/after tracks).

## See also

- [Disc & Data Format](data-format.md) — what the Track Creator generates
- [CD-ROM BIOS API](bios-api.md) — what your runtime code calls
- [Programming Procedures & Guidelines](programming-guide.md)
- [CD-ROM Hardware](hardware.md)

<!-- nav:bottom -->
---

◀ **Prev:** [CD-ROM Programming Procedures & Guidelines](programming-guide.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [CD-ROM Hardware](hardware.md) ▶

**Jump to:** [Architecture](../architecture/overview.md) · [Memory Map](../architecture/memory-map.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md) · [CD-ROM](overview.md)
<!-- /nav:bottom -->
