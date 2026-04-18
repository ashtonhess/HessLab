# local_food_curation_agent

Builds a route-aware food plan that turns a trip into a real regional experience instead of a generic one. It picks where the main meal should happen, what the signature dish is, what quick bites are actually worth the stop, and what fallback options exist if the primary choice is crowded or closed.

## Model

- **Model:** `gemini-3-flash`
- **Why:** Food curation is browse-heavy and pattern-driven. It benefits from lots of quick source checks more than from heavyweight synthesis.

## Cost & latency

- **Cost profile:** cheap
- **Latency profile:** medium
- **Run in parallel:** yes — especially by region or meal slot.

## Inputs

- Route or stop list.
- Meal slots.
- Budget guidance.
- Cuisine preferences or restrictions.
- Queue / reservation tolerance.
- Date and crowd context.

## Outputs

- **Main Meal Stop** — best primary food destination and what to order.
- **Backup Option** — the fallback if the primary place is weak or crowded.
- **Quick Bites** — worth-it snack stops and what to get.
- **Hours / Risk Notes** — reservation, queue, or holiday issues.
- **Sources** — URLs grounding the recommendations.

Full response format: see the canonical definition.

## Used by workflows

- [`research`](../workflows/research.md)

## Canonical definition

[`../../.cursor/agents/local_food_curation.md`](../../.cursor/agents/local_food_curation.md)

## Notes

- Best used once a route skeleton already exists.
- If a route has several equally good food areas, this agent should help the orchestrator pick the one that best fits the day’s pacing.
- This agent should prefer local specialties over generic convenience unless the user explicitly values speed over depth.
