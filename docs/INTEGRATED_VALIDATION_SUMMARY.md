# Integrated Validation Summary

## Final validation status

**PASS — integrated validation complete for the software/simulation scope.**

### Backend commissioning

- 17 executed commissioning tests: **PASS**
- CT-18 Valve availability / health: **NOT EXECUTED** because the current Factory I/O scene has no real valve-health feedback
- 3 corrective actions: **CLOSED**

### Integrated SCADA FAT

**24/24 PASS**

Coverage included:

- Overview baseline and process values
- Node-RED / OPC UA communications and heartbeat
- LT / FT / PT process tags
- Process state and valve indication
- Leak warning and leak-confirmed behavior
- Blockage detection
- Active alarms and alarm clear
- Trends
- Diagnostics
- Operator access denial to Engineering / Diagnostics
- Engineer access to Engineering / Diagnostics
- Administrator access to Security Audit
- Audit activity recording
- Navigation and cross-browser checks
- Communication-loss negative test
- Recovery after communication restoration
- Save / refresh persistence
- Final normal state

## Representative evidence

| Event | Result |
|---|---|
| E-STOP | State changed to TRIPPED; trip reason E-STOP |
| START while tripped | State remained TRIPPED |
| Transfer fault | Injected at 17:55:53 |
| Persistent no-flow | Trip recorded at 17:55:59 |
| Reset with hazard active | Rejected; system stayed TRIPPED |
| Reset after hazard removed | Returned to STOPPED; trip cleared |
| Flow sensor fault | Outlet indication changed from ~730 to ~474.5 |
| Pressure sensor fault | Apparent pressure drop increased |
| Final acceptance | STOPPED, Trip OFF, Alarms 0, Faults NONE |

## Corrective actions

### CA-01 — Tank B model accumulation
Reduced `model_tankBGain` from 0.08 to 0.01 to prevent premature HH during unrelated tests.

### CA-02 — Persistent no-flow path
Changed the protection path to use `effectiveFlowIn` with physical-flow fallback so the transfer-fault scenario could be detected correctly.

### CA-03 — Reset permissive
Added `simTransferFault` to reset permissive / `safeToReset` logic so reset remains rejected while the simulated transfer hazard is active.

## Engineering limitation

High-pressure functional logic was validated using a temporary 5.0 bar commissioning threshold and the production value was restored to 6.0 bar after testing.

This validation is for a software/simulation environment and does not represent SIL validation, site SAT, or certification of a real Safety Instrumented System.
