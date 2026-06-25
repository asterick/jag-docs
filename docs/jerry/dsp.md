<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ Jerry — Sound & I/O ▸ **Digital Sound Processor (DSP)**
<!-- /nav:top -->

# Digital Sound Processor (DSP)

The DSP is Jerry's RISC processor — a sound-optimized variant of Tom's GPU with 8 KB of internal RAM, a wave-table ROM, extended-precision multiply/accumulate, circular-buffer addressing, and full bus-master access to the system.

> **Source:** *Software Reference Manual — Tom & Jerry* (V10), pp. 82–88; *Appendix* (Atari original, 26 April 1995), Appendix A. © Atari Corp. 1995.

## Overview

The DSP is part of the Jerry chip and is a variant of the GPU within Tom. It uses a very similar instruction set and programming model, with certain differences. The DSP has full access to the system memory map as a bus master, and its internal memory may be accessed by other bus masters within the Jaguar system.

The DSP performs two roles within the Jaguar:

- **Sound synthesis** (its primary function) — playback of sampled sounds, algorithmic sound generation, or a mixture of the two. Because it is a fast, general-purpose processor it supports a broad range of synthesis techniques.
- **Additional graphics processing** — since many sound techniques do not require its full power, it can also be used as a second graphics processor (e.g. matrix multiplies for 3D object rotation), with sound synthesis running under a sample-rate interrupt.

Note that the DSP has only a 16-bit interface to external memory, so its external bus bandwidth is lower than the GPU's.

## How it differs from the GPU

The DSP shares the GPU's programming model, design philosophy, pipe-lining behavior, load/store operations, program control flow, divide unit, register file, and external CPU access. (Refer to the corresponding sections of the GPU description — see [Graphics Processor (GPU)](../tom/gpu.md).) The DSP adds several optimizations for sound processing:

- Higher-precision (40-bit) multiply/accumulate operations.
- Hardware circular buffer management.
- Audio wave tables in local ROM.
- Additional local fast RAM (8 KB vs. the GPU's 4 KB).
- Audio hardware (PWM DACs and the synchronous serial interface) mapped within its internal address space.
- Signed saturation functions (`SAT16S`, `SAT32S`) replacing the GPU's unsigned saturation functions.

## Memory Map

The DSP has 8 KB of local fast RAM (twice as much as the GPU) and 2 KB of wave tables for sound synthesis:

| Address range | Contents |
|---------------|----------|
| `$F1A000` – `$F1A1FF` | DSP control registers |
| `$F1B000` – `$F1CFFF` | Local RAM (8 KB) |
| `$F1D000` – `$F1DFFF` | Wave table ROM (2 KB) |

DSP-internal addresses are only available as 16-bit memory, into which 32-bit transfers must be performed in the order **low address then high address**.

## Wave Table ROM

The wave table ROM contains eight 128-entry wave tables. Entries are signed 16-bit values, sign-extended to 32 bits, so the ROM appears to occupy 1K of 32-bit locations. Only the bottom 16 bits are significant.

| Address  | Equate        | Wave |
|----------|---------------|------|
| `$F1D000` | `ROM_TRI`     | A triangle wave |
| `$F1D200` | `ROM_SINE`    | A full-wave sine |
| `$F1D400` | `ROM_AMSINE`  | An amplitude-modulated sine wave |
| `$F1D600` | `ROM_12W`     | A sine wave and its second-order harmonic |
| `$F1D800` | `ROM_CHIRP16` | A chirp — a sine wave increasing in frequency |
| `$F1DA00` | `ROM_NTRI`    | A triangle wave with noise superimposed |
| `$F1DC00` | `ROM_DELTA`   | A spike |
| `$F1DE00` | `ROM_NOISE`   | White noise |

## Arithmetic and Saturation

The DSP replaces the GPU's unsigned saturation functions with two signed operations:

- **`SAT16S`** — takes a signed 32-bit operand and saturates it to a signed 16-bit value: values less than `$FFFF8000` become `$FFFF8000`, and values greater than `$00007FFF` become `$00007FFF`. It operates only on its 32-bit register operand and takes no account of the accumulator overflow bits.
- **`SAT32S`** — takes a signed 40-bit operand (see *Extended Precision Multiply/Accumulates* below) and saturates it to a signed 32-bit value in a similar manner.

## Extended Precision Multiply / Accumulates

When multiply/accumulate operations are performed using the `IMULTN`, `IMACN` and `RESMAC` instructions, or the `MMULT` instruction, the accumulated result is calculated as a **40-bit signed integer**. The top eight bits are effectively overflow bits; after a `RESMAC` they are readable at `$F1A120` (`D_MACHI`).

`SAT32S` takes its 40-bit input as the register operand (low 32 bits) plus the eight accumulator overflow bits (top 8 bits), and saturates the 40-bit signed integer to 32 bits — values below `$FF80000000` become `$FF80000000`, and values above `$007FFFFFFF` become `$007FFFFFFF`.

`SAT32S` should therefore only be applied to the result of a multiply/accumulate operation, and before any further multiply/accumulate operations are performed.

## Circular Buffer Management

Circular buffers are common in DSP algorithms (sample looping, FIFOs, etc.), so there is hardware support for addressing them. Buffers must be 2ⁿ words long and aligned to a 2ⁿ boundary, where n is any practical value.

Support takes the form of two variants of `ADDQ` and `SUBQ` — namely **`ADDQMOD`** and **`SUBQMOD`** — which update pointers with the value wrapping (counting modulo 2ⁿ). This is controlled by the modulo register (`D_MOD`), which masks the result of these instructions: where a bit is `1` the result is unaffected, where it is `0` the result may be modified. Normally the high bits are set to `1` and the low bits to `0`.

## Divide Unit

Refer to the GPU's *Divide Unit* section. The DSP exposes the divide-unit control and remainder via `D_DIVCTRL` and `D_REMAIN` (see registers below). The divide unit can perform either 32-bit unsigned integer division or unsigned 16.16 fixed-point division, selected by the `DIV_OFFSET` bit.

## Interrupts

The DSP has six interrupt sources, allocated as follows (refer to the GPU *Interrupts* section for general behavior):

| #  | Interrupt           |
|----|---------------------|
| 5  | External interrupt 1 |
| 4  | External interrupt 0 |
| 3  | Timer interrupt 2    |
| 2  | Timer interrupt 1    |
| 1  | I²S interface interrupt |
| 0  | CPU interrupt        |

The external interrupts come from additional Jaguar hardware outside the Tom and Jerry system. The Timer interrupts are from Jerry's local programmable timers, the I²S interrupt is from the local synchronous serial interface, and the CPU interrupt is generated by any processor writing to the DSP control register.

## Program Control Flow

Refer to the GPU *Program Control Flow* section. The DSP runs from local RAM; the program counter (`D_PC`) must always be written before setting the `DSPGO` control bit.

## Peripheral I/O in DSP Space

Certain peripheral I/O functions are mapped into the internal DSP space for higher efficiency when the DSP is controlling them. These are effectively 32-bit locations: the **PWM DACs** and the **synchronous serial interface**. (See [Audio Subsystem & Synthesis](audio.md) and [Serial I/O](serial-io.md).)

## Internal Registers

| Register     | Description                          | Address   | Access |
|--------------|--------------------------------------|-----------|--------|
| `D_FLAGS`    | DSP Flags Register                   | `$F1A100` | RW |
| `D_MTXC`     | DSP Matrix Control Register          | `$F1A104` | WO |
| `D_MTXA`     | DSP Matrix Address Register          | `$F1A108` | WO |
| `D_END`      | DSP Data Organization Register       | `$F1A10C` | WO |
| `D_PC`       | DSP Program Counter                  | `$F1A110` | RW |
| `D_CTRL`     | DSP Control/Status Register          | `$F1A114` | RW |
| `D_MOD`      | DSP Modulo Instruction Mask          | `$F1A118` | WO |
| `D_REMAIN`   | DSP Divide Unit Remainder            | `$F1A11C` | RO |
| `D_DIVCTRL`  | DSP Divide Unit Control              | `$F1A11C` | WO |
| `D_MACHI`    | DSP Multiply & Accumulate High Bits  | `$F1A120` | RO |

### D_FLAGS — DSP Flags Register (`$F1A100`, RW)

Provides status and control bits for several important DSP functions.

| Bits  | Equates | Description |
|-------|---------|-------------|
| 0  | `ZERO_FLAG`  | ALU zero flag, set if the result of the last arithmetic operation was zero. Certain arithmetic instructions do not affect the flags. |
| 1  | `CARRY_FLAG` | ALU carry flag, set/cleared by carry/borrow out of the adder/subtractor; reflects carry out of some shift operations, but undefined after other arithmetic operations. |
| 2  | `NEGA_FLAG`  | ALU negative flag, set if the result of the last arithmetic operation was negative. |
| 3  | `IMASK`      | Interrupt mask, set by the interrupt control logic at the start of the service routine; cleared by the service routine writing a `0`. Writing a `1` has no effect. |
| 4–8 | `D_CPUENA`, `D_I2SENA`, `D_TIM1ENA`, `D_TIM2ENA`, `D_EXT0ENA` | Interrupt enable bits for interrupts 0–4 (CPU, I²S, Timer 1, Timer 2, EINT[0]). Status overridden by `IMASK`. |
| 9–13 | `D_CPUCLR`, `D_I2SCLR`, `D_TIM1CLR`, `D_TIM2CLR`, `D_EXT0CLR` | Interrupt latch clear bits for interrupts 0–4. Writing a `0` to any bit leaves it unchanged; the read value is always zero. |
| 14 | `REGPAGE`    | Switches from register bank 0 to register bank 1. Overridden by `IMASK`, which forces register bank 0. |
| 15 | `DMAEN`      | **Must not be set** due to a bug in the Jaguar console — always write as `0`. |
| 16 | `D_EXT1ENA`  | Interrupt enable bit for interrupt 5 (EINT[1]). Functions as bits 4–8. |
| 17 | `D_EXT1CLR`  | Interrupt latch clear bit for interrupt 5. Functions as bits 9–13. |

> **Pipe-lining caveat:** Values written to `D_FLAGS` may not appear to have changed in the following two instructions due to pipe-lining. Writing a value to the flag bits and using those flags in the next instruction will not work properly. If flags set by a `STORE` must be used, ensure at least **two** other instructions lie between the `STORE` and the flags-dependent instruction; for an **indexed** `STORE`, ensure at least **four**.

### D_MTXC — DSP Matrix Control Register (`$F1A104`, WO)

Controls the function of the `MMULT` instruction.

| Bits | Equates       | Description |
|------|---------------|-------------|
| 0–3  | `MATRIX3-15`  | Matrix width, in the range 3 to 15. |
| 4    | `MATCOL`      | When set, accesses the matrix held in memory down one column, as opposed to along one row. |

### D_MTXA — DSP Matrix Address Register (`$F1A108`, WO)

Determines where, in local RAM, the matrix is held.

| Bits | Equates | Description |
|------|---------|-------------|
| 2–11 | —       | Matrix address. |

> **`MMULT` usage.** A matrix held in RAM is **word-packed**, exactly as in the registers. `MMULT` was designed for algorithms that operate on word-packed structures such as the 8×8 matrices of a discrete cosine transform — it is **not intended for general-purpose matrix math**; for that, an explicit `IMULTN` / `IMACN` / `RESMAC` sequence is preferable. Note also that the matrix address reaches only the **first 4 KB of DSP RAM** (see [Hardware Bugs → DSP matrix multiplies](../architecture/hardware-bugs.md#gpu--dsp-bugs)). *(Source: Appendix A — Frequently Asked Questions About Jaguar.)*

### D_END — DSP Data Organization Register (`$F1A10C`, WO)

Controls the physical layout of DSP I/O registers. If its current contents are unknown, the same data should be written to both the low and high 16 bits.

| Bit | Equates    | Description |
|-----|------------|-------------|
| 0   | `BIG_IO`   | When set, 32-bit registers in the CPU I/O space are big-endian (the more significant 16 bits appear at the lower address). |
| 2   | `BIG_INST` | When set, the DSP does word program fetches like a big-endian processor. |

### D_PC — DSP Program Counter (`$F1A110`, RW)

May be written whenever the DSP is idle (`DSPGO` clear) — normally used by the CPU to set where execution starts when `DSPGO` is set. May be read at any time, giving the address of the instruction currently executing. If the DSP reads it, this must be via the `MOVE PC,Rn` instruction, not a load. **Must always be written before setting `DSPGO`.** When `DSPGO` is cleared, the PC value is corrupted because the pre-fetch queue is discarded.

### D_CTRL — DSP Control/Status Register (`$F1A114`, RW)

Governs the interface between the CPU and DSP.

| Bit   | Equates | Description |
|-------|---------|-------------|
| 0  | `DSPGO`       | Stops/starts the DSP. CPU or DSP may write at any time. Reset state may be externally configured. The DSP **must not** be stopped by an external processor writing directly to `D_CTRL` — only the DSP should turn off the DSP. (To shut down another processor, signal it via a semaphore + interrupt and let its handler stop itself.) |
| 1  | `CPUINT`      | Writing `1` causes the DSP to interrupt the CPU. No acknowledge or clear needed. Writing `0` has no effect; reads always return `0`. |
| 2  | `FORCEINT0`   | Writing `1` causes a DSP interrupt type 0. No acknowledge or clear needed. Writing `0` has no effect; reads always return `0`. |
| 3  | `SINGLE_STEP` | When set, DSP single-stepping is enabled — execution pauses after each instruction until a `SINGLE_GO` is issued. The read status of this flag, `SINGLE_STOP`, indicates whether the DSP has actually stopped and should be polled before issuing a further single-step command (a `1` means the DSP is awaiting `SINGLE_GO`). |
| 4  | `SINGLE_GO`   | Writing `1` advances DSP execution by one instruction when paused in single-step mode. Writing at any other time, or writing `0`, has no effect; reads always return `0`. |
| 5  | Unused        | Write zero. |
| 6–10 | `D_CPULAT`, `D_I2SLAT`, `D_TIM1LAT`, `D_TIM2LAT`, `D_EXT0LAT` | Interrupt latches for interrupts 0–4 (CPU, I²S, Timer 1, Timer 2, EINT[0]). Indicate which request latch is active; clear via the `INT_CLR` bits in `D_FLAGS`. Writing to these bits has no effect. |
| 11 | `BUS_HOG`     | **Must not be set** in the Jaguar console — always write as `0`. |
| 12–15 | `VERSION`  | DSP version code. Current value: `2` (first production release). Future versions are intended to be a superset of this DSP. |
| 16 | `D_EXT1LAT`   | Interrupt latch for interrupt 5 (EINT[1]). Same function as bits 6–10 for interrupts 0–4. |

### D_MOD — DSP Modulo Instruction Mask (`$F1A118`, WO)

32-bit register governing which bits are modified by the `ADDQMOD` and `SUBQMOD` instructions. A `1` means the bit is unaffected; a `0` means it may be changed. Normally the higher bits are set to `1` and the lower bits to `0`, allowing addresses to be generated for circular buffers of size 2ⁿ bytes, where n is between 0 and 31.

### D_REMAIN — DSP Divide Unit Remainder (`$F1A11C`, RO)

32-bit register containing a value from which the remainder after a division may be calculated. (Refer to the divide-unit section.)

### D_DIVCTRL — DSP Divide Unit Control (`$F1A11C`, WO)

| Bit | Equates      | Description |
|-----|--------------|-------------|
| 0   | `DIV_OFFSET` | If set, the divide unit performs divisions of unsigned 16.16 numbers; otherwise 32-bit unsigned integer division is performed. |

### D_MACHI — DSP Multiply & Accumulate High Bits (`$F1A120`, RO)

32-bit register that allows the high bits of the accumulated result to be read. After a `RESMAC`, the `RESMAC` results register contains the bottom 32 bits of the accumulated value, and this register contains the top eight bits, sign-extended to 32 bits.

## See also

- [System Architecture Overview](../architecture/overview.md)
- [Memory Map / Register List](../architecture/memory-map.md)
- [Graphics Processor (GPU)](../tom/gpu.md)
- [RISC Instruction Set (GPU/DSP)](../reference/risc-instruction-set.md)
- [Audio Subsystem & Synthesis](audio.md)
- [Serial I/O](serial-io.md)

<!-- nav:bottom -->
---

◀ **Prev:** [CRY Color & Color Mapping](../tom/color-cry.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [Audio Subsystem & Synthesis](audio.md) ▶

**Jump to:** [Architecture](../architecture/overview.md) · [Memory Map](../architecture/memory-map.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md) · [CD-ROM](../cdrom/overview.md)
<!-- /nav:bottom -->
