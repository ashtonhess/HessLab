# Agentic

Entry point for running multi-agent workflows in Cursor from this repo.

This folder contains:

- **`agent_setup.md`** — doctrine for how orchestrators and subagents should work together. Workflow-agnostic.
- **`agents/`** — human-facing catalog of available subagents. Each file describes one subagent: purpose, model, cost, I/O, and which workflows use it. Canonical subagent definitions live in [`../.cursor/agents/`](../.cursor/agents/) (auto-discovered by Cursor).
- **`workflows/`** — one runbook per workflow. Each runbook includes required subagents, bootstrap instructions, example user prompts, hard rules, and output format.

## Quick Start

### Prerequisite: open `HessLab/` as your Cursor workspace root

**File → Open Folder → select `HessLab`.** This is required. Cursor only auto-discovers subagents from the workspace root's `.cursor/agents/` folder — if you open a parent directory (e.g. a monorepo containing `HessLab/` as a subfolder), the subagents won't be dispatchable and the workflow will fail mid-run. All paths in the bootstrap prompts below are relative to this workspace root.

### 1. Pick a workflow

Browse [`workflows/`](workflows/) and pick one (e.g. [`workflows/car_search.md`](workflows/car_search.md)).

### 2. Start a fresh Cursor chat

Use a strong orchestrator model — e.g. **Opus 4.5+ with max reasoning**, or **GPT 5.4 with extra-high reasoning**. See [`agent_setup.md`](agent_setup.md) for model guidance.

### 3. Paste the bootstrap prompt

This is the universal bootstrap. Replace `<WORKFLOW_NAME>` with the filename of the workflow you picked (e.g. `car_search`).

```
You are the orchestrating agent for the <WORKFLOW_NAME> workflow in this repo.

Before we start, set yourself up:

1. Read `agentic/README.md`.
2. Read `agentic/agent_setup.md` in full and internalize the orchestrator doctrine.
3. Read `agentic/workflows/<WORKFLOW_NAME>.md` in full.
4. For every subagent the workflow lists as required, read its catalog entry in
   `agentic/agents/<NAME>.md` and confirm the canonical definition exists at
   `.cursor/agents/<NAME>.md`.
5. Report back with:
   - The workflow scope in one sentence.
   - The list of required subagents and that each one is installed.
   - A list of every Stage 1 input the workflow requires, marked provided or
     missing based on this message. Any field I did NOT explicitly provide must
     be listed as a clarifying question.

Do not start the workflow yet. Per doctrine §7.6, the workflow does not begin
until every Stage 1 input is explicitly provided by me and you have restated
the full spec for my confirmation.
```

### 4. Run the workflow

Once the orchestrator reports ready, give it your actual task. Every workflow file has copy-paste example prompts under its "Example User Prompts" section.

## How this is organized

```
agentic/
├── README.md            ← you are here
├── agent_setup.md       ← orchestrator + subagent doctrine
├── agents/              ← subagent catalog (human-facing)
│   ├── README.md
│   └── <agent_name>.md  ← one file per subagent
└── workflows/           ← workflow runbooks
    ├── README.md
    └── <workflow>.md    ← one file per workflow

../.cursor/agents/       ← canonical subagent definitions (Cursor auto-discovers)
└── <agent_name>.md
```

## Adding new things

- **New subagent** → see [`agents/README.md`](agents/README.md).
- **New workflow** → see [`workflows/README.md`](workflows/README.md).
- **Revising doctrine** → edit [`agent_setup.md`](agent_setup.md). Keep it workflow-agnostic.
