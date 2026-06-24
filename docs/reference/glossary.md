<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ Reference ▸ **Glossary**
<!-- /nav:top -->

# Glossary

Hardware, chip, and CD-ROM terms used across this reference. CD-specific media
terms (sectors, sessions, subcodes, etc.) are defined in the
[CD-ROM Overview glossary](../cdrom/overview.md#glossary-of-cd-rom-terms).

**On this page:** [Chips & processors](#chips--processors) · [Graphics & display](#graphics--display) · [Memory & system](#memory--system) · [Development hardware & tools](#development-hardware--tools) · [Addresses worth memorising](#addresses-worth-memorising)

## Chips & processors

| Term | Meaning |
|------|---------|
| **Tom** | The graphics custom chip; contains the Object Processor, GPU, Blitter, memory controller, and video timing. See [overview](../architecture/overview.md). |
| **Jerry** | The sound/I-O custom chip; contains the DSP, timers, serial interfaces, and joystick interface. See [overview](../architecture/overview.md). |
| **CPU / 68000** | Motorola 68000 acting as the Jaguar's system *manager* — control flow, not the performance path. |
| **Object Processor (OP)** | Display-list engine in Tom that builds each scan line from the [object list](../tom/object-processor.md). |
| **GPU** | Graphics Processor — RISC co-processor in Tom, 4 KB internal RAM. See [gpu](../tom/gpu.md). |
| **DSP** | Digital Sound Processor — RISC co-processor in Jerry, 8 KB internal RAM. See [dsp](../jerry/dsp.md). |
| **Blitter** | Block-transfer engine in Tom (move/fill, line draw, Gouraud, Z-buffer). See [blitter](../tom/blitter.md). |
| **Butch** | The CD-ROM interface chip; routes CD data to Jerry's I²S interrupt. Rev 1 vs 2 affects which CD BIOS runs. See [cdrom/hardware](../cdrom/hardware.md). |
| **Midsummer** | Codename of a later/enhanced Jaguar chipset (see the Midsummer Tech Ref source). |

## Graphics & display

| Term | Meaning |
|------|---------|
| **Object list** | The list of objects (commands) the OP processes each scan line: bit-map (scaled/unscaled), branch, stop, and GPU-interrupt objects. |
| **Object header** | Per-object descriptor — 2 phrases (unscaled) or 3 phrases (scaled). |
| **Line buffer** | One of two 360×32-bit RAMs the OP writes a scan line into while the other is displayed. |
| **Phrase** | 64 bits (8 bytes) — the Jaguar's wide-memory transfer unit. The Blitter's *phrase mode* reads/writes 64 bits at a time. |
| **Long / longword** | 32 bits (4 bytes). CD data maintains long alignment only. |
| **CRY** | Cyan-Red-Intensity — a 16-bit colour model that is cheap to Gouraud-shade and near-indistinguishable from 24-bit RGB. See [color-cry](../tom/color-cry.md). |
| **CLUT** | Colour Look-Up Table at `$F00400`–`$F007FE`; palette for sub-16-bit objects. |
| **Gouraud shading** | Hardware smooth shading across a span; 16-bit-pixel mode only. |
| **Z-buffer** | Per-pixel depth test the Blitter can apply; 16-bit-pixel mode only. |
| **SCRENX / SRCENX** | Blitter "source enable extra (read)" — forces an extra phrase read when first-write data isn't in the first read. See [blitter](../tom/blitter.md). |

## Memory & system

| Term | Meaning |
|------|---------|
| **Phrase mode** | Blitter mode operating on 64-bit phrases; both address generators must be in it. |
| **Scoreboarding** | GPU/DSP pipeline hazard mechanism that stalls on register dependencies. See [gpu](../tom/gpu.md). |
| **I²S** | Inter-IC Sound serial format used for Jerry's audio output and the CD data path. See [audio](../jerry/audio.md). |
| **ComLynx** | Asynchronous serial link (shared with the Atari Lynx) on Jerry. See [serial-io](../jerry/serial-io.md). |
| **NVRAM / EEPROM** | 128-byte serial EEPROM in cartridges for saves/high scores; accessed via `JOYSTICK` bit 0. |

## Development hardware & tools

| Term | Meaning |
|------|---------|
| **Stubulator** | A retail Jaguar modified with a debugging boot ROM + stop-button cable — the dev "Jaguar Test Station." See [TechOver](../architecture/overview.md). |
| **ROMulator / Alpine Board** | Dev board emulating a ROM cartridge with battery-backed SRAM (2–4 MB) at `$800000`, plus host PC link. |
| **Debugger stub** | Debug ROM code in the dev console; reserves low memory (below `$4000`) and the first `$2000` of ROM space. |
| **SkunkBoard** | A USB Flash-ROM cartridge for development on a retail console (4 MB or 8 MB). |
| **MADMAC / SMAC** | Atari's macro assembler (68000 + GPU/DSP). |
| **ALN / SLN** | The object linker. |
| **DB / RDBJAG / WDB** | The Atari symbolic debugger and its terminal (RDBJAG) and windowed (WDB) front ends. |
| **CD BIOS** | The required API layer for all CD access. See [cdrom/bios-api](../cdrom/bios-api.md). |
| **Partition marker** | A 64-byte block of 16 identical longwords tagging a data block on a Jaguar CD. See [cdrom/data-format](../cdrom/data-format.md#partition-markers). |

## Addresses worth memorising

| Address / range | Meaning |
|-----------------|---------|
| `$000000`–`$003FFF` | reserved low RAM in the dev environment (debugger stub); usable RAM starts at `$004000` |
| `$000000`–`$1FFFFF` | 2 MB main DRAM (64-bit) |
| `$00002C00` | where the CD Boot ROM / `CD_getoc` places the TOC |
| `$1FFFFC` | recommended 68000 stack location |
| `$800000`–`$801FFF` | reserved cartridge/ROM security-code area |
| `$800000` | cartridge / ROMulator base (32-bit) |
| `$802000` | mandatory start address for cartridge programs |
| `$F00000`–`$F0FFFF` | Tom register / internal space |
| `$F03000`–`$F03FFF` | GPU internal RAM (4 KB) |
| `$F10000`–`$F1FFFF` | Jerry register / internal space |
| `$F14000` | `JOYSTICK` register |
| `$F1B000`–`$F1CFFF` | DSP internal RAM (8 KB) |

## See also

- [Memory Map / Register List](../architecture/memory-map.md)
- [Complete Register List](register-list.md)
- [System Architecture Overview](../architecture/overview.md)
- [CD-ROM glossary](../cdrom/overview.md#glossary-of-cd-rom-terms)

<!-- nav:bottom -->
---

◀ **Prev:** [RISC Instruction Set (GPU/DSP)](risc-instruction-set.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [Source Documents](../sources/index.md) ▶

**Jump to:** [Architecture](../architecture/overview.md) · [Memory Map](../architecture/memory-map.md) · [Registers](register-list.md) · [Instructions](risc-instruction-set.md) · [CD-ROM](../cdrom/overview.md)
<!-- /nav:bottom -->
