# Skill 3: Schema & Code Generation

## Purpose
Generate the KQL schema script and Python generator notebook from the architecture design (Skill 2) and use case spec (Skill 1).

## Input
- Use case spec from Skill 1
- Architecture document from Skill 2

## Output
1. `{usecase}_createAll.kql` — Complete KQL schema script
2. `{UseCase}_Generator.ipynb` — Fabric notebook for synthetic data generation

---

## Part A: KQL Schema Script (`{usecase}_createAll.kql`)

### File Structure
The KQL file must contain commands in this exact order (each run separately in Fabric):

```
// Block 1: Silver Table
.create table Silver{EventType} (...)

// Block 2: Helper Functions
.create-or-alter function {HelperFunction}(...) { ... }

// Block 3: Transformation Function
.create-or-alter function Transform{EventType}() { ... }

// Block 4: Update Policy
.alter table Silver{EventType} policy update "..."

// Block 5+: Materialized Views
.create materialized-view mv_{Name} on table Silver{EventType} { ... }

// Block N: Gold Functions (optional)
.create-or-alter function {GoldFunction}() { ... }
```

### Silver Table Creation
Map Silver schema from Skill 2 architecture:
```kql
.create table Silver{EventType} (
    field1: datetime,
    field2: string,
    field3: real,
    ...
)
```

### Helper Functions
Common pattern — a bucket/label function:
```kql
.create-or-alter function {BucketFunction}(score: real) {
    case(
        score <= -0.6, "Very Negative",
        score <= -0.2, "Negative",
        score <= 0.2,  "Neutral",
        score <= 0.6,  "Positive",
        "Very Positive"
    )
}
```

Adapt the buckets/labels to the use case (e.g., risk levels for fraud, severity for incidents).

### Transformation Function
This is the core Bronze → Silver logic. MUST follow these rules:

1. **Cast all JSON string fields** to proper types using `todatetime()`, `toreal()`, `toint()`, `tobool()`
2. **Use a single `extend` block** for all computed fields
3. **End with `project`** to select only Silver columns

```kql
.create-or-alter function Transform{EventType}() {
    Bronze{EventType}
    | extend
        timestamp     = todatetime(timestamp),
        numericField  = toreal(numericField),
        intField      = toint(intField),
        boolField     = tobool(boolField),
        computedLabel = {BucketFunction}(toreal(scoreField)),
        isAnomaly     = (toreal(scoreField) <= threshold) and (category in ("cat1", "cat2")),
        timeBucket    = bin(todatetime(timestamp), 1h)
    | project
        timestamp, field1, field2, ..., computedLabel, isAnomaly, timeBucket
}
```

### Update Policy
MUST be single-line escaped JSON:
```kql
.alter table Silver{EventType} policy update "[{\"IsEnabled\":true,\"Source\":\"Bronze{EventType}\",\"Query\":\"Transform{EventType}()\",\"IsTransactional\":false,\"PropagateIngestionProperties\":false}]"
```

> NEVER use `@'...'` syntax — Fabric does not support it.

### Materialized Views
One per Gold aggregation from Skill 2:
```kql
.create materialized-view mv_{Name} on table Silver{EventType} {
    Silver{EventType}
    | summarize 
        metric1 = avg(field), 
        metric2 = count(), 
        metric3 = sum(field)
      by dimension1, dimension2, bin(timestamp, 1h)
}
```

### Gold Functions (Optional)
Convenience functions that filter Silver by category:
```kql
.create-or-alter function {CategoryView}() {
    Silver{EventType}
    | where category in ("cat1", "cat2")
    | project timestamp, field1, field2, ...
}
```

---

## Part B: Generator Notebook (`{UseCase}_Generator.ipynb`)

### Notebook Structure (13-15 cells)

| Cell # | Type | Content |
|---|---|---|
| 1 | Markdown | Title, demo quick-start instructions |
| 2 | Markdown + Code | Install dependencies: `!pip install azure-eventhub faker --quiet` |
| 3 | Markdown + Code | Configuration: EVENT_HUB_NAME, EVENT_HUB_CONN_STRING (placeholders) |
| 4 | Markdown + Code | Entity definitions, content templates, macro events |
| 5 | Markdown + Code | Event generator functions |
| 6 | Markdown + Code | Start background streaming thread |
| 7 | Markdown + Code | Demo trigger cell |

### Cell Details

#### Cell 1 — Title & Quick-Start
```markdown
# {Industry} – {UseCase}: Synthetic Event Generator

Streams synthetic events for {N} entities into Fabric via Eventstream Custom Endpoint.
**Upload and run inside Microsoft Fabric.**

---
## ⚡ Demo Quick-Start
1. Run Cell 2 – install packages
2. Run Cell 3 – paste connection string  
3. Run Cell 4 – load definitions
4. Run Cell 5 – load generator functions
5. Run Cell 6 – starts background stream
6. Run Cell 7 – demo trigger (crisis/anomaly on demand)
```

#### Cell 3 — Configuration
```python
EVENT_HUB_NAME        = "<paste-your-event-hub-name>"
EVENT_HUB_CONN_STRING = "<paste-your-connection-string>"
EVENTS_PER_BATCH      = 8
INTERVAL_SECONDS      = 3
AUTO_CRISIS_ENABLED   = True
AUTO_MACRO_ENABLED    = True
```

#### Cell 4 — Entity & Content Definitions

Use the `entities` from the use case spec:
```python
ENTITIES = [
    {"name": "Entity1", "product": "Product1", "sector": "Sector1",
     "crisis_issues": ["issue1", "issue2"],
     "crisis_keywords": ["keyword1", "keyword2", ...]},
    ...
]
```

Define message templates using industry-appropriate language from the spec:
```python
POSITIVE_MSGS = ["...", "...", "..."]
NEUTRAL_MSGS  = ["...", "...", "..."]  
NEGATIVE_MSGS = ["...", "...", "..."]
CRISIS_MSGS   = ["...", "...", "..."]
```

Define macro events from the spec:
```python
MACRO_EVENTS = [
    {"id": "macro1", "label": "...", "sectors": ["..."],
     "issue": "...", "keywords": ["..."],
     "messages": ["...", "..."]},
    ...
]
```

#### Cell 5 — Generator Functions

**Score distribution (normal mode)**:
- Positive: 35% → score range 0.3 to 1.0
- Neutral: 50% → score range 0.05 to 0.35
- Negative: 15% → score range -0.8 to -0.2
- Crisis: score range -1.0 to -0.6

> These weights ensure a clearly positive baseline that makes crisis drops visually dramatic on the dashboard.

**For non-sentiment use cases** (fraud, IoT), adapt the score to the relevant metric:
- Transaction risk score: normal 0-30, suspicious 60-80, fraud 85-100
- Sensor reading: normal within range, anomaly = outside ±2 standard deviations
- Response time: normal 50-200ms, degraded 500-1000ms, outage >5000ms

**Event generation function must produce a JSON dict** with all Bronze schema fields.

**Background thread pattern**:
```python
import threading, time

GENERATOR_STATE = {
    "running": True,
    "force_crisis_now": False,
    "total_sent": 0, "crisis_triggered": 0
}

def _loop():
    while GENERATOR_STATE["running"]:
        batch = [generate_event() for _ in range(EVENTS_PER_BATCH)]
        send_batch(batch)
        GENERATOR_STATE["total_sent"] += len(batch)
        time.sleep(INTERVAL_SECONDS)

thread = threading.Thread(target=_loop, daemon=True)
thread.start()
```

#### Cell 6 — Start Streaming
Instantiates the EventHubProducerClient and starts the background thread.
Must print: `🚀 Generator running in background`

#### Cell 7 — Demo Trigger
```python
TRIGGER_TYPE   = "macro"       # or "crisis"
TRIGGER_TARGET = "macro_id"    # from MACRO_EVENTS
# Sets GENERATOR_STATE flags → generator fires burst within ~3 seconds
```

### Generator Rules
- All events must have `eventId` (UUID), `eventDate` (ISO datetime), and all Bronze schema fields
- Use `faker` library for realistic names, locations, text
- Sources/channels must match what's in the use case spec
- Crisis events have higher reach/volume and strongly negative scores
- Macro events affect specific entity sectors (from spec)
- Background thread runs as daemon — kernel stays free for user interaction

---

## Fabric-Specific Code Rules
- Use `azure-eventhub` EventHubProducerClient to send events
- Connection string format: `Endpoint=sb://...;SharedAccessKeyName=...;SharedAccessKey=...;EntityPath=...`
- Send events as JSON via `EventData(json.dumps(event).encode('utf-8'))`
- Always use placeholders for connection strings — NEVER hardcode real secrets
- `faker` must be installed via pip every session (Fabric kernel doesn't persist packages)
