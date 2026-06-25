<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ Programming Examples ▸ **Sample Programs**
<!-- /nav:top -->

# Sample Programs

A guide to the example programs shipped with the Jaguar development system, illustrating particular techniques for talking to the hardware directly.

> **Source:** *Jaguar Sample Programs* (scanned), © Atari Corp. Reproduced for developer reference.

The sample programs are included with the Jaguar Workshop tools. They are not part of the Workshop series itself; each one illustrates a particular technique or method. In most cases the samples are *not* the fastest possible implementation — they trade speed for clarity so the idea is easy to understand. The samples are updated frequently, so each `SOURCE` subdirectory usually contains a `README.TXT` describing late changes; check it. The executables themselves are not always shipped, because each project is meant to be built with the same tools in your developer's kit, which doubles as a sanity check that your installation is correct.

> **Note:** Only samples that exercise the hardware directly — or rely on standard, freely-redistributable startup/utility code — are documented here. Samples built around third-party libraries are omitted.

---

## Jaguar Mandelbrot / Fractal Demo

**What it demonstrates** — How to set up a full-screen bitmap object and then use the GPU to draw into it. The program draws a Mandelbrot set, then a Julia set, and switches back and forth between the two images. The 68000 sets up the parameters for the GPU, then the GPU draws the entire screen.

As shipped, the whole screen is drawn in about 5 seconds. It could be sped up by a factor of 100% or more with a little more optimization — for example, using the DSP to calculate half the picture while the GPU calculates the other half.

**Where it lives** — `\JAGUAR\SOURCE\JAGMAND`.

**Key hardware** — GPU (pixel-by-pixel fractal calculation), a full-screen bitmap object in 320-pixel CRY mode, a 256-entry CRY-mode colour palette, and the vertical-blank interrupt for refreshing the object list. Video mode is set up by the Standard Jaguar Startup Code.

**Files**

| Filename | Description |
| --- | --- |
| `CALCMAND.S` | The actual Mandelbrot calculation code that runs in the Jaguar GPU. |
| `CRY.PAL` | Data for a 256-entry CRY-mode colour palette for palette-based objects. |
| `JAGMAND.S` | Takes control after the startup code has initialized the system. Creates an object list for the background picture, installs an interrupt handler to refresh the object list, and sets the video mode to 320-pixel CRY mode. Then it clears the memory that will be used for the image, and calls the `Mandle` function (in `MANDLE.S`). |
| `MAKEFILE` | Used with MAKE to build the executable program file from source code and data files. |
| `MANDLE.S` | Uses the 68000 to set up the fractal parameters and then calls the GPU to calculate the image. |
| `STARTUP.RGB` | A copy of `\JAGUAR\STARTUP`, used by several of the sample programs in the Jaguar Developer's Kit. |
| `STARTUP.S` | Standard Jaguar Startup Code. Contains the code necessary to properly initialize the Jaguar hardware and display a startup picture, then passes control to the `_start` label in the `JAGMAND.S` module. |

**Flow of execution**

```asm
; STARTUP.S  - Standard Jaguar Startup Code (entry point)
;   * sets up interrupts
;   * sets the video registers correctly for either NTSC or PAL
;   * does other related things that must be done properly at startup
;     time for your program to function
;   * displays a startup screen
;   * when finished, passes control to the _start label in your program
;     (JAGMAND.S in this example)
;
; NOTE: This STARTUP.S has been modified slightly from the version in
; \JAGUAR\STARTUP to allow the use of a different startup picture. That
; type of change is the only one allowed in this file. Changing other
; portions of the file may result in errors that can prevent your
; program from functioning properly.

; JAGMAND.S  - _start
;   * after the startup code has initialized the system, this delays a
;     few seconds so we can look at the startup screen
;   * creates an object list for the background picture
;   * installs an interrupt handler to refresh the object list
;   * sets the video mode to 320-pixel CRY mode
;   * clears the memory that will be used for the image
;   * jumps into the Mandle function (located in MANDLE.S)
```

The object-list creation routine `make_list` is almost identical to the routine `InitLister` in the `STARTUP.S` module; the only parts that change are the labels for the address where the list information is stored.

```asm
; MANDLE.S
;   * uses the 68000 to set up the fractal parameters
;     (coordinates, zoom range, etc.)
;   * calls the GPU to start creating the fractal image

; CALCMAND.S
;   * GPU routine that calculates the fractal image for each pixel of
;     the picture, using the parameters (coordinates, zoom range, etc.)
;     set up by the 68000
```

---

## JagLine, JagSlant, JagBlock, JagSkew, JagShade

**What they demonstrate** — Very simple Blitter usage. Each program sets up a narrow bitmap object and then draws one shape into it with the Blitter.

> **Warning** *(from the source):* the current versions of these programs are *not* intended as general examples of Jaguar programming. They are simple examples of specific Blitter operations, and they take short cuts to this end. Do not use these examples to obtain startup code or as a shell for creating your own programs.

| Program | What it draws |
| --- | --- |
| **JagLine** | A horizontal line, using the Blitter. Sets up a narrow bitmap object, then draws a single yellow line into the top of it. |
| **JagSlant** | A diagonal line, using the Blitter. Sets up a narrow bitmap object, then draws a single yellow line into the top of it. |
| **JagBlock** | A solid yellow rectangle, using the Blitter. Sets up a narrow bitmap object, then draws a single yellow box into the top of it. |
| **JagSkew** | A skewed yellow rectangle, using the Blitter. Sets up a narrow bitmap object, then draws a non-shaded yellow polygon into it. |
| **JagShade** | A shaded yellow parallelogram, using the Blitter. Sets up a narrow bitmap object, then draws a shaded yellow 4-sided polygon into it. |

**Where they live** — `\JAGUAR\SOURCE\BLIT`. This directory contains several demos which share a number of common source code files.

**Key hardware** — the Blitter (line, box and polygon draws), a CRY-mode bitmap object, the vertical-blank interrupt, and the Object Processor list ("Lister").

**Files**

| Filename | Description |
| --- | --- |
| `BLITBLCK.S` | Code for **JagBlock** that calls the Blitter. |
| `BLITLINE.S` | Code for **JagLine** that calls the Blitter. |
| `BLITSHAD.S` | Code for **JagShade** that calls the Blitter. |
| `BLITSKEW.S` | Code for **JagSkew** that calls the Blitter. |
| `BLITSLNT.S` | Code for **JagSlant** that calls the Blitter. |
| `CLEARBAR.S` | Routine, in this file, that uses the Blitter to clear the bitmap memory used by the program. |
| `CRY.PAL` | Data for a 256-entry CRY-mode colour palette for palette-based objects. |
| `INTSERV.S` | The interrupt-handling routines used by all the programs. |
| `JAGLINE.S` | The main program file for **JagBlock**, **JagLine**, **JagShade**, **JagSkew**, **JagSlant**. |
| `LISTBAR.S` | The routines that set up the object list used by all the programs. |
| `MAKEFILE` | Used with MAKE to build the executable program files from source code and data files. |
| `VIDEOINI.S` | The routines that set up the video display used by all the programs. |

**Shared source files**

`BLITBLCK.S`, `BLITLINE.S`, `BLITSHAD.S`, `BLITSKEW.S`, `BLITSLNT.S` — each contains the Blitter code for an individual program. Only one of these files is used by each program (see table above).

`CLEARBAR.S` — a simple subroutine which uses the Blitter to clear the memory used by the bitmap object that displays the picture. It sets up a pattern containing all zeroes, then blits that pattern into the bitmap.

`CRY.PAL` — data for a CRY-mode colour palette, used by objects with 8 bits per pixel or less.

`INTSERV.S` — the routine that installs the vertical-blank interrupt, plus the vertical-blank interrupt service routine (ISR). The ISR simply calls the `Lister` function (in `LISTBAR.S`) which creates the object list.

> Note that re-creating the object list from scratch during each vertical blank is a terrible way to maintain your object list; **please don't do it this way.** It is much more efficient to change only those objects which get changed every frame by the Object Processor. For better examples of creating and maintaining an object list, see the programs in `\JAGUAR\WORKSHOP`, which create object lists of various sizes and complexity. For a specific example of an object list like those used by JagLine, etc., see the routines in `MOU_LIST.S`, located in the `\JAGUAR\WORKSHOP\MOU` directory.

`JAGLINE.S` — the main source file for these programs. It performs program initialization, then transfers control to the `DoBlit` function (which is different for each program — that routine is contained in the `BLITBLCK.S`, `BLITLINE.S`, `BLITSHAD.S`, `BLITSKEW.S` and `BLITSLNT.S` files; each program uses just one of these).

`LISTBAR.S` — the `Lister` routine used to create the object list, plus the routines which save and restore the fields of the object list which are modified during each frame by the Object Processor.

`VIDINIT.S` — the routine that detects the current video standard (NTSC or PAL) and sets up the video registers which control aspects of the video such as the size and position of the borders at the edges of the screen.

---

## Joypad Reading Example

**What it demonstrates** — How to read the Jaguar joypad controllers. The current buttons pressed on the joypad are printed to the screen. Controller #1 is shown on the left side, and Controller #2 is shown on the right side.

**Where it lives** — `\JAGUAR\SOURCE\JOYTEST`.

**Key hardware** — the controller ports (joypad button matrix). See [Controllers & Controller Ports](../controllers/controllers.md) for the register details.

---

## EEPROM Example

**What it demonstrates** — How to read and write information to the EEPROM of a cartridge. The EEPROM is 128 bytes of non-volatile memory on a standard Jaguar cartridge, normally used for storing the user's controller preference settings, high scores, etc. This program demonstrates how to access it.

> **Note:** this program demonstrates the *exact* method required for accessing the EEPROM. Use the code from this program **as is**, without change.

**Where it lives** — `\JAGUAR\SOURCE\EEPROM`.

**Key hardware** — the cartridge serial EEPROM (128 bytes non-volatile). See [Cartridges, EEPROM Saves & ROM Building](../architecture/cartridges.md) for the access protocol.

---

## RGB True Color Bitmap Display Example

**What it demonstrates** — How to set the system up for RGB mode instead of CRY mode. It creates a 16-bit true-colour RGB bitmap object, then draws a number of bands of colour into the object. The program uses only the 68000; while it is not exactly fast, it could be done much faster using the GPU and/or Blitter.

**Where it lives** — `\JAGUAR\SOURCE\TESTRGB`.

**Key hardware** — a 16-bit true-colour RGB bitmap object and the video registers set to RGB mode (as opposed to CRY mode).

---

## Simple DSP Waveform Output

**What it demonstrates** — How to play back a simple waveform using one of the samples in the DSP waveform ROM. Nothing is shown on screen, but you should hear a tone from your speakers.

> **Warning** *(from the source):* the current version of this program is *not* intended as a general example of Jaguar programming. It is a simple example of a specific DSP operation, and it takes short cuts to this end. Do not use this example to obtain startup code or as a shell for creating your own programs.

**Where it lives** — `\JAGUAR\SOURCE\SIMPLE`.

**Key hardware** — the DSP (Jerry) and its waveform sample ROM. See [Digital Sound Processor (DSP)](../jerry/dsp.md).

---

## Blitter Demo

**What it demonstrates** — A "Blitter recipe book" by François-Yves Bertrand. It uses the Blitter to copy a bitmapped picture from the source bitmap to the screen. The tool lets you plug values into the Blitter registers to see what happens.

This is really much more of a tool you can use to figure out what values to use with your own Blitter code, than a sample program. Playing with this program as you read through the Blitter sections of the code is a great way to learn the Jaguar Blitter. With this tool, you can program any of the Blitter register and see the results directly on screen.

The actual program uses two objects:

- The first one is an ATARI logo, 64 x 64, 16 bits per pixel. This is used as the source.
- The second one is the destination buffer. It is 320 x 256, 16 bits per pixel, 3 layers (2 for double buffering and one for *ZBuffer*).

You can move around the buffer with the **UP/DOWN** keys or faster with **1/7** keys on paddle 1. You can change the value of a register with **LEFT/RIGHT** keys or faster with **C/B** keys. The actual register you can change is the base register (for both A1 and A2). If you set the DSTA2 register (on A1 is the source and A2 the reception), the program swaps the A1 base and A2 base. You will have to swap manually all the other registers (`PITCH`, `PIXEL_SIZE`...) to have the correct result on screen.

**Where it lives** — `\JAGUAR\BLITTER`.

**Key hardware** — the full set of Blitter registers (base/`A1`/`A2`, `DSTA2`, `PITCH`, `PIXEL_SIZE`, and others), double-buffered and Z-buffered destination bitmaps. See the [Blitter](../tom/blitter.md) for register-level detail.

---


## See also

- [Object Processor](../tom/object-processor.md)
- [Controllers & Controller Ports](../controllers/controllers.md)
- [Digital Sound Processor (DSP)](../jerry/dsp.md)
- [Cartridges, EEPROM Saves & ROM Building](../architecture/cartridges.md)

<!-- nav:bottom -->
---

◀ **Prev:** [CD-ROM Hardware](../cdrom/hardware.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [Complete Register List](../reference/register-list.md) ▶

**Jump to:** [Architecture](../architecture/overview.md) · [Memory Map](../architecture/memory-map.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md) · [CD-ROM](../cdrom/overview.md)
<!-- /nav:bottom -->
