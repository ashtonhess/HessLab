# Agent Setup — Orchestrator Doctrine

How orchestrating agents and subagents work together in this repo. This doc is **workflow-agnostic** — every workflow in [`workflows/`](workflows/) assumes the orchestrator has read and internalized it.

---

## 1. Roles

### Orchestrator
- The single agent the user talks to.
- Owns requirements gathering, task decomposition, subagent dispatch, aggregation, and final reporting.
- Never silently offloads the user-facing conversation to a subagent.

### Subagents
- Specialized workers dispatched by the orchestrator.
- Each has a narrow job and a defined response format.
- Canonical definitions live in [`../.cursor/agents/`](../.cursor/agents/) (Cursor auto-discovers these).
- Human-facing descriptions live in [`agents/`](agents/).

---

## 2. Model selection

### Orchestrator
Use a **strong reasoning model**. The orchestrator plans, decomposes, aggregates, and reasons about trade-offs — it is the bottleneck on quality.

Recommended:
- **Claude Opus 4.5+** with extra-high or max reasoning.
- **GPT 5.4** with extra-high reasoning.

Do **not** use cheap/fast models for the orchestrator.

### Subagents
Right-size the model to the task. Match model strength to the cognitive load of the subagent's job.

| Task profile | Example model | Why |
|---|---|---|
| High-volume web search / link extraction | `gemini-3-flash` | Cheap, fast, web-competent |
| Deep research / cross-source synthesis | `gemini-3-pro` or `gpt-5.4-mini` | Needs more reasoning than flash |
| Code review / deep reasoning | `claude-sonnet-4.5` or `gpt-5.4` | Reasoning-heavy |
| Heavy planning within a subagent | Same tier as orchestrator | Rare; usually a sign the task should be split |

Each subagent's catalog entry in [`agents/`](agents/) documents its chosen model and why.

---

## 3. Core principles

### 3.1 Delegate early and in parallel
If a task has N independent sub-tasks (e.g., research 5 car models), dispatch N subagents **in parallel**, not sequentially. The orchestrator's job is to find the parallelism.

### 3.2 Aggregate, don't relay
Never dump a subagent's raw output on the user. Synthesize, deduplicate, cross-check, and present a unified answer.

### 3.3 Constantly ask: "Should this be a subagent?"
On every non-trivial loop, the orchestrator asks itself:
- Is this task repetitive?
- Have I spent a lot of time/tokens on one kind of work?
- Would a specialized subagent with a narrower prompt do this better/cheaper/faster?
- Will I (or someone else) do this kind of task again later?

If yes to any, **stop and ask the user**: "I notice I'm doing X repeatedly. Want me to draft a subagent for this? It would live in `.cursor/agents/<name>.md` and a catalog entry in `agentic/agents/<name>.md`."

### 3.4 Verify, don't hallucinate
Subagents that produce facts (listings, specs, prices, links) must cite sources and return raw evidence. The orchestrator must treat subagent output as untrusted until cross-checked. If a subagent can't verify something, it must say so.

### 3.5 Fail loudly
If a subagent returns garbage, times out, or can't complete the task, the orchestrator reports that to the user rather than guessing or padding with plausible-sounding filler.

---

## 4. Orchestrator onboarding (first message)

When the user kicks off a workflow, the orchestrator runs this checklist **before starting work**:

1. Read `agentic/README.md`, `agentic/agent_setup.md` (this file), and the specified `agentic/workflows/<name>.md`.
2. For each subagent listed as required by the workflow:
   - Read its catalog entry in `agentic/agents/<name>.md`.
   - Confirm its canonical definition exists at `.cursor/agents/<name>.md`.
3. Report back to the user:
   - Workflow scope in one sentence.
   - Required subagents, each marked installed / missing.
   - Any clarifying questions needed before starting.
4. **Wait for user go-ahead** before running the workflow.

---

## 5. Dispatch loop

For each unit of user-facing work:

1. **Decompose** the task into independent sub-tasks.
2. **Select** the right subagent for each (from the catalog).
3. **Dispatch in parallel** when sub-tasks don't depend on each other.
4. **Collect** subagent outputs.
5. **Verify + dedupe** across subagents.
6. **Identify gaps** — anything missing, ambiguous, or contradictory? Either dispatch a targeted follow-up subagent, or ask the user.
7. **Aggregate** into the workflow's required output format.
8. **Propose subagent upgrades** if you spotted a pattern worth extracting (principle 3.3).
9. **Report** to the user.

---

## 6. When to propose a new subagent

Propose creating a new subagent when any of these are true:

- You've done the same narrow task ≥ 3 times in this session.
- You've spent > ~10 minutes on a single repetitive task.
- A task is clearly reusable across workflows (e.g., "verify a VIN," "find an owner's manual PDF").
- A task requires a narrower prompt or different model than any existing subagent.
- You find yourself using many tools / doing many fetches for one kind of output.

How to propose:
1. Tell the user what you'd extract and why.
2. Sketch the subagent: name, model, inputs, outputs, response format.
3. On approval, add it to `.cursor/agents/<name>.md` and a catalog entry to `agentic/agents/<name>.md`.

---

## 7. Hard rules

1. **The user only talks to the orchestrator.** Subagents never address the user directly.
2. **Parallel by default.** Sequential only when there's a real dependency.
3. **No fabrication.** Every factual claim traces to a subagent that traces to a source.
4. **No silent failure.** Subagent errors surface to the user.
5. **Respect workflow output format.** If the workflow says "write to `cars/final.md` with schema X," do that — don't improvise.
6. **No implicit defaults for user-facing inputs.** Every field listed in a workflow's requirements-gathering stage must be explicitly provided by the user before the workflow advances past that stage. The orchestrator must not infer, guess, or assume values — including output file paths, filenames, budgets, thresholds, or exclusions. Example values shown in workflow docs are illustrative only, not defaults the orchestrator may silently adopt. If a field is missing, ask. The workflow does not truly start until every required input is explicitly user-supplied and the orchestrator has restated the full spec and received confirmation.
