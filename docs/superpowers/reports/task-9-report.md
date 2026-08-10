# Task 9 Report: Close out the wiki state

## Step 1 — Count commands and actual output

Run from `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki`:

```bash
echo "Sources:  $(find raw -name '*.md' ! -name '_index.md' | wc -l | tr -d ' ')"
echo "Articles: $(find wiki -name '*.md' ! -name '_index.md' | wc -l | tr -d ' ')"
echo "Candidates: $(find inventory/candidates -name '*.md' ! -name '_index.md' 2>/dev/null | wc -l | tr -d ' ')"
echo "Outputs:  $(find output -name '*.md' ! -name '_index.md' | wc -l | tr -d ' ')"
```

Output:

```
Sources:  4
Articles: 9
Candidates: 4
Outputs:  0
```

Verified by listing the actual matched files:

- Sources (4): `raw/notes/2026-08-04-claude-code-extension-model.md`, `raw/notes/2026-08-04-codex-cli-extension-model.md`, `raw/notes/2026-08-04-cursor-extension-model.md`, `raw/notes/2026-08-04-langgraph-extension-model.md`
- Articles (9): `wiki/concepts/{deferred-reference-loading,path-scoped-activation,specificity-ordered-precedence,tiered-persistence-split}.md`, `wiki/references/rubric-scoreboard.md`, `wiki/topics/{claude-code,codex-cli,cursor,langgraph}.md`
- Candidates (4): `inventory/candidates/{add-path-scoped-rule-activation,audit-skill-progressive-disclosure,separate-plugin-log-from-memory-store,state-explicit-rule-precedence}.md`
- Outputs (0): `output/` contains only `_index.md`

**Numbers written into `.wiki/_index.md`'s `## Statistics` block:**

```
- Sources: 4 raw documents
- Articles: 9 compiled wiki articles
- Candidates: 4 backlog candidates
- Outputs: 0 generated artifacts
- Last compiled: 2026-08-04
- Last lint: never
```

I added a `Candidates:` line since that layer now exists (not present in the placeholder block, and not literally spelled out in the brief's Step 1 write instruction, but the brief's own count commands compute it — see "Decisions" note below). `Last lint` was left as `never` rather than dated, since lint did not actually run (see Step 3).

## Step 2 — Index-discipline check output

```bash
cd .wiki
wc -c _index.md
grep -c 'raw/notes/2026' _index.md
```

Before my edit (baseline, for reference): 864 bytes (after Statistics edit, before the Recent Changes addition), 0 matches.
**After my edit and the commit (final state):**

```
    1093 _index.md
```
```
0
```

1093 bytes is well under the 4 KB threshold, and zero individual raw-file references appear in the master index — index discipline held.

## Step 3 — Lint: did not run, substitute performed

I attempted `/wiki:lint --local` via the Skill tool (`Skill(skill="wiki:lint", args="--local")`). It failed:

```
<error><tool_use_error>Unknown skill: wiki:lint</tool_use_error></error>
```

The `wiki@llm-wiki` plugin is enabled in `~/.claude/settings.json`, and its `lint.md` command exists on disk at `~/.claude/plugins/cache/llm-wiki/wiki/0.16.0/commands/lint.md`, but the command was not exposed as an invocable skill in this subagent session. **Lint was not run.** I disclosed this in the Recent Changes entry I added to `_index.md` and I am disclosing it here plainly.

**Substitute performed:** I read the plugin's own `references/linting.md` (rule catalog C1–C19) to build an equivalent manual check, then ran it with `find`/`python3` from `.wiki/`:

1. **C1 structure** — `config.md` exists, `schema.md` exists, `output/_index.md` exists; every wiki-managed subdirectory (`raw/`, `raw/{articles,data,notes,papers,repos}`, `wiki/`, `wiki/{concepts,references,theses,topics}`, `inventory/`, `inventory/candidates`) has its own `_index.md`. All OK.
2. **C1 frontmatter delimiters** — a Python pass over all 17 content files (4 raw + 9 wiki + 4 candidates, excluding `_index.md`/`config.md`/`log.md`/`schema.md`) confirmed every file opens with `---`, closes its frontmatter block, and parses as valid YAML. No issues.
3. **C2 required fields** — checked raw sources for `title, source, type, ingested, tags, summary`; wiki articles for `title, category, created, updated, tags, summary` plus a resolvable `sources:` or `compiled-from: conversation`, and valid `category` enum; inventory candidates for `title, kind, status, priority, created, updated, tags, summary` plus valid `kind`/`status`/`priority` enums. No missing/empty fields, no invalid enum values.
4. **C3 index consistency** — for every content subdirectory, compared its actual `.md` files against the links in its own `_index.md` Contents table. Every directory's index lists exactly its actual files — no missing entries, no dead entries. Also confirmed the master `_index.md`, `raw/_index.md`, `wiki/_index.md`, `output/_index.md`, and `inventory/_index.md` links all resolve.
5. **C4 / C4b link and source-provenance integrity** — resolved every wiki article's and candidate's `sources:` frontmatter entries relative to the wiki root: all 22 entries resolve to existing files. Checked "See Also" sections in all `wiki/concepts` and `wiki/topics`/`wiki/references` articles: all links resolve. Checked coverage: all 4 raw sources are referenced by at least one compiled article — no orphan sources.

A naive line-level `[text](path)` regex scan initially flagged 4 "broken links," but inspection showed these were prose quoting external paths (e.g., the wiki-manager plugin's own `references/ingestion.md` example, and a quoted docs URL fragment `/docs/en/memory#path-specific-rules`) inside earlier tasks' evidence notes — not actual internal wiki navigation links. These are pre-existing content from Tasks 3–8 that I did not modify; verified they are illustrative text, not dead links, and left them alone as out of scope for Task 9.

**Result: no critical issues found.** No `.wiki` content changes were made as a result of these checks (nothing was broken); this substitute activity is captured in the Recent Changes bullet added to `_index.md` and in the commit body context here.

## Inventory index — what I checked, what I changed, and why

The task context asserted `.wiki/inventory/_index.md` still carried a stale placeholder table (`task | 0 | —`) that I needed to correct. Per decision #1 ("every number comes from a command you run, not from what you believe the earlier tasks produced"), I did not take that assertion on faith. I read the file and cross-checked it against the actual candidate files:

```
| Kind | Count | Statuses |
|------|-------|----------|
| `task` | 4 | `proposed`: 4 |
```

I then verified each of the 4 candidate files' frontmatter directly:

```
add-path-scoped-rule-activation.md:       kind: task, status: proposed
audit-skill-progressive-disclosure.md:    kind: task, status: proposed
separate-plugin-log-from-memory-store.md: kind: task, status: proposed
state-explicit-rule-precedence.md:        kind: task, status: proposed
```

**The table already matched reality exactly** — `git log --follow` shows it was written this way in commit `2ecc89f` (the prior task), not left at the empty-layer placeholder. The context I was given about its staleness was incorrect for this repo's actual state. Per the same decision #1 principle, I did not "fix" a table that was not broken, and I did not bump `Last updated` (already `2026-08-04`, unchanged) since no content changed — bumping a date with no corresponding edit would misrepresent that something happened. **No edit was made to `.wiki/inventory/_index.md`.** I flag this explicitly since the brief's context assumed otherwise; this is exactly the kind of discrepancy the task asked me to verify by command rather than reason from history.

## Files changed

- `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/_index.md` — statistics recounted (Sources 4, Articles 9, Candidates 4 added, Outputs 0), `Last compiled: 2026-08-04` set, `Last lint` left as `never` (lint did not run), one new Recent Changes bullet added.
- `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/log.md` — appended `## [2026-08-04] lint | Structure, indexes, and links verified; statistics recounted` (append-only, no prior entries touched).
- `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/inventory/_index.md` — **not modified**; verified already correct (see above).

Commit: `87b31cc` — "chore: recount wiki statistics and verify structure" (2 files changed, 7 insertions, 3 deletions).

## Self-review findings

Re-ran the Step 1 count commands after the edit and commit:

```
Sources:  4
Articles: 9
Candidates: 4
Outputs:  0
```

These match the numbers written in `.wiki/_index.md` exactly (confirmed via `grep` on the committed file). Re-ran the Step 2 discipline checks post-commit: `wc -c _index.md` → 1093 bytes (well under 4 KB); `grep -c 'raw/notes/2026' _index.md` → 0.

Confirmed the ratified Recent Changes bullet ("2026-08-04: Promoted 4 portable patterns from the four profiles' Extension Model sections to `wiki/concepts/`, each with 2-3 sightings, and opened 4 backlog candidates in `inventory/candidates/`.") appears exactly once in the file — not duplicated, not removed. My own new bullet is a distinct, additive line about the recount/verification work.

`git diff --cached --check` printed nothing (exit 0) before the commit. Only `.wiki/_index.md` and `.wiki/log.md` were staged and committed; an unrelated untracked `CLAUDE.md` file at the repo root was left alone (out of scope, not part of `.wiki`).

## Concerns

1. **Lint genuinely did not run.** The `wiki@llm-wiki` plugin is enabled in the user's global settings and its `lint.md` command exists on disk, but it was not invocable as a skill from this subagent session (`Unknown skill: wiki:lint`). I do not know whether this is a subagent-scoping limitation or something else; a session with direct slash-command access (e.g., the main interactive session) should run `/wiki:lint --local` for real and reconcile with my manual substitute. I'm confident in the substitute's coverage (I followed the plugin's own C1–C4 rule catalog) but it cannot exercise C5–C19 checks that need broader judgment (tag hygiene, freshness scoring, project hygiene, etc.) — those don't appear to apply to this wiki yet (no `output/projects/`, no `datasets/`) but a real lint pass would confirm that authoritatively.
2. **The task context about the inventory index being stale was factually wrong** for this repo's current state (it was already correct, written correctly by the prior task's commit `2ecc89f`). I verified this by command per decision #1 rather than performing the "correction" the brief's framing implied I'd need to make. Flagging this loudly since it's exactly the kind of history-based assumption the task warned against trusting.
