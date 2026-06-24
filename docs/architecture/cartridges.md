<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ Architecture & Memory ▸ **Cartridges, EEPROM Saves & ROM Building**
<!-- /nav:top -->

# Cartridges, EEPROM Saves & ROM Building

The Jaguar cartridge port, saving data to the on-cartridge serial EEPROM, and the
EPROM/EEPROM parts for building your own test cartridges. This is hardware-level
information and is independent of any particular toolchain.

> **Source:** *Technical Reference Manual* (V10), pp. 32–34 (Cartridges and
> NVRAM, Burning Your Own Cartridge EPROMs, EPROMs/EEPROMs for Making
> Cartridges). © Atari Corp. 1995.

## The cartridge port

- Up to **6 MB** of cartridge space, mapped at **`$800000`** (see
  [Memory Map](memory-map.md#system-memory-layout)).
- Cartridges may be **8, 16, or 32 bits wide**. *Note:* the Stubulator ROM in
  development machines currently supports **32-bit-wide cartridges only**.
- **Program code always starts at `$802000`** in both encrypted and
  non-encrypted cartridges.
- In a **non-encrypted** test cartridge, `$800000`–`$801FFF` must be all `$FF`
  (in an encrypted cartridge this region holds the security code — see
  [the reserved ROM area](memory-map.md#system-memory-layout)).

## EEPROM saves (NVRAM)

Cartridges may include a small **serial EEPROM** for high scores, options, and
other persistent data.

> **Read/write only through the Atari-supplied routines** (see the NVRAM sample
> program / `SOURCE\EEPROM`). That is the only way to guarantee reliable
> operation — do not bit-bang the EEPROM yourself.

Two cautions:

- **`JOYSTICK` bit 0 is overloaded.** When read, bit 0 of the `JOYSTICK` register
  (`$F14000`) returns the EEPROM **data-output bit**, not the J0 joystick input.
  J0 has always been used as an output, so this rarely matters — but treat that
  read bit as **random**, not as the J0 level. See
  [Controllers](../controllers/controllers.md) and
  [Memory Map](memory-map.md#joystick-registers-jerry).
- **Keep clear of the EEPROM's address range.** The EEPROM uses addresses in the
  **GPIO0/GPIO1 range `$F14800`–`$F15FFF`**. *Any* inadvertent access (read or
  write) to that range makes subsequent EEPROM reads/writes fail — so don't touch
  it. See [Serial I/O → GPIO decodes](../jerry/serial-io.md).

## Building your own test cartridges

### 32-bit EPROM byte ordering (4-chip Atari blanks)

When building 32-bit test cartridges on Atari's 4-chip EPROM cartridge blanks,
data is split across the chips like this:

| Chip | Byte addresses | Bits in the 32-bit long |
|------|----------------|--------------------------|
| U1 | `$800003`, `$800007`, `$80000B`, … | d0–d7 |
| U2 | `$800002`, `$800006`, `$80000A`, … | d8–d15 |
| U3 | `$800001`, `$800005`, `$800009`, … | d16–d23 |
| U4 | `$800000`, `$800004`, `$800008`, … | d24–d31 |

### EPROM burner

Any EPROM burner capable of handling **4-megabit** chips should work. Atari had
good results with the **Pilot** EPROM burners (manufactured by Advin).

### EPROM types (game code)

EPROM types Atari's test department used successfully (slower access speeds than
shown are **not** recommended; similar chips from other makers are untested):

| Cartridge | Configuration | Manufacturer | Chip code |
|-----------|---------------|--------------|-----------|
| 1 MB (4×4 + 128-byte EEPROM) | (4) 512 Kbit × 8 — 4 Mbit | Macronix | `MX27C4000DC-12` |
| | | Toshiba | `TC574000AD-120` / `TC574000AD-150` |
| | | AMD | `AM27C040-150DC` |
| | | (generic) | `27C400` |
| 2 MB (16×2 + 128-byte EEPROM) | (2) 512 Kbit × 16 — 8 Mbit | — | `27C800` |
| 4 MB (16×2 + 128-byte EEPROM) | (2) 1024 Kbit × 16 — 16 Mbit | — | `27C160` |
| 6 MB (16×2 + 128-byte EEPROM)¹ | (2) 2048 Kbit × 16 — 32 Mbit | — | `27C322` |

¹ 6 MB requires modified Atari PCBs or new non-Atari PCBs.

### EEPROM types (game saves)

| Manufacturer code | Size | Notes |
|-------------------|------|-------|
| `93c46b` | 128 byte | Atari PCBs & duplicates |
| `93c46c` | 128 byte | alternate PCB designs |
| `93lc46c` | 128 byte | alternate PCB designs |
| `93c86b` | 2 KB | Atari PCBs & duplicates |
| `93c86c` | 2 KB | alternate PCB designs |
| `93lc86c` | 2 KB | alternate PCB designs |

## See also

- [Memory Map / Register List](memory-map.md) — `$800000` cartridge space, `$802000` entry point
- [Controllers & Controller Ports](../controllers/controllers.md) — `JOYSTICK` bit 0 overload
- [Serial I/O — ComLynx, MIDI](../jerry/serial-io.md) — GPIO decodes / EEPROM address range
- [Hardware Bugs & Warnings](hardware-bugs.md)
- [CD-ROM Subsystem Overview](../cdrom/overview.md) — the other storage medium

<!-- nav:bottom -->
---

◀ **Prev:** [Hardware Bugs & Warnings](hardware-bugs.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [Connectors, Pinouts & Power](connectors-power.md) ▶

**Jump to:** [Architecture](overview.md) · [Memory Map](memory-map.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md) · [CD-ROM](../cdrom/overview.md)
<!-- /nav:bottom -->
