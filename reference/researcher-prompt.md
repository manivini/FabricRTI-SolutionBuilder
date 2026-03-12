# Researcher Prompt — Generate Industry RTI Template

Use this prompt with a research-capable AI (such as M365 Copilot Researcher, ChatGPT with web browsing, or similar) to generate a detailed industry template for a new industry.

Save the output as `industry-templates/{industry}.md` in this repo.

---

## Prompt

> I am building reusable demo templates for Microsoft Fabric Real-Time Intelligence (RTI). RTI enables streaming data ingestion via Eventstream, real-time transformation in KQL (Eventhouse), live dashboards, and automated alerts.
>
> For the **[INDUSTRY]** sector, suggest **4-5 real-time intelligence use cases** that would be compelling for a [INDUSTRY] company. Think broadly — not just sentiment monitoring. Consider operational monitoring, fraud/anomaly detection, IoT, market signals, compliance, customer experience, etc.
>
> For each use case:
> 1. Title and 2-line description
> 2. Why real-time matters (vs batch/daily)
> 3. Data sources (what systems/channels generate the events)
> 4. Who consumes the output (business role)
> 5. Estimated event volume
>
> For the **top 2 use cases**, deep dive:
> 1. Complete event schema per stream (field name, data type, description) — as markdown tables
> 2. What transformations/enrichments to apply (Bronze → Silver)
> 3. What aggregations to pre-compute (Silver → Gold)
> 4. 4 fictional entities to monitor
> 5. 5 anomaly/crisis scenarios with trigger conditions
> 6. 3 industry-wide macro events
> 7. Alert thresholds with business justification
> 8. Dashboard tile suggestions (6-8 tiles with query descriptions)
> 9. Relevant regions and regulatory considerations
>
> Format schemas as markdown tables. Use clear headers and bullet points for everything else.

---

## Industries Already Templated
- [x] Banking (`banking.md`)
- [x] Insurance (`insurance.md`)
- [ ] Payments / Fintech
- [ ] Manufacturing
- [ ] Shipping / Logistics
- [ ] Retail / E-Commerce
- [ ] Healthcare
- [ ] Private Equity (reverse-engineer from existing PE demo)
