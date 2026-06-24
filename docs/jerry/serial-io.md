<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ Jerry — Sound & I/O ▸ **Serial I/O — ComLynx, MIDI, Synchronous Serial**
<!-- /nav:top -->

# Serial I/O — ComLynx, MIDI, Synchronous Serial

Jerry provides an asynchronous UART (for ComLynx networking and MIDI), a synchronous serial interface, and the joystick / general-purpose IO decodes.

> **Source:** *Software Reference Manual — Tom & Jerry* (V10), pp. 78–82. © Atari Corp. 1995.

## Asynchronous Serial Interface (ComLynx and MIDI)

The asynchronous serial interface consists of two wires: `UARTI` (receive data input) and `UARTO` (transmit data output). It is primarily designed to support **ComLynx** but can also be used for **MIDI** transmit and receive. A prescaler register allows programmable baud rates.

Both transmitter and receiver are **double-buffered**: a character can be written to the data register before the previous character has finished transmitting, and a second character can be received before the previous one has been read.

Parity may be ODD, EVEN, or none. Parity on both input and output can be programmed active high or low (the printed format diagram shows active-low polarity; the diagram itself is *(illegible)* in the source).

Two classes of interrupt can be generated, each individually enabled:

- **Receiver interrupts:** Parity Error, Framing Error, Overrun Error, Receive Buffer Full.
- **Transmitter interrupts:** Transmit Buffer Empty.

> **Register naming caveat.** The source documents these registers as `ASICLK`, `ASICTRL`, `ASISTAT`, and `ASIDATA`. The task references them as `ASICLK1`, `ASICTRL1`, `ASISTAT1`, `ASIDATA1` (the numbered ComLynx-channel naming). **These ASI registers were dropped from the latest `JAGUAR.INC`** include file, so a current build environment may not define them — see the UART bug caveat below. Register descriptions are reproduced from the manual as printed.

### ASICLK — Asynchronous Serial Interface Clock (`$F10034`, RW)

16-bit register determining the baud rate:

```
Clock Frequency = System Clock Frequency / (N + 1)
```

where N is the value written. The generated frequency is further divided by sixteen to give the baud rate.

### ASICTRL — Asynchronous Serial Control (`$F10032`, WO)

| Bit | Name     | Description |
|-----|----------|-------------|
| 0   | `ODD`    | Writing a 1 selects odd parity. |
| 1   | `PAREN`  | Parity enable. When parity is disabled, the value of the EVEN bit is transmitted in the parity bit time. |
| 2   | `TXOPOL` | Transmitter output polarity. Writing a 1 makes the `UARTO` output active low. |
| 3   | `RXIPOL` | Receiver input polarity. Writing a 1 makes `UARTI` an inverting input. |
| 4   | `TINTEN` | Enable transmitter interrupts. Note that the asynchronous serial interface bit in the Interrupt Control Register (`JINTCTRL`) also needs to be set. |
| 5   | `RINTEN` | Enable receiver interrupts. As for `TINTEN`, the asynchronous serial interface bit in the Interrupt Control Register also needs to be set. |
| 6   | `CLRERR` | Clear error. Writing a 1 clears any parity, framing, or overrun error conditions. |
| 14  | `TXBRK`  | Transmit break. Setting this bit causes a break level to be transmitted on `UARTO`, forcing the output active. This may be high or low depending on the state of `TXOPOL`. |

All unused bits are reserved and should be written 0.

### ASISTAT — Asynchronous Serial Status (`$F10032`, RO)

| Bit  | Name     | Description |
|------|----------|-------------|
| 0–5  | —        | Reflect the state of the corresponding bits in the `ASICTRL` register. |
| 7    | `RBF`    | Receive Buffer Full. When set, a character has been received and is available in the `ASIDATA` register. |
| 8    | `TBE`    | Transmit Buffer Empty. |
| 9    | `PE`     | Parity Error. A parity error occurred on a received character. |
| 10   | `FE`     | Framing Error. Detected when a non-zero character is received without a stop bit at the expected time. |
| 11   | `OE`     | Overrun Error. Detected when a character is received before the last character was read from `ASIDATA`. |
| 13   | `SERSIN` | Serial Input. Reflects the state of the `UARTI` pin. Sense can be inverted by setting `RXIPOL` in `ASICTRL`. |
| 14   | `TXBRK`  | Transmit Break. Reflects the corresponding bit in `ASICTRL`. |
| 15   | `ERROR`  | Error. Logical OR of `PE`, `FE`, and `OE` — allows a single test for error conditions. |

All unused bits are reserved and may return any value.

### ASIDATA — Asynchronous Serial Data (`$F10030`, RW)

When **read**, returns the last character received in bits [0..7] and zero in bits [8..15]. The act of reading clears the receive-buffer-full condition, allowing subsequent characters to be received.

When **written**, bits [0..7] are transmitted from the `UARTO` pin. Bits [8..15] are unused and should be written as zeros.

### UART Bug Caveat

> There is a bug in the Jaguar UART. If a start bit is detected at a certain phase in the UART's divide-by-16 timer, it will be shifted twice, resulting in a left shift of the data byte.
>
> **Workaround:** Precede a data packet with a dummy byte whose MSB is set (e.g. `$80`); the receiver code should discard this dummy byte. Subsequent bytes should be aligned (i.e. exactly 2, 3, or 4 stop bits before the next start bit). This causes the falling edge of the next start bit to miss the phase of the UART counter that triggers the problem.
>
> If a gap longer than 2 bit-times is left after a byte, or a byte is not exactly aligned with the previous one, the dummy byte must be re-transmitted to re-align the UART counter.

This bug is the reason the ASI registers were removed from the latest `JAGUAR.INC`.

## Synchronous Serial Interface

The synchronous serial interface is controlled by seven registers, all within the DSP's local address space, so the DSP may access them without external bus overhead. Other processors may access them at these addresses. All transfers should be 32-bit, though the registers themselves are only 16-bit. This interface carries the I²S audio stream (transmit/receive sample registers, serial clock, and word strobe).

| Register | Description                   | Address   | Access |
|----------|-------------------------------|-----------|--------|
| `SCLK`   | Serial Clock Frequency         | `$F1A150` | WO |
| `SMODE`  | Serial Mode                    | `$F1A154` | WO |
| `L_DAC` / `LTXD` | Left transmit data    | `$F1A148` | WO |
| `R_DAC` / `RTXD` | Right transmit data   | `$F1A14C` | WO |
| `LRXD`   | Left receive data (from I²S)   | `$F1A148` | WO |
| `RRXD`   | Right receive data (from I²S)  | `$F1A14C` | WO |
| `SSTAT`  | Serial Status                  | `$F1A150` | RO |

The serial clock frequency is `System Clock / (2 * (N+1))`. `SMODE` bit 0 (`INTERNAL`) enables the clock and word strobe outputs; interrupts can be generated on the rising/falling edge of the word strobe or on the MSB of every word. Full register-level details, including the audio sample path, are documented in [Audio Subsystem & Synthesis](audio.md).

## Joystick Interface

Jerry has four outputs which together control four external TTL ICs to provide the joystick interface. There are two registers.

### JOYSTICK — Joystick Register (`$F14000`, RW)

When **read**, the joystick input buffers are enabled and the data reflects the state of the sixteen joystick inputs. Output `JOYLO` is asserted (active low) during the read.

When **written**, the low eight data bits are latched into the joystick output latch, and output `JOYL2` is asserted (active low) during the write. The most significant bit (bit 15) enables the joystick outputs; this bit is cleared (disabled) by reset. Output `JOYL3` is the inverse of the value in bit 15.

> **Bit 8** of this register is the audio mute control — see [Audio Subsystem & Synthesis](audio.md).

### JOYBUTS — Button Register (`$F14002`, RW)

When **read**, the button input buffer is enabled and the data reflects the state of the four button inputs. Output `JOYL1` is asserted (active low) during the read.

## General Purpose IO Decodes

Jerry has six general-purpose IO decode outputs, asserted (active low) in the following address ranges. The term "General Purpose" is a misnomer because most of the outputs are reserved.

| Decode  | Address range          | Notes |
|---------|------------------------|-------|
| `GPIO0` | `$F14800 – $F14FFF`    | RESERVED |
| `GPIO1` | `$F15000 – $F15FFF`    | RESERVED |
| `GPIO2` | `$F16000 – $F16FFF`    | RESERVED (printed as `F16000 - F15FFFh` in source — apparent OCR typo) |
| `GPIO3` | `$F17000 – $F177FF`    | RESERVED |
| `GPIO4` | `$F17800 – $F17BFF`    | RESERVED |
| `GPIO5` | `$F17C00 – $F17FFF`    | RESERVED |

## See also

- [Audio Subsystem & Synthesis](audio.md)
- [Digital Sound Processor (DSP)](dsp.md)
- [Controllers & Controller Ports](../controllers/controllers.md)
- [Memory Map / Register List](../architecture/memory-map.md)

<!-- nav:bottom -->
---

◀ **Prev:** [Audio Subsystem & Synthesis](audio.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [Controllers & Controller Ports](../controllers/controllers.md) ▶

**Jump to:** [Architecture](../architecture/overview.md) · [Memory Map](../architecture/memory-map.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md) · [CD-ROM](../cdrom/overview.md)
<!-- /nav:bottom -->
