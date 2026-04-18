# traffic_risk_synthesis_agent

Synthesizes congestion, closure, holiday timing, weather dependence, and route fragility into a practical go/no-go recommendation. It is the agent you use when a route might look good on paper but could fail in real life because of bottlenecks, event traffic, or a bad departure window.

## Model

- **Model:** `gpt-5.4-mini`
- **Why:** This task is reasoning-heavy. It needs to combine several uncertain inputs into a clear practical recommendation rather than simply gather more facts.

## Cost & latency

- **Cost profile:** medium
- **Latency profile:** fast
- **Run in parallel:** with caveats — parallel by route family, not by tiny fragments.

## Inputs

- Route concept or itinerary.
- Travel date and departure window.
- Key corridors and choke points.
- Evidence on congestion, closures, weather, or events.
- User tolerance for delay and risk.

## Outputs

- **Risk Summary** — overall viability and major choke points.
- **Best / Medium / Worst Case** — scenario framing.
- **Decision Rule** — when to run, shorten, or drop the route.
- **Timing Guidance** — realistic departure and return windows.
- **Sources** — URLs that support the judgment.

Full response format: see the canonical definition.

## Used by workflows

- [`research`](../workflows/research.md)

## Canonical definition

[`../../.cursor/agents/traffic_risk_synthesis.md`](../../.cursor/agents/traffic_risk_synthesis.md)

## Notes

- This is especially useful on Japanese holiday travel days, bad-weather windows, or routes with one structural bottleneck like the Aqua-Line or Chuo.
- Best used after initial route ideas exist but before final commitment.
