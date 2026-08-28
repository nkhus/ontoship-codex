---
name: kb-map
description: Build the OntoShip KB graph (gitmark map) — collapsible tree + rendered markdown + force/radial link graph as a self-contained HTML — and point the user to it. Use for "show the docs graph/map", "visualize the KB", or when invoked as $kb-map [output-path].
---

Generate and surface the knowledge-base map.

Optional argument = output path (default `docs-map.html`).

`G="python3 <skills>/kb-search/gitmark.py"` — `gitmark.py` lives in the sibling
`kb-search` skill folder (repo checkout: `skills/kb-search/gitmark.py`). Run from the
repo root.

Do:
1. Refresh the index: `$G index`
2. Build the map: `$G map -o "<output-path or docs-map.html>"`
3. Tell the user the output path and that it's a self-contained HTML (tree + link graph), and offer to open it (`open <path>`) or `serve` it.
