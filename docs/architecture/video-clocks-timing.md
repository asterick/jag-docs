<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ Architecture & Memory ▸ **Video & System Clocks, Timing**
<!-- /nav:top -->

# Video & System Clocks, Timing

System/video/chroma clock generation, NTSC vs PAL video timing, the programmable video timing registers, and the hardware port pinouts (cartridge/expansion, DSP, video) plus the multi-console network and modem ports.

> **Source:** *Technical Reference Manual* (V10), pp. 8–14. © Atari Corp. 1995.

## Clock Generation

In the Jaguar console, the video clock is chosen to allow an inexpensive RF modulator system. This requires slightly different clock speeds for NTSC and PAL systems (but the difference is only about 0.01%). To be cost effective, the GPU/DSP processor clock speed is the same as the video clock speed, and the 68000 runs at 50% of this clock rate.

| Clock | NTSC | PAL |
|---|---|---|
| Video Clock / GPU/DSP Clock Rate | 26.590906 MHz | 26.593900 MHz |
| 68000 Clock Rate (50% of Video Clock) | 13.295453 MHz | 13.296695 MHz |

### Clock Divider Registers — DO NOT TOUCH

The clock dividers live in Jerry and are set up by the BOOTROM (retail console) or the STUBULATOR (development console). **Never write to them.** The settings in `CLK2`, `CLK3` and `HP` in particular must be correct to make the hardware work at all and to prevent dot crawl. The manual is emphatic: *"We really mean it: DON'T TOUCH THIS!"*

| Register | Address | Function |
|---|---|---|
| `CLK1` | `$F10010` | Processor clock divider |
| `CLK2` | `$F10012` | Video clock divider |
| `CLK3` | `$F10014` | Chroma clock divider (aka CHROMA DIV) |

The [`VMODE`](../tom/object-processor.md#vmode--video-mode-f00028-wo) register and object processor are initialised and started after reset by the bootcode. The only object then in the object list is a stop object, which displays a blank screen and sends the correct video sync signals to the monitor or TV. This also lets the phase-locked loop settle (about a second at start-up). **Do not ever turn video off again** (i.e. by writing zero to [`VMODE`](../tom/object-processor.md#vmode--video-mode-f00028-wo)).

## Pixel (Dot) Clock and Resolution

The video system is programmable within the precision of the supplied video clock. From the video clock the system produces the pixel (or dot) clock. The ratio between the video and pixel clock is set by the high-order bits of the [`VMODE`](../tom/object-processor.md#vmode--video-mode-f00028-wo) register. Pixel counts are the same for NTSC and PAL.

For both PAL and NTSC the "safe" video area is about 40µS wide; the area required to guarantee overscan is about 50µS. These numbers are rough guidelines for artwork and object sizes only — they should **not** be used to calculate values for the video hardware registers.

| Pixel Divisor (VMODE) | # Pixels Non-Overscanned | # Pixels Overscanned |
|---|---|---|
| 1 | 1046 | 1330 |
| 2 | 532 | 655 |
| 3 | 355 | 443 |
| 4 | 266 | 332 |
| 5 | 213 | 266 |
| 6 | 177 | 222 |
| 7 | 152 | 190 |
| 8 | 133 | 166 |

Atari recommends overscanning both vertically and horizontally for all Jaguar software.

- **Divisor 1** requires the object processor be started twice per line and produces a ridiculously high TV resolution — ignored.
- **Divisor 3** gives ~355 overscanned pixels: a good match for 320-pixel-wide computer-style screens.
- **Divisor 4** gives roughly square pixels (recommended for art creation). Overscanned, this allows ~332 pixels; a 320-pixel-wide bitmap gives <4% error. Only the middle 266 should be counted on as visible, leaving a ~27-pixel border each side that may be visible but must not carry essential game information.
- **Divisor 5** gives an overscanned count (256) usefully close to a blittable width.

For vertical overscan, use a screen height of **240 lines for NTSC** and **288 lines for PAL**. The guaranteed visible region for crucial game information is **200 lines for NTSC** and **240 lines for PAL**. Using 200 lines of critical video for both systems is an acceptable simplification.

To properly initialise your program (including video) you must use the standard Jaguar Start-up Code described in the Jaguar Libraries section.

## Programmable Video Timing Registers

These registers define the video raster timing. They are set up at boot and listed for information only — **do not modify them** or unpredictable results will occur.

| Register | Address | Description |
|---|---|---|
| `HP` | `$F0002E` | Horizontal Period |
| `HBB` | `$F00030` | Horizontal Blanking Begin |
| `HBE` | `$F00032` | Horizontal Blanking End |
| `HS` | `$F00034` | Horizontal Sync |
| `HVS` | `$F00036` | Horizontal Vertical Sync |
| [`HDB1`](memory-map.md#system-set-up--video-registers-tom) | `$F00038` | Horizontal Display Begin 1 |
| [`HDB2`](memory-map.md#system-set-up--video-registers-tom) | `$F0003A` | Horizontal Display Begin 2 |
| [`HDE`](memory-map.md#system-set-up--video-registers-tom) | `$F0003C` | Horizontal Display End |
| `VP` | `$F0003E` | Vertical Period |
| `VBB` | `$F00040` | Vertical Blanking Begin |
| `VBE` | `$F00042` | Vertical Blanking End |
| `VS` | `$F00044` | Vertical Sync |
| [`VDB`](memory-map.md#system-set-up--video-registers-tom) | `$F00046` | Vertical Display Begin |
| [`VDE`](memory-map.md#system-set-up--video-registers-tom) | `$F00048` | Vertical Display End |
| [`VEB`](memory-map.md#system-set-up--video-registers-tom) | `$F0004A` | Vertical Equalisation Begin |
| `VEE` | `$F0004C` | Vertical Equalisation End |
| [`VI`](memory-map.md#system-set-up--video-registers-tom) | `$F0004E` | Vertical Interrupt |
| `HEQ` | `$F00054` | Horizontal Equalisation End |

## Video Ports

*Informational only — do not attempt to change these timings or unpredictable results will occur.*

There are four versions of the Jaguar console:

| Video Standard | Where Used |
|---|---|
| NTSC | USA / Canada |
| PAL-I | United Kingdom |
| PAL-B | Germany / other European countries |
| Peritel/Scart | France |

The console has an external video connector supporting Composite video, S-Video, and RGB. An RF Modulator output is present on all versions except the French Peritel/Scart version. The Peritel/Scart version is identical to PAL-B except that it has no RF modulator; Composite, S-Video, and RGB are all available on it with the same timings and characteristics as PAL-B.

### RF and Composite

*Informational only — do not attempt to change these timings.*

| | Chroma Clock (MHz) | Subcarrier | Sound Carrier (MHz) |
|---|---|---|---|
| PAL-I | 4.43361875 | 591.250 | 6 |
| PAL-B | 4.43361875 | 591.250 | 5.5 |
| NTSC Channel 3 | 3.579545 | 61.25 | 4.5 |
| NTSC Channel 4 | 3.579545 | 61.25 | 4.5 |

## Video Timings

*Informational only — do not attempt to change these timings.*

| Parameter | PAL | NTSC | Notes |
|---|---|---|---|
| Video master clock | 26.593900 MHz | 26.590906 MHz | |
| Horizontal period | 64.0 µS | 63.5555 µS | |
| Hsync width | 4.7 µS | 4.76 µS | |
| Hback porch | 5.7 µS | 4.45 µS | |
| Hfront porch | 1.65 µS | 1.27 µS | |
| Equalisation pulse width | 2.35 µS | 2.54 µS | |
| Vertical sync pulse width | 27.3 µS | 29.26 µS | |
| Vertical lines (interlaced) | 625 | 525 | |
| Vertical lines (non-interlaced) | 624 | 524 | |
| Vertical sync pulses | 5 | 6 | Non-interlaced |
| Vertical eq pulses before sync | 5 | 6 | Non-interlaced |
| Vertical eq pulses after sync | 6 | 6 | Non-interlaced |
| Vertical front porch | 12 lines | 12 lines | Non-interlaced |
| Vertical back porch | 17 lines | 12 lines | Non-interlaced |

## Jaguar Console Hardware Ports

### Cartridge / Expansion Port

A custom 50-pin (54-position with key), double-row PCB-mounting edge connector. The far (back) row is **row A**, the near row is **row B**, when looking at the console from the front. Pins 13 and 14 are the KEY (no signal). Pin numbering runs through 54.

| Pin | A | B | | Pin | A | B |
|---|---|---|---|---|---|---|
| 1 | EA10 | GND | | 28 | ED6 | NC |
| 2 | EA9 | GND | | 29 | ED9 | GND |
| 3 | EA11 | EA23 | | 30 | ED13 | UART1 |
| 4 | EA8 | EA22 | | 31 | ED2 | UART0 |
| 5 | EA12 | EA21 | | 32 | ED5 | GND |
| 6 | EA7 | EA20 | | 33 | ED10 | RESET1L |
| 7 | EA13 | EA19 | | 34 | ED12 | CART_IN |
| 8 | EA6 | GND | | 35 | ED3 | CART_OUT |
| 9 | EA14 | NC | | 36 | ED4 | VCC |
| 10 | EA5 | GND | | 37 | ED11 | VCC |
| 11 | EA15 | NC | | 38 | VCC | PLL |
| 12 | EA4 | GND | | 39 | ED31 | NC |
| 13 | KEY | KEY | | 40 | ED16 | E2DATA |
| 14 | KEY | KEY | | 41 | ED23 | NC |
| 15 | EA16 | EA1 | | 42 | ED24 | GPIO0 |
| 16 | EA3 | EA0 | | 43 | ED30 | GPIO1 |
| 17 | EA17 | WAITL | | 44 | ED17 | GPIO2 |
| 18 | EA2 | RESETL | | 45 | ED22 | GPIO3 |
| 19 | EA18 | EWE0L | | 46 | ED25 | GPIO4 |
| 20 | ROM1 | EWE2L | | 47 | ED29 | SCK |
| 21 | GND | ERW | | 48 | ED18 | WS |
| 22 | ED15 | EOE1L | | 49 | ED21 | TXD |
| 23 | ED0 | EOE0L | | 50 | ED26 | RXD |
| 24 | ED7 | GND | | 51 | ED28 | GND |
| 25 | ED8 | EINT0 | | 52 | ED19 | ECPUCLK |
| 26 | ED14 | EINT1 | | 53 | ED20 | GND |
| 27 | ED1 | 9V | | 54 | ED27 | GND |

### DSP Port

A custom 12-pin, two-row edge connector. The top row is **row A**, the bottom row is **row B**. Pin 1 is on the left, pin 6 on the right when looking at the console from the rear.

| Pin | Name | Description |
|---|---|---|
| 1A | GND | Ground |
| 2A | SCK | Synchronous serial clock |
| 3A | WS | Synchronous serial word strobe |
| 4A | TXD | Synchronous serial transmit data (data out) |
| 5A | RXD | Synchronous serial receive data (data in) |
| 6A | GND | Ground |
| 1B | +5V | +5V, 50mA maximum load |
| 2B | UART_TXD | Asynchronous transmit data (data out) |
| 3B | UART_RXD | Asynchronous receive data (data in) |
| 4B | Reserved | Do not connect |
| 5B | Reserved | Do not connect |
| 6B | GND | Ground |

All active signals have 5V TTL levels. The `SCK`, `WS`, `TXD` and `RXD` signals are also connected to the cartridge expansion connector. They are used on the CD-ROM peripheral, so care must be taken to avoid contention (see the audio subsystem section).

### Video Connector

A custom 24-pin, two-row edge connector. The top row is **row A**, the bottom row is **row B**. Pin 1 is on the left, pin 12 on the right when looking at the console from the rear.

| Pin | Name | Description |
|---|---|---|
| 1A | Audio_Left | EIAJ Line Level, left audio |
| 2A | Audio_Gnd | Audio Return (ground) |
| 3A | Reserved | |
| 4A | Video_Gnd | Video Return (ground) |
| 5A | Blue | Blue video, 75 Ohm, 0.7V peak-to-peak |
| 6A | HSync | Horizontal Sync, 75 Ohm, 3.0V peak-to-peak |
| 7A | Green | Green video, 75 Ohm, 0.7V peak-to-peak |
| 8A | Chroma | S-Video Chroma, 75 Ohm, 1.0V peak-to-peak |
| 9A | Reserved | |
| 10A | Reserved | |
| 11A | 9V | 9V DC, 100mA maximum load |
| 12A | Reserved | |
| 1B | Audio_Right | EIAJ Line Level, right audio |
| 2B | Audio_Gnd | Audio Return (ground) |
| 3B | Video_Gnd | Video Return (ground) |
| 4B | Red | Red video, 75 Ohm, 0.7V peak-to-peak |
| 5B | VSL | Composite Sync, +5V, TTL Levels |
| 6B | Reserved | |
| 7B | Video_Gnd | Video Return (ground) |
| 8B | Luma | S-Video Luma, 75 Ohm, 1.0V peak-to-peak |
| 9B | Reserved | |
| 10B | Video_Gnd | Video Return (ground) |
| 11B | Composite | Composite Video, 75 Ohm, 1.0V peak-to-peak |
| 12B | Reserved | |

Reserved signals should be left unconnected — they may be used in future console versions and should be passed through on video adaptors. Terminate the active signals correctly; do not load the 75 Ohm outputs with more than 75 Ohms.

## Multi-Console Games

There are two types of multi-console game:

- **Jaguar Network** — a special Local-Area-Network of multiple Jaguar consoles connected together via the consoles' asynchronous serial port. The low-level networking drivers were in development at the time of writing; contact Jaguar Development Support for further information.
- **Jaguar Modem** — connects two Jaguar consoles over telephone lines. The specification for using the Jaguar modem is described in the document titled *Jaguar Voice Modem*.

## See also

- [System Architecture Overview](overview.md)
- [Memory Map / Register List](memory-map.md) (the do-not-touch timing registers)
- [Object Processor](../tom/object-processor.md)
- [Controllers & Controller Ports](../controllers/controllers.md)

<!-- nav:bottom -->
---

◀ **Prev:** [Memory Map / Register List](memory-map.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [Hardware Bugs & Warnings](hardware-bugs.md) ▶

**Jump to:** [Architecture](overview.md) · [Memory Map](memory-map.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md) · [CD-ROM](../cdrom/overview.md)
<!-- /nav:bottom -->
