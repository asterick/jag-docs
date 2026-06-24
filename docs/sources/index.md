<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ Source Files ▸ **Source Documents**
<!-- /nav:top -->

# Source Documents

The original source files this reference is built from. Click any entry to open or
download it. **Text** = has an extractable text layer; **Scanned** = image-only
(transcribed via rendered pages); **Schematic/image** = a drawing or bitmap.

## Core system

| File | Type | Description / used by |
|------|------|-----------------------|
| [TechRef_V10.pdf](TechRef_V10.pdf) | Text | Technical Reference — console hardware, memory map, video timing, controllers, audio, cartridges. Used by [memory-map](../architecture/memory-map.md), [video-clocks-timing](../architecture/video-clocks-timing.md), [hardware-bugs](../architecture/hardware-bugs.md), [cartridges](../architecture/cartridges.md), [connectors-power](../architecture/connectors-power.md), [controllers](../controllers/controllers.md), [audio](../jerry/audio.md). |
| [SoftRef_V10.pdf](SoftRef_V10.pdf) | Text | Software Reference (Tom & Jerry) — the programmer's model. Used by [overview](../architecture/overview.md), [object-processor](../tom/object-processor.md), [gpu](../tom/gpu.md), [blitter](../tom/blitter.md), [color-cry](../tom/color-cry.md), [dsp](../jerry/dsp.md), [audio](../jerry/audio.md), [serial-io](../jerry/serial-io.md), [risc-instruction-set](../reference/risc-instruction-set.md). |
| [TechOver_V10.pdf](TechOver_V10.pdf) | Text | Technical Overview. Used by [overview](../architecture/overview.md), [blitter](../tom/blitter.md). |
| [Other Documents/.../HWBugs (Atari Original).pdf](Other%20Documents/Original%20version%20of%20updated%20documents/HWBugs%20%28Atari%20Original%29.pdf) | Scanned | Original Hardware Bugs & Warnings. Used by [hardware-bugs](../architecture/hardware-bugs.md). |

The "Version 10" PDFs are community restorations (© Stephen Moss, 2021) of the original Atari documentation.

## CD-ROM subsystem

| File | Type | Description / used by |
|------|------|-----------------------|
| [CD-ROM.pdf](CD-ROM.pdf) | Scanned | The Jaguar CD-ROM developer manual. Source for all of [the CD-ROM docs](../cdrom/overview.md). |
| [Schematics/Jaguar CD ROM.pdf](Schematics/Jaguar%20CD%20ROM.pdf) | Schematic | CD-ROM unit schematic. Used by [cdrom/hardware](../cdrom/hardware.md). |
| [Other Documents/Jag CD testpro4.pdf](Other%20Documents/Jag%20CD%20testpro4.pdf) | Text | CD-ROM drive-module manufacturing test spec. Used by [cdrom/hardware](../cdrom/hardware.md). |

> **Known gap:** the CD-ROM.pdf scan has 39 images for a manual whose printed pages run 1–40 — **printed page 4 is missing** (between "Some Definitions" and §2 "Jaguar CD-ROM BIOS"). No BIOS/programming content appears lost. Worth sourcing a complete scan if found.

## Controllers & schematics

| File | Type | Used by |
|------|------|---------|
| [Schematics/Jaguar Standard Controller.pdf](Schematics/Jaguar%20Standard%20Controller.pdf) | Text | [controllers](../controllers/controllers.md), [connectors-power](../architecture/connectors-power.md) |
| [Schematics/Jaguar Pro Controller.pdf](Schematics/Jaguar%20Pro%20Controller.pdf) | Text | [controllers](../controllers/controllers.md), [connectors-power](../architecture/connectors-power.md) |
| [Schematics/Jaguar Rotary Controller.pdf](Schematics/Jaguar%20Rotary%20Controller.pdf) | Text | [controllers](../controllers/controllers.md), [connectors-power](../architecture/connectors-power.md) (Tempest) |
| [Schematics/Jaguar Team Tap.pdf](Schematics/Jaguar%20Team%20Tap.pdf) | Text | [controllers](../controllers/controllers.md), [connectors-power](../architecture/connectors-power.md) (4-player) |
| [Schematics/Jaguar Schemaic (Clear).pdf](Schematics/Jaguar%20Schemaic%20%28Clear%29.pdf) · [PNG](Schematics/Jaguar%20schematics%20%28Clear%29.png) | Schematic | Console schematic — [connectors-power](../architecture/connectors-power.md) |
| [Schematics/jagcart.gif](Schematics/jagcart.gif) | Image | Cartridge pinout — [connectors-power](../architecture/connectors-power.md) |

## Older / alternate references (cross-check only)

| File | Type | Notes |
|------|------|-------|
| [Other Documents/TechRef V8/TechRef V8.PDF](Other%20Documents/TechRef%20V8/TechRef%20V8.PDF) ([txt](Other%20Documents/TechRef%20V8/TechRef%20V8.txt)) | Text | Earlier Technical Reference; ships with a text extraction. |
| [Other Documents/Midsummer Tech Ref Version 6.pdf](Other%20Documents/Midsummer%20Tech%20Ref%20Version%206.pdf) | Text | "Midsummer" chipset tech ref (large, detailed). |
| [Other Documents/Jaguar TechRef (Flair II - 1992).pdf](Other%20Documents/Jaguar%20TechRef%20%28Flair%20II%20-%201992%29.pdf) | Scanned | Early (1992) Flair II tech ref. |
| `Other Documents/Original version of updated documents/` | Scanned | Original Atari scans (HWBugs, Index, SoftRef, TechOver, TechRef) later restored as the V10 set. |

## Supporting / general

| File | Type | Notes |
|------|------|-------|
| [Index_V10.pdf](Index_V10.pdf) | Text | The original documentation index. |

## Peripherals & examples

| File | Type | Used by |
|------|------|---------|
| [VModem.pdf](VModem.pdf) | Scanned | [peripherals/voice-modem](../peripherals/voice-modem.md) |
| [Samples.pdf](Samples.pdf) | Scanned | [examples/sample-programs](../examples/sample-programs.md) (copyright-library samples excluded) |

## Not yet written (deferred)

| File | Type | Topic |
|------|------|-------|
| [Workshop.pdf](Workshop.pdf) | Scanned | Object-processor / Blitter worked-example tutorials. |
| [Appendix.pdf](Appendix.pdf) | Scanned | Developer FAQ — mostly dev-system/toolchain support; only a few hardware items. |

## Excluded (removed, not redistributed)

These were removed because they are third-party/copyright IP or tied to a specific toolchain rather than the hardware, so they are **not** present in this folder:

| Source | Reason |
|--------|--------|
| `Madmac.pdf`, `ALN.pdf`, `Tools.pdf`, `Debugger.pdf` | toolchain (assembler / linker / debugger / utilities) |
| `librarys.pdf` (3D, JPEG, networking, music) | library IP |
| `Cinepak.pdf` | licensed video codec |
| `QSound_V10.pdf` (+ "Atari Original" copy) | licensed 3D-audio module |
| `GStarted_V10.pdf` / `Getting Started_V10.pdf` (+ "Atari Original" copy) | toolchain-dependent setup guide |

> **Samples caveat:** sample programs may be included, but any sample built around a copyright library must be excluded; samples depending only on standard/freely-redistributable code (e.g. C `stdlib`) are fine.

<!-- nav:bottom -->
---

◀ **Prev:** [Glossary](../reference/glossary.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [Home](../index.md) ▶

**Jump to:** [Architecture](../architecture/overview.md) · [Memory Map](../architecture/memory-map.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md) · [CD-ROM](../cdrom/overview.md)
<!-- /nav:bottom -->
