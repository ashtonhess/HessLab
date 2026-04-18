# Agent Catalog

Human-facing index of subagents available in this repo. Each entry describes **what** the subagent does, **which model** it uses and **why**, **cost/latency profile**, **inputs/outputs**, and **which workflows depend on it**.

> **Canonical subagent definitions live in [`../../.cursor/agents/`](../../.cursor/agents/).** That's where Cursor auto-discovers them. The files in this folder are catalog entries — they describe and link to the canonical definitions, they are not the definitions themselves.

## Available subagents

| Name | Purpose | Model | Catalog entry | Canonical definition |
| --- | --- | --- | --- | --- |
| `car_listings_agent` | Find direct links to individual vehicle listings across major US car platforms. | `gemini-3-flash` | [car_listings.md](car_listings.md) | [`.cursor/agents/car_listings.md`](../../.cursor/agents/car_listings.md) |
| `deep_research_agent` | Deep, cross-source web research on a focused topic. | `gemini-3-flash` | [deep_research.md](deep_research.md) | [`.cursor/agents/deep_research.md`](../../.cursor/agents/deep_research.md) |
| `research_synthesizer_agent` | Turn verified research into structured options, recommendations, and final artifacts. | `gpt-5.4-mini` | [research_synthesizer.md](research_synthesizer.md) | [`.cursor/agents/research_synthesizer.md`](../../.cursor/agents/research_synthesizer.md) |

## Adding a new subagent

1. **Write the canonical definition** at `.cursor/agents/<name>.md` with Cursor frontmatter:

   ```yaml
   ---
   is_background: true
   name: <name>_agent
   model: <model>
   description: <one-line description Cursor uses to decide when to dispatch>
   readonly: true
   ---
   ```

   Follow the pattern in the existing canonical files. Keep the subagent's scope narrow.

2. **Write the catalog entry** at `agentic/agents/<name>.md` using the template below.

3. **Register it** by adding a row to the table above.

4. **Link it from any workflow** that should use it (add to "Required Agents" in the relevant `agentic/workflows/<workflow>.md`).

### Catalog entry template

```markdown
# <name>_agent

<one-paragraph description of what it does>

## Model

- **Model:** `<model>`
- **Why:** <reason this model fits this task>

## Cost & latency

- **Cost profile:** cheap / medium / expensive
- **Latency profile:** fast / medium / slow
- **Run in parallel:** yes / no / with caveats

## Inputs

- <input 1>
- <input 2>

## Outputs

<describe the response format — link to a format section in the canonical file if possible>

## Used by workflows

- [`<workflow>`](../workflows/<workflow>.md)

## Canonical definition

[`../../.cursor/agents/<name>.md`](../../.cursor/agents/<name>.md)

## Notes

<anything an orchestrator should know: known failure modes, tips, when NOT to use>
```
