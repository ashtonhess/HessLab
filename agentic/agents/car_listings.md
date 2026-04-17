# car_listings_agent

Searches major US car listing platforms (CarGurus, TrueCar, Cars.com, AutoTrader, AutoTempest, dealer sites) for vehicles matching a set of criteria and returns **direct links to individual listings** — not search-result pages.

## Model

- **Model:** `gemini-3-flash`
- **Why:** Task is web-heavy (many searches, many fetches, light reasoning per page). Flash is cheap, fast, and competent at structured extraction.

## Cost & latency

- **Cost profile:** cheap
- **Latency profile:** medium (bounded by web fetches)
- **Run in parallel:** **yes** — dispatch one instance per (model, year-range) tuple, or one per platform, for large fan-out.

## Inputs

- Make, model, optional trim.
- Year range.
- Max price.
- Required features (hard filter) and nice-to-have features (soft filter, flag if missing).
- Location exclusions (e.g., "no NYC metro," "no LA").
- Optional mileage ceiling.

## Outputs

Structured table of verified listings with: price, year, mileage, location, VIN (when available), confirmed features, and a direct link to the individual listing page. Plus a notes section for excluded listings and features that still need dealer verification.

Full response format: see the canonical definition.

## Used by workflows

- [`car_search`](../workflows/car_search.md)

## Canonical definition

[`../../.cursor/agents/car_listings.md`](../../.cursor/agents/car_listings.md)

## Notes

- **Direct links only.** If the agent returns a search-results URL instead of an individual listing, that is a failure — re-dispatch with a sharper query.
- **VINs are valuable.** They let the orchestrator cross-check the same car across platforms and deduplicate.
- **Location filtering matters.** Big-city listings often have misleading low miles (mostly city stop-and-go). Exclusions are load-bearing.
- **Feature verification is non-trivial.** Adaptive cruise, in particular, is often packaged (e.g., BMW ZPP). The agent should mark features as "confirmed in listing" vs. "needs dealer verification" rather than assuming.
- **Parallelize aggressively.** This agent is the primary fan-out point for any search-heavy workflow.
