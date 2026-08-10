# Final Fix Wave — Quotation Fidelity Audit

Branch `feat/agentic-tools-wiki`. Base `068c2d1`. Commits added: `9cc1419`, `e5df27a`.
No score changed; all twenty-four scoreboard cells verified byte-identical to `068c2d1`.

## Method note first, because it changed three outcomes

The reviewer warned that WebFetch renders pages through a summarizing model, so "phrase
absent" is strong evidence but not proof. That warning was correct and load-bearing.

Three of the four documentation sites in this wiki are Mintlify-hosted and serve a **raw
markdown** version of any page by appending `.md` to the URL:

- `docs.langchain.com/oss/python/langgraph/<page>.md` → 200 `text/markdown`
- `code.claude.com/docs/en/<page>.md` → 200 `text/markdown`
- `learn.chatgpt.com/docs/<page>.md` → 200 `text/markdown`
- `cursor.com/docs/<page>.md` → **404**; for Cursor the raw HTML RSC payload was used instead

Every finding below was settled against raw markdown or the raw HTML payload, retrieved with
`curl`, and matched with `grep -F` after normalising case and punctuation. Where a claim rests
on a raw-source check rather than a rendered fetch, that is stated.

This mattered: **my own first WebFetch of the LangGraph `add-memory` page reproduced the
disputed list as four bullets with "Custom strategies" presented separately — exactly the
reviewer's reading. The raw markdown shows five bullets in one list.** Two independent
summarizer reads agreeing with each other is not evidence about the page.

---

## Critical

### C1 — Cursor `globs` trigger phrase. CONFIRMED, fixed.

**Fetched:** `https://cursor.com/docs/context/rules` via WebFetch, then the raw HTML payload
via `curl` (209 KB). Searched every text node with `grep -F`.

**Result:** the quoted phrase `when file paths match patterns in globs` is **absent** —
zero hits in the raw payload. The page's real text, both confirmed present:

- Rule-type table, `Apply to Specific Files` row → **"When file matches a specified pattern"**
- A second table introduced by "Under the hood, the three frontmatter fields interact to
  determine when a rule is included:", row `alwaysApply: false` / `description: —` /
  `globs: provided` → **"Auto-attached when a matching file is in context."**

All eight example glob patterns the note lists (`*`, `**`, `*.ts`, `**/*.ts`, `src/**`,
`src/**/*.tsx`, `docs/**/*.md, docs/**/*.mdx`, `tailwind.config.*`) **are** on the page —
that part of the note was accurate.

**Changed** in three places, in each case substituting the two real trigger sentences and
moving "a mechanical, tool-computed path match" outside the quotation marks as the author's
characterization:

- `.wiki/raw/notes/2026-08-04-cursor-extension-model.md` — mechanism (3), plus a dated
  correction paragraph recording that the phrase is not on the page and that no score depends
  on the difference.
- `.wiki/wiki/concepts/path-scoped-activation.md` — Technique section and Sightings entry.

### C2 — Codex resident-skill-list budget stitch. CONFIRMED, fixed.

**Fetched:** `https://learn.chatgpt.com/docs/build-skills.md` (raw markdown, 11 KB).

**Result:** the wiki's single quotation is not on the page. The real paragraph reads, as four
consecutive sentences:

> In Codex, the initial list also includes each skill's file path. To avoid crowding out the
> rest of the prompt, this list uses at most 2% of the model's context window, or 8,000
> characters when the context window is unknown. If many skills are installed, Codex shortens
> skill descriptions first. For large skill sets, Codex may omit some skills from the initial
> list and show a warning.

The stitch dropped **"when the context window is unknown"**, converting a conditional fallback
into a second ceiling. The reviewer's reading is exactly right.

**Changed** in four places — the two sentences now quoted separately, with the third
(overflow/warning) sentence added since it was already implied by the note's gloss:

- `.wiki/raw/notes/2026-08-04-codex-cli-extension-model.md` — quote plus a dated correction
  paragraph stating explicitly that `context-budget = 3` is unaffected.
- `.wiki/wiki/topics/codex-cli.md:25` — the **unquoted paraphrase** ("~2%-of-context /
  8,000-character budget") corrected to name 8,000 as the unknown-context-window fallback.
- `.wiki/wiki/topics/codex-cli.md` `context-budget` row — quotation corrected, **score left
  at 3**, which the real sentence still supports as a tool-enforced numeric cap.
- `.wiki/wiki/concepts/deferred-reference-loading.md` — Codex sighting.

---

## Important

### I1 — LangGraph default reducer. **REVIEWER WRONG.** Left alone.

**Fetched:** `https://docs.langchain.com/oss/python/langgraph/graph-api.md` (raw markdown).

The page contains **both** sentences, in different places:

- line 218: "Custom reducers combine the left and right arguments. The [default
  reducer](#default-reducer) **discards the left argument and keeps only the right.**"
- line 222: "The default reducer **ignores the left argument and replaces the state value with
  the right argument.** This example shows how to use the default reducer:"

The wiki quotes the line-218 wording. It is **verbatim** (modulo the stripped inline link and a
lower-cased leading "the" where it is folded into the note's sentence). The reviewer appears to
have found line 222 and concluded the note's rendering was a paraphrase of it.

No change made to `.wiki/raw/notes/2026-08-04-langgraph-extension-model.md:30` or
`.wiki/wiki/topics/langgraph.md:23`.

### I2 — Claude Code hook parallel/dedup. CONFIRMED, fixed — and the real text is materially narrower.

**Fetched:** `https://code.claude.com/docs/en/hooks.md` (raw markdown, 251 KB).

Line 403 reads, as three sentences:

> All matching hooks run in parallel. If you define the same handler in more than one settings
> file, it runs once. A plugin's or skill's copy of the same handler stays separate.

The wiki's "All matching hooks run in parallel, and identical handlers are deduplicated
automatically" is not on the page, and the stitch **over-generalised the rule**: deduplication
applies to the same handler declared in more than one *settings file*, and explicitly does not
collapse a plugin's or skill's copy, which stays separate and runs on its own.

**Changed:**

- `.wiki/raw/notes/2026-08-04-claude-code-extension-model.md:116` — all three sentences quoted
  separately, with a dated correction naming the over-generalisation.
- `.wiki/wiki/topics/claude-code.md` — two **unquoted** paraphrases carried the same error and
  were corrected: the opening blockquote ("merged and deduplicated across scopes") and the
  plugins paragraph ("deduplicated by command string"). Both now state that same-handler
  duplicates collapse *across settings files* while plugin/skill copies stay separate.
  `composition = 3` untouched — it rests on same-name skill precedence and deny-always-wins
  permission merging, not on this nuance. See Concerns.

### I3 — Score-gap attribution. CONFIRMED, fixed.

The card's own Technique and Sightings sections document that Claude Code has the identical
`paths` glob field on both skills and rules, so the mechanism cannot explain a gap between
Cursor and Claude Code. **Changed** both places to attribute the gap to how the axis was
aggregated, and cross-referenced the scoreboard's open finding:

- `.wiki/wiki/concepts/path-scoped-activation.md:17` — now states the gap is "a product of how
  the axis was aggregated ... not of the mechanism being absent from one tool", and links the
  `## Where the Rubric Strained` section, quoting its own sentence about `paths` being "the same
  class of mechanical match this scoreboard credits for Cursor's `discoverability` = 3".
- `.wiki/inventory/candidates/add-path-scoped-rule-activation.md:43` — rationale reworded; adds
  that the case for the change "does not rest on the score gap at all: it rests on the mechanism
  being present, documented, and unused."

### I4 — Rule-file counts. CONFIRMED against the live directory, fixed.

**Verified locally:** `ls ~/.claude/rules/*.md` → 8 files. `grep -l "^paths:"` → none.
Reading each header: exactly five are TypeScript/JavaScript-specific (`coding-style.md`,
`hooks.md`, `patterns.md`, `security.md`, `testing.md`, each titled "TypeScript/JavaScript …");
`agents.md`, `git-workflow.md`, `performance.md` are stack-neutral.

**Changed** `.wiki/wiki/concepts/path-scoped-activation.md:21`: eight files carry no `paths`;
five of them are TS/JS-specific and are now listed by name; the three stack-neutral files are
named as legitimately global. The "all scoped to TypeScript/JavaScript" claim is gone.

While there, a **misattributed quotation** in the same sentence was fixed: the card attributed
"are loaded unconditionally at every session start alongside `CLAUDE.md`" to Claude Code's
documentation. That phrasing is the *evidence note's*, not the docs'. The real sentence, in
`code.claude.com/docs/en/memory.md`, is "Rules without a `paths` field are loaded
unconditionally and apply to all files." Same fix applied to the candidate file, which carried
the shorter variant "are loaded unconditionally at every session start."

### I5 — Missing source. CONFIRMED, fixed.

Both Claude Code quotes the card's Problem and Technique sections rest on are verbatim
(confirmed against `code.claude.com/docs/en/memory.md`) and are recorded in the Claude Code
note's composition-gap paragraph. **Changed** `.wiki/wiki/concepts/specificity-ordered-precedence.md`:
added `raw/notes/2026-08-04-claude-code-extension-model.md` to the `sources:` frontmatter and to
`## Sources`, with the entry stating explicitly that this is a third *listed source*, not a third
sighting — Claude Code appears in the card as the tool that lacks the technique. The card's
two-sighting qualification is unchanged.

### I6 — CLAUDE.md vs AGENTS.md. CONFIRMED, fixed.

- `AGENTS.md:9`: "Measurement summaries under `.wiki/output/projects/*/data/` are the record and
  belong in version control; anything a re-ingest can reproduce stays ignored." `CLAUDE.md:18`
  said "**Never commit** generated output, dependency directories, or local audit results" —
  directly contradicting it.
- `AGENTS.md:7` grants an exemption for code serving the wiki
  (`.wiki/output/projects/<slug>/code/` with tests alongside). `CLAUDE.md:17` omitted it.

**Changed** both bullets to match the narrowed `AGENTS.md` text.

---

## Minor

### M1 — Count. CONFIRMED, fixed.
`.wiki/raw/notes/2026-08-04-codex-cli-extension-model.md:20` said "Four distinct kinds" and lists
five (items **1.**–**5.**). One word: "Four" → "Five". The `codex-cli.md` profile already said
"five" in both its blockquote and Unit line, so no downstream change was needed.

### M2 — LangGraph `Send`. **REVIEWER WRONG.** Left alone.

**Fetched:** `graph-api.md` raw markdown. Both quoted passages are verbatim **and contiguous**:

- Source line 618, one sentence: "The number of objects may be unknown ahead of time (meaning
  the number of edges may not be known) and the input `State` to the downstream `Node` should be
  different (one for each generated object)." The "and" here joins two clauses **inside a single
  source sentence** — which is what the sweep pattern looks for, but the sentence is real.
- Source line 620, one paragraph: "To support this design pattern, LangGraph supports returning
  [`Send`](…) objects from conditional edges. `Send` takes two arguments: first is the name of
  the node, and second is the state to pass to that node."

The note renders both faithfully. No change.

### M3 — README. Fixed.
Added `.wiki/raw/` as the **first** layout row ("The evidence layer — one hand-written note per
tool under `raw/notes/`, the source every article cites"), added `.wiki/wiki/theses/` (marked
"created, none written yet" — the directory holds only its index), and added a sentence stating
`.wiki/` is a local wiki registered in the hub at
`/Users/kikyeongoh/Documents/opterk/llm-wiki` under `local_wikis`. Also added one line making
the traceability premise explicit, which is why `raw/` leads the table.

### M4 — "Three mechanical checks". CONFIRMED, fixed.
The evidence note (`cursor…model.md`, Scoping and Selection) is careful: "(1) always-apply and
(3) glob match are deterministic, tool-executed checks … (4) manual mention is an explicit user
action." **Changed** three places to match:

- `.wiki/wiki/topics/cursor.md:27` — two are tool-executed; `@`-mention is deterministic but
  user-initiated. Also replaced the truncated quote "check the rule type... ensure the file
  pattern matches" with the page's full verbatim sentence.
- `.wiki/wiki/topics/cursor.md` `discoverability` row — same correction, **score left at 3**,
  with the ceiling read named explicitly ("the tool-computed `globs` path sets the score").
- `.wiki/wiki/references/rubric-scoreboard.md:34` — same correction.

### M5 — LangGraph context-management list count. **REVIEWER WRONG.** Left alone.

**Fetched:** `add-memory.md` raw markdown. Lines 1196–1204:

```
1196  … long conversations can exceed the LLM's context window. Common solutions are:
1198  * [Trim messages](#trim-messages): Remove first or last N messages (before calling LLM)
1199  * [Delete messages](#delete-messages) from LangGraph state permanently
1200  * [Summarize messages](#summarize-messages): Summarize earlier messages …
1201  * [Manage checkpoints](#manage-checkpoints) to store and retrieve message history
1202  * Custom strategies (e.g., message filtering, etc.)
1204  This allows the agent to keep track of the conversation without exceeding …
```

Five bullets, one contiguous list, no blank line before line 1202. "Custom strategies" **is**
the fifth bullet. The note is correct and its earlier post-review correction was the right call.
My own WebFetch of the same page reported four bullets with the fifth "outside and separate" —
a summarizer artifact, and the strongest single piece of evidence in this wave for reading raw
sources.

---

## The sweep

**Passages checked: 265.** Every quoted string of 30+ characters in the four evidence notes and
four pattern cards, extracted mechanically and matched against downloaded sources with `grep -F`
after case/punctuation normalisation. Quotations in the four topic profiles and the scoreboard
were checked against the same corpora.

**Passages changed: 23.**

Of the 265, 114 did not match on the first pass. Each was triaged by hand; the large majority
were not defects: frontmatter titles/summaries/aliases, `sources:` paths, local `--help` output
and local file content (legitimately cited to the installed binaries and the filesystem, not to
the web), the authors' own scare-quotes, quotations carrying deliberate `...` elisions whose
halves both verify, the one sentence the LangGraph note itself flags as invented, and — the
largest group — false negatives caused by Cursor's HTML fragmenting prose across inline `<code>`
nodes, which broke whole-sentence matching and required fragment-level checks.

Beyond the ten named findings, the sweep surfaced **nine further non-verbatim quotations**, all
fixed:

| Location | Quoted as | Actually on the page |
|---|---|---|
| cursor note, `permissions.json` | "when both exist, their arrays are concatenated rather than one replacing the other." | Two consecutive sentences: "When both exist, Cursor concatenates the arrays inside every field." / "Per-user and per-repo entries combine; one does not replace the other." |
| cursor note, `permissions.json` | "…it overrides the corresponding in-app allowlist in Cursor Settings" | "When `permissions.json` defines a key, that key's value **replaces** the corresponding IDE allowlist entirely." |
| cursor note, `permissions.json` | "the in-app allowlist editor becomes read-only." | "The in-app editor for that allowlist becomes read-only and the 'Add to allowlist' button is hidden." (non-adjacent to the sentence above; now marked as such) |
| cursor note, MCP admin | "empty allowlist permits all tools" | "Leave a tool allowlist empty to allow all tools from that server." |
| codex note, skills layout | `SKILL.md` (required, "Contains instructions and metadata") | File-tree annotation "Required: instructions + metadata"; all five annotations now quoted, plus the page's plain-prose definition |
| codex note, hooks use cases | four gerund phrases ("scanning your team's prompts…") | Five imperative bullets ("Scan your team's prompts to block accidentally pasting API keys", …) |
| codex note, AGENTS.md | "Codex follows a hierarchical search pattern across three scopes" | Not on the page. Real: "Codex builds an instruction chain when it starts … Discovery follows this precedence order:" — and the third numbered item is **Merge order**, not a third scope |
| codex note, config layering | "follow a precedence order, with CLI flags taking highest priority." | "Codex resolves values in this order (highest precedence first):" + a six-item list beginning "CLI flags and `--config` overrides" |
| codex note, `/status` | "displays current session state and agent details" | "Display session configuration and token usage." |

Two further fidelity repairs of the same kind: the `untrusted` approval-mode quote silently
dropped a parenthetical mid-sentence (now quoted in full, including
"(for example, destructive Git operations or Git output/config-override flags)"), and the
`project_doc_max_bytes` quote truncated before "(32 KiB by default)". The Claude Code note's
transcript-protection quote had "and" rewritten to "because"; the full source sentence is now
quoted.

**Source documents fetched: 34.** LangGraph 11 (`graph-api`, `add-memory`, `checkpointers`,
`persistence`, `stores`, `interrupts`, `streaming`, `observability`, `use-subgraphs`,
`errors/INVALID_CONCURRENT_GRAPH_UPDATE`, `errors.py`); Codex 14 (`build-skills`, `agents-md`,
`hooks`, `agent-configuration/rules`, `config-reference`, `config-basic`,
`environment-variables`, `cli/reference`, `customization/memories`, `plugins`,
`developer-commands`, `sandboxing`, `agent-approvals-security`, repo `docs/config.md`); Cursor 5
(`context/rules`, `context/mcp`, `agent/security`, `agent/security/run-modes`,
`reference/permissions`); Claude Code 4 (`hooks`, `memory`, `skills`, `permission-modes`).

---

## Files changed

Commit `9cc1419` — wiki quotation and framing fixes:

- `.wiki/raw/notes/2026-08-04-cursor-extension-model.md`
- `.wiki/raw/notes/2026-08-04-codex-cli-extension-model.md`
- `.wiki/raw/notes/2026-08-04-claude-code-extension-model.md`
- `.wiki/wiki/concepts/path-scoped-activation.md`
- `.wiki/wiki/concepts/deferred-reference-loading.md`
- `.wiki/wiki/concepts/specificity-ordered-precedence.md`
- `.wiki/wiki/topics/cursor.md`
- `.wiki/wiki/topics/codex-cli.md`
- `.wiki/wiki/topics/claude-code.md`
- `.wiki/wiki/references/rubric-scoreboard.md`
- `.wiki/inventory/candidates/add-path-scoped-rule-activation.md`
- `.wiki/log.md` (one appended entry; no existing entry touched)

Commit `e5df27a` — repository documentation:

- `CLAUDE.md`
- `README.md`

Untouched, as instructed: `docs/superpowers/`, `.wiki/schema.md`, all four score tables,
`.wiki/raw/notes/2026-08-04-langgraph-extension-model.md` (all three findings against it were
rejected), `.wiki/wiki/topics/langgraph.md`, `.wiki/wiki/concepts/tiered-persistence-split.md`.
All date fields remain `2026-08-04`; correction paragraphs are dated 2026-08-05 in prose only,
matching the existing in-note convention for post-review verification.

---

## Concerns

1. **Three of thirteen reviewer findings were wrong, all in the same direction** — a summarizing
   fetch reporting present text as absent or as a paraphrase. Any future verification pass on
   this wiki should use the `.md` raw endpoints (or the raw HTML payload for Cursor) as the
   primary evidence and treat WebFetch output as a lead, not a verdict. Consider recording this
   in `.wiki/schema.md` under Source Conventions; I did not add it there, since the schema is
   human-owned and changing evidence rules is not a quotation fix.

2. **The Cursor note's `permissions.json` paragraph produced four of the nine sweep defects.**
   That page is not version-stamped and carries no "last updated" date (the note says so
   itself), so for two of them — "overrides … in-app allowlist in Cursor Settings" vs "replaces
   … IDE allowlist entirely" — I cannot distinguish a misquotation from page drift between
   2026-08-04 and 2026-08-05. I recorded both the current verbatim text and the earlier draft's
   wording rather than silently overwriting, so a future reader can tell what changed. The
   `state`/`side-effect-control` scores do not depend on the wording either way.

3. **Correcting I2 narrowed a mechanism the Claude Code profile leans on.** Hook dedup is
   per-settings-file and explicitly does *not* collapse plugin or skill copies. I corrected the
   prose in three places and left `composition = 3`, which is justified primarily by same-name
   skill precedence and deny-always-wins permission merging. But `claude-code.md`'s Portability
   section still credits "deterministic hook merging" as something a plugin author need not
   invent, and the `composition` justification still lists "hook dedup/parallel-run" among the
   enforced collision surfaces. Both remain defensible; a re-score pass should decide whether
   the plugin-copy exception is a confirmed gap of the kind that caps other axes at 2. I did not
   touch the score, per instructions.

4. **`.wiki/log.md` history note.** My appended entry initially undercounted the changed
   passages (14 rather than 23). Because the entry was already committed, I unwound my own two
   commits with `git reset --soft 068c2d1` and rebuilt them with the corrected figure rather
   than editing a committed log entry in a follow-up commit. `068c2d1` was never amended and is
   intact at `068c2d1307c3ee598c12415824b1181bf894aa8e`. The final history contains exactly one
   log entry for this wave.

5. **Lint is still outstanding** (pre-existing, recorded in the log at `068c2d1`).
   `/wiki:lint --local` was not invocable; the checks here were quotation verification, not a
   structural lint. The master index still correctly records "Last lint: never".

6. **Not re-verified in this wave:** the Claude Code note's local-filesystem claims other than
   `~/.claude/rules/` (the 87-skill glob, plugin manifests, `~/.claude.json` MCP entries) and
   the two version strings read from installed binaries. Out of scope here; unchanged.

7. `git diff --cached --check` printed nothing before each commit, and the working tree is clean.
