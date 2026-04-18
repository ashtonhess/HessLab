# Research

Run a generic multi-question research workflow that decomposes a user's problem, fans out focused web research in parallel, and writes a decision-ready artifact to a user-specified file.

## Scope

**Covers:**

- General web research and synthesis.
- Buyer guides, travel/day-trip planning, comparisons, market scans, product/spec research, and decision memos.
- Deliverables that need multiple options, ranked recommendations, timelines, cost conversions, or structured markdown writeups.
- Reusable research jobs where the user wants the result written to a file inside the repo.

**Does NOT cover:**

- Authenticated/private data sources the agent cannot access.
- Exhaustive academic literature reviews with formal citation standards unless the user explicitly asks for that exact format.
- Workflows where the user wants unsourced opinion rather than research.
- Pure code implementation tasks without a meaningful research component.

## Required agents

- [`deep_research_agent`](../agents/deep_research.md) — gathers source-grounded evidence for each focused research question.
- [`research_synthesizer_agent`](../agents/research_synthesizer.md) — turns the verified evidence package into ranked options, structured findings, and a final artifact.

Confirm both catalog entries exist in [`../agents/`](../agents/) **and** both canonical definitions exist in [`../../.cursor/agents/`](../../.cursor/agents/) before starting.

## Bootstrap prompt

**Prerequisite:** `HessLab/` must be opened as your Cursor workspace root (File → Open Folder → select `HessLab`). Cursor auto-discovers subagents from the workspace root's `.cursor/agents/` folder only — the orchestrator cannot dispatch `deep_research_agent` or `research_synthesizer_agent` otherwise.

Paste into a fresh Cursor chat with a strong orchestrator model (Opus 4.5+ max reasoning, or GPT 5.4 extra-high reasoning):

```text
You are the orchestrating agent for the research workflow in this repo.

Before we start, set yourself up:

1. Read `agentic/README.md`.
2. Read `agentic/agent_setup.md` in full and internalize the orchestrator doctrine.
3. Read `agentic/workflows/research.md` in full.
4. Read the catalog entries for `deep_research_agent` and
   `research_synthesizer_agent` in `agentic/agents/` and confirm their
   canonical definitions exist at `.cursor/agents/deep_research.md` and
   `.cursor/agents/research_synthesizer.md`.
5. Report back with:
   - The workflow scope in one sentence.
   - That both subagents are installed.
   - A list of every Stage 1 input the workflow requires, marked provided or
     missing based on this message. Any field I did NOT explicitly provide must
     be listed as a clarifying question.

Do not start researching yet. Per doctrine §7.6, the workflow does not begin
until every Stage 1 input is explicitly provided by me and you have restated
the full spec for my confirmation.
```

## Example user prompts

### Example 1 — Day-trip research with a written plan

```text
Create a new document called `notes/japan/driving_day_trips.md`.

Research the best self-drive day-trip options from central Tokyo for a rented
sports car. I want scenic mountain driving, several short stops, hard time and
mileage constraints, and a few different route options ranked from best fit to
backup plan. Convert all distances to miles and all prices to USD as well as
JPY.
```

### Example 2 — Product / buying research

```text
Create `notes/cameras/travel_camera_shortlist.md`.

I want a shortlist of compact travel cameras under $1,500. Prioritize low-light
performance, reliable autofocus, USB-C charging, and lens ecosystem. Give me a
decision-ready comparison with 4-6 options, trade-offs, and a final
recommendation.
```

### Example 3 — Generic decision memo

```text
Create `notes/software/vector_db_options.md`.

Research which vector database is the best fit for a small team building a
document search product this quarter. Compare hosted vs self-hosted options,
price model, operational burden, and developer ergonomics. I want a short
decision memo with a clear recommendation and 2 backup options.
```

## Workflow stages

### Stage 1 — Requirements interview

**Orchestrator (no subagent).** Every field below must be **explicitly provided by the user**. Per doctrine §7.6, the orchestrator does not infer, assume, or silently default any of these values — even if the opening prompt feels complete.

Required inputs:

- Core research question / objective.
- Scope boundaries (geography, product/category, time window, audience, or other domain limits as applicable).
- Deliverable type (comparison, plan, itinerary, memo, shortlist, etc.).
- Required sections / output schema **or** explicit instruction to use the workflow's standard research format.
- Decision criteria / ranking priorities.
- Constraints, exclusions, and red lines.
- Recency requirement (for example: "current as of today," "latest available," or "historical is fine").
- Output file path (full relative path, e.g. `notes/topic/result.md`). No default.

Exit: orchestrator restates the full spec back to the user and receives explicit confirmation. **The workflow does not advance to Stage 2 before this confirmation.**

### Stage 2 — Subagent roster re-evaluation

**Orchestrator (no subagent).** Before fanning out, ask:

- Do `deep_research_agent` and `research_synthesizer_agent` cover everything this brief needs?
- Is there a repetitive domain-specific task that warrants a new specialized subagent?

If yes, propose it to the user per doctrine §3.3 / §6. Only proceed once the roster is settled.

### Stage 3 — Research decomposition

**Orchestrator (no subagent).**

- Break the brief into focused research questions.
- Separate fact-finding from synthesis.
- Decide what can be researched in parallel.
- Define what evidence each sub-question must return.

Exit: a research plan with a clear sub-question list and dispatch plan.

### Stage 4 — Evidence gathering (parallel)

**Subagent:** `deep_research_agent`.

Dispatch **one instance per independent research question** in parallel. Each prompt should ask for:

- the focused question,
- relevant constraints,
- source preferences if any,
- a concise summary,
- detailed findings,
- source URLs,
- and follow-up recommendations.

Exit: orchestrator has a source-grounded evidence package covering all required sub-questions.

### Stage 5 — Verification, dedupe, and gap fill

**Orchestrator + targeted `deep_research_agent` follow-ups.**

- Remove duplicated findings across subagents.
- Cross-check key numbers, dates, prices, distances, and route details.
- Flag stale, uncertain, or conflicting facts.
- Dispatch targeted follow-ups where the first pass is thin or contradictory.

Exit: a clean, verified note set ready for final synthesis.

### Stage 6 — Artifact synthesis

**Subagent:** `research_synthesizer_agent`.

Give the synthesizer:

- the approved Stage 1 brief,
- the cleaned evidence package,
- the ranking criteria,
- the required output schema.

Exit: a structured draft that contains findings, options, recommendations, risks, and source references in the requested format.

### Stage 7 — Write output

**Orchestrator.** Write the final artifact (see [Output format](#output-format)).

### Stage 8 — Report

Summarize to the user in chat:

- what was researched,
- the main recommendation,
- major alternatives,
- any important unknowns or time-sensitive caveats,
- subagents used and roughly how many dispatches,
- and any new subagent proposals created during the run.

## Hard rules

1. **No implicit defaults for Stage 1 inputs.** Missing fields must be asked explicitly.
2. **Parallel by default.** Independent research questions are never handled sequentially without a dependency reason.
3. **Source every important factual claim.** If a claim is not verified, label it as approximate, tentative, or unresolved.
4. **Separate facts from judgments.** The final artifact must make it obvious what is sourced versus what is the orchestrator's recommendation.
5. **No raw subagent dump.** The orchestrator aggregates, deduplicates, and writes a unified result.
6. **Propose a new subagent** when the work reveals a reusable, repetitive task not well-covered by the existing roster.
7. **Write only to the user-specified path.** The orchestrator must not invent filenames or directories.
8. **Respect the requested artifact shape.** If the user approved a custom schema, do not fall back to the standard one.

## Output format

### File

Write to the full path the user specified in Stage 1. There is no default — the orchestrator must not invent a filename or directory. If the target directory doesn't exist, create it (after confirming the path with the user in Stage 1).

### Standard structure

If and only if the user explicitly approved the workflow's standard research format in Stage 1, use this structure:

```markdown
# <Title>

## Spec

- Objective: ...
- Scope: ...
- Deliverable: ...
- Ranking priorities: ...
- Constraints / exclusions: ...
- Recency requirement: ...

## Executive Summary

- ...

## Findings

- ...

## Options / Scenarios

### Option 1 — <name>

- Fit for the brief: ...
- Strengths: ...
- Weaknesses: ...
- Evidence: ...

### Option 2 — <name>

- ...

## Recommendation

- Best fit: ...
- Why: ...

## Open Questions / Risks

- ...

## Sources

- <URL> — <why it matters>

## Subagent log

- `deep_research_agent` dispatched N times for: <topics>
- `research_synthesizer_agent` dispatched N times for: <artifact sections or scenarios>
```

### Chat response

Short. Summarize per Stage 8 and link to the file you wrote. Do not paste the full artifact into chat.
