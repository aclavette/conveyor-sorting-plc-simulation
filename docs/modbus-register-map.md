# Modbus Register Map

> **Status: designed, not tested.** The browser-based FORGE simulator has no Modbus interface, so this map was never exercised against a Modbus client. It documents how the program's tags would be exposed on a real controller.

Tag names match `logic/conveyor-sorting.json`. See [io-table.md](io-table.md) for tag descriptions.

## Conventions

- **Addresses** are zero-based protocol addresses, as sent on the wire. The reference column shows the traditional 5-digit numbers (add 1 to the address and use the prefix for the table: 0xxxx coils, 1xxxx discrete inputs, 3xxxx input registers, 4xxxx holding registers).
- **Registers** are unsigned 16-bit integers.
- **Times** are in milliseconds, matching the program's timer presets.
- **Unit ID, port, byte order, and polling rate** are not specified, because there was no client to test against.

## Coils (read/write bits)

Function codes 01 (read), 05 (write single), 15 (write multiple). These are remote commands. Each is momentary: the client writes 1, then 0, just like pressing and releasing a pushbutton.

| Address | Reference | Tag | Description |
|---------|-----------|-----|-------------|
| 0 | 00001 | `StartPB` | Remote start request |
| 1 | 00002 | `ResetPB` | Remote fault reset request |
| 2 | 00003 | `StopPB` | Remote stop request. **Inverted:** 1 = healthy (default), write 0 to request a stop, then write 1 to restore |

## Discrete inputs (read-only bits)

Function code 02. Status for monitoring.

| Address | Reference | Tag | Description |
|---------|-----------|-----|-------------|
| 0 | 10001 | `SystemOK` | Safety permissive satisfied |
| 1 | 10002 | `Run` | Run latch on |
| 2 | 10003 | `Fault` | Jam fault latched |
| 3 | 10004 | `ConveyorMotor` | Conveyor motor output |
| 4 | 10005 | `Diverter` | Diverter output |
| 5 | 10006 | `GreenLamp` | Running indicator |
| 6 | 10007 | `RedLamp` | Fault indicator |
| 7 | 10008 | `EStop` | E-stop circuit; 1 = healthy, 0 = e-stop triggered |
| 8 | 10009 | `PartSensor` | Part detected at the sensor |
| 9 | 10010 | `TallPart` | Current part classified as tall |
| 10 | 10011 | `Tracking` | A part is traveling toward the diverter |
| 11 | 10012 | `DivertActive` | Diverter sequence active |

## Input registers (read-only 16-bit)

Function code 04.

| Address | Reference | Tag | Description |
|---------|-----------|-----|-------------|
| 0 | 30001 | `TotalCount` | Total parts detected |
| 1 | 30002 | `TallCount` | Tall parts detected |
| 2 | 30003 | `ShortCount` | Short parts detected |
| 3 | 30004 | `TravelTimer` accumulated value | Elapsed travel time (ms) |
| 4 | 30005 | `DiverterTimer` accumulated value | Elapsed diverter time (ms) |
| 5 | 30006 | `JamTimer` accumulated value | Elapsed jam-detection time (ms), up to 5000 |

## Holding registers (read/write 16-bit)

Function codes 03 (read), 06 (write single), 16 (write multiple). Optional tuning values.

| Address | Reference | Tag | Description | Units |
|---------|-----------|-----|-------------|-------|
| 0 | 40001 | `TravelTimer` preset | Sensor-to-diverter travel time (default 2000) | ms |
| 1 | 40002 | `DiverterTimer` preset | Diverter on-time (default 1000) | ms |

## Design notes

- **E-stop is read-only.** It is exposed for monitoring but cannot be written. A safety function should stay hardwired and never depend on a network command.
- **Remote commands go through the same logic as the pushbuttons.** A remote start still needs `SystemOK`. A remote reset still needs the sensor clear and a healthy e-stop, so the network cannot override the safety permissive or the fault latch.
- **Stop polarity.** `StopPB` is healthy-high, so the stop coil is inverted. A client that holds it at 0 keeps the system stopped, and writing 1 restores it.
- **Jam time is not writable.** `JamTimer`'s preset is a protection setting, so it is left out of the holding registers. Only the two process timers can be tuned remotely.
- **Counters** fit comfortably in 16 bits (preset 1000).
- **Not defined:** behavior on loss of communication, write-protection of the holding registers, and any authentication. A real deployment would need all three.