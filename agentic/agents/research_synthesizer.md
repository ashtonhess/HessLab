# research_synthesizer_agent

Transforms verified research notes into decision-ready artifacts such as comparisons, ranked options, itineraries, recommendation memos, and structured markdown deliverables. It is the "last mile" agent for research workflows: not the one that goes broad on the web, but the one that turns evidence into something a user can act on quickly.

## Model

- **Model:** `gpt-5.4-mini`
- **Why:** This job is synthesis-heavy rather than browse-heavy. It needs stronger structure and trade-off reasoning than a cheap web-search model, but it does not need full orchestrator-grade reasoning.

## Cost & latency

- **Cost profile:** medium
- **Latency profile:** fast
- **Run in parallel:** with caveats — parallelize by independent output section or scenario, not by tiny fragments that need heavy recombination.

## Inputs

- Research brief / user request.
- Verified notes, quotes, facts, and URLs from the orchestrator.
- Ranking criteria, constraints, exclusions, or scoring rubric.
- Required output schema if the user asked for one.

## Outputs

- **Executive Summary** — short answer first.
- **Structured Findings** — grouped evidence, facts separated from judgment.
- **Options / Scenarios** — clear trade-offs for each option.
- **Recommendation** — best-fit answer tied to the user's criteria.
- **Open Questions / Risks** — unknowns, caveats, stale-data warnings.
- **Sources Used** — URLs that support the important claims.

Full response format: see the canonical definition.

## Used by workflows

- [`research`](../workflows/research.md)

## Canonical definition

[`../../.cursor/agents/research_synthesizer.md`](../../.cursor/agents/research_synthesizer.md)

## Notes

- This agent should not paper over missing evidence. If the facts are thin, route back to [`deep_research_agent`](deep_research.md).
- Best used after the orchestrator has already decomposed the topic into focused research questions and gathered evidence in parallel.
- Particularly useful when the final artifact needs multiple options, ranked recommendations, timelines, or comparison sections.
