# Roaster Controller — External Control Protocol

**Document version:** 1.2
**Applies to firmware:** V1.2.0+

---

## Table of Contents

1. [Overview](#1-overview)
2. [Communication Links](#2-communication-links)
3. [Frame Format](#3-frame-format)
4. [Response Format](#4-response-format)
5. [Communication Rules](#5-communication-rules)
6. [Command Availability by Edition](#6-command-availability-by-edition)
7. [Command Reference](#7-command-reference)
8. [Typical Session](#8-typical-session)
9. [Implementation Checklist for Host Software](#9-implementation-checklist-for-host-software)
10. [Appendix — Command Summary](#appendix--command-summary)

---

## 1. Overview

The roaster exposes a plain-text, request/response command protocol. The host sends a **command frame**; the device executes the command(s) it contains and replies with exactly one **response frame**.

Key characteristics:

- ASCII text only (printable characters `0x20`–`0x7E`).
- Each request frame may carry one or several commands separated by commas.
- Responses are `KEY:VALUE` pairs, compatible with the data format used by Artisan.
- The device only answers requests; it does not send unsolicited command responses.
- The same frame format and command set are used on every link (serial port, Bluetooth SPP, WebSocket, Bluetooth LE).

---

## 2. Communication Links

Four links are available. Each link has its own independent receive buffer and command pipeline, so they can be used at the same time without interfering with each other.

| Link                 | Transport            | Max payload | Max response fields | Response terminator |
|----------------------|----------------------|:-----------:|:-------------------:|---------------------|
| Serial port (UART)   | Wired, 57600 8N1     | 62 chars    | 121 chars           | `\r\n`              |
| Bluetooth SPP        | Bluetooth Classic    | 62 chars    | 121 chars           | `\r\n`              |
| WebSocket            | Wi-Fi, TCP port 80   | 62 chars    | 121 chars           | None (one message)  |
| Bluetooth LE         | GATT write / notify  | **30 chars**| **89 chars**        | None (one notification) |

> Bluetooth SPP and Bluetooth LE are mutually exclusive: the device boots into one of the two modes (see [`DEV BOOT`](#716-dev--device-management)).

### 2.1 Wired serial port (UART)

| Parameter    | Value      |
|--------------|------------|
| Baud rate    | **57600**  |
| Data bits    | 8          |
| Parity       | None       |
| Stop bits    | 1          |
| Flow control | None       |

Configuration string: **57600 8N1**, no flow control.

### 2.2 Bluetooth Classic (SPP)

| Parameter      | Value                                                                |
|----------------|----------------------------------------------------------------------|
| Profile        | Serial Port Profile (SPP)                                            |
| Device name    | Model-specific (shown in the host's Bluetooth device list)           |
| Baud rate      | Not applicable (the virtual COM port accepts any setting; use 57600) |

The device must be booted in **Bluetooth Classic (BT)** mode for SPP to be available. See [`DEV BOOT`](#716-dev--device-management).

### 2.3 WebSocket (Wi-Fi)

| Parameter        | Value                                         |
|------------------|-----------------------------------------------|
| URL              | `ws://<device-ip>:80/ws`                      |
| Port             | 80                                            |
| Path             | `/ws`                                         |
| Encryption       | None (`ws://`, not `wss://`)                  |
| Message type     | **Text** frames (binary frames are ignored)   |

**Availability.** The WebSocket server starts automatically once the device has joined a Wi-Fi network. The device's IP address can be found on the device screen or in your router's client list. Wi-Fi setup itself is done from the device or the official app and is outside the scope of this document.

**Client roles.**

| Role                | How it is assigned                          | Can send commands | Receives responses |
|---------------------|---------------------------------------------|:-----------------:|:------------------:|
| **Controller**      | The first client to connect                 | ✔                 | ✔                  |
| **Observer**        | The second client to connect                | ✘ (ignored)       | ✔                  |
| —                   | Any further client                          | Connection is closed by the device | — |

- At most two clients may be connected at once: one controller and one observer.
- Responses are **broadcast** to all connected clients, so an observer sees the controller's responses.
- If the controller disconnects, the device closes **all** WebSocket connections. The next client to connect becomes the new controller.

**Message mapping.**

- Put one or more complete request frames in a text message. Sending exactly one frame per message is recommended.
- Each response is sent as a separate text message. It contains the response frame only, **without** the `\r\n` line terminator used on the serial links.

### 2.4 Bluetooth LE (direct connection)

The device must be booted in **Bluetooth LE (BLE)** mode. See [`DEV BOOT`](#716-dev--device-management).

**Advertising**

| Item                     | Value                                                         |
|--------------------------|---------------------------------------------------------------|
| Device name              | `KLD_` followed by the last characters of the serial number   |
| Advertised service UUID  | `712eefe8-cafc-4bfb-ad34-ba1a079ec7b1`                        |

Filter scan results by the advertised service UUID rather than by name.

**GATT characteristics used by this protocol** (all in service `712eefe8-cafc-4bfb-ad34-ba1a079ec7b1`)

| Purpose        | Characteristic UUID                      | Properties | Direction       |
|----------------|------------------------------------------|------------|-----------------|
| Command        | `11060000-317b-4b94-9513-fd87be4801d3`   | Write (with response) | Host → Device |
| Response       | `11050000-317b-4b94-9513-fd87be4801d3`   | Notify     | Device → Host   |

The service also contains other characteristics used by the official app for network provisioning. **Do not write to them**: once provisioning data has been written, the command characteristic is used for provisioning instead of commands until the device restarts.

The standard Device Information Service (`0x180A`) is also available and can be read for model (`0x2A24`), serial number (`0x2A25`), firmware (`0x2A26`), hardware (`0x2A27`) and manufacturer (`0x2A29`).

**Connection procedure**

1. Scan for the service UUID above and connect.
2. Request an ATT MTU of **128** bytes. With the default MTU of 23, notifications are limited to 20 bytes and longer responses (e.g. `RD A1`) will be truncated.
3. Enable notifications on the response characteristic (write `0x0001` to its CCCD descriptor `0x2902`).
4. Write request frames as ASCII text to the command characteristic. Send one complete frame per write.
5. Each response arrives as one notification containing the complete response frame, with no `\r\n`.

**Notes**

- Only one BLE central can be connected at a time. After it disconnects, the device advertises again automatically.
- The response characteristic may also carry a plain-text IP address notification during Wi-Fi provisioning. Ignore any notification that does not start with `{`.
- The BLE link has smaller buffers than the other links: payload ≤ **30** characters, response fields ≤ **89** characters, and at most **3** frames queued. Avoid combining several commands in one frame on BLE.

---

## 3. Frame Format

### 3.1 Request frame

```
{[<payload>]}
```

| Field        | Length     | Description                         |
|--------------|------------|-------------------------------------|
| `{[`         | 2 bytes    | Start of frame                      |
| `<payload>`  | 1–62 bytes (BLE: 1–30) | One or more commands (see below) |
| `]}`         | 2 bytes    | End of frame                        |

Examples:

```
{[RD A1]}
{[HP 50,FC 60]}
```

### 3.2 Payload syntax

```
<payload>  ::= <command> { "," <command> }
<command>  ::= <NAME> [ " " <arg> { " " <arg> } ]
```

- **`NAME`** — command mnemonic. **Case-sensitive, always upper-case** (`RD`, not `rd`).
- **Arguments** — separated from the name and from each other by one or more spaces. Argument keywords (e.g. `UP`, `DW`, `A1`, `AR`) are also case-sensitive and upper-case.
- **Multiple commands** — separate with a comma: `{[HP 50,FC 60]}`. They are executed left to right.

### 3.3 Reserved characters

The following must **not** appear inside `<payload>`:

| Sequence | Reason                                        |
|----------|-----------------------------------------------|
| `{[`     | Restarts frame reception                      |
| `]`      | Interpreted as end of payload                 |
| `}`      | Interpreted as end of frame                   |
| `,`      | Command separator (use only between commands) |

The frame must end with `]}` directly; do not put any character between `]` and `}`.

### 3.4 Receiver behaviour

- Any non-printable byte (`< 0x20` or `> 0x7E`), including `CR` and `LF`, is silently discarded. The host may therefore append `\r\n` after a frame if convenient.
- Bytes outside a frame (before `{[`) are ignored.
- If `{[` is received while a frame is in progress, the partial frame is discarded and a new one starts.
- An empty frame `{[]}` is ignored.
- Frames whose payload exceeds the maximum length are discarded without a response.

---

## 4. Response Format

### 4.1 Structure

```
{<sid>[,<KEY>:<VALUE>]...}
```

| Field            | Description                                                                              |
|------------------|------------------------------------------------------------------------------------------|
| `{`              | Start of response                                                                        |
| `<sid>`          | One character, session/event ID. Normally `0`. Treat as opaque; do not use it for error detection. |
| `,KEY:VALUE`     | Zero or more fields produced by the executed commands, in execution order                |
| `}`              | End of response                                                                          |

Line terminator: on the serial port and Bluetooth SPP, every response is followed by `\r\n`. On WebSocket, each response is one text message with no terminator. On Bluetooth LE, each response is one notification with no terminator.

Examples:

```
Request : {[RD A1]}
Response: {0,BT:185.3,ET:221.0,AT:28.5,HP:65,FC:40,RC:60,AH:1,TS:200.0,HS:1}

Request : {[HP 50,FC 60]}
Response: {0,HP:50,FC:60}

Request : {[PI]}
Response: {0}
```

String values (`FW`, `SN` in `VER`) are enclosed in double quotes; numeric values are not.

### 4.2 Success and error indication

The response does **not** carry an explicit status code. Success is indicated by the presence of the field(s) the command is documented to return.

| Situation                                    | Response                                                  |
|----------------------------------------------|-----------------------------------------------------------|
| All commands succeeded                       | All expected fields present                               |
| A command failed (unknown name, bad or missing argument, not permitted in this edition, execution error) | Fields of the commands executed **before** the failing one, then `}`. Commands after it are **not executed**. |
| Single failing command                       | `{0}`                                                     |
| Malformed or over-length frame               | **No response**                                           |

Example:

```
Request : {[HP 50,XX 1,FC 60]}
Response: {0,HP:50}          ← HP executed, XX unknown, FC not executed
```

> **Note:** `PI` succeeds with `{0}` and no fields. For every other command, a bare `{0}` means the command failed.

### 4.3 Maximum response length

The field section of a response (between `<sid>` and `}`) holds at most **121 characters** (**89 characters** on Bluetooth LE). If a multi-command frame would exceed this, the command that overflows fails and the remaining commands are not executed. Keep combined requests short; `RD A1` alone uses about 70 characters.

---

## 5. Communication Rules

| Item                              | Value / Rule                                                                 |
|-----------------------------------|-------------------------------------------------------------------------------|
| Max payload length                | 62 characters (between `{[` and `]}`); 30 characters on Bluetooth LE          |
| Request buffer                    | Up to 8 complete frames (128 bytes total) per link; Bluetooth LE: 3 frames (64 bytes) |
| Execution order                   | Strictly first-in, first-out; one response per accepted frame, in order       |
| Recommended mode                  | Send one frame, wait for its response, then send the next                     |
| Recommended response timeout      | 1 s on serial / SPP; 2 s on WebSocket and Bluetooth LE. On timeout, resend the frame |
| Recommended polling interval      | 0.5 – 1 s for `RD A1`                                                          |
| Temperature unit                  | Celsius by default; switchable with `TU` (setting is shared by all links, not saved across power cycles) |

The serial port and Bluetooth SPP links require PC / Artisan connectivity to be enabled on the device (enabled by default). If it has been disabled from the device's own interface, commands on these two links are ignored.

---

## 6. Command Availability by Edition

The device firmware runs as one of four editions. Each command is only accepted when allowed for the current edition and link; otherwise it fails (`{0}`). The first table below applies to the serial port, Bluetooth SPP and WebSocket links; Bluetooth LE has its own table.

The current edition number is reported in parentheses by `VER FW`, e.g. `FW:"1.2.3-249B-M2(2)"` → edition **2**.

| Edition no. | Edition    |
|-------------|------------|
| 1           | Smart      |
| 2           | PRO        |
| 3           | APP        |
| 4           | Dual       |

| Command group                                              | Smart (1) | PRO (2) | APP (3) | Dual (4) |
|------------------------------------------------------------|:---------:|:-------:|:-------:|:--------:|
| `PI`, `VER`, `DEV`, `PIDS`, `CSQ`                          | ✔         | ✔       | ✔       | ✔        |
| `RD`, `HP`, `FC`, `RC`, `TS`, `TU`, `AH`, `EV`, `CS`, `SC`, `CL`, `HS`, `CD`, `DD`, `PIDFAN` | ✘ | ✔ | ✔ | ✔ |

**Bluetooth LE**

| Command group                                              | Smart (1) | PRO (2) | APP (3) | Dual (4) |
|------------------------------------------------------------|:---------:|:-------:|:-------:|:--------:|
| `CSQ`                                                      | ✔         | ✔       | ✔       | ✔        |
| All other public commands                                  | ✘         | ✘       | ✔       | ✔        |

In other words, full monitoring and control from your own software requires the **PRO**, **APP** or **Dual** edition over the serial port, Bluetooth SPP or WebSocket, and the **APP** or **Dual** edition over Bluetooth LE.

---

## 7. Command Reference

Conventions used below:

- `<n>` — decimal integer. `<t>` — decimal number, may contain a fractional part (e.g. `200` or `200.5`).
- Temperatures are in the unit selected by `TU` (°C by default).
- Percentages are `0`–`100`.

> **Important:** for `HP`, `FC` and `RC`, any argument that is not `UP`, `DW` or a number is interpreted as **0**. Always validate arguments in your host software before sending.

---

### 7.1 `RD` — Read status

Reads the current roaster state.

| Syntax   | Description                         |
|----------|-------------------------------------|
| `RD A0`  | Read status (without heater state)  |
| `RD A1`  | Read status (with heater state) — **recommended** |

**Response fields**

| Key  | Type    | Unit      | Description                                                       |
|------|---------|-----------|-------------------------------------------------------------------|
| `BT` | float, 1 dp | °C / °F | Bean temperature                                             |
| `ET` | float, 1 dp | °C / °F | Environment (exhaust / drum air) temperature                 |
| `AT` | float, 1 dp | °C / °F | Ambient temperature                                          |
| `HP` | int     | %         | Heating power currently applied (manual value in manual mode, PID output in auto mode) |
| `FC` | int     | %         | Exhaust (smoke) fan speed                                         |
| `RC` | int     | %         | Drum speed                                                        |
| `AH` | 0 / 1   | —         | Auto-heat (PID) mode: `1` = auto, `0` = manual                    |
| `TS` | float, 1 dp | °C / °F | Auto-heat target temperature                                 |
| `HS` | 0 / 1   | —         | Heater enabled (`RD A1` only): `1` = on, `0` = off                |

**Example**

```
→ {[RD A1]}
← {0,BT:185.3,ET:221.0,AT:28.5,HP:65,FC:40,RC:60,AH:1,TS:200.0,HS:1}
```

---

### 7.2 `HP` — Heating power

Sets the heating power. **Any `HP` command switches the controller to manual heating mode** (`AH` becomes `0`).

| Syntax     | Description                                                   |
|------------|---------------------------------------------------------------|
| `HP <n>`   | Set power to `<n>` % (values > 100 are clamped to 100)        |
| `HP UP`    | Increase by 5 % (max 100). In auto mode, starts from the current PID output |
| `HP DW`    | Decrease by 5 % (min 0). In auto mode, starts from the current PID output   |

The new power is applied immediately if the heater is enabled (`HS 1`); otherwise it is stored and used when heating is started.

**Response:** `HP:<n>` — the resulting manual power.

```
→ {[HP 50]}
← {0,HP:50}
```

---

### 7.3 `FC` — Exhaust fan speed

| Syntax     | Description                                     |
|------------|-------------------------------------------------|
| `FC <n>`   | Set exhaust fan to `<n>` % (clamped to 100)     |
| `FC UP`    | Increase by 5 % (max 100)                       |
| `FC DW`    | Decrease by 5 % (min 0)                         |

**Response:** `FC:<n>`

```
→ {[FC 60]}
← {0,FC:60}
```

---

### 7.4 `RC` — Drum speed

| Syntax     | Description                                                              |
|------------|--------------------------------------------------------------------------|
| `RC <n>`   | Set drum speed to `<n>` %. Values > 100 → 100; values **1–19 → 0** (minimum running speed is 20 %) |
| `RC UP`    | Increase by 10 % (max 100)                                                |
| `RC DW`    | Decrease by 10 %; if current speed is below 30 %, the drum stops (0)      |

**Response:** `RC:<n>`

```
→ {[RC 70]}
← {0,RC:70}
```

---

### 7.5 `TS` — Auto-heat target temperature

| Syntax   | Description                                                                       |
|----------|-----------------------------------------------------------------------------------|
| `TS <t>` | Set the PID target temperature to `<t>` in the current `TU` unit                   |

- Internally stored as an integer in °C (fractional part is truncated after unit conversion).
- Clamped to the model's maximum set-point limit.
- Valid range: 0 to the model maximum. Do not send negative values.
- Setting the target does **not** enable auto mode; use `AH 1` for that.

**Response:** `TS:<t>` — the effective target, in the current unit.

```
→ {[TS 210]}
← {0,TS:210.0}
```

---

### 7.6 `TU` — Temperature unit

| Syntax | Description                    |
|--------|--------------------------------|
| `TU C` | Report / accept Celsius        |
| `TU F` | Report / accept Fahrenheit     |

Affects `BT`, `ET`, `AT`, `TS` in responses and the argument of `TS`. Default after power-on: `C`.

**Response:** `TU:C` or `TU:F`

---

### 7.7 `AH` — Auto-heat (PID) mode

| Syntax | Description                                                        |
|--------|--------------------------------------------------------------------|
| `AH 1` | Enable auto heating: the PID controls power to reach `TS`          |
| `AH 0` | Disable auto heating: manual power (`HP`) is used                  |

**Response:** `AH:1` or `AH:0`

---

### 7.8 `HS` — Heater on/off

| Syntax | Description          |
|--------|----------------------|
| `HS 1` | Enable the heater    |
| `HS 0` | Disable the heater   |

**Response:** `HS:1` or `HS:0`

---

### 7.9 `CS` — Cooling fan

| Syntax | Description                       |
|--------|-----------------------------------|
| `CS 1` | Turn cooling fan on (100 %)       |
| `CS 0` | Turn cooling fan off              |

**Response:** `CS:1` or `CS:0`

---

### 7.10 `EV` — Roast event

Marks a roast event and, for some events, performs an action.

| Syntax    | Action                                                   | Response |
|-----------|----------------------------------------------------------|----------|
| `EV 8`    | **Drop:** heater off **and** cooling fan on              | `EV:8`   |
| `EV 80`   | Drop without cooling: heater off only                    | `EV:8`   |
| `EV 9`    | **Cool end:** cooling fan off                            | `EV:9`   |
| `EV <n>`  | `<n>` = 0–7: event marker only, no action                | `EV:<n>` |

Any other value fails.

---

### 7.11 `SC` — Start connection

Notifies the device that a host application has connected. Send once after opening the link.

| Syntax  | Description                  |
|---------|------------------------------|
| `SC AR` | Host identifies as Artisan   |
| `SC CP` | Host identifies as Cropster  |

Third-party software may use either identifier (`AR` recommended).

**Response:** `SN:<serial_number>`

```
→ {[SC AR]}
← {0,SN:AB12345678}
```

---

### 7.12 `CL` — Close connection

Notifies the device that the host is disconnecting.

| Syntax  | Description          |
|---------|----------------------|
| `CL AR` | Artisan disconnects  |
| `CL CP` | Cropster disconnects |

**Response:** `SN:<serial_number>`

---

### 7.13 `PI` — Ping

Link test / keep-alive. No arguments.

**Response:** `{0}` (no fields)

```
→ {[PI]}
← {0}
```

---

### 7.14 `CSQ` — Echo

Returns its argument unchanged. Useful for link testing.

| Syntax        | Description                     |
|---------------|---------------------------------|
| `CSQ <text>`  | `<text>`: any printable text without reserved characters |

**Response:** `CSQ:<text>`

```
→ {[CSQ HELLO]}
← {0,CSQ:HELLO}
```

---

### 7.15 `VER` — Version information

| Syntax   | Response                                  | Description                                                 |
|----------|-------------------------------------------|-------------------------------------------------------------|
| `VER FW` | `FW:"<ver>-<compat>-<model>(<edition>)"`  | Firmware version and edition number (see [Section 6](#6-command-availability-by-edition)) |
| `VER SN` | `SN:"<serial_number>"`                    | Device serial number                                        |

```
→ {[VER FW]}
← {0,FW:"1.2.3-249B-M2(2)"}
```

---

### 7.16 `DEV` — Device management

| Syntax           | Description                                                                   | Response |
|------------------|-------------------------------------------------------------------------------|----------|
| `DEV REBOOT`     | Restarts the device after ~1 s                                                | None (the link is lost) |
| `DEV UNP`        | Clears Bluetooth LE pairing; the device enters pairing mode                   | `UNP:1`  |
| `DEV BOOT BLE`   | Select Bluetooth LE mode at next boot                                         | `BLE:1`  |
| `DEV BOOT BT`    | Select Bluetooth Classic (SPP) mode at next boot                              | `BLE:0`  |

`DEV BOOT` is saved permanently and takes effect after a reboot.

> **Caution:** sending `DEV BOOT BLE` over Bluetooth SPP makes the SPP link unavailable after the next reboot, and `DEV BOOT BT` over Bluetooth LE makes the BLE link unavailable after the next reboot.

---

### 7.17 `CD` — Charge door

| Syntax | Description          |
|--------|----------------------|
| `CD 1` | Open charge door     |
| `CD 0` | Close charge door    |

**Response:** `CD:1` or `CD:0`

> Reserved for models with a motorized charge door. On other models the command is acknowledged but has no physical effect.

---

### 7.18 `DD` — Discharge (drop) door

| Syntax | Description              |
|--------|--------------------------|
| `DD 1` | Open discharge door      |
| `DD 0` | Close discharge door     |

**Response:** `DD:1` or `DD:0`

> Only effective on models equipped with a motorized discharge door. Do not use on other models.

---

### 7.19 `PIDS` — Heating PID parameters (advanced)

Sets the parameters of the auto-heat PID controller. Takes effect immediately; **not saved** across power cycles.

```
PIDS <Kp> <Ki> <Kd> <avg> <stab>
```

| Arg      | Type  | Description                                                |
|----------|-------|------------------------------------------------------------|
| `Kp`     | float ≥ 0 | Proportional gain                                      |
| `Ki`     | float ≥ 0 | Integral gain                                          |
| `Kd`     | float ≥ 0 | Derivative gain                                        |
| `avg`    | int 0–255 | Moving-average sample count for the PID input          |
| `stab`   | int 0–255 | Stabilisation count                                    |

Exactly 5 arguments are required.

**Response:** `PIDS:<Kp>,<Ki>,<Kd>,<avg>,<stab>` (gains with 3 decimals)

```
→ {[PIDS 2.5 0.05 10 5 3]}
← {0,PIDS:2.500,0.050,10.000,5,3}
```

> Inappropriate values can cause temperature overshoot. Use with care.

---

### 7.20 `PIDFAN` — Closed-loop exhaust fan (advanced)

Only meaningful on models with an RPM-controlled exhaust fan.

| Syntax                         | Description                                           | Response |
|--------------------------------|-------------------------------------------------------|----------|
| `PIDFAN ST`                    | Read fan status                                       | `PIDFAN:<state>,<targetRPM>,<actualRPM>,<pwm>,<FC>` |
| `PIDFAN GET`                   | Read fan PID gains                                    | `PIDFAN:GET,<Kp>,<Ki>,<Kd>` |
| `PIDFAN PID <Kp> <Ki> <Kd>`    | Set fan PID gains (≥ 0; runtime only, not saved)      | `PIDFAN:PID,<Kp>,<Ki>,<Kd>` |
| `PIDFAN FILTER ST`             | Read RPM filter state                                 | `PIDFAN_FILTER:<enabled>,<alpha>` |
| `PIDFAN FILTER 1 [<alpha>]`    | Enable EWMA RPM filter, optionally set alpha          | `PIDFAN_FILTER:OK` |
| `PIDFAN FILTER 0`              | Disable RPM filter                                    | `PIDFAN_FILTER:OK` |

`<state>` codes:

| Code | State        |
|------|--------------|
| 0    | Stopped      |
| 1    | Starting     |
| 2    | Running      |
| 3    | Stalled      |
| 4    | Under-speed  |
| 5    | Over-speed   |
| 255  | Unknown      |

`<pwm>` is the PWM duty in % (1 dp); `<FC>` is the requested exhaust fan speed in %. Gains are reported with 4 decimals; alpha with 2.

---

### 7.21 Reserved commands

The following names are reserved for manufacturer service and firmware update. They are not part of the public interface and must not be sent by third-party software:

`SOTA`, `EPP`, `CCFG`

---

## 8. Typical Session

The exchange below is the same on every link. On WebSocket, open `ws://<device-ip>:80/ws` instead of a serial port; each line is then one text message. On Bluetooth LE, each request is one write to the command characteristic and each response is one notification; split the combined requests below into single commands to stay within the 30-character payload limit.

```
Host                                   Device
 |  open link                            |
 |---- {[SC AR]} ----------------------->|
 |<--- {0,SN:AB12345678} ----------------|
 |---- {[VER FW]} ---------------------->|
 |<--- {0,FW:"1.2.3-249B-M2(2)"} --------|
 |---- {[TU C]} ------------------------>|
 |<--- {0,TU:C} -------------------------|
 |                                       |
 |---- {[RC 60,FC 30]} ----------------->|   drum + exhaust on
 |<--- {0,RC:60,FC:30} ------------------|
 |---- {[TS 200,AH 1,HS 1]} ------------>|   preheat to 200 °C under PID
 |<--- {0,TS:200.0,AH:1,HS:1} -----------|
 |                                       |
 |---- {[RD A1]} ---------- every 1 s -->|
 |<--- {0,BT:...,ET:...,...,HS:1} -------|
 |                                       |
 |---- {[HP 70]} ----------------------->|   manual power during roast
 |<--- {0,HP:70} ------------------------|
 |---- {[EV 8]} ------------------------>|   drop: heater off, cooling on
 |<--- {0,EV:8} -------------------------|
 |---- {[EV 9]} ------------------------>|   cooling finished
 |<--- {0,EV:9} -------------------------|
 |---- {[CL AR]} ----------------------->|
 |<--- {0,SN:AB12345678} ----------------|
 |  close link                           |
```

---

## 9. Implementation Checklist for Host Software

- [ ] Serial port: open at **57600 8N1**, no flow control.
- [ ] Bluetooth SPP: pair with the device and open its virtual COM port.
- [ ] WebSocket: connect to `ws://<device-ip>:80/ws` as the first client and use text messages.
- [ ] Bluetooth LE: connect by service UUID, request MTU 128, enable notifications on `11050000-…`, write frames to `11060000-…`.
- [ ] Build frames as `{[` + payload + `]}`.
- [ ] Use upper-case command names and keywords.
- [ ] Keep payloads ≤ 62 characters (≤ 30 on Bluetooth LE) and avoid `{[`, `]`, `}` inside them.
- [ ] Serial / SPP: read responses line by line (`\r\n`). WebSocket: one response per text message. Bluetooth LE: one response per notification.
- [ ] Parse the content between `{` and `}`; skip the first field (`sid`); split the rest on `,` and each item on the first `:`.
- [ ] Detect failure by the absence of the expected key(s), not by `sid`.
- [ ] Use one outstanding request at a time, with a timeout and retry.
- [ ] Validate numeric arguments locally (0–100 for `HP`/`FC`/`RC`).
- [ ] Check the edition with `VER FW` and disable unsupported features in the UI.

---

## Appendix — Command Summary

| Command  | Arguments                         | Response field(s)                         | Function                         |
|----------|-----------------------------------|-------------------------------------------|----------------------------------|
| `RD`     | `A0` \| `A1`                      | `BT,ET,AT,HP,FC,RC,AH,TS[,HS]`            | Read status                      |
| `HP`     | `<0-100>` \| `UP` \| `DW`         | `HP`                                      | Heating power (manual mode)      |
| `FC`     | `<0-100>` \| `UP` \| `DW`         | `FC`                                      | Exhaust fan speed                |
| `RC`     | `<0-100>` \| `UP` \| `DW`         | `RC`                                      | Drum speed                       |
| `TS`     | `<temp>`                          | `TS`                                      | Auto-heat target                 |
| `TU`     | `C` \| `F`                        | `TU`                                      | Temperature unit                 |
| `AH`     | `0` \| `1`                        | `AH`                                      | Auto-heat on/off                 |
| `HS`     | `0` \| `1`                        | `HS`                                      | Heater on/off                    |
| `CS`     | `0` \| `1`                        | `CS`                                      | Cooling fan on/off               |
| `EV`     | `0-9` \| `80`                     | `EV`                                      | Roast event                      |
| `SC`     | `AR` \| `CP`                      | `SN`                                      | Host connected                   |
| `CL`     | `AR` \| `CP`                      | `SN`                                      | Host disconnected                |
| `PI`     | —                                 | —                                         | Ping                             |
| `CSQ`    | `<text>`                          | `CSQ`                                     | Echo                             |
| `VER`    | `FW` \| `SN`                      | `FW` \| `SN`                              | Version / serial number          |
| `DEV`    | `REBOOT` \| `UNP` \| `BOOT BLE` \| `BOOT BT` | — \| `UNP` \| `BLE`            | Device management                |
| `CD`     | `0` \| `1`                        | `CD`                                      | Charge door                      |
| `DD`     | `0` \| `1`                        | `DD`                                      | Discharge door                   |
| `PIDS`   | `<Kp> <Ki> <Kd> <avg> <stab>`     | `PIDS`                                    | Heating PID parameters           |
| `PIDFAN` | `ST` \| `GET` \| `PID ...` \| `FILTER ...` | `PIDFAN` \| `PIDFAN_FILTER`      | Exhaust fan closed loop          |
