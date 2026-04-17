# Car Search

Find used cars for sale in the US that match a user's hard requirements (engine, trim, features) and soft preferences (reliability year, mileage, location), using parallel subagents for research and listing discovery.

## Scope

**Covers:**
- US market, used vehicles.
- Multi-model searches (e.g., "any car with the B58 engine").
- Reliability-aware year selection before searching listings.
- Direct-link inventory discovery across CarGurus, AutoTempest, TrueCar, Cars.com, AutoTrader, and dealer sites.
- Feature verification (especially packaged options like adaptive cruise).

**Does NOT cover:**
- New-car configurator pricing.
- Non-US markets.
- Auctions (Bring a Trailer, Cars & Bids) unless the user explicitly opts in.
- Negotiation / purchase logistics.

## Required agents

- [`car_listings_agent`](../agents/car_listings.md) — fan-out listing discovery across platforms.
- [`deep_research_agent`](../agents/deep_research.md) — reliability / best-year research, feature-package identification, spec verification.

Confirm both catalog entries exist in [`../agents/`](../agents/) **and** both canonical definitions exist in [`../../.cursor/agents/`](../../.cursor/agents/) before starting.

## Bootstrap prompt

**Prerequisite:** `HessLab/` must be opened as your Cursor workspace root (File → Open Folder → select `HessLab`). Cursor auto-discovers subagents from the workspace root's `.cursor/agents/` folder only — the orchestrator cannot dispatch `car_listings_agent` or `deep_research_agent` otherwise.

Paste into a fresh Cursor chat with a strong orchestrator model (Opus 4.5+ max reasoning, or GPT 5.4 extra-high reasoning):

```
You are the orchestrating agent for the car_search workflow in this repo.

Before we start, set yourself up:

1. Read `agentic/README.md`.
2. Read `agentic/agent_setup.md` in full and internalize the orchestrator doctrine.
3. Read `agentic/workflows/car_search.md` in full.
4. Read the catalog entries for `car_listings_agent` and `deep_research_agent` in
   `agentic/agents/` and confirm their canonical definitions exist at
   `.cursor/agents/car_listings.md` and `.cursor/agents/deep_research.md`.
5. Report back with:
   - The workflow scope in one sentence.
   - That both subagents are installed.
   - A list of every Stage 1 input the workflow requires, marked provided or missing
     based on this message. Any field I did NOT explicitly provide must be listed as
     a clarifying question.

Do not start searching yet. Per doctrine §7.6, the workflow does not begin until
every Stage 1 input is explicitly provided by me and you have restated the full
spec for my confirmation.
```

## Example user prompts

### Example 1 — Engine-family search (from the real B58 use case)

```
Create a new document called `cars/b58.md`. In that document, find me used cars
available in the USA that have the B58 engine and match as many of these features
as possible. If a car is missing a couple of non-critical features, note that but
include it.

Hard requirements:
- Adaptive cruise control — NON-NEGOTIABLE.
- US market, under $35k.
- Exclude listings from big metros (NYC, LA) where miles don't reflect wear.

Year guidance:
- X3 M40i: 2020+ only (more reliable).
- For every other candidate model, FIRST research the best reliability year
  you can buy under $35k before looking at listings.

Process:
- Use AutoTempest or equivalent to pull across platforms in one place.
- Return direct links to individual listings (not search pages).
- Verify every listing — no fabrications.
- Use `deep_research_agent` for reliability/best-year research.
- Use `car_listings_agent` for listing discovery.
- Fan out in parallel across candidate models.

Do not take shortcuts. Verify every listing.
```

### Example 2 — Single-model search

```
Create a new document called `cars/x3_m40i.md`. Find me 2020+ BMW X3 M40i
listings under $32k, must have adaptive cruise and heated seats. Exclude NYC
and LA metros. Use `deep_research_agent` to confirm which option packages
include adaptive cruise for this model/year, then dispatch `car_listings_agent`
across all major platforms in parallel. Verify every listing.
```

### Example 3 — Reliability-first exploration

```
I have ~$30k. I want a reliable, turbocharged, RWD or AWD sedan or wagon
from the last ~5 years that is fun to drive. Use `deep_research_agent` to
draft a shortlist of 5–8 candidates with best-reliability year each, then
use `car_listings_agent` to find current listings for each. Write the
result to `cars/shortlist.md`.
```

## Workflow stages

### Stage 1 — Requirements interview

**Orchestrator (no subagent).** Every field below must be **explicitly provided by the user**. Per doctrine §7.6, the orchestrator does not infer, assume, or silently default any of these values — even if the user's opening prompt looks complete. If a field wasn't addressed, ask.

Required inputs:

- Budget ceiling.
- Hard feature requirements (things that must be present — dropped if missing).
- Soft feature requirements (flag if missing, still include).
- Engine / drivetrain / body-style constraints.
- Candidate models (if user is supplying them) OR explicit instruction to build a reliability-first shortlist.
- Location exclusions (the user must state these — there is no built-in default, even for large metros).
- Mileage ceiling (user may say "none," but must say something).
- Output file path (full relative path, e.g. `cars/b58.md`). No default.

Exit: orchestrator restates the full spec back to the user and receives explicit confirmation. **The workflow does not advance to Stage 2 before this confirmation.**

### Stage 2 — Subagent roster re-evaluation

**Orchestrator (no subagent).** Before fanning out, ask:

- Do the existing subagents cover everything this spec needs?
- Is there a repetitive sub-task that warrants a new specialized subagent (e.g., "VIN_verifier," "owner_manual_fetcher")?

If yes, propose it to the user per doctrine §3.3 / §6. Only proceed once the roster is settled.

### Stage 3 — Reliability & best-year research (parallel)

**Subagent:** `deep_research_agent`.

Dispatch **one instance per candidate model** in parallel. Each asks:
- "What's the most reliable sub-$BUDGET year for <model>?"
- "Which option package(s) include <required feature> in <year range>?"
- "Are there known-bad years to avoid?"

Exit: orchestrator has a per-model target year range and feature-package notes.

### Stage 4 — Listings discovery (parallel)

**Subagent:** `car_listings_agent`.

Dispatch **one instance per (model, year-range)** in parallel. Each gets:
- Make, model, trim, year range (from Stage 3).
- Max price.
- Required and nice-to-have features (with package names from Stage 3).
- Location exclusions.

Exit: raw table of candidate listings per model with VIN, price, miles, location, direct link.

### Stage 5 — Verification & dedupe

**Orchestrator + optional `deep_research_agent` follow-ups.**

- Deduplicate across platforms using VIN.
- Spot-check suspicious listings (price too low, missing VIN, ambiguous feature claims).
- If a listing's adaptive-cruise claim isn't clear, dispatch `deep_research_agent` to verify the package for that specific VIN/trim.
- Drop listings that fail hard rules.

### Stage 6 — Write output

**Orchestrator.** Write the final artifact (see [Output format](#output-format)).

### Stage 7 — Report

Summarize to the user in chat:
- Candidate models, best year chosen for each, and why.
- Count of verified listings per model.
- Any listings that were dropped and why.
- Subagents used and roughly how many dispatches.
- Any subagent proposals to consider for next time.

## Hard rules

1. **Adaptive cruise (or any user-flagged hard feature) is non-negotiable.** Drop any listing that can't confirm it.
2. **Direct links only.** No search-result pages in the output.
3. **Verify every listing.** No fabricated URLs, VINs, prices, or features. If `car_listings_agent` can't verify, the listing is excluded.
4. **Honor the user's location exclusions exactly.** The orchestrator does not invent exclusions. If the user wants large-metro listings filtered out (commonly NYC, LA, etc., because city miles poorly reflect wear), they must say so in Stage 1. Ask about it if they didn't.
5. **Reliability-year research comes before listings search** for any model not pinned by the user.
6. **Parallel by default.** Never dispatch `car_listings_agent` or `deep_research_agent` sequentially when the tasks are independent.
7. **Propose a new subagent** if Stage 2's re-evaluation or any later stage shows repetitive work that should be extracted (doctrine §3.3).
8. **No guessing model trims, engine codes, or package contents.** If uncertain, route through `deep_research_agent`.

## Output format

### File

Write to the full path the user specified in Stage 1. There is no default — the orchestrator must not invent a filename or directory. If the target directory doesn't exist, create it (after confirming the path with the user in Stage 1).

### Structure

```markdown
# <Title — e.g., "B58 Engine Cars Under $35k">

## Spec

- Budget: $X
- Hard requirements: ...
- Soft requirements: ...
- Location exclusions: ...
- Candidate models considered: ...

## <Model 1>

**Best-year research:** <summary from deep_research_agent, with sources>

| Price | Year | Miles | Location | VIN | Adaptive Cruise | Other Features | Link |
|---|---|---|---|---|---|---|---|
| $XX,XXX | YYYY | XX,XXX | City, ST | <VIN> | confirmed / needs-verification | ... | <direct URL> |

**Notes:**
- Listings excluded and why.
- Features needing dealer verification.

## <Model 2>

(same structure)

## Subagent log

- `deep_research_agent` dispatched N times for: <topics>
- `car_listings_agent` dispatched N times for: <(model, year) tuples>
```

### Chat response

Short. Summarize per Stage 7 and link to the file you wrote. Do not paste the whole table into chat.
