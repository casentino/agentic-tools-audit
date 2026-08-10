# Task 5 Report: Profile Cursor

## Post-review fix pass (2026-08-04)

The reviewer independently re-fetched citations and confirmed the version via `plutil`; two findings came back.

### Finding 1 (Critical): misattributed "best-effort guardrails" quote

**What I found on re-verification.** I fetched `https://cursor.com/docs/agent/security` directly and confirmed the sentence sits there: "By default, terminal commands need your approval. To let trusted calls run without prompting, configure Run Modes. They range from a simple allowlist to the Auto-review classifier, and they're best-effort guardrails rather than a hard security boundary." I then fetched `https://cursor.com/docs/agent/security/run-modes` twice with targeted prompts asking specifically for "best-effort" / "guardrail" — both came back "not present on this page." This matches the reviewer's finding exactly.

While re-verifying, I also re-checked the adjacent "classifier can make mistakes" quote (which I had also attributed to the run-modes page) because an earlier probe of that page returned a false negative for it too. A follow-up direct fetch confirmed that quote genuinely does live on the run-modes page, under a section titled "Auto-review is not a security boundary" — so that citation was correct and untouched. This resolved my concern that the WebFetch tool's summarizing step was occasionally missing content that is actually present, rather than the note containing a second, undiscovered fabrication.

**What I changed:**
- `.wiki/raw/notes/2026-08-04-cursor-extension-model.md`: removed the "best-effort guardrails..." quote from the paragraph discussing the run-modes page and added an explicit note there that this framing lives on the security overview page instead; extended the existing `https://cursor.com/docs/agent/security` quote ("By default, terminal commands need your approval.") to include the contiguous "best-effort guardrails rather than a hard security boundary" sentence, in its real place; added a short "Re-attribution note (post-review fix)" recording the correction and how it was verified; updated both affected Sources bullets (run-modes bullet now states it does *not* contain the phrase; the agent/security bullet now claims it).
- `.wiki/wiki/topics/cursor.md`: the `side-effect-control` justification cell quoted the same phrase without a page-level citation, so there was no literal misattribution there, but the surrounding text sat next to "Run Modes," which could imply the wrong source. Changed "despite the docs' own..." to "despite the security overview docs' own..." to remove that implication.

### Finding 2 (Important): discoverability ceiling read vs. other axes' floor read

I chose to defend the asymmetry rather than flatten it, because on reflection the two families of axis genuinely ask different questions: `discoverability` asks whether an agent has *any* enforced way to learn an extension applies now (an existence question — one enforced path answers it), while `context-budget`/`composition`/`state`/`observability` ask how completely the tool's enforcement covers that axis's full scope (a coverage question — one confirmed, documented gap genuinely means "not fully enforced," regardless of what else is enforced elsewhere on that axis).

**What I changed:**
- `.wiki/wiki/topics/cursor.md`: added a sentence directly under the Rubric Scores table making this ceiling-vs-floor distinction and its justification explicit, so a second reader isn't left to infer it.
- `.wiki/raw/notes/2026-08-04-cursor-extension-model.md`: added a new paragraph in the "Scoping and Selection" section recording that `.wiki/schema.md` defines the four score levels but not how to aggregate a single axis score across several mechanisms with different enforcement levels — flagged explicitly as a rubric gap for the scoreboard task, not resolved silently.

### Commands run and actual output

Step 4 check block (brief's exact block, re-run from `.wiki/`):

```
(no output)
```

Coordinator's verification commands:

```
$ cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki
$ grep -n 'run-modes' raw/notes/2026-08-04-cursor-extension-model.md wiki/topics/cursor.md
raw/notes/2026-08-04-cursor-extension-model.md:68:**Run Modes** (https://cursor.com/docs/agent/security/run-modes) generalize approval across MCP tools, shell commands, and fetch calls. Three modes, quoted verbatim:
raw/notes/2026-08-04-cursor-extension-model.md:73:The classifier's own documented limitation, under a section literally titled "Auto-review is not a security boundary": "The classifier can make mistakes. It can allow a call you would have blocked, or block a call you would have allowed." (This is the only self-limitation quote confirmed on the run-modes page itself; the broader "best-effort guardrails" framing, quoted below, was verified to live on the security overview page instead, not here.)
raw/notes/2026-08-04-cursor-extension-model.md:81:**Other approval-gated surfaces** (https://cursor.com/docs/agent/security): ... "By default, terminal commands need your approval. To let trusted calls run without prompting, configure Run Modes. They range from a simple allowlist to the Auto-review classifier, and they're best-effort guardrails rather than a hard security boundary." **Re-attribution note (post-review fix):** this "best-effort guardrails rather than a hard security boundary" sentence was originally miscited in this note to the run-modes page; a direct fetch of `https://cursor.com/docs/agent/security` confirmed it verbatim in this exact paragraph, and a direct fetch of `https://cursor.com/docs/agent/security/run-modes` confirmed the phrase does not appear there. ...
raw/notes/2026-08-04-cursor-extension-model.md:91:- https://cursor.com/docs/agent/security/run-modes — ... Does NOT contain the "best-effort guardrails" sentence — confirmed absent by direct re-fetch.
raw/notes/2026-08-04-cursor-extension-model.md:93:- https://cursor.com/docs/agent/security — ... the "best-effort guardrails rather than a hard security boundary" framing (confirmed here, not on the run-modes page, by direct fetch), ...

$ grep -n 'best-effort guardrails' raw/notes/2026-08-04-cursor-extension-model.md wiki/topics/cursor.md
wiki/topics/cursor.md:41:| `side-effect-control` | 3 | ... despite the security overview docs' own "best-effort guardrails rather than a hard security boundary" caveat. |
raw/notes/2026-08-04-cursor-extension-model.md:73: ... (This is the only self-limitation quote confirmed on the run-modes page itself; the broader "best-effort guardrails" framing, quoted below, was verified to live on the security overview page instead, not here.)
raw/notes/2026-08-04-cursor-extension-model.md:81: ... "best-effort guardrails rather than a hard security boundary." **Re-attribution note (post-review fix):** ...
raw/notes/2026-08-04-cursor-extension-model.md:91:- https://cursor.com/docs/agent/security/run-modes — ... Does NOT contain the "best-effort guardrails" sentence — confirmed absent by direct re-fetch.
raw/notes/2026-08-04-cursor-extension-model.md:93:- https://cursor.com/docs/agent/security — ... the "best-effort guardrails rather than a hard security boundary" framing (confirmed here, not on the run-modes page, by direct fetch), ...

$ grep -n 'agent/security' raw/notes/2026-08-04-cursor-extension-model.md
68:**Run Modes** (https://cursor.com/docs/agent/security/run-modes) ...
81:**Other approval-gated surfaces** (https://cursor.com/docs/agent/security): ...
91:- https://cursor.com/docs/agent/security/run-modes — ...
93:- https://cursor.com/docs/agent/security — ...

$ grep -n 'aggregat' raw/notes/2026-08-04-cursor-extension-model.md wiki/topics/cursor.md
raw/notes/2026-08-04-cursor-extension-model.md:46:**Rubric-aggregation gap, recorded here as a finding, not resolved here.** `.wiki/schema.md` defines what each of the four score levels (`0`-`3`) means but says nothing about how to aggregate a single axis score when a tool exposes several mechanisms with different enforcement levels for that same axis — e.g. Cursor's four discoverability mechanisms split 3 enforced / 1 model-judged. Whether the axis score should reflect the strongest mechanism available (a ceiling read) or be capped by the weakest one still in play (a floor read) is left to the profile author's judgment; this note flags the gap for the scoreboard task rather than silently picking one convention.

$ grep -n '^| `discoverability` |' wiki/topics/cursor.md
37:| `discoverability` | 3 | Three of the four documented apply-mechanisms (`alwaysApply`, `globs`, manual `@`-mention) are mechanical, tool-computed checks rather than model judgment, with only the description-driven "Apply Intelligently" path left unverified. |
```

### Files changed in this fix pass

- Modified: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/raw/notes/2026-08-04-cursor-extension-model.md`
- Modified: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/wiki/topics/cursor.md`
- Commit: `64bb03e` — "fix: correct Cursor citation attribution and note rubric aggregation gap" (2 files changed, 9 insertions, 5 deletions, no trailer). Staged only these two files; `git diff --cached --check` printed nothing before commit.

### Scores (unchanged)

`discoverability=3 context-budget=2 composition=2 state=2 side-effect-control=3 observability=2`


## Version recorded

`3.14.7` — the installed application bundle version, read via:

```
plutil -p /Applications/Cursor.app/Contents/Info.plist | grep -i CFBundleShortVersionString
```

Output: `"CFBundleShortVersionString" => "3.14.7"`. Cursor.app is installed on this machine, so I used the installed-application-version path from the decision guidance rather than a documentation-stated version (the docs pages consulted carry no version number or "last updated" date at all — confirmed by a direct query that returned no matches). Recorded 2026-08-04.

## Documentation pages consulted

- `https://cursor.com/docs/context/rules` — the primary source. Established: the four rule storage shapes (Project `.mdc`, `AGENTS.md` incl. nested, User Rules, Team Rules) and their persistence/version-control characteristics; the exact three-field frontmatter (`description`, `globs`, `alwaysApply`) and a direct re-check confirming no fourth field exists; the four application mechanisms (Always Apply, Apply Intelligently, Apply to Specific Files, Apply Manually) with their exact trigger wording; the glob pattern examples; the "under 500 lines" guidance; the Team→Project→User cross-scope precedence quote; the nested-`AGENTS.md` parent/child precedence quote; a targeted re-query confirming the documentation is silent on same-scope (two Project Rules) glob-overlap arbitration; the `@filename.ts` deferred-content quote; and the manual troubleshooting checklist.
- `https://cursor.com/docs/context/mcp` — MCP config file locations/schema (`.cursor/mcp.json` project + global, stdio/SSE transports, interpolation variables), the default per-tool approval requirement, and the "Output panel → MCP Logs" observability surface with its documented contents and failure-marking behavior.
- `https://cursor.com/docs/agent/security/run-modes` — the three Run Modes (Auto-review, Allowlist, Run Everything) verbatim, the classifier's self-described fallibility, the "best-effort guardrails rather than a hard security boundary" framing, and sandbox mechanics (protected paths, default-deny network, macOS Seatbelt / Linux Landlock+seccomp).
- `https://cursor.com/docs/reference/permissions` — `permissions.json` locations, concatenation behavior across the two locations, its three fields (`mcpAllowlist`, `terminalAllowlist`, `autoRun`), the team-admin > permissions.json > IDE-settings precedence, the override-and-read-only-lock UI behavior, and the "only takes effect when a Run Mode is enabled" constraint.
- `https://cursor.com/docs/agent/security` — approval-by-default behavior for reads/edits/config-changes/terminal/MCP, default-deny network egress with named exceptions, workspace-trust default-off state, and a direct re-check confirming the page names no logging/diagnostics surface at all.
- `https://forum.cursor.com/t/0-51-memories-feature/98509` (forum, cited only to explain an exclusion) and `https://cursor.com/changelog` (checked, no Memories entry in the visible window) — used solely to document why a "Memories" state mechanism was **not** used as evidence: the docs URL that should cover it (`cursor.com/docs/context/memories`) resolves to the Rules page content (confirmed by checking its H1/opening paragraph), so no primary source for it was found.

## Six scores

| Axis | Score | One-line evidence |
|---|---|---|
| `discoverability` | 3 | 3 of 4 documented apply-mechanisms (`alwaysApply`, `globs`, `@`-mention) are tool-computed/deterministic; only description-driven "Apply Intelligently" is model judgment. |
| `context-budget` | 2 | Staged loading exists (four mechanisms gate what's resident) and a 500-line guideline is documented, but no enforced rule-count or token cap was found. |
| `composition` | 2 | Cross-scope (Team→Project→User) and nested-AGENTS.md precedence are documented; same-tier Project-Rule glob overlap has no documented arbitration — a confirmed gap. |
| `state` | 2 | Each rule tier has a defined, documented persistence location, but no enforced size cap or write-time integrity marker is documented for any of them. |
| `side-effect-control` | 3 | Approval-by-default + OS-level sandbox (Seatbelt/Landlock+seccomp) + classifier + a `permissions.json` allowlist that demonstrably overrides and locks the in-app UI — multiple enforced layers, despite the docs' own "best-effort" caveat. |
| `observability` | 2 | MCP has a concrete documented log surface (Output panel → "MCP Logs"); rules have only a manual troubleshooting checklist; general agent-security docs name no logging surface at all. |

## Axes scored 0 or 1

None scored 0 or 1. The lowest scores are 2 (`context-budget`, `composition`, `state`, `observability`), each because the documentation establishes a real, named mechanism or convention but stops short of describing an enforced cap, a same-scope arbitration rule, or an automated (non-manual) diagnostic surface. These gaps are called out explicitly in both the note and the profile rather than rounded up.

## Several rules matching one file

The documentation addresses two overlap cases explicitly:
1. **Cross-scope** (Team/Project/User all applicable at once): "Rules are applied in this order: Team Rules → Project Rules → User Rules. All applicable rules are merged; earlier sources take precedence when guidance conflicts."
2. **Nested `AGENTS.md`**: "Instructions from nested `AGENTS.md` files are combined with parent directories, with more specific instructions taking precedence."

It says **nothing** about the case the brief specifically flagged — two `.mdc` Project Rules, same scope, both matching the currently-edited file via `globs`. A targeted re-query of the rules page for exactly this scenario confirmed the absence directly (quoted in the note). The note and profile both label the resulting conclusion ("presumably concatenated with no order between them") as an **inference from absence**, not a recorded fact, per `.wiki/schema.md`'s source-conventions rule.

## Step 4 check block and output

Command run (verbatim from the brief, from `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki`):

```bash
f=wiki/topics/cursor.md
for k in title category sources created updated tags aliases confidence volatility verified summary; do
  grep -q "^$k:" "$f" || echo "MISSING frontmatter: $k"
done
for a in discoverability context-budget composition state side-effect-control observability; do
  grep -q "\`$a\`" "$f" || echo "MISSING axis: $a"
done
for s in "## Extension Model" "## Rubric Scores" "## Portability" "## Sources"; do
  grep -q "^$s" "$f" || echo "MISSING section: $s"
done
grep -qE '^\| `[a-z-]+` \| [0-3] \|' "$f" || echo "FAIL: no scored axis rows"
grep -q '\[0-3\]' "$f" && echo "FAIL: unfilled score placeholder remains"
test -f raw/notes/2026-08-04-cursor-extension-model.md || echo "MISSING evidence note"
```

**Actual output: none** (ran it twice, before and after a post-self-review fix pass; both runs produced zero output, matching "Expected: no output.").

## Files changed

- Created: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/raw/notes/2026-08-04-cursor-extension-model.md`
- Created: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/wiki/topics/cursor.md`
- Modified: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/raw/notes/_index.md` (added one Contents row)
- Modified: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/wiki/topics/_index.md` (added one Contents row)
- Modified: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/log.md` (appended one entry)
- `.wiki/_index.md` left untouched (confirmed via `git diff --stat` before commit: no diff).
- `.wiki/inventory/` was not created (confirmed absent).
- Commit: `58a14df` — "feat: profile Cursor against the audit rubric" (5 files changed, 147 insertions, no trailers).

## Self-review findings

- **Evidence-chain check (profile → note → source)**: every quote and figure used in `cursor.md` was traced back into the evidence note and each note claim carries its own cited URL. No profile quote was found that the note lacked.
- **Found and fixed during self-review, before committing:**
  1. A copy error in the note's glob-pattern list — I had written `docs/**/*.md, docs/**/*.md` (accidental duplicate) instead of the source's actual `docs/**/*.md, docs/**/*.mdx`. Fixed in the note prior to staging.
  2. Two quotes in the profile had drifted from their exact source wording during paraphrase: a Team-Rule quote using a bracketed substitution or the actual quoted text (`"cannot [disable it] in Customize"`), and the Run-Modes "best-effort guardrails" quote had `rather than` swapped for a comma+`not`. Both were rewritten to use the exact contiguous substring from the source quote (matching what's captured in the note).
  3. Re-ran the Step 4 check block after these fixes; still zero output.
- **Count-word check**: "three storage tiers... plus one file-based alternative" (note) and "Four storage shapes" (profile) both resolve to the same 3+1=4 total and are not contradictory; "four distinct application mechanisms" / "Three of these four are deterministic" all match their enumerated lists exactly (checked by recount).
- **Scoring independence check**: Cursor's scores (`3,2,2,2,3,2`) diverge from both existing profiles' patterns in a reasoned way — higher than Claude Code (2) and Codex CLI (2) on `discoverability` specifically because of the mechanical glob/alwaysApply/manual-mention triggers the brief flagged as the reason this tool is in the set, but at or below their `composition`/`context-budget`/`state` scores (Claude Code scored 3 on all three; Cursor scores 2 on all three) because the specific enforcement/arbitration details those axes need (same-scope rule arbitration, an enforced size/token cap, a write-time integrity marker) are documented as absent for Cursor, not merely unexamined.

## Concerns

- Cursor's "Memories" feature (mentioned only in community forum threads, e.g. a "Generate Memories" toggle under Settings → Rules) is plausibly relevant to the `state` axis but has no stable, independently-confirmable official documentation page as of this research — `cursor.com/docs/context/memories` resolves to the Rules page's own content, and the visible changelog window (2026-07-17 through 2026-08-03) has no Memories entry. I deliberately excluded it from the score's evidentiary basis rather than guess at its current behavior; this is flagged explicitly in both the note and as a labeled absence, not silently omitted.
- An untracked `CLAUDE.md` file exists at the repo root (not created by this task, not part of the brief's file list) and was left untouched and unstaged, per the instruction to stage only the brief's Step 6 paths.
