<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ CD-ROM Subsystem ▸ **CD-ROM Programming Procedures & Guidelines**
<!-- /nav:top -->

# CD-ROM Programming Procedures & Guidelines

Practical recipes for booting from CD, reading data reliably, minimizing delays,
swapping discs, and the rules your title must follow for production approval.

> **Source:** *Jaguar CD-ROM* developer manual (scanned), © Atari Corp. 1995,
> §2 and §6 (pp. 5–14, 27–31, printed). This is a "living document" in the
> original — details may change, but not in ways that require game-code changes.

## Boot sequence

1. The CD Boot ROM authenticates the disc, then loads the **TOC to `$2C00`**.
2. It reads the **boot track** (first track of Session #1), takes the load
   address and code length from the two Motorola longwords after the header, and
   loads up to **64 KB** of your startup code to that address.
3. It transfers 68000 control to the start of your startup code.
4. Your code parses the TOC at `$2C00`, finds the first track of Session #1, and
   computes all other track/time codes as offsets from it.

See [boot track format](data-format.md#the-boot-track).

> **Do:** rely on the TOC at `$2C00` (already loaded for you).
> **Don't:** call [`CD_getoc`](bios-api.md#cd_getoc) in shipping code, and don't
> reference absolute track numbers — mastering renumbers them.

## Reading data reliably

Typical flow (GPU path):

1. [`CD_setup`](bios-api.md#cd_setup) — once, before anything else.
2. [`CD_mode`](bios-api.md#cd_mode) — set speed (double) and mode (data).
3. One of [`CD_init`](bios-api.md#cd_init) / [`CD_initf`](bios-api.md#cd_initf) /
   [`CD_initm`](bios-api.md#cd_initm) — once, to load the GPU ISR.
4. [`CD_read`](bios-api.md#cd_read) — as many times as needed; it returns
   immediately.
5. Poll [`CD_ptr`](bios-api.md#cd_ptr) for the write position; check `err_flag`.
6. [`CD_uread`](bios-api.md#cd_uread) to stop a transfer early or free resources.

**Inexact reads.** With `CD_init`/`CD_initf`, request a time code **6 frames
before** the data you need and **search the first 31 frames (72,912 bytes)** for
your [partition marker](data-format.md#partition-markers). With
[`CD_initm`](bios-api.md#cd_initm), the BIOS finds the marker (passed in `D1`)
and auto-locates the data — no manual search.

**The latency rule.** At double speed the CD interrupt fires every ~90 µs,
leaving a usable budget of **≈54 µs** (less with heavy Object Processor use). No
processor with priority over the GPU may hold the bus longer than that.
- The classic failure: a **68000 vertical-blank handler that takes too long** —
  symptom: `CD_read` stops after transferring 5–20 KB.
- **Don't build object lists in the 68000 VBL.** Do object-list updates on the
  GPU, or keep the 68k handler tiny.
- While a `CD_init` handler is active, **don't enable other interrupts in
  `JINTCTRL`** — they'll be misread as CD interrupts.

**DSP read path (alternative).** Install a DSP I²S handler, call `CD_jeri`, set
`SMODE = $14` (restore the Boot ROM default `$15` when done). No `CD_init`
needed, but DSP transfers can have **infrequent unreported errors** — checksum
data that must be perfect.

## Error handling (mandatory)

- Check `err_flag` after every call documented to set it
  ([see error model](bios-api.md#error-handling)).
- Always implement a **timeout** so you never wait forever for a call.
- **Read retry** (double-speed read failed per [`CD_ptr`](bios-api.md#cd_ptr)):
  [`CD_mode`](bios-api.md#cd_mode) to single speed → `CD_mode` back to double
  speed → re-issue [`CD_read`](bios-api.md#cd_read).
- Proper error handling is a **production-approval requirement**.

## Minimizing startup delay

Startup runs disc authentication, which scans your code for
[partition markers](data-format.md#partition-markers) that split data into
manageable blocks.

- Use a **small number of sessions**.
- Place a partition marker roughly every **128 KB–1 MB** in any track over 1 MB.
- Worst-case authentication delay ≈ the time to read the data between the two
  most widely separated headers — so don't leave huge unmarked gaps.

## Minimizing loading delay

- **Plan ahead:** design so there's time to load new data in the background
  (the technique the Cinepak demos use for continuous data far larger than DRAM
  with *no* loading delays).
- Use [`CD_initm`](bios-api.md#cd_initm)'s circular-buffer
  [`CD_read`](bios-api.md#cd_read) to read continuously with no extra code.
- Designing both gameplay and code to hide loading is real effort but worth it.

## Accessing additional discs

For multi-disc titles, or to let the user swap in a Red Book audio disc,
[`CD_switch`](bios-api.md#cd_switch) (BIOS rev 4.0+) accepts a new disc and
re-reads the TOC **without a reset**.

Procedure:

1. [`CD_stop`](bios-api.md#cd_stop) with "wait for completion" set.
2. Display a graphic asking the user to insert the disc.
3. Call [`CD_switch`](bios-api.md#cd_switch) (stay in "wait for completion").
4. The BIOS waits for the lid to open then close; if a disc was inserted it reads
   the new TOC to `$2C00` and returns. (If no disc was inserted, it keeps
   waiting.)
5. Assume **nothing** about CD state afterward — reissue
   [`CD_mode`](bios-api.md#cd_mode) etc.
6. Parse the new TOC at `$2C00` and branch on what it is.

Disc-switch decision flow (from the manual's flowchart):

![CD_switch disc-swap flowchart: CD_stop (wait), show an insert-disc graphic, call CD_switch, the BIOS waits for the lid to open then close, loops until a disc is inserted, reads the TOC to $2C00, then your code parses the TOC and branches on whether it is the wrong multi-session disc, the requested multi-session disc, or a single-session disc.](../assets/cdrom-switch-flow.svg)

## Do's and don'ts (summary)

**Do**
- Route *all* CD access through the [CD BIOS](bios-api.md).
- Call [`CD_setup`](bios-api.md#cd_setup) first.
- Work from the TOC at `$2C00`; use offsets, not absolute track numbers.
- Pre-seek 6 frames early and search 31 frames (or use `CD_initm`).
- Check `err_flag`; implement timeouts; checksum DSP-path data.
- Keep 68k interrupt handlers tiny; update object lists on the GPU.
- Add the required Session #0 dummy audio track and the last-session dummy end
  track (156,192 bytes).

**Don't**
- Touch the CD hardware directly.
- Call [`CD_getoc`](bios-api.md#cd_getoc) in shipping code.
- Use `ATRI`, `0x00000000`, or `0xFFFFFFFF` as a partition marker.
- Enable extra `JINTCTRL` interrupts while a `CD_init` handler runs.
- Build object lists in the 68000 vertical-blank handler.
- Put non–Red-Book data in Session #0 (it blocks compatibility encoding).

## See also

- [CD-ROM BIOS API](bios-api.md) — full call reference
- [Disc & Data Format](data-format.md) — headers, partition markers, boot track
- [CD-ROM Subsystem Overview](overview.md)
- [CD-ROM Development Tools](tools.md) — emulator & authoring for testing this

<!-- nav:bottom -->
---

◀ **Prev:** [Disc & Data Format](data-format.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [CD-ROM Development Tools](tools.md) ▶

**Jump to:** [Architecture](../architecture/overview.md) · [Memory Map](../architecture/memory-map.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md) · [CD-ROM](overview.md)
<!-- /nav:bottom -->
