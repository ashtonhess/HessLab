# deep_research_agent

Performs in-depth, cross-source web research on a focused question (specs, reliability, reviews, comparisons, market analysis, etc.) and returns a structured report with sources.

## Model

- **Model:** `gemini-3-flash`
- **Why:** Task is browse-heavy and synthesis-light per source. Flash handles many fetches cheaply. If a workflow needs heavier synthesis, upgrade this to `gemini-3-pro` or `gpt-5.4-mini` on a case-by-case basis.

## Cost & latency

- **Cost profile:** cheap
- **Latency profile:** medium (bounded by web fetches)
- **Run in parallel:** **yes** — dispatch one per sub-question (e.g., one per candidate car model for reliability research).

## Inputs

- A focused research question, ideally scoped to a single topic or entity.
- Any constraints (e.g., "US market only," "2019+ model years," "skip marketing pages, prefer forums and Edmunds").

## Outputs

- **Summary** — concise overview of key findings.
- **Detailed Report** — structured bullet points (Specs, Pricing, Pros/Cons, etc.).
- **Sources** — list of URLs with brief descriptions.
- **Recommendations** — follow-ups the orchestrator might want.

Full response format: see the canonical definition.

## Used by workflows

- [`car_search`](../workflows/car_search.md) (reliability research, best-year selection, feature-package lookup)

## Canonical definition

[`../../.cursor/agents/deep_research.md`](../../.cursor/agents/deep_research.md)

## Notes

- **Scope tightly.** Broad questions ("tell me about SUVs") produce shallow output. Break them down first in the orchestrator.
- **Cross-verify.** Subagent is instructed to check 3–5 sources per claim — but the orchestrator should still sanity-check numbers (HP, torque, years) against a second source.
- **Not a listings finder.** Use `car_listings_agent` for finding inventory; use this one for understanding *what* to buy.
