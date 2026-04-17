# HessLab

My public lab — shareable pieces of my Cursor / AI dev setup. Today that's agents and multi-agent workflows; this repo will grow to include more (prompts, rules, hooks, skills, utilities) over time.

## What's here

- **[`agentic/`](agentic/)** — multi-agent workflow library (orchestrator doctrine, subagent catalog, workflow runbooks). Start at [`agentic/README.md`](agentic/README.md).
- **[`.cursor/agents/`](.cursor/agents/)** — canonical Cursor subagent definitions (auto-discovered by Cursor on clone).

## Quick start

1. Clone this repo.
2. **Open `HessLab/` as your Cursor workspace root** (File → Open Folder → select `HessLab`). This is required — Cursor only auto-discovers subagents from the workspace root's `.cursor/agents/` folder. If you open a parent directory, the subagents won't be dispatchable.
3. Open [`agentic/README.md`](agentic/README.md) and pick a workflow.
4. Follow the bootstrap prompt in that workflow's runbook.
