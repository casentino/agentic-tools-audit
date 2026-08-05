# agentic-tools-audit

A personal knowledge library comparing how agentic tools express extensions, kept to inform Claude Code plugin development.

The content lives in `.wiki/`, an [llm-wiki](https://github.com/nvk/llm-wiki) local wiki. Start at `.wiki/_index.md`. It is a *local* wiki registered in the llm-wiki hub at `/Users/kikyeongoh/Documents/opterk/llm-wiki` under `local_wikis`, so the hub can find it while the content stays versioned in this repository.

Every claim in the wiki is meant to trace to a source. `.wiki/raw/` is where those sources live, which is why it leads the table below.

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
| `docs/superpowers/` | Design specs and implementation plans |

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

## Conventions

`AGENTS.md` governs the repository. Inside `.wiki/`, llm-wiki conventions govern.
