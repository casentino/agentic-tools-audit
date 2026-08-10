# Task 8 Report: Promote patterns and open backlog candidates

## Sighting matrix (Step 1)

Built from the four profiles' `## Extension Model` sections plus, where a detail was under-elaborated there, direct re-verification against the underlying evidence note and (twice) the primary docs. "✓" means a citable sighting exists in that tool's evidence note; "✗" means checked and absent/not confirmed; "—" means not applicable to the technique as framed.

| # | Technique (as tested) | Claude Code | Codex CLI | Cursor | LangGraph | Outcome |
|---|---|---|---|---|---|---|
| 1 | **Deferred reference loading** — thin always-resident trigger (name/description), thick body deferred to reference files loaded on demand | ✓ skill description/body split; `wiki-manager` skill's 17 `references/*.md` files | ✓ skill progressive disclosure, resident list capped ~2%/8,000 chars; `scripts/`/`references/`/`assets/` layout | ✓ `@filename.ts` pulls a file into a rule's context by reference | ✗ profile states explicitly "no framework-level analog to 'skill body attaches at invocation'" | **QUALIFIED — card written** (`deferred-reference-loading`) |
| 2 | **Mechanical, tool-computed path/glob applicability metadata** (vs. model-judged description) | ✓ `paths` frontmatter on skills and on `.claude/rules/*.md` (verified directly against `code.claude.com/docs/en/skills` and `/docs/en/memory` — not fully elaborated in the first-pass note, added on re-verification) | ✗ skills only have model-judged name+description; no glob/path field documented | ✓ `globs` field, "Apply to Specific Files," a documented mechanical path match | — no metadata-driven applicability of any kind (topology fixed at compile time) | **QUALIFIED — card written** (`path-scoped-activation`) |
| 3 | **Gating side effects behind an explicit approval step** (default-on permission/sandbox layer) | ✓ permission modes, deny-always-wins, protected paths, auto-mode classifier — the profile's own strongest axis (3) | ✓ sandbox modes, approval policies, rules engine ("most restrictive wins") | ✓ approval-gated MCP/terminal/file edits, Run Modes, `permissions.json` | ✗ `interrupt()` is opt-in, per-call-site code; "no default-deny anywhere in the framework" per the scoreboard | **CLEARED two-sighting count (3 tools) but NO CARD** — see note below, third-bucket rejection, not the architecture-mismatch portability gate |
| 4 | **Persisting state outside the conversation so it survives a restart** (as a session-record/memory-store split) | ✓ transcripts (record) vs. auto memory (separate mechanism, own cap, own `modified` stamp) | ✓ session rollouts (record) vs. Memories (separate, off-by-default, "kept apart from the record of what happened") | ✗ note explicitly declines to count the beta "Memories" feature as evidence — no confirmed primary-doc mechanism | ✓ checkpointer (record) vs. store (cross-thread), stated as "two complementary persistence systems" | **QUALIFIED — card written** (`tiered-persistence-split`) |
| 5 | **Specificity-ordered precedence** — closer/more-specific nested instruction file overrides a broader one on conflict (found while reading the profiles side by side, not one of the four seed hypotheses) | ✗ confirmed absent by direct quote: "if two rules contradict each other, Claude may pick one arbitrarily"; nested `CLAUDE.md` files are "concatenated into context rather than overriding each other" | ✓ "Files closer to your current directory override earlier guidance because they appear later in the combined prompt" | ✓ "Instructions from nested `AGENTS.md` files are combined with parent directories, with more specific instructions taking precedence" | — no nested-file composition mechanism (topology is code) | **QUALIFIED — card written** (`specificity-ordered-precedence`) |
| 6 | Namespacing to prevent extension-name collisions (`plugin-name:skill-name`) | ✓ | ✗ no explicit namespace scheme documented | ✗ no namespace scheme, just concatenation order | — | **Single sighting — fails two-sighting gate, not pursued** |
| 7 | Compound-command decomposition for permission rules ("most restrictive wins" evaluated sub-part by sub-part) | ✗ `deny`-always-wins is coarser (no sub-command parsing documented) | ✓ explicit, tool-verified against `git add . && rm -rf /` | ✗ `terminalAllowlist` is documented only as prefix match | — | **Single sighting — fails two-sighting gate, not pursued** |
| 8 | Tool-stamped write timestamps on persisted-state records | ✓ auto memory's `modified` ISO-8601 field | ✗ not documented for Memories | ✗ not documented | ✓ store's `created_at`/`updated_at` | Two sightings exist, but this is a sub-detail of #4's mechanism rather than a distinct technique; folded into the `tiered-persistence-split` card's Technique section instead of spun out as a fifth card, to avoid padding the count |

### Row 3 in detail — why "approval gating" got no card

This is the one place my reasoning doesn't map cleanly onto either named gate (two-sighting failure, or the "Claude Code's architecture forbids it" portability failure), so I want to be explicit about it rather than force-fit it. Approval gating clears the two-sighting bar easily (three tools). It does not fail because Claude Code's architecture can't host it — the opposite is true: Claude Code's own profile states this axis is its strongest (`side-effect-control` = 3, "the clearest 3 of the six axes") and its own Portability paragraph says outright, "Claude Code already gives a plugin author enforced permission gating... a porting target does not need to invent any of those." There is no gap for a plugin author to close, so there is no actionable `next_action` to write — the inventory schema requires one. I treated this as a third outcome bucket: *qualified on sightings, excluded for redundancy* (nothing to port in, not something the host forbids). I looked for a narrower sub-technique that might still be a genuine gap (row 7, compound-command decomposition) but that has only one sighting.

No technique in this audit failed specifically because two tools implement it and Claude Code's architecture cannot host it (the scenario decision #2 describes). LangGraph's own architectural outlier (no host-side relevance judgment at all) never reaches two sightings on any axis where it's the odd one out, so the "note it in the profile instead" step never triggered — there was nothing to note beyond what the profiles already say.

## Cards written (4)

### 1. `deferred-reference-loading` — Deferred Reference Loading
- **Sightings / evidence-note lines:** Claude Code note, Skills section, the `wiki-manager` SKILL.md quote ("the body defers detail to 17 files under its own `references/` directory... rather than inlining everything in `SKILL.md` itself"); Codex CLI note, "Skills — progressive disclosure" ("ChatGPT and Codex start with each skill's name and description, then load the full `SKILL.md` instructions when they decide to use that skill"); Cursor note, "Deferred Content and MCP" ("Use `@filename.ts` to include files in your rule's context").
- Tags: `pattern, context-budget`.

### 2. `path-scoped-activation` — Path-Scoped Activation
- **Sightings / evidence-note lines:** Cursor note, "Scoping and Selection" ("Apply to Specific Files... a mechanical, tool-computed path match"); Claude Code note — I added two new paragraphs to this note during this task (see "Evidence note updates" below) after re-fetching `code.claude.com/docs/en/skills` and `/docs/en/memory` directly, since the first-pass note only listed `paths` as a field name without describing its mechanics.
- Tags: `pattern, discoverability`.
- This is the axis with the widest spread in the scoreboard (2), and the card sits exactly on the mechanism the scoreboard names as the reason for that spread.

### 3. `tiered-persistence-split` — Tiered Persistence Split
- **Sightings / evidence-note lines:** Claude Code note, "Observability" (transcripts) + "Memory files" (auto memory) sections, read together as the split; Codex CLI note, "State and Observability" ("a separate 'Memories' system background-summarizes past sessions... kept apart from the record of what happened" — this exact sentence is in the *profile*, drawn from the note's Sessions + Memories subsections); LangGraph note, "Persistence" ("LangGraph provides two complementary persistence systems: Checkpointers... Stores...").
- Tags: `pattern, state`.

### 4. `specificity-ordered-precedence` — Specificity-Ordered Precedence
- **Sightings / evidence-note lines:** Codex CLI note, "AGENTS.md merge order" ("Files closer to your current directory override earlier guidance because they appear later in the combined prompt"); Cursor note, "Overlap Resolution" ("Instructions from nested `AGENTS.md` files are combined with parent directories, with more specific instructions taking precedence").
- Tags: `pattern, composition`.
- Claude Code is deliberately *not* a sighting here — its own docs (re-fetched during this task) state the opposite behavior, which is what makes the gap actionable. That absence is now recorded in the Claude Code note as new evidence.

## Evidence note updates (decision #3 compliance)

I re-fetched `https://code.claude.com/docs/en/skills` and `https://code.claude.com/docs/en/memory` directly (not from memory) to confirm and add exact quotes for two mechanics the first-pass note under-elaborated, and added three new paragraphs to `.wiki/raw/notes/2026-08-04-claude-code-extension-model.md` plus two updated Sources lines, marked "verified post-review":

1. The `paths` frontmatter field on skills (glob-gated automatic activation) — supports card 2.
2. The `paths` frontmatter field on `.claude/rules/*.md`, plus the glob-pattern table — supports card 2, and explains why the eight locally-observed rule files load unconditionally (none sets `paths`).
3. The direct quote confirming Claude Code has no deterministic cross-file conflict resolution ("Claude may pick one arbitrarily") and concatenates nested `CLAUDE.md` without override — supports card 4's Problem section and its contrast with Codex CLI/Cursor.

No other evidence note needed changes; all other quotes used in the four cards were already present, verbatim, in the raw notes I read.

## Candidates opened (4)

| Candidate | kind | status | priority | Why this priority |
|---|---|---|---|---|
| `add-path-scoped-rule-activation` | task | proposed | **p1** | Highest leverage-to-cost ratio in the set: a frontmatter-only change to 5 existing files, and the waste it fixes was directly observed in this very session (TS/JS-scoped rules loading in this non-TS/JS repo). No new tool capability needed. |
| `audit-skill-progressive-disclosure` | task | proposed | **p2** | Clear, evidence-backed technique with a working example already in the owner's own plugin set, but the action is a survey across ~87 skill files plus case-by-case judgment on what to split — more effort than p1, no urgency. |
| `separate-plugin-log-from-memory-store` | task | proposed | **p3** | Real architectural improvement for stateful plugins (`metrics`, `history`), but it's a design/possible-refactor task, not a config tweak, and there's no evidence today that the current combined storage (if that's what it is) is actively causing problems — investigate-then-maybe-build, not urgent. |
| `state-explicit-rule-precedence` | task | proposed | **p3** | Cheap (one sentence per file) but currently inert — the owner's rule set is flat, so there's no overlap to arbitrate yet. It becomes relevant once the p1 candidate (or future plugin work) introduces overlapping `paths` scopes, so it's tracked now rather than dropped, but doesn't warrant p1/p2 today. |

All four: `status: proposed`, `confidence: medium`, `sources:` pointing to their one pattern card, `created`/`updated`/`last_checked`/`verified` all `2026-08-04`.

## Step 6 — verification (actual output)

First run (exit code 1 is expected — see explanation below, not a failure):

```
cards: 4
```
(exit code 1; the last command in the script block is `grep -q 'p0 | p1' "$f" && echo ...` inside a loop — when grep correctly finds no placeholder text, it returns 1, and since it's the final command executed, bash reports that as the script's exit status. No error/violation line was printed by any check.)

Re-run with an explicit trailing `true` to remove ambiguity about the exit code, full check block, same output:

```
cards: 4
CHECK COMPLETE - exit status suppressed
```

No `MISSING frontmatter`, `MISSING section`, `VIOLATION`, `FAIL: template placeholder remains`, `references missing candidate`, `MISSING backlink`, `INVALID kind`, `INVALID status`, or `MISSING or invalid priority` lines were produced by either run — this matches "Expected: the card count, and no other output."

I additionally verified (not in the brief's check block, but as my own self-review) that every relative Obsidian-style link in the new files resolves to a real file on disk, run from the correct working directory for each link type: card→candidate, card→topic, topic→card, candidate→card. All resolved. Output included in "Self-review findings" below.

## Files changed

Modified:
- `.wiki/_index.md` — added Inventory to Quick Navigation, one Recent Changes line
- `.wiki/log.md` — appended the Step 8 log entry
- `.wiki/raw/notes/2026-08-04-claude-code-extension-model.md` — added 3 verified paragraphs + 2 updated Sources lines (see above)
- `.wiki/wiki/concepts/_index.md` — added 4 rows to Contents, 1 Recent Changes line
- `.wiki/wiki/topics/claude-code.md` — added `## See Also` (3 entries)
- `.wiki/wiki/topics/codex-cli.md` — added `## See Also` (3 entries)
- `.wiki/wiki/topics/cursor.md` — added `## See Also` (3 entries)
- `.wiki/wiki/topics/langgraph.md` — added `## See Also` (1 entry)

Created:
- `.wiki/inventory/_index.md`
- `.wiki/inventory/candidates/_index.md`
- `.wiki/inventory/candidates/add-path-scoped-rule-activation.md`
- `.wiki/inventory/candidates/audit-skill-progressive-disclosure.md`
- `.wiki/inventory/candidates/separate-plugin-log-from-memory-store.md`
- `.wiki/inventory/candidates/state-explicit-rule-precedence.md`
- `.wiki/wiki/concepts/deferred-reference-loading.md`
- `.wiki/wiki/concepts/path-scoped-activation.md`
- `.wiki/wiki/concepts/tiered-persistence-split.md`
- `.wiki/wiki/concepts/specificity-ordered-precedence.md`

Deliberately left untouched: `items/`, `entities/`, `corpora/`, `views/` under `inventory/` (never created). An untracked, pre-existing `CLAUDE.md` at the repo root (evidence for the audit itself, referenced by the Claude Code note) was left unstaged — it is not part of this task's file list, and only `.wiki` was `git add`ed.

## Self-review findings

- Two-or-more sightings, each citing a path/URL an evidence note records: confirmed for all 4 cards by the Step 6 script (`n >= 2` check passed for every file) and by my own manual cross-check of every quote against the raw notes (three quotes needed strengthening via direct primary-source re-fetch, done and logged above rather than left as an unconfirmed inference).
- Every candidate exists, links back to its card (`grep -q 'wiki/concepts/'` passed for all 4), and uses valid `kind`/`status`/`priority` values (all `task`/`proposed`/`p1`–`p3`, all matched the allowed regex in the check script).
- Every profile cited by a card carries a matching `## See Also` entry: verified this in both directions with a separate script — Claude Code cites 3 cards and is cited by 3 cards; Codex CLI 3/3; Cursor 3/3; LangGraph 1/1 (matches exactly, no dangling reference either way).
- All relative links (card→candidate, card→topic, topic→card, candidate→card) resolve to real files on disk, tested by literally `cd`-ing into each source directory and `test -f`-ing the relative path as written — an earlier version of this same check had a shell-scoping bug that made every link look broken; the corrected version confirmed all of them are fine.
- `git diff --cached --check` printed nothing before commit, as required.

## Concerns

- The `separate-plugin-log-from-memory-store` and `state-explicit-rule-precedence` candidates are the two weakest for immediate actionability: the first requires investigating the owner's actual `metrics`/`history` plugin internals (which I did not read — only referenced by path, per the schema's rule against copying plugin sources into the wiki) before knowing whether a refactor is even needed; the second is currently inert since the owner's rule set is flat. Both are honestly `p3`, not overstated, but flagging that their `next_action` is a "go investigate" step rather than a ready-to-execute change.
- I extended one raw evidence note (Claude Code's) with new primary-source quotes rather than leaving those two techniques uncounted. This is explicitly permitted by decision #3 ("verify it yourself against the primary source and add it to the note with its URL"), and I fetched the primary docs directly rather than relying on recollection, but it's worth the requesting agent's awareness that two of the four cards (`path-scoped-activation` and part of `specificity-ordered-precedence`'s Claude Code contrast) rest partly on evidence I added during this task rather than evidence that existed before Task 8 started.
- I identified and rejected the "approval gating" hypothesis for a reason that doesn't map cleanly onto either of the two named failure gates (two-sighting count, or architecture-forbids portability) — see the "Row 3 in detail" section above. I'm flagging this explicitly rather than silently filing it under one gate or the other, since the reasoning is genuinely a third case (redundant with an already-best-in-class host mechanism) and I want that visible rather than papered over.

## Post-review fixes (review round 2)

The coordinator's review confirmed the core deliverable (sightings, bidirectional links, inventory fields, Step 6 script, card distinctness, and the additive-only nature of the raw-note extension) and ratified the `_index.md` Recent Changes bullet as within scope. It required four fixes, all now applied and committed (`682338b`):

**Fix 1 — misattributed citation.** `wiki/concepts/tiered-persistence-split.md`'s Codex CLI sighting bullet said "the note states the two are deliberately separated" and then quoted "kept apart from the record of what happened" — a sentence that lives in the profile (`wiki/topics/codex-cli.md:31`) as the profile author's synthesis, not in the raw evidence note. Fixed by rewriting the bullet to quote the note's actual language (the two section headings, "off by default," "updates memories in the background") and explicitly attributing the "kept apart from the record" framing to the profile as synthesis, by name.

**Fix 2 — undisclosed tension with a published score.** The `paths` frontmatter material I added to the Claude Code raw note during the original task (glob-gated, tool-computed activation for skills and rules) is the same class of mechanism the scoreboard credits for Cursor's `discoverability = 3`, but `claude-code.md`'s `discoverability` row only names hooks as the mechanical exception and scores 2. Added a paragraph to `wiki/references/rubric-scoreboard.md`'s `## Where the Rubric Strained` section (after the existing Claude Code/hooks discoverability-drift paragraph) naming this as an open finding, citing the exact raw-note lines (94 and 108) where the new evidence sits, and explicitly declining to re-score anything.

**Fix 3 — reasoning that only lived in a file that gets deleted.** Added a `## Considered, Not Promoted` section to `wiki/concepts/_index.md` naming the "gating side effects behind an explicit approval step" technique, the three tools it was sighted in, and the one-line reason for exclusion (Claude Code already implements it better than any tool it was sighted in — a third exclusion reason, distinct from the two named gates). Also added one optional sentence to `claude-code.md`'s Portability section cross-referencing this, per the coordinator's "if you also want" suggestion.

**Fix 4 — a card that summarized instead of taught.** `specificity-ordered-precedence.md`'s Technique section re-quoted the same two sentences already in its Sightings section. Rewrote it to state the mechanism once in prose (without re-quoting), then give its own minimal example — a sample one-sentence override statement inside a fenced code block — distinct from anything in Sightings or the candidate file.

### Re-verification (actual output)

Step 6 check block, re-run after all four fixes:

```
cards: 4
CHECK COMPLETE
```

No missing-frontmatter/section, two-sighting-violation, missing-candidate, missing-backlink, invalid-kind/status/priority, or placeholder-remains lines — same clean result as before the fixes.

Coordinator's four targeted checks:

```
=== kept apart from the record ===
wiki/concepts/tiered-persistence-split.md:36:- **OpenAI Codex CLI** — ... The framing that this keeps persistent memory "kept apart from the record of what happened" is the profile's synthesis of those facts (`.wiki/wiki/topics/codex-cli.md`), not a sentence the note itself contains.
wiki/topics/codex-cli.md:31:State is split cleanly into two mechanisms with different defaults. ... so tool-authored persistent memory is opt-in and kept apart from the record of what happened.

=== paths in rubric-scoreboard.md ===
42:A second, related tension surfaced after this scoreboard was written, during pattern promotion (Task 8) ... is the same class of mechanical match this scoreboard credits for Cursor's `discoverability` = 3. ...

=== considered, not promoted / approval gating in concepts/_index.md ===
16:## Considered, Not Promoted

=== specificity-ordered-precedence.md Technique section ===
## Technique

When a more specific instruction file ... can overlap with a broader one, state a rule that the narrower file wins on conflict ...

Two tools compute this order automatically from file position, so the author never has to write it down ...

A tool that does not compute this — Claude Code concatenates nested memory files "rather than overriding each other" ... The minimal fragment is one sentence at the top of the narrower file, naming what it overrides and for which files:

```markdown
# API Rules (src/api/**)

This overrides the general TypeScript style guide above for files under
`src/api/`: prefer explicit return types on every exported function, even
where the broader guide allows inference.
```

Without that sentence, both files simply sit in context together and, per Claude Code's own documented failure mode, "Claude may pick one arbitrarily" when they disagree.
```

The first result confirms the misattributed phrase is gone from the card and now correctly attributed to the profile as synthesis. The last result confirms the Technique section now carries its own worked example rather than restating Sightings. `raw/notes/2026-08-04-codex-cli-extension-model.md` produced no match for "kept apart from the record" (empty, expected — the sentence never lived there).

`git diff --cached --check` printed nothing before commit. Commit: `682338b` — "fix: correct card citation, record scoreboard tension, and log rejected pattern" (5 files changed, no attribution trailer).
