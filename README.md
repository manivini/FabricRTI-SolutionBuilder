# Fabric RTI Demo Builder

A reusable template for solution engineers to quickly build Microsoft Fabric Real-Time Intelligence (RTI) demos for any industry.

## What This Is

A set of **AI agent skills** + **industry templates** that, when used with GitHub Copilot in VS Code, can generate a complete RTI demo in ~30 minutes — including KQL schema, synthetic data generator, real-time dashboard, setup guide, and CI/CD documentation.

## How to Use

### Option A — Use with GitHub Copilot in VS Code
1. Clone this repo (or use as GitHub Template)
2. Open in VS Code with GitHub Copilot enabled
3. Open **Copilot Chat** and type your first message with `@workspace`:
   ```
   @workspace I need to build an RTI demo for a banking customer
   ```
   > **Important**: Start your first message with `@workspace` — this tells Copilot to read the skill files and industry templates from this project. After the first message, you can type normally in the same chat thread.
4. Copilot will guide you step by step — asking for industry, use case, then generating all files
5. Import generated files to Fabric and follow the setup guide

### Option B — Manual (without Copilot)
1. Browse `industry-templates/` → pick your industry
2. Choose a use case from the template
3. Follow the skills (`skills/skill-1` through `skill-5`) as a step-by-step guide
4. Use the KQL patterns in `reference/fabric-kql-quirks.md` to avoid common pitfalls

## Repo Structure

```
FabricRTI-DemoBuilder/
├── .github/prompts/
│   └── create-rti-demo.prompt.md    ← Main Copilot prompt (auto-discovered)
├── skills/
│   ├── skill-1-discovery.md         ← Industry & use case discovery
│   ├── skill-2-architecture.md      ← Fabric RTI architecture design
│   ├── skill-3-schema-codegen.md    ← KQL schema + Python generator
│   ├── skill-4-dashboard-alerts.md  ← Dashboard tiles + Data Activator
│   └── skill-5-docs-deployment.md   ← Setup guide, README, CI/CD
├── industry-templates/
│   ├── banking.md                   ← 5 use cases, 2 deep dives
│   ├── insurance.md                 ← 5 use cases, 2 deep dives
│   ├── payments.md                  ← (coming soon)
│   ├── manufacturing.md             ← (coming soon)
│   ├── shipping.md                  ← (coming soon)
│   ├── retail.md                    ← (coming soon)
│   └── healthcare.md                ← (coming soon)
├── reference/
│   ├── fabric-kql-quirks.md         ← Critical Fabric-specific gotchas
│   └── researcher-prompt.md         ← Prompt to generate new templates
├── examples/
│   └── (links to completed demo repos)
└── README.md                        ← This file
```

## Skills Overview

| Skill | Purpose | Input | Output |
|---|---|---|---|
| **1 — Discovery** | Identify industry + use case | User conversation | Structured use case spec |
| **2 — Architecture** | Design Fabric RTI architecture | Use case spec | Architecture document, item names, schemas |
| **3 — Schema & Code** | Generate KQL + Python notebook | Architecture doc | `.kql` file + `.ipynb` generator |
| **4 — Dashboard** | Design tiles + alerts | Schema + architecture | Tile queries, alert config |
| **5 — Docs** | Generate setup guide + README | All above | Setup notebook, README, CI/CD ref |

## Industry Templates

Pre-researched by deep-thinking AI (M365 Copilot Researcher). Each template contains:
- 4-5 real-time use cases with business context
- Deep dive on top 2: event schemas, Bronze→Silver→Gold design
- Crisis/anomaly scenarios with trigger conditions
- Dashboard tile suggestions
- Regulatory considerations

### Adding a New Industry
1. Open `reference/researcher-prompt.md`
2. Copy the prompt, replace `[INDUSTRY]` with your industry
3. Run with M365 Copilot Researcher (or any research-capable AI)
4. Save output as `industry-templates/{industry}.md`
5. Use with skills as normal

## What Gets Generated (Per Demo)

| File | Description |
|---|---|
| `{usecase}_createAll.kql` | KQL schema — tables, functions, update policy, materialized views |
| `{UseCase}_Generator.ipynb` | Synthetic data generator — background thread, demo trigger |
| `{UseCase}_SetupGuide.ipynb` | 11-step Fabric UI walkthrough |
| `README.md` | Project overview, architecture, quick start |
| `CICD_Patterns_Reference.md` | Git integration + deployment pipeline patterns |
| `.gitignore` | Standard Python + Fabric ignores |

## Examples

| Demo | Industry | Use Case | Repo |
|---|---|---|---|
| PE Sentiment Monitoring | Private Equity | Customer sentiment across portfolio | [FabricRTI_PE_Sentiment](https://github.com/manivini/FabricRTI_PE_Sentiment) |

## Prerequisites

- Microsoft Fabric F2+ capacity (or Trial)
- VS Code + GitHub Copilot (for AI-assisted generation)
- Python packages in Fabric notebook: `azure-eventhub`, `faker`
- GitHub account (for Git integration)

## Contributing

To add a new industry template:
1. Run the Researcher prompt for the new industry
2. Save as `industry-templates/{industry}.md`
3. Submit a PR

To improve a skill:
1. Test the skill against 2-3 industry templates
2. Update the skill `.md` file
3. Submit a PR with before/after examples
