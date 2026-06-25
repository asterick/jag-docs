<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ Tom — Graphics & Video ▸ **Graphics Processor (GPU)**
<!-- /nav:top -->

# Graphics Processor (GPU)

The Jaguar's fast pipelined RISC co-processor inside Tom, used for 3D modeling, shading, animation, and image decompression, with 4 KB of local RAM at `$F03000`.

> **Source:** *Software Reference Manual — Tom & Jerry* (V10), pp. 34–55; *Appendix* (Atari original, 26 April 1995), Appendix B. © Atari Corp. 1995.

## Graphics Processor Subsystem

The Graphics Subsystem of the Jaguar is a self-contained processing unit. Its view of the external system processor and memory is controlled by a separate memory controller, which is not part of the graphics system.

The subsystem transfers data to or from external memory by becoming the master of the **co-processor bus**. This bus has a 64-bit (phrase) data path and a 24-bit address with byte resolution. It has multiple masters; ownership is gained by a prioritized bus request / acknowledge system — ownership can be lost during a request, but not during a memory cycle. The graphics subsystem contains two bus masters: the **Graphics Processor** and the **Blitter**.

The graphics subsystem also acts as a slave on the **I/O bus**. This bus normally has a 16-bit data path and allows external processors to access memory and registers within the graphics subsystem. As the data path within the graphics subsystem is 32-bit, all reads and writes must be performed in pairs.

The memory within the Graphics Subsystem appears to be part of the general machine address space — both to the GPU and Blitter, and to external processors. The advantage to the GPU of having local memory is that it is both faster and does not require ownership of the system bus to be accessed.

> **Note:** Local RAM is described both as "1K Bytes (1000 hex addresses of 32-bit data)" in the subsystem diagram note and as **4 Kilobytes** (1K locations × 32 bits) in the Memory Interface section. The correct figure is **4 KB = 1024 × 32-bit locations** at `$F03000`.

## Memory Map

The Graphics Subsystem address space contains the following locations.

### GPU registers

| Address | Equate | Access | Description |
|---------|--------|--------|-------------|
| `$F02100` | `G_FLAGS` | RW | GPU flags |
| `$F02104` | `G_MTXC` | W | GPU matrix control |
| `$F02108` | `G_MTXA` | W | GPU matrix address |
| `$F0210C` | `G_END` | W | GPU big / little endian control |
| `$F02110` | `G_PC` | RW | GPU program counter |
| `$F02114` | `G_CTRL` | RW | GPU operation control / status |
| `$F02118` | `G_HIDATA` | RW | GPU bus interface high data |
| `$F0211C` | `G_DIVCTRL` | W | GPU division method |
| `$F0211C` | `G_REMAIN` | R | GPU division remainder |
| `$F03000` | `G_RAM` | RW | Local RAM base |

> `G_DIVCTRL` and `G_REMAIN` share address `$F0211C` (write vs. read).

### Blitter registers (for reference)

| Address | Equate | Access | Description |
|---------|--------|--------|-------------|
| `$F02200` | [`A1_BASE`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter A1 base |
| `$F02204` | [`A1_FLAGS`](blitter.md#a1_flags-f02204-wo) | W | Blitter A1 flags |
| `$F02208` | [`A1_CLIP`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter A1 clipping size |
| `$F0220C` | [`A1_PIXEL`](../architecture/memory-map.md#blitter-registers-tom) | RW | Blitter A1 pixel pointer |
| `$F02210` | [`A1_STEP`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter A1 step |
| `$F02214` | [`A1_FSTEP`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter A1 step fraction |
| `$F02218` | [`A1_FPIXEL`](../architecture/memory-map.md#blitter-registers-tom) | RW | Blitter A1 pixel pointer fraction |
| `$F0221C` | [`A1_INC`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter A1 pixel pointer increment |
| `$F02220` | [`A1_FINC`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter A1 pixel pointer increment fraction |
| `$F02224` | [`A2_BASE`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter A2 base |
| `$F02228` | [`A2_FLAGS`](blitter.md#a2_flags-f02228-wo) | W | Blitter A2 flags |
| `$F0222C` | [`A2_MASK`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter A2 mask |
| `$F02230` | [`A2_PIXEL`](../architecture/memory-map.md#blitter-registers-tom) | RW | Blitter A2 pixel pointer |
| `$F02234` | [`A2_STEP`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter A2 step |
| `$F02238` | [`B_CMD`](blitter.md#b_cmd--command-register-f02238-wo) | W | Blitter command |
| `$F0223C` | [`B_COUNT`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter loop counters |
| `$F02240` | [`B_SRCD`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter source data |
| `$F02248` | [`B_DSTD`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter destination data |
| `$F02250` | [`B_DSTZ`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter destination Z data |
| `$F02258` | [`B_SRCZ1`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter source Z data 1 |
| `$F02260` | [`B_SRCZ2`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter source Z data 2 |
| `$F02268` | [`B_PATD`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter pattern data |
| `$F02270` | [`B_IINC`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter intensity increment |
| `$F02274` | [`B_ZINC`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter Z increment |
| `$F02278` | [`B_STOP`](blitter.md#b_stop--collision-control-f02278-wo) | W | Blitter collision stop control |
| `$F0227C` | [`B_I3`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter intensity register 3 |
| `$F02280` | [`B_I2`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter intensity register 2 |
| `$F02284` | [`B_I1`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter intensity register 1 |
| `$F02288` | [`B_I0`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter intensity register 0 |
| `$F0228C` | [`B_Z3`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter Z register 3 |
| `$F02290` | [`B_Z2`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter Z register 2 |
| `$F02294` | [`B_Z1`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter Z register 1 |
| `$F02298` | [`B_Z0`](../architecture/memory-map.md#blitter-registers-tom) | W | Blitter Z register 0 |

These locations may be accessed by all processors **except the GPU** for read or write as appropriate at the above addresses, where they appear to the system as 16-bit memory. As they are all actually 32-bit, transfers should always be performed in pairs, in the order **low address then high address**.

For high-speed write operations by 32-bit or 64-bit bus masters (especially for blit transfers), they may be written as 32-bit locations at an offset of **plus `$8000`** from the addresses above. They are **not readable** at these offset addresses.

The GPU addresses them all directly as 32-bit locations in 32-bit internal memory, and they are **not** accessible to the GPU at the `+$8000` offset.

## What is the Graphics Processor?

The Graphics Processor (the GPU — Graphics Processor Unit) is a simple, very fast microprocessor. It is intended for performing the functions associated with generating graphics — three-dimensional modeling, shading, fast animation, and unpacking compressed images.

It corresponds to the accepted notion of a **RISC** (Reduced Instruction Set Computer) processor:

- Most instructions execute in one tick.
- All computational instructions involve registers.
- Memory transfers are performed by load/store instructions.
- Instructions are of a simple fixed format, with few addressing modes.
- There is a wealth of registers and local high-speed memory.

Features that give high computational power:

- Highly pipe-lined architecture
- One instruction per tick peak throughput
- Internal program and data RAM
- Register score-boarding
- Sixty-four 32-bit registers
- ALU includes barrel shifter and parallel multiplier
- Systolic matrix multiplication
- Fast hardware divide unit
- High-speed interrupt response, including video object interrupts
- Close coupling with the Blitter

The GPU is a full 32-bit processor: all internal data paths are 32-bit wide, and all arithmetic instructions (except multiply) perform 32-bit computations. Instructions are 16-bits wide. It has **64 internal 32-bit general purpose registers**, of which **32 are visible at one time**, plus **1K (addresses) of local high-speed 32-bit RAM** (= 4 KB), where its instructions and working data are normally stored. It can also access external memory via the 64-bit co-processor bus (byte, word, long-word and phrase transfers) and can execute its instructions from external RAM.

Addressing modes for load/store: register indirect, register indirect plus register offset, or register indirect plus immediate offset. It has jump-relative and jump-absolute instructions, both of which may depend on combinations of the zero, carry and negative flags.

## Programming the Graphics Processor

### Design Philosophy

The GPU is a RISC processor, normally executing one instruction per tick. The RISC approach was chosen principally because it occupies less silicon. The design has no micro-code — effectively the instruction set *is* the micro-code. Instructions execute quicker, at the cost of some operations requiring more instructions.

The GPU is intended to perform rapid floating-point arithmetic. It has no floating-point instructions as such, but has specific simple instructions that allow a limited-precision floating-point library to exceed **1 MegaFlop**. It is intended to be programmed in **assembly language**, not a compiled language.

### Pipe-Lining

The GPU makes extensive use of pipe-lining. Although it can achieve a peak rate of one instruction per tick, each instruction is actually executed over several ticks, spending one tick at each pipe-line stage.

For a typical instruction such as `ADD`, the pipe-line stages are:

1. Decode instruction
2. Read operands from registers
3. Add operands
4. Write result back to register

In addition, a pre-fetch unit attempts to maintain a small queue of unexecuted instructions to keep the execution unit busy.

### Register Score-Boarding

The main side effect of pipe-lining is the interaction of instructions at different stages of the pipe-line, which can conflict over the same operand or piece of hardware.

For example, if a second `ADD` adds to the same register as a preceding `ADD`, then without protection the second would use the old value (from before the first `ADD`). The GPU hardware detects this and **suspends execution until the correct value is ready**. Clock cycles lost during these hold-ups are called **wait states**.

Two problems arise from the architecture:

1. The register RAM has only **two data ports**, so if the instruction at stage three must write back to a register different from the two being read by the instruction at stage one, a clash occurs.
2. The instruction at stage one may need a value being computed by the instruction at stage two, but that value is not available until stage two reaches stage three.

The GPU uses a **score-board** to help avoid these problems. It tags registers that will change once an operation completes, and forces program flow to wait if an instruction reads a tagged register. The mechanism also applies to the flags. It will wait if:

- An instruction would read a register still being computed by the ALU.
- An instruction would perform a conditional jump, or add/subtract with carry, before the flags have been set as the result of an arithmetic operation.
- An instruction would read a register being read from internal memory.
- An instruction would read a register that is the target of a divide operation (the divide unit is relatively slow, so this can cause a significant delay).
- An instruction would read from a register waiting to be loaded from slow external memory (variable time).

> **Scoreboard limitation — indexed stores.** The scoreboard mechanism does **not** work on the data of any indexed store instruction. Any indexed store that stores data from a long-latency operation (such as a divide or external load) should place an `or` instruction prior to the store. For example:
>
> ```asm
> div    r0,r3
> store  r3,(r14+6)
> ```
>
> should be written as:
>
> ```asm
> div    r0,r3
> or     r0,r3
> store  r3,(r14+6)
> ```

### Register Write-Back

The score-board unit also controls writing back of computed values. The registers are dual-port RAM, so it is not possible to read two register values simultaneously while writing to a third.

If the register to be written back is being read by the instruction currently at stage 1, or if one of that instruction's operands does not involve a register read, the write-back is **concealed**. Otherwise the instruction is held up one cycle while the value is written back.

A wait state is also generated if the instruction that would have executed reads two registers, neither of which is the target of the write. Write-back data sources are:

- The result of an ALU computation
- The result of a divide operation (occurs in parallel with the ALU)
- The data from an internal load operation
- The data from an external load operation

If two of these are to be written back simultaneously, execution is always held up for a tick.

**Optimization tip:** interleave two sets of calculations so that consecutive instructions do not use the same registers, but instructions two apart generally do.

#### Destination write-only corruption bug

In any instruction where the destination register is written without being read, the destination register is **not protected** by the score-boarding mechanism of the GPU/DSP. This includes `MTOI`, `MORMI`, `RESMAC`, all `MOVE` variations, and all `LOAD` variations.

If one of these "destination write-only" instructions writes to the same destination register as a prior instruction, with no intervening reads from that register, the second instruction may complete **before (or simultaneously with)** the first, corrupting the register. This is only a problem with "dummy" instruction sequences:

```asm
   div     r2,r4   ; Divide starts (takes 18 ticks)
   moveq   #4,r4   ; Move completes before divide
```

This might appear at the end of a loop:

```asm
Loop:
     jr      EQ, loop
     div     r2,r4
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
; Any number of instructions could appear here.  ;
; Unless one of them reads R4, the result of the ;
; MOVEQ will be unreliable                       ;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
     moveq   #4,r4
```

When the loop condition fails, the `DIV`/`MOVEQ` sequence runs and R4 is corrupted. Prevent it by reading the destination register before the move:

```asm
Loop:
     jr      EQ, loop
     div     r2,r4
     or      r4,r4
     moveq   #4,r4
```

The same applies to any instruction that writes a register followed (with no intervening reads) by a "destination write-only" instruction to the same register:

```asm
Loop:
     jr      EQ, loop
     add     r10,r12
     moveq   #1,r12 ; ADD will trash this
```

In normal code, where the result of a register write is used, the bug does not occur:

```asm
Loop:
     load    (r2),r4
     add     r4,r6
     moveq   #4,r4 ; Safe because R4 was read above
```

### Jump Instructions

Pipe-lining also affects jumps. **Transfer of control does not occur until the instruction after the jump has been executed.** The safest technique is to follow all jumps with a `NOP`, though almost any other instruction may be placed there (see Program Control Flow and Illegal Instruction Combinations).

> **Restriction:** Neither the DSP nor GPU will reliably execute `jr` or `jump` instructions unless they are in **internal RAM**.

## Memory Interface

The GPU is intended to operate in parallel with the other processing elements. A well-behaved GPU program should make only occasional use of the main memory bus. The GPU therefore has **4 KB of local memory, organized as 1K locations of 32 bits**.

This memory is used for both program and data. It can be cycled at the GPU clock rate, so it is extremely fast. It may be viewed as a simple cache RAM with software cache control — a technique known as **visible caching**. When executing code out of internal RAM, program fetch cycles occupy less than half the RAM bandwidth.

To load a program into GPU RAM, the best technique is the **Blitter** — set it to blit phrases and use the 32-bit GPU address range.

To the GPU programmer, local RAM, local hardware registers, and external memory all appear in the same address space. The GPU memory controller determines whether a transfer is local or external and generates the appropriate cycle. The only programming difference: only **32-bit transfers** are possible within the GPU local address space, whereas **8, 16, 32 or 64-bit** transfers are permitted externally.

The local RAM sits on an internal GPU 32-bit bus, along with various GPU control registers and the Blitter control registers. When a transfer occurs outside the local address space, a gateway connects the local bus to the main bus. For a 64-bit transfer, a special register (`G_HIDATA`) holds the other half of the data.

The address space is organized as follows:

| Range | Contents |
|-------|----------|
| `$F02000` – `$F021FF` | Graphics processor control registers |
| `$F02200` – `$F022FF` | Blitter registers |
| `$F02300` – `$F02FFF` | Reserved |
| `$F03000` – `$F03FFF` | Local RAM (1K hexadecimal locations) |
| `$F04000` – `$F0FFFF` | Reserved |

This local address space is also available to external devices via the I/O mechanism.

The GPU local bus performs transfers for three separate mechanisms, in **decreasing order of priority**:

1. CPU I/O access
2. Operand data transfer
3. Instruction fetch

### External View of GPU Space

The GPU internal address space is accessible by any other Jaguar bus master (CPU, Blitter, DSP) — it is part of the Jaguar I/O space within Tom. Normally viewed as 16-bit read/write memory, it is also available as **32-bit write-only** memory by adding `$8000` to the addresses, which is faster for masters that can perform 32-bit transfers. This lets the Blitter copy data into GPU space more rapidly than via the 16-bit space — for maximum speed, use the Blitter in **phrase mode** writing to the 32-bit address range.

> The 68000 in the Jaguar Console may **not** address this 32-bit-wide memory.

Transfers to/from addresses within `$F02000`–`$F07FFF` and `$F1A000`–`$F1F000` are executed 32 bits at a time using a **latch mechanism** and must be handled carefully by external processors (see External CPU Access):

- When a 16-bit word is **read** from a long-word aligned address, a 32-bit read is performed: the high word is transferred and the low word is latched. A 16-bit read at *long-word aligned address + `$2`* simply transfers the latched data.
- When a 16-bit word is **written** to a long-word aligned address, the data is latched. When a 16-bit word is written to *long-word aligned address + `$2`*, 32 bits (the written word plus the latched data) are transferred.

### The GPU and Data Ordering Conventions

The GPU can operate in both big-endian and little-endian environments. As long as the memory interface is programmed to the correct endian mode and the transfer width matches the operand width, this is largely invisible to the programmer.

The GPU is itself **either-endian** — the first instruction of the pair in a long-word is programmable, controlled by the `BIG_INST` bit (in `G_END`).

## Load and Store Operations

The GPU has load and store instructions, each taking two register operands: one supplies the address, the other supplies data to be stored or is written with load data.

Loads and stores may be at **byte, word, long-word or phrase** width. Bytes and words are aligned with bit 0, and when loaded the rest of the register is set to zero. For phrase transfers, a register within the GPU local address space should already contain the other long-word (for stores) or is loaded with the other long-word (for loads). Phrase loads/stores are the fastest way of transferring blocks.

Two simple **indexed addressing** schemes are available, both using `R14` or `R15` as base register, with either a five-bit unsigned offset (in long-words) encoded into a register field, or another register containing the offset. There is a **two tick overhead** for these, as the address must be computed.

In local memory, **only long-word reads and writes are permitted.**

Load/store operations normally complete in one tick (two for indexed). The transfer may not be complete at that point; if another load/store occurs before the previous one has completed, it is held up. Load data is written under control of the score-board unit.

The gateway between the GPU local bus and the external co-processor bus contains a control block for external transfers. When idle, load/store complete as fast as in local memory. For loads, the data is not loaded into the target register until the external transfer takes place; the score-board prevents use before it is loaded, though other computation may proceed. Another load/store before the gateway completes is held up until the gateway is idle.

> **Console bugs / cautions:**
> - Due to a bug in the Jaguar Console, DMA transfers are not permitted. The **`DMAEN`** bit of the `G_FLAGS` register must be cleared to 0. *(Source spells this both `DMEAN` and `DMAEN`; the register bit is `DMAEN`, bit 15.)*
> - The value in the **High Data Register** (`G_HIDATA`) is changed after **ANY** external load, not just `loadp`. So if a GPU interrupt loads from external memory, the underlying program may not use `loadp`.

## Arithmetic Functions

The GPU contains a powerful ALU. As well as the normal arithmetic and Boolean functions (all 32-bit word size), it contains a **16-by-16 fast parallel multiplier** and a **32-bit barrel shifter**, each performing its function in one tick.

The GPU also contains a **divide unit**: serial division at two bits per tick on 32-bit unsigned operands, producing a 32-bit quotient, running in parallel with normal GPU operation.

The ALU has the following flags:

| Flag | Name | Description |
|------|------|-------------|
| Z | zero | Set appropriately by all arithmetic operations, normally set if the result was zero. |
| N | negative | Set appropriately by all arithmetic operations, normally set if the result was negative (bit 31 is a one). |
| C | carry | Set according to carry or borrow out of all add and subtract operations; set with the bit shifted out of shift and rotate operations for shift-by-one; left undefined by other arithmetic operations. |

## Interrupts

The GPU can be interrupted by **five sources**. Interrupts force a call to an address in local RAM given by **sixteen times the interrupt number (in bytes)** from the base RAM address. It is the programmer's responsibility to preserve the registers and flags of the underlying code.

- Primary register **R31** is the interrupt stack pointer.
- Primary register **R30** is corrupted when instruction flow is transferred to the interrupt service routine.
- Neither register should be used for any other purpose when interrupts are enabled.

> **Initialize the interrupt stack pointer.** After a console reset the GPU and DSP interrupt stack pointers (R31) are in an **undefined state** — always initialize R31 before enabling interrupts. *(Source: Appendix B — Programming Tips & General Procedures.)*

> **A RISC processor only services interrupts while it is running.** For GPU or DSP interrupts to be handled, that processor must be executing — a stopped processor handles nothing. If you have no other work for it but still need its interrupts, leave a **small loop in internal RAM that polls a semaphore** until told to shut down; because the semaphore lives in internal RAM this consumes no bus bandwidth. **Do not** use a one-instruction (single-line) tight loop for this. *(Source: Appendix B — Programming Tips & General Procedures.)*

Interrupt allocation:

| # | Interrupt |
|---|-----------|
| 4 | Blitter |
| 3 | Object Processor |
| 2 | Timing generator |
| 1 | Jerry Interrupt |
| 0 | CPU interrupt |

The flags register contains individual interrupt enables for each source plus a master interrupt mask. When the master mask is set, the **primary register bank** is selected.

When an interrupt occurs, the master mask bit is set. Individual enables are unaffected, but no further interrupts are serviced until the mask is cleared. The service routine should normally clear the master mask and the appropriate interrupt latch, and enable higher-priority interrupts immediately.

The value pushed onto the R31 stack is the **address of the last instruction executed before the interrupt**. The service routine should therefore **add two** to this value before using it to return.

The interrupt latches may be read in the status port and are cleared by writing a **one** to their clear bits; writing a zero leaves them unchanged.

The cause of an interrupt may be determined by the location jumped to, **not** from the Flags register, as more than one latch bit may be set.

There is some interrupt prioritization: if two interrupts arrive within a few ticks, the higher-numbered is serviced first. Beyond that, prioritization is under software control.

Only single instructions, or certain instruction combinations (see Atomic Operations), are atomic. Interrupts may be disabled by clearing all the enable bits. It is therefore not practical for the interrupt stack to be shared with the underlying code unless all interrupts are masked across stack operations.

### Example interrupt service routine

This routine does no more than clear the interrupt. The interrupt source was interrupt 2.

```asm
int_serv:
     movei  #G_FLAGS,r30   ; point R30 at flags register
     load   (r30),r29      ; get flags
     bclr   #3,r29         ; clear IMASK
     bset   #11,r29        ; and interrupt 2 latch
     load   (r31),r28      ; get last instruction address
     addq   #2,r28         ; point at next instruction to be executed
     addq   #4,r31         ; update the stack pointer
     jump   (r28)          ; and return
     store  r29,(r30)      ; restore flags
```

Notes on this code:

- Registers **R28 and R29** may not be used by the underlying code as they are corrupted here (you may choose any two registers in bank #0), in addition to **R30 and R31** which are always corrupted by the interrupt process itself. R30 is automatically corrupted when an interrupt occurs, not just by this service code.
- Interrupts are re-enabled on the instruction **after** the jump. If enabled sooner, another service routine could corrupt R28/R29 before this routine completed.

If the source was the **Object Processor**, the service routine should read the Object Code registers (if required) and then re-start the Object Processor by writing to the Object Processor Flag register, as quickly as possible.

## Atomic Operations

Certain operations must be atomic — interrupts may not occur during them. Three GPU instruction types temporarily lock out interrupts while they complete:

- **Immediate data moves (`MOVEI`).** Interrupts are locked out while the two words of immediate data are fetched.
- **Matrix multiply (`MMULT`).** Interrupts are locked out until the operation completes.
- **Multiply and accumulate (`IMULTN` and `IMACN`).** The result register is not preserved by interrupts; therefore any multiply/accumulate operation must consist of a sequence of `IMULTN` and `IMACN` instructions followed by `RESMAC`, with no intervening instructions. `IMULTN` and `IMACN` are always atomic with the succeeding instruction.

Additionally, **jump instructions are always atomic with the instruction that succeeds them.**

## Program Control Flow

Program control normally runs upwards through memory, executing instructions sequentially. The GPU can transfer flow with jump instructions.

Two types of jump are supported:

- **Jump relative** — takes a signed five-bit offset, treated as an offset in **words**, added to the program counter.
- **Jump absolute** — transfers the contents of a register into the program counter.

Both types may be conditional on the ALU flags. If the condition is not met, the jump is ignored and flow continues with the next instruction.

The instruction after a jump is **always executed** (a side-effect of the pre-fetch queue). Place a `NOP` after every jump, or take advantage of this to place a useful instruction that runs whichever branch is followed.

The program counter may also be copied into a register.

The GPU can cease operation by clearing the `GPUGO` bit in the GPU control register. It may then only be restarted by an external write to that register, or by a reset.

## Single Step Operation

As a debugging aid, the GPU can single-step, pausing between instructions until restarted. Controlled by an external CPU:

1. Set up the program counter, then set the `GPUGO` and `SINGLE_STEP` control bits in the control register.
2. Poll for the `SINGLE_STOP` flag in the status register — at this point the first instruction has been executed.
3. Set the `SINGLE_GO` bit (keeping `GPUGO` and `SINGLE_STEP` set).
4. Poll for the `SINGLE_STOP` flag being set (the read version of `SINGLE_STEP`), indicating the next instruction has executed.
5. Repeat from step 3.

If the GPU register file must be read or written, single-stepping must be suspended and a transfer routine run, which requires clearing `GPUGO` first and modifying the program counter. **Clearing `GPUGO` alters the program counter value, because the pre-fetch queue is discarded.** Therefore, after step 4:

- Read the program counter value.
- Clear the `GPUGO` control bit.
- Read or write the register file as required.
- Add two to the program counter value read.
- Restart from step 1.

Adding two is necessary because the value read reflects the last instruction executed (or last word of immediate data if it was `MOVEI`).

## Illegal Instruction Combinations

- Do **not** place a `MOVEI` instruction after a jump — the jump takes effect before the data is fetched, changing where the immediate data is fetched from.
- Do **not** place two jump instructions sequentially — results are unpredictable.
- Do **not** place a "MOVE PC to register" instruction immediately after a jump — the value read cannot be relied upon.
- Do **not** follow an `IMULTN` instruction by anything other than an `IMACN` instruction.
- Do **not** follow an `IMACN` instruction by anything other than another `IMACN` or a `RESMAC` instruction.
- Do **not** precede an `MMULT` instruction by a `LOAD` or `STORE` instruction.

## Conditional Jumps

Conditional jumps encode from a five-bit flag field:

| Bit | Condition |
|-----|-----------|
| 0 | Zero flag must be **clear** for jump to occur. |
| 1 | Zero flag must be **set** for jump to occur. |
| 2 | Flag selected by bit 4 must be **clear** for jump to occur. |
| 3 | Flag selected by bit 4 must be **set** for jump to occur. |
| 4 | If set, select negative flag; if clear, select carry. |

This gives the useful jumps below (other codes are jump always or jump never, reserved for future modifications):

| Code | # | Condition | Description |
|------|---|-----------|-------------|
| `00000` | 0 | | Jump always |
| `00001` | 1 | NZ | Jump if zero flag is clear |
| `00010` | 2 | Z | Jump if zero flag is set |
| `00100` | 4 | NC | Jump if carry flag is clear |
| `00101` | 5 | NC NZ | Jump if carry flag is clear and zero flag is clear |
| `00110` | 6 | NC Z | Jump if carry flag is clear and zero flag is set |
| `01000` | 8 | C | Jump if carry flag is set |
| `01001` | 9 | C NZ | Jump if carry flag is set and zero flag is clear |
| `01010` | A | C Z | Jump if carry flag is set and zero flag is set |
| `10100` | 14 | NN | Jump if negative flag is clear |
| `10101` | 15 | NN NZ | Jump if negative flag is clear and zero flag is clear |
| `10110` | 16 | NN Z | Jump if negative flag is clear and zero flag is set |
| `11000` | 18 | N | Jump if negative flag is set |
| `11001` | 19 | N NZ | Jump if negative flag is set and zero flag is clear |
| `11010` | 1A | N Z | Jump if negative flag is set and zero flag is set |
| `11111` | 1F | | Jump never |

## Multiply and Accumulate Instructions

The GPU supports multiply-and-accumulate (MAC) operations: multiplying two values together and adding the product to the sum of previous products. These are typically used for matrix multiply and digital filtering.

Due to pipe-lining, the multiply and its associated add do **not** take place in the same cycle. A special instruction (`RESMAC`) is needed to write back the result.

Example — multiply R8×R9, R10×R11, R12×R13, and place the sum of products in R2 (all values signed):

```asm
imultn r8,r9    ; compute the first product, into the result
imacn  r10,r11  ; second product, added to first
imacn  r12,r13  ; third product, accumulated in result
resmac r2       ; sum of products is written to r2
```

MAC instructions may only be followed by further MAC instructions or by `RESMAC`. No other combinations are permitted.

## Systolic Matrix Multiplies

The GPU contains a mechanism for performing integer matrix multiplies at a burst rate of one multiply per tick (the maximum from the hardware multiplier). It was designed in particular for the matrix multiplies required by the **Discrete Cosine Transform** algorithm — one technique performs two 8×8 integer matrix multiplies in succession on a matrix, using the same fixed coefficients but rotated for the second multiply.

The **`MMULT`** instruction initiates a sequence of between three and fifteen multiply/accumulate instructions, corresponding to one product term of the result matrix. One source matrix is held in the **secondary register bank**, the other in **local RAM**. The matrix held in registers is **packed** — two elements per register — which allows an entire 8×8 matrix to be stored in the secondary register bank (the raison d'être of the second bank).

`MMULT` takes as its **source** parameter the register (always in the secondary register bank) containing the first two elements of the matrix row. Its **destination** parameter is the register (in the currently selected bank) to write the result.

The matrix held in RAM may be accessed in either increasing row or increasing column order — the data for each successive multiply is either one location apart or the matrix width apart.

Like interrupts, the systolic operation forces internally generated instructions into the instruction stream: the first is `IMULTN`, the middle ones `IMACN`, and the last `RESMAC`, with operands modified as described above.

> `MMULT` should **not** be preceded by a `LOAD` or `STORE` instruction.

## Divide Unit

The divide unit performs **unsigned division**, taking 32-bit divisor and dividend, giving a 32-bit quotient and a 32-bit remainder. The quotient is the result of the divide instruction and replaces the dividend in the destination register. Divides run at **two bits per tick**, completing in **sixteen ticks**. The divide instruction has **no effect on the flags**.

If another instruction attempts to read the quotient or start another divide while the divide unit is active, wait states are inserted until it completes.

The remainder register (`G_REMAIN`) may be read after the divide completes. The value may be **positive** (the actual remainder) or **negative** (the remainder minus the divisor).

Divides may also be performed on **unsigned 16.16 bit** values by setting the offset control flag (`DIV_OFFSET`) in the divide control register (`G_DIVCTRL`); the quotient is then also an unsigned 16.16 value.

> **Divider bug (GPU and DSP):** Two consecutive divides with fewer than one clock cycle of idle time between them produce a wrong result for the second divide. This occurs only when the two divides are separated by **fewer than 16 clock cycles** and the second divide has the quotient of the first as one of its register operands.
>
> Work-around: either ensure more than 16 clock cycles between divide instructions, or ensure that an instruction dependent on the quotient of the first divide occurs before the second divide.
>
> ```asm
> ; Problematic:
>      div     r0,r1
>      moveq   #3,r5
>      div     r5,r1
> ; Fixed:
>      div     r0,r1
>      moveq   #3,r5
>      or      r1,r1
>      div     r5,r1
> ```
>
> *(The source lists "Example #1" and "Example #2" as identical; immediate operands shown as `#3.r5` in the source are OCR artifacts for `#3,r5`.)*

## Register File

The GPU contains a register file of **sixty-four 32-bit registers**. All may be used as general purpose registers, though some have special functions.

All instructions contain **two five-bit register operand fields** (not always used as such). There are **two banks**: primary (bank 0) and secondary (bank 1).

- The **primary bank (bank 0)** is always used for interrupt service. This is forced by the `IMASK` bit — when set, bank 0 selection is forced.
- If `IMASK` is clear, **`REGPAGE`** is obeyed.

Bank select bits are in the Flags register, and special `MOVE` instructions allow data to be moved between banks.

## External CPU Access

The GPU internal address space is accessible to an external bus master at any time — external access has the **highest priority** on the GPU local bus. This means the Blitter may be used to load data into the local RAM.

The local address space is accessible for reading or writing at the addresses given above, presented as **16-bit memory**, which must always be accessed as long words in the order **low address then high address**.

For faster transfers into GPU space, all registers are also available as **32-bit memory** at an offset of `$8000` from their normal addresses; at this offset the internal memory is **write only**. The 68000 may not access this memory, as it transfers 16 bits at a time.

If the Blitter writes into GPU space, **phrase-wide transfers** may be performed; the bus control mechanism automatically divides these to suit the width of the memory being addressed.

> **68000 instruction restriction:** The `clr.l <ea>` and `move.l <ea>,-(an)` instructions of the 68000 do **not** work correctly when writing to Jaguar GPU & DSP hardware registers and internal RAM. The affected address ranges are `$F02000`–`$F07FFF` and `$F1A000`–`$F1F000`; these instructions are safe outside those ranges.
>
> Because the 68000 has a 16-bit data bus, 32-bit writes occur as two successive 16-bit writes. With these instructions the high/low word order is reversed, which breaks writes to those ranges. Best practice: use the GPU and/or DSP (not the 68000) to write to GPU/DSP registers, and use the Blitter to copy into GPU/DSP RAM. With a high-level compiler, ensure it does not generate `clr.l` for this address space.

## Pack and Unpack

The pack and unpack instructions provide a means for **averaging up to 32 CRY pixels**. The **unpack** operation leaves the intensity value unchanged, shifts the lower color nibble up 5 bits, and the higher color nibble up 10 bits. The **pack** operation reverses this.

There are five unused bits above each field in an unpacked pixel, allowing up to 32 unpacked pixels to be added together. If a power-of-two number of unpacked pixel values are added, a shift can re-align them prior to packing the average value. The bits that do not contain packed or unpacked pixel data are always set to zero. This is useful for anti-aliasing and scaling effects.

## Internal Registers

This section describes the internal registers of the Graphics Processor. Some are read- or write-only. **All GPU registers are 32-bit and require all 32 bits to be written.**

### G_FLAGS — GPU Flags Register (`$F02100`, RW)

Provides status and control bits for several important GPU functions.

| Bits | Equate(s) | Description |
|------|-----------|-------------|
| 0 | `ZERO_FLAG` | ALU zero flag — set if the result of the last arithmetic operation was zero. Certain arithmetic instructions do not affect the flags. |
| 1 | `CARRY_FLAG` | ALU carry flag — set or cleared by a carry/borrow out of the adder/subtractor; reflects carry out of some shift operations; undefined after other arithmetic operations. |
| 2 | `NEGA_FLAG` | ALU negative flag — set if the result of the last arithmetic operation was negative. |
| 3 | `IMASK` | Interrupt mask — set by the interrupt control logic at the start of the service routine; cleared by the service routine writing a 0. Writing a 1 has no effect. |
| 4–8 | `G_CPUENA`, `G_JERENA`, `G_PITENA`, `G_OPENA`, `G_BLITENA` | Interrupt enable bits for interrupts 0–4. Overridden by `IMASK`. Meaning: 0 = CPU Interrupt; 1 = Jerry Interrupt; 2 = Timing Generator; 3 = Object Processor; 4 = Blitter. |
| 9–13 | `G_CPUCLR`, `G_JERCLR`, `G_PITCLR`, `G_OPCLR`, `G_BLITCLR` | Interrupt latch clear bits. Clear the interrupt latches (readable in the status register). Writing a zero leaves a bit unchanged; the read value is always zero. |
| 14 | `REGPAGE` | Switches from register bank 0 to register bank 1. Overridden by `IMASK`, which forces bank 0. |
| 15 | `DMAEN` | Must **not** be set (bug in the Jaguar Console). Write as zero only. |

> **Pipe-lining caution:** Values written to `G_FLAGS` may not appear to have changed in the following two instructions due to pipe-lining. Writing a value to the flag bits and using those flag bits in the next instruction will not work. If you must use flags set by a `STORE`, ensure at least **two** other instructions lie between the `STORE` and the flags-dependent instruction. For an **indexed `STORE`**, ensure at least **four** other instructions lie between.

### G_MTXC — Matrix Control Register (`$F02104`, WO)

Controls the function of the `MMULT` instruction.

| Bits | Equate(s) | Description |
|------|-----------|-------------|
| 0–3 | `MATRIX3-15` | Matrix width, in the range 3 to 15. |
| 4 | `MATCOL` | When set, the matrix held in memory is accessed down one column, as opposed to along one row. |

### G_MTXA — Matrix Address Register (`$F02108`, WO)

Determines where, in local RAM, the matrix held in memory is.

| Bits | Equate(s) | Description |
|------|-----------|-------------|
| 2–11 | --- | Matrix address. |

### G_END — Data Organization Register (`$F0210C`, WO)

Controls the physical layout of pixel data and GPU I/O registers. If its current contents are unknown, the same data should be written to both the low and high 16-bits.

| Bits | Equate(s) | Description |
|------|-----------|-------------|
| 0 | `BIG_IO` | When set, 32-bit registers in the CPU I/O space are big-endian (the more significant 16-bits appear at the lower address). |
| 1 | `BIG_PIX` | When set, the pixel organization is big-endian. |
| 2 | `BIG_INST` | When set, the order of word program fetches is big-endian. |

### G_PC — GPU Program Counter (`$F02110`, RW)

May be written whenever the GPU is idle (`GPUGO` clear). Normally used by the CPU to govern where execution starts when `GPUGO` is set.

May be read at any time, giving the address of the instruction currently being executed. If the GPU reads it, this must be done by the `MOVE PC, Rn` instruction, **not** by a load from it.

`G_PC` must always be written before setting `GPUGO`. When `GPUGO` is cleared, the program counter value is corrupted, as the pre-fetch queue is discarded at that point.

### G_CTRL — GPU Control/Status Register (`$F02114`, RW)

Governs the interface between the CPU and the GPU.

| Bits | Equate(s) | Description |
|------|-----------|-------------|
| 0 | `GPUGO` | Stops and starts the GPU. CPU or GPU may write at any time. Status after reset is externally configurable. The GPU must **not** be stopped by an external processor writing directly to `G_CTRL` — only the GPU should turn off the GPU. (To shut down another processor, signal it via a semaphore + interrupt and let it shut itself down.) |
| 1 | `CPUINT` | Writing a 1 causes the GPU to interrupt the CPU. No acknowledge needed, no need to clear. Writing a zero has no effect. Reads as zero. |
| 2 | `FORCEINT0` | Writing a 1 causes a GPU interrupt type 0. No acknowledge needed, no need to clear. Writing a zero has no effect. Reads as zero. |
| 3 | `SINGLE_STEP` | When set, GPU single-stepping is enabled (execution pauses after each instruction until a `SINGLE_GO` command). The read status of this flag, **`SINGLE_STOP`**, indicates whether the GPU has actually stopped and should be polled before issuing a further single-step command. A one means the GPU is awaiting a `SINGLE_GO`. |
| 4 | `SINGLE_GO` | Writing a one advances execution by one instruction when paused in single-step mode. Writing at any other time, or writing zero, has no effect. Reads as zero. |
| 5 | unused | Write zero. |
| 6–10 | `G_CPULAT`, `G_JERLAT`, `G_PITLAT`, `G_OPLAT`, `G_BLITLAT` | Interrupt latches. Indicate which interrupt request latch is active; the appropriate bit should be cleared by the service routine using the `INT_CLR` bits in the flags register. Writing to these bits has no effect. Meaning: 0 = CPU Interrupt; 1 = Jerry Interrupt; 2 = Timing Generator; 3 = Object Processor; 4 = Blitter. |
| 11 | `BUS_HOG` | Should not be set in the Jaguar Console; always write zero. |
| 12–15 | `VERSION` | GPU version code (read). Current codes: 1 = Pre-production test silicon; 2 = First production release. Future versions intended to be a superset. |

### G_HIDATA — High Data Register (`$F02118`, RW)

This 32-bit register provides the high part of GPU phrase reads and writes. It is physically a single register, so a phrase read followed by a phrase write will write back the same high data unless this register is modified.

### G_REMAIN — Divide Unit Remainder (`$F0211C`, RO)

This 32-bit register contains a value from which the remainder after a division may be calculated. Refer to the Divide Unit section.

### G_DIVCTRL — Divide Unit Control (`$F0211C`, WO)

| Bits | Equate(s) | Description |
|------|-----------|-------------|
| 0 | `DIV_OFFSET` | If set, the divide unit performs division of unsigned 16.16 bit numbers; otherwise 32-bit unsigned integer division. |

## Performance notes (community)

> **Community source:** developer findings by Atari Owl (Joe), *The Owl Project* —
> ["What's this 'Lay off the 68k' and 'GPU in Main' Malarkey?"](https://atariowlproject.blogspot.com/2009/10/atari-jaguar-homebrew-whats-this-lay.html)
> (21 Oct 2009). These go beyond the original Atari manuals; treat the exact rules
> as developer-reported and verify against the post.

### Freeing the bus — halt the 68000

The GPU, DSP, Blitter and 68000 share the bus, so a 68000 left spinning (for
example in a divide loop) steals memory cycles from the RISC engines. **Halting
the 68000 with `STOP #$2000`** removes that contention and gives roughly a
**5–10 % boost** to RISC-heavy workloads (e.g. 3D). Atari's Leonard Tramiel:
*"The interleaving of GPU and 68k code has never, in our experience, gained any
performance. The best thing that can be done with the 68k for overall system
performance is to execute a halt instruction."* This is an optimization, not a
requirement — 2D games, ST ports, and the like run fine with the 68000 active.

### Running GPU code from main RAM

The GPU normally executes from its 4 KB of local RAM, but it *can* run from main
DRAM when a program needs more code than local RAM holds. The catch is the
[`jr`/`jump` reliability bug](../architecture/hardware-bugs.md#3-jr--jump-only-reliable-from-internal-ram) —
jumps must be aligned and the pipeline cleared. Reported rules:

- A **`JUMP` (absolute) source** must be **long-aligned** (address ending 0/4/8/C);
  jumps that cross between local and main RAM may need **phrase alignment** (0/8).
- **Precede** a local↔main jump with a **`MOVEI`** to clear the pipeline (its
  register may need to differ from the `JUMP` register).
- A **`JR` (relative) source** has **no** alignment restriction.
- Jumps to **another page** must be long-aligned; jumps **within a page** must be
  word-offset from long alignment (ending 2/6/A/E).
- Follow a `JUMP`/`JR` with **two `NOP`s** (the first may sometimes be a useful
  single-operand instruction).

Throughput from main RAM is roughly **20–90 % of local-RAM speed** (worse under
heavy Blitter/bus load or when code straddles pages) — but even at the low end it
beats the 68000. Keep tight inner loops, especially `LOAD`/`STORE`, in local RAM.
Rough scale: Atari ST 68000 ≈ 1 MIPS; Jaguar 68000 (13.3 MHz) ≈ 1.65 MIPS;
GPU ≈ 17–27 MIPS.

## See also

- [System Architecture Overview](../architecture/overview.md)
- [Memory Map / Register List](../architecture/memory-map.md)
- [Object Processor](object-processor.md)
- [Blitter](blitter.md)
- [RISC Instruction Set (GPU/DSP)](../reference/risc-instruction-set.md)
- [Digital Sound Processor (DSP)](../jerry/dsp.md)

<!-- nav:bottom -->
---

◀ **Prev:** [Object Processor (Tom)](object-processor.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [Blitter (Tom)](blitter.md) ▶

**Jump to:** [Architecture](../architecture/overview.md) · [Memory Map](../architecture/memory-map.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md) · [CD-ROM](../cdrom/overview.md)
<!-- /nav:bottom -->
