---
node_type: reference
title: OntoShip skills
service: _platform
status: active
updated: 2026-08-28
tags: [skills, commands, reference]
links:
  documents: [../../skills/kb/SKILL.md, ../../skills/kb-map/SKILL.md, ../../skills/doc/SKILL.md, ../../skills/onto-doc/SKILL.md, ../../skills/ship/SKILL.md]
  relates_to: [../services/gitmark-cli/README.md, ../services/dev-flow/README.md]
---

# OntoShip skills

Reference for the skills shipped by the **OntoShip** Codex plugin. Each verb is a
`skills/<name>/SKILL.md` that drives a capability skill or the engine. Invoke one
explicitly with `$name …` (or pick it from `/skills`); Codex can also choose it implicitly
when the task matches the skill `description`. Two families:

- **KB curation & search** — `$kb`, `$kb-map`, `$doc`, `$onto-doc` — drive the GitMark
  CLI (`skills/kb-search/gitmark.py`) and the `kb-curate` ontology rules.
- **Dev-flow** — `$ship` — drives the gated `dev-flow` pipeline from idea to production.

The GitMark CLI is `gitmark.py`, sitting next to `skills/kb-search/SKILL.md`; when the
plugin is installed it lives at `<plugin-root>/skills/kb-search/gitmark.py`. Codex prints
each skill's source path in the skills list, so the agent can always resolve it.

## Summary

| Skill | What it does | Args | Drives |
|---|---|---|---|
| `$kb` | Search the project KB (all `.md`) and answer from the top hits | `<query>` (empty → stat + usage) | GitMark CLI `search` / `stat` / `index` |
| `$kb-map` | Build a self-contained HTML map of the KB (tree + link graph) | `[output-path]` (default `docs-map.html`) | GitMark CLI `map` / `index` |
| `$doc` | Compose or update **one** KB document per the ontology | `<topic>` | `kb-curate` skill + GitMark CLI |
| `$onto-doc` | Build (or rebuild) the **whole** KB by fanning out curator subagents | `[scope]` (empty → whole repo) | `kb-curate` via `kb_curator` fan-out + GitMark CLI |
| `$ship` | Take a feature/fix from idea to production through the gated pipeline | `<change description>` | `dev-flow` skill |

---

## `$kb` — search the knowledge base

- **Definition:** `skills/kb/SKILL.md`
- **What it does:** searches every `.md` in the project (docs/, READMEs, etc.) via GitMark
  (FTS5 bm25 ranking + trigram/fuzzy matching), then summarizes the top hits and answers
  from the 1–2 most relevant files.
- **Args:** the text after `$kb` = the search query. **Empty** → runs `gitmark.py stat` and
  shows the `$kb <query>` syntax instead of searching.
- **Behavior:**
  1. Empty query → `gitmark.py stat` + usage hint.
  2. Otherwise: refresh index if needed (`gitmark.py index`), then
     `gitmark.py search "<query>" -k 8`, summarize hits as `file:line` (no full
     snippets), and open the most relevant files to answer.
- **Drives:** GitMark CLI (`kb-search` skill).

## `$kb-map` — render the KB graph

- **Definition:** `skills/kb-map/SKILL.md`
- **What it does:** generates a self-contained HTML map of the knowledge base — a
  collapsible tree, rendered markdown, and a force/radial link graph built from the typed
  links in frontmatter.
- **Args:** the text after `$kb-map` = output path. **Default `docs-map.html`** when omitted.
- **Behavior:**
  1. Refresh the index: `gitmark.py index`.
  2. Build the map: `gitmark.py map -o "<output-path>"`.
  3. Report the output path and offer to `open <path>` or `serve` it.
- **Drives:** GitMark CLI `map` (`kb-search` skill).

## `$doc` — compose/update one KB document

- **Definition:** `skills/doc/SKILL.md`
- **What it does:** composes or updates a **single** KB document for a topic, following the
  `kb-curate` ontology (node_type, frontmatter, typed links, folder README index).
- **Args:** the text after `$doc` = the topic/document subject.
- **Behavior (per `kb-curate`):**
  1. **Search first** (`gitmark.py search`) — if the topic exists, edit that doc; never
     create a duplicate.
  2. **Pick a `node_type`** (`service` · `reference` · `runbook` · `gotcha` · `decision` ·
     `plan` · `guide` · `report` · `index`) and the right folder.
  3. **Write frontmatter** — `node_type`, `title`, `service`, `status: active`,
     `updated: <today>`.
  4. **Add ≥1 typed link** (to code or a sibling doc) — no orphans.
  5. **Add a line to the folder `README.md`** index.
  6. **Lint + reindex** — `gitmark.py lint` then `gitmark.py index`.
- **Drives:** `kb-curate` skill + GitMark CLI.

## `$onto-doc` — build the whole KB

- **Definition:** `skills/onto-doc/SKILL.md` · **Subagent:** `.codex/agents/kb-curator.toml`
- **What it does:** builds (or rebuilds) the **entire** OntoShip KB for the repo by
  surveying the codebase and fanning out `kb_curator` subagents per area, then
  linting, indexing, and mapping. Used to bootstrap or rebuild a project's whole KB.
- **Args:** the text after `$onto-doc` = scope hint. **Empty** → whole repo; or a subset
  (e.g. `services/api services/billing`, or "only reference docs").
- **Behavior:**
  1. **Survey** the repo (dirs, services, entry points, build/deploy, existing docs);
     check coverage with `gitmark.py stat`.
  2. **Decompose** into doc areas — service READMEs (`service`), cross-cutting specs
     (`reference`), ops procedures (`runbook`/`gotcha`), decisions (`decision`).
  3. **Dispatch curators (fan-out)** — one `kb_curator` subagent per area, each following
     `kb-curate` on its slice only (search first, pick node_type + folder, frontmatter,
     ≥1 typed link, README index line). Independent areas run in parallel, scoped to avoid
     collisions; the parent waits for all of them.
  4. **Entry point + indexes** — ensure `AGENTS.md` exists, `docs/README.md`
     is the master index, every folder has a README index.
  5. **Verify & derive** — `gitmark.py lint` (fix broken links/orphans/missing
     frontmatter), then `gitmark.py index`, then `gitmark.py map -o docs-map.html`.
  6. **Report** — docs created/updated, coverage before→after, lint result, map path,
     areas needing a human decision.
- **Drives:** `kb-curate` skill via `kb_curator` fan-out + GitMark CLI.

## `$ship` — run the dev-flow

- **Definition:** `skills/ship/SKILL.md` · **Subagent:** `.codex/agents/kb-reviewer.toml`
- **What it does:** drives a feature/fix from idea to production through the gated
  OntoShip dev-flow pipeline.
- **Args:** the text after `$ship` = the change to ship (feature/fix description).
- **Pipeline (per the `dev-flow` skill, end to end):**
  1. **Research** — understand from facts (logs, traces, code); reproduce before fixing.
  2. **Tasks** — decompose into tracked tasks.
  3. **Goal** — one clear goal + a "done" criterion.
  4. **Spec** — write it as markdown in the KB via `kb-curate` (node_type, frontmatter,
     typed links `documents:[src/…]`); search the KB first — don't duplicate.
  5. **Isolate** — work in a dedicated `git worktree`.
  6. **Implement** — code to the spec.
  7. **Tests** — write/adjust unit + E2E.
  8. **Independent review** — run an independent model over the diff: the read-only
     `kb_reviewer` subagent on a different model, `/review`, or an external CLI.
  9. **Dev-tests** — MR + commits into `dev`; run the full suite. Red → fix, don't merge.
  10. **Prod-tests** — E2E/smoke against the real prod contour.
  11. **Ship** — merge `dev → main` and deploy (build-before-stop + healthcheck-poll).
- **Gates:** tests + independent review are not skippable; the spec is the carrier of
  knowledge, not a throwaway ticket.
- **Drives:** `dev-flow` skill.
