# Skill 1: Industry & Use Case Discovery

## Purpose
Conduct an interactive discovery conversation to understand the customer's industry, identify relevant real-time intelligence use cases, and produce a structured use case specification that subsequent skills can use to build a complete Fabric RTI demo.

## When to Use This Skill
- A solution engineer needs to build an RTI demo for a new customer
- The customer's industry is known but the specific use case hasn't been decided
- You need to explore what real-time monitoring scenarios resonate with the customer

## Interaction Style

**Be conversational, not transactional.** Do NOT dump all industries and use cases at once. Guide the user step by step, one question at a time. Wait for their answer before proceeding.

**Start with a SHORT introduction** — 2-3 sentences max, then ask the first question:

> "I can help you build a complete Fabric Real-Time Intelligence demo — streaming data, live dashboards, automated alerts, all ready to deploy. What industry is your customer in?"

Do NOT show tables of skills, file lists, or process steps in the intro. Keep it tight.

## Interaction Flow

### Step 1 — Ask for the Industry (one question only)
Ask:
> "What industry is your customer in?"

Do NOT list all available industries. Let the user tell you. If they're unsure, then offer: "I have templates for Banking, Insurance, Manufacturing, Healthcare, Retail, and Shipping. Which is closest?"

### Step 2 — Ask if They Have a Use Case in Mind
Ask:
> "Do you already have a real-time use case in mind, or would you like me to suggest some relevant ones for [Industry]?"

If they have one → skip to Step 4.
If they want suggestions → go to Step 3.

### Step 3 — Present Use Cases ONE AT A TIME (not a big list)
Check `industry-templates/` for a matching template.
- If found: Present the **top use case first** with a 2-line description and why real-time matters.
- Ask: "Does this resonate? Or shall I show you another option?"
- Only show the next use case if they say no.
- Do NOT dump all 4-5 use cases in one message.

If no template found: "I don't have a pre-built template for [Industry] yet. I can brainstorm with you, or you can generate a detailed template using the Researcher prompt in `reference/researcher-prompt.md`. Which do you prefer?"

### Step 4 — Deep Dive on Selected Use Case
Once the user picks a use case, extract from the template (or brainstorm if no template):
- Event schema per stream
- Bronze → Silver transformations
- Silver → Gold aggregations
- Fictional entities to monitor
- Crisis/anomaly scenarios
- Dashboard tile suggestions

### Step 5 — Customize for the Customer
Ask:
- "Any preferences for company/entity names? Or shall I generate fictional ones?"
- "What region should the demo focus on?" (default: APAC)
- "How many entities to monitor?" (default: 4)
- "Any specific crisis scenarios you want to highlight?"

### Step 6 — Produce the Use Case Spec
Output a structured specification that Skills 2-5 can consume. Format:

```yaml
use_case_spec:
  industry: "[Industry]"
  use_case_title: "[Title]"
  use_case_description: "[Description]"
  why_realtime: "[Why real-time matters]"
  consumer_role: "[Who uses the output]"
  region: "[Primary region]"
  
  entities:
    - name: "[Entity 1]"
      product: "[Product/Brand]"
      sector: "[Sub-sector]"
      crisis_issues: ["issue1", "issue2", "issue3"]
      crisis_keywords: ["keyword1", "keyword2", ...]
    - name: "[Entity 2]"
      ...
  
  event_streams:
    - stream_name: "[Stream 1]"
      description: "[What this stream contains]"
      fields:
        - name: "[field]"
          type: "[type]"
          description: "[description]"
      estimated_volume: "[events/sec]"
    - stream_name: "[Stream 2]"
      ...
  
  channels_or_sources: ["source1", "source2", ...]
  issue_categories: ["cat1", "cat2", ...]
  
  bronze_to_silver:
    - "[Transformation 1]"
    - "[Transformation 2]"
  
  gold_aggregations:
    - name: "[Aggregation 1]"
      description: "[What it computes]"
    - name: "[Aggregation 2]"
      ...
  
  crisis_scenarios:
    - title: "[Scenario 1]"
      trigger: "[Condition]"
      affected_entities: ["Entity1"]
    - ...
  
  macro_events:
    - title: "[Event 1]"
      description: "[What happens]"
      affected_sectors: ["sector1"]
    - ...
  
  alert_thresholds:
    - metric: "[What to watch]"
      condition: "[Threshold]"
      justification: "[Why this threshold]"
    - ...
  
  dashboard_tiles:
    - title: "[Tile 1]"
      visual_type: "[chart type]"
      query_description: "[What the query does]"
    - ...
  
  regulatory_considerations: "[Key regulations to be aware of]"
```

## Key Principles
- **Be open-ended** — don't constrain to predefined patterns. Let the use case drive the architecture.
- **Use industry templates as starting points**, not rigid structures. The user may want something different.
- **Ask, don't assume** — if the user's needs don't match a template, adapt.
- **Think broadly** — RTI is not just sentiment. Consider operational monitoring, fraud, IoT, market signals, compliance, customer experience.
- **Produce a complete spec** — Skills 2-5 depend on this output. Missing fields cause downstream problems.

## If No Template Exists

When the user asks for an industry that doesn't have a template in `industry-templates/`, offer TWO options:

### Option A — Agent Brainstorms (faster, less detailed)
> "I don't have a pre-researched template for [Industry], but I can brainstorm use cases based on my general knowledge. The output won't be as detailed as the researched templates, but it'll be enough to generate a working demo. Want me to go ahead?"

If yes:
1. Suggest 3-4 RTI use cases for the industry (think broadly — not just sentiment)
2. Present them one at a time (per interaction style rules)
3. Once user picks one, brainstorm the event schema, entities, crisis scenarios, etc.
4. Fill in the use case spec as best you can — flag any areas where you're less confident
5. Proceed to Skill 2

**The output quality will be good but not as deep as Researcher-backed templates.** This is fine for quick demos or initial exploration.

### Option B — User Runs Researcher Prompt (slower, much more detailed)
> "For the best results, I'd recommend running a research prompt to generate a detailed template first. Here's the prompt — copy it and run it with M365 Copilot Researcher (or any research-capable AI):"

Then display the prompt from `reference/researcher-prompt.md` with [INDUSTRY] already replaced.

> "Save the output as `industry-templates/[industry].md` in this project, then come back and tell me it's ready. I'll use it to generate a much richer demo."

### Option C — Hybrid (recommended for unfamiliar industries)
> "I can brainstorm some initial use cases now so you can evaluate the direction. If you like one, you can then run the Researcher prompt focused on that specific use case for deeper data schemas and scenarios. This gives you the best of both approaches."

**Always let the user choose.** Don't force one path.

Then save the output as a new template in `industry-templates/[industry].md` for future reuse.
