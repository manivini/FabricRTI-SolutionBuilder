# Skill 4: Dashboard & Alert Design

## Purpose
Design the Real-Time Dashboard tile queries and Data Activator alert configurations based on the architecture (Skill 2) and schema (Skill 3).

## Input
- Use case spec from Skill 1 (dashboard_tiles suggestions)
- Architecture document from Skill 2 (table names, Gold views)
- KQL schema from Skill 3 (exact field names and types)

## Output
1. Dashboard tile KQL queries (6-8 tiles)
2. Visual type recommendations per tile
3. Data Activator alert configuration
4. Dashboard setup instructions

---

## Dashboard Design Principles

### Layout Strategy
Every RTI dashboard should tell a story in this order (top-left to bottom-right):

| Position | Tile Type | Purpose |
|---|---|---|
| **Top row** | Stat cards (3-5) | At-a-glance KPIs — one per entity or one per key metric |
| **Middle left** | Time chart (line/area) | The "hero" tile — shows trends over time, makes crises visible |
| **Middle right** | Bar/pie chart | Distribution or breakdown (by category, channel, entity) |
| **Bottom left** | Table | Detailed records — crisis events, anomalies, top-N lists |
| **Bottom right** | Bar chart | Secondary metric (reach, volume, count by entity) |

### Time Range Parameters
All queries should use `_startTime` and `_endTime` parameters (built into Fabric Real-Time Dashboards):
```kql
| where timestamp between (_startTime .. _endTime)
```
This enables the time range picker in the dashboard toolbar.

### Auto-Refresh
Set to **30 seconds** for demos. Use **Continuous** for maximum responsiveness.

---

## Standard Tile Templates

### Tile 1 — KPI Stat Cards (one per entity)
```kql
Silver{EventType}
| where timestamp between (_startTime .. _endTime)
| summarize key_metric = round(avg(metricField), 2) by entityField
| order by entityField asc
```
- Visual: **Stat** (multi-stat or one card per entity)
- Add conditional formatting: green (good), yellow (warning), red (critical)
- Thresholds from `alert_thresholds` in the spec

### Tile 2 — Trend Over Time (Hero Tile)
```kql
Silver{EventType}
| where timestamp between (_startTime .. _endTime)
| summarize key_metric = avg(metricField) by entityField, bin(timestamp, 15m)
| render timechart
```
- Visual: **Line chart**
- X-axis: time, Y-axis: metric, Series: entity
- This is the **most important tile** — it shows the crisis/anomaly moment clearly

### Tile 3 — Volume by Entity
```kql
mv_{VolumeView}
| where timeBucket between (_startTime .. _endTime)
| summarize total = sum(countField) by entityField
| order by total desc
```
- Visual: **Bar chart** (horizontal)

### Tile 4 — Crisis/Anomaly Table
```kql
mv_{CrisisView}
| where timeBucket >= ago(2h)
| project entityField, timeBucket, crisis_count, top_categories, sources
| order by crisis_count desc
```
- Visual: **Table**
- This populates when a crisis fires — empty during normal operation

### Tile 5 — Distribution by Category/Channel
```kql
Silver{EventType}
| where timestamp between (_startTime .. _endTime)
| summarize count = count() by categoryField, labelField
```
- Visual: **Stacked bar chart** or **Pie chart**
- Use consistent colors: positive=green, neutral=gray, negative=orange/red

### Tile 6 — Top-N Items
```kql
Silver{EventType}
| where timestamp between (_startTime .. _endTime)
| where labelField == "negative" or isAnomaly == true
| mv-expand keywords
| summarize keyword_count = count() by tostring(keywords)
| top 10 by keyword_count
```
- Visual: **Table** or **Horizontal bar chart**

### Tile 7 — Regional/Dimensional Breakdown
```kql
Silver{EventType}
| where timestamp between (_startTime .. _endTime)
| summarize key_metric = avg(metricField), count = count() by regionField
```
- Visual: **Table** or **Map** (if geo-coordinates available)

### Tile 8 — Impact/Reach Metric
```kql
mv_{VolumeView}
| where timeBucket between (_startTime .. _endTime)
| summarize total_impact = sum(impactField) by entityField
| order by total_impact desc
```
- Visual: **Bar chart**

---

## Adapting Tiles to Use Case Type

### Sentiment Monitoring
- Stat: avg sentiment score per entity
- Hero: sentiment trend line
- Crisis table: negative sentiment surges
- Distribution: sentiment by channel

### Fraud Detection
- Stat: count of flagged transactions, total $ at risk
- Hero: flagged transaction rate over time
- Crisis table: high-risk transactions queue
- Distribution: fraud by channel/type

### Operational Monitoring
- Stat: uptime %, error rate, avg response time
- Hero: error rate or response time over time
- Crisis table: active incidents/outages
- Distribution: errors by system/component

### IoT / Manufacturing
- Stat: sensor readings, production rate
- Hero: sensor values over time with threshold lines
- Crisis table: anomaly events
- Distribution: alerts by machine/line

---

## Data Activator (Reflex) Alert Configuration

### Standard Alert Setup
1. From the dashboard, click **...** on the Crisis/Anomaly tile → **Set alert**
2. Configure:

| Field | Value |
|---|---|
| Check | On each event grouped by |
| Grouping field | `entityField` (e.g., `portfolioCompany`, `accountId`) |
| When | `crisis_count` or `risk_score` |
| Condition | Becomes greater than |
| Value | [from `alert_thresholds` in spec] |
| Action | **Message me in Teams** |

3. Name: `{UseCase}Alert` → **Create**

### Alert Design Guidelines
- Use thresholds from the use case spec — they have business justification
- Group by entity so each entity triggers independently
- For fraud: alert on individual high-risk transactions (not just aggregate)
- For sentiment: alert on crisis_count exceeding threshold per entity
- For operations: alert on error_rate exceeding % threshold or response_time exceeding SLA

### Advanced Actions (mention in demo)
- Power Automate: create ServiceNow ticket, send email to C-suite
- Teams Adaptive Card: rich notification with context
- API webhook: trigger external system

---

## Color Scheme Recommendations

| Label/State | Color | Hex |
|---|---|---|
| Positive / Normal / Low Risk | Green | `#2ECC71` |
| Neutral / Medium | Gray | `#95A5A6` |
| Negative / High Risk / Alert | Orange/Red | `#E74C3C` |
| Crisis / Critical | Dark Red | `#C0392B` |

Apply consistently across all tiles for a cohesive dashboard appearance.
