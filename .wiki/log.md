# Wiki Activity Log

## [2026-08-04] init | Wiki initialized as a local wiki in agentic-tools-audit

## [2026-08-04] schema | Adopted the topic guide with six rubric axes and a 0-3 score scale

## [2026-08-04] compile | 1 source → claude-code profile scored on six axes

## [2026-08-04] compile | 1 source → codex-cli profile scored on six axes

## [2026-08-04] compile | 1 source → cursor profile scored on six axes

## [2026-08-04] compile | 1 source → langgraph profile scored on six axes

## [2026-08-04] compile | 4 profiles → rubric scoreboard with per-axis spread

## [2026-08-04] compile | 4 profiles → 4 pattern cards, 4 backlog candidates

## [2026-08-04] lint | Structure, indexes, and links verified; statistics recounted

## [2026-08-04] note | `/wiki:lint --local` was not invocable this session; the checks in the entry above were manual structural substitutes (structure, indexes, links, statistics recounted by hand), not a lint run. Lint remains outstanding; master index correctly records "Last lint: never".

## [2026-08-04] fix | Quotation-fidelity wave across all 4 notes and 4 cards. Every quoted passage was re-checked against the cited page's raw markdown or raw HTML payload rather than a rendered summary, after a reviewer flagged that summarizing fetches can report a present phrase as absent. 265 quoted strings of 30+ characters were extracted from those eight files and machine-matched against downloaded sources; every non-match was triaged by hand (most were frontmatter text, source paths, local `--help` output, the author's own scare-quotes, deliberately ellipsis-marked elisions, or artifacts of Cursor's fragmented HTML), and 23 quotations were rewritten. Quotations in the four topic profiles and the scoreboard were checked against the same corpora. Two critical fixes: Cursor's `globs` trigger, quoted as "when file paths match patterns in `globs`," is not on the page (real text: "When file matches a specified pattern" and "Auto-attached when a matching file is in context") — corrected in the Cursor note and both places in the path-scoped-activation card, with the "mechanical, tool-computed" gloss moved outside the quotation marks; and Codex CLI's resident skill-list budget, quoted as one stitched sentence, is two sentences whose stitch dropped the qualifier that 8,000 characters is a fallback used only when the context window is unknown — corrected in the Codex note, the codex-cli profile (quoted and unquoted forms), and the deferred-reference-loading card. Three reviewer findings were rejected as incorrect after direct source checks: LangGraph's "the default reducer discards the left argument and keeps only the right" is verbatim on the graph-api page (a second, differently worded sentence about the same default also exists); the `Send` problem statement and solution are verbatim and contiguous; and the context-management list really does have five bullets with "Custom strategies" as the fifth. Framing fixes: the Cursor `discoverability` gap is now attributed to axis aggregation rather than to Claude Code lacking `paths`, cross-referenced to the scoreboard's open finding; the local rule-file count is corrected to eight files carrying no `paths`, five of them TypeScript/JavaScript-specific; manual `@`-mention is no longer counted among Cursor's tool-computed checks; and the specificity-ordered-precedence card now lists the Claude Code note it was already quoting. No score changed — all twenty-four scoreboard cells are untouched.
