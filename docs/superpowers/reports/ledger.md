# SDD ledger — plan: docs/superpowers/plans/2026-08-04-agentic-tools-audit-wiki.md
branch: feat/agentic-tools-wiki
base: 8ea24187d8cd7b4d3f265f6767ff2f584f268974

Task 1: minor (deferred): .wiki/_index.md links schema.md which Task 2 creates — dead link until then (plan-mandated, verbatim brief text)
Task 1: fix round 1/5 (untracked .wiki/inbox/.processed — fix committed d51890e; scoped re-review dispatched; commits 4fb3e9b..d51890e)
Task 1: complete (commits 8ea2418..d51890e, review clean after 1 fix round; hub commit 1d8b5f9 verified by controller)
Task 1: minor RESOLVED by Task 2 — schema.md now exists, _index.md link no longer dead
Task 2: minor (deferred): schema.md `## State` section only restates frontmatter, and "state" collides with the `state` rubric axis name (plan-mandated, brief authored it)
Task 2: minor (deferred): Step 3 check block greps names only — cannot catch reworded axis questions or score meanings; weak evidence for future re-runs
Task 2: complete (commits d51890e..cc368c1, review clean, controller verified no attribution trailer on any branch commit)
Task 3: minor (deferred): evidence note drifts into synthesis at Observability section — interpretive framing belongs in the profile
Task 3: minor (deferred): note omits "type": "command" key when illustrating the statusLine JSON value — paraphrase, not misquote
Task 3: fix round 1/5 (3 addressed: evidence-chain quote, 18->17 file count, composition score 2->3 on newly surfaced enforced-precedence evidence; commits 835e8ff..d4f5294)
Task 3: complete (commits cc368c1..d4f5294, review clean after 1 fix round; final scores disc=2 ctx=3 comp=3 state=3 side=3 obs=2)

PLAN PREMISE FALSIFIED (Task 4): the plan's Seed tool selection table justifies Codex CLI as "a deliberately thin extension surface... the low end of the scale." Research found skills, hooks, a plugin marketplace, and memories; scores came in disc=2 ctx=3 comp=2 state=3 side=3 obs=2 — one axis away from Claude Code. No axis scored 0 or 1.
  Consequence: the seed set may not exercise the low end of the rubric at all. Task 7's "Where the Rubric Strained" section is the designated place to record this; the decision on whether to add a genuinely thin fifth tool belongs to the human and is deferred until Task 7 shows the empirical spread.
Task 4: minor (deferred): raw/notes/2026-08-04-codex-cli-extension-model.md:20 says "Four distinct kinds of extension unit" then lists five (AGENTS.md, Skills, MCP, Hooks, Plugins); profile correctly says Five. One-word fix, candidate for the final fix wave.
Task 4: complete (commits d4f5294..b0c8f04, review clean, no fix round; scores disc=2 ctx=3 comp=2 state=3 side=3 obs=2; reviewer independently fetched 12 cited URLs, all quotes verbatim)

RUBRIC GAP (found in Task 5 review): .wiki/schema.md defines the four score levels but NOT how to aggregate when one axis has several mechanisms with mixed enforcement. Task 5 used a ceiling read on discoverability (one enforced mechanism among four -> 3) and a floor read on the other four axes (one unenforced gap -> capped at 2). Tasks 3-4 have not been checked for which read they used.
  Consequence: Task 7's per-axis spread compares numbers that may be derived by different rules. Carry this into Task 7's dispatch as a mandatory item for its "Where the Rubric Strained" section. Codifying the rule in schema.md and re-checking all profiles is a human decision, deferred until Task 7 shows the spread.
Task 5: minor (deferred): cursor.md Portability is one long markdown line of three sentences — consistent with the precedent set by claude-code.md and codex-cli.md, flagged for awareness only
Task 5: fix round 1/5 (2 findings sent: Critical misattributed citation URL for the best-effort-guardrails quote; Important unstated aggregation asymmetry on discoverability)
Task 5: complete (commits b0c8f04..64bb03e, review clean after 1 fix round; scores disc=3 ctx=2 comp=2 state=2 side=3 obs=2)
Task 5: aggregation convention now ON RECORD in cursor.md — discoverability gets a ceiling read (does ANY enforced path exist), the other four get a floor read (does enforcement cover the full scope). Task 6 must state which read it uses per axis so Task 7 compares like with like.
Task 6: fix round 1/5 (2 Critical fabricated quotes: invented LangSmith sentence attributed to graph-api; invented "four named solutions" quote undercounting a five-item list. 4 Important: false "no error-on-conflict documented" claim reversed by InvalidUpdateError docs; two paraphrases inside quotation marks; omitted "static interrupts not recommended for human-in-the-loop" caveat. Plus a full quote-fidelity sweep of both files.)
Task 6: FOOTGUN RESOLVED by reviewer — LangGraph raises InvalidUpdateError on same-superstep concurrent writes to an unreduced key; silent override applies only to sequential writes across super-steps. Not an enforcement gap. composition stays 3; only the justification was wrong. This closes the open question the implementer had tried to defer to Task 7.
Task 6: minor (deferred): note line 37 splices two separate clauses from the Send page into one quoted phrase (same class as the Criticals; the sweep should catch it)
Task 6: fix round 1 applied after an API-error interruption (partial edits recovered, no work lost). Sweep verified 51 source-attributed citations, fixed 13 total: the 6 reviewer-named findings plus 7 further defects the sweep alone caught. Commit 8de6237.
Task 6: fix round 1 verdict — all 6 findings ADDRESSED and verified verbatim; reviewer spot-checked 3 of the 7 self-caught defects by fetching pages itself, all matched, confirming the sweep was real. Correction asides read transparently, not grep-shaped.
Task 6: fix round 2/5 (1 new Important introduced BY the fix: the Important-1 correction quote splices two source paragraphs, drops a leading "However," without ellipsis, and renders a colon as a period — same defect class the sweep claims to have eliminated. Claim itself verified accurate; only the quotation rendering is wrong.)
Task 6: minor (deferred): note line 32 renders "doesn't" with a straight apostrophe where the live source uses U+2019 — predates the round-2 fix, same quote-fidelity class, candidate for the final fix wave
Task 6: complete (commits 64bb03e..59209c1, review clean after 2 fix rounds; scores disc=1 ctx=2 comp=3 state=3 side=2 obs=2)

ALL FOUR PROFILES COMPLETE. Scores as committed:
  claude-code  disc=2 ctx=3 comp=3 state=3 side=3 obs=2
  codex-cli    disc=2 ctx=3 comp=2 state=3 side=3 obs=2
  cursor       disc=3 ctx=2 comp=2 state=2 side=3 obs=2
  langgraph    disc=1 ctx=2 comp=3 state=3 side=2 obs=2
Task 7: fix round 1/5 (1 Important: "four of six axes" is arithmetically five of six. Plus a CONTROLLER DEVIATION FROM THE BRIEF: the brief's Step 1 template mandates the See Also blurb "— thin extension surface" for codex-cli, but the same file's strained section records that framing as falsified by research. Two plan requirements collide; accuracy wins. Deviation recorded here rather than escalated, since it is a four-word blurb.)
Task 7: minor (deferred): See Also blurb tension was resolved by the authorized deviation; no residual
Task 7: complete (commits 59209c1..6a81d0f, review clean after 1 fix round; spread disc=2 ctx=1 comp=1 state=1 side=1 obs=0)
Task 7: OPEN QUESTION recorded in the wiki's "Where the Rubric Strained" — Claude Code's discoverability=2 may be a 3 under the now-explicit ceiling convention, since its own profile describes hooks matching mechanically. Two reviewers independently judged this a real inconsistency. Not fixed: changing it moves the spread and Task 8's search basis, and codifying the aggregation rule in schema.md is a human decision.

Task 8: 4 cards written (deferred-reference-loading, path-scoped-activation, tiered-persistence-split, specificity-ordered-precedence) + 4 candidates, all kind=task status=proposed p1-p3.
Task 8: CONTROLLER RATIFICATION — .wiki/_index.md got a Recent Changes bullet beyond my "Inventory line only" constraint. The constraint existed to protect Task 9's statistics recount; a Recent Changes bullet is not statistics and the line is accurate. Ratified, stays. Task 9 must not duplicate it.
Task 8: fix round 1/5 (4 Important: card cites profile synthesis as if it were note evidence; new `paths` evidence contradicts claude-code discoverability=2 and was recorded nowhere; the third selection gate "target already does it better" lives only in the SDD report which gets deleted; one card's Technique section restates its own Sightings)
Task 8: minor (deferred): raw note prose says "fetched 2026-08-05" while frontmatter dates stay 2026-08-04 — accurate, a consequence of the controller's date-freeze decision
Task 8: complete (commits 6a81d0f..682338b, review clean after 1 fix round; 4 cards + 4 candidates; third selection gate now documented in wiki/concepts/_index.md "Considered, Not Promoted"; paths-vs-discoverability tension recorded in the scoreboard)
Task 9: complete (commits 682338b..87b31cc, review clean, no fix round; counts sources=4 articles=9 candidates=4 outputs=0; master index 1093 bytes, 0 raw-file refs)
Task 9: OPEN — /wiki:lint --local never ran (subagents cannot invoke slash commands). Last lint stays "never" and the deferral is disclosed in the wiki's Recent Changes. Controller to run it in the main session before finishing.
Task 9: controller gave a WRONG instruction (claimed inventory/_index.md counts table was stale); implementer verified against the file, found it already correct, declined the edit, and reported the contradiction. Reviewer confirmed the implementer was right.
Task 10: fix round 1/5 (3 items: CLAUDE.md "Planned layout" bullet false for docs/ which already exists and is populated; AGENTS.md "no commit history from which to infer a house style" false after 20+ Conventional Commits; and a misleading .wiki/log.md lint entry.)
Task 10: LOG-ENTRY DEFECT — Task 9's log line "## [2026-08-04] lint | Structure, indexes, and links verified" uses the lint operation verb although /wiki:lint never ran. The Task 10 reviewer read it and stated in its report that "lint has in fact been run." A careful reader was demonstrably misled. Controller lifted the "do not touch .wiki/" constraint for exactly one appended log entry recording the deferral; the original entry stays byte-identical (log.md is append-only).
Task 10: minor (deferred): README does not mention the wiki is a local wiki registered in a personal hub — that context lives only in CLAUDE.md, which is written for agents rather than human readers
Task 10: minor (deferred): .wiki/wiki/theses/ exists on disk but is absent from README's layout table (inherited from the brief's template)
Task 10: complete (commits 87b31cc..068c2d1, review clean after 1 fix round)

ALL 10 TASKS COMPLETE. Branch feat/agentic-tools-wiki, 339dd2d..068c2d1.
Next: final whole-branch review, then controller runs /wiki:lint --local (still outstanding), then finishing-a-development-branch.

FINAL WHOLE-BRANCH REVIEW (opus, 339dd2d..068c2d1): verdict "with fixes".
  Verified clean: 24/24 scoreboard cells match profiles; all 6 spreads correct; every wikilink slug resolves; master index 1093 bytes with counts matching disk; hub registration intact; local evidence (codex 0.144.1, Cursor 3.14.7, 17 reference files, 87 SKILL.md) independently confirmed; ~20 of ~25 sampled quotes exact.
  2 Critical + 6 Important found: 4 quotations absent from the pages they cite (2 inside deliverable pattern cards; one alters a documented limit's meaning), a card contradicting its own evidence, a count contradiction, an incomplete Sources list, and CLAUDE.md still summarizing the un-narrowed AGENTS.md rules.
  KEY: one defective quote sits on a page Task 6's "51-citation sweep" claimed to have verified — that sweep was incomplete, so remaining citations carry less assurance than the ledger implied.
  Reviewer ruling: none of the three recorded rubric strains, nor the Claude Code discoverability open question, should block merge. Recording the falsified plan premise rather than rewriting it was called the branch's best judgment call.
  Fix wave dispatched (opus, one dispatch per the skill's rule). One scoped re-review follows; residuals get adjudicated, no second wave.

CORRECTION TO THE LEDGER'S FINAL-REVIEW ENTRY: the claim that "one defective quote sits on a page Task 6's sweep verified, so that sweep was incomplete" does NOT hold. That data point was finding I1 (LangGraph default reducer), which the fix wave verified IS verbatim on the page's raw markdown. The reviewer had been misled by WebFetch's summarizing render.
  METHOD FINDING (important, reusable): WebFetch renders pages through a summarizing model and will report a present phrase as absent. 3 of the 6 quote-related findings in the final review were this artifact (I1, M2, M5). Mintlify docs serve raw markdown by appending .md to the doc URL (works for LangGraph, Claude Code, Codex; Cursor 404s, use the raw HTML payload). Verification passes must use raw sources, not WebFetch.
  Fix wave result: 265 passages checked, 23 changed, 34 source documents fetched. Commits 9cc1419, e5df27a. 068c2d1 verified still an ancestor of HEAD; working tree clean; one log entry for the wave.
  New residual raised by the wave: hook dedup is per-settings-file and explicitly does NOT collapse a plugin's or skill's copy of the same handler. This IS recorded in the wiki at raw/notes/2026-08-04-claude-code-extension-model.md:116 as a marked post-review correction. Open question is whether it caps claude-code composition=3 under the floor read; claude-code.md:39's justification still cites "hook dedup/parallel-run". Controller to adjudicate after the scoped re-review.

SCOPED RE-REVIEW OF THE FIX WAVE (opus, 068c2d1..e5df27a): all 10 assigned findings ADDRESSED; all 3 rejections (I1, M2, M5) independently confirmed CORRECT against raw markdown — the earlier review's method was the problem, not the wiki; no score moved; 24/24 cells byte-identical; log append-only; 068c2d1 intact; docs/ untouched.

CONTROLLER ADJUDICATION (no second fix wave per the skill; residuals parked with rulings):
  R1. PARKED, real, one-sentence repair: raw/notes/2026-08-04-cursor-extension-model.md:79 now claims the page says "replaces"/"IDE allowlist" and NOT "overrides"/"in-app allowlist" — false. The page states the rule twice, in an intro paragraph and in a Precedence bullet, so the earlier draft was quoting the intro accurately. Same line also introduced a quotation missing its opening condition "When Run Mode is not admin-controlled and" with no ellipsis — the same dropped-qualifier class as C2, introduced by the fix meant to repair that class. Affects no score and nothing cites it. Recommend fixing before merge; surfacing to the human.
  R2. PARKED, citation repair not a score change: wiki/topics/claude-code.md:39 justifies composition=3 partly on "hook dedup/parallel-run", the one hook mechanism with a documented exception (a plugin's or skill's copy stays separate). The re-review found the warrant that actually holds — documented decision precedence deny > defer > ask > allow, continue:false overriding, additionalContext concatenated (code.claude.com/docs/en/hooks.md:1649, :848, :1477, :918). Score sound; warrant misattributed.
  R3. SURFACED TO HUMAN, affects a published score: the composition 3-vs-2 split between claude-code and codex-cli is not supported by either tool's docs — Codex also documents union execution plus deny-wins arbitration (learn.chatgpt.com/docs/hooks.md:713, :927). Codex was capped at 2 for "no arbitration", which is false. This is a re-score decision, not a quotation fix, and it would move the composition spread.
  R4. OUTSTANDING: /wiki:lint --local has never run. CONTROLLER CANNOT RUN IT EITHER — it is a slash command, not a Skill, so it is unavailable to the main loop as well as to subagents. The user must type it from the repo root. Master index correctly records "Last lint: never" and log.md discloses it.

RESIDUAL FIX (owner-authorized, overriding the no-second-wave rule): commit 9fd72f7.
  Took four attempts — two API 529s on opus, one stalled stream on sonnet. The third attempt completed all edits and staged them but died before committing; the controller inspected the full staged diff line by line, verified all four fixes correct and 0 score rows touched, then committed the pre-staged agent work directly rather than risk a fourth failure re-editing on top of it.
  R1 fixed: the false "the page says X not Y" claim retracted and both coexisting wordings quoted with their sections; the dropped condition "When Run Mode is not admin-controlled and" restored.
  R2 fixed: claude-code.md composition warrant now cites the documented decision-precedence rules instead of hook dedup. Score unchanged at 3.
  R3 recorded per the owner's decision: scoreboard's "Where the Rubric Strained" now carries a third open finding — the composition 3-vs-2 split rests on a reason Codex CLI's own docs contradict. No score changed.
  R4 fixed: quotation punctuation in three places.

STILL OPEN AT HANDOFF:
  - /wiki:lint --local has never run and cannot be run by any agent (slash command, not a Skill). The user must type it from the repo root. Master index says "Last lint: never"; log.md discloses it.
  - RECOMMENDED, not done: the method finding (WebFetch reports present phrases as absent; use raw markdown via <url>.md, or raw HTML for Cursor) currently lives only in this ledger, which the SDD process deletes. It belongs in .wiki/schema.md's Source Conventions, since anyone re-verifying this wiki's citations needs it. Workspace NOT deleted for this reason plus nothing being merged yet.
