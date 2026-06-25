# Atari Jaguar Developer Reference

A cross-linked developer reference for the **Atari Jaguar** console, drawn from
the original Atari Corp. documentation (1993–1995) and the community-restored
"Version 10" editions. It covers the **core system** (the Tom and Jerry chip set)
and the **CD-ROM subsystem**.

## Start here

- New to the Jaguar? Begin at the [System Architecture Overview](architecture/overview.md).
- Looking for an address or register? See the [Memory Map / Register List](architecture/memory-map.md) or the [Complete Register List](reference/register-list.md).
- Writing CD-ROM code? Start at the [CD-ROM Subsystem Overview](cdrom/overview.md).

## Core system

The Jaguar is a custom chip set built around two ASICs — **Tom** (graphics/video)
and **Jerry** (sound/I-O) — plus a Motorola 68000 acting as system manager.

### CPU
- [Motorola 68000 — Programmer's Model](cpu/68000.md) — registers, addressing modes, instruction set, timings, exceptions

### Architecture & memory
- [System Architecture Overview](architecture/overview.md) — the five processors and how they cooperate
- [Memory Map / Register List](architecture/memory-map.md) — address space, banks, hardware registers
- [Video & System Clocks, Timing](architecture/video-clocks-timing.md) — clock generation, NTSC/PAL timing, video ports
- [Hardware Bugs & Warnings](architecture/hardware-bugs.md) — known silicon bugs and cautions
- [Cartridges, EEPROM Saves & ROM Building](architecture/cartridges.md) — cartridge port, on-cart saves, EPROM/EEPROM parts
- [Connectors, Pinouts & Power](architecture/connectors-power.md) — power supply, current draw, connector/schematic hub

### Tom — graphics & video
- [Object Processor](tom/object-processor.md) — the display list engine (sprite + framebuffer hybrid)
- [Graphics Processor (GPU)](tom/gpu.md) — the RISC graphics co-processor
- [Blitter](tom/blitter.md) — block move/fill, line draw, Gouraud shading, Z-buffer
- [CRY Color & Color Mapping](tom/color-cry.md) — the CRY color model and RGB modes

### Jerry — sound & I/O
- [Digital Sound Processor (DSP)](jerry/dsp.md) — the RISC sound co-processor
- [Audio Subsystem & Synthesis](jerry/audio.md) — sample playback, timers, output
- [Serial I/O — ComLynx, MIDI, Synchronous Serial](jerry/serial-io.md)

### Input
- [Controllers & Controller Ports](controllers/controllers.md) — Joypad, rotary, Team Tap, advanced controllers

## Peripherals

- [Jaguar Voice Modem](peripherals/voice-modem.md) — voice/data modem interface, control flow, command reference

## CD-ROM subsystem

- [CD-ROM Subsystem Overview](cdrom/overview.md) — what the Jaguar CD is and how it differs from other CD systems
- [CD-ROM BIOS API](cdrom/bios-api.md) — the calls your code uses to talk to the drive
- [Disc & Data Format](cdrom/data-format.md) — sessions, tracks, frames, partition markers, audio
- [Programming Procedures & Guidelines](cdrom/programming-guide.md) — boot, read, error handling, do's & don'ts
- [CD-ROM Hardware](cdrom/hardware.md) — drive module, schematic, connections

## Programming examples

- [Sample Programs](examples/sample-programs.md) — hardware demos (copyright-library samples excluded)

## Reference

- [Complete Register List](reference/register-list.md)
- [RISC Instruction Set (GPU/DSP)](reference/risc-instruction-set.md)
- [Glossary](reference/glossary.md)
- [Source Provenance](reference/source-documents.md) — which source each page is built from

## Source files

- [Source files index](sources/index.md) — the original PDFs, with descriptions, notes, and download links

---

*Sources: Atari Corp. Jaguar developer documentation, © Atari Corporation
1993–1995; "Version 10" restorations © Stephen Moss, 2021. Reproduced here for
developer reference.*
