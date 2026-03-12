# Fabric RTI Solution Builder

A reusable template to quickly build Microsoft Fabric Real-Time Intelligence (RTI) solutions for any industry.

## What This Is

A set of **AI agent skills** + **industry templates** that, when used with GitHub Copilot in VS Code, can generate a complete RTI solution in ~30 minutes - including KQL schema, synthetic data generator, real-time dashboard, setup guide, and CI/CD documentation.

## How to Use

### Option A - Use with GitHub Copilot in VS Code
1. Use this template (or clone this repo)
2. Open in VS Code with GitHub Copilot enabled
3. Open **Copilot Chat** and type your first message with `@workspace`:
   ```
   @workspace I need to build an RTI solution for a banking customer
   ```
   > **Important**: Start your first message with `@workspace` - this tells Copilot to read the skill files and industry templates from this project. After the first message, you can type normally in the same chat thread.
4. Copilot will guide you step by step - asking for industry, use case, then generating all files
5. Import generated files to Fabric and follow the setup guide

### Option B - Manual (without Copilot)
1. Browse `industry-templates/` - pick your industry
2. Choose a use case from the template
3. Follow the skills (`skills/skill-1` through `skill-5`) as a step-by-step guide
4. Use the KQL patterns in `reference/fabric-kql-quirks.md` to avoid common pitfalls

## Repo Structure

```
FabricRTI-SolutionBuilder/
+-- .github/prompts/
|   +-- create-rti-demo.prompt.md      <- Main Copilot prompt (auto-discovered)
+-- skills/
|   +-- skill-1-discovery.md           <- Industry and use case discovery
|   +-- skill-2-architecture.md        <- Fabric RTI architecture design
|   +-- skill-3-schema-codegen.md      <- KQL schema + Python generator
|   +-- skill-4-dashboard-alerts.md    <- Dashboard tiles + Data Activator
|   +-- skill-5-docs-deployment.md     <- Setup guide, README, CI/CD
+-- industry-templates/
|   +-- banking.md                     <- Fraud detection, AML, CX, trading, ATM monitoring
|   +-- insurance.md                   <- Claims, underwriting, telematics, policyholder CX
|   +-- manufacturing.md               <- Predictive maintenance, quality, OEE, supply chain
|   +-- retail.md                      <- Customer experience, inventory, fraud, demand sensing
|   +-- shipping.md                    <- Fleet tracking, port operations, delivery monitoring
|   +-- healthcare.md                  <- Patient experience, clinical monitoring, pharma
|   +-- privateequity.md              <- Portfolio monitoring, sentiment, deal sourcing, fraud
+-- reference/
|   +-- fabric-kql-quirks.md           <- Critical Fabric-specific gotchas
|   +-- researcher-prompt.md           <- Prompt to generate new industry templates
+-- examples/
|   +-- manufacturing-predictive-maintenance/  <- Complete generated example
+-- README.md                          <- This file
```

## Skills Overview

| Skill | Purpose | Input | Output |
|---|---|---|---|
| **1 - Discovery** | Identify industry + use case | User conversation | Structured use case spec |
| **2 - Architecture** | Design Fabric RTI architecture | Use case spec | Architecture document, item names, schemas |
| **3 - Schema and Code** | Generate KQL + Python notebook | Architecture doc | .kql file + .ipynb generator |
| **4 - Dashboard** | Design tiles + alerts | Schema + architecture | Tile queries, alert config |
| **5 - Docs** | Generate setup guide + README | All above | Setup notebook, README, CI/CD ref |

## Industry Templates

Pre-researched by deep-thinking AI (M365 Copilot Researcher). Each template contains:
- 4-5 real-time use cases with business context
- Deep dive on top 2: event schemas, Bronze to Silver to Gold design
- Crisis/anomaly scenarios with trigger conditions
- Dashboard tile suggestions
- Regulatory considerations

### Adding a New Industry
1. Open `reference/researcher-prompt.md`
2. Copy the prompt, replace `[INDUSTRY]` with your industry
3. Run with M365 Copilot Researcher (or any research-capable AI)
4. Save output as `industry-templates/{industry}.md`
5. Use with skills as normal

## What Gets Generated

| File | Description |
|---|---|
| `{usecase}_createAll.kql` | KQL schema - tables, functions, update policy, materialized views |
| `{UseCase}_Generator.ipynb` | Synthetic data generator - background thread with crisis/anomaly trigger |
| `{UseCase}_SetupGuide.ipynb` | 11-step Fabric UI walkthrough |
| `README.md` | Project overview, architecture, quick start |
| `CICD_Patterns_Reference.md` | Git integration + deployment pipeline patterns |
| `.gitignore` | Standard Python + Fabric ignores |

## Examples

| Solution | Industry | Use Case |
|---|---|---|
| Manufacturing Predictive Maintenance | Manufacturing | IoT sensor monitoring, equipment health, anomaly detection |

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
2. Update the skill .md file
3. Submit a PR with before/after examples