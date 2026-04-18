# visual_sourcing_agent

Finds representative images and visual source pages for stops, routes, and destinations. It helps the orchestrator confirm whether a place actually looks as compelling as the text suggests and supports brochure-style outputs with better visual grounding.

## Model

- **Model:** `gemini-3-flash`
- **Why:** This is a browse-heavy retrieval task. It benefits from quick source discovery and filtering more than from heavyweight synthesis.

## Cost & latency

- **Cost profile:** cheap
- **Latency profile:** medium
- **Run in parallel:** yes — by stop or route segment.

## Inputs

- Stop list or route list.
- Desired visual mood.
- Source preferences (official pages preferred, etc.).

## Outputs

- **Recommended Source Page** — the most useful page to cite or browse from.
- **Direct Image URL** — only when confidence is high.
- **Why the Image Works** — what it communicates about the stop.
- **Attribution / Licensing Caveat** — where reuse certainty is weak.
- **Gaps** — stops that still need stronger visual support.

Full response format: see the canonical definition.

## Used by workflows

- [`research`](../workflows/research.md)

## Canonical definition

[`../../.cursor/agents/visual_sourcing.md`](../../.cursor/agents/visual_sourcing.md)

## Notes

- This agent should be conservative about direct image links.
- It is especially useful for travel guides, slide decks, or any artifact where the visual feel of a stop affects the recommendation.
