# Pipeline Operations SCADA System (PO-SCADA) - Software FAT & Commissioning Report

**Prepared by:** Ibrahim Ali Ghanim  
**Date:** 2026-09-14  
**Revision:** 1.0  
**Disposition:** Backend accepted for Ignition Perspective integration

## 1. Purpose
This report records the software Factory Acceptance Test (FAT) / simulation commissioning of the PO-SCADA Node-RED control, process-model, safety/interlock, fault-injection, and diagnostic logic before Ignition Perspective integration.

> Scope note: this is a software/simulation commissioning record. It is not site SAT, SIL validation, or certification of a real Safety Instrumented System.

## 2. System Under Test
`Factory I/O -> Modbus TCP -> Node-RED -> protocol-neutral SCADA interface -> Ignition Perspective (next phase)`

Physical simulation signals: Tank A Level, Flow In, Fill Valve, Discharge Valve. Node-RED calculates virtual Flow Out, Tank B Level, Pressure In/Out, Flow Difference %, Pressure Drop, fault signatures, trip state, interlocks, alarms/events, and commissioning status.

## 3. Test Method
Precondition -> Execute test -> Compare expected vs actual -> PASS/FAIL -> Corrective action when required -> Retest -> Final acceptance.

## 4. Commissioning Test Matrix

| ID | Test | Acceptance Criteria | Evidence / Actual Result | Status |
|---|---|---|---|---|
| CT-01 | Factory I/O / Modbus I/O verification | Live Tank A / Flow In reads and Fill / Discharge valve writes respond correctly. | Verified during live Modbus operation at 192.168.0.100:502. | PASS |
| CT-02 | Startup sequence | Valid START progresses through startup states and reaches RUNNING without active trip. | Repeated runs reached STABILIZING then RUNNING. | PASS |
| CT-03 | Manual controlled stop | STOP executes controlled shutdown and returns system to STOPPED. | Observed STOPPING_FLOW -> CLOSE_OUTLET -> CLOSE_INLET -> STOPPED. | PASS |
| CT-04 | Emergency stop | E-STOP immediately latches emergency trip and forces safe state. | E-STOP produced TRIPPED with trip reason E-STOP. | PASS |
| CT-05 | START rejection during active trip | START must not restart process while a trip / E-STOP is active. | START command issued while TRIPPED; state remained TRIPPED. | PASS |
| CT-06 | Tank A Low-Low trip | Tank A <= LL setpoint during transfer causes emergency trip. | Tank A reached 0; state changed to TRIPPED; reset succeeded after level recovery. | PASS |
| CT-07 | Tank B High-High shutdown | Tank B >= HH requests controlled shutdown, not emergency trip. | Repeatedly verified; HH preempted early tests before model tuning. | PASS |
| CT-08 | High pressure trip | Pressure >= trip setpoint causes immediate emergency trip and latches. | Functional trip path verified using temporary 5.0 bar commissioning threshold; production value restored to 6.0 bar. | PASS* |
| CT-09 | Persistent no-flow trip | Loss of effective transfer flow while RUNNING trips after configured persistence delay (~5 s). | Transfer Fault ON at 17:55:53; TRIPPED at 17:55:59 with PERSISTENT NO FLOW. | PASS |
| CT-10 | Leak warning | Flow imbalance above warning threshold creates warning without premature trip. | Verified by automated commissioning sequence. | PASS |
| CT-11 | Leak confirmed trip | Persistent flow imbalance + pressure-drop condition causes latched emergency trip. | Verified by automated commissioning sequence after race-condition hardening. | PASS |
| CT-12 | Blockage detection | Low flow ratio + pressure-drop signature is detected. | Verified; blockage also used to validate high-pressure path. | PASS |
| CT-13 | Flow sensor fault simulation | Injected outlet-flow sensor bias is visible and detection reacts. | 730 -> 474.5 (~65%); imbalance alarms / trip path reacted. | PASS |
| CT-14 | Pressure sensor fault simulation | Injected pressure sensor bias increases apparent pressure drop and is observable. | Fault produced increased Delta-P while flow remained approximately unchanged. | PASS |
| CT-15 | RESET rejection with active hazard | RESET must be rejected while the transfer fault remains active. | RESET attempted while TRANSFER fault remained ON; system stayed TRIPPED. | PASS |
| CT-16 | Safe reset / recovery | After hazard removal, RESET clears latched trip and returns safe STOPPED state. | Transfer Fault OFF -> RESET -> STOPPED, trip clear, alarms clear. | PASS |
| CT-17 | Final safe-state acceptance | Final state must be STOPPED with no trip, alarm, or simulated fault. | 18:02:25: STOPPED, TankA=557, TankB=100, Flow=0/0, Pin=1.2, Trip=OFF, Alarms=0, Faults=NONE. | PASS |
| CT-18 | Valve availability / health rejection | Unavailable valve feedback should block startup / relevant permissive. | Not executed: current Factory I/O scene has no real valve-health feedback; availability is virtual. | NOT EXECUTED |

`PASS*` CT-08: functional high-pressure trip path was validated with a temporary 5.0 bar commissioning threshold. The production setting was restored to 6.0 bar after the test.

## 5. Corrective Action Register

| ID | Finding | Corrective Action | Retest Result | Status |
|---|---|---|---|---|
| CA-01 | Tank B model accumulated too quickly and repeatedly reached HH before unrelated tests could complete. | Reduced model_tankBGain from 0.08 to 0.01. | Retest showed slower, usable Tank B rise; no premature HH during subsequent safety tests. | CLOSED |
| CA-02 | Transfer-fault test did not produce no-flow trip because safety used physical Flow In while modeled transfer flow was zero. | Stored effectiveFlowIn in Group 04; Group 05 persistent no-flow logic now uses effectiveFlowIn with physical-flow fallback. | Transfer Fault caused PERSISTENT NO FLOW trip after ~5 s. | CLOSED |
| CA-03 | RESET was accepted after a no-flow trip while the Transfer Fault injector was still active. | Added simTransferFault to RESET permissive / safeToReset logic and cleared timing state on accepted reset. | RESET stayed rejected until Transfer Fault was removed, then reset succeeded. | CLOSED |

## 6. Engineering Observations / Known Limitations
- **OBS-01 - High-pressure testability:** With the current simulation model and blockage gain, normal Pin is ~4.1-4.3 bar and blockage raises it by ~1 bar, so the production limit of 6.0 bar is not normally reached. Functional trip logic was validated with a temporary 5.0 bar commissioning threshold, then 6.0 bar was restored.
- **OBS-02 - Sensor-fault discrimination:** The Flow Sensor Fault intentionally biases Flow Out to ~65%, which can resemble a leak signature and drive the leak-detection path. This validates detection response but also exposes a future need for sensor-plausibility / fault-discrimination logic.
- **OBS-03 - Safety scope:** Node-RED is acting as an educational / simulation control backend. This test does not represent SIL validation or a real Safety Instrumented System. A real installation would keep critical safety functions in suitable PLC/SIS hardware and follow formal functional-safety lifecycle requirements.
- **OBS-04 - SCADA boundary:** The Node-RED <-> Ignition transport remains intentionally protocol-neutral at this stage. Protocol selection and interface testing are part of the next integration phase.

## 7. Key Evidence Log

| Time (UTC+3) | Event | Evidence |
|---|---|---|
| 17:43:33 | E-STOP trip | State=TRIPPED \| TRIP=E-STOP \| Alarms=2 \| Faults=NONE |
| 17:43:37 | START while tripped | START command issued; state remained TRIPPED. |
| 17:55:53 | Transfer fault injected | State=RUNNING \| Flow=729/0 \| Faults=TRANSFER |
| 17:55:59 | Persistent no-flow trip | State=TRIPPED \| TRIP=PERSISTENT NO FLOW |
| 17:56:08 | Reset attempted with hazard active | State remained TRIPPED while TRANSFER fault was still active. |
| 17:56:38 | Reset after hazard removal | State=STOPPED \| trip cleared. |
| 17:59:33 | Flow sensor fault | Flow=730/474.5 (~65% outlet indication). |
| 18:00:39 | Pressure sensor fault | Flow approximately unchanged; Delta-P increased. |
| 18:02:25 | Final acceptance state | State=STOPPED \| TankA=557 \| TankB=100 \| Flow=0/0 \| Pin=1.2 \| DeltaP=0 \| TRIP=OFF \| Alarms=0 \| Faults=NONE |

## 8. Final Acceptance
**FINAL STATUS: PASSED - Node-RED backend accepted for the next SCADA integration phase.**

Final observed state: `STOPPED | TankA=557 | TankB=100 | Flow=0/0 | Pin=1.2 | DeltaP=0 | TRIP=OFF | Alarms=0 | Faults=NONE`.

## 9. Next Professional Hardening Steps
1. Define the Node-RED <-> Ignition Interface Control Document (ICD) and select the supervisory protocol.
2. Create an ISA-style tag / equipment naming convention and a simplified P&ID / process drawing.
3. Build a Cause & Effect matrix linking initiators, alarms, trips, shutdown actions, reset conditions, and final elements.
4. Add alarm rationalization: priority, deadband, delay, shelving/acknowledgement philosophy, and nuisance-alarm checks.
5. Add sensor plausibility / fault-discrimination logic so sensor faults are distinguishable from real leak/blockage conditions.
6. Add communication-loss / stale-data / watchdog tests between Factory I/O, Node-RED, and Ignition.
7. Add historian, event/audit log, user roles, change log, backup/restore, and cybersecurity zoning considerations.
8. Later maintain separate FAT and SAT documents if the project is moved to real hardware.
