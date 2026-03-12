# Fabric KQL Quirks & Gotchas

Hard-won lessons from building Fabric RTI demos. Reference this when generating KQL schema scripts.

## Command Execution
- **Run each `.create` / `.alter` command individually** — running multiple management commands at once causes `Recognition Error` in Fabric KQL editor
- Copy-paste one block at a time, run, wait for completion, then next block

## Data Types
- **All fields from Eventstream arrive as strings** — even numbers, booleans, and dates
- Always cast in the transformation function:
  - `todatetime(fieldName)` for timestamps
  - `toreal(fieldName)` for decimals/floats
  - `toint(fieldName)` for integers
  - `tobool(fieldName)` for booleans
- Without casting, `avg()`, `sum()`, `bin()` will fail with semantic errors

## Update Policy Syntax
- **Use single-line escaped JSON** for update policy:
  ```kql
  .alter table MyTable policy update "[{\"IsEnabled\":true,\"Source\":\"SourceTable\",\"Query\":\"TransformFunction()\",\"IsTransactional\":false,\"PropagateIngestionProperties\":false}]"
  ```
- **Do NOT use `@'...'` multi-line string syntax** — Fabric does not support it (even though standard Kusto does)

## Materialized Views
- Query materialized views using `materialized_view('mv_name')` for guaranteed fresh results
- Direct table-style query also works but may return slightly stale data
- Materialized views auto-refresh — no manual scheduling needed

## OneLake Mirroring
- If OneLake Availability is enabled, `.clear table` is blocked
- Fix: Toggle OneLake off → clear → toggle back on
- Or use: `.clear table MyTable data with (forceMirroringContinuousExportOperation=true)` (may not work in all Fabric versions)

## Eventstream
- Custom Endpoint uses Event Hub protocol (`azure-eventhub` Python SDK)
- Connection string from Eventstream Keys tab — contains `SharedAccessKey`
- After pausing/resuming Fabric capacity, Eventstream destinations may become inactive — reactivate manually
- Connection strings may rotate on capacity pause/resume — always verify

## Notebooks
- `pip install` packages are lost on kernel restart — always re-run the install cell
- Fabric kernel times out after ~20-60 min of inactivity — generator thread dies
- Increase session timeout: Run → Configure session → set to 60-120 min
- Background threads (daemon=True) run while kernel is alive but don't prevent timeout

## Dashboard
- Auto-refresh: set to 30 seconds for demos, Continuous for maximum responsiveness
- Use `_startTime` and `_endTime` parameters in tile queries for time range picker
- Color customization is in the visual config panel per tile (no global theme)
- No native word cloud visual — use horizontal bar chart of top keywords instead

## Git Integration
- Fabric syncs items by **name** — if notebook name in Fabric ≠ filename in Git, they won't match
- Fabric stores notebooks in its own folder format (`ItemName/notebook-content.py`), separate from root `.ipynb` files
- Changes made in VS Code to root `.ipynb` may not sync to Fabric if the item uses the Fabric folder format
- Safest: edit in Fabric → Commit to Git (not the other way, unless you understand the folder structure)
