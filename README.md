# PO-SCADA — Pipeline Operations SCADA System

**Integrated SCADA, process protection, diagnostics, security, audit, historian, and validation portfolio project.**

![PO-SCADA System Architecture](media/system-architecture-and-protection.png)

## Project at a glance

| Metric | Result |
|---|---:|
| Validated tests passed | **41** |
| Integrated FAT | **24/24 PASS** |
| Closed corrective actions | **3** |
| Security role levels | **3** |

PO-SCADA is a three-layer simulation and supervisory-control project integrating **Factory I/O**, **Node-RED**, and **Ignition Perspective 8.3**. It monitors tank level, flow, and pressure; implements startup/stop logic and protection against leak, no-flow, blockage, high pressure, low/high tank levels, and E-stop conditions; and adds historian, alarms, diagnostics, role-based security, and audit traceability.

## Operational problem

Pipeline operators need early indication of abnormal flow and pressure conditions, deterministic protection behavior, traceable actions, and reliable recovery from communication failures.

## Engineering response

The project uses a layered architecture:

```text
Factory I/O process simulation
        ↓ Modbus TCP
Node-RED control / process model / protection
        ↓ OPC UA
Ignition Perspective SCADA
        ↓
Historian • Alarm Journal • Audit • PostgreSQL • Role-Based Security
```

### Layer 1 — Factory I/O

- Tank A level and inlet flow simulation
- Fill and discharge valve actuation
- Modbus TCP/IP Server
- Local deployment endpoint used in the final setup: `127.0.0.1:502`, Slave ID `1`

### Layer 2 — Node-RED

- Command processing and sequence control
- Startup permissives and controlled shutdown
- Safety/interlock logic
- Leak, blockage, high-pressure and persistent no-flow detection
- Calculated/virtual process values
- Fault injection and commissioning support
- OPC UA supervisory interface to Ignition

### Layer 3 — Ignition Perspective

- Process Overview
- Alarms and Trends
- Diagnostics and Heartbeat
- Engineering Settings
- Reusable UDT-based instrument and valve faceplates
- Operator / Engineer / Administrator access control
- Audit logging and historian integration
- PIFDE explainable rule-based advisory layer

## Live operational evidence

### Process Overview

![Process Overview](media/process-overview.png)

### Diagnostics

![System Diagnostics](media/system-diagnostics.png)

### Engineering settings

![Engineering Settings](media/engineering-settings.png)

### Reusable faceplates

| Tank instrument | Valve |
|---|---|
| ![Tank Faceplate](media/tank-faceplate.png) | ![Valve Faceplate](media/valve-faceplate.png) |

### PIFDE Integrity Advisor

![PIFDE Integrity Advisor](media/pifde-integrity-advisor.png)

PIFDE is an **advisory / explainable diagnostic layer**. Deterministic protection logic remains authoritative.

## Protection and control scope

- Start permissives and startup sequence
- Controlled operator stop
- Emergency stop and trip latching
- Tank A low-low protection
- Tank B high-high shutdown
- High-pressure protection
- Leak warning and leak-confirmed logic
- Blockage detection
- Persistent no-flow detection
- Trip reset permissives and safe recovery
- Simulated flow and pressure sensor faults

## Validation highlights

The backend software FAT recorded **17 executed tests PASS** plus one valve-health test not executed because the simulation has no real valve-position health feedback.

The integrated Ignition FAT recorded **24/24 PASS** across process, protection, alarms, trends, diagnostics, security, audit, cross-browser behavior, communication loss, and recovery.

A representative protection test injected a transfer fault at `17:55:53`; the persistent no-flow trip was recorded at `17:55:59` (~6 s), validating the configured persistence behavior.

Three commissioning findings were corrected, retested, and formally closed:

1. Tank B model accumulation was too aggressive.
2. Persistent no-flow logic initially used physical flow instead of effective modeled flow.
3. Reset permissive initially allowed reset while a simulated transfer fault remained active.

## Security model

| Role | Operational access |
|---|---|
| Operator | Operational views; restricted from engineering/diagnostic protected views according to the configured policy |
| Engineer | Engineering and diagnostics access |
| Administrator | Full access including security/audit functions |

Perspective view permissions were validated using positive and negative access tests, including **Access Denied** behavior for unauthorized users.

## Technology stack

- **Ignition Perspective 8.3.x**
- **Node-RED**
- **Factory I/O**
- **Modbus TCP**
- **OPC UA**
- **PostgreSQL**
- Ignition Tag Historian
- Alarm Journal
- Audit Profile
- UDTs and reusable faceplates

## Documentation

- [Executive Project Brief](docs/PO_SCADA_Executive_Project_Brief.pdf) — concise 3-page portfolio brief
- [Final Integrated Project Report — Rev 2.0](docs/PO_SCADA_Final_Integrated_Project_Report_Rev2.pdf) — final detailed report
- [Software FAT & Commissioning Report](docs/PO_SCADA_Software_FAT_Commissioning_Report.pdf) — backend commissioning evidence
- [Phase 13 Integrated FAT / Validation](docs/PO_SCADA_Phase_13_Integrated_FAT_Validation.pdf) — end-to-end SCADA FAT
- [Commissioning Report (Markdown)](docs/COMMISSIONING_REPORT.md)
- [Commissioning Test Matrix (CSV)](docs/COMMISSIONING_TEST_MATRIX.csv)

## Reusable engineering example

- [Analog Instrument UDT Instances](examples/PO_SCADA_AnalogInstrument_Instances_v1.json)

This example demonstrates reusable instrument instances for LT / FT / PT tags bound to the PO-SCADA process tag structure.

## Engineering scope note

This is a **software/simulation commissioning project**. It is not a site SAT, SIL assessment, or certification of a real Safety Instrumented System. In a real installation, critical safety functions would remain in appropriate PLC/SIS hardware and follow the applicable functional-safety lifecycle.

## Author

**Ibrahim Ali Ghanim**

Industrial Automation • SCADA • OT / IIoT • System Integration
