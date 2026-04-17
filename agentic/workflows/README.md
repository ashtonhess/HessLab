# Workflows

Each workflow is a runbook the orchestrator follows for a specific kind of task. A workflow file bundles: scope, required subagents, bootstrap instructions, example user prompts, workflow stages, hard rules, and expected output format.

## Available workflows

| Workflow | Purpose | Required subagents |
|---|---|---|
| [`car_search`](car_search.md) | Find used cars for sale in the US matching feature, budget, and reliability constraints. | `car_listings_agent`, `deep_research_agent` |

## Adding a new workflow

1. **Copy the template below** into `agentic/workflows/<name>.md`.
2. **Fill in every section.** If a section doesn't apply, say so explicitly rather than deleting it.
3. **List required subagents** by their catalog name and link to their catalog entries in [`../agents/`](../agents/). If a needed subagent doesn't exist yet, create it first (see [`../agents/README.md`](../agents/README.md)).
4. **Register it** by adding a row to the table above.
5. **Link it** from the top-level [`../README.md`](../README.md) if it's a flagship workflow.

### Workflow runbook template

```markdown
# <workflow name>

One-sentence summary of what this workflow does.

## Scope

- What it covers.
- What it does NOT cover.

## Required agents

- [`<agent_a>`](../agents/<agent_a>.md) — role in this workflow.
- [`<agent_b>`](../agents/<agent_b>.md) — role in this workflow.

## Bootstrap prompt

**Prerequisite:** `HessLab/` must be opened as the Cursor workspace root (File → Open Folder → select `HessLab`) so Cursor auto-discovers subagents from `.cursor/agents/`. State this prerequisite explicitly at the top of every workflow's bootstrap section.

Paste this into a fresh Cursor chat with a strong orchestrator model:

    ```
    You are the orchestrating agent for the <workflow> workflow.

    Set yourself up:
    1. Read `agentic/README.md`.
    2. Read `agentic/agent_setup.md`.
    3. Read `agentic/workflows/<workflow>.md`.
    4. For each required subagent, read its catalog entry in `agentic/agents/`
       and confirm its canonical definition exists at `.cursor/agents/`.
    5. Report scope, installed subagents, and for every Stage 1 input the
       workflow requires, mark it provided or missing based on this message.
       Any field I did NOT explicitly provide must be asked as a clarifying
       question.
    6. Wait for my go-ahead. Per doctrine §7.6, the workflow does not begin
       until every Stage 1 input is explicitly provided by me and you have
       restated the full spec for my confirmation.
    ```

## Example user prompts

At least one copy-paste example prompt a user can give once the orchestrator reports ready. Examples are illustrative only — the orchestrator must still confirm every Stage 1 input explicitly with the user per doctrine §7.6.

## Workflow stages

Numbered stages. For each, state:
- What the orchestrator does.
- Which subagent is dispatched (if any).
- What runs in parallel vs. sequentially.
- Exit criteria for the stage.

**Stage 1 must be a requirements interview.** It lists every required input explicitly. Per doctrine §7.6 the orchestrator does not infer any of them — each field is either user-supplied or asked. Stage 1 exits only after the orchestrator restates the spec and the user confirms.

## Hard rules

- Non-negotiable constraints the orchestrator must enforce.
- Quality bars.
- Things that count as failure.

## Output format

- Where the final artifact is written (file path, directory — always taken from a user-supplied Stage 1 input, never defaulted).
- Schema / headers / table format.
- What goes in the chat response vs. the written artifact.
```
