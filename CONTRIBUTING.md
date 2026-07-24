# Contributing

Improvements are welcome — this skill gets better every time someone encodes a lesson learned the hard way. A few rules keep it trustworthy:

## Ground rules

1. **Every language claim must be backed by evidence.** Cite either the official docs (link the specific page, e.g. `https://docs.guidedtrack.com/api/#collection-sort-direction`) or a tested GuidedTrack program ("verified in production on YYYY-MM-DD" with a short description of the test). Agents follow this guide literally; a plausible-but-wrong claim produces silently broken programs.
2. **Do not add keywords or syntax from memory.** If it isn't in the official docs and you haven't run it, it doesn't go in.
3. **`bin/gt` changes must stay portable** across macOS (BSD userland), Linux (GNU), and Windows Git Bash. Watch for the classics: BSD sed brace syntax (`1{/^$/d;}` needs the semicolon), bash process substitution (breaks native Windows jq.exe), unquoted variables (break program names with spaces). Test on at least one platform and say which in the PR.
4. **Keep the two files' roles distinct.** `SKILL.md` = workflow, tooling, pushing, data access. `references/complete_guide.md` = the GuidedTrack language itself. Don't duplicate content across them; cross-reference instead.
5. **Prefer patterns over prose.** A 10-line ```gt example teaches an agent more reliably than a paragraph. Use tabs (never spaces) inside ```gt blocks — agents copy them verbatim.
6. **Don't remove caveats you don't understand.** Warnings like "the search page is client-side rendered" or "the export excludes test runs" each cost someone real debugging time. If one seems wrong, test it and document the result rather than deleting it.

## Workflow

Branch, make your change, and open a PR describing (a) what an agent would do wrong without your change, and (b) how you verified the new content. Small focused PRs merge fast.
