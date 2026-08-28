---
name: kb
description: Search the project knowledge base via GitMark (FTS5 bm25 + trigram/fuzzy) and answer from the top hits. Use for "where do the docs say X", "search the KB", or when invoked as $kb <query>; with no query it prints index stats and the syntax.
---

Search the project's markdown knowledge base (all `.md`: docs/, README files, etc.).

User query: the text passed with this invocation (`$kb <query>`).

`G="python3 <skills>/kb-search/gitmark.py"` — `gitmark.py` lives in the sibling
`kb-search` skill folder (repo checkout: `skills/kb-search/gitmark.py`). Run from the
repo root.

Do:
1. If the query is empty — run `$G stat` and show the `$kb <query>` syntax.
2. Otherwise:
   - Refresh the index if needed: `$G index`.
   - `$G search "<query>" -k 8`
   - Summarize the top hits (which files, `file:line`), don't reprint whole snippets. Open the 1–2 most relevant files and answer the question.
