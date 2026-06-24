<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ Reference ▸ **Complete Register List**
<!-- /nav:top -->

# Complete Register List

Every Jaguar hardware register, sorted by address for quick lookup. For grouped
tables with context and the boot-time **do-not-touch** warnings, see
[Memory Map / Register List](../architecture/memory-map.md); for bit-field detail
see the per-subsystem docs linked below.

> **Source:** *Technical Reference Manual* (V10), pp. 5–7; equate names from
> `JAGUAR.INC`. © Atari Corp. 1995.

Legend: **RW** read/write · **WO** write-only · **RO** read-only ·
⚠️ boot-configured, do not modify.

## Sorted by address

| Address | Equate | Register | Mode | Subsystem |
|---------|--------|----------|------|-----------|
| `$F00000` | MEMCON1 ⚠️ | Memory Control 1 | RW | [Tom/system](../architecture/memory-map.md) |
| `$F00002` | MEMCON2 ⚠️ | Memory Control 2 | RW | Tom/system |
| `$F00004` | HC | Horizontal Count | RW | [Video](../architecture/video-clocks-timing.md) |
| `$F00006` | VC | Vertical Count | RW | Video |
| `$F00008` | LPH | Horizontal Light Pen | RO | Video |
| `$F0000A` | LPV | Vertical Light Pen | RO | Video |
| `$F00010`–`16` | OB[0-3] | Object Code | RO | [Object Processor](../tom/object-processor.md) |
| `$F00020` | OLP | Object List Pointer | WO | Object Processor |
| `$F00026` | OBF | Object Processor Flag | WO | Object Processor |
| `$F00028` | VMODE | Video Mode | WO | Video |
| `$F0002A` | BORD1 | Border Colour (R&G) | WO | Video |
| `$F0002C` | BORD2 | Border Colour (B) | WO | Video |
| `$F0002E` | HP ⚠️ | Horizontal Period | WO | Video |
| `$F00030` | HBB ⚠️ | Horizontal Blanking Begin | WO | Video |
| `$F00032` | HBE ⚠️ | Horizontal Blanking End | WO | Video |
| `$F00034` | HS ⚠️ | Horizontal Sync | WO | Video |
| `$F00036` | HVS ⚠️ | Horizontal Vertical Sync | WO | Video |
| `$F00038` | HDB1 | Horizontal Display Begin 1 | WO | Video |
| `$F0003A` | HDB2 | Horizontal Display Begin 2 | WO | Video |
| `$F0003C` | HDE | Horizontal Display End | WO | Video |
| `$F0003E` | VP ⚠️ | Vertical Period | WO | Video |
| `$F00040` | VBB ⚠️ | Vertical Blanking Begin | WO | Video |
| `$F00042` | VBE ⚠️ | Vertical Blanking End | WO | Video |
| `$F00044` | VS ⚠️ | Vertical Sync | WO | Video |
| `$F00046` | VDB | Vertical Display Begin | WO | Video |
| `$F00048` | VDE | Vertical Display End | WO | Video |
| `$F0004A` | VEB | Vertical Equalisation Begin | WO | Video |
| `$F0004C` | VEE ⚠️ | Vertical Equalisation End | WO | Video |
| `$F0004E` | VI | Vertical Interrupt | WO | Video |
| `$F00050`–`52` | PIT[0-1] | Programmable Interrupt Timer | WO | Video |
| `$F00054` | HEQ ⚠️ | Horizontal Equalisation End | WO | Video |
| `$F00058` | BG | Background Colour | WO | Video |
| `$F000E0` | INT1 | CPU Interrupt Control | RW | Object Processor |
| `$F000E2` | INT2 | CPU Interrupt Resume | WO | Object Processor |
| `$F00400`–`7FE` | CLUT | Colour Look-Up Table | RW | [CRY/colour](../tom/color-cry.md) |
| `$F00800`–`1D9E` | LBUF | Line Buffer | RW | Object Processor |
| `$F02100` | G_FLAGS | GPU Flags | RW | [GPU](../tom/gpu.md) |
| `$F02104` | G_MTXC | Matrix Control | WO | GPU |
| `$F02108` | G_MTXA | Matrix Address | WO | GPU |
| `$F0210C` | G_END | Data Organisation | WO | GPU |
| `$F02110` | G_PC | GPU Program Counter | RW | GPU |
| `$F02114` | G_CTRL | GPU Control/Status | RW | GPU |
| `$F02118` | G_HIDATA | GPU High Data | RW | GPU |
| `$F0211C` | G_REMAIN | Division Remainder | RO | GPU |
| `$F0211C` | G_DIVCTRL | Division Control | WO | GPU |
| `$F02200` | A1_BASE | A1 Base | WO | [Blitter](../tom/blitter.md) |
| `$F02204` | A1_FLAGS | A1 Flags | WO | Blitter |
| `$F02208` | A1_CLIP | A1 Clipping Size | WO | Blitter |
| `$F0220C` | A1_PIXEL | A1 Pixel Pointer | RW | Blitter |
| `$F02210` | A1_STEP | A1 Step (integer) | WO | Blitter |
| `$F02214` | A1_FSTEP | A1 Step (fraction) | WO | Blitter |
| `$F02218` | A1_FPIXEL | A1 Pixel Pointer (fraction) | RW | Blitter |
| `$F0221C` | A1_INC | A1 Increment (integer) | WO | Blitter |
| `$F02220` | A1_FINC | A1 Increment (fraction) | WO | Blitter |
| `$F02224` | A2_BASE | A2 Base | WO | Blitter |
| `$F02228` | A2_FLAGS | A2 Flags | WO | Blitter |
| `$F0222C` | A2_MASK | A2 Window Mask | WO | Blitter |
| `$F02230` | A2_PIXEL | A2 Pixel Pointer | RW | Blitter |
| `$F02234` | A2_STEP | A2 Step (integer) | WO | Blitter |
| `$F02238` | B_CMD | Command/Status | RW | Blitter |
| `$F0223C` | B_COUNT | Counters | WO | Blitter |
| `$F02240` | B_SRCD | Source Data | WO | Blitter |
| `$F02248` | B_DSTD | Destination Data | WO | Blitter |
| `$F02250` | B_DSTZ | Destination Z | WO | Blitter |
| `$F02258` | B_SRCZ1 | Source Z 1 (integer) | WO | Blitter |
| `$F02260` | B_SRCZ2 | Source Z 2 (fraction) | WO | Blitter |
| `$F02268` | B_PATD | Pattern Data | WO | Blitter |
| `$F02270` | B_IINC | Intensity Increment | WO | Blitter |
| `$F02274` | B_ZINC | Z Increment | WO | Blitter |
| `$F02278` | B_STOP | Collision Control | WO | Blitter |
| `$F0227C` | B_I3 | Intensity 3 | WO | Blitter |
| `$F02280` | B_I2 | Intensity 2 | WO | Blitter |
| `$F02284` | B_I1 | Intensity 1 | WO | Blitter |
| `$F02288` | B_I0 | Intensity 0 | WO | Blitter |
| `$F0228C` | B_Z3 | Z3 | WO | Blitter |
| `$F02290` | B_Z2 | Z2 | WO | Blitter |
| `$F02294` | B_Z1 | Z1 | WO | Blitter |
| `$F02298` | B_Z0 | Z0 | WO | Blitter |
| `$F10000` | JPIT1 | Timer 1 Pre-scaler | WO | [Audio/Jerry](../jerry/audio.md) |
| `$F10002` | JPIT2 | Timer 1 Divider | WO | Audio/Jerry |
| `$F10004` | JPIT3 | Timer 2 Pre-scaler | WO | Audio/Jerry |
| `$F10006` | JPIT4 | Timer 2 Divider | WO | Audio/Jerry |
| `$F10010` | CLK1 ⚠️ | Processor clock divider | WO | Jerry |
| `$F10012` | CLK2 ⚠️ | Video clock divider | WO | Jerry |
| `$F10014` | CLK3 ⚠️ | Chroma clock divider | WO | Jerry |
| `$F10020` | J_INT | Interrupt Control | RW | Audio/Jerry |
| `$F10032` | ASICTRL1 † | Async Serial Control | WO | [Serial I/O](../jerry/serial-io.md) |
| `$F10032` | ASISTAT1 † | Async Serial Status | RO | Serial I/O |
| `$F10034` | ASICLK1 † | Async Serial Clock | RW | Serial I/O |
| `$F10039` | ASIDATA1 † | Async Serial Data | RW | Serial I/O |
| `$F14000` | JOYSTICK | Joystick Register | RW | [Controllers](../controllers/controllers.md) |
| `$F14002` | JOYBUTS | Button Register | RW | Controllers |
| `$F1A100` | D_FLAGS | DSP Flags | RW | [DSP](../jerry/dsp.md) |
| `$F1A104` | D_MTXC | DSP Matrix Control | WO | DSP |
| `$F1A108` | D_MTXA | DSP Matrix Address | WO | DSP |
| `$F1A10C` | D_END | DSP Data Organisation | WO | DSP |
| `$F1A110` | D_PC | DSP Program Counter | RW | DSP |
| `$F1A114` | D_CTRL | DSP Control/Status | RW | DSP |
| `$F1A118` | D_MOD | Modulo Instruction Mask | WO | DSP |
| `$F1A11C` | D_REMAIN | Divide Remainder | RO | DSP |
| `$F1A11C` | D_DIVCTRL | Divide Control | WO | DSP |
| `$F1A120` | D_MACHI | MAC High Result Bits | RO | DSP |
| `$F1A148` | L_I2S / LTXD1 / LRXD1 | Left I²S / transmit / receive | RW | [Audio](../jerry/audio.md) |
| `$F1A14C` | R_I2S / RTXD1 / RRXD1 | Right I²S / transmit / receive | RW | Audio |
| `$F1A150` | SCLK / SSTAT1 † | Serial Clock Freq / Status | WO/RO | Audio |
| `$F1A154` | SMODE | Serial Mode | WO | Audio |

† Caveat: these async-serial / status registers are **not** in the latest
`JAGUAR.INC` (1995-02-16) and presumably should not be used (UART bug). Prefer
`L_I2S`/`R_I2S` over the L/R TXD/RXD aliases. See
[memory-map notes](../architecture/memory-map.md#jerry--clocks-timers-serial).

## See also

- [Memory Map / Register List](../architecture/memory-map.md) — grouped, with warnings
- [Glossary](glossary.md)
- Per-subsystem bit-field detail: [Object Processor](../tom/object-processor.md),
  [GPU](../tom/gpu.md), [Blitter](../tom/blitter.md), [DSP](../jerry/dsp.md),
  [Audio](../jerry/audio.md), [Serial I/O](../jerry/serial-io.md),
  [Video timing](../architecture/video-clocks-timing.md),
  [Controllers](../controllers/controllers.md)

<!-- nav:bottom -->
---

◀ **Prev:** [Sample Programs](../examples/sample-programs.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [RISC Instruction Set (GPU/DSP)](risc-instruction-set.md) ▶

**Jump to:** [Architecture](../architecture/overview.md) · [Memory Map](../architecture/memory-map.md) · [Instructions](risc-instruction-set.md) · [Glossary](glossary.md) · [CD-ROM](../cdrom/overview.md)
<!-- /nav:bottom -->
