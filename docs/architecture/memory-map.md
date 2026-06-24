<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ Architecture & Memory ▸ **Memory Map / Register List**
<!-- /nav:top -->

# Memory Map / Register List

The Jaguar address space, memory banks, and the complete hardware register list.

> **Source:** *Technical Reference Manual* (V10), pp. 5–7 (register list);
> *Software Reference Manual — Tom & Jerry* (V10), pp. 9–20 (memory controller,
> internal map); *Technical Overview* (V10), p. 9. © Atari Corp. 1995.

## System memory layout

| Region | Address | Notes |
|--------|---------|-------|
| Main DRAM | `$000000`–`$1FFFFF` | Single 2 MB bank, 64 bits wide |
| *(reserved by debugger/stub)* | `$000000`–`$003FFF` | Lower 16 KB — **do not use** below `$4000` in dev environment |
| Cartridge / ROMulator / ROM | `$800000`+ | Treated as 32-bit; cartridge programs start at **`$802000`** |
| *(reserved security code)* | `$800000`–`$801FFF` | First 16 KB of ROM space reserved |
| Tom internal registers | `$F00000`+ | Video/OP/GPU/Blitter; see below |
| GPU/DSP internal SRAM & regs | `$F00000`+ | 32-bit access only, long-word aligned |
| Jerry registers | `$F10000`+ | Clocks, timers, serial, DSP |

Key rules (see [Architecture Overview](overview.md#memory-and-the-data-path)):
- Main RAM is **64 bits wide**; GPU/DSP/Blitter internal registers are **32 bits
  wide and must be accessed as 32-bit longs** by the 68000.
- Recommended 68000 stack location: **`$1FFFFC`** (one long below end of RAM).
- Use `move`, not `clr.l`, to clear GPU/DSP internal RAM (hardware bug).

## Do-not-touch registers

The BOOTROM (retail) or STUBULATOR (dev console) configures these at boot. The
`CLK2`, `CLK3`, and `HP` settings in particular must be correct for the hardware
to work at all and to prevent dot crawl. **Never write to them:**

`MEMCON1` `MEMCON2` `CLK1` `CLK2` `CLK3` `HP` `HS` `HBE` `HBB` `HVS` `HEQ`
`VP` `VBB` `VBE` `VS` `VEE`

Likewise, the Object Processor and `VMODE` are started by boot code, after which a
single *stop object* displays a blank screen and lets the PLL settle (~1 s at
startup). **Never turn video off** by writing 0 to `VMODE`.

Audio is **muted after a power-on reset** — enable it by setting bit 8 of the
`JOYSTICK` register.

## Register list

Legend: **RW** = read/write, **WO** = write-only, **RO** = read-only.
Equate names are from `JAGUAR.INC`. Addresses are offsets in the `$F-----` space.
**Bold** entries are boot-configured — do not modify (see above).

### System set-up / video registers (Tom)

| Equate | Register | Address | Mode |
|--------|----------|---------|------|
| **MEMCON1** | Memory Control Register 1 | `$F00000` | RW |
| **MEMCON2** | Memory Control Register 2 | `$F00002` | RW |
| HC | Horizontal Count | `$F00004` | RW |
| VC | Vertical Count | `$F00006` | RW |
| LPH | Horizontal Light Pen | `$F00008` | RO |
| LPV | Vertical Light Pen | `$F0000A` | RO |
| OB[0-3] | Object Code | `$F00010`–`$F00016` | RO |
| OLP | Object List Pointer | `$F00020` | WO |
| OBF | Object Processor Flag | `$F00026` | WO |
| VMODE | Video Mode | `$F00028` | WO |
| BORD1 | Border Colour (Red & Green) | `$F0002A` | WO |
| BORD2 | Border Colour (Blue) | `$F0002C` | WO |
| **HP** | Horizontal Period | `$F0002E` | WO |
| **HBB** | Horizontal Blanking Begin | `$F00030` | WO |
| **HBE** | Horizontal Blanking End | `$F00032` | WO |
| **HS** | Horizontal Sync | `$F00034` | WO |
| **HVS** | Horizontal Vertical Sync | `$F00036` | WO |
| HDB1 | Horizontal Display Begin 1 | `$F00038` | WO |
| HDB2 | Horizontal Display Begin 2 | `$F0003A` | WO |
| HDE | Horizontal Display End | `$F0003C` | WO |
| **VP** | Vertical Period | `$F0003E` | WO |
| **VBB** | Vertical Blanking Begin | `$F00040` | WO |
| **VBE** | Vertical Blanking End | `$F00042` | WO |
| **VS** | Vertical Sync | `$F00044` | WO |
| VDB | Vertical Display Begin | `$F00046` | WO |
| VDE | Vertical Display End | `$F00048` | WO |
| VEB | Vertical Equalisation Begin | `$F0004A` | WO |
| **VEE** | Vertical Equalisation End | `$F0004C` | WO |
| VI | Vertical Interrupt | `$F0004E` | WO |
| PIT[0-1] | Programmable Interrupt Timer | `$F00050`–`$F00052` | WO |
| **HEQ** | Horizontal Equalisation End | `$F00054` | WO |
| BG | Background Colour | `$F00058` | WO |
| INT1 | CPU Interrupt Control Register | `$F000E0` | RW |
| INT2 | CPU Interrupt Resume Register | `$F000E2` | WO |
| CLUT | Colour Look-Up Table | `$F00400`–`$F007FE` | RW |
| LBUF | Line Buffer | `$F00800`–`$F01D9E` | RW |

> Note: the V10 source lists both HBB and the next entry as "Horizontal Blanking
> Begin"; the second (`$F00032`, `HBE`) is **Horizontal Blanking End**.

### GPU registers (Tom)

| Equate | Register | Address | Mode |
|--------|----------|---------|------|
| G_FLAGS | GPU Flags Register | `$F02100` | RW |
| G_MTXC | Matrix Control Register | `$F02104` | WO |
| G_MTXA | Matrix Address Register | `$F02108` | WO |
| G_END | Data Organisation Register | `$F0210C` | WO |
| G_PC | GPU Program Counter | `$F02110` | RW |
| G_CTRL | GPU Control/Status Register | `$F02114` | RW |
| G_HIDATA | GPU High Data Register | `$F02118` | RW |
| G_REMAIN | GPU Division Remainder | `$F0211C` | RO |
| G_DIVCTRL | GPU Division Control | `$F0211C` | WO |

GPU internal RAM is 4 KB starting at `$F03000`. See [Graphics Processor](../tom/gpu.md).

### Blitter registers (Tom)

Footnotes from source: `*` must be refreshed after a BLIT; `**` must be refreshed
if used to store dynamic data (inner-loop read, or `GOURD`/`GOURZ` set);
`***` Software Reference Manual v2.2 & earlier reversed the order of these
descriptions — the equates are unchanged.

| Equate | Register | Address | Mode |
|--------|----------|---------|------|
| A1_BASE | A1 Base Register | `$F02200` | WO |
| A1_FLAGS | A1 Flags Register | `$F02204` | WO |
| A1_CLIP | A1 Clipping Size | `$F02208` | WO |
| A1_PIXEL | A1 Pixel Pointer | `$F0220C` | RW* |
| A1_STEP | A1 Step Value (integer) | `$F02210` | WO |
| A1_FSTEP | A1 Step Value (fraction) | `$F02214` | WO |
| A1_FPIXEL | A1 Pixel Pointer (fraction) | `$F02218` | RW* |
| A1_INC | A1 Increment (integer) | `$F0221C` | WO |
| A1_FINC | A1 Increment (fraction) | `$F02220` | WO |
| A2_BASE | A2 Base Register | `$F02224` | WO |
| A2_FLAGS | A2 Flags Register | `$F02228` | WO |
| A2_MASK | A2 Window Mask | `$F0222C` | WO |
| A2_PIXEL | A2 Pixel Pointer | `$F02230` | RW* |
| A2_STEP | A2 Step Value (integer) | `$F02234` | WO |
| B_CMD | Command/Status Register | `$F02238` | RW* |
| B_COUNT | Counters Register | `$F0223C` | WO* |
| B_SRCD | Source Data Register | `$F02240` | WO** |
| B_DSTD | Destination Data Register | `$F02248` | WO** |
| B_DSTZ | Destination Z Register | `$F02250` | WO** |
| B_SRCZ1 | Source Z Register 1 (integer) | `$F02258` | WO** |
| B_SRCZ2 | Source Z Register 2 (fraction) | `$F02260` | WO** |
| B_PATD | Pattern Data Register | `$F02268` | WO** |
| B_IINC | Intensity Increment | `$F02270` | WO |
| B_ZINC | Z Increment | `$F02274` | WO |
| B_STOP | Collision Control | `$F02278` | WO |
| B_I3 | Intensity 3 *** | `$F0227C` | WO |
| B_I2 | Intensity 2 *** | `$F02280` | WO |
| B_I1 | Intensity 1 *** | `$F02284` | WO |
| B_I0 | Intensity 0 *** | `$F02288` | WO |
| B_Z3 | Z3 *** | `$F0228C` | WO |
| B_Z2 | Z2 *** | `$F02290` | WO |
| B_Z1 | Z1 *** | `$F02294` | WO |
| B_Z0 | Z0 *** | `$F02298` | WO |

See [Blitter](../tom/blitter.md) for register semantics and modes.

### Jerry — clocks, timers, serial

| Equate | Register | Address | Mode |
|--------|----------|---------|------|
| **CLK1** | Processor clock divider | `$F10010` | WO |
| **CLK2** | Video clock divider | `$F10012` | WO |
| **CLK3** | Chroma clock divider | `$F10014` | WO |
| JPIT1 | Timer 1 Pre-scaler | `$F10000` | WO |
| JPIT2 | Timer 1 Divider | `$F10002` | WO |
| JPIT3 | Timer 2 Pre-scaler | `$F10004` | WO |
| JPIT4 | Timer 2 Divider | `$F10006` | WO |
| J_INT | Interrupt Control Register | `$F10020` | RW |
| SCLK | Serial Clock Frequency | `$F1A150` | WO |
| SMODE | Serial Mode | `$F1A154` | WO |
| LTXD1¹ | Left transmit data | `$F1A148` | WO |
| RTXD1¹ | Right transmit data | `$F1A14C` | WO |
| LRXD1¹ | Left receive data | `$F1A148` | RO |
| RRXD1¹ | Right receive data | `$F1A14C` | RO |
| L_I2S | Left I²S Serial Interface | `$F1A148` | RW |
| R_I2S | Right I²S Serial Interface | `$F1A14C` | RW |
| SSTAT1¹ | Serial Status | `$F1A150` | RO |
| ASICLK1¹ | Asynchronous Serial Interface Clock | `$F10034` | RW |
| ASICTRL1¹ | Asynchronous Serial Control | `$F10032` | WO |
| ASISTAT1¹ | Asynchronous Serial Status | `$F10032` | RO |
| ASIDATA1¹ | Asynchronous Serial Data | `$F10039` | RW |

¹ **Source caveat:** `LTXD`/`RTXD`/`LRXD`/`RRXD` and the `SSTAT`/`ASI*` registers
are *not* in the latest `JAGUAR.INC` (modified 1995-02-16) and presumably should
not be used. Use `L_I2S`/`R_I2S` (same addresses) instead of the L/R TXD/RXD
registers. The ASI registers may have been removed to discourage networked games
because of the UART bug; add them back to `JAGUAR.INC` yourself if needed. See
[Serial I/O](../jerry/serial-io.md).

### Joystick registers (Jerry)

| Equate | Register | Address | Mode |
|--------|----------|---------|------|
| JOYSTICK | Joystick Register | `$F14000` | RW |
| JOYBUTS | Button Register | `$F14002` | RW |

Bit 8 of `JOYSTICK` enables audio; bit 0 is used by EEPROM cartridges (do not rely
on its read state — it is random). See [Controllers](../controllers/controllers.md).

### DSP registers (Jerry)

| Equate | Register | Address | Mode |
|--------|----------|---------|------|
| D_FLAGS | DSP Flags Register | `$F1A100` | RW |
| D_MTXC | DSP Matrix Control Register | `$F1A104` | WO |
| D_MTXA | DSP Matrix Address Register | `$F1A108` | WO |
| D_END | DSP Data Organisation Register | `$F1A10C` | WO |
| D_PC | DSP Program Counter | `$F1A110` | RW |
| D_CTRL | DSP Control/Status Register | `$F1A114` | RW |
| D_MOD | Modulo Instruction Mask | `$F1A118` | WO |
| D_REMAIN | Divide Unit Remainder | `$F1A11C` | RO |
| D_DIVCTRL | Divide Unit Control | `$F1A11C` | WO |
| D_MACHI | Multiply & Accumulate High Result Bits | `$F1A120` | RO |

DSP internal RAM is 8 KB. See [Digital Sound Processor](../jerry/dsp.md).

## See also

- [System Architecture Overview](overview.md)
- [Video & System Clocks, Timing](video-clocks-timing.md)
- [Complete Register List](../reference/register-list.md) — consolidated, all groups

<!-- nav:bottom -->
---

◀ **Prev:** [System Architecture Overview](overview.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [Video & System Clocks, Timing](video-clocks-timing.md) ▶

**Jump to:** [Architecture](overview.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md) · [CD-ROM](../cdrom/overview.md)
<!-- /nav:bottom -->
