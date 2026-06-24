<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ Architecture & Memory ▸ **Hardware Bugs & Warnings**
<!-- /nav:top -->

# Hardware Bugs & Warnings

Known bugs and cautions in the Jaguar silicon. **Do not rely on the side-effects
of these bugs** — they may be fixed in future hardware revisions. This page
consolidates the standalone *Hardware Bugs & Warnings* document; the same notes
also appear inline in the relevant subsystem pages.

> **Source:** *Hardware Bugs & Warnings* (Atari original, 26 April 1995);
> cross-checked against the *Technical Reference* and *Software Reference* (V10).
> © Atari Corp. 1994–1995.

## GPU / DSP bugs

See [Graphics Processor](../tom/gpu.md), [DSP](../jerry/dsp.md), and the
[RISC instruction set](../reference/risc-instruction-set.md).

### 1. Scoreboard ignores indexed-store data
The scoreboard does not protect the data of an **indexed store**. If you store
data produced by a long-latency operation (a divide or external load), insert a
dummy read (`or`) of the register before the store:

```asm
        div     r0,r3
        or      r3,r3       ; force the divide to complete
        store   r3,(r14+6)
```

### 2. "Destination write-only" instructions aren't scoreboarded
`MTOI`, `NORMI`, `RESMAC`, all `MOVE` variants, and all `LOAD` variants write
their destination without reading it, so the scoreboard doesn't protect it. If
such an instruction targets the same register as a prior (still in-flight)
instruction with no intervening read, the register can corrupt:

```asm
        div     r2,r4       ; divide starts (18 ticks)
        moveq   #4,r4       ; completes before the divide → r4 corrupts
```

Fix by reading the register first (`or r4,r4`) between the two. In normal code
where the written value is actually used, the bug does not occur.

### 3. `jr` / `jump` only reliable from internal RAM
Neither the GPU nor the DSP will reliably execute `jr` or `jump` unless running
from its own internal RAM.

### 4. DSP must not run in high bus priority
Keep the **`DMAEN` bit of `D_FLAGS` = 0**. With it set, an external load/store
hangs the DSP and needs a reset to recover.

### 5. GPU/Blitter must not outrank the Object Processor
While the Object Processor is running, keep the **`DMAEN` bit of `G_FLAGS` = 0**
and the **`BUSHI` bit of `B_CMD` = 0**. No bus master may run at higher priority
than the OP — if something steals the bus between the 2nd and 3rd phrases of an
object header, the line-buffer address corrupts, giving horizontal black stripes
and other artifacts. See [Object Processor](../tom/object-processor.md).

### 6. Don't stop a RISC processor by external write to its control register
Never stop the GPU/DSP by writing `G_CTRL`/`D_CTRL` from another processor — only
the GPU should stop the GPU, only the DSP should stop the DSP. To shut one down,
set a semaphore and interrupt it so it stops itself.

### 7. DSP external write must follow a completing external read
A DSP external write must be preceded by an external **read that completes before
the write starts**. The bug is intermittent (easily missed in testing):

```asm
; Example #1 — FAILS (nothing forces the load to finish first)
        load    (r1),r2
        or      r10,r11
        store   r11,(r3)

; Example #2 — works (the load result r2 is needed, so it stalls)
        load    (r1),r2
        or      r2,r11
        store   r11,(r3)

; Example #3 — fix for #1 (dummy use of the load result)
        load    (r1),r2
        or      r2,r2
        or      r10,r11
        store   r11,(r3)
```

### 8. GPU High Data Register clobbered by *any* external load
The GPU High Data Register changes after **any** external load, not just `loadp`.
So if a GPU interrupt handler loads from external memory, the interrupted program
must not use `loadp`.

### 9. Divider bug on back-to-back divides
Two consecutive divides less than **16 clock cycles** apart corrupt the second
result *when* the second divide uses the first divide's quotient as an operand
and nothing has created a scoreboard dependency on that quotient. Work-around:
leave >16 cycles between divides, or insert an instruction dependent on the first
quotient (e.g. `or r1,r1`) before the second divide.

### 10. DSP matrix multiplies only work in the low 4 KB of DSP RAM
The DSP matrix register can only address the first 4 KB of DSP RAM; the rest of
the matrix address is hard-wired to `$F1Bxxx`. See [DSP](../jerry/dsp.md).

### 11. `G_FLAGS` / `D_FLAGS` write latency
After writing `G_FLAGS`/`D_FLAGS`, the change may not be visible for two
instructions (pipelining). If you'll use flags set by a `STORE`, or change a bit
like the register bank, put **two `NOP`s** after the write.

## Blitter bugs

See [Blitter](../tom/blitter.md).

### 1. A1/A2 Y add-control bits not differentiated
The A2 Y add-control bit is ignored; the A1 Y add-control bit affects **both**
address generators. (If a Y sign bit is set, the corresponding add-control bit
must be set for the value to be negative.) Either don't use this function, or
apply it to both generators.

### 2. `SRCSHADE` requires `GOURZ`
`SRCSHADE` only works if `GOURZ` is set. No Z data need actually be computed or
written, but `GOURZ` must be set.

### 3. `A1_CLIP` X off a phrase boundary clips the right side
If `A1_CLIP` X is not on a phrase boundary, right-side clipping occurs even when
the clip bit is clear (and on the destination even with `DSTA2` set). Set
`A1_CLIP` to 0 when not clipping, and make the source an even phrase width when
using `DSTA2`.

### 4. Unaligned 2-bpp mode is unreliable
Unaligned blits in 2-bits-per-pixel mode are unreliable — use 1-bit-per-pixel
blits instead.

### 5. Z-buffer + `ADDDSEL`/`SRCSHADE` corrupts data
With Z-buffering enabled and `ADDDSEL` or `SRCSHADE` set, data is sometimes
corrupted. Split into two blits: do the `SRCSHADE`/`ADDDSEL` into an off-screen
buffer, then a second blit for the Z-buffer pass onto the screen.

## Object Processor bugs

See [Object Processor](../tom/object-processor.md).

### 1. Last column of a RMW object can corrupt
The last column of a read-modify-write object can corrupt if it is followed by
another bitmap object (right side, or left side if `REFLECT` is set). Pad the
object's last pixels transparent, keep the next object off the same scanlines, or
place an always-false branch object after the RMW object.

### 2. `VSCALE` > 7.0 fails
A scaled bitmap object with `VSCALE` greater than **7.0** (`$1111.00000`) fails.
`HSCALE` may go as high as 7.1F (`$0111.11111`).

### 3. 24-bit scaled object needs `HSCALE` = 1.0
Setting `HSCALE` to anything other than **1.0** on a 24-bit scaled bitmap object
distorts it.

## Miscellaneous

### 1. UART start-bit double-shift
If a start bit arrives at a certain phase of the UART's ÷16 timer it is shifted
in twice, left-shifting the data byte. Work-around: precede a packet with a dummy
byte whose MSB is set (e.g. `$80`) and have the receiver discard it; keep
subsequent bytes exactly aligned (exactly 2–4 stop bits before the next start
bit). Re-send the dummy byte after any gap longer than ~2 bit times or any
misalignment. This is the **UART bug** behind the dropped ComLynx/networking
registers — see [Serial I/O](../jerry/serial-io.md).

### 2. 68000 `clr.l`/`move.l …,-(An)` corrupt GPU/DSP space
The 68000 **`clr.l <ea>`** and **`move.l <ea>,-(An)`** instructions write a long
as two 16-bit halves in *reversed* order, which corrupts writes to GPU/DSP
registers and internal RAM. Affected ranges: **`$F02000`–`$F07FFF`** and
**`$F1A000`–`$F1F000`** (the source prints the first low bound as `$F0200`, which
appears to be missing a digit). Outside those ranges the instructions are safe.

Work-arounds: use the GPU/DSP itself (not the 68000) to write GPU/DSP registers,
and use the Blitter to copy into GPU/DSP RAM. To clear a long in internal RISC
RAM use `move`, not `clr.l`. If using a high-level compiler, ensure it does not
emit `clr.l` for accesses to this space. See
[Memory Map](memory-map.md#do-not-touch-registers).

> Other instruction/addressing-mode combinations may share this flaw; these are
> just the ones documented.

## See also

- [Memory Map / Register List](memory-map.md) — the do-not-touch registers
- [Graphics Processor (GPU)](../tom/gpu.md) · [Digital Sound Processor (DSP)](../jerry/dsp.md)
- [Blitter](../tom/blitter.md) · [Object Processor](../tom/object-processor.md)
- [Serial I/O — ComLynx, MIDI](../jerry/serial-io.md) — the UART bug
- [RISC Instruction Set (GPU/DSP)](../reference/risc-instruction-set.md)

<!-- nav:bottom -->
---

◀ **Prev:** [Video & System Clocks, Timing](video-clocks-timing.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [Cartridges, EEPROM Saves & ROM Building](cartridges.md) ▶

**Jump to:** [Architecture](overview.md) · [Memory Map](memory-map.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md) · [CD-ROM](../cdrom/overview.md)
<!-- /nav:bottom -->
