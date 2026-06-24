<!-- nav:top -->
[🏠 Atari Jaguar Developer Reference](../index.md) ▸ Peripherals ▸ **Jaguar Voice Modem**
<!-- /nav:top -->

# Jaguar Voice Modem

A high-performance (v32terbo) DSP-based modem for the Atari Jaguar that carries simultaneous compressed voice and game data over a single telephone line, enabling head-to-head play with live voice chat.

> **Source:** *The Jaguar Voice Modem* (scanned), © Atari Corp. Reproduced for developer reference.

> **Note from the manual:** The Voice Modem section of the original documentation was still undergoing significant revisions. This page reproduces the 26 April 1995 draft, which covers the *voice-plus-data* command set. A full reference manual of all commands existed but was not complete at the time; the full command set is needed only for full-featured fax/data communication systems (without simultaneous voice).

## Introduction

The Jaguar Voice Modem is a high-performance (v32terbo) DSP-based modem with many additional features and modes that make it particularly suitable for an interactive and consumer-friendly game environment.

This reference covers:

- The Modem Interface
- Data Communications and Bandwidth
- Flow Control (Call, Hang Up, Answer Sequence, Parsing Received Data, Call Waiting)
- A reference of the commands and unsolicited responses used in **voice-plus-data** mode

## Modem Interface

The interface between the Jaguar and the modem is via the built-in Jaguar UART. Communications in both directions are in the form of 2- or 3-byte packets, at a baud rate of 57600 or 19200 (1 start bit, no parity, 2 stop bits).

After reset, all communications are initiated by the Jaguar. Typically the Jaguar sends a command to the modem and the modem responds. In simultaneous voice-plus-data applications the baud rate between the Jaguar and modem is usually reduced, in order to ease the interrupt-response requirements.

The modem can also enable various types of *"unsolicited"* data packets back to the Jaguar. In this case the modem may send a data/command packet to the Jaguar unsolicited. These packets are typically used for incoming data, call-waiting detection, loss of the line, and other errors.

- **Commands from the Jaguar to the modem** are always sent as a two-byte packet, with the **least significant byte sent first**.
- **Replies from the modem to the Jaguar** are sent as two-byte packets, with the **most significant byte (usually the command byte) first**.
- The modem also sends a padding byte of `$FF` prior to a packet if there was a significant gap since the previous packet.

The parse-data flow diagram (see *Parsing the Received Data*) shows how to handle received data.

## Data Communications and Bandwidth

In voice-plus-data mode — known hereafter as **SVD** (Simultaneous Voice and Data) — compressed voice data is sent over the telephone line in packets which have a one-byte header. Game data packets can be inserted into this data stream at any time with a one-byte overhead. The game data packets actually interrupt the voice data stream to keep transport latency to an absolute minimum (necessary for good interactivity).

Developers need to understand the data bandwidth that is available, then decide which packet sizes are most appropriate for their game. The following equations describe the available bandwidth:

```
Total data bandwidth = Line Speed / 8   (in bytes per second)
```

Modem data is sent with an embedded clock, so there is no need for start or stop bits.

```
Voice data bandwidth = (Voice sampling frequency / 4)
                     + (Voice sampling frequency / (4 * Voice packet size))
```

This gives the voice data bandwidth in bytes per second, and shows that each voice sample uses 2 data bits, or 4 samples per byte, plus one voice packet header byte.

```
Game data bandwidth = (number of game data packets per second)
                    * (game data packet size + x)

(x = 1 in normal mode, 2 in error detection mode)
```

The following table shows the voice sampling rates the modem uses by default (assuming 80-byte voice packets, and the default adaptive voice sampling rates):

| Line Speed | Total Bytes/Sec | Voice Sample Rate | Voice Data Rate | Voice Packet Size | Voice Headers | Voice Bytes/Sec | Remaining Bytes/Sec |
|------------|-----------------|-------------------|-----------------|-------------------|---------------|-----------------|---------------------|
| 19200 | 2400 | 7200 | 1800 | 80 | 22.5  | 1822.5 | 577.5 |
| 16800 | 2100 | 6800 | 1700 | 80 | 21.25 | 1721.25 | 378.75 |
| 14400 | 1800 | 5600 | 1400 | 80 | 17.5  | 1417.5 | 382.5 |
| 12000 | 1500 | 4400 | 1100 | 80 | 13.75 | 1113.75 | 386.25 |
| 9600  | 1200 | 3200 | 800  | 80 | 10    | 810    | 390 |

The remaining bandwidth is available for data packets. Game data packets have a one-byte overhead each, plus an additional overhead byte for error detection. **You MUST use a form of error detection**, since errors do occur over the line. Error correction is usually achieved by requesting that the packet be present (re-sent). Assuming a worst-case data rate of 378 bytes per second, the following data-packet options are possible:

| Total Data Rate (Bytes/Sec) | Data Packet Size | Packet Overhead | Total Packet Size | Packets Per Second | Total Data (Bytes/Sec) |
|-----------------------------|------------------|-----------------|-------------------|--------------------|------------------------|
| 378 | 1  | 2 | 3  | 126.00 | 126.00 |
| 378 | 2  | 2 | 4  | 94.50  | 189.00 |
| 378 | 4  | 2 | 6  | 63.00  | 252.00 |
| 378 | 6  | 2 | 8  | 47.25  | 283.00 |
| 378 | 10 | 2 | 12 | 31.50  | 315.00 |
| 378 | 20 | 2 | 22 | 17.18  | 343.64 |
| 378 | 40 | 2 | 42 | 9.00   | 360.00 |

As the table shows, the smaller the data packet size, the less efficient this method is in terms of total bytes per second. However, the smaller packets provide a higher packets-per-second rate, which increases interactivity.

## Flow Control

Example code is provided for initialization and overall flow control, and Atari suggests everyone use it. Once the two modems have completed *"handshaking"*, the users can talk over their headsets while the Jaguars send each other data packets.

The game will need a *"Modem"* option-selection screen. This allows selection of any of the following:

1. **Call.** Brings up an edit field to enter the number to dial. When entered and OK selected, the modem goes off hook and dials the number. The user hears the dialing via the headset. If the line is answered, she can talk to the answerer via the headset. If there is no answer, she can select *"Hang Up"*.
2. **Hang up.** This does a graceful cleardown (i.e. causes both ends to hang up together). If the modem was communicating digitally with the other end, and if the modem was still in analog mode, it simply hangs up the line.
3. **Answer.** This is the selection used by the answerer after the two parties have verbally agreed over the analog line to play the game. This selection mutes the headsets and commences handshaking.
4. **Adjust voice volume.**

An outline of the modem commands used for each of the four options is given below; example code is also available, and a flow chart is included. Details of each command are given in the *Command Reference*.

### Call

Flow (per the manual's flowchart):

```
Prompt user for number to dial
  -> Initialize modem as caller
  -> Go off hook, wait for dial tone --(No dial tone)--> Report "No dial tone"
  -> Dial number
  -> Offer "Hang up" option to user
     -> Hangup requested? --Yes--> Go on hook --> Main Menu
     -> Tone detected?
     -> Magic DTMF sequence?
     -> Send DTMF reply sequence  (-> A)
A -> Report "Handshake in progress"
  -> Wait up to 10 seconds for handshaking
  -> Timeout or Fail? --Yes--> Report "Line Error"
  -> Less than 9600 bps? --Yes--> Report "Line not good enough for Voice plus Data"
  -> Report connection rate -> Start Game
  (Error paths -> Go On Hook -> Main Menu)
```

Command sequence for the **Call** flow:

| Command | Response | Description |
|---------|----------|-------------|
| `$FFFF` | `$B80x` | Reset modem and do a self test |
| `$FFFE` | none   | Set baud rate to 19200 |
| `$000F` | `$000F` | Enable echo back of commands |
| `$B000` | `$B000` | Enable Analog Line to Headset connection |
| `$2C80` | `$2C80` | Set this modem up as a Caller, and enable call-waiting detection |
| `$3962` | `$3962` | Set miscellaneous configuration items |
| `$A021` | `$A021` | Set target error rate to better than 1 in 10e6 bits (i.e. minimum) |
| `$F207` | `$F207` | Enable unsolicited error detection codes |
| `$B602` | `$B602` | Enable error detection mode |
| `$B5xx` | `$B5xx` | Set data packet size |
| `$B405` | `$B405` | Set voice packet size to 80 bytes |
| `$A37E` | `$A37E` | Enable unsolicited line status |
| `$A060` | `$A060` | Go off hook |
| `$8C01` | `$8C0x` | Wait for dial tone (4-second timeout if necessary) |
| `$8A2x` | `$8A2x` | Dial number (repeat command for each digit in phone number) |
| `$6800` | `$000x` | Poll DTMF tone detector at the end of the dial-tone sequence |
| —       | `$FFFE` | If no tone detected, the Caller will never see the "Handshake in progress" status, but the users will still be able to talk and discuss the problem over the analog line. |
| `$8A2x` | `$8A2x` | Send magic DTMF reply sequence (when magic tone is detected) |
| `$2C80` | `$2C80` | Display "Handshake in progress" status message |
| `$8000` | `$8000` | Set me up as the caller |
| `$8100` | `$8xyz` | Poll for handshake successful (timeout after 15 seconds) |

### Hang Up

| Command | Response | Description |
|---------|----------|-------------|
| `$9000` | `$9000` | Graceful cleardown and hang up in data mode |
| `$A040` | `$A040` | Just hang up in analog mode |

### Answer Sequence

Flow (per the manual's flowchart):

```
Go off hook
  -> Prompt user to "Hang up any other lines": OK, or QUIT
  -> User selected "QUIT"? --Yes--> Hang up, Go to Main Menu
  -> User selected "OK"? --Yes--> Send Magic DTMF sequence
  -> DTMF reply within 4 seconds? --No--> Report "No Response from caller -
       check connections" (OK or QUIT)
  -> Yes -> (B)
B -> Report "Handshake in progress"
  -> Wait up to 15 seconds for handshaking
  -> Timeout or Fail? --Yes--> Report "Line Error"
  -> Less than 9600 bps? --Yes--> Report "Line not good enough for voice plus data"
  -> Report connection rate -> Start Game
  (Error paths -> Go On Hook -> Main Menu)
```

Command sequence for the **Answer** flow:

| Command | Response | Description |
|---------|----------|-------------|
| `$FFFF` | `$B80x` | Reset modem and do a self test |
| `$FFFE` | none   | Set baud rate to 19200 |
| `$000F` | `$000F` | Enable echo back of commands |
| `$B000` | `$B000` | Enable Analog Line to Headset connection |
| `$2480` | `$2480` | Set this modem up as an answerer, and enable call-waiting detection |
| `$3962` | `$3962` | Set miscellaneous configuration items |
| `$A021` | `$A021` | Set target error rate to better than 1 in 10e6 bits (i.e. minimum) |
| `$F207` | `$F207` | Enable unsolicited error detection codes |
| `$B602` | `$B602` | Enable error detection mode |
| `$B5xx` | `$B5xx` | Set data packet size |
| `$B405` | `$B405` | Set voice packet size to 80 bytes |
| `$A37E` | `$A37E` | Enable unsolicited line status |
| `$A060` | `$A060` | Go off hook |
| —       | —       | Prompt user "Hang up any other hand sets" |
| —       | —       | Wait for user to acknowledge other lines are hung up |
| `$8A2x` | `$8A2x` | Send magic DTMF sequence |
| `$6800` | `$000x` | Poll DTMF tone detector for magic DTMF reply sequence |
| —       | `$FFFE` | (no tone detected). Timeout after 4 seconds. If timeout, prompt user "No response from caller modem. Check modem connections" |
| `$2480` | `$2480` | When magic reply detected, send "Send magic DTMF sequence". Display "Handshake in progress" status message |
| `$8000` | `$8000` | Set handshake |
| `$8100` | `$8xyz` | Poll for handshake successful (timeout after 15 seconds) |

### Parsing the Received Data

Once the modem has been initialized and handshaking has occurred, data transmissions are possible. The flow chart for received data is given below.

```
Main data Parse loop
  -> 2 Bytes ready? --No--> Exit
  -> $F0xx? --Yes--> Put byte xx in packet buffer
  -> $FF?
       Yes --> Discard byte
       No  --> Mark packet as "Good", point to next packet buffer
  -> $F3xx? --Yes--> $F301? --Yes--> Discard packet, transmit "Resend" command
  -> $B1FF? --Yes--> Pause game --> Report "Call Waiting" --> (C)
  -> $A4xx? --Yes--> $A4x1? --Yes--> Report "Line Lost - possibly call waiting
       at other end" --> Go to D in "Call"
                    --No --> Discard Byte Pair
            --No --> Modem Error
```

### Call Waiting

The line which gets a call-waiting tone will receive the unsolicited data packets `$B1FF` then `$A4??`. The other line will get a `$A4??` packet. Both ends will then immediately go into analog line mode, which will allow them to talk, and for the call-waiting receiver to ask the other party to wait while she picks up the call waiting. She then selects the *"go to call waiting"* box, which flashes the line for her, has the conversation, then selects *"reconnect"*, which will flash the line again (back to the first party), and send the magic DTMF tone sequence — starting handshake again.

Flow (per the manual's flowchart):

```
Offer Menu: "Flash to other line", "Hang up", "Restart game"
  -> Flash to other line? --Yes--> Flash Line
  -> Hang up? --Yes--> Go On Hook --> Main Menu
  -> Restart Game? --Yes--> Go to E in "Answer"
```

When a call-waiting tone is detected by the remote modem, the local modem will just get the lost-line response (`$A4xx`) on its own. Both ends will in fact switch to analog mode, allowing the users to talk, take care of the call waiting, and then restart communications and handshaking.

## Command Reference for Voice-Plus-Data

Unless otherwise noted, all command and response values are in hexadecimal.

### Initialize/Report Software Reset — `0xFFFF`

**Function:** This command causes the Voice Modem to reset all parameters to the default conditions. After resetting, the Voice Modem will return self-test results executed during the previous POR (Power-On Reset). This command may be issued at any time. **CAUTION: care should be taken because the command will clear all operating parameters to the default values.**

The modem will internally issue the following commands during reset:

| Command Name | Value |
|--------------|-------|
| HALF MODE ON | `0xA031` |
| HOST ECHO ON | `0x000F` |
| AUDIO LOW | `0x0102` |
| Set Configuration Word 1 | `0x3962` |
| Set Configuration Word 2 | `0x3962` |
| Enable Unsolicited Error Detection Responses | `0xF207` |
| Set Bit Error Rate Target | `0xA3FF` |
| V24 MASK | `0x8602` |
| Connect Headset to Analog Line | `0xB000` |

> Several of the fine hex values in the reset table above are at the resolution limit of the scan; the `AUDIO LOW`, the two `Set Configuration Word` values, `V24 MASK`, and `Set Bit Error Rate Target` cells should be treated as low-confidence. (illegible in places)

Since it is not always possible to determine whether the modem lost baud rate is set to 57600 or 19200, the following procedure is recommended for issuing the reset command:

- Send Reset command at 57600.
- If a successful response (`0xB800`) is received within 1 second, then exit reset.
- If a successful response is not received within 1 second, issue the reset command at 19200 and ignore the response (if any).
- Then issue a reset command again at 57600, and wait for the response.

**Response:** The response is returned at a host baud rate of 57600, after the reset is completed and within about 1 second. It is in the form `0xB80x` where x has the bit form:

```
[DSP] [AFE] [ROM] [SRAM]
```

where 0 is a pass and 1 is a fail. Thus a successful self-test will give a response of `0xB800`.

**Default:** N/A

### Change Host Baud Rate to 19200 — `0xFFFE`

**Function:** Set host baud rate to 19200. Only reset (`0xFFFF`) can change the baud rate back to 57600.

**Response:** none

**Default:** N/A

### Connect Headset to Analog Line — `0xB000`

**Function:** Allows the headset to be used as a telephone handset, as if it were directly connected to the analog line. (In reality, a digital connection is made between the line Codec and the headset Codec.)

This command also causes the modem to switch to SVD mode immediately after handshaking is complete.

**Response:** The command will be echoed back within 1.2 ms.

### Set Configuration Word 1 — `0x2nnn`

**Function:** This command writes 12 bits, specified by `nnn`, to the modem Configuration Word 1. Bits 0–5 specify the modem type, and bits 6–11 specify other modem configuration items. The meaning and function of these bits are described below.

Bit definitions:

- **Bit 11 — Answer/Call:** selects the answer mode or call mode handshake sequence for the modem type selected. Should only be changed when the modem is off-line.
- **Bit 10 — Accept/Reject Remote Loop Request:** allows or disallows response to remote digital loopback when requested by the far-end modem. Valid for V.32terbo/V.32bis, V.32, V.22bis, V.22, and Bell 212 modem types.
- **Bits 9–8 — Tx Clock:** selects the source of the transmit bit timing, either locked to the external clock XTCLK, internal on-board crystal, or locked to the received clock RDCLK derived from the far-end modem signal.
- **Bit 7 — Enable call-waiting detection.**
- **Bit 6 — Reserved:** reserved for future use and should be set to 0.
- **Bits 5–0 — Modem Type:** these 6 bits select the modem type desired. When selecting a V.32terbo/V.32bis/V.32 configuration, the desired rates should be defined using the Set Rate Sequence Command `1NNN`. The combinations of these two commands would have the effect of either setting a single speed, negotiating with a restricted set of speeds, or allowing all possible speeds. When using a reset command, the highest rate enabled is used.

Modem type selection (bits 5–0):

| Modem Type | Data Rate (bit/s) | Modulation |
|------------|-------------------|------------|
| V.27ter   | 2400  | DPSK     |
| V.17      | 14400 | QAM-TCM  |
| V.17      | 12000 | QAM-TCM  |
| V.17      | 9600  | QAM-TCM  |
| V.17      | 7200  | QAM-TCM  |
| V.21 Ch2  | 300   | FSK      |
| Voice Mode| 14400 | ADPCM    |

> The per-modem-type 6-bit patterns in the scanned table are too small to transcribe reliably. (illegible)

**Response:** The command is echoed back within 1.2 ms after it is written.

**Default:** `2480` hex

### Set Configuration Word 2 — `0x3nnn`

**Function:** This command writes 12 bits, specified by `nnn`, to the modem Configuration Word 2. The meaning and function of these bits are described below.

Configuration Word 2 controls (per the scanned bit table):

| Function | Bit |
|----------|-----|
| Enable/Disable Answer Tone | 11 |
| No Tones | 10 |
| 550 Hz Guard Tone ON | 9 |
| 1800 Hz Guard Tone ON | (see bits 10–9) |
| Echo Protection Tone ON | 8 |
| Enable Auto-mode | 8 |
| Dial-up/Leased-Line | 7 |
| Enable/Disable Auto-retrain/Rate Renegotiation | (Auto-retrain / Auto-rate) |
| Asynchronous Normal | 5–4 |
| Asynchronous Extended/HDLC | 5–4 |
| Reserved | 3–2 |

Bit definitions:

- **Bit 11 — Enable/Disable Answer Tone:** function depends on the state of Bit 11 of Configuration Word 1. When answer mode handshake is selected (Configuration Word 1, Bit 11 = 0), clearing this bit enables the transmission of 3600 ms of 2100 Hz tone prior to beginning the appropriate handshake sequence according to V.25 recommendation. Setting this bit to one causes no 2100 Hz tone to be transmitted prior to the handshake sequence. When an originate-mode handshake is selected (Configuration Word 1, Bit 11 = 1) this bit has no effect. This bit is not used with Bell 103 or Bell 212A modem types and will have no effect if these modem types are selected. This bit may be changed at any time.
- **Bits 10–9 — Tones Selection:** these two bits allow the generation of 550 and 1800 Hz guard tones for V.33, V.17, V.29 and V.27ter half-duplex modes. For other modem types, no tone (00) should be selected. These bits should only be changed when the modem is off line.
- **Bit 8 — Enable/Disable Auto-mode:** this feature supports Annex A of V.32terbo/V.32bis/V.32 CCITT recommendations and EIA PN-2330 (draft proposal) for automode handshake which allows the Voice Modem to automatically determine the mode of the far-end modem during handshake and to reconfigure itself appropriately. This feature works if the far-end modem is a V.32terbo/V.32bis/V.32, V.22bis, V.22, V.21, V.23, Bell 212A or Bell 103.
- **Bit 7 — Dial-up/Leased-Line:** modifies the handshake from normal dial-up to a specified leased-line sequence if applicable.
- **Bit 6 — Enable/Disable Auto-retrain and Auto-rate Renegotiation:** if this feature is enabled, the Voice Modem will initiate a retrain or a rate renegotiation if the actual mean square error (MSE), which represents signal quality, is higher (or lower than a dynamically set threshold). For a more detailed explanation refer to Section 8.2.
- **Bits 5–4 — Async/Sync Select:** these bits function in conjunction with Configuration Word 2, bit 1 as follows: if Configuration Word 2, bit 1 = 0 (serial data), then async mode is selected with bit 5 = 0. Bit 1 allows the choice of normal operation in the +1.0% to -2.5% rate range or continuous, extended operation in the +2.3% to -2.5% rate range according to V.14 recommendations. However, if bit 1 = 1 (i.e. parallel data), then bit 4 = 1 configures the data interface for HDLC operation and bit 4 = 0 for asynchronous (8,N,1) operation as described in the parallel data mode section. Synchronous operation, either in serial or parallel data modes, is selected by setting bit 4 = 1, bit 5 = 1.

Async/Sync function table:

| Bit 5 | Bit 4 | Function |
|-------|-------|----------|
| 0 | 0 | Serial Async Normal Rate |
| 0 | 1 | Serial Async Extended Rate |
| 1 | 0 | Reserved |
| 1 | 1 | Serial Synchronous |
| —     | —    | Parallel Async 10-bit Character |
| —     | —    | Parallel Sync w/HDLC |
| —     | —    | Parallel Sync w/Bit Stream |

(Serial V.14 Character Length, Serial/Parallel Data Mode, and Enable Adaptive MSE/RLSD Thresholds are also represented in the full bit table.)

- **Bits 3–2 — Character Length:** these bits are used to select the correct character length for the Serial V.14 asynchronous converter. They are only used when the modem is operating in asynchronous serial mode (Configuration Word 2, bit 5 = 0, bit 1 = 0). The presence of bit 7 includes one start bit and one stop bit. This automatically requires a character length of 10 bits (10). In asynchronous parallel mode (Configuration Word 2, bit 5 = 0, bit 4 = 0, bit 1 = 1), the character length is always 10 bits.
- **Bit 1 — Serial/Parallel Data Mode:** configures the Voice Modem to pass data serially through the V.24 Pins RXD, TXD or in bytes through the parallel data interface. Set bit 1 = 0 in conjunction with Word 2, bit 4 and bit 5. *Note: Serial mode is not available in "V.32terbo" 19,200 bit/s mode.*
- **Bit 0 — Enable/Disable Adaptive RLSD Detection:** this bit enables or disables the adaptive determination of RLSD thresholds to permit fast and consistent RLSD (loss) detection. For a more detailed explanation refer to Section 8.2.

**Response:** The command is echoed back within 1.2 ms.

**Default:** `3000` hex

### Set Bit Error Rate Target — `0xA02n`

**Function:** This command sets the BER target for the auto-speed selection feature. This feature enables the Voice Modem to automatically select the highest data rate allowable by the modems and supported by the line conditions such that BER does not exceed the target value. The command variable `n` assumes the following values:

| n | Meaning |
|---|---------|
| 0 | Disabled |
| 1 | BER = 10E-6 |
| 2 | BER = 10E-5 |
| 3 | BER = 10E-4 |
| 4 | BER = 10E-3 |

**Response:** The command is echoed back within 1.2 ms.

**Default:** `A021` hex

### Enable Unsolicited Error Detection Responses — `0xF207`

**Function:** This command causes the modem to return the `0xF3xx` error-check responses (if enabled) at the end of data packets.

**Response:** The command is echoed back within 1.2 ms.

### Enable Error Detection Mode — `0xB60n`

**Function:** Selects data modes:

| Value of n | Meaning |
|------------|---------|
| 0 | non real-time data |
| 1 | real-time data, without error detection |
| 2 | real-time data, with error detection |

**Response:** The command is echoed back within 1.2 ms.

### Set Data Packet Size — `0xB5xx`

**Function:** Set real-time data packet size to `xx` bytes.

**Response:** The command will be echoed back within 1.2 ms.

**Default:** `0xB504`

### Enable Unsolicited Line Status — `0xA3FE`

**Function:** Enable the unsolicited responses `0xA4xx` (see the unsolicited response section below).

**Response:** The command is echoed back within 1.2 ms.

### Report Dial-Tone Detector — `0x8C01`

**Function:** This command is used to detect the presence or absence of dial tone within a very short interval.

If a dial tone was detected, the response will be `0x8C0x` where `xx` is not 01. If a dial tone was not detected, the response will be `0x8Cxx` where `xx` is not 01.

**Response:** A response is returned within 1.2 ms after it was issued.

### Set Voice Sampling Frequency — `0xB30x`

**Function:** Set the compressed voice sampling frequency, as shown below:

| Sample Rate | x |
|-------------|---|
| Adaptive sampling (Default) | 0 |
| 3200 Hz | 1 |
| 3600 Hz | 2 |
| 4400 Hz | 3 |
| 4400 Hz | 4 |
| 4800 Hz | 5 |
| 5200 Hz | 6 |
| 5600 Hz | 7 |
| 6000 Hz | 8 |
| 6400 Hz | 9 |
| 6800 Hz | A |
| 7200 Hz | B |

The default adaptive sampling rates are as follows:

| Connection Speed | Sampling Rate |
|------------------|---------------|
| 19200 bps | 7200 Hz |
| 16800 bps | 6800 Hz |
| 14400 bps | 4400 Hz |
| 12000 bps | 4400 Hz |
| 9600 bps  | 3200 Hz |

**Response:** The command will be echoed back within 1.2 ms.

### Dial Number / Transmit DTMF Tone — `0x8A2x`

**Function:** This command is used to dial a digit based on the mode selected using the Set Dial Mode command. The command is of the form `8A2x` hex, where `x` denotes the digit to be dialed. The status of digit dialing can be known using the **Report Call Progress Detector** command.

```
x      = 0 1 2 3 4 5 6 7 8 9
Number = 0 1 2 3 4 5 6 7 8 9 * # A B C D
```

**Response:** The command is echoed back within 1.2 ms.

### Poll DTMF Detector — `0x6800`

**Function:** This command starts the DTMF tone detector and returns the status of the detector with a response of `000x` hex. The least significant digit of the response reports the DTMF tone code that was received as follows:

```
x            = 0 1 2 3 4 5 6 7 8 9 A B C D E F
DTMF Tone Pair = 0 1 2 3 4 5 6 7 8 9 * # A B C D
```

If no digit is detected, the response of `FFFE` hex is returned. The digit detected is held until it is read by the controller or another digit is detected.

**Response:** A response is returned within 1.2 ms after it was written.

### Report Handshake Status — `0x8100`

**Function:** This command causes the Voice Modem to return a 12-bit response indicating the progress through the handshake, retrain, or rate negotiation.

**Response:** The response is returned in the form `8xyz` hex, where x, y, and z are shown below.

Examples:

```
V.32bis handshake completed at 14.4k bits:   0x80B2
V.32bis handshaker before rate determined:   0x80C0   (handshake before rate determined)
Auto-moding, no mode or rate is determined:   0x8000
```

Handshake/Retrain State (`y`):

| State | y |
|-------|---|
| Undetermined | 0 |
| 1200/75 | 1 |
| 75/1200 | 2 |
| 0-300 | 3 |
| 2400 | 4 |
| 4800 | 5 |
| 7200 | 6 |
| 9600 Non-trellis | 7 |
| 9600 | 8 |
| 12000 | 9 |
| 14400 | A |
| 16800 | B |
| 19200 | C |

Data Rate Response (`x`):

| State | x |
|-------|---|
| V.32 | 1 |
| V.32terbo/V.32bis | 2 |
| V.22bis | 3 |
| V.22 | 4 |
| Bell 212 | 5 |
| V.23 | 6 |
| V.21 | 7 |
| Bell 103 | 8 |
| V.29 | 9 |
| V.27 | A |
| V.26 | B |
| V.17 | C |

State (`z`):

| State | z |
|-------|---|
| Auto-mode Handshake in Progress | 0 |
| Non-Automode Handshake in Progress | 1 |
| Abort/Idle | 4 |
| Handshake in Progress | 5 |
| Rate Renegotiation in Progress | 5 (illegible — possibly 6) |
| Data Mode | 6 |

**Response:** The response is returned within 1.2 ms after the command is written.

### Set Voice Volume — `0xBA0x`

**Function:** Adjust voice volume. The allowable values for `x` are:

| Level | x |
|-------|---|
| Maximum volume (default) | 0 |
| -3 db | 1 |
| -6 db | 2 |
| -3 x db | x |
| Mute | F |

**Response:** The command will be echoed back within 1.2 ms.

### Send Real-Time Data — `0xF0xx`

**Function:** Send data byte `xx` in real-time (low-latency) mode.

The data byte `xx` will be sent once the controller has received a full packet of bytes (packet size is set by the `B5xx` command). The typical latency is around 18 ms.

**Response:** The command will be echoed back within 1.2 ms.

## Unsolicited Response Reference

This section summarizes the various types of unsolicited data that can be expected from the modem.

### Receive Real-Time Data — `0xF0xx`

**Function:** The byte `xx` was received from the remote modem.

If error detection has been enabled, note that the packet error status will only be received at the end of the packet (after all packet bytes have been received).

### Packet Error Status — `0xF3xx`

**Function:** If error detection has been enabled (using the `$B602` command), this response will be received after all bytes in a packet have been received. The format is:

| Response | Meaning |
|----------|---------|
| `$F301` | No Errors in packet |
| `$F311` | Error occurred in packet |

### Call Waiting Detected — `0xB1FF`

**Function:** When call-waiting detection has been enabled with the `$2C80` or `$2480` command, this response indicates that a call-waiting tone has been detected.

This response will be followed by a `$A4??` response, indicating that the line has been lost (see below).

### Line Lost — `0xA4xx`

**Function:** This unsolicited response type is enabled with the command `0xA3FE`. When enabled, the modem will report line lost, and occasionally also report that line lost is still good. As shown in the parse data flow chart, the line-good response needs to be taken into account, and discarded. The least significant bit of the response indicates the line status:

```
%xxxx xxx1 = Line Lost
%xxxx xxx0 = Line Good
```

Only the LSB is valid; all other bits should be ignored.

Immediately subsequent to losing the line, the modem will switch back to analog mode, where the headset and microphone are connected to the analog line.

When a call-waiting tone is detected by the remote modem, the local modem will just get this lost-line response on its own. Both ends will in fact switch to analog mode, allowing the users to talk, take care of the call waiting, and then restart communications and handshaking.

## See also

- [Serial I/O — ComLynx, MIDI](../jerry/serial-io.md)
- [Audio Subsystem & Synthesis](../jerry/audio.md)
- [Memory Map / Register List](../architecture/memory-map.md)
- [System Architecture Overview](../architecture/overview.md)

<!-- nav:bottom -->
---

◀ **Prev:** [Controllers & Controller Ports](../controllers/controllers.md) &nbsp;·&nbsp; 🏠 **[Home](../index.md)** &nbsp;·&nbsp; **Next:** [CD-ROM Subsystem Overview](../cdrom/overview.md) ▶

**Jump to:** [Architecture](../architecture/overview.md) · [Memory Map](../architecture/memory-map.md) · [Registers](../reference/register-list.md) · [Instructions](../reference/risc-instruction-set.md) · [Glossary](../reference/glossary.md) · [CD-ROM](../cdrom/overview.md)
<!-- /nav:bottom -->
