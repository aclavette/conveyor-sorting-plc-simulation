# Test Plan

Tested in the FORGE browser-based PLC simulator.

- Program: `logic/conveyor-sorting.forge.json`
- Scan time: 20 ms. Timer presets: TravelTimer 2000 ms, DiverterTimer 1000 ms, JamTimer 5000 ms.


## A. Permissives and run latch

| # | Test | Precondition | Action | Expected | Actual | Pass/Fail |
|---|------|--------------|--------|----------|--------|-----------|
| 1 | Initial state | Fresh reset | Observe | SystemOK on; all other outputs and internal bits off; StopPB and EStop true; counters at 0 | After reset, all outputs and internal bits were off, StopPB and EStop were true, and all counters were at 0. | Pass |
| 2 | Start | Fresh reset | Pulse StartPB | SystemOK, Run, ConveyorMotor, and GreenLamp turn on | After pulsing StartPB, Run, ConveyorMotor, and GreenLamp turned on and stayed on. RedLamp, Fault, and Diverter stayed off. | Pass |
| 3 | Run seals in | Running (after T2) | Release StartPB | Run, ConveyorMotor, and GreenLamp stay on | With StartPB released, Run, ConveyorMotor, and GreenLamp stayed on. SystemOK stayed on; Fault, RedLamp, and Diverter stayed off. | Pass |
| 4 | Stop | Running | Set StopPB false, then back to true | Run, ConveyorMotor, and GreenLamp drop; SystemOK stays on; nothing restarts when StopPB returns to true | With StopPB false, Run, ConveyorMotor, and GreenLamp turned off and SystemOK stayed on. After StopPB was restored, all three stayed off. | Pass |
| 5 | E-stop while running | Running | Set EStop false | SystemOK, Run, ConveyorMotor, and GreenLamp all drop | With EStop false, SystemOK, Run, ConveyorMotor, and GreenLamp turned off. Fault, RedLamp, and Diverter stayed off. | Pass |
| 6 | Start blocked during e-stop | EStop false | Pulse StartPB | Run and ConveyorMotor stay off | With EStop false, pulsing StartPB had no effect. Run, ConveyorMotor, and GreenLamp stayed de-energized and SystemOK stayed off. | Pass |
| 7 | Restart after e-stop | After T6 | Set EStop true, observe, then pulse StartPB | Nothing restarts when EStop is restored; Run and ConveyorMotor come on only after StartPB | When EStop was restored, SystemOK turned back on but Run, ConveyorMotor, and GreenLamp stayed off.All three turned on and stayed on after pulsing StartPB. | Pass |

## B. Part tracking and diverter

Start each test with the system running (pulse StartPB after reset).

| # | Test | Precondition | Action | Expected | Actual | Pass/Fail |
|---|------|--------------|--------|----------|--------|-----------|
| 8 | Part detection | Running, TallPart false | Set PartSensor true and hold about 1 s, then false | PartDetected is true for one scan only (not held while PartSensor stays true); Tracking latches; TravelTimer starts counting | With PartSensor true, Tracking turned on and TravelTimer started counting. TotalCount increased by 1 and stayed at 1 while PartSensor was held, so PartDetected pulsed once. Tracking stayed on after PartSensor was released. | Pass |
| 9 | Classification stored | Running | Set TallPart true, then pulse PartSensor | TallMemory latches at detection and clears when TravelTimer finishes (~2 s) | With TallPart true, TallMemory turned on when PartSensor went on and stayed on after PartSensor was released. TallMemory turned off at ~2 s when TravelTimer finished. TallCount increased to 1. | Pass |
| 10 | Tall part diverted | Running | Set TallPart true, pulse PartSensor, watch for 4 s | Diverter off until ~2.0 s, on at ~2.0 s, off at ~3.0 s; DivertActive and Tracking clear at ~3.0 s | With TallPart true, Diverter stayed off until ~2.0 s, turned on at ~2.0 s, and turned off at ~3.0 s. DivertActive and Tracking turned off at ~3.0 s. At 4 s, Tracking, TallMemory, DivertActive, and Diverter were all off. | Pass |
| 11 | Short part passes | Running | Set TallPart false, pulse PartSensor, watch for 4 s | Diverter never turns on; Tracking clears when TravelTimer finishes at ~2.0s | Diverter correctly never turns on. Tracking latched at detection and was still on at 4 s | Fail |
| 12 | Short part then tall part | Running, Tracking false | Pulse a short part, wait 3 s, then pulse a tall part | Tall part is diverted ~2.0 s after its own detection, not immediately | After the short part, Tracking turned off at ~2 s. For the tall part, Diverter stayed off until ~2.0 s after its detection, turned on at ~2.0 s, and turned off at ~3.0 s. | Pass |

## C. Counters

T13-15 share one run. Reset the simulator, start the system, then send 2 tall parts and 1 short part (TallPart true for the tall ones, false for the short one), waiting 3 s between parts so each finishes.

| # | Test | Precondition | Action | Expected | Actual | Pass/Fail |
|---|------|--------------|--------|----------|--------|-----------|
| 13 | Total count | Counters at 0 | Run the sequence above | TotalCount = 3 | After 2 tall parts and 1 short part, TotalCount read 3. | Pass |
| 14 | Tall count | Same run | Read TallCount | TallCount = 2 | After the same run, TallCount read 2. | Pass |
| 15 | Short count | Same run | Read ShortCount | ShortCount = 1 | After the same run, ShortCount read 1. | Pass |

## D. Jam fault and reset

| # | Test | Precondition | Action | Expected | Actual | Pass/Fail |
|---|------|--------------|--------|----------|--------|-----------|
| 16 | Jam detected | Running | Set PartSensor true and hold it | JamTimer counts; Fault latches at ~5.0 s | With PartSensor held true, Fault turned on at ~5.0 s. | Pass |
| 17 | Fault response | T16 state, PartSensor still true | Observe | SystemOK, Run, ConveyorMotor, and GreenLamp off; RedLamp on | With Fault on, SystemOK, Run, ConveyorMotor, and GreenLamp turned off and RedLamp turned on. | Pass |
| 18 | Reset blocked while jammed | T17 state | Pulse ResetPB with PartSensor still true | Fault and RedLamp stay on | With PartSensor still true, pulsing ResetPB had no effect. Fault and RedLamp stayed on. | Pass |
| 19 | Fault stays latched | T18 state | Set PartSensor false | Fault and RedLamp stay on until reset | After PartSensor was released, Fault and RedLamp stayed on. | Pass |
| 20 | Reset | T19 state, EStop true | Pulse ResetPB | Fault and RedLamp clear; SystemOK returns; Run and ConveyorMotor stay off until StartPB is pulsed | After pulsing ResetPB, Fault and RedLamp turned off and SystemOK turned on. Run, ConveyorMotor, and GreenLamp stayed off until StartPB was pulsed. | Pass |
| 21 | Reset blocked during e-stop | Fault latched, PartSensor false | Set EStop false, pulse ResetPB | Fault stays on until EStop is restored and ResetPB is pulsed again | With EStop false, pulsing ResetPB had no effect and Fault stayed on. After EStop was restored and ResetPB was pulsed again, Fault and RedLamp turned off. | Pass |

## E. Documented behavior

| # | Test | Precondition | Action | Expected | Actual | Pass/Fail |
|---|------|--------------|--------|----------|--------|-----------|
| 11 | Short part passes | Running | Set TallPart false, pulse PartSensor, watch for 4 s | Diverter never turns on; Tracking clears when TravelTimer finishes | | |


## Notes
- Test failures and fixes
    - Test 11 failed on first run; Tracking stayed latched after TravelTimer finished instead of clearing at 2.0 s. 
        - Cause: the only rung that clears Tracking is triggered by DiverterTimer being done; a short part never clears Tracking.
        - Fix: Added rung that clears Tracking when TravelTimer finishes and the part is not tall. Found and deleted redundant rung.
    - Re-test: after adding the rung, T11 passed (Tracking cleared at ~2 s, diverter stayed off)
- Test 16 also registers a part detection, so counters increment during jam tests.
