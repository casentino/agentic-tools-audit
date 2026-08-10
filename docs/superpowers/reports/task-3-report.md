# Task 3 Report: Profile Claude Code

## Fix report (post-review, 2026-08-04)

**Score change: `composition` moved from 2 to 3.** See finding 3 below — a directly documented, tool-enforced precedence order for same-named skills, missed in the original research pass, justifies the higher score. Updated six-score line: `disc=2 ctx=3 comp=3 state=3 side=3 obs=2`.

Three Important findings from independent review, addressed as follows:

**Finding 1 (evidence chain skip, `wiki/topics/claude-code.md:37`).** The profile's `discoverability` justification quoted "the description helps Claude decide when to load the skill automatically" from `code.claude.com/docs/en/skills`, but the evidence note never captured that verbatim sentence — only a paraphrase. Fix: added the verbatim quote to the note's Skills subsection, with source URL, so the profile → note → path chain holds. While doing this I found my first attempt at the fix still didn't satisfy a literal substring match, because I had wrapped "description" in backticks inside the quote (an artifact of how the WebFetch tool rendered the page as markdown, not part of the actual sentence). Removed the stray backticks in both the note and the profile so the quote is a clean, exact substring in both places.

**Finding 2 (misremembered local count, `raw/notes/...:26`).** The note said the `wiki-manager` skill's `references/` directory held 18 files. Recounted with `ls -1 ... | wc -l`: it's 17. Fixed the note's wording (and added the exact command inline as a self-check) and updated the note's cross-reference in the same sentence. This didn't feed any score, so no profile change was needed.

**Finding 3 (missed composition mechanism, `wiki/topics/claude-code.md` composition row).** I re-fetched `code.claude.com/docs/en/skills` and grepped the persisted page for precedence/conflict/namespacing language. Found and confirmed a directly on-topic, previously-uncited passage:

> "When skills share the same name across levels, enterprise overrides personal, and personal overrides project. A skill at any of these levels also overrides a bundled skill with the same name. For example, a `code-review` skill in your project's `.claude/skills/` replaces the bundled `/code-review`. Plugin skills use a `plugin-name:skill-name` namespace, so they cannot conflict with other levels. If you have files in `.claude/commands/`, those work the same way, but if a skill and a command share the same name, the skill takes precedence."

This is a fixed, deterministic, tool-enforced order (enterprise > personal > project > bundled) plus namespace isolation for plugin skills — squarely inside the rubric's "the tool enforces or verifies it" (score `3`), not just documented convention (`2`). I added this quote and its implications to the note's Skills subsection (new paragraph, no restructuring — same subsection, no new headings) and updated the note's Sources line for that URL. I then raised the profile's `composition` score from 2 to 3 and rewrote its justification sentence to cite this mechanism plus the previously-cited hook dedup/parallel-run and deny-always-wins permission merging.

I kept the existing inference label, narrowed to what it actually still covers: the precedence rule above only resolves *same-named* collisions. It says nothing about two *differently-named* extensions (e.g. two subagents, or two skills with different names) whose descriptions both plausibly match one prompt — no source, local or documented, describes a tool-enforced tie-break for that case. The profile's composition row and the note both now state this narrower residual gap explicitly as "an inference from absence, not a confirmed one," per `.wiki/schema.md`'s inference-labeling rule. I left `Portability` unedited since it already describes this narrower gap accurately and editing it wasn't one of the three findings.

### Commands run and actual output

```
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki
f=wiki/topics/claude-code.md
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
grep -q 'Version examined: \[' "$f" && echo "FAIL: version not recorded"
test -f raw/notes/2026-08-04-claude-code-extension-model.md || echo "MISSING evidence note"
```
Output: **none** (pass), re-run after every edit including the backtick fix.

```
ls -1 ~/.claude/plugins/cache/llm-wiki/wiki/0.16.0/skills/wiki-manager/references/ | wc -l
```
Output: `17`

```
grep -n '18 files\|17 files' raw/notes/2026-08-04-claude-code-extension-model.md
```
Output:
```
31:- `~/.claude/plugins/cache/llm-wiki/wiki/0.16.0/skills/wiki-manager/SKILL.md` — a long `description` frontmatter field (dozens of trigger phrases) that drives ambient activation; the body defers detail to 17 files under its own `references/` directory (`ls -1 references/ | wc -l` → 17; e.g. `references/ingestion.md`, `references/linting.md`) rather than inlining everything in `SKILL.md` itself. Each workflow section in `SKILL.md` is one line plus a link, e.g. `### Ingestion\nSee [references/ingestion.md](references/ingestion.md).`
```

```
grep -n 'description helps Claude decide' raw/notes/2026-08-04-claude-code-extension-model.md wiki/topics/claude-code.md
```
Output:
```
wiki/topics/claude-code.md:37:| `discoverability` | 2 | Skill/subagent applicability is a documented convention — the tool keeps description text in context and the model judges relevance (code.claude.com/docs/en/skills: "the description helps Claude decide when to load the skill automatically") — but nothing verifies the model picked the *right* one; hooks are the exception, matching mechanically on event+matcher. |
raw/notes/2026-08-04-claude-code-extension-model.md:33:Trigger: the tool keeps skill descriptions in context at session start; Claude compares the current task against them and loads the full body only when it decides a skill applies, or the user explicitly types `/name`. Per official docs (https://code.claude.com/docs/en/skills): "Every skill needs a SKILL.md file with two parts: YAML frontmatter between --- markers that tells Claude when to use the skill, and markdown content with the instructions Claude follows when the skill runs. The directory name becomes the command you type, and the description helps Claude decide when to load the skill automatically."
```

```
grep -n '^| `composition` |' wiki/topics/claude-code.md
```
Output:
```
39:| `composition` | 3 | Same-named skills resolve via a documented, tool-enforced order — "enterprise overrides personal, and personal overrides project... also overrides a bundled skill with the same name" — and plugin skills are namespaced (`plugin-name:skill-name`) so they cannot collide at all; combined with hook dedup/parallel-run and deny-always-wins permission merging, the tool enforces composition wherever the collision surface is well-defined (same name, same event, same permission target). The narrower residual gap, still an inference from absence rather than a confirmed one: no source describes a tool-enforced tie-break when two *differently-named* extensions (e.g. two subagents) both plausibly match the same prompt — that case is left to model judgment. |
```

### Files changed (this fix pass)

- `.wiki/raw/notes/2026-08-04-claude-code-extension-model.md` — fixed reference-file count (18→17), added the verbatim discoverability quote, added the same-name precedence quote and its analysis, updated the relevant Sources line.
- `.wiki/wiki/topics/claude-code.md` — removed stray backticks from the discoverability quote, raised `composition` from 2 to 3 with a rewritten justification, kept the inference label but narrowed its scope.

Staged and committed only these two files (`git add .wiki/raw/notes/2026-08-04-claude-code-extension-model.md .wiki/wiki/topics/claude-code.md`); `git diff --cached --check` printed nothing before commit. Commit: `d4f5294` "fix: correct evidence chain, file count, and composition score in Claude Code profile" — no attribution trailer, Conventional Commit subject, new commit rather than an amend.

```
2.1.221 (Claude Code)
```

Produced by running `claude --version` on this machine on 2026-08-04. Recorded verbatim in both the evidence note and the profile's "Version examined" line.

## Sources consulted

**Local (Read/Glob/Grep/Bash only, no subagents dispatched):**
- `~/.claude/plugins/cache/*/*/*/skills/*/SKILL.md` (87 files globbed) — skill frontmatter shape and volume.
- `~/.claude/plugins/cache/llm-wiki/wiki/0.16.0/skills/wiki-manager/{SKILL.md,references/*.md}` — progressive-disclosure pattern (short SKILL.md, 18 deferred reference files).
- `~/.claude/plugins/cache/casentino/{history,metrics,wf}/*/.claude-plugin/plugin.json` — plugin manifest schema, hook declarations.
- `~/.claude/plugins/marketplaces/{casentino,team-attention-plugins}/.claude-plugin/marketplace.json` — marketplace manifest schema.
- `~/.claude/settings.json`, `~/.claude/settings.local.json` — live hooks, `permissions.defaultMode`, `allow` rules, `enabledPlugins`, statusline.
- `~/.claude/agents/{plan-validator,prompt-analyzer,prompt-scorer}.md` — subagent frontmatter fields.
- `~/.claude.json` (`mcpServers` key, secrets redacted) — global MCP server registration.
- `~/.claude/CLAUDE.md` (and its `@RTK.md` import), `~/.claude/rules/*.md`, project `CLAUDE.md` — memory-file scopes and import mechanism.
- `~/.claude/projects/` directory listing — transcript storage naming convention.

**Remote (WebFetch, official docs only):**
- `code.claude.com/docs/en/memory` — CLAUDE.md scope/load-order table, `@import` syntax + 4-hop depth limit, auto-memory size enforcement.
- `code.claude.com/docs/en/skills` — SKILL.md frontmatter fields, description-driven auto-invocation, progressive disclosure, compaction budget for invoked skills.
- `code.claude.com/docs/en/hooks` — parallel hook execution, cross-scope merge/dedup, plugin-hook merging.
- `code.claude.com/docs/en/permission-modes` — full permission-mode table, protected-path rules, auto-mode classifier behavior.
- `code.claude.com/docs/en/permissions`, `code.claude.com/docs/en/settings` — tiered approval system, settings-scope precedence order (preview-only due to page size; not fully re-read).

I did not dispatch the `claude-code-guide` agent (the orchestrator's constraint #5 overrode the brief's suggestion to prefer it) — all remote evidence came from direct WebFetch of official docs.

## The six scores

| Axis | Score | Evidence in one line |
|------|-------|------------------------|
| `discoverability` | 2 | Skill/subagent relevance is judged by the model reading `description` text kept in context (documented convention); hooks are the mechanical exception. |
| `context-budget` | 3 | Tool enforces auto-memory's 200-line/25KB cap with an error-triggered rewrite, and stages skill bodies (description-only until invoked, capped re-attachment after compaction). |
| `composition` | 2 | Hook/permission merging across scopes is documented and deterministic (dedup, parallel, deny-wins); skill/subagent candidate arbitration has no found tool-enforced tie-break (flagged as an inference from absence). |
| `state` | 3 | Auto memory is tool-managed: defined location, enforced size limit, and a `modified` timestamp Claude Code itself stamps on write. |
| `side-effect-control` | 3 | Mode-based permission system + deny-always-wins rule engine + hard-coded protected-path list checked ahead of allow rules + auto-mode classifier blocking named categories by default. |
| `observability` | 2 | Built-in transcripts (`~/.claude/projects/.../*.jsonl`) and introspection commands (`/context`, `/status`, `/permissions`, `/doctor`) exist, but no dedicated hook-failure log was found beyond exit code/stderr. |

## Axes where evidence was thin, and how I handled it

- **Composition**: strong evidence for hooks/permission-rule merge (documented, deterministic), but no source — local or remote — states how the tool arbitrates two *skills* or *subagents* whose descriptions both plausibly match a prompt. I did not invent a mechanism; the profile explicitly labels this as "an inference from absence, not a confirmed gap" and the score (2, not 3) reflects that the enforced part of composition is real but partial.
- **Hook event list**: local plugin manifests and doc quotes directly confirm seven events (`SessionStart`, `SessionEnd`, `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `Stop`, `PreCompact`). An initial broader WebFetch summary listed 20+ additional event names (e.g. `TeammateIdle`, `WorktreeCreate`, `Elicitation`). I did not re-verify each with a direct quote, so the evidence note explicitly marks that longer list as "likely but unconfirmed by direct quote" rather than presenting it as settled fact, and none of the six scores rely on it.
- **Settings precedence exact order**: the `permissions`/`settings` doc page was too large for a full re-read (71-88KB persisted outputs); I used the WebFetch model's structured summary (with one direct quote: "if your user settings set `spinnerTipsEnabled` to `true`... the project value applies") rather than the raw page, and cross-checked it against the more detailed `permission-modes` fetch, which agreed on the mode table and protected-path behavior. This affects the `side-effect-control` justification, which I kept to claims corroborated by the more thoroughly quoted `permission-modes` page.

## Check block run and output

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki
f=wiki/topics/claude-code.md
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
grep -q 'Version examined: \[' "$f" && echo "FAIL: version not recorded"
test -f raw/notes/2026-08-04-claude-code-extension-model.md || echo "MISSING evidence note"
```

**Actual output: none** (matches the brief's "Expected: no output"). Re-ran after two post-write precision edits (see below) and still produced no output.

## Files changed

- `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/raw/notes/2026-08-04-claude-code-extension-model.md` (new)
- `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/wiki/topics/claude-code.md` (new)
- `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/raw/notes/_index.md` (added one Contents row; `Last updated` was already 2026-08-04, no bump needed)
- `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/wiki/topics/_index.md` (added one Contents row; `Last updated` already current)
- `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/log.md` (appended one entry, nothing reordered)

Not staged/touched: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/CLAUDE.md` — this file exists untracked in the working tree but predates this task and is outside the brief's Step 6 file list; I left it alone per the "stage only what Step 6 lists" instruction.

## Self-review

Walked all six axes against the evidence note:
- **discoverability** → traces to evidence note "Skills" section (description-in-context, model decides) and the doc quote cited there.
- **context-budget** → traces to "Memory files" and "Loading and Injection" sections (200-line/25KB cap + enforced rewrite error; skill description-vs-body staging; 25,000-token compaction budget).
- **composition** → traces to "Hooks" and "Permission and Hook Surface" sections (parallel/dedup/merge quotes); the unenforced-skill-arbitration half is explicitly flagged in the profile as an inference from absence, per `.wiki/schema.md`'s requirement.
- **state** → traces to "Memory files" section (auto-memory location, size enforcement, `modified` frontmatter stamping quote).
- **side-effect-control** → traces to "Permission and Hook Surface" section (mode table, protected paths, deny-always-wins, auto-mode classifier).
- **observability** → traces to "Observability" section (transcript path convention, slash commands, absence of a dedicated hook-failure log, third-party `metrics` plugin built on the primitives).

While re-reading the committed profile, I caught and fixed two precision issues before finalizing: a malformed doc-URL citation (backtick placement made it read as a code span rather than a domain) and an unlabeled inference in the composition row (fixed to explicitly call out that the "no tool-enforced tie-break for overlapping skills" claim is an inference from absence, not a directly documented fact). Both fixes are separate small commits rather than amendments, per the standing git-safety rule to never amend without explicit request.

## Concerns

- The full canonical hook-event list (beyond the seven confirmed by direct quote + local usage) was not independently re-verified; the evidence note flags this explicitly and no score depends on it.
- Two of the settings/permissions doc pages exceeded the WebFetch inline-output size and were only available as model-summarized previews or grep'd excerpts of the persisted cache file; I treated these as slightly lower-confidence than the directly-quoted `permission-modes` page and scoped `side-effect-control`'s justification to claims the latter corroborates.
- Three commits exist for this task instead of one (main commit `616c452` + two follow-up precision fixes `e2e0ad9`, `835e8ff`) because I caught issues on self-review after the initial commit and chose new commits over amending. If a single commit is required, these three would need to be squashed.
