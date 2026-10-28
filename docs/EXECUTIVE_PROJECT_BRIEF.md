# PO-SCADA — Executive Project Brief

## Pipeline Operations SCADA System

PO-SCADA is a three-layer simulation and supervisory-control project integrating **Factory I/O**, **Node-RED**, and **Ignition Perspective**. The system monitors tank level, flow and pressure; implements startup/stop logic, leak/no-flow/blockage/high-pressure protection, alarms and trips; and adds historian, diagnostics, role-based access control and audit traceability.

## Key results

| KPI | Result |
|---|---:|
| Validated tests passed | **41** |
| Integrated FAT | **24/24 PASS** |
| Closed corrective actions | **3** |
| Security role levels | **3** |

## Operational problem

A pipeline operator needs early indication of abnormal flow/pressure conditions, deterministic protection behavior, traceable actions, and reliable recovery from communication failures.

## Engineering response

A layered SCADA architecture combines simulated process I/O, control and protection logic, supervisory HMI, alarm/trend history, diagnostics, RBAC and audit logging.

```text
Factory I/O
   ↓ Modbus TCP
Node-RED control / process model / protection
   ↓ OPC UA
Ignition Perspective
   ↓
Historian • Alarm Journal • Audit • PostgreSQL
```

## Operational capabilities

- Process monitoring: Tank A/B level, Flow In/Out, Pressure In/Out, flow imbalance and pressure drop
- Protection: LL/HH tank level, leak warning/confirmed, blockage, high pressure, persistent no-flow, E-stop, trip/reset
- Operations history: historian, trends, alarm journal and audit log
- Security: Operator, Engineer and Administrator role levels
- Reusable engineering: UDT-based instruments, faceplates, engineering settings and diagnostics
- PIFDE: explainable rule-based advisory classification with live evidence and recommended operator action

## Validation summary

- Backend software FAT / commissioning: **17 executed tests PASS**, with one valve-health test not executed because the simulation has no real valve-health feedback.
- Integrated SCADA FAT: **24/24 PASS** across process, protection, alarms, trends, diagnostics, security, audit, cross-browser, communication loss and recovery.
- Corrective-action closure: **3 findings corrected, retested and CLOSED**.
- Representative protection result: transfer fault injected at **17:55:53**; persistent no-flow trip recorded at **17:55:59 (~6 s)**.

## Technology stack

Ignition Perspective 8.3.x · Node-RED · Factory I/O · Modbus TCP · OPC UA · PostgreSQL · Tag Historian · Alarm Journal · Audit Profile · UDTs / Faceplates

## Scope note

This is a **software/simulation commissioning project**. It is not a site SAT, SIL assessment, or certification of a real Safety Instrumented System.
