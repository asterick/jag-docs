<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ CD-ROM Subsystem ▸ **CD-ROM Subsystem Overview**
<!-- /nav:top -->

# CD-ROM Subsystem Overview

What the Jaguar CD is, how it differs from other CD systems, and the CD-ROM
terminology used throughout these docs.

> **Source:** *Jaguar CD-ROM* developer manual (scanned), © Atari Corp. 1995,
> pp. 1–3 ("The Jaguar CD-ROM", "A Bit About CD-ROMs", "Some Definitions").

## What the Jaguar CD is

The Atari Jaguar CD is a high-capacity optical storage device:

| Property | Value |
|----------|-------|
| Capacity | 746.9 MB |
| Speed | Double speed (≈353 kb/sec) |
| Sustained transfer rate | 352,800 bytes/sec (full double-speed) |
| Uncorrectable error rate | < 1 in 10¹¹ (all errors flagged for re-read) |
| Sector / block size | 2352 bytes (588 longs) |
| Frames per second | 75 (at 1× single speed) |
| Format | "Raw" CD-DA, Red Book compliant, Motorola byte order |
| Sessions | Multi-session (Orange Book); appears as up to 40 stacked CDs |
| Tracks | ≤ 99 total across the whole disc (TOC has ≤ 99 entries) |

The high sustained rate (twice the MPC minimum of 150 kb/sec, and without the
40%-CPU-bandwidth penalty an MPC drive incurs) is achievable because of the
Jaguar's very large bus bandwidth.

## How it differs from other CD systems

- **No file system.** Data is accessed *directly*, not through a directory
  structure. You address data by its **time stamp**, not by filename.
- **Time-stamp addressing.** Any position is reached via a time code
  `mm:ss:ff` (minutes : seconds : frames). Time stamps assume single-speed play
  and start at the beginning of the disc. The minimum addressable unit is one
  **frame** (588 longs / 2352 bytes); there are 75 frames/sec at single speed.
- **Reading is inexact.** A read requested at a given time code is **not**
  guaranteed to land memory exactly at the start of that frame. The standard
  technique: **start reading 6 frames early** and **search the first 31 frames
  (72,912 bytes)** for your *partition marker*. Some writer software adds extra
  skew needing more pre-seeking; manufactured (pressed) discs stay well within
  tolerance. See [Programming Guide](programming-guide.md) and
  [Disc & Data Format](data-format.md#partition-markers).
- **Long alignment only.** CD data maintains *long* alignment, **not** phrase
  alignment. Graphics data that must be phrase-aligned has to be moved/realigned
  in your code. (Phrase = 64 bits; see [Blitter](../tom/blitter.md).)
- **All access via the CD BIOS.** To insulate titles from CD-vendor and
  data-transfer changes, **every** access to the CD and its controls **must** go
  through the [CD BIOS](bios-api.md). This is a hard requirement, not a
  recommendation.

## A bit about CD-ROMs

CDs are **constant linear velocity (CLV)**, single-data-track optical media with
one data surface — a spiral roughly a mile long. Absolute position is encoded as
a time code within the disc, resolvable to a single 2352-byte sector. Jaguar CDs
are recorded in CD-DA "raw data" format with Motorola byte ordering: 2352 bytes
per sector, all usable as data.

A standard CD divides into four region types: **lead-in**, **tracks**,
**pauses**, and **lead-out**:

- **Lead-in** (~10,000 sectors, inner diameter) repeats the Table of Contents
  endlessly in the Q subcode.
- **First pause** follows lead-in, 150 or 225 sectors long.
- **Tracks** are the data regions; multiple tracks are separated by 2–3 second
  pause regions.
- **Lead-out** marks the end; primary data is all zeros with an alternating P
  subcode bit.

**Multi-session** CDs appear as up to 40 standard CDs arranged as sequential
annular rings. Regardless of session count, the **total tracks must be ≤ 99**
for the whole disc. (In theory each session could hold 99 tracks for 3960 total,
but that is not officially supported by Philips/Sony; the limit is usually
worked around with a software "logical block / logical file" layer.)

See [Disc & Data Format](data-format.md) for how Jaguar lays sessions and tracks
out, and the Atari header/tailer and partition-marker conventions.

## Glossary of CD-ROM terms

| Term | Definition |
|------|-----------|
| **Absolute Time** | Q-subcode time code, `00:00:00`–`73:59:75`, starting at the first pause region. |
| **Area / Region** | A physical ring-shaped portion of the disc surface. |
| **Channel Frame** | The 588-bit fundamental packet from the laser head: 24 bytes primary + 1 byte secondary (1 bit each of P–W subcodes) plus overhead. |
| **Finalize** | Writing the lead-in (with the main TOC) so a recordable CD is readable by standard players. Unfinalized CDs are generally unplayable except on players designed for it (Jaguar, Photo CD). |
| **Index** | A pointer within the currently playing track, used to access parts of a track independently of time code. |
| **Lead-in** | Inner-diameter region holding the TOC. |
| **Lead-out** | Region marking end of disc/session; primary data zeroed, P subcode alternates at 2 Hz. |
| **Mode** | Track type currently being read (audio, ROM, CD+G, Karaoke, CDI, …). |
| **Open/Closed Session** | A session is "closed" by writing its lead-in/lead-out; while open, data can be appended. **Jaguar cannot access an open session.** |
| **Pause** | Region of digital zeros with the P subcode all ones; some software calls it "Track Lead-in." |
| **Program** | The main data region(s) of a CD. |
| **Relative Time** | Q-subcode time code from `00:00:00` to a track's end-time while reading that track. |
| **Sector / Block** | Smallest addressable primary-data unit: 2352 bytes, readable without post-processing. |
| **Session** | An area with at least one full set of regions (lead-in, pause, track, lead-out). Up to 99 sessions per disc (~40 fit physically). |
| **Subcode Data Channel** | Secondary serial data at 1/192 the primary rate; 8 subcodes P–W. The **Q subcode** carries position info as `minutes:seconds:frames`. |
| **Subcode Frame** | Subcode info from one sector; 75/sec at 1×, 150/sec at 2×. |
| **Table of Contents (TOC)** | Disc directory read from the Q subcode; ≤ 99 entries, one per program, plus disc/manufacturer info. |
| **Track Number** | The number of a program (e.g. an audio selection) on the CD. |

## See also

- [CD-ROM BIOS API](bios-api.md) — the calls your code uses
- [Disc & Data Format](data-format.md) — sessions, tracks, partition markers, headers
- [Programming Procedures & Guidelines](programming-guide.md) — boot, read, error handling
- [CD-ROM Hardware](hardware.md)
- [System Architecture Overview](../architecture/overview.md)

<!-- nav:bottom -->
---

◀ **Prev:** [Jaguar Voice Modem](../peripherals/voice-modem.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [CD-ROM BIOS API](bios-api.md) ▶

**Jump to:** [Architecture](../architecture/overview.md) · [Memory Map](../architecture/memory-map.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md)
<!-- /nav:bottom -->
