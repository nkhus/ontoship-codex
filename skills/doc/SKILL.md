---
name: doc
description: Compose or update ONE knowledge-base document for the given topic following the OntoShip ontology (node_type, frontmatter, typed links, folder README index). Wraps the kb-curate skill. Use for "document X", "record this decision", "write up how Y works", or when invoked as $doc <topic>.
---

Compose or update a KB document for the topic passed with this invocation (`$doc <topic>`).

`G="python3 <skills>/kb-search/gitmark.py"` — `gitmark.py` lives in the sibling
`kb-search` skill folder (repo checkout: `skills/kb-search/gitmark.py`). Run from the
repo root.

Follow the `kb-curate` skill (ontology over code):

1. **Search first** — `$G search "<topic>"`.
   If the topic already exists → **edit that doc**, don't create a second one.
2. **Pick a `node_type`** — `service` · `reference` · `runbook` · `gotcha` · `decision` ·
   `plan` · `guide` · `report` · `index` (unsure → spec = `reference`, how-to = `guide`)
   and the **right folder** (service → `docs/services/<svc>/`, cross-cutting → `docs/reference/`,
   ops → `docs/ops/`, plan → `docs/plans/`, decision → `docs/decisions/`).
3. **Write frontmatter** — `node_type`, `title`, `service`, `status: active`, `updated: <today>`.
4. **Add ≥1 typed link** — to code (`documents`/`implemented_by`) or a sibling doc
   (`depends_on`/`relates_to`). No orphans.
5. **Add a line to the folder `README.md`** (its index): `- [Title](file.md) — hook`.
6. **Lint + reindex** — `$G lint` then `$G index`.

Report which file you created/updated, its `node_type`, and the links you added.
