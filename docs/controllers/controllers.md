<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ Input ▸ **Controllers & Controller Ports**
<!-- /nav:top -->

# Controllers & Controller Ports

Pinouts, the JOYSTICK/JOYBUTS digital-input registers, the standard Joypad matrix, controller identification, and the full range of standard and advanced Jaguar controllers (Tempest rotary, Team Tap 4-player adaptor, 6D, head-mounted tracker, analogue/driving, keyboard/mouse).

> **Source:** *Technical Reference Manual* (V10), pp. 15–30. © Atari Corp. 1995.

## Overview

The Jaguar console has two controller ports: **Controller port 1 (left)** and **Controller port 2 (right)**. Each port provides:

- Four bi-directional digital pins (used as outputs to select which row of controller data to read).
- Six input-only digital pins (split into 4 + 2 button inputs).

> **Note:** Early versions of the Jaguar included an 8-bit ADC on the motherboard. This has been deleted — analogue controllers now require their own ADC chip.

All `J0`–`J15` and `B0`–`B3` signals are TTL-level digital inputs and outputs.

## Signals and Pinouts

Connector: DB15 (female on console). The bi-directional pins are always used as outputs.

| Pin # | Port 1 | Port 2 | Description |
|-------|--------|--------|-------------|
| 1  | J3       | J4       | Bi-directional signal — output to specify which data the controller should return |
| 2  | J2       | J5       | Bi-directional signal — output to specify which data the controller should return |
| 3  | J1       | J6       | Bi-directional signal — output to specify which data the controller should return |
| 4  | J0       | J7       | Bi-directional signal — output to specify which data the controller should return |
| 5  | —        | —        | Reserved |
| 6  | B0 / LP  | B2       | Button input / Light Gun on Port 1 |
| 7  | +5V DC   | +5V DC   | +5V, 50 mA maximum load |
| 8  | n/c      | n/c      | Pulled up to +5V on 4-player adaptor |
| 9  | Gnd      | Gnd      | Ground |
| 10 | B1       | B3       | Button input |
| 11 | J11      | J15      | Input-only signal |
| 12 | J10      | J14      | Input-only signal |
| 13 | J9       | J13      | Input-only signal |
| 14 | J8       | J12      | Input-only signal |
| 15 | —        | —        | Reserved |

**Light gun:** Controller Port 1 also has a light gun input. A TTL rising edge on the `LP` signal (pin 6 of Port 1, shared with `B0`) latches the light pen registers (`LPH` and `LPV`).

## Register Addressing — Digital Inputs

The JOYSTICK and JOYBUTS registers carry the controller signals. Some bits are used for non-controller purposes (audio mute, EEPROM, video standard).

### JOYSTICK — `$F14000` (Read/Write)

Bits `15......8  7......0`

**Read:** `fedcba98 7654321q`

| Bits | Meaning |
|------|---------|
| f–1 (bits 15–1) | Signals J15 to J1 |
| q (bit 0)       | Cartridge EEPROM output data |

**Write:** `exxxxxxm 76543210`

| Bits | Meaning |
|------|---------|
| e (bit 15) | `1` = enable J7 to J0 outputs; `0` = disable J7 to J0 outputs |
| x          | don't care |
| m (bit 8)  | Audio mute — `0` = audio muted (reset state); `1` = audio enabled |
| 7–4        | J7–J4 outputs (Port 2) |
| 3–0        | J3–J0 outputs (Port 1) |

### JOYBUTS — `$F14002` (Read Only)

Bits `15......8  7......0`

**Read:** `xxxxxxxx rrdv3210`

| Bits | Meaning |
|------|---------|
| r (bit 7) | don't care |
| r (bit 6) | reserved |
| d (bit 5) | reserved |
| v (bit 4) | `1` = NTSC video hardware; `0` = PAL video hardware |
| 3–2       | Button inputs B3 & B2 (Port 2) |
| 1–0       | Button inputs B1 & B0 (Port 1) |

## Device Addressing

All controller devices are addressed through the digital lines on the controller ports. Each port has 4 bi-directional pins and 6 input pins; the bi-directional pins are always used as outputs. By writing a 4-bit code to these outputs, 16 rows of 6 bits each can be addressed. Each controller is allocated 4 rows of data, so up to four controllers may be connected to each port (via a 4-player adaptor) — a maximum of 8 controllers total.

Controllers may be connected to the Jaguar in two ways:

1. Directly to the controller port.
2. Via a multi-player adaptor (usually a 4-player adaptor).

## Reading a Jaguar Controller

Reading a controller is done in two steps:

1. **Write a 4-bit row code** to the port's output bits, selecting which row of controller data to read. Bits 3–0 of JOYSTICK are the output bits for Port 1; bits 7–4 are the output bits for Port 2. **The codes for Port 2 are a mirror image of the codes for Port 1 (bit order reversed).** Bit 15 of JOYSTICK must also be set to enable the outputs. Bit 8 controls audio muting, so take care not to clear it accidentally or you will disable your program's sound.
2. **Read back** the JOYBUTS and JOYSTICK registers; they contain the 6 data bits returned by each port.

### Worked example

Writing `$817E` to JOYSTICK reads row 0 of the first controller on Port 1 and the first controller on Port 2:

| Bits | Meaning |
|------|---------|
| `$8000` | Enable JOYSTICK outputs J0–J7 |
| `$0100` | Enable audio (bit 8 of JOYSTICK controls audio mute) |
| `$0070` | Set up read of row 0 (code `%0111`) of controller 0, port 2 |
| `$000E` | Set up read of row 0 (code `%1110`) of controller 0, port 1 |
| **`$817E`** | **combined value written to the JOYSTICK register** |

### Row code / data layout

The six bits of data per row are returned across the input pins. The meaning depends on the row being read and on the controller type. Output codes are written to J3–J0 (Port 1) / J4–J7 (Port 2).

**Controller Port 1** — output pins 1,2,3,4 = (J3)(J2)(J1)(J0); input pins 6,10,14,13,12,11 = (B0)(B1)(J8)(J9)(J10)(J11):

| Output code (J3 J2 J1 J0) | Row | B0 | B1 | J8 | J9 | J10 | J11 |
|---------------------------|-----|----|----|----|----|----|----|
| 0 1 1 1 | Row 3 | C3 | data | data | data | data | data |
| 1 0 1 1 | Row 2 | C2 | data | data | data | data | data |
| 1 1 0 1 | Row 1 | C1 | data | data | data | data | data |
| 1 1 1 0 | Row 0 | data* | data | data | data | data | data |

**Controller Port 2** — output pins 1,2,3,4 = (J4)(J5)(J6)(J7); input pins 6,10,14,13,12,11 = (B2)(B3)(J12)(J13)(J14)(J15):

| Output code (J4 J5 J6 J7) | Row | B2 | B3 | J12 | J13 | J14 | J15 |
|---------------------------|-----|----|----|----|----|----|----|
| 0 1 1 1 | Row 3 | C3 | data | data | data | data | data |
| 1 0 1 1 | Row 2 | C2 | data | data | data | data | data |
| 1 1 0 1 | Row 1 | C1 | data | data | data | data | data |
| 1 1 1 0 | Row 0 | data* | data | data | data | data | data |

> \* Bit `B0` on Port 1 and bit `B2` on Port 2 are used as a special "Bank 0" flag by bank-switching controllers. See *Reading Bank Switching Controllers*.

## Identifying Controller Types

The basic controller type is specified by the `C2` & `C3` bits returned when you read the controller:

| C2 | C3 | Controller Type |
|----|----|-----------------|
| 0 | 0 | Reserved |
| 0 | 1 | Bank Switching (analogue joystick, head-mounted tracker, etc.) |
| 1 | 0 | "Tempest" Rotary |
| 1 | 1 | "Standard" Jaguar Joypad (or nothing connected) |

Software must scan all possible controller positions, including those on a 4-player adaptor, to determine which types are connected. It should then offer the user a choice of compatible controller (where more than one type is attached) or prompt them to attach a supported type when none is found.

The identifying scan must ensure that the Row 0 code for each socket tested is valid for **at least 100 µs** — this allows for an advanced controller being attached, and is only necessary during this phase, after which Row 0 durations can be set to the minimum for the identified controller type. The identifying scan is a separate form of controller read; its only purpose is to allow quick identification of an attached controller type within a single four-row read.

Advanced controllers use a bank-switching technique to return more than the 24 bits available from a standard controller. The specific controller type is identified by bits in the **last bank** of data returned by each controller.

### Bank-switching controller sub-type — Data Bits B1/B3, last bank

| Row 3 | Row 2 | Row 1 | Row 0 | Bank Switching Controller Type |
|-------|-------|-------|-------|--------------------------------|
| 0 | 0 | 0 | 0 | Reserved |
| 0 | 0 | 0 | 1 | Reserved |
| 0 | 0 | 1 | 0 | Reserved |
| 0 | 0 | 1 | 1 | Reserved |
| 0 | 1 | 0 | 0 | Reserved |
| 0 | 1 | 0 | 1 | Reserved |
| 0 | 1 | 1 | 0 | Reserved |
| 0 | 1 | 1 | 1 | Head-mounted Tracker |
| 1 | 0 | 0 | 0 | Reserved |
| 1 | 0 | 0 | 1 | Reserved |
| 1 | 0 | 1 | 0 | Reserved |
| 1 | 0 | 1 | 1 | Reserved |
| 1 | 1 | 0 | 0 | Reserved (PS Controller) |
| 1 | 1 | 0 | 1 | Keyboard / Mouse |
| 1 | 1 | 1 | 0 | 6D Controller |
| 1 | 1 | 1 | 1 | Analogue Joystick or Driving Controller |

> **Note:** The specification for identifying controllers was changed on March 31, 1995. The differences are important but fairly minor from an implementation point of view, and do not affect any hardware on the market as of that date.

## Standard Jaguar Controller Matrix

Matrix for the standard Joypad packed with every Jaguar console, plugged directly into the console. **Reading a zero means the button is depressed.**

| Row | B0 / B2 | B1 / B3 | J8/J12 | J9/J13 | J10/J14 | J11/J15 |
|-----|---------|---------|--------|--------|---------|---------|
| Row 3 | C3 | Option | # | 9 | 6 | 3 |
| Row 2 | C2 | C | 0 | 8 | 5 | 2 |
| Row 1 | C1 | B | * | 7 | 4 | 1 |
| Row 0 | Pause | A | Up | Down | Left | Right |

(Columns map to Port 1 = B0, B1, J8, J9, J10, J11; Port 2 = B2, B3, J12, J13, J14, J15. Row select via J3 J2 J1 J0 / J4 J5 J6 J7 as in the read table above.)

## Rotary "Tempest" Controller

With one or two possible exceptions, all existing rotary controllers were aftermarket products created by Jaguar owners modifying standard Joypad controllers. They should be read just like a standard controller using Socket 0 row codes, giving this matrix:

| Row | B0 / B2 | B1 / B3 | J8/J12 | J9/J13 | J10/J14 | J11/J15 |
|-----|---------|---------|--------|--------|---------|---------|
| Row 3 | 0 (C3) | Option | # | 9 | 6 | 3 |
| Row 2 | 1 (C2) | C | 0 | 8 | 5 | 2 |
| Row 1 | 1 (C1) | B | * | 7 | 4 | 1 |
| Row 0 | Pause | A | — | — | Phase 0 | Phase 1 |

Because these controllers have no Up/Down function, the following buttons are recommended for menu navigation (e.g. Option menu):

- **A = Up**, **B = Select/Change**, **C = Down**

This device is similar to the original Tempest arcade controller. It uses a two-phase switch that software reads to determine the direction of rotation. The phase signals (`Phase 0` and `Phase 1`) form a 2-bit Gray code:

**Anticlockwise sequence**

| Signal | Sequence |
|--------|----------|
| J10 (pin 12) | 0 1 1 0 0 1 1 … |
| J11 (pin 11) | 0 0 1 1 0 0 1 … |

**Clockwise sequence**

| Signal | Sequence |
|--------|----------|
| J10 (pin 12) | 0 0 1 1 0 0 1 … |
| J11 (pin 11) | 0 1 1 0 0 1 1 … |

## 4-Player Adaptor (Team Tap)

Because 16 rows of data can be addressed, a four-controller adaptor can connect to each console port (8 controllers total using two adaptors). The 4-player adaptor expands one console port to four controller sockets (DB15 females, the same as the console) plus a short cable with a DB15 male connector that plugs into the console.

The controller sockets on the adaptor have the 6 inputs wire-OR'd together. The four output lines are an active-low, 4-to-16 de-multiplexed version of the 4 console outputs.

Each socket recognises four unique row codes. **Socket 0 uses the same row codes as a single controller connected directly to a console port.**

### Row codes output from the Jaguar

Output code on J3 J2 J1 J0 (Port 1) / J4 J5 J6 J7 (Port 2):

| Code (bits) | Socket 0 | Socket 1 | Socket 2 | Socket 3 |
|-------------|----------|----------|----------|----------|
| 0 0 0 0 | Row 0 | | | |
| 0 0 0 1 | Row 1 | | | |
| 0 0 1 0 | Row 2 | | | |
| 0 0 1 1 | Row 3 | | | |
| 0 1 0 0 | | Row 0 | | |
| 0 1 0 1 | | Row 1 | | |
| 0 1 1 0 | | Row 2 | | |
| 0 1 1 1 | | Row 3 | | |
| 1 0 0 0 | | | Row 3 | |
| 1 0 0 1 | | | Row 0 | |
| 1 0 1 0 | | | Row 1 | |
| 1 0 1 1 | | | Row 2 | |
| 1 1 0 0 | | | | Row 2 |
| 1 1 0 1 | | | | Row 1 |
| 1 1 1 0 | | | | Row 0 |
| 1 1 1 1 | | | | Row 3 |

> Except for socket 0, the row codes in the table are **not** the codes seen by the controllers themselves. To stay transparent to the controllers, the adaptor converts the row codes for sockets 1–3 so those controllers only see socket 0 row codes. For example, when your program outputs `%0101` (read Row 1 of the controller on socket 2), the adaptor converts it to `%1101` and passes that to socket 2 — the same code used for a single controller plugged directly into the Jaguar.

### 4-Player Adaptor and Advanced Controllers

Originally advanced controllers responded to socket 1 row codes (instead of socket 0), allowing a "pass-through" connector for a standard Joypad. They were then required to change behaviour on detecting a 4-player adaptor, disabling the pass-through and responding to socket 0 row codes.

Due to advancements in microcontroller design this has changed (see *Advanced Controllers*). Advanced controllers are no longer required to check for a 4-player adaptor (indicated by a +5V DC signal on pin 8 of each adaptor socket), though they may still do so if the designer deems it necessary.

Because the 4-player adaptor converts the socket 1–3 row codes to socket 0 row codes, **only a controller read is possible when advanced controllers are connected through a 4-player adaptor.** Software control of advanced features (rumble motors, force feedback, analogue/digital select) will not be possible.

**Summary — socket / controller positions (Ports 1 & 2 are identical in these respects):**

- **With a 4-player adaptor (Sockets 0–3):** The adaptor converts the row codes output by Jaguar programs and routes them to the appropriate socket. Socket 0 is the same as a controller plugged directly into the port. Standard and advanced controllers respond only to socket 0 row codes.
- **Without a 4-player adaptor:** A standard controller plugged directly into the port is the same as socket 0 of a 4-player adaptor. Advanced controllers plugged directly into a port respond to **socket 0 row codes for reads** and **socket 2 row codes for accessing configuration modes and advanced features.**

### Bank Switching Controllers

Because 4 row codes are allocated to each socket, the 4-player adaptor only supports 4-row controller devices. Without additional logic, each input supports up to 24 bits of data (4 rows × 6 bits). Three bits are reserved for the controller-type identifier code, leaving 21 bits for data.

Intelligent controllers (those using a microcontroller) can multiplex more data onto the same lines. One way is for the microcontroller to "bank switch" whenever it sees a transition from row 3 back to row 0; different data is presented in each bank. See *Reading Bank Switching Controllers*.

### Detecting the 4-Player Adaptor & Connected Controllers

To detect a 4-player adaptor, inquire the status of **Row 1 of controller socket #3**. If an adaptor is present, the `B0/B2` bit will be **clear (0)**; otherwise it will be **set (1)**.

```text
For PORT = 1 to 2
    if PORT:SOCKET3:C1 = 0 then  { 4-player adaptor found }
        for SOCKET = 0 to 3
            PORT:SOCKET:CONTROLLERTYPE = PORT:SOCKET:C2/C3
            if PORT:SOCKET:CONTROLLERTYPE = BANK-SWITCHING then
                PORT:SOCKET:BANKSWITCHTYPE = DETECT_BANK_SWITCH_TYPE
            end if
        next SOCKET
    else
        PORT:SOCKET0:CONTROLLERTYPE = STANDARD
        if PORT:SOCKET:C2/C3 = ROTARY then
            PORT:SOCKET1:CONTROLLERTYPE = ROTARY
        else if PORT:SOCKET1:C2/C3 = BANK_SWITCHING then
            PORT:SOCKET:BANKSWITCHTYPE = DETECT_BANK_SWITCH_TYPE
        endif
    endif
next PORT

FUNCTION_DETECT_BANK_SWITCH_TYPE
    DO
        READ ROWS 0, 1, 2, 3
    UNTIL ROW0:B0/B2 = 0   { Bank 0 }
    BANKCOUNT = 0
    DO
        READ ROWS 0, 1, 2, 3
        SAVE ROWDATA( BANKCOUNT )
        BANKCOUNT = BANKCOUNT + 1
    UNTIL ROW0:B0/B2 = 0   { Bank 0 }
    return ROWDATA(BANKCOUNT - 1) : ROWS0-3:B1/B3
END FUNCTION
```

### Caveats

The JOYSTICK and JOYBUTS registers return the same data in the same bits regardless of which socket is being read. However, **without a 4-player adaptor, reading sockets 1–3 of a port may return an "echo" of the standard Joypad at socket 0.** To avoid reading incorrect data, unless your program has detected that a 4-player adaptor is connected, it should not try to read from sockets 1–3.

## Advanced Controllers

### General Information

All advanced controllers must contain a microcontroller as their interface to the Jaguar. Where analogue values are used, the microcontroller should use its own internal ADC or interface to a separate ADC chip within the controller.

Advanced controllers must conform to the following:

1. When powered separately, controllers must use the +5 pin from the controller port to detect when the Jaguar is switched off, and be held in Master Clear/Reset while the Jaguar is off.
2. Set their outputs to logic 1 at Jaguar power-up to prevent issues with boot ROM controller reads.
3. Respond to **socket 0 row codes for controller reads** and **socket 2 row codes for accessing configuration modes and advanced features.**
4. Consume no more than 50 mA of current from the controller port's +5V pin — and no more than 10 mA to be suitable for games supporting 3 or more players via a 4-player adaptor.
5. Provide for connecting an external high-current power source where the controller incorporates high-current loads such as rumble motors or force feedback.
6. If an advanced controller cannot house the required number of buttons internally (especially the critical Pause and Option buttons), it must have a DB15 female connector for attaching a standard Jaguar controller. The advanced controller must then read the standard controller and send that information as one of its banks of output data (see *Reading Bank Switching Controllers*).
7. During the Jaguar's "identifying connected controller" read, advanced controllers must output a special bank of data allowing full identification and current configuration in a single bank read / one pass. See *Identifying Connected Controller Read*.

To prevent problems caused by boot ROM row codes, advanced controllers should time the duration of Socket 0, Row 0 codes and not output their initial Row 0 data until they see a Socket 0, Row 0 code valid for **at least 100 µs**. This is only necessary during the "identifying connected controller" read; afterward all row-code timings are assumed correct for the controller type. After the identifying phase it is recommended (but not required) that advanced controllers issue data starting at Bank 0, since Bank 0 identification is included in the bank data — but doing so makes it easier for the programmer.

### Reading Bank Switching Controllers

Most advanced controllers output more than the single 24 bits of the standard Joypad. Bank switching splits data into multiple **banks** (groups) of 24 bits, where one standard Joypad read is required per bank. Unlike the standard Joypad, the meaning of the 24 bits changes per bank and depends on the controller type.

- Bank switching is done **automatically** when the controller sees a transition from row 3 to row 0 (of the same socket). You cannot read only a particular bank — you must always read all banks even if you don't need all the information.
- Programs must always read an entire bank at once. It is **not** required to read all banks from a single controller in a single pass — you may interleave bank reads across controllers.
- The rows of each bank must be read **in sequence: Row 0, Row 1, Row 2, Row 3.** The controller relies on this so it can pre-process the next row's data. Reading out of sequence gives undefined results.

Example read order for an analogue joystick:

| Bank | Rows read |
|------|-----------|
| Bank 0 | Row 0 (controller automatically bank-switches here, then returns Row 0 data), then Row 1, Row 2, Row 3 |
| Bank 1 | Row 0 (controller automatically bank-switches here, then returns Row 0 data), then Row 1, Row 2, Row 3 |

You need not know in advance which bank is active. Read all banks into a table, then find Bank 0: bank-switching controllers indicate **Bank 0** by clearing **bit 0 (B0, Port 1)** or **bit 2 (B2, Port 2)** of the JOYBUTS value read from Row 0. The bit is `0` for Bank 0 and `1` for all other banks. Because banks are always read in sequence, once you locate Bank 0 you know where every other bank's data is. (Example: reading 3 banks of a 6D controller into words 0–23; if word 9's bit 0 is clear, Bank 0 is words 8–15, Bank 1 is words 16–23, and Bank 2 is words 0–7.)

**Timing:** There is processing time between rows because the controller's microcontroller must put a new data set on the outputs — normally ~10 µs (worst case ~20 µs) row-to-row within the same bank. Analogue controllers typically require an additional ~200 µs going from one bank to the next (so analogue inputs can be digitised).

### Identifying Connected Controller Read

Special output bank data for the identifying-connected-controller sweep:

| Row | B2/B0 | B3/B1 | J12/J8 | J13/J9 | J14/J10 | J15/J11 |
|-----|-------|-------|--------|--------|---------|---------|
| Row 3 | 1 (C3)* | 1** | #Con1 | #Con2 | #Con3 | #Con4 |
| Row 2 | 0 (C2)* | 1** | CurCon1 | CurCon2 | CurCon3 | CurCon4 |
| Row 1 | 1 (C1)* | 1** | 1 | 1 | 1 | 1 |
| Row 0 | 1* | 0** | 1 | 1 | 1 | 1 |

> \* Bits B0/B2 show values received for a bank-switching controller, where the C3 and C2 bits identify the basic controller type.
> \*\* When the C2 and C3 bits indicate a bank-switching controller, the B1/B3 bits identify the bank-switching controller sub-type (see *Identifying Controller Types*).
>
> **#Con** bits indicate the number of different configurations the controller supports (e.g. a mouse may have two configurations: two-button and three-button — see controller documentation).
> **CurCon** bits indicate the default (current) controller configuration.

### 6D Controllers

These support 6 degrees of freedom: Pitch, Yaw, Roll, X, Y and Z. Pitch is Z torque, Yaw is X torque, Roll is Y torque — giving 6 values: X, Y, Z, TX, TY, TZ. Seven buttons A–G are defined. Three banks are required (55 bits: 8-bit values for each of 6 DOF = 48 bits, plus 7 buttons).

**Bank 0**

| Row | B2/B0 | B3/B1 | J12/J8 | J13/J9 | J14/J10 | J15/J11 |
|-----|-------|-------|--------|--------|---------|---------|
| Row 3 | 1 (C3)** | D | X4 | X5 | X6 | X7 |
| Row 2 | 0 (C2)** | C | Z0 | Z1 | Z2 | Z3 |
| Row 1 | 1 (C1) | B | Y0 | Y1 | Y2 | Y3 |
| Row 0 | 0* | A | X0 | X1 | X2 | X3 |

**Bank 1**

| Row | B2/B0 | B3/B1 | J12/J8 | J13/J9 | J14/J10 | J15/J11 |
|-----|-------|-------|--------|--------|---------|---------|
| Row 3 | 1 (C3)** | E | Y4 | Y5 | Y6 | Y7 |
| Row 2 | 0 (C2)** | F | TZ0 | TZ1 | TZ2 | TZ3 |
| Row 1 | 1 (C1) | G | TY0 | TY1 | TY2 | TY3 |
| Row 0 | 1* | Rezero | TX0 | TX1 | TX2 | TX3 |

**Bank 2**

| Row | B2/B0 | B3/B1 | J12/J8 | J13/J9 | J14/J10 | J15/J11 |
|-----|-------|-------|--------|--------|---------|---------|
| Row 3 | 1 (C3)** | 1** | Z4 | Z5 | Z6 | Z7 |
| Row 2 | 0 (C2)** | 1** | TZ4 | TZ5 | TZ6 | TZ7 |
| Row 1 | 1 (C1) | 1** | TY4 | TY5 | TY6 | TY7 |
| Row 0 | 1* | 0** | TX4 | TX5 | TX6 | TX7 |

> \* Bit B0/B2 of row 0 synchronises the bank cycle: always 0 in Bank 0, 1 in all others. Banks cycle Bank 0, Bank 1, Bank 2, Bank 0, …
> \*\* The C3/C2 bits identify the basic controller type. The B1/B3 bits of the **last** bank identify the specific bank-switching controller type.

| Value | Meaning |
|-------|---------|
| X(7:0)  | X axis force |
| Y(7:0)  | Y axis force |
| Z(7:0)  | Z axis force |
| TX(7:0) | X axis, anticlockwise rotation torque |
| TY(7:0) | Y axis, anticlockwise rotation torque |
| TZ(7:0) | Z axis, anticlockwise rotation torque |

Sign conventions: **X** is positive right to left; **Y** is positive UP; **Z** is positive coming BACK (towards the user). Torques are all positive in the **counter-clockwise** direction when facing the positive axis direction.

### Reading a Joypad Attached to an Advanced Controller

When an advanced controller reads an attached Joypad, it should behave as the Jaguar itself: send the four Socket 0 row codes and read back 24 bits as four rows of six data bits. However, the positions of two data bits — **Pause and C3 — must be switched** before the Joypad data is output as one of the advanced controller's data banks:

| Row | B2/B0 | B3/B1 | J12/J8 | J13/J9 | J14/J10 | J15/J11 |
|-----|-------|-------|--------|--------|---------|---------|
| Row 3 | Pause | Option | # | 9 | 6 | 3 |
| Row 2 | 1 (C2) | C | 0 | 8 | 5 | 2 |
| Row 1 | 1 (C1) | B | * | 7 | 4 | 1 |
| Row 0 | 1 (C3) | A | Up | Down | Left | Right |

This prevents Bank 0 synchronisation issues (Pause would otherwise present as a false Bank 0 indicator) when reading the data banks into a table.

### Head Mounted Tracker

These devices provide three angular values according to the orientation of the user's head.

**Bank 0**

| Row | B2/B0 | B3/B1 | J12/J8 | J13/J9 | J14/J10 | J15/J11 |
|-----|-------|-------|--------|--------|---------|---------|
| Row 3 | 1 (C3)** | 1 | 1 | 1 | 1 | 1 |
| Row 2 | 0 (C2)** | 1 | AZ0 | AZ1 | AZ2 | AZ3 |
| Row 1 | 1 (C1) | 1 | AY0 | AY1 | AY2 | AY3 |
| Row 0 | 0* | 1 | AX0 | AX1 | AX2 | AX3 |

**Bank 1**

| Row | B2/B0 | B3/B1 | J12/J8 | J13/J9 | J14/J10 | J15/J11 |
|-----|-------|-------|--------|--------|---------|---------|
| Row 3 | 1 (C3)** | 0** | 1 | 1 | 1 | 1 |
| Row 2 | 0 (C2)** | 1** | AZ4 | AZ5 | AZ6 | AZ7 |
| Row 1 | 1 (C1) | 1** | AY4 | AY5 | AY6 | AY7 |
| Row 0 | 1* | 1** | AX4 | AX5 | AX6 | AX7 |

> \* Bit B0/B2 of row 0 synchronises the bank cycle: always 0 in Bank 0, 1 in all others. Banks cycle Bank 0, Bank 1, Bank 2, Bank 0, …
> \*\* The C3/C2 bits identify the basic controller type. The B1/B3 bits of the last bank identify the specific bank-switching controller type.

| Value | Meaning |
|-------|---------|
| AX(7:0) | Rotation angle around X (= roll = head tilted) axis |
| AY(7:0) | Rotation angle around Y (= yaw = looking left/right) axis |
| AZ(7:0) | Rotation angle around Z (= pitch = looking up/down) axis |

Zero is facing straight ahead. Positive values are tilt left / look left / look up. Values are linear angle values, where +180° = `$7F`, -179° = `$80`.

### Analogue (Joystick and Driving) Controller

These devices typically require 8 bits of analogue resolution in 2 dimensions (X and Y). Two 100K-ohm linear potentiometers are typically used with +5V across the ends; the centre wiper reads a voltage between 0 and +5.

Reading this voltage requires an ADC. A good solution is an ARM or Microchip microcontroller; e.g. a 16F73 or 18F24K20 has 6 ADC channels and 16 general-purpose digital I/O lines. The four controller row outputs select one of four 6-bit addresses. The two 8-bit ADC values use 16 bits, leaving room for 5 switches and the 3 controller identifier codes.

The example below uses bank switching to support more switches. The bank is switched when the microcontroller sees a transition from Row 3 to Row 0; bank identification is by reading bits B0/B2 of Row 0.

**Bank 0**

| Row | B2/B0 | B3/B1 | J12/J8 | J13/J9 | J14/J10 | J15/J11 |
|-----|-------|-------|--------|--------|---------|---------|
| Row 3 | 1 (C3)** | D | Y4 | Y5 | Y6 | Y7 |
| Row 2 | 0 (C2)** | C | Y0 | Y1 | Y2 | Y3 |
| Row 1 | 1 (C1) | B | X4 | X5 | X6 | X7 |
| Row 0 | 0* | A | X0 | X1 | X2 | X3 |

**Bank 1**

| Row | B2/B0 | B3/B1 | J12/J8 | J13/J9 | J14/J10 | J15/J11 |
|-----|-------|-------|--------|--------|---------|---------|
| Row 3 | 1 (C3)** | 1** | 1 | 1 | 1 | 1 |
| Row 2 | 0 (C2)** | 1** | 1 | 1 | 1 | 1 |
| Row 1 | 1 (C1) | 1** | 1 | 1 | 1 | 1 |
| Row 0 | 1* | 1** | Up | Down | Left | Right |

> \* Bit B0/B2 of row 0 synchronises the bank cycle: always 0 in Bank 0, 1 in all others. Banks cycle Bank 0, Bank 1, Bank 2, Bank 0, …
> \*\* The C2/C3 bits identify the basic controller type. The B1/B3 bits of the last bank identify the specific bank-switching controller type.

| Signal | "Stick" Controller | "Driving" Controller |
|--------|--------------------|----------------------|
| X (7:0) | Roll — Right = positive delta from centre; Left = negative delta | Steering — Right = positive delta from centre; Left = negative delta |
| Y (7:0) | Pitch — Forward = positive delta from centre; Backward = negative delta | Accelerator/Brake — Accelerator = positive delta from centre; Brake = negative delta |
| Up    | Hat Switch "Up"    | Gear shift Up |
| Down  | Hat Switch "Down"  | Gear shift Down |
| Left  | Hat Switch "Left"  | Spare 1 |
| Right | Hat Switch "Right" | Spare 2 |
| A | Top Switch     | Spare 3 |
| B | Trigger Switch | Spare 4 |
| C | Middle Switch  | Spare 5 |
| D | Lower Switch   | Spare 6 |

**Calibration:** The range of possible X and Y values is 0–255, but not all controllers use the entire range, and the range used is not predefined. Do not assume fixed values for centre, hard-right, and hard-left positions — analogue devices differ controller to controller, and even day to day as temperature and humidity change. (E.g. one driving controller might return 160 centred / 245 hard right / 75 hard left, while another of the same type returns 150 / 240 / 55.)

Provide a calibration routine that asks the user to move the controller to certain positions to read the values there; offer this as an option on your controller configuration screen, ideally also allowing recalibration while paused mid-game. Storing the current calibration values into the cartridge EEPROM lets the user avoid recalibrating each session.

**Timing:** Analogue controllers require processing time from when the row code is written until the read-back data is valid — normally ~10 µs (worst case ~20 µs) row-to-row within the same bank (this delay applies to all bank-switching controllers), and ~200 µs between banks. Handle this with a small delay loop that uses the bus as little as possible (avoid memory access), or by writing the row code on one timer/GPU interrupt and reading the value on a later interrupt. Avoid wasting CPU time and bus bandwidth just waiting on the controllers when other work could be done.

> **Note:** The ~10/20/200 µs figures were arrived at using a prototype Dual Shock controller interface comprising a Microchip PIC microcontroller running at 64 MHz. Consequently, all microcontrollers used in advanced controllers should run at 64 MHz or higher for the given timings to be relevant.

## Advanced Controller Configuration Modes

When connected directly to a Jaguar controller port, advanced controllers use **Socket 2 row codes** to enter and exit configuration modes:

| Socket 2 Row Code | Action |
|-------------------|--------|
| Socket 2, Row 0 | Enter Vibration/Force Feedback control mode |
| Socket 2, Row 1 | Exit Vibration/Force Feedback control mode |
| Socket 2, Row 2 | Enter configuration mode |
| Socket 2, Row 3 | Exit configuration mode |

Row codes sent between the respective pairs of Enter and Exit row codes are used to set or turn on/off the respective features, and should be detailed in the relevant controller documentation.

## Keyboard/Mouse Interface

> **Note:** The specifications for this controller type are still in the preliminary stages and are subject to change without notice.

A Keyboard / Mouse controller is identified as a bank-switching controller sub-type via the last-bank B1/B3 bits (Row 3=1, Row 2=1, Row 1=0, Row 0=1 — see *Identifying Controller Types*). The number of supported configurations is reported via the `#Con` bits, and the default via the `CurCon` bits, in the *Identifying Connected Controller Read* (e.g. a mouse may have two-button and three-button configurations).

## See also

- [Memory Map / Register List](../architecture/memory-map.md)
- [Serial I/O — ComLynx, MIDI](../jerry/serial-io.md)
- [System Architecture Overview](../architecture/overview.md)
- [Video & System Clocks, Timing](../architecture/video-clocks-timing.md)

<!-- nav:bottom -->
---

◀ **Prev:** [Serial I/O — ComLynx, MIDI, Synchronous Serial](../jerry/serial-io.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [Jaguar Voice Modem](../peripherals/voice-modem.md) ▶

**Jump to:** [Architecture](../architecture/overview.md) · [Memory Map](../architecture/memory-map.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md) · [CD-ROM](../cdrom/overview.md)
<!-- /nav:bottom -->
