# Next session

Written at the close of the build session that created this wiki (branch merged at `9fd72f7`, docs and records at `313d7d7`). It records what is left, in the order that costs least to do wrong.

## Already done — do not redo

- The wiki is built, reviewed, merged to `main`, and pushed. Four tool profiles, four evidence notes, a scoreboard, four pattern cards, four backlog candidates.
- `README.md`, `CLAUDE.md`, and `AGENTS.md` describe the implemented state. `AGENTS.md`'s two conflicting rules were narrowed.
- The build ledger and twelve per-task verification reports are preserved in `docs/superpowers/reports/`.
- The raw-source verification rule is in `.wiki/schema.md`'s Source Conventions, in `CLAUDE.md`, and in `README.md`.

## 1. Run the lint — do this first

```
/wiki:lint --local
```

From the repository root. **A human has to type it.** It is a slash command, not a Skill, so no subagent and no automated main loop can invoke it — this was tested during the build and is why `.wiki/_index.md` still reads `Last lint: never` and `.wiki/log.md` carries an entry disclosing the deferral.

It goes first because a structural finding could change what the rest of this list should be. Afterwards, update `Last lint:` in `.wiki/_index.md` and append an entry to `.wiki/log.md` (append-only — never edit an existing entry).

## 2. Decide the three open score questions — as one batch

All three are recorded in `.wiki/wiki/references/rubric-scoreboard.md` under `## Where the Rubric Strained`, and listed in `CLAUDE.md` under "Open questions — do not silently resolve these".

- **Claude Code `discoverability`, currently `2`.** Its documented `paths` glob field gates automatic activation on both skills and rules — the same class of tool-computed mechanism that earned Cursor a `3`. Two reviewers independently judged this a real inconsistency. Note that raising it does not change the axis spread (3, 2, 3, 1 still spans 2); what changes is the narrative that Cursor is uniquely mechanical, which is why the `path-scoped-activation` card was reworded to attribute the gap to aggregation rather than to a missing mechanism.
- **The `composition` 3-vs-2 split** between Claude Code and Codex CLI. Codex CLI is capped at `2` on the stated grounds that its hooks run with no arbitration, but its own hook documentation describes deny-wins arbitration for exactly the conflict case. Both tools document union execution plus a stated precedence rule.
- **`observability`, `2` for all four tools.** Spread 0. An axis that cannot separate four very different tools either needs revision or needs a fifth, more divergent tool to test it against.

**Batch them.** The judgment is the expensive part; the edit is not. But any score that moves has to stay consistent in four places:

| Location | What changes |
|----------|--------------|
| `.wiki/wiki/references/rubric-scoreboard.md` | the cell, the spread column, and the `Reading the Spread` / `Where the Rubric Strained` prose |
| `.wiki/wiki/topics/<tool>.md` | the score cell and its justification |
| `README.md` | its copy of the score table |
| `CLAUDE.md` | the corresponding "Open questions" bullet |

Deciding to leave a score as it stands is also a result worth recording in `.wiki/log.md`, so a later session does not reopen it from scratch.

**Fold in one related nuance while deciding `composition`:** `.wiki/wiki/topics/claude-code.md`'s Portability section credits Claude Code with "deterministic hook merging." That remains defensible — merging is deterministic by documented precedence (`deny` > `defer` > `ask` > `allow`) — but the dedup half of it is narrower than it sounds, since a plugin's or skill's copy of a handler stays separate. `docs/superpowers/reports/final-fix-report.md` flagged the phrase and left it pending this decision.

## 3. Apply the p1 backlog candidate

`.wiki/inventory/candidates/add-path-scoped-rule-activation.md` — add `paths:` frontmatter to the five TypeScript/JavaScript-specific files under `~/.claude/rules/` (`coding-style.md`, `hooks.md`, `patterns.md`, `security.md`, `testing.md`), leaving the three stack-neutral ones (`agents.md`, `git-workflow.md`, `performance.md`) unconditional. Verify with `/context` in a non-TypeScript repository.

This changes the owner's live Claude Code configuration, not this repository. Only the candidate's `status:` needs updating here once applied — `proposed` to `ingested`, per the fixed vocabulary in llm-wiki's inventory reference.

The other three candidates are lower priority and independent: `audit-skill-progressive-disclosure` (p2), `separate-plugin-log-from-memory-store` (p3, an investigation before any refactor), `state-explicit-rule-precedence` (p3, only actionable once item 3 introduces overlapping globs).

## Deferred, with specs already written

- **`citation-verifier` skill.** Proposed during session wrap and deliberately not built. It would extract quoted passages, fetch each cited source by a raw method, normalize, match, and emit a triage table — matched / no-match / near-match — rather than a verdict or an auto-fix, because most naive mismatches are false positives. The build session checked 265 passages by hand across two sweeps and found 23 real defects. The design is written up in the wrap analysis; building it is mostly transcription.
- **The measurement pipeline (milestone 4).** Scoped in `docs/superpowers/specs/2026-08-04-agentic-tools-audit-wiki-design.md` and deliberately deferred to its own spec. It adds Python, `ruff`, `pytest`, and `.wiki/output/projects/ecosystem-metrics/`, and triggers all five obligations in `CLAUDE.md`'s "first real change" section. It is the largest item here and depends on nothing above it.
- **The SDD workspace** at `.superpowers/sdd/2026-08-04-agentic-tools-audit-wiki/` is git-ignored scratch and can be deleted whenever. The reason it was kept — that the ledger held the only copy of the raw-fetch finding — no longer applies; the ledger is committed at `docs/superpowers/reports/ledger.md` and the finding is in `.wiki/schema.md`.

## One habit worth keeping

Three of the thirteen findings in the final whole-branch review were wrong, all in the same direction: a summarizing fetch reported present text as absent. A review's false-positive rate is a signal about the review method, not only about the work under review. When several findings fail in the same way, suspect the instrument before the artifact.
