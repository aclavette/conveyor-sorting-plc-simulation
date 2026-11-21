# I/O Table

Tag list for the conveyor sorting PLC simulation. Names match `logic/conveyor-sorting.forge.json` exactly.

## Inputs

| Tag | Type | Initial | Description |
|-----|------|---------|-------------|
| StartPB | BOOL | 0 | Start button |
| StopPB | BOOL | 1 | Stop circuit; true = healthy, false = stop requested |
| EStop | BOOL | 1 | E-stop circuit; true = healthy, false = e-stop triggered |
| ResetPB | BOOL | 0 | Reset button |
| PartSensor | BOOL | 0 | True = part detected |
| TallPart | BOOL | 0 | Part classification (tall part) |

## Outputs

| Tag | Type | Initial | Description |
|-----|------|---------|-------------|
| ConveyorMotor | BOOL | 0 | Runs while the system is on |
| Diverter | BOOL | 0 | Pushes tall parts off the line |
| GreenLamp | BOOL | 0 | Running indicator |
| RedLamp | BOOL | 0 | Fault indicator |

## Internal Bits

| Tag | Type | Initial | Description |
|-----|------|---------|-------------|
| SystemOK | BOOL | 0 | Safety permissive |
| Run | BOOL | 0 | Motor run latch |
| Fault | BOOL | 0 | Latched fault |
| PartDetected | BOOL | 0 | Rising-edge pulse from PartSensor |
| Tracking | BOOL | 0 | True if a part is traveling |
| TallMemory | BOOL | 0 | Stores part classification |
| DivertActive | BOOL | 0 | Diverter sequence active |

## Timers

| Tag | Type | Preset | Purpose |
|-----|------|--------|---------|
| TravelTimer | TIMER | 2000 | Part travel time to the diverter |
| DiverterTimer | TIMER | 1000 | Diverter sequence duration |
| JamTimer | TIMER | 5000 | Jam detection; trips Fault |

## Counters

| Tag | Type | Purpose |
|-----|------|---------|
| TotalCount | COUNTER | Total parts |
| TallCount | COUNTER | Tall parts diverted |
| ShortCount | COUNTER | Short parts passed through |

## Notes

- StopPB and EStop are normally true (healthy circuits). A broken wire or a press reads false, which drops out the run latch (fail-safe).
- Timer presets are in milliseconds.
- Modbus register mappings are in [modbus-register-map.md](modbus-register-map.md) (designed, not tested).