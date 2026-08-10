# agentic-tools-audit

A personal knowledge library comparing how agentic tools express extensions, kept to inform Claude Code plugin development.

The content lives in `.wiki/`, an [llm-wiki](https://github.com/nvk/llm-wiki) local wiki. Start at `.wiki/_index.md`. It is a *local* wiki registered in the llm-wiki hub at `/Users/kikyeongoh/Documents/opterk/llm-wiki` under `local_wikis`, so the hub can find it while the content stays versioned in this repository.

Every claim in the wiki is meant to trace to a source. `.wiki/raw/` is where those sources live, which is why it leads the table below.

## What the wiki contains

Four tools are profiled, each against a six-axis rubric defined in `.wiki/schema.md`. Scores run `0` (no such mechanism), `1` (implicit convention), `2` (documented convention), `3` (the tool enforces or verifies it).

| Axis | Claude Code | Codex CLI | Cursor | LangGraph | Spread |
|------|-------------|-----------|--------|-----------|--------|
| `discoverability` | 2 | 2 | 3 | 1 | 2 |
| `context-budget` | 3 | 3 | 2 | 2 | 1 |
| `composition` | 3 | 2 | 2 | 3 | 1 |
| `state` | 3 | 3 | 2 | 3 | 1 |
| `side-effect-control` | 3 | 3 | 3 | 2 | 1 |
| `observability` | 2 | 2 | 2 | 2 | 0 |

Each profile in `.wiki/wiki/topics/` cites a hand-written evidence note in `.wiki/raw/notes/`, and the scoreboard at `.wiki/wiki/references/rubric-scoreboard.md` reads its cells from those profiles rather than re-deriving them.

Techniques sighted in two or more tools were promoted to pattern cards in `.wiki/wiki/concepts/`, each paired with an actionable backlog item in `.wiki/inventory/candidates/`:

| Pattern card | Backlog candidate | Priority |
|--------------|-------------------|----------|
| `deferred-reference-loading` | `audit-skill-progressive-disclosure` | p2 |
| `path-scoped-activation` | `add-path-scoped-rule-activation` | p1 |
| `tiered-persistence-split` | `separate-plugin-log-from-memory-store` | p3 |
| `specificity-ordered-precedence` | `state-explicit-rule-precedence` | p3 |

A further technique — gating side effects behind an explicit approval step — cleared the two-sighting bar in three tools but was deliberately not promoted, because Claude Code already scores that axis at its ceiling and there is no gap left to close. The reasoning is recorded under "Considered, Not Promoted" in `.wiki/wiki/concepts/_index.md`.

## Known limits

The wiki records what it could not settle, rather than smoothing it over. All of this lives in the scoreboard's `## Where the Rubric Strained` section:

- **`observability` never discriminates.** All four tools score `2`. An axis that cannot separate four very different tools is a candidate for revision.
- **The rubric does not define aggregation.** `.wiki/schema.md` gives four score levels but never says how to score an axis whose several mechanisms have mixed enforcement. Two profiles state the convention they used; two predate it.
- **The seed set never touches the floor.** No cell reads `0`. The plan chose Codex CLI expecting a thin extension surface; research falsified that.
- **Two scores carry open questions.** Whether Claude Code's `discoverability` should be `3` given its documented `paths` field, and whether the `composition` 3-vs-2 split between Claude Code and Codex CLI survives scrutiny. Both are recorded as findings, not resolved.
- **`/wiki:lint --local` has never run.** The master index says `Last lint: never` and `.wiki/log.md` explains why.

## Layout

| Path | Holds |
|------|-------|
| `.wiki/raw/` | The evidence layer — one hand-written note per tool under `raw/notes/`, the source every article cites |
| `.wiki/schema.md` | The six-axis rubric, vocabulary, and evidence rules |
| `.wiki/wiki/topics/` | One profile per tool |
| `.wiki/wiki/concepts/` | Portable techniques, each sighted in two or more tools |
| `.wiki/wiki/references/` | The cross-tool scoreboard |
| `.wiki/wiki/theses/` | Thesis investigations (created, none written yet) |
| `.wiki/inventory/candidates/` | Proposed plugin changes |
| `docs/next-session.md` | What is left to do, in the order that costs least to do wrong |
| `docs/superpowers/specs/` | Design specs |
| `docs/superpowers/plans/` | Implementation plans |
| `docs/superpowers/reports/` | Build records — the ledger and per-task verification reports |

## Commands

All wiki commands run from the repository root with `--local`, which resolves the wiki to `.wiki/`:

```
/wiki:query "<question>" --local
/wiki:lint --local
/wiki:ingest <url|file> --local
```

Repository checks:

```bash
git status --short
git diff --check
```

No build or test toolchain exists yet. The measurement pipeline will add Python, `ruff`, and `pytest`; this file gains those commands at that point.

## Verifying a citation

Do not use a summarizing fetch to check whether a quoted phrase is on a page — it reports present phrases as absent, and three findings during this wiki's review turned out to be that artifact rather than real defects. Fetch the raw source:

- Mintlify-hosted docs serve raw markdown when you append `.md` to the doc URL. This works for `code.claude.com/docs/en/*`, `learn.chatgpt.com/docs/*`, and `docs.langchain.com/oss/python/langgraph/*`.
- `cursor.com/docs/*.md` returns 404; fetch the page with `curl` and search its raw HTML payload.

## Conventions

`AGENTS.md` governs the repository. Inside `.wiki/`, llm-wiki conventions govern.
