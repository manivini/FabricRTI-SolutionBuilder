# Skill 5: Documentation & Deployment Guide

## Purpose
Generate the setup guide notebook, README, CI/CD reference, and supporting files that complete the demo package.

## Input
- Use case spec from Skill 1
- Architecture document from Skill 2
- KQL schema filename from Skill 3
- Generator notebook filename from Skill 3
- Dashboard tile queries from Skill 4

## Output
1. `{UseCase}_SetupGuide.ipynb` — Step-by-step Fabric UI walkthrough
2. `README.md` — Project overview and quick start
3. `CICD_Patterns_Reference.md` — Git integration and deployment patterns
4. `.gitignore` — Standard ignores

---

## Part A: Setup Guide Notebook

### Structure (11 steps as markdown cells)

#### Cell 1 — Title & Business Context
```markdown
# {Industry} – {UseCase}
# Microsoft Fabric RTI – Setup Guide

## Business Context
[From use case spec — why_realtime, consumer_role, description]

## Architecture
[ASCII diagram from Skill 2]

## What You'll Build
| Fabric Item | Name |
|---|---|
| Eventhouse | {name from Skill 2} |
| Eventstream | {name from Skill 2} |
| Notebook | {name from Skill 3} |
| Dashboard | {name from Skill 4} |
| Data Activator | {name from Skill 4} |

## Checklist
- [ ] Step 1–11 items
```

#### Steps 1-9 (Standard — adapt names only)

| Step | Content |
|---|---|
| 1 | Log in & create Workspace |
| 2 | Create Eventhouse |
| 3 | Enable OneLake Availability |
| 4 | Create Eventstream with Custom Endpoint + copy connection details |
| 5 | Upload & run generator notebook |
| 6 | Define Eventstream topology → Bronze table |
| 7 | Run KQL schema script (with instructions to run blocks individually) |
| 8 | Build Real-Time Dashboard (include all tile queries from Skill 4) |
| 9 | Configure Data Activator crisis alert |

#### Step 10 — Git Integration
```markdown
## STEP 10 – Connect Fabric Workspace to GitHub
1. Settings → Git integration → GitHub
2. Authenticate → select repo → branch: main → folder: /
3. Connect and sync
```

#### Step 11 — Deployment Pipeline
```markdown
## STEP 11 – Set Up Deployment Pipeline (Dev → Prod)
1. Deployment Pipelines → New → assign workspaces
2. Deploy Dev → Prod
```

#### Final Cells
- Demo Flow Recommendation (talking points per tile)
- Stop the Generator instructions
- Troubleshooting table

### Troubleshooting Table (standard for all demos)
| Problem | Fix |
|---|---|
| Bronze table empty | Check generator is running; check Eventstream is Active |
| Silver table empty | Update Policy only fires on new ingest; wait for next Bronze insert |
| Crisis/anomaly view empty | Wait for crisis burst or check anomaly condition in Silver |
| Dashboard tiles empty | Set time range to Last 1 hour; check data source connection |
| Kernel timeout | Re-run all cells; increase session timeout in Run → Configure session |
| Eventstream inactive after capacity resume | Open Eventstream → Activate destination |
| `.clear table` blocked | Toggle OneLake availability OFF, clear, toggle back ON |

---

## Part B: README.md

### Template
```markdown
# Fabric RTI – {Industry} {UseCase}

{One-line description from spec}

## Architecture
{ASCII diagram from Skill 2}

## What's in the Repo
| File | Description |
|---|---|
| `{UseCase}_SetupGuide.ipynb` | Step-by-step Fabric UI walkthrough |
| `{UseCase}_Generator.ipynb` | Synthetic event generator |
| `{usecase}_createAll.kql` | KQL schema script |
| `CICD_Patterns_Reference.md` | Git + CI/CD patterns reference |

## Quick Start
1. Create Eventhouse + KQL Database (F2+ capacity)
2. Create Eventstream with Custom Endpoint
3. Copy Event Hub Name & Connection String
4. Import generator notebook → paste credentials → Run All
5. Run KQL blocks one-by-one
6. Create Real-Time Dashboard → add tiles

## Medallion Layers
| Layer | Table | Purpose |
|---|---|---|
| Bronze | Bronze{EventType} | Raw JSON from Eventstream |
| Silver | Silver{EventType} | Typed, enriched, anomaly-flagged |
| Gold | mv_{View1} | {Description} |
| Gold | mv_{View2} | {Description} |

## Demo Trigger
Run Cell 7 in the generator notebook to inject a crisis/anomaly burst on demand.

## Known Fabric KQL Quirks
- Run each `.create` / `.alter` block individually
- Use `todatetime()`, `toreal()`, `toint()`, `tobool()` casts
- Update Policy JSON: single-line escaped, not `@'...'`

## Git Integration
[Standard section — workspace → settings → Git integration]

## Deployment Pipelines
[Standard section — Dev → Prod promotion]

## Prerequisites
- Microsoft Fabric F2+ capacity (or Trial)
- Python packages: `azure-eventhub`, `faker`
- GitHub account (for Git integration)
```

---

## Part C: CICD_Patterns_Reference.md

Use the reference document from the PE demo as a template. It covers:
1. Three CI/CD mechanisms (Git Integration, Deployment Pipelines, fabric-cicd)
2. Architecture patterns (A, B, C)
3. Feature branch challenges
4. What syncs to Git
5. Security considerations
6. Decision matrix
7. Demo flow recommendation
8. Fabric Toolbox accelerator references

This file is **industry-agnostic** — copy as-is from the reference folder.

---

## Part D: .gitignore

Standard for all demos:
```
# Python
__pycache__/
*.py[cod]
*.egg-info/

# Jupyter
.ipynb_checkpoints/

# Environment
.env
.env.*
*.local

# OS
Thumbs.db
.DS_Store

# VS Code
.vscode/
*.code-workspace

# Fabric
.fabric/
```

---

## Pre-Demo Checklist (include in setup guide)

| # | Action | Time |
|---|---|---|
| 1 | Resume Fabric capacity | 2 min |
| 2 | Open generator notebook → Run Cells 1-6 | 3 min |
| 3 | Wait for `🚀 Generator running` message | 10 sec |
| 4 | Verify: `Bronze{Table} \| count` is growing | 30 sec |
| 5 | Open dashboard → confirm tiles updating | 30 sec |
| 6 | Wait 10-15 min for baseline data | 15 min |
| 7 | Cell 7 ready for live crisis trigger | - |

**Total lead time: ~20 min before demo**

---

## Post-Demo Cleanup
1. Stop generator: `GENERATOR_STATE['running'] = False` in a new cell
2. Optionally clear data: `.clear table` commands in KQL Queryset
3. Pause capacity in Azure Portal to stop billing
