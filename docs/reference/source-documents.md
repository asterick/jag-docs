<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ Reference ▸ **Source Documents Index**
<!-- /nav:top -->

# Source Documents Index

Every source PDF in this collection, its extraction status, and which Markdown
docs draw from it.

Legend: **Text** = has a real text layer (extracted directly). **Scanned** =
image-only, transcribed via rendered PNG + the Read tool. **Schematic/data** =
drawing or netlist, cited as a pointer.

## Core system

| PDF | Pages | Status | Used by |
|-----|------:|--------|---------|
| `sources/TechRef_V10.pdf` | 36 | Text | [memory-map](../architecture/memory-map.md), [video-clocks-timing](../architecture/video-clocks-timing.md), [hardware-bugs](../architecture/hardware-bugs.md), [cartridges](../architecture/cartridges.md), [connectors-power](../architecture/connectors-power.md), [controllers](../controllers/controllers.md), [audio](../jerry/audio.md) |
| `sources/SoftRef_V10.pdf` | 107 | Text | [overview](../architecture/overview.md), [object-processor](../tom/object-processor.md), [gpu](../tom/gpu.md), [blitter](../tom/blitter.md), [color-cry](../tom/color-cry.md), [dsp](../jerry/dsp.md), [audio](../jerry/audio.md), [serial-io](../jerry/serial-io.md), [hardware-bugs](../architecture/hardware-bugs.md), [risc-instruction-set](risc-instruction-set.md) |
| `sources/TechOver_V10.pdf` | 18 | Text | [overview](../architecture/overview.md), [blitter](../tom/blitter.md) |
| `sources/Other Documents/Original version of updated documents/HWBugs (Atari Original).pdf` | 6 | Scanned | [hardware-bugs](../architecture/hardware-bugs.md) |

These "Version 10" PDFs are community restorations (© Stephen Moss, 2021) of the
original Atari documentation.

## CD-ROM subsystem

| PDF | Pages | Status | Used by |
|-----|------:|--------|---------|
| `sources/CD-ROM.pdf` | 39 | **Scanned** | all of [docs/cdrom/](../cdrom/overview.md) — [overview](../cdrom/overview.md), [bios-api](../cdrom/bios-api.md), [data-format](../cdrom/data-format.md), [programming-guide](../cdrom/programming-guide.md), [tools](../cdrom/tools.md), [hardware](../cdrom/hardware.md) |
| `sources/Schematics/Jaguar CD ROM.pdf` | 7 | Schematic | [cdrom/hardware](../cdrom/hardware.md) |
| `sources/Other Documents/Jag CD testpro4.pdf` | 23 | Text | [cdrom/hardware](../cdrom/hardware.md) (manufacturing test spec) |

> **Known gap:** the `sources/CD-ROM.pdf` scan contains 39 images for a manual whose
> printed pages run 1–40 — **printed page 4 is missing** from the scan (it falls
> between the end of "Some Definitions" on printed p3 and the start of §2
> "Jaguar CD-ROM BIOS" on printed p5). No BIOS/programming content appears lost;
> §2 begins intact on the next page. Worth sourcing a complete scan if found.

## Controllers & schematics

| PDF | Status | Used by / notes |
|-----|--------|-----------------|
| `sources/Schematics/Jaguar Standard Controller.pdf` | Text | [controllers](../controllers/controllers.md), [connectors-power](../architecture/connectors-power.md) |
| `sources/Schematics/Jaguar Pro Controller.pdf` | Text | [controllers](../controllers/controllers.md), [connectors-power](../architecture/connectors-power.md) |
| `sources/Schematics/Jaguar Rotary Controller.pdf` | Text | [controllers](../controllers/controllers.md), [connectors-power](../architecture/connectors-power.md) (Tempest) |
| `sources/Schematics/Jaguar Team Tap.pdf` | Text | [controllers](../controllers/controllers.md), [connectors-power](../architecture/connectors-power.md) (4-player) |
| `sources/Schematics/Jaguar Schemaic (Clear).pdf` / `Jaguar schematics (Clear).png` | Schematic | [connectors-power](../architecture/connectors-power.md) — console schematic |
| `sources/Schematics/jagcart.gif` | Image | [connectors-power](../architecture/connectors-power.md) — cartridge pinout image |

## Older / alternate references (cross-check only)

| PDF | Pages | Status | Notes |
|-----|------:|--------|-------|
| `sources/Other Documents/TechRef V8/TechRef V8.PDF` (+ `.txt`) | 141 | Text | earlier Technical Reference; ships with a `.txt` extraction |
| `sources/Other Documents/Midsummer Tech Ref Version 6.pdf` | 77 | Text | "Midsummer" chipset tech ref (large, detailed) |
| `sources/Other Documents/Jaguar TechRef (Flair II - 1992).pdf` | 90 | Scanned | early (1992) Flair II tech ref |
| `sources/Other Documents/Original version of updated documents/*` | — | Scanned | original Atari scans (HWBugs, Index, SoftRef, TechOver, TechRef) later restored as the V10 set |

## Supporting / general

| PDF | Pages | Status | Notes |
|-----|------:|--------|-------|
| `sources/Index_V10.pdf` | 12 | Text | documentation index |

## Peripherals & examples

| PDF | Pages | Status | Used by |
|-----|------:|--------|---------|
| `sources/VModem.pdf` | 22 | Scanned | [peripherals/voice-modem](../peripherals/voice-modem.md) |
| `sources/Samples.pdf` | 13 | Scanned | [examples/sample-programs](../examples/sample-programs.md) (copyright-library samples excluded) |

## Candidate topics not yet written

| PDF | Pages | Status | Topic |
|-----|------:|--------|-------|
| `sources/Workshop.pdf` | 22 | Scanned | object-processor / Blitter worked-example tutorials (deferred) |
| `sources/Appendix.pdf` | 22 | Scanned | developer FAQ — mostly dev-system/toolchain support; only a few hardware items |

## Excluded from this reference

The following have been **removed from `sources/`** and are **not documented or
redistributed**, because they are third-party/copyright IP or are tied to a
specific toolchain rather than the hardware:

| Source | Reason |
|--------|--------|
| `Madmac.pdf`, `ALN.pdf`, `Tools.pdf`, `Debugger.pdf` | toolchain (assembler / linker / debugger / utilities) |
| `librarys.pdf` (3D, JPEG, networking, music) | library IP |
| `Cinepak.pdf` | licensed video codec |
| `QSound_V10.pdf` (+ the "Atari Original" copy) | licensed 3D-audio module |
| `GStarted_V10.pdf` / `Getting Started_V10.pdf` (+ the "Atari Original" copy) | toolchain-dependent setup guide |

> **Sample programs (`sources/Samples.pdf`):** samples may be included, **but any
> sample built around a copyright library must be excluded** from the published
> reference. Samples that depend only on standard/freely-redistributable code
> (e.g. a C `stdlib`) are fine.

## See also

- [README](../index.md) — navigable root

<!-- nav:bottom -->
---

◀ **Prev:** [Glossary](glossary.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [Home](../index.md) ▶

**Jump to:** [Architecture](../architecture/overview.md) · [Memory Map](../architecture/memory-map.md) · [Registers](register-list.md) · [Instructions](risc-instruction-set.md) · [Glossary](glossary.md) · [CD-ROM](../cdrom/overview.md)
<!-- /nav:bottom -->
