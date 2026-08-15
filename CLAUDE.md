# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository state

This repository holds a knowledge wiki at `.wiki/`, an llm-wiki local wiki registered in the hub at `/Users/kikyeongoh/Documents/opterk/llm-wiki` under `local_wikis`. Start at `.wiki/_index.md`; the rubric and evidence rules live in `.wiki/schema.md`.

The wiki is populated: four tool profiles (Claude Code, OpenAI Codex CLI, Cursor, LangGraph) in `wiki/topics/`, each citing a hand-written evidence note in `raw/notes/`; a cross-tool scoreboard in `wiki/references/`; four pattern cards in `wiki/concepts/`; four backlog candidates in `inventory/candidates/`. `README.md` carries the score table and the wiki's known limits.

There is still no build or test toolchain. The only repository-level checks are `git status --short` and `git diff --check`. Wiki operations run through llm-wiki commands with `--local` from the repository root.

`docs/superpowers/` holds the design spec, the implementation plan, and — under `reports/` — the build ledger and per-task verification reports. Read the ledger before reopening a decision; it records what was tried, what was rejected, and why.

## Working in the wiki

These rules are what get work rejected here. `.wiki/schema.md` is authoritative; this is the short version.

**Evidence before assertion.** Every quoted passage must appear in the page it cites. Three of the four profiles failed review on citation defects — a quote attributed to a page that did not contain it, an invented sentence, a count that did not match its list. The recurring failure is compressing two adjacent source sentences into one quotation, or rendering the gist in your own words, and leaving the result inside quotation marks. Quote sentences separately, or drop the quotation marks and present the summary as your own with the citation intact.

**Verify against raw sources, never a summarizing fetch.** A summarizing fetch reports present phrases as absent; three findings during this wiki's review were that artifact rather than real defects. Mintlify docs serve raw markdown when you append `.md` to the doc URL (`code.claude.com/docs/en/*`, `learn.chatgpt.com/docs/*`, `docs.langchain.com/oss/python/langgraph/*`). `cursor.com/docs/*.md` returns 404 — fetch the page with `curl` and search its raw HTML payload.

**Quantitative claims come from a command, not from memory.** If you state a count, run the command that produces it and inline that command beside the number. Two defects on this branch were counts recalled rather than measured. Make count words agree with what they enumerate.

**Score aggregation follows a stated convention.** `.wiki/schema.md` defines four score levels but not how to aggregate an axis whose mechanisms have mixed enforcement. `wiki/topics/cursor.md` settles it: `discoverability` gets a ceiling read — one tool-enforced path earns a `3` even if others remain model-judged — and the other five get a floor read, where one confirmed gap caps the score. State which read you used, per axis.

**A pattern card requires two sightings.** One sighting stays in its profile. Zero qualifying techniques is a legitimate result, not a reason to lower the bar. Portability to Claude Code is a second gate, and "the target already does this better" is a third — see "Considered, Not Promoted" in `wiki/concepts/_index.md`.

**Mark inferences as inferences.** Never present one as recorded fact.

**The owner's own plugins are evidence, but their source stays out of `raw/`.** Quote a line and cite the path; never copy their files in.

**`log.md` is append-only.** Never edit or reorder an existing entry. Correct a past entry by appending a new one.

## Open questions — do not silently resolve these

All previously recorded open questions (discoverability score aggregation, composition 3-vs-2 split, observability review, and lint execution) were resolved in the 2026-08-15 session. There are currently no open questions.

## AGENTS.md is authoritative

Read `AGENTS.md` before making changes — it is the repository's own contributor guide and takes precedence over generic defaults. Its binding decisions:

- **Planned layout** (create these only when implementation actually begins): outside `.wiki/`, code in `src/`, tests mirroring `src/` under `tests/`, sanitized fixtures in `tests/fixtures/`. Code that serves the wiki is the stated exception — it belongs in `.wiki/output/projects/<slug>/code/` with its tests alongside, matching how llm-wiki organizes project code. `docs/` is not planned — it already exists and holds specs, plans, and build reports under `docs/superpowers/`. Keep the root for project-wide files (`README.md`, manifests, tool config).
- **Do not commit** dependency directories or raw collection material. Measurement summaries under `.wiki/output/projects/*/data/` are the record and *do* belong in version control; anything a re-ingest can reproduce stays ignored.
- **Indentation**: two spaces for Markdown, YAML, and JSON. UTF-8, final newline, spaces over tabs unless the language demands otherwise.
- **Naming**: `kebab-case` for docs and shell files, `snake_case` for Python modules, `PascalCase` for exported types/classes.
- **Commits**: Conventional Commit subjects, one focused change per commit.
- **Secrets**: no tokens, private keys, raw customer data, or populated `.env` files. Ship `.env.example` with placeholders instead, and redact sensitive fields from audit artifacts before sharing.

## The first real change carries extra obligations

`AGENTS.md` requires that whichever change first introduces a language or framework must, in the same change:

1. Add the ecosystem's standard formatter and linter, with their config committed.
2. Add reproducible scripts (e.g. `npm test`, `pytest`) — not undocumented local aliases.
3. Document those commands in `README.md`.
4. Update `AGENTS.md`'s "Build, Test, and Development Commands" section to replace the placeholder git-only checks.
5. Update this file's **Repository state** section.

The measurement pipeline is the change that will trigger all five. It is scoped in the spec but deferred to its own spec, and nothing in `.wiki/output/projects/` exists yet.

Tests ship with every feature and bug fix, covering success, failure, and boundary cases. Fixtures must be minimal, deterministic, and stripped of credentials or customer data.
