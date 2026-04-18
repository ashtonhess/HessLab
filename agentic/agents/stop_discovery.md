# stop_discovery_agent

Finds and ranks route-worthy stops between anchors or within a drive radius, optimizing for trip theme, detour cost, stop spacing, and stop variety. This is the best helper when an orchestrator needs to turn a rough route into a day that feels intentionally packed rather than front-loaded or repetitive.

## Model

- **Model:** `gemini-3-flash`
- **Why:** This job is browse-heavy and ranking-heavy, but usually not deep-synthesis-heavy. A fast, cheap search model is the right fit when the orchestrator wants lots of candidates and clear pruning.

## Cost & latency

- **Cost profile:** cheap
- **Latency profile:** medium
- **Run in parallel:** yes — especially when exploring multiple route families or regions at once.

## Inputs

- Route anchors or region.
- Time budget or target trip duration.
- Trip theme.
- Preferred stop types and excluded stop types.
- Date / season / holiday context.
- Practical constraints such as parking friction, no long hikes, or family-friendliness.

## Outputs

- **Best Candidates** — ranked stops with why they are worth it.
- **Schedule Cost** — approximate time required or detour burden.
- **Practical Notes** — access, crowd risk, parking, opening-hour caveats.
- **Weak Candidates / Cut List** — useful stops that are still not worth using.
- **Sources** — URLs supporting the recommendations.

Full response format: see the canonical definition.

## Used by workflows

- [`research`](../workflows/research.md)

## Canonical definition

[`../../.cursor/agents/stop_discovery.md`](../../.cursor/agents/stop_discovery.md)

## Notes

- Best used early, before the final itinerary exists.
- This agent is about **curation**, not broad topic education.
- Particularly useful for route planning, city breaks, or any guide where stop quality matters more than sheer quantity.
