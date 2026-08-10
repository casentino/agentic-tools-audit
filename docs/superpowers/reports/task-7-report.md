# Task 7 Report: Build the rubric scoreboard

## 4×6 score matrix as read off the profiles

| Axis | Claude Code | Codex CLI | Cursor | LangGraph | Spread |
|------|-------------|-----------|--------|-----------|--------|
| `discoverability` | 2 | 2 | 3 | 1 | 2 |
| `context-budget` | 3 | 3 | 2 | 2 | 1 |
| `composition` | 3 | 2 | 2 | 3 | 1 |
| `state` | 3 | 3 | 2 | 3 | 1 |
| `side-effect-control` | 3 | 3 | 3 | 2 | 1 |
| `observability` | 2 | 2 | 2 | 2 | 0 |

Source of each cell (line numbers as read):
- Claude Code column: `.wiki/wiki/topics/claude-code.md` lines 37–42 (2, 3, 3, 3, 3, 2)
- Codex CLI column: `.wiki/wiki/topics/codex-cli.md` lines 39–44 (2, 3, 2, 3, 3, 2)
- Cursor column: `.wiki/wiki/topics/cursor.md` lines 37–42 (3, 2, 2, 2, 3, 2)
- LangGraph column: `.wiki/wiki/topics/langgraph.md` lines 35–40 (1, 2, 3, 3, 2, 2)

All 24 cells were read directly from each profile's Rubric Scores table (Score column, the field between the axis backtick-name and the justification prose) — none reconstructed from memory of earlier reads.

## Cross-check runs (all four columns)

Ran the brief's Step 2 comparison loop against each profile in turn, substituting the profile filename and awk field number ($3 = Claude Code, $4 = Codex CLI, $5 = Cursor, $6 = LangGraph), from `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki`:

```
=== Cross-check: Claude Code (col 3) ===
(no output — all six axes match claude-code.md)
=== Cross-check: Codex CLI (col 4) ===
(no output — all six axes match codex-cli.md)
=== Cross-check: Cursor (col 5) ===
(no output — all six axes match cursor.md)
=== Cross-check: LangGraph (col 6) ===
(no output — all six axes match langgraph.md)
```

Also ran the placeholder/axis-presence/dual-link checks from Step 2 verbatim: no `[0-3]` placeholders remained, no `max minus min` remained, all six axis names present, all four `[[tool|` dual links present. All checks printed nothing (pass).

Independently re-verified the Spread column arithmetically with a small script (max − min per row): discoverability=2, context-budget=1, composition=1, state=1, side-effect-control=1, observability=0 — matches the table exactly.

## Per-axis spread

- **Widest:** `discoverability` (spread 2) — LangGraph=1, Claude Code=Codex CLI=2, Cursor=3.
- **Narrowest:** `observability` (spread 0) — all four tools score 2. This axis never discriminates.
- **Tied at spread 1:** `context-budget`, `composition`, `state`, `side-effect-control`.

The scoreboard's "Reading the Spread" section names `discoverability` as the clear widest, then picks `composition` and `side-effect-control` out of the four-way tie as the two most substantively interesting disagreements to hand off to Task 8 (unarbitrated-hooks gap vs. fully-resolved collisions; default-on enforcement vs. LangGraph's opt-in-only `interrupt()`).

## The three mandatory `Where the Rubric Strained` findings

**a. Aggregation convention — Claude Code and Codex CLI are silent.**
Grepped both files for "ceiling", "floor", "aggregat" (case-insensitive): zero hits in `claude-code.md` and zero hits in `codex-cli.md`. Cursor and LangGraph both state the convention explicitly (Cursor line 44: ceiling read for `discoverability`, floor read for the other five, because discoverability asks "does *any* enforced path exist" vs. the others asking whether enforcement covers the full scope; LangGraph line 42 names the identical convention explicitly by the same words).

Reading Claude Code's and Codex CLI's five non-`discoverability` justifications for *implicit* method: both look consistent with the floor-read-with-*confirmed*-gap logic Cursor later put into words. Specifically:
- Claude Code's `composition` (scored 3) explicitly distinguishes a merely *inferred* absence ("still an inference from absence rather than a confirmed one") from a confirmed gap, and does not cap the score for it — the same distinction Cursor draws when it *does* cap its own `composition` at 2 for a gap it calls "confirmed."
- Codex CLI's `composition` (2) and `observability` (2) each cap cleanly on a plainly confirmed, stated gap (hooks explicitly documented as unarbitrated; introspection surfaces explicitly documented as user-invoked, none running by default) — matching the floor-with-confirmed-gap pattern exactly.

`discoverability` is the one axis where this cannot be confirmed, and Claude Code is a genuine open question rather than a clean match: its justification says hooks match "mechanically on event+matcher" (language that reads as an enforced path) yet the axis still scores 2, with hooks called "the exception" rather than the mechanism that sets a ceiling. Under Cursor's stated ceiling rule ("does any enforced path exist"), that description would argue for a 3. Codex CLI's `discoverability` is not a useful test either way, because its text explicitly excludes hooks and `AGENTS.md` from counting as discovery mechanisms at all ("they fire/load unconditionally"), leaving only one candidate mechanism — ceiling and floor reads agree by default in that case.

**Conclusion reported in the file:** the five non-discoverability axes in both undated profiles read as consistent with the now-stated convention; `discoverability` cannot be confirmed either way from the text, and Claude Code's case in particular reads as a plausible (not certain) inconsistency with the ceiling rule as later stated. This is reported as an open finding, not corrected — no profile was edited.

**b. Seed set and the rubric's low end.**
The plan chose Codex CLI to anchor the floor on the grounds its surface was "deliberately thin." Actual scores: Codex CLI = 2/3/2/3/3/2, which matches or beats Cursor's 3/2/2/2/3/2 on five of six axes (`context-budget` 3>2, `composition` 2=2, `state` 3>2, `side-effect-control` 3=3, `observability` 2=2; it loses only on `discoverability`, 2<3) and ties Claude Code's 3s on three axes. No cell anywhere in the 24-cell matrix reads `0` ("no such mechanism") — the scale's bottom rung is never exercised by any of the four tools. The only `1` in the matrix is LangGraph's `discoverability`, and per LangGraph's own profile text this comes from a structural mismatch (a compiled graph has no host-side relevance-judging moment at all), not from thinness. So the floor the scoreboard actually has came from a different tool, for a different reason, than the plan anticipated. Recorded as a candidate concern for the next tool to be profiled: pick one that can plausibly land a real `0`, or accept the rubric's 1–3 band is what this seed set can show.

**c. Axis that never discriminates.**
`observability` scores 2 for all four tools — spread 0, confirmed both from the table and independently via script. Each tool has a real documented introspection surface and a named gap that keeps it off 3; flagged in the file as a strong candidate for revising the axis (four structurally different tools converging exactly is either a scale-granularity problem or a sign this seed set hasn't found a tool divergent enough to test it).

## Files changed

- Created: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/wiki/references/rubric-scoreboard.md`
- Modified: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/wiki/references/_index.md` (added Contents row, kept `Last updated: 2026-08-04`)
- Modified: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/log.md` (appended the brief's exact log line)
- Untouched (verified via `git status --porcelain .wiki/_index.md`, no output): `.wiki/_index.md` master index

## Self-review findings

- All 24 cells verified twice: once via the Step 2 grep/awk cross-check (all four columns, all clean), once by re-reading the four profile tables directly against the scoreboard table side by side while drafting.
- Spread column re-verified arithmetically with an independent script; matches the table.
- `Where the Rubric Strained` covers all three mandatory items (aggregation silence in Claude Code/Codex CLI with an honest "cannot be told from the text" verdict on `discoverability`; the falsified Codex-CLI-as-floor premise and where the actual floor/1 came from; the `observability` axis with spread 0).
- No profile file was edited under any circumstance.
- `sources:` in frontmatter lists the four raw notes, not the four profiles, per decision 5.
- All date fields (`created`, `updated`, `verified`, log entry, index row) are `2026-08-04`; caught and corrected one slip where I initially wrote `Last updated: 2026-08-05` (today's actual date) into `_index.md` before fixing it back to `2026-08-04` per decision 6.
- `git diff --cached --check` printed nothing before commit (exit 0). Staged only `.wiki/wiki/references` and `.wiki/log.md`, per Step 4. Commit message is exactly `feat: compile rubric scoreboard across four tools`, no attribution/co-author trailer.
- Commit created: `508efaa` — "feat: compile rubric scoreboard across four tools" (3 files changed, 59 insertions).

## Concerns

- A pre-existing untracked `CLAUDE.md` file sits at the repo root (dated 08-04, not created by me this session, out of scope for this task). Left untouched and unstaged.
- The `discoverability` aggregation question for Claude Code is reported as genuinely open (not resolved) — Task 8 or a later fix round may want to revisit whether Claude Code's `discoverability` score should be reconsidered under the now-explicit ceiling convention. This task did not change the profile, per instructions.

## Fix round (post-review)

The reviewer confirmed all 24 cells, independently re-derived all six spread values, judged the three mandatory `Where the Rubric Strained` items substantive, and confirmed the Claude Code `discoverability` inconsistency was a real finding rather than a false alarm. Two issues came back:

1. **Arithmetic error in item (b).** The scoreboard and this report both said Codex CLI "matched or beat Cursor... on four of six axes." Computed from the table, Codex CLI ties or beats Cursor on `context-budget` (3>2), `composition` (2=2), `state` (3>2), `side-effect-control` (3=3), and `observability` (2=2) — **five** of six, losing only on `discoverability` (2<3). Fixed in both places:
   - `.wiki/wiki/references/rubric-scoreboard.md`, `## Where the Rubric Strained`, item (b) paragraph — now reads "five of six axes" with the per-axis breakdown spelled out.
   - This report, item (b) above — corrected to match.
   No score, cell, or spread value in the `## Scores` table changed; this was prose-only.

2. **Self-contradiction in `## See Also`.** The brief's Step 1 template mandates the Codex CLI blurb read "— thin extension surface," but the file's own `Where the Rubric Strained` item (b) documents that this framing was falsified by research. Per the coordinator's explicit, recorded authorization, this is a deliberate, knowing deviation from the brief's verbatim template — made because the brief's own evidence rules forbid asserting something the file itself shows to be false. Changed the blurb to: **"a second CLI, scoring high across the board"** — accurate to the findings, asserts no thinness. The other three `See Also` blurbs (Claude Code, Cursor, LangGraph) were left exactly as the brief specifies, since they were not falsified.

Also took the optional suggestion: the aggregation paragraph in `Where the Rubric Strained` now names `.wiki/schema.md` (Score Scale) by name as the place the rubric's silence on aggregation lives, so a future reader can check provenance without hunting.

**Verification run (coordinator's exact commands):**
```
$ grep -n 'four of six\|five of six' wiki/references/rubric-scoreboard.md
42:...matching or beating Cursor's 3/2/2/2/3/2 on five of six axes (...)...

$ grep -n 'thin extension surface' wiki/references/rubric-scoreboard.md
(no output)

$ grep -n '^| `' wiki/references/rubric-scoreboard.md
23:| `discoverability` | 2 | 2 | 3 | 1 | 2 |
24:| `context-budget` | 3 | 3 | 2 | 2 | 1 |
25:| `composition` | 3 | 2 | 2 | 3 | 1 |
26:| `state` | 3 | 3 | 2 | 3 | 1 |
27:| `side-effect-control` | 3 | 3 | 3 | 2 | 1 |
28:| `observability` | 2 | 2 | 2 | 2 | 0 |
```
All three checks match the expected result exactly: only "five of six" appears, "thin extension surface" returns nothing, and the six score rows are byte-identical to before the fix.

Staged only `.wiki/wiki/references/rubric-scoreboard.md` (confirmed via `git status --porcelain` — `.wiki/log.md` and `.wiki/wiki/references/_index.md` were not touched in this round, matching the coordinator's "stage only the scoreboard file" instruction). `git diff --cached --check` printed nothing (exit 0). Committed as a new commit (not an amend), Conventional Commit subject, no attribution trailer:

```
6a81d0f fix: correct scoreboard arithmetic and stale Codex CLI blurb
```

Two commits now exist for this task: `508efaa` (original scoreboard) and `6a81d0f` (this fix round).
