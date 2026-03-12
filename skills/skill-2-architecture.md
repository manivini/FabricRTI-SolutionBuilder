# Skill 2: Architecture Design

## Purpose
Take the use case specification from Skill 1 and design the complete Microsoft Fabric RTI architecture — what items to create, data flow topology, medallion layer design, and how streams connect.

## Input
The `use_case_spec` YAML from Skill 1.

## Output
An architecture document containing:
1. Fabric items to create (with names)
2. Data flow diagram
3. Stream topology (Eventstream configuration)
4. Bronze table schema (raw ingest)
5. Silver table schema (enriched) + transformation function design
6. Gold layer design (materialized views)
7. Update Policy specification
8. Whether an operations agent is beneficial (and what it would do)

## Architecture Decision Process

### Step 1 — Determine the Stream Pattern

Analyze the `event_streams` from the spec and classify:

**Single-Stream Pattern** (like PE Sentiment demo):
- One event type flows through one Eventstream → one Bronze table
- Update Policy handles Bronze → Silver enrichment
- Best for: sentiment monitoring, simple event processing
- Indicators: 1 stream in spec, all events have same schema

**Multi-Stream Pattern** (like Portfolio Performance):
- Multiple event types from different sources
- Each stream → its own Bronze table (or use a single Bronze with event type discriminator)
- Silver layer joins/correlates across streams
- Best for: financial monitoring, operational dashboards
- Indicators: 2+ streams in spec with different schemas

**Threshold/Anomaly Pattern** (like Fraud Detection):
- High-volume transaction stream
- Real-time scoring/rules applied in Silver
- Alert events generated as separate output
- Best for: fraud, anomaly detection, compliance
- Indicators: spec mentions thresholds, risk scores, anomaly detection

**Hybrid** — Most real-world demos combine patterns. Design accordingly.

### Step 2 — Design Fabric Items

For every demo, you need at minimum:

| Item | Naming Convention | Notes |
|---|---|---|
| Eventhouse | `{UseCase}Monitoring_EH` | Contains the KQL Database |
| KQL Database | Same as Eventhouse (auto-created) | All tables and views live here |
| Eventstream | `{UseCase}Stream_ES` | One per major data source |
| Custom Endpoint | `{UseCase}Source` | Entry point for synthetic data |
| Real-Time Dashboard | `{UseCase}_Dashboard` | 6-8 tiles |
| Data Activator (Reflex) | `{UseCase}Alert` | Crisis/anomaly alerts |
| Notebook (Generator) | `{UseCase}_Generator` | Synthetic data generator |

For multi-stream patterns, create multiple Eventstreams or use a single Eventstream with routing.

### Step 3 — Design Bronze Layer

The Bronze table receives raw JSON from Eventstream. Design principles:
- **No transformation** at ingestion — raw JSON as-is
- Let Eventstream auto-create the table (it infers schema from first events)
- Table name: `Bronze{EventType}` (e.g., `BronzeTransactions`, `BronzeFeedback`)
- For multi-stream: one Bronze table per stream, or one table with `eventType` discriminator

Map the `event_streams[].fields` from the spec directly to Bronze columns.

### Step 4 — Design Silver Layer

Silver = Bronze + enrichment. For each Bronze table, design a corresponding Silver table and transformation function.

**Standard enrichments to consider** (pick what's relevant from the spec):
- Type casting: `todatetime()`, `toreal()`, `toint()`, `tobool()` — JSON arrives as strings in Fabric
- Computed categories/buckets (e.g., `sentimentBucket`, `riskLevel`, `severityLabel`)
- Boolean flags (e.g., `isAnomaly`, `isCrisis`, `isHighRisk`)
- Time bucketing: `bin(timestamp, 1h)` for hourly aggregation downstream
- Derived fields (e.g., `transactionVelocity`, `geoRegion`)

**Transformation function pattern**:
```kql
.create-or-alter function Transform{EventType}() {
    Bronze{EventType}
    | extend
        field1 = todatetime(field1),
        field2 = toreal(field2),
        computedField = <business logic>,
        timeBucket = bin(todatetime(timestamp), 1h)
    | project <all Silver columns>
}
```

**Update Policy** (auto-fires on Bronze insert → populates Silver):
```kql
.alter table Silver{EventType} policy update
"[{\"IsEnabled\":true,\"Source\":\"Bronze{EventType}\",\"Query\":\"Transform{EventType}()\",\"IsTransactional\":false,\"PropagateIngestionProperties\":false}]"
```

> CRITICAL: Use single-line escaped JSON for Update Policy. Fabric does NOT support `@'...'` syntax.

### Step 5 — Design Gold Layer

Gold = pre-computed aggregations from Silver. Use **Materialized Views**.

**Common Gold patterns**:

| Aggregation Type | KQL Pattern | When to Use |
|---|---|---|
| Time-series by entity | `summarize avg/sum/count by entity, bin(time, 1h)` | Trend dashboards |
| Anomaly/crisis rollup | `summarize count, sum by entity, bin(time, 30m) where isAnomaly` | Alert panels |
| Top-N ranking | `summarize count by category \| top 10` | Keyword/category tiles |
| Distribution | `summarize count by bucket/label` | Pie charts, histograms |

Map the `gold_aggregations` from the spec to materialized views:
```kql
.create materialized-view mv_{AggregationName} on table Silver{EventType} {
    Silver{EventType}
    | summarize <aggregations> by <dimensions>, bin(timestamp, <window>)
}
```

### Step 6 — Assess Operations Agent Need

Ask: "Does this use case benefit from an intelligent agent beyond threshold alerts?"

**Add an agent if**:
- Multiple entities need cross-correlation ("2 companies declining simultaneously")
- Actions need reasoning, not just notification
- Users want to ask questions about live data
- Summaries/recommendations should be auto-generated

**Skip agent if**:
- Simple threshold alerts suffice
- Audience just needs a dashboard

If agent is recommended, note:
- What data it monitors (which Gold views)
- What reasoning it applies
- What actions it takes
- Suggested implementation (Foundry Agent, Notebook + LLM, or Copilot Agent)

### Step 7 — Produce Architecture Document

Output format:

```
## Architecture: {UseCase} – {Industry}

### Fabric Items
| Item | Name | Type |
|---|---|---|
| ... | ... | ... |

### Data Flow
```
{Source} → Eventstream → Bronze{Table}
                              │
                         Update Policy
                              │
                         Silver{Table}
                              │
                    ┌─────────┼─────────┐
               mv_{Gold1}         mv_{Gold2}
                    │                  │
              Dashboard          Data Activator
```

### Bronze Schema
| Field | Type | Description |
|---|---|---|

### Silver Schema (enriched)
| Field | Type | Source | Description |
|---|---|---|---|

### Gold Materialized Views
| View | Aggregation | Window |
|---|---|---|

### Update Policy
[exact KQL command]

### Operations Agent
[Recommended / Not needed] — [rationale]
```

## Fabric-Specific Rules (MUST follow)
- Run each `.create` / `.alter` KQL command **individually** — batch causes recognition errors
- Use `todatetime()`, `toreal()`, `toint()`, `tobool()` — JSON fields from Eventstream arrive as strings
- Update Policy JSON must be **single-line escaped** (`"[{\"...\"}]"`), NOT `@'...'` syntax
- Materialized views are queried via `materialized_view('mv_name')` function in KQL
- Eventstream Custom Endpoint uses Event Hub SDK (`azure-eventhub` Python package)
- Connection strings contain `SharedAccessKey` — NEVER commit to Git
