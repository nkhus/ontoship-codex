---
name: onto-doc
description: Build the ENTIRE knowledge base for this repo — survey the codebase, then fan out kb_curator subagents per area to produce docs/ (per-service READMEs, reference specs, runbooks, decisions, entry point) following the OntoShip ontology, then lint + index + map. Use to bootstrap or rebuild a project's whole KB, or when invoked as $onto-doc [scope].
---

Build (or rebuild) the **whole OntoShip KB** for this repository by fanning out curator agents.

Scope hint = the text passed with this invocation (`$onto-doc [scope]`) — empty = whole repo; or a subset (e.g. `services/api services/billing`, or "only reference docs").

`G="python3 <skills>/kb-search/gitmark.py"` — `gitmark.py` lives in the sibling
`kb-search` skill folder (repo checkout: `skills/kb-search/gitmark.py`). Run from the
repo root.

## Plan

1. **Survey the repo** — map top-level dirs, services/modules, entry points, build/deploy
   files, and existing docs. Check current coverage: `$G stat`.

2. **Decompose into doc areas** — one unit of work per area:
   - each service/component → `docs/services/<svc>/README.md` (`node_type: service`)
   - cross-cutting specs (architecture, billing, limits, security) → `docs/reference/` (`reference`)
   - operational procedures → `docs/ops/` (`runbook` / `gotcha`)
   - architectural decisions → `docs/decisions/` (`decision`)

3. **Dispatch curators (fan-out)** — for each area **spawn a `kb_curator` subagent**
   (`.codex/agents/kb-curator.toml`) that follows the `kb-curate` skill on that slice only:
   - search first (don't duplicate); pick `node_type` + correct folder;
   - write frontmatter (`node_type`, `title`, `service`, `status: active`, `updated`);
   - add ≥1 typed link to the code it documents (`documents:[src/…]` / `implemented_by`);
   - add a line to the folder `README.md` index.
   Run independent areas in parallel, wait for all of them, and keep each agent scoped to
   its area to avoid collisions (write-heavy agents must not share files).

4. **Entry point + indexes** — ensure `AGENTS.md` exists as the entry point,
   `docs/README.md` is the master index, and every folder has a README index.

5. **Verify & derive** — `$G lint` (fix broken links / orphans / missing frontmatter),
   then `$G index`, then `$G map -o docs-map.html`.

6. **Report** — how many docs created/updated, KB coverage before→after, lint result, and the
   map path. List any areas that need a human decision.

Principle: md+git is the source of truth; derived (index, graph) is regenerated. Never
duplicate — edit existing docs. Keep curator agents scoped so they don't fight over files.
