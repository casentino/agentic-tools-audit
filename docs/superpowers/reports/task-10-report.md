# Task 10 Report: Document the repository and narrow the AGENTS.md rules

## llm-wiki URL: what I did and why

The brief's templates rendered llm-wiki as `[llm-wiki](https://github.com/casentino)` in both `README.md` and the `AGENTS.md` replacement text. `casentino` is the git user configured for this repository (confirmed via `git status` metadata and commit authorship), not a URL related to the llm-wiki plugin — copying it would have been a fabricated citation in a repository whose entire purpose is citation accuracy.

I verified a canonical URL instead of dropping the link:

1. `/Users/kikyeongoh/.claude/plugins/known_marketplaces.json` records how this machine's `llm-wiki` plugin was actually installed: `"llm-wiki": {"source": {"source": "github", "repo": "nvk/llm-wiki"}, ...}`.
2. The plugin's own manifest (`.claude-plugin/plugin.json` inside the cached plugin at version 0.16.0) names the author `nvk`, consistent with the marketplace record.
3. I confirmed the repository exists and is the right one with `gh repo view nvk/llm-wiki --json name,description,url,owner`, which returned:
   `{"description":"LLM-compiled knowledge bases for any AI agent. Parallel multi-agent research, thesis-driven investigation, source ingestion, wiki compilation, querying, and artifact generation.","name":"llm-wiki","owner":{"login":"nvk"},"url":"https://github.com/nvk/llm-wiki"}`
   The description matches llm-wiki's actual behavior in this repository (wiki compilation, ingestion, querying), so this is not just a same-named repo — it's the source plugin.

I used `https://github.com/nvk/llm-wiki` in both `README.md` and `AGENTS.md`. This is a verified canonical URL, not a guess, so I did not fall back to the "name it in plain text, no link" option.

## Verifying the `/wiki:*` command names

I checked the llm-wiki plugin's cached command definitions directly rather than trusting the brief's template:

- `find "$HOME/.claude/plugins/cache/llm-wiki" -maxdepth 6 -iname "*.md" | grep -i -E "command|query|lint|ingest"` located `commands/query.md`, `commands/lint.md`, and `commands/ingest.md` under `llm-wiki/wiki/0.16.0/commands/`.
- The plugin manifest names the plugin `wiki` (`"name": "wiki"` in `plugin.json`), so its slash commands are namespaced `/wiki:query`, `/wiki:lint`, `/wiki:ingest` — matching what the brief's README template used.
- I read all three command files. Each supports `--local` in its `argument-hint`:
  - `query.md`: `argument-hint: "<question> [--quick|--deep] [--raw] [--list] ... [--wiki <name>] [--local]"`
  - `lint.md`: `argument-hint: "[--fix] [--deep] [--include-archived] [--archived-only] [--wiki <name>] [--local]"`
  - `ingest.md`: `argument-hint: "<url|filepath|\"text\"> [--type ...] ... [--wiki <name>] [--local]"`, and its resolution logic explicitly states `--local` → `.wiki/` in CWD.

All three commands and the `--local` flag are real and documented as written in the brief's template, so I kept that section of `README.md` unchanged in substance.

## The three files as changed

### `README.md` (new file)

Created exactly per the brief's Step 1 template, with one change: the llm-wiki link uses the verified URL `https://github.com/nvk/llm-wiki` instead of the template's `https://github.com/casentino`. Sections: title/description, a pointer into `.wiki/_index.md`, a Layout table (verified every path listed actually exists — `.wiki/schema.md`, `.wiki/wiki/topics/`, `.wiki/wiki/concepts/`, `.wiki/wiki/references/`, `.wiki/inventory/candidates/`, `docs/superpowers/`), a Commands section (verified `/wiki:query`, `/wiki:lint`, `/wiki:ingest` with `--local`, plus the two git checks), and a Conventions section stating `AGENTS.md` governs the repository and llm-wiki conventions govern inside `.wiki/`.

### `AGENTS.md` (modified)

Replaced only the `## Project Structure & Module Organization` section per Step 2, with the same URL substitution as above. One additional wording change beyond the brief's template: the Step 4 check requires the literal substring `llm-wiki conventions govern` to appear in `AGENTS.md`, but the brief's own template text — even copied verbatim with any URL — does not produce that contiguous substring, because the markdown link `[llm-wiki](url)` sits directly between the word "llm-wiki" and the word "conventions" in "follows [llm-wiki](url) conventions; ... those conventions govern". I verified this by piping the brief's exact template sentence through `grep -q 'llm-wiki conventions govern'`, which returned no match.

To make the check pass without changing the paragraph's meaning, I reworded that clause from:
> "...and follows [llm-wiki](url) conventions; inside that directory, those conventions govern layout, frontmatter, and indexes."

to:
> "..., built with [llm-wiki](https://github.com/nvk/llm-wiki); inside that directory, llm-wiki conventions govern layout, frontmatter, and indexes."

This preserves the rule (llm-wiki conventions govern layout/frontmatter/indexes inside `.wiki/`) and satisfies the mechanical check. The rest of `AGENTS.md` (`Build, Test, and Development Commands`, `Coding Style`, `Testing Guidelines`, `Commit & Pull Request Guidelines`, `Security & Configuration`) is untouched, per decision 2 — the commands section still says no toolchain exists and still lists only `git status --short` and `git diff --check`.

### `CLAUDE.md` (new commit, file existed untracked)

Replaced `## Repository state` per Step 3. Before committing this, I independently verified the new section's claim that the wiki is "registered in the hub at `/Users/kikyeongoh/Documents/opterk/llm-wiki` under `local_wikis`" by reading `/Users/kikyeongoh/Documents/opterk/llm-wiki/wikis.json`, which contains:
```json
"local_wikis": [
  {
    "path": "/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki",
    "description": "Agentic tool extension mechanisms, rubric scores, and portable patterns."
  }
]
```
This confirms the statement is true, not merely copied from the brief. Deleted `## Domain note` entirely per decision 4. Kept `## AGENTS.md is authoritative` and `## The first real change carries extra obligations` unchanged — both still describe future obligations that hold.

## False statements found and corrected

- `AGENTS.md`: "This repository is in its bootstrap state: it contains no tracked source, tests, assets, or build configuration" — false; `.wiki/` is a large tracked knowledge base. Corrected by Step 2's replacement paragraph describing the wiki and the `src/`/`tests/`/`.wiki/output/projects/` split.
- `CLAUDE.md`: "`AGENTS.md` is the only tracked file (single commit, `339dd2d`); there is no source, test, build config, `README.md`, or dependency manifest yet" — false on multiple counts (there is now a large tracked `.wiki/` tree, multiple commits since `339dd2d`, and this task adds `README.md`). Corrected by Step 3's replacement.
- `CLAUDE.md`: `## Domain note` guessed the repository's scope from vocabulary alone ("confirm intent with the user rather than assuming a design") — superseded by the now-settled spec in `docs/superpowers/specs/`. Deleted per decision 4.
- Both templates' unverified `https://github.com/casentino` link for llm-wiki — corrected to the verified `https://github.com/nvk/llm-wiki` in both `README.md` and `AGENTS.md`.

I did not find any false statement in the untouched sections of `AGENTS.md` or in the `AGENTS.md`-is-authoritative / first-real-change sections of `CLAUDE.md` — those describe rules that still hold (no toolchain exists yet, the first language/framework change still carries the documented obligations).

## Check block run and actual output

First run (with the brief's template text copied literally aside from the URL fix) produced one failure:
```
FAIL: AGENTS.md lacks the .wiki/ exemption
```
I diagnosed this as the adjacency issue described above (verified with a standalone `grep -q` test against the literal template sentence, which also failed), reworded the `AGENTS.md` clause as described, and reran:

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
test -f README.md || echo "MISSING README.md"
grep -q '\.wiki/' README.md || echo "README does not point into .wiki/"
grep -q 'bootstrap state' AGENTS.md && echo "FAIL: AGENTS.md still claims bootstrap state"
grep -q 'only tracked file' CLAUDE.md && echo "FAIL: CLAUDE.md still claims one tracked file"
grep -q '## Domain note' CLAUDE.md && echo "FAIL: CLAUDE.md still carries the superseded domain guess"
grep -q 'llm-wiki conventions govern' AGENTS.md || echo "FAIL: AGENTS.md lacks the .wiki/ exemption"
```

Output: none (all six checks passed silently, matching "Expected: no output").

`git diff --cached --check` after staging also printed nothing (exit 0).

## Self-review

**`README.md` as a first-time visitor**: It states what the repository is (a personal knowledge library comparing agentic-tool extension mechanisms), where the content lives (`.wiki/`, with an entry point at `.wiki/_index.md`), what each major path holds (Layout table, all six paths verified to exist), how to interact with it (`/wiki:query`, `/wiki:lint`, `/wiki:ingest --local`, both verified real commands supporting `--local`), what checks exist at the repo level (`git status --short`, `git diff --check`), what's deliberately absent (no build/test toolchain yet, with a note on when that changes), and which conventions apply where (`AGENTS.md` at the root, llm-wiki inside `.wiki/`). I believe this answers "what is this, where does it live, how do I work with it" without requiring the reader to open any other file first.

**`AGENTS.md` and `CLAUDE.md` re-read after edits**: No remaining statement in either file describes a bootstrap/empty repository, a single tracked file, or an unsettled domain guess. Every other section (commands, coding style, testing, commit conventions, security in `AGENTS.md`; the authoritative-precedence and first-real-change sections in `CLAUDE.md`) still describes present-tense-true or explicitly-future obligations, none of which this task's scope touches.

**`.wiki/` untouched**: I only read files under `.wiki/` (to verify README table paths and the local-wiki registration) and never wrote to it. `git status --short` after the commit shows no pending changes.

## Concerns

- The one non-cosmetic deviation from the brief is the reworded `AGENTS.md` sentence (Step 2). It was necessary because the brief's own template text cannot satisfy its own Step 4 check — I verified this independently before changing anything, rather than assuming the check or the template was wrong. The meaning is unchanged; only the sentence structure moved the link away from the "llm-wiki conventions govern" phrase the check requires.
- No other concerns. The llm-wiki URL was independently verified (not guessed), the three `/wiki:*` command names and their `--local` support were confirmed against the plugin's own cached command files, and the hub registration claim in `CLAUDE.md` was checked against `/Users/kikyeongoh/Documents/opterk/llm-wiki/wikis.json` rather than taken on faith from the brief.

## Commit

`b3f0ccb docs: document the wiki layout and reconcile repository conventions` on `feat/agentic-tools-wiki`, containing `README.md` (new), `AGENTS.md` (modified), `CLAUDE.md` (new — first commit of a previously untracked file). No attribution or co-author trailers, per instruction.

---

## Addendum: post-review corrections (commit `068c2d1`)

The review approved the above, then the coordinator relayed three follow-up corrections found by the reviewer's independent check. All three are addressed in a second commit, `068c2d1`.

### 1. `CLAUDE.md` — false "planned" claim about `docs/`

The `## AGENTS.md is authoritative` bullet said: "**Planned layout** (create these only when implementation actually begins): code in `src/`, tests mirroring `src/` under `tests/`, docs in `docs/`, sanitized fixtures in `tests/fixtures/`." This told a reader not to create `docs/` yet, while the same commit's `README.md` documents `docs/superpowers/` as an already-populated, present-tense location (`docs/superpowers/plans/` and `docs/superpowers/specs/` are tracked and real). The bullet mixed a true claim (`src/`, `tests/`, `tests/fixtures/` are still genuinely absent) with a false one (`docs/` is not planned — it exists).

Fix: split the tense within the same bullet rather than deleting it —

> "- **Planned layout** (create these only when implementation actually begins): code in `src/`, tests mirroring `src/` under `tests/`, sanitized fixtures in `tests/fixtures/`. `docs/` is not planned — it already exists and holds design specs and plans at `docs/superpowers/`. Keep the root for project-wide files (`README.md`, manifests, tool config)."

This keeps the still-accurate planned-layout guidance for `src/`/`tests/`/`tests/fixtures/` intact while correcting the `docs/` claim, per the coordinator's explicit instruction not to remove the bullet.

### 2. `AGENTS.md` — false "no commit history" claim

The `## Commit & Pull Request Guidelines` section opened: "The repository has no commit history from which to infer a house style." This was outside my Step 2 brief scope, but the repository now has more than twenty commits in an established Conventional Commits style (visible via `git log`), so the sentence was false and inconsistent with this task's goal of leaving no bootstrap-era claim standing.

Fix: replaced only that sentence with "The repository's commit history already establishes a Conventional Commit house style." Left the rest of the section (the example subjects, "Keep each commit focused," the PR guidance) untouched, per instruction.

Verification: `grep -n 'no commit history' AGENTS.md` returns nothing (exit code 1) after the fix.

### 3. `.wiki/log.md` — misleading unqualified `lint` entry

The reviewer read the existing entry `## [2026-08-04] lint | Structure, indexes, and links verified; statistics recounted` and concluded lint had actually run — it had not; `/wiki:lint --local` was not invocable in that earlier session, which instead ran manual structural substitutes and correctly left `Last lint: never` in the master index. Only this log line, copied from the plan's template verb, implied an actual lint operation.

Per the coordinator's explicit, scoped exception to the "do not touch `.wiki/`" rule, I appended one new entry to `.wiki/log.md` — the file is append-only, so the existing entry was not edited, only followed by a new line:

```
## [2026-08-04] note | `/wiki:lint --local` was not invocable this session; the checks in the entry above were manual structural substitutes (structure, indexes, links, statistics recounted by hand), not a lint run. Lint remains outstanding; master index correctly records "Last lint: never".
```

I chose `note` as the operation verb specifically because it does not match any operation name used elsewhere in the log (`init`, `schema`, `compile`, `lint`) and cannot be mistaken for a second lint run. `git diff .wiki/log.md` (before committing) showed a pure two-line addition with `+` markers only — the prior `lint` entry's line was untouched, byte-for-byte identical before and after. No other file under `.wiki/` was touched, and `git ls-files .wiki | wc -l` still reports 39 tracked files.

### Verification block run (post-correction)

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
grep -n 'docs in `docs/`\|Planned layout' CLAUDE.md
grep -n 'no commit history' AGENTS.md
tail -4 .wiki/log.md
git diff --stat b3f0ccb HEAD
```

Actual output:

```
=== 1 ===
17:- **Planned layout** (create these only when implementation actually begins): code in `src/`, tests mirroring `src/` under `tests/`, sanitized fixtures in `tests/fixtures/`. `docs/` is not planned — it already exists and holds design specs and plans at `docs/superpowers/`. Keep the root for project-wide files (`README.md`, manifests, tool config).
=== 2 (must be empty) ===
exit2=1
=== 3 ===

## [2026-08-04] lint | Structure, indexes, and links verified; statistics recounted

## [2026-08-04] note | `/wiki:lint --local` was not invocable this session; the checks in the entry above were manual structural substitutes (structure, indexes, links, statistics recounted by hand), not a lint run. Lint remains outstanding; master index correctly records "Last lint: never".
=== 4 ===
 .wiki/log.md | 2 ++
 AGENTS.md    | 2 +-
 CLAUDE.md    | 2 +-
 3 files changed, 4 insertions(+), 2 deletions(-)
```

The second command (grep for "no commit history") returned nothing, as required. The `git diff --stat` confirms only the three intended files changed, with `.wiki/log.md` showing pure additions (`2 ++`, no deletions) — consistent with a byte-identical prior entry plus one appended line.

`git diff --cached --check` printed nothing before this commit.

### Commit (addendum)

`068c2d1 docs: correct residual bootstrap claims and flag the unqualified lint log entry` on `feat/agentic-tools-wiki`, containing `CLAUDE.md`, `AGENTS.md`, and `.wiki/log.md` (one appended line only). No attribution or co-author trailers.

### Concerns (addendum)

None. All three corrections were independently verified (grep checks, `git diff` inspection for the append-only file, tracked file count) before committing.
