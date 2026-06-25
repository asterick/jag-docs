<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ CD-ROM Subsystem ▸ **CD-ROM BIOS API**
<!-- /nav:top -->

# CD-ROM BIOS API

The Jaguar CD BIOS is the **only** supported way to access the CD subsystem. It
provides hardware-transparent control of speed, data path, transfer, and audio.

> **Source:** *Jaguar CD-ROM* developer manual (scanned), © Atari Corp. 1995,
> §2 "Jaguar CD-ROM BIOS" (pp. 5–14, printed). Routine addresses are defined in
> `CD.INC`.

> **Requirement:** *All* access to the CD must be through the BIOS. The BIOS
> insulates your title from CD-vendor and data-transfer-mechanism changes.

## Calling convention

1. Load the documented values into the required registers.
2. `jsr CD_routine` (a 68000 subroutine call) to the routine you want.
3. Routine addresses come from `CD.INC`.

Each CD BIOS call may use **up to 64 bytes of stack**, so `A7` must be set up
correctly before any call.

In a **retail** Jaguar CD system the BIOS is present automatically; in a
development setup it must be loaded into DRAM by your tooling before any CD BIOS
call. Two BIOS revisions exist (2.x and 4.x): a **Butch 1** CD interface chip can
run only revision 2, while **Butch 2** supports either — see
[CD-ROM Hardware](hardware.md#butch--the-cd-interface-chip).

## Error handling

`CD.INC` defines a global error variable **`err_flag`**:

- `0` = no error; non-zero = error.
- Valid **only** immediately after a call documented as setting it; other calls
  may overwrite it.

> **Proper error checking is mandatory.** Failure to check and handle errors may
> block your product from final production approval. Always check `err_flag`
> after calls that set it, and implement a **timeout** so your program never
> waits forever for a call to return.

### Error recovery for read operations (§2.6)

To retry a failed CD read (i.e. [`CD_ptr`](#cd_ptr) returns an error) while in
double-speed mode:

1. Switch to **single speed** with [`CD_mode`](#cd_mode).
2. Switch back to **double speed** with [`CD_mode`](#cd_mode).
3. Re-execute the [`CD_read`](#cd_read).

This makes recovery reliable wherever recovery is actually possible (i.e. the
disc isn't physically damaged).

## Command acknowledge (§2.5)

Several calls offer a choice: **wait for an acknowledge** that the command
completed, or **return immediately**. The rule for "return immediately" mode:
a **[`CD_ack`](#cd_ack) must be issued before any subsequent CD BIOS command.**
With [`CD_read`](#cd_read) in seek mode this delayed acknowledge is implied, so
you must still `CD_ack` before the next CD BIOS command. Returning immediately
lets you do other processing while the command runs.

## Reading data (§2.4)

Data is normally read by calling one of three `CD_init` variants **once**,
followed by any number of [`CD_read`](#cd_read) calls. With current hardware each
`CD_init` loads GPU interrupt code that handles interrupts redirected from
Jerry's I²S interrupt.

| Variant | Speed | Locates data? | Registers | Notes |
|---------|-------|---------------|-----------|-------|
| [`CD_init`](#cd_init) | average | no | none (non-interrupt) | general use |
| [`CD_initf`](#cd_initf) | fastest (~30% faster) | no | more | uses R18–R31 |
| [`CD_initm`](#cd_initm) | slowest | **yes** | none (non-interrupt) | partition-marker search + circular buffers |

> **Warning:** the CD BIOS GPU code cannot tell which interrupts truly came from
> Jerry. **Never enable other interrupts in [`JINTCTRL`](../architecture/memory-map.md#jerry--clocks-timers-serial)** while any `CD_init`
> handler is active, or they will be mistaken for CD interface interrupts.

**Latency budget:** at double speed these interrupts occur about every **90 µs**;
interrupt overhead reduces the usable max latency to **≈54 µs** (less if the
Object Processor is heavily used). No processor with priority over the GPU may
hold the bus longer than this. 68000 vertical-blank handlers are a common
offender — prefer doing object-list updates on the GPU, or keep the 68k handler
tiny.

**Using the DSP instead of the GPU:** install a DSP I²S interrupt handler, call
`CD_jeri` appropriately, and set [`SMODE`](../jerry/audio.md#smode--serial-mode-f1a154-wo) to `$14` (Boot ROM default is `$15`,
restore it when done). This needs no `CD_init`. DSP transfers are subject to
**infrequent unreported data errors** — checksum any data that must be perfect.

**Red Book audio:** use a simple handler that reads incoming CD data and writes
it to the DACs (see `INOUT.DAS` in `\JAGUAR\CDROM`). Then call
[`CD_read`](#cd_read) with the "Just Seek" bit set and the track's time code;
audio plays via your handler and no `CD_init` stores data.

---

## Function reference (§2.7)

Conventions below mirror the manual: **Input**, **Register Usage** (registers the
call clobbers), **Returns**, **Purpose**. "NADA"/"none" = nothing. Many calls set
`err_flag` rather than returning a value in registers.

**Calls:**

- *Init & setup* — [`CD_setup`](#cd_setup) · [`CD_mode`](#cd_mode) · [`CD_init`](#cd_init) · [`CD_initf`](#cd_initf) · [`CD_initm`](#cd_initm)
- *Reading* — [`CD_read`](#cd_read) · [`CD_uread`](#cd_uread) · [`CD_ptr`](#cd_ptr) · [`CD_ack`](#cd_ack)
- *Audio & playback* — [`CD_jeri`](#cd_jeri) · [`CD_mute`](#cd_mute) · [`CD_umute`](#cd_umute) · [`CD_osamp`](#cd_osamp) · [`CD_paus`](#cd_paus) · [`CD_upaus`](#cd_upaus)
- *Transport & disc* — [`CD_spin`](#cd_spin) · [`CD_stop`](#cd_stop) · [`CD_switch`](#cd_switch)
- *Debug-only* — [`CD_getoc`](#cd_getoc)

### Initialization & setup

#### CD_setup
- **Input:** none
- **Register usage:** none
- **Returns:** none
- **Purpose:** **Must** be called to initialize the CD system before *any* other call.

#### CD_mode
- **Input:** `D0.W` speed/mode: bit 0 = speed (0 = single, 1 = double); bit 1 = mode (0 = audio, 1 = data)
- **Register usage:** `D1, D2`
- **Returns:** error code in `err_flag`
- **Purpose:** Sets CD speed and data mode. **Note:** in audio mode the mechanism may alter or mute data to correct for "spikes."

#### CD_init
- **Input:** `A0.L` address of a long-aligned block of GPU RAM **224 bytes** long
- **Register usage:** `A1, D0`
- **Returns:** none
- **Purpose:** Loads GPU support code for [`CD_read`](#cd_read). Uses the DSP interrupt in the GPU; the ISR needs 16 longs of stack and uses only R28–R31 in Bank #0 (the usual GPU-interrupt registers). A primary GPU process must be running and must define the interrupt stack in R31.
- **See also:** [`CD_initf`](#cd_initf), [`CD_initm`](#cd_initm)

#### CD_initf
- **Input:** `A0.L` address of a long-aligned block of GPU RAM **216 bytes** long
- **Register usage:** `A1, D0`
- **Returns:** none
- **Purpose:** ~30% faster `CD_init` that uses more registers (R18–R31, Bank #0). Same GPU-process requirement as `CD_init`.
- **See also:** [`CD_init`](#cd_init), [`CD_initm`](#cd_initm)

#### CD_initm
*(CD BIOS Rev 3.0 and up)*
- **Input:** `A0.L` address of a long-aligned block of GPU RAM **336 bytes** long
- **Register usage:** `A1, D0`
- **Returns:** none
- **Purpose:** Slower `CD_init` variant that adds **automatic data positioning** (partition-marker search) and **circular buffer** support. Uses the DSP interrupt in the GPU; ISR needs 18 longs of stack, R28–R31 Bank #0. Same GPU-process requirement.
- **See also:** [`CD_init`](#cd_init), [`CD_initf`](#cd_initf)

### Reading

#### CD_read
- **Input:**
  - `A0.L` start of destination buffer.
  - `A1.L` end of destination buffer (up to 64 bytes may be written **past** this address).
  - `D0.L` time code to start at: bit 31 set ⇒ "Just Seek"; remaining bits = `mm:ss:ff`.
  - `D1.L` partition-marker longword to search for (*`CD_initm` only*).
  - `D2.L` `N` where 2^N = circular buffer size in bytes; buffer aligned on a 2^(N+1) boundary; minimum `N` = 3; set 0 to disable (*`CD_initm` only*).
- **Register usage:** `A2, D1`
- **Returns:** error code in `err_flag`
- **Purpose:** Transfers data from the CD starting at a time code. **Always returns immediately** — use [`CD_ptr`](#cd_ptr) to track the next write address. If "Just Seek" is set, no data transfers but the CD keeps advancing at the current speed; a [`CD_ack`](#cd_ack) may follow **only** when "Just Seek" is set. The read stops if the buffer pointer exceeds `A1`, even with a circular buffer defined.
- **See also:** [`CD_uread`](#cd_uread), [error recovery](#error-recovery-for-read-operations-26)

Behaviour by `CD_init` variant in effect:

| In effect | Behaviour |
|-----------|-----------|
| `CD_init` | Reads into the buffer until its end. Request a time code **6 frames before** the data you need; the start-of-data partition marker may be anywhere in the first **31 frames (72,912 bytes)**. |
| `CD_initf` | Same as `CD_init`, faster. |
| `CD_initm` | Scans incoming data for the partition marker in `D1`; data immediately past the marker is read into the buffer (auto-located in memory). Supports circular buffers (reads indefinitely until [`CD_uread`](#cd_uread)). **If the marker is never found, this call searches forever.** |
| (none / DSP) | If `CD_jeri` set the path to I²S (`$14`), your custom interrupt handler reads the data. |

#### CD_uread
- **Input:** none
- **Register usage:** `D0`
- **Returns:** error code in `err_flag`
- **Purpose:** Stops the data transfer started by [`CD_read`](#cd_read) (the disc keeps spinning). Used for early termination on error, or to free the resources when CD transfer is no longer needed.
- **See also:** [`CD_read`](#cd_read)

#### CD_ptr
- **Input:** none
- **Register usage:** none
- **Returns:**
  - `A0.L` address of last written data.
  - `A1.L` approximate address of most recent error.
- **Purpose:** Returns the address of the last longword written. If no data has been read it is one long before the buffer start (often a sign the GPU interrupt code isn't running). The address advances in jumps whose size may change between CD versions. Also returns the position of the last detected read error since the last [`CD_read`](#cd_read).
- **See also:** [error recovery](#error-recovery-for-read-operations-26)

#### CD_ack
- **Input:** none
- **Register usage:** `D1`
- **Returns:** nothing in registers; error code in `err_flag`
- **Purpose:** When a call used "return immediately," `CD_ack` waits for the requested action to complete. Any call that does *not* return immediately uses this internally (so it sets `err_flag`).

### Audio & playback control

#### CD_jeri
- **Input:** `D0.W` 0 ⇒ no data to Jerry; `10` ⇒ send data to Jerry
- **Register usage:** `D1`
- **Returns:** none
- **Purpose:** Routes CD data to Jerry's I²S port so audio enters the system without using main system bandwidth.

*(Set the serial path to `$14` for the DSP-handler read method described in [Reading data](#reading-data-24).)*

#### CD_mute
- **Input:** `D0.W` 0 ⇒ return immediately; 1 ⇒ wait for completion
- **Register usage:** `D1`
- **Returns:** error code in `err_flag`
- **Purpose:** Mutes the CD. Audio mode only.
- **See also:** [`CD_umute`](#cd_umute)

#### CD_umute
- **Input:** `D0.W` 0 ⇒ return immediately; 1 ⇒ wait for completion
- **Register usage:** `D1`
- **Returns:** error code in `err_flag`
- **Purpose:** Unmutes the CD. Audio mode only.
- **See also:** [`CD_mute`](#cd_mute)

#### CD_osamp
- **Input:** `D0.W` oversample by 2^x: 0 ⇒ none, 1 ⇒ 2×, 2 ⇒ 4×, 3 ⇒ 8×
- **Register usage:** none
- **Returns:** error code in `err_flag`
- **Purpose:** Sets audio-mode oversampling. The mechanism does the next-best it can if it cannot do the requested amount. **Important:** this multiplies the data rate to Jerry's I²S by the oversample factor — your Jerry software must keep up.

#### CD_paus
- **Input:** `D0.W` 0 ⇒ return immediately; 1 ⇒ wait for completion
- **Register usage:** `D1`
- **Returns:** error code in `err_flag`
- **Purpose:** Pauses the CD. In data mode, data is still sent but is not sensible, and the CD **does not advance** — so a [`CD_read`](#cd_read) while paused fills the buffer with nonsense.
- **See also:** [`CD_upaus`](#cd_upaus)

#### CD_upaus
- **Input:** `D0.W` 0 ⇒ return immediately; 1 ⇒ wait for completion
- **Register usage:** `D1`
- **Returns:** error code in `err_flag`
- **Purpose:** Reverses [`CD_paus`](#cd_paus).
- **See also:** [`CD_paus`](#cd_paus)

### Transport & disc control

#### CD_spin
- **Input:**
  - `D0.W` 0 ⇒ return immediately, 1 ⇒ wait for completion.
  - `D1.W` session to spin up on.
- **Register usage:** none
- **Returns:** error code in `err_flag`
- **Purpose:** Sets the drive to a specific session. **Note:** not actually required to read data in another session.

#### CD_stop
- **Input:** `D0.W` 0 ⇒ return immediately; 1 ⇒ wait for completion
- **Register usage:** `D1`
- **Returns:** error code in `err_flag`
- **Purpose:** Stops the CD.

#### CD_switch
*(CD BIOS Rev 4.0 and up)*
- **Input:** none
- **Register usage:** `D0, D1, A0`
- **Returns:** nothing in registers
- **Purpose:** Lets a different disc be inserted **without a reset**. Call only after a [`CD_stop`](#cd_stop) with "wait for completion" set, then display a graphic asking the user to insert a new disc. When a new disc is inserted, its TOC is read to `$2C00` and control returns. Assume **nothing** about CD state afterward — e.g. reissue [`CD_mode`](#cd_mode). See [Accessing additional discs](programming-guide.md#accessing-additional-discs).

### Debug-only

#### CD_getoc
> **Debug only — never use on a bootable CD.** The Boot ROM loads the TOC to
> `$2C00` automatically.

- **Input:** `A0.L` address of a 1024-byte buffer for the returned multi-session TOC
- **Register usage:** none
- **Returns:** TOC data in the buffer at `A0.L` (see layout below)
- **Purpose:** Fills the buffer with 8-byte records, one per track in track/time order. The first ("0th track") record holds overall disc info.

**First record (disc info):**

| Offset | Meaning |
|--------|---------|
| +0 | Unused, reserved (0) |
| +1 | Unused, reserved (0) |
| +2 | Minimum track number |
| +3 | Maximum track number |
| +4 | Total number of sessions |
| +5 | Start of last lead-out time, absolute minutes |
| +6 | Start of last lead-out time, absolute seconds |
| +7 | Start of last lead-out time, absolute frames |

**Track records:**

| Offset | Meaning |
|--------|---------|
| +0 | Track # (non-zero) |
| +1 | Absolute minutes (0..99), start of track |
| +2 | Absolute seconds (0..59), start of track |
| +3 | Absolute frames (0..74), start of track |
| +4 | Minute # (0..99) |
| +5 | Track duration minutes |
| +6 | Track duration seconds |
| +7 | Track duration frames |

## See also

- [CD-ROM Subsystem Overview](overview.md)
- [Disc & Data Format](data-format.md) — partition markers, headers, TOC layout
- [Programming Procedures & Guidelines](programming-guide.md) — boot & read recipes
- [Memory Map / Register List](../architecture/memory-map.md) — [`SMODE`](../jerry/audio.md#smode--serial-mode-f1a154-wo), [`JINTCTRL`](../architecture/memory-map.md#jerry--clocks-timers-serial), I²S
- [Digital Sound Processor (DSP)](../jerry/dsp.md) and [Audio Subsystem](../jerry/audio.md)

<!-- nav:bottom -->
---

◀ **Prev:** [CD-ROM Subsystem Overview](overview.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [Disc & Data Format](data-format.md) ▶

**Jump to:** [Architecture](../architecture/overview.md) · [Memory Map](../architecture/memory-map.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md) · [CD-ROM](overview.md)
<!-- /nav:bottom -->
