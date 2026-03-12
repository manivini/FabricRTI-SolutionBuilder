# Fabric RTI – Manufacturing Predictive Maintenance & Equipment Health

Real-time collection of machine sensor data (vibration, temperature, pressure, current) to predict and prevent equipment failures. Streaming analytics detect early warning signs of degradation, allowing maintenance teams to schedule repairs proactively — minimising unplanned downtime and extending the life of costly industrial assets.

## Architecture

```
IoT Sensors → Eventstream → BronzeEquipmentTelemetry
                                    │
                              Update Policy
                                    │
                           SilverEquipmentTelemetry
                                    │
                  ┌─────────────────┼─────────────────┐
          mv_MachineHealth    mv_AnomalyCounts    mv_SensorTrends
                  │                  │                   │
            Dashboard          Data Activator       Dashboard
```

## What's in the Repo

| File | Description |
|---|---|
| `PredictiveMaintenance_SetupGuide.ipynb` | Step-by-step Fabric UI walkthrough (11 steps) |
| `PredictiveMaintenance_Generator.ipynb` | Synthetic equipment telemetry generator |
| `predictivemaintenance_createAll.kql` | KQL schema script (tables, functions, policies, views) |
| `CICD_Patterns_Reference.md` | Git integration + deployment pipeline patterns |

## Quick Start

1. **Create Eventhouse** + KQL Database (F2+ capacity)
2. **Create Eventstream** with Custom Endpoint
3. **Copy** Event Hub Name & Connection String
4. **Import** generator notebook → paste credentials → Run All
5. **Run** KQL blocks one-by-one from `predictivemaintenance_createAll.kql`
6. **Create** Real-Time Dashboard → add tiles (queries in Setup Guide)

## Medallion Layers

| Layer | Table/View | Purpose |
|---|---|---|
| Bronze | `BronzeEquipmentTelemetry` | Raw JSON from Eventstream (auto-created) |
| Silver | `SilverEquipmentTelemetry` | Typed, enriched, anomaly-flagged sensor data |
| Gold | `mv_MachineHealth` | Hourly health score averages per machine |
| Gold | `mv_AnomalyCounts` | Hourly anomaly counts by machine & sensor type |
| Gold | `mv_SensorTrends` | 15-minute rolling sensor averages for trend charts |

## Monitored Machines

| Machine ID | Name | Location | Key Sensors |
|---|---|---|---|
| CNC-007 | CNC Milling Machine #7 | Plant A – Bay 3 | Temperature, Vibration, SpindleRPM |
| HYD-003 | Hydraulic Press Line 3 | Plant A – Bay 7 | Pressure, Vibration, Temperature |
| CMP-A1 | Industrial Air Compressor A1 | Plant B – Utility Room | Temperature, Vibration, Current |
| CNV-012 | Conveyor Belt Unit 12 | Plant B – Assembly Line 2 | Torque, Speed, Temperature |

## Crisis Scenarios

| Scenario | Machine | Trigger |
|---|---|---|
| Overheat Spike | CNC-007 | Motor temperature > 85°C |
| Vibration Anomaly | HYD-003 | Vibration 3× above baseline |
| Pressure Drop | HYD-003 | Pressure falls > 30% in 2 min |
| Recurring Stops | CNV-012 | 5+ unplanned stops/hour |
| Power Surge | CMP-A1 | Current draw +20% above normal |

## Demo Trigger

Run **Cell 6** in the generator notebook to inject a crisis burst on demand:

```python
TRIGGER_TYPE   = "crisis"     # or "macro"
TRIGGER_TARGET = "CNC-007"    # Machine ID or macro event ID
```

Available macro events:
- `spare_parts_shortage` — Global bearing/seal supply crunch
- `energy_price_spike` — Electricity costs surge 40%
- `safety_regulation` — Stricter HSE monitoring requirements

## Known Fabric KQL Quirks

- Run each `.create` / `.alter` block **individually** (batch = Recognition Error)
- Use `todatetime()`, `toreal()`, `toint()`, `tobool()` — Eventstream sends strings
- Update Policy JSON: **single-line escaped** (`"[{\"...\"}]"`), NOT `@'...'` syntax
- Materialized views: query via `materialized_view('mv_name')` for fresh results
- OneLake enabled → `.clear table` blocked → toggle off first

## Git Integration

1. Workspace → **Settings** → **Git integration** → GitHub
2. Authenticate → select repo → branch: `main` → folder: `/`
3. Connect and sync

## Deployment Pipelines

1. **Deployment Pipelines** → **+ New pipeline**
2. Assign Dev workspace → Create Prod workspace
3. Deploy Dev → Prod for promotion

## Prerequisites

- Microsoft Fabric **F2+ capacity** (or Trial)
- Python packages: `azure-eventhub`, `faker`
- GitHub account (for Git integration)
