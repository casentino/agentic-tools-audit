# Agentic Tools Audit Wiki Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stand up an llm-wiki local wiki in this repository that scores four agentic tools against a six-axis rubric and converts the recurring techniques into portable pattern cards.

**Architecture:** The wiki lives at `.wiki/` following llm-wiki 0.16.0 conventions. Tool profiles go in `wiki/topics/`, pattern cards in `wiki/concepts/`, the comparison table in `wiki/references/`, and plugin backlog items in `inventory/candidates/`. This plan covers milestones 1 through 3 of the spec — structure, profiles, patterns. The measurement pipeline (milestone 4) gets its own spec once the axes stop moving.

**Tech Stack:** Markdown with YAML frontmatter, llm-wiki 0.16.0 conventions, `bash` for structural verification. **No Python in this plan.** The spec ties the Python toolchain to the measurement pipeline, so `pyproject.toml`, `ruff`, and `pytest` arrive with milestone 4, not here.

**Spec:** `docs/superpowers/specs/2026-08-04-agentic-tools-audit-wiki-design.md`

## Global Constraints

Every task's requirements implicitly include this section.

- Wiki root: `.wiki/` in this repository. Hub: `/Users/kikyeongoh/Documents/opterk/llm-wiki`.
- **`.wiki/` must be tracked by git.** This repository owns the wiki history. See Task 1.
- Rubric has exactly six axes: `discoverability`, `context-budget`, `composition`, `state`, `side-effect-control`, `observability`. Scores are `0`, `1`, `2`, or `3`.
- Score meanings: `0` no such mechanism, `1` implicit convention, `2` documented convention, `3` the tool enforces or verifies it.
- Tool profiles: `wiki/topics/<tool>.md`, `category: topic`, `volatility: hot`.
- Pattern cards: `wiki/concepts/<pattern>.md`, `category: concept`, `volatility: warm`.
- **A pattern card requires sightings in at least two places.** One sighting stays in its profile.
- The master `_index.md` lists collections, never individual raw files.
- Official documentation and source code carry the argument; blog posts support it.
- Every profile records the version examined, or the commit SHA when the tool publishes no version.
- Mark inferences as inferences. Never present one as a recorded fact.
- The owner's plugin sources stay in their own repositories. Reference paths; never copy into `raw/`.
- Cross-references between wiki articles use both link forms on one line: `[[slug|Display]] ([Display](../category/slug.md))`.
- Inside `.wiki/`, llm-wiki conventions govern. `AGENTS.md`'s `src/` and `tests/` rules apply only outside it.
- Markdown uses two-space indentation, UTF-8, and a final newline.
- Commit subjects follow Conventional Commits. Attribution trailers are disabled globally.
- Today's date for all `created:`/`updated:`/`ingested:` fields in this plan: `2026-08-04`. Use the actual current date when executing later.

## File Structure

| Path | Responsibility |
|------|----------------|
| `README.md` | What this repository is, and how to reach the wiki |
| `AGENTS.md` | Repository conventions, amended to exempt `.wiki/` |
| `CLAUDE.md` | Agent guidance, updated once the wiki exists |
| `.gitignore` | Ignores re-derivable material only. Must not ignore `.wiki/`. |
| `.wiki/config.md` | Wiki title, scope, conventions |
| `.wiki/schema.md` | Rubric axes, entity types, relationship verbs, evidence rules |
| `.wiki/_index.md` | Statistics, navigation, recent changes |
| `.wiki/log.md` | Append-only activity log |
| `.wiki/wiki/topics/<tool>.md` | One profile per tool — extension model and scores |
| `.wiki/wiki/concepts/<pattern>.md` | One card per portable technique |
| `.wiki/wiki/references/rubric-scoreboard.md` | Cross-tool score table |
| `.wiki/inventory/candidates/<slug>.md` | One plugin backlog item per card |
| `<hub>/wikis.json` | Registers this wiki under `local_wikis` |

### Seed tool selection

This plan profiles four tools. The spec allows three to five and names none, so this plan chooses for axis diversity:

| Tool | Slug | Why this one |
|------|------|--------------|
| Claude Code | `claude-code` | The target platform. Establishes the baseline every pattern must port back to. |
| OpenAI Codex CLI | `codex-cli` | A CLI with a deliberately thin extension surface. Contrast for `discoverability` and `composition`. |
| Cursor | `cursor` | IDE-hosted, glob-scoped rules. A different answer to `discoverability` than description matching. |
| LangGraph | `langgraph` | A framework where state and interrupts are first-class. The strongest available contrast on `state`. |

Swap any of these before execution if a better contrast appears. Four tools give enough overlap for the two-sighting rule while keeping the plan executable.

---

### Task 1: Create the wiki structure and keep it tracked

**Files:**
- Create: `.wiki/` tree (13 directories, 13 `_index.md` files, `.obsidian/app.json`, `.obsidian/appearance.json`, `inbox/.gitkeep`)
- Create: `.gitignore`
- Modify: `/Users/kikyeongoh/Documents/opterk/llm-wiki/wikis.json`

**Interfaces:**
- Consumes: nothing. This is the first task.
- Produces: the directory tree every later task writes into, and the tracked-git guarantee those tasks depend on.

**Why this task builds the tree by hand.** `/wiki init agentic-tools-audit --local` produces the same tree, but its documented step 2 ends with *"For local wikis (`--local`): append `.wiki/` to the project's `.gitignore`."* That default treats a local wiki as project-private scratch space. Here the wiki **is** the deliverable, and the spec commits it. Building the tree explicitly avoids creating a rule this task would immediately have to undo.

- [ ] **Step 1: Create the directory tree**

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
mkdir -p .wiki/inbox/.processed
mkdir -p .wiki/raw/{articles,papers,repos,notes,data}
mkdir -p .wiki/wiki/{concepts,topics,references,theses}
mkdir -p .wiki/output
mkdir -p .wiki/.obsidian
touch .wiki/inbox/.gitkeep
```

Do **not** create `.wiki/inventory/` or `.wiki/datasets/` here. llm-wiki creates those layers lazily, and a half-populated `inventory/` trips lint check C1. Task 8 creates `inventory/candidates/` when it has a record to put there.

- [ ] **Step 2: Verify the tree matches the convention**

```bash
for d in inbox inbox/.processed raw raw/articles raw/papers raw/repos raw/notes \
         raw/data wiki wiki/concepts wiki/topics wiki/references wiki/theses output .obsidian; do
  test -d ".wiki/$d" || echo "MISSING dir: $d"
done
test -d .wiki/inventory && echo "UNEXPECTED: inventory/ must not exist yet"
test -d .wiki/datasets  && echo "UNEXPECTED: datasets/ must not exist yet"
```

Expected: no output.

- [ ] **Step 3: Write the Obsidian vault config**

Write `.wiki/.obsidian/app.json`:

```json
{
  "showFrontmatter": true,
  "alwaysUpdateLinks": true,
  "newLinkFormat": "relative",
  "useMarkdownLinks": false
}
```

Write `.wiki/.obsidian/appearance.json`:

```json
{
  "accentColor": ""
}
```

- [ ] **Step 4: Generate an `_index.md` for every wiki-managed directory**

Every existing wiki-managed directory carries an `_index.md`. `inbox/` is the exception — it holds `.gitkeep` instead.

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki
write_index() {
  local dir="$1" name="$2" desc="$3"
  cat > "$dir/_index.md" <<EOF
# $name Index

> $desc

Last updated: 2026-08-04

## Contents

| File | Summary | Tags | Updated |
|------|---------|------|---------|

## Recent Changes

- 2026-08-04: Directory created.
EOF
}

write_index raw             "Raw Sources"      "Immutable source material ingested into this wiki."
write_index raw/articles    "Articles"         "Ingested documentation pages and written sources."
write_index raw/papers      "Papers"           "Ingested papers and formal specifications."
write_index raw/repos       "Repositories"     "Collection manifests for ingested code repositories."
write_index raw/notes       "Notes"            "Hand-written source notes."
write_index raw/data        "Data"             "Ingested structured data sources."
write_index wiki            "Compiled Wiki"    "Articles compiled from raw sources."
write_index wiki/concepts   "Concepts"         "Portable techniques extracted from two or more tools."
write_index wiki/topics     "Topics"           "One profile per agentic tool."
write_index wiki/references "References"       "Cross-tool comparison tables and indexes."
write_index wiki/theses     "Theses"           "Thesis investigations."
write_index output          "Outputs"          "Generated artifacts and project folders."
```

- [ ] **Step 5: Write the master `_index.md`**

Write `.wiki/_index.md`:

```markdown
# Agentic Tools Audit Index

> Extension mechanisms, rubric scores, and portable patterns across agentic tools.

Last updated: 2026-08-04

## Statistics

- Sources: 0 raw documents
- Articles: 0 compiled wiki articles
- Outputs: 0 generated artifacts
- Last compiled: never
- Last lint: never

## Quick Navigation

- [Topic Guide](schema.md)
- [Inbox](inbox/)
- [All Sources](raw/_index.md)
- [Concepts](wiki/concepts/_index.md)
- [Topics](wiki/topics/_index.md)
- [References](wiki/references/_index.md)
- [Outputs](output/_index.md)

## Recent Changes

- 2026-08-04: Initialized the wiki.
```

Omit `Inventory` and `Datasets` from Quick Navigation. The convention adds those lines only once the layers exist.

- [ ] **Step 6: Write `log.md`**

Write `.wiki/log.md`:

```markdown
# Wiki Activity Log

## [2026-08-04] init | Wiki initialized as a local wiki in agentic-tools-audit
```

This file is append-only. Never edit or delete an existing entry, and always append rather than rewriting the whole file.

- [ ] **Step 7: Write `.gitignore` — the tracked-git guarantee**

This repository has no `.gitignore` yet. Create it with narrow ignores only:

```gitignore
# Re-derivable measurement inputs. Re-ingesting restores them.
.wiki/output/projects/*/data/raw/

# macOS
.DS_Store

# Python (arrives with the measurement pipeline)
__pycache__/
.pytest_cache/
.ruff_cache/
```

**`.wiki/` itself must never appear in this file.** If a later `/wiki init --local` run adds it, delete that line.

- [ ] **Step 8: Verify git tracks the wiki**

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
grep -q '^\.wiki/$' .gitignore && echo "FAIL: .wiki/ is ignored"
git check-ignore -q .wiki/_index.md && echo "FAIL: master index is ignored"
git status --short .wiki | head -5
```

Expected: no `FAIL` lines, and `git status` lists `.wiki` as untracked.

- [ ] **Step 9: Register the wiki in the hub**

Read `/Users/kikyeongoh/Documents/opterk/llm-wiki/wikis.json`, then add one entry to the `local_wikis` array. Local wikis take absolute paths — the portable `<HUB>`-relative form applies only to hub-owned topics.

```json
{
  "path": "/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki",
  "description": "Agentic tool extension mechanisms, rubric scores, and portable patterns."
}
```

Leave the `wikis` map and `default` untouched. This wiki is not a hub topic.

- [ ] **Step 10: Verify the registration parses**

```bash
python3 -c "
import json
p='/Users/kikyeongoh/Documents/opterk/llm-wiki/wikis.json'
d=json.load(open(p))
hits=[w for w in d['local_wikis'] if 'agentic-tools-audit' in w['path']]
print('OK' if len(hits)==1 else f'FAIL: {len(hits)} entries')
print('topics untouched:', list(d['wikis'].keys()))
"
```

Expected: `OK`, and the topics list still shows only `hub` and `experience-knowledge`.

- [ ] **Step 11: Append the hub log entry**

Append one line to `/Users/kikyeongoh/Documents/opterk/llm-wiki/log.md`:

```markdown
## [2026-08-04] init | Registered local wiki agentic-tools-audit (/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki)
```

- [ ] **Step 12: Commit both repositories**

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
git add .gitignore .wiki
git diff --cached --check
git commit -m "feat: initialize agentic tools audit local wiki"

cd /Users/kikyeongoh/Documents/opterk/llm-wiki
git add wikis.json log.md
git commit -m "chore: register agentic-tools-audit local wiki"
```

`git diff --cached --check` is the whitespace gate `AGENTS.md` prescribes. It must print nothing.

---

### Task 2: Write `config.md` and `schema.md`

**Files:**
- Create: `.wiki/config.md`
- Create: `.wiki/schema.md`

**Interfaces:**
- Consumes: the tree from Task 1.
- Produces: the six axis names and score meanings that every profile in Tasks 3–6 cites, the entity types and relationship verbs those profiles use, and the evidence rules Task 8 enforces when promoting patterns.

**Why these two files carry the rubric.** `schema.md` is llm-wiki's human-owned topic guide: local vocabulary that keeps agents from drifting in taxonomy. It must not redefine global primitives such as raw folder names, article categories, or required frontmatter. The rubric is exactly the kind of local vocabulary it exists to hold.

- [ ] **Step 1: Write `config.md`**

```markdown
---
title: "Agentic Tools Audit"
description: "Extension mechanisms, rubric scores, and portable patterns across agentic tools."
created: 2026-08-04
freshness_threshold: 70
---

# Wiki Configuration

## Scope

This wiki covers agentic CLIs and any tool exposing an extension mechanism, including IDE extensions and agent frameworks. It exists to inform the owner's Claude Code plugin development. Records earn their place by answering a plugin design question.

Live task experiments, self-measurement of the owner's plugins, and adoption recommendations stay out of scope.

## Conventions

- Score every profile against all six axes in [schema.md](schema.md).
- Record the version examined, or the commit SHA when the tool publishes no version.
- Promote a technique to a pattern card only after sighting it in two or more places.
- Reference the owner's plugin sources by path. Never copy them into `raw/`.
- List collections in the master index, never individual raw files.
```

- [ ] **Step 2: Write `schema.md`**

```markdown
---
title: "Agentic Tools Audit Topic Guide"
schema_state: advisory
created: 2026-08-04
updated: 2026-08-04
summary: "Human-owned rubric, vocabulary, and evidence rules for comparing agentic tool extension mechanisms."
---

# Agentic Tools Audit Topic Guide

> This guide adds local vocabulary. It redefines no global llm-wiki primitive — raw source folders, article categories, and required frontmatter keep their standard meanings.

## State

- `schema_state`: `advisory`

## Rubric Axes

Each axis states a plugin design question. Every tool profile scores all six.

| Axis | Question |
|------|----------|
| `discoverability` | How does the agent learn that this extension applies now? |
| `context-budget` | How much context does an extension consume, and how does the tool defer or stage it? |
| `composition` | How does the tool order and reconcile overlapping extensions? |
| `state` | Where does state that outlives a session live? |
| `side-effect-control` | How does the tool gate permissions and isolate effects? |
| `observability` | What logs, metrics, and failure diagnostics exist? |

## Score Scale

| Score | Meaning |
|-------|---------|
| `0` | No such mechanism. |
| `1` | Implicit. The convention exists only in practice. |
| `2` | Documented convention. |
| `3` | The tool enforces or verifies it. |

## Entity Types

| Type | Meaning |
|------|---------|
| `tool` | An agentic tool or extension host. |
| `extension-unit` | Whatever the tool treats as an extension: skill, rule, agent, hook, node. |
| `mechanism` | The machinery that discovers, loads, or runs an extension unit. |
| `pattern` | A portable technique sighted in two or more places. |
| `axis` | A rubric evaluation axis. |
| `candidate` | A proposed change to the owner's plugins. |

## Relationship Verbs

Added to the global set:

- `evaluates` — a profile against an axis
- `implements` — a tool realizing a pattern
- `informs` — a pattern producing a candidate
- `measured-by` — a profile against measurement data

## Source Conventions

- Official documentation and source code carry the argument. Blog posts and social media support it.
- Record the version examined in every profile. When a tool publishes no version, record the commit SHA.
- A pattern card requires sightings in at least two places. A single sighting stays in its profile.
- Reference the owner's plugin sources by path. Never copy them into `raw/`.
- Mark inferences as inferences. Never present one as a recorded fact.

## Article Boundaries

- `wiki/topics/` holds one profile per tool. Profiles describe extension models; they do not teach techniques.
- `wiki/concepts/` holds pattern cards. Cards teach techniques; they do not introduce tools.
- `wiki/references/` holds the rubric scoreboard and any cross-tool index.
- Every factual claim traces to a source listed in the article's Sources section.
```

- [ ] **Step 3: Verify both files**

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki
for k in title description created freshness_threshold; do
  grep -q "^$k:" config.md || echo "config.md MISSING: $k"
done
for k in title schema_state created updated summary; do
  grep -q "^$k:" schema.md || echo "schema.md MISSING: $k"
done
for a in discoverability context-budget composition state side-effect-control observability; do
  grep -q "\`$a\`" schema.md || echo "schema.md MISSING axis: $a"
done
grep -c '^| `[0-3]` |' schema.md
```

Expected: no `MISSING` lines, and the final count prints `4` for the four score rows.

- [ ] **Step 4: Append the log entry**

Append to `.wiki/log.md`:

```markdown
## [2026-08-04] schema | Adopted the topic guide with six rubric axes and a 0-3 score scale
```

- [ ] **Step 5: Commit**

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
git add .wiki/config.md .wiki/schema.md .wiki/log.md
git diff --cached --check
git commit -m "feat: define audit rubric and topic vocabulary"
```

---

## How the four profile tasks work

Tasks 3 through 6 share one shape. Each produces two files:

1. **An evidence note** in `raw/notes/`. Hand-written, immutable, holding exact quotes, file paths, URLs, and the version or commit SHA examined. This is the provenance chain — the profile cites it, and freshness scoring reads its `ingested:` date.
2. **A profile** in `wiki/topics/`. Compiled from the note, scoring all six axes.

**The scores are not in this plan.** A plan that pre-filled them would be fabricating research. Each task states where to look and what question each axis asks; the executing engineer reads the sources and scores from what they find. A score with no justification sentence is an incomplete task.

**Two-file order matters.** Write the note first. If the note cannot support a score, the answer is more research, not a guess in the profile.

---

### Task 3: Profile Claude Code

**Files:**
- Create: `.wiki/raw/notes/2026-08-04-claude-code-extension-model.md`
- Create: `.wiki/wiki/topics/claude-code.md`
- Modify: `.wiki/raw/notes/_index.md`, `.wiki/wiki/topics/_index.md`, `.wiki/log.md`

**Interfaces:**
- Consumes: the six axes and score scale from `.wiki/schema.md` (Task 2).
- Produces: `wiki/topics/claude-code.md` — the baseline profile. Task 7 reads its six scores into the scoreboard. Task 8 reads its extension-model section when hunting for second sightings, and every pattern card's porting section targets this tool.

**Why this tool goes first.** Claude Code is where every pattern must eventually land. Scoring it first gives the other three profiles a reference point, and it is the one tool whose evidence sits on this machine rather than behind a docs site.

- [ ] **Step 1: Gather evidence**

Record the version first:

```bash
claude --version
```

Local evidence, all readable now:

- Installed skills: `~/.claude/plugins/cache/*/*/*/skills/*/SKILL.md` — read the `name` and `description` frontmatter to see what drives skill selection.
- A skill that stages its own context: `~/.claude/plugins/cache/llm-wiki/wiki/0.16.0/skills/wiki-manager/` — note how `SKILL.md` defers detail to `references/*.md`.
- Hook and permission configuration: `~/.claude/settings.json`.
- Subagent definitions: `~/.claude/agents/`.
- Plugin manifests: `~/.claude/plugins/marketplaces/*/`.

Official documentation covers the parts local files do not explain — the loading order, the permission model, and the hook event list. The `claude-code-guide` agent answers these directly if the executing session has agent access; otherwise fetch the official Claude Code documentation.

**Constraint reminder:** the owner's own plugins (`history`, `metrics`, `wf`, `session-wrap`) are evidence *about Claude Code's mechanism*, but their source must not be copied into `raw/`. Quote a line or two and cite the path.

- [ ] **Step 2: Write the evidence note**

Create `.wiki/raw/notes/2026-08-04-claude-code-extension-model.md`:

```markdown
---
title: "Claude Code Extension Model"
source: "MANUAL"
type: notes
ingested: 2026-08-04
tags: [claude-code, extension-unit, mechanism]
summary: "Evidence on Claude Code's extension units, their loading path, and its permission and hook surface. Collected from the local installation and official documentation."
---

# Claude Code Extension Model

## Version Examined

[Output of `claude --version`, verbatim.]

## Extension Units

[One subsection per unit: skills, plugins, hooks, subagents, MCP servers, memory files. For each, record what declares it, where it lives, and what triggers it. Cite exact paths.]

## Loading and Injection

[When each unit enters context. Quote the documentation.]

## Permission and Hook Surface

[Permission modes, tool allowlists, hook events. Cite `~/.claude/settings.json` keys and the documented event names.]

## Observability

[Logs, transcripts, and diagnostics available when an extension misfires.]

## Sources

- [Exact local paths and documentation URLs, one per line, each with what it established.]
```

Every bracketed line above is a slot to fill from the sources, not text to keep.

- [ ] **Step 3: Write the profile**

Create `.wiki/wiki/topics/claude-code.md`:

```markdown
---
title: "Claude Code"
category: topic
sources: ["raw/notes/2026-08-04-claude-code-extension-model.md"]
created: 2026-08-04
updated: 2026-08-04
tags: [claude-code, agentic-cli, extension-unit]
aliases: ["Claude Code CLI"]
confidence: high
volatility: hot
verified: 2026-08-04
summary: "Claude Code's extension model and its scores across the six audit axes."
---

# Claude Code

> [One paragraph: what Claude Code treats as an extension, and the single most distinctive thing about how it loads one.]

## Extension Model

**Unit:** [what the tool treats as an extension]
**Loading:** [how a unit reaches the agent]
**Injection point:** [when it enters context]

[Two or three paragraphs of detail. Describe the mechanism; do not teach techniques — that belongs to pattern cards.]

## Rubric Scores

Version examined: [version string]

| Axis | Score | Justification |
|------|-------|---------------|
| `discoverability` | [0-3] | [one sentence] |
| `context-budget` | [0-3] | [one sentence] |
| `composition` | [0-3] | [one sentence] |
| `state` | [0-3] | [one sentence] |
| `side-effect-control` | [0-3] | [one sentence] |
| `observability` | [0-3] | [one sentence] |

## Portability

[Three lines or fewer. This tool is the porting target, so record what its mechanism already handles well and where a plugin author has to work around it.]

## Sources

- [Claude Code Extension Model](../../raw/notes/2026-08-04-claude-code-extension-model.md) — extension units, loading order, permission surface
```

- [ ] **Step 4: Verify the profile**

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

Expected: no output.

- [ ] **Step 5: Update the two directory indexes**

Add one row to the `## Contents` table in `.wiki/raw/notes/_index.md`:

```markdown
| [2026-08-04-claude-code-extension-model.md](2026-08-04-claude-code-extension-model.md) | Evidence on Claude Code's extension units, loading, and permission surface. | claude-code, extension-unit | 2026-08-04 |
```

Add one row to the `## Contents` table in `.wiki/wiki/topics/_index.md`:

```markdown
| [claude-code.md](claude-code.md) | Claude Code's extension model and six-axis scores. | claude-code, agentic-cli | 2026-08-04 |
```

Bump `Last updated:` in both files to today.

- [ ] **Step 6: Append the log entry and commit**

Append to `.wiki/log.md`:

```markdown
## [2026-08-04] compile | 1 source → claude-code profile scored on six axes
```

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
git add .wiki/raw/notes .wiki/wiki/topics .wiki/log.md
git diff --cached --check
git commit -m "feat: profile Claude Code against the audit rubric"
```

---

### Task 4: Profile OpenAI Codex CLI

**Files:**
- Create: `.wiki/raw/notes/2026-08-04-codex-cli-extension-model.md`
- Create: `.wiki/wiki/topics/codex-cli.md`
- Modify: `.wiki/raw/notes/_index.md`, `.wiki/wiki/topics/_index.md`, `.wiki/log.md`

**Interfaces:**
- Consumes: the six axes and score scale from `.wiki/schema.md` (Task 2).
- Produces: `wiki/topics/codex-cli.md`. Task 7 reads its six scores. Task 8 treats its techniques as candidate second sightings against Claude Code.

**Why this tool.** Codex CLI keeps its extension surface deliberately thin, which makes it the useful low end of the scale. A rubric where every tool scores 2 or 3 measures nothing.

- [ ] **Step 1: Gather evidence**

Primary source: the `openai/codex` repository on GitHub. Read its documentation directory and configuration reference rather than the marketing page.

Record the version. If the CLI is installed locally:

```bash
codex --version
```

If it is not installed, record the release tag examined, or the commit SHA of the default branch when no tag applies. The constraint is explicit: a tool with no published version gets a commit SHA.

Look specifically for:

- The instruction-file convention (`AGENTS.md`) — what discovers it, and how nested files combine.
- The configuration file and what it can declare.
- MCP server support, if present.
- Approval and sandbox modes — this is where Codex CLI is likely to be strongest, so read the actual mode list rather than summarizing.
- Any persistence of state between runs.

- [ ] **Step 2: Write the evidence note**

Create `.wiki/raw/notes/2026-08-04-codex-cli-extension-model.md`:

```markdown
---
title: "OpenAI Codex CLI Extension Model"
source: "https://github.com/openai/codex"
type: notes
ingested: 2026-08-04
tags: [codex-cli, extension-unit, mechanism]
summary: "Evidence on Codex CLI's instruction files, configuration surface, and approval modes. Collected from the repository documentation."
---

# OpenAI Codex CLI Extension Model

## Version Examined

[Version string, release tag, or commit SHA — with which one it is.]

## Extension Units

[What can be added or configured, where it lives, and what discovers it. Quote the documentation.]

## Loading and Injection

[How instruction files combine, and in what order.]

## Approval and Sandbox Modes

[The documented mode list, verbatim.]

## State and Observability

[What persists between runs, and what logs exist.]

## Sources

- [URLs with the exact path inside the repository, each with what it established.]
```

- [ ] **Step 3: Write the profile**

Create `.wiki/wiki/topics/codex-cli.md`:

```markdown
---
title: "OpenAI Codex CLI"
category: topic
sources: ["raw/notes/2026-08-04-codex-cli-extension-model.md"]
created: 2026-08-04
updated: 2026-08-04
tags: [codex-cli, agentic-cli, extension-unit]
aliases: ["Codex CLI"]
confidence: high
volatility: hot
verified: 2026-08-04
summary: "Codex CLI's extension model and its scores across the six audit axes."
---

# OpenAI Codex CLI

> [One paragraph: what Codex CLI treats as an extension, and the most distinctive thing about it.]

## Extension Model

**Unit:** [what the tool treats as an extension]
**Loading:** [how a unit reaches the agent]
**Injection point:** [when it enters context]

[Two or three paragraphs. Where the surface is genuinely absent, say so plainly — an absent mechanism is a finding, not a gap in the research.]

## Rubric Scores

Version examined: [version, tag, or SHA]

| Axis | Score | Justification |
|------|-------|---------------|
| `discoverability` | [0-3] | [one sentence] |
| `context-budget` | [0-3] | [one sentence] |
| `composition` | [0-3] | [one sentence] |
| `state` | [0-3] | [one sentence] |
| `side-effect-control` | [0-3] | [one sentence] |
| `observability` | [0-3] | [one sentence] |

## Portability

[Three lines or fewer: what a Claude Code plugin author could take from this tool.]

## Sources

- [OpenAI Codex CLI Extension Model](../../raw/notes/2026-08-04-codex-cli-extension-model.md) — instruction files, configuration, approval modes
```

- [ ] **Step 4: Verify the profile**

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki
f=wiki/topics/codex-cli.md
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
test -f raw/notes/2026-08-04-codex-cli-extension-model.md || echo "MISSING evidence note"
```

Expected: no output.

- [ ] **Step 5: Update the two directory indexes**

Add to `.wiki/raw/notes/_index.md`:

```markdown
| [2026-08-04-codex-cli-extension-model.md](2026-08-04-codex-cli-extension-model.md) | Evidence on Codex CLI's instruction files, configuration, and approval modes. | codex-cli, extension-unit | 2026-08-04 |
```

Add to `.wiki/wiki/topics/_index.md`:

```markdown
| [codex-cli.md](codex-cli.md) | Codex CLI's extension model and six-axis scores. | codex-cli, agentic-cli | 2026-08-04 |
```

Bump `Last updated:` in both.

- [ ] **Step 6: Append the log entry and commit**

Append to `.wiki/log.md`:

```markdown
## [2026-08-04] compile | 1 source → codex-cli profile scored on six axes
```

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
git add .wiki/raw/notes .wiki/wiki/topics .wiki/log.md
git diff --cached --check
git commit -m "feat: profile OpenAI Codex CLI against the audit rubric"
```

---

### Task 5: Profile Cursor

**Files:**
- Create: `.wiki/raw/notes/2026-08-04-cursor-extension-model.md`
- Create: `.wiki/wiki/topics/cursor.md`
- Modify: `.wiki/raw/notes/_index.md`, `.wiki/wiki/topics/_index.md`, `.wiki/log.md`

**Interfaces:**
- Consumes: the six axes and score scale from `.wiki/schema.md` (Task 2).
- Produces: `wiki/topics/cursor.md`. Task 7 reads its six scores. Task 8 pairs its rule-scoping mechanism against Claude Code's description matching when looking for `discoverability` patterns.

**Why this tool.** Cursor answers `discoverability` with file globs and explicit apply flags rather than description matching. That is a genuinely different mechanism for the same problem, which is what makes it worth a profile.

- [ ] **Step 1: Gather evidence**

Primary source: the official Cursor documentation at `docs.cursor.com`, specifically its rules reference. The `context7` MCP server also resolves Cursor documentation if the executing session has it available; prefer whichever gives the current version.

Record the application version. Cursor publishes no CLI version flag for this purpose, so take the version from the application's About panel and record it as the version examined.

Look specifically for:

- The rule file format and location, including its frontmatter fields.
- How a rule declares when it applies — glob patterns, always-apply flags, description-driven selection, manual mention.
- What happens when several rules match the same file.
- Whether rules can defer content, or must inline everything they need.
- MCP server support and its permission model.

- [ ] **Step 2: Write the evidence note**

Create `.wiki/raw/notes/2026-08-04-cursor-extension-model.md`:

```markdown
---
title: "Cursor Extension Model"
source: "https://docs.cursor.com"
type: notes
ingested: 2026-08-04
tags: [cursor, extension-unit, mechanism]
summary: "Evidence on Cursor's rule files, their scoping fields, and its MCP surface. Collected from the official documentation."
---

# Cursor Extension Model

## Version Examined

[Application version string, and where it was read from.]

## Rule Files

[Format, location, and frontmatter fields. Quote the field list.]

## Scoping and Selection

[How a rule declares when it applies. Record every mechanism the documentation names, not just the first.]

## Overlap Resolution

[What the documentation says happens when several rules match. If it says nothing, record that as an absence and mark the conclusion an inference.]

## Deferred Content and MCP

[Whether rules can reference other files, and what the MCP surface allows.]

## Sources

- [Documentation URLs, each with what it established.]
```

- [ ] **Step 3: Write the profile**

Create `.wiki/wiki/topics/cursor.md`:

```markdown
---
title: "Cursor"
category: topic
sources: ["raw/notes/2026-08-04-cursor-extension-model.md"]
created: 2026-08-04
updated: 2026-08-04
tags: [cursor, ide-agent, extension-unit]
aliases: ["Cursor IDE"]
confidence: high
volatility: hot
verified: 2026-08-04
summary: "Cursor's rule-based extension model and its scores across the six audit axes."
---

# Cursor

> [One paragraph: what Cursor treats as an extension, and how its scoping differs from description matching.]

## Extension Model

**Unit:** [what the tool treats as an extension]
**Loading:** [how a unit reaches the agent]
**Injection point:** [when it enters context]

[Two or three paragraphs. Give the scoping fields their exact names.]

## Rubric Scores

Version examined: [version string]

| Axis | Score | Justification |
|------|-------|---------------|
| `discoverability` | [0-3] | [one sentence] |
| `context-budget` | [0-3] | [one sentence] |
| `composition` | [0-3] | [one sentence] |
| `state` | [0-3] | [one sentence] |
| `side-effect-control` | [0-3] | [one sentence] |
| `observability` | [0-3] | [one sentence] |

## Portability

[Three lines or fewer: what a Claude Code plugin author could take from this tool.]

## Sources

- [Cursor Extension Model](../../raw/notes/2026-08-04-cursor-extension-model.md) — rule format, scoping fields, MCP surface
```

- [ ] **Step 4: Verify the profile**

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki
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

Expected: no output.

- [ ] **Step 5: Update the two directory indexes**

Add to `.wiki/raw/notes/_index.md`:

```markdown
| [2026-08-04-cursor-extension-model.md](2026-08-04-cursor-extension-model.md) | Evidence on Cursor's rule files, scoping fields, and MCP surface. | cursor, extension-unit | 2026-08-04 |
```

Add to `.wiki/wiki/topics/_index.md`:

```markdown
| [cursor.md](cursor.md) | Cursor's rule-based extension model and six-axis scores. | cursor, ide-agent | 2026-08-04 |
```

Bump `Last updated:` in both.

- [ ] **Step 6: Append the log entry and commit**

Append to `.wiki/log.md`:

```markdown
## [2026-08-04] compile | 1 source → cursor profile scored on six axes
```

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
git add .wiki/raw/notes .wiki/wiki/topics .wiki/log.md
git diff --cached --check
git commit -m "feat: profile Cursor against the audit rubric"
```

---

### Task 6: Profile LangGraph

**Files:**
- Create: `.wiki/raw/notes/2026-08-04-langgraph-extension-model.md`
- Create: `.wiki/wiki/topics/langgraph.md`
- Modify: `.wiki/raw/notes/_index.md`, `.wiki/wiki/topics/_index.md`, `.wiki/log.md`

**Interfaces:**
- Consumes: the six axes and score scale from `.wiki/schema.md` (Task 2).
- Produces: `wiki/topics/langgraph.md`. Task 7 reads its six scores. Task 8 relies on it for `state` patterns, which the three CLI-shaped tools are unlikely to supply.

**Why this tool.** LangGraph treats persistence and interruption as first-class rather than incidental. The `state` axis needs at least one tool that scores high on it, or the axis produces four low scores and teaches nothing.

- [ ] **Step 1: Gather evidence**

Primary source: the official LangGraph documentation. The `context7` MCP server resolves LangGraph documentation directly and is the faster path when available.

Record the released version:

```bash
python3 -m pip show langgraph 2>/dev/null | grep -i '^version:'
```

If the package is not installed locally, record the current released version from the documentation or package index, and say which.

Look specifically for:

- The unit of composition — nodes, edges, subgraphs — and how one gets registered.
- Checkpointers: what they persist, and where.
- The long-term memory store, and how it differs from a checkpointer.
- Interrupts and human-in-the-loop gates.
- Tracing and inspection facilities.

Note the shape mismatch honestly: LangGraph is a library whose extensions are code, not declarative files. Several axes will read differently here, and that difference is itself a finding worth a sentence in the profile.

- [ ] **Step 2: Write the evidence note**

Create `.wiki/raw/notes/2026-08-04-langgraph-extension-model.md`:

```markdown
---
title: "LangGraph Extension Model"
source: "https://langchain-ai.github.io/langgraph/"
type: notes
ingested: 2026-08-04
tags: [langgraph, extension-unit, state]
summary: "Evidence on LangGraph's graph composition, checkpointers, store, and interrupts. Collected from the official documentation."
---

# LangGraph Extension Model

## Version Examined

[Version string, and whether it came from a local install or the published release.]

## Unit of Composition

[Nodes, edges, subgraphs. How each is declared and registered.]

## Persistence

[Checkpointers and the store: what each persists, where, and across what boundary.]

## Interrupts and Gates

[Human-in-the-loop mechanisms, quoted.]

## Tracing

[What inspection the framework offers when a run misbehaves.]

## Sources

- [Documentation URLs, each with what it established.]
```

- [ ] **Step 3: Write the profile**

Create `.wiki/wiki/topics/langgraph.md`:

```markdown
---
title: "LangGraph"
category: topic
sources: ["raw/notes/2026-08-04-langgraph-extension-model.md"]
created: 2026-08-04
updated: 2026-08-04
tags: [langgraph, agent-framework, state]
aliases: ["LangGraph framework"]
confidence: high
volatility: hot
verified: 2026-08-04
summary: "LangGraph's graph-based extension model and its scores across the six audit axes."
---

# LangGraph

> [One paragraph: what LangGraph treats as an extension, and why its answer to state differs from the CLI-shaped tools.]

## Extension Model

**Unit:** [what the tool treats as an extension]
**Loading:** [how a unit reaches the agent]
**Injection point:** [when it enters context]

[Two or three paragraphs. State plainly where an axis fits a declarative tool better than a library, and score what the framework actually provides rather than forcing an analogy.]

## Rubric Scores

Version examined: [version string]

| Axis | Score | Justification |
|------|-------|---------------|
| `discoverability` | [0-3] | [one sentence] |
| `context-budget` | [0-3] | [one sentence] |
| `composition` | [0-3] | [one sentence] |
| `state` | [0-3] | [one sentence] |
| `side-effect-control` | [0-3] | [one sentence] |
| `observability` | [0-3] | [one sentence] |

## Portability

[Three lines or fewer: what a Claude Code plugin author could take from this framework. Persistence and gating are the likely candidates.]

## Sources

- [LangGraph Extension Model](../../raw/notes/2026-08-04-langgraph-extension-model.md) — graph composition, checkpointers, store, interrupts
```

- [ ] **Step 4: Verify the profile**

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki
f=wiki/topics/langgraph.md
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
test -f raw/notes/2026-08-04-langgraph-extension-model.md || echo "MISSING evidence note"
```

Expected: no output.

- [ ] **Step 5: Update the two directory indexes**

Add to `.wiki/raw/notes/_index.md`:

```markdown
| [2026-08-04-langgraph-extension-model.md](2026-08-04-langgraph-extension-model.md) | Evidence on LangGraph's graph composition, persistence, and interrupts. | langgraph, state | 2026-08-04 |
```

Add to `.wiki/wiki/topics/_index.md`:

```markdown
| [langgraph.md](langgraph.md) | LangGraph's graph-based extension model and six-axis scores. | langgraph, agent-framework | 2026-08-04 |
```

Bump `Last updated:` in both.

- [ ] **Step 6: Append the log entry and commit**

Append to `.wiki/log.md`:

```markdown
## [2026-08-04] compile | 1 source → langgraph profile scored on six axes
```

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
git add .wiki/raw/notes .wiki/wiki/topics .wiki/log.md
git diff --cached --check
git commit -m "feat: profile LangGraph against the audit rubric"
```

---

### Task 7: Build the rubric scoreboard

**Files:**
- Create: `.wiki/wiki/references/rubric-scoreboard.md`
- Modify: `.wiki/wiki/references/_index.md`, `.wiki/log.md`

**Interfaces:**
- Consumes: the six scores from each of `wiki/topics/claude-code.md`, `wiki/topics/codex-cli.md`, `wiki/topics/cursor.md`, `wiki/topics/langgraph.md` (Tasks 3–6).
- Produces: `wiki/references/rubric-scoreboard.md` — the one place showing all four tools on one axis at once. Task 8 reads its columns to find the axes where tools disagree, which is where portable techniques hide.

**Why by hand, and why sources point at notes.** Four tools by six axes is twenty-four cells; a script to assemble them would cost more than the table. The `sources:` field lists the four evidence notes rather than the four profiles, because freshness scoring reads `ingested:` from each source and only the raw notes carry that field. The profiles are reached through dual links in See Also.

- [ ] **Step 1: Write the scoreboard**

Create `.wiki/wiki/references/rubric-scoreboard.md`:

```markdown
---
title: "Rubric Scoreboard"
category: reference
sources: ["raw/notes/2026-08-04-claude-code-extension-model.md", "raw/notes/2026-08-04-codex-cli-extension-model.md", "raw/notes/2026-08-04-cursor-extension-model.md", "raw/notes/2026-08-04-langgraph-extension-model.md"]
created: 2026-08-04
updated: 2026-08-04
tags: [rubric, comparison, extension-unit]
aliases: ["Scoreboard", "Axis Comparison"]
confidence: high
volatility: hot
verified: 2026-08-04
summary: "Six-axis rubric scores for all profiled tools, with the spread that identifies where mechanisms genuinely differ."
---

# Rubric Scoreboard

> Every profiled tool on every axis. The axes with the widest spread are where the tools solve the same problem differently, and where portable techniques are most likely to be found.

## Scores

| Axis | Claude Code | Codex CLI | Cursor | LangGraph | Spread |
|------|-------------|-----------|--------|-----------|--------|
| `discoverability` | [0-3] | [0-3] | [0-3] | [0-3] | [max minus min] |
| `context-budget` | [0-3] | [0-3] | [0-3] | [0-3] | [max minus min] |
| `composition` | [0-3] | [0-3] | [0-3] | [0-3] | [max minus min] |
| `state` | [0-3] | [0-3] | [0-3] | [0-3] | [max minus min] |
| `side-effect-control` | [0-3] | [0-3] | [0-3] | [0-3] | [max minus min] |
| `observability` | [0-3] | [0-3] | [0-3] | [0-3] | [max minus min] |

Copy each score from its profile. A cell that disagrees with its profile is a bug in this table, not a revision of the score.

## Reading the Spread

[One paragraph naming the two or three axes with the widest spread, and what the disagreement consists of. This paragraph is the handoff to Task 8.]

## Where the Rubric Strained

[One paragraph. Record any axis that scored the same for all four tools, or that fit one tool's shape badly. An axis that never discriminates is a candidate for revision before the next tool is profiled — that finding is worth more than the scores themselves.]

## See Also

- [[claude-code|Claude Code]] ([Claude Code](../topics/claude-code.md)) — the porting target
- [[codex-cli|OpenAI Codex CLI]] ([OpenAI Codex CLI](../topics/codex-cli.md)) — thin extension surface
- [[cursor|Cursor]] ([Cursor](../topics/cursor.md)) — glob-scoped rules
- [[langgraph|LangGraph]] ([LangGraph](../topics/langgraph.md)) — first-class persistence

## Sources

- [Claude Code Extension Model](../../raw/notes/2026-08-04-claude-code-extension-model.md) — Claude Code scores
- [OpenAI Codex CLI Extension Model](../../raw/notes/2026-08-04-codex-cli-extension-model.md) — Codex CLI scores
- [Cursor Extension Model](../../raw/notes/2026-08-04-cursor-extension-model.md) — Cursor scores
- [LangGraph Extension Model](../../raw/notes/2026-08-04-langgraph-extension-model.md) — LangGraph scores
```

- [ ] **Step 2: Verify the scores match their profiles**

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki
f=wiki/references/rubric-scoreboard.md
grep -q '\[0-3\]' "$f" && echo "FAIL: unfilled score placeholder remains"
grep -q 'max minus min' "$f" && echo "FAIL: unfilled spread column remains"
for a in discoverability context-budget composition state side-effect-control observability; do
  grep -q "\`$a\`" "$f" || echo "MISSING axis: $a"
done
for t in claude-code codex-cli cursor langgraph; do
  grep -q "\[\[$t|" "$f" || echo "MISSING dual link: $t"
done

# Cross-check every cell against its profile.
for a in discoverability context-budget composition state side-effect-control observability; do
  board=$(grep "^| \`$a\` |" "$f" | awk -F'|' '{gsub(/ /,"",$3); print $3}')
  prof=$(grep "^| \`$a\` |" wiki/topics/claude-code.md | awk -F'|' '{gsub(/ /,"",$3); print $3}')
  [ "$board" = "$prof" ] || echo "MISMATCH $a: scoreboard=$board claude-code=$prof"
done
```

Expected: no output. The cross-check covers the Claude Code column; repeat the same comparison against the other three profiles by substituting their filenames and column numbers (`$4`, `$5`, `$6`).

- [ ] **Step 3: Update the references index**

Add one row to the `## Contents` table in `.wiki/wiki/references/_index.md` and bump `Last updated:`:

```markdown
| [rubric-scoreboard.md](rubric-scoreboard.md) | Six-axis scores for all four profiled tools, with spread. | rubric, comparison | 2026-08-04 |
```

- [ ] **Step 4: Append the log entry and commit**

Append to `.wiki/log.md`:

```markdown
## [2026-08-04] compile | 4 profiles → rubric scoreboard with per-axis spread
```

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
git add .wiki/wiki/references .wiki/log.md
git diff --cached --check
git commit -m "feat: compile rubric scoreboard across four tools"
```

---

### Task 8: Promote patterns and open backlog candidates

**Files:**
- Create: `.wiki/inventory/_index.md`, `.wiki/inventory/candidates/_index.md`
- Create: `.wiki/wiki/concepts/<pattern-slug>.md` — one per qualifying technique
- Create: `.wiki/inventory/candidates/<candidate-slug>.md` — one per pattern card
- Modify: `.wiki/wiki/concepts/_index.md`, `.wiki/wiki/topics/*.md` (See Also links), `.wiki/_index.md`, `.wiki/log.md`

**Interfaces:**
- Consumes: the extension-model sections of all four profiles (Tasks 3–6) and the spread analysis from `wiki/references/rubric-scoreboard.md` (Task 7).
- Produces: pattern cards in `wiki/concepts/` and candidates in `inventory/candidates/`, linked to each other in both directions. These are the deliverable the whole wiki exists to produce.

**The card count is not fixed in advance.** How many techniques appear in two or more places is a finding, not a plan input. Step 1 produces the sighting matrix that decides the count. **If no technique reaches two sightings, write zero cards and record that outcome** — the two-sighting rule exists precisely to make that a legitimate result rather than a reason to lower the bar.

- [ ] **Step 1: Build the sighting matrix**

Read the four profiles' `## Extension Model` sections side by side. For each distinct technique, list which tools show it and where. Keep this scratch work in the session; it is not a committed file.

A technique qualifies as a pattern when two or more tools show it **and** it is portable to a Claude Code plugin. A technique in two tools that Claude Code's architecture forbids stays out; note it in the relevant profile instead.

Likely candidates, listed as hypotheses to test rather than conclusions:

- Deferring detail out of the always-loaded file into referenced files.
- Declaring applicability conditions in the extension's own metadata.
- Gating side effects behind an explicit approval step.
- Persisting state outside the conversation so it survives a restart.

- [ ] **Step 2: Create the inventory layer**

llm-wiki creates `inventory/` lazily, so it does not exist yet. Create only the subdirectory this task fills.

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki
mkdir -p inventory/candidates
```

Write `.wiki/inventory/_index.md`:

```markdown
# Inventory Index

> Durable tracking records for this wiki.

Last updated: 2026-08-04

## Counts by Kind

| Kind | Count | Statuses |
|------|-------|----------|
| `task` | 0 | — |

## Categories

- [Candidates](candidates/_index.md) — proposed changes to the owner's plugins

## Recent Changes

- 2026-08-04: Created the inventory layer with a candidates subdirectory.
```

The inventory index summarizes counts by kind and links to category indexes. It does not list individual records — that is each category index's job.

Write `.wiki/inventory/candidates/_index.md`:

```markdown
# Candidates Index

> Proposed changes to the owner's Claude Code plugins, each traced to a pattern card.

Last updated: 2026-08-04

## Contents

| File | Summary | Tags | Updated |
|------|---------|------|---------|

## Recent Changes

- 2026-08-04: Directory created.
```

Leave `items/`, `entities/`, `corpora/`, and `views/` uncreated. Absent optional layers pass lint check C1; blank scaffolding is what the convention avoids.

- [ ] **Step 3: Write one pattern card per qualifying technique**

For each technique from Step 1, create `.wiki/wiki/concepts/<pattern-slug>.md`. Use a slug naming the technique, not the tool — `deferred-reference-loading`, not `claude-code-references`.

```markdown
---
title: "[Technique Name]"
category: concept
sources: ["raw/notes/[first-tool]-extension-model.md", "raw/notes/[second-tool]-extension-model.md"]
created: 2026-08-04
updated: 2026-08-04
tags: [pattern, [relevant-axis]]
aliases: ["[alternate name]"]
confidence: high
volatility: warm
verified: 2026-08-04
summary: "[Two sentences: the problem and the shape of the fix.]"
---

# [Technique Name]

> [One paragraph abstract.]

## Problem

[What hurts without this technique. Be concrete about the failure, not the abstraction.]

## Technique

[How it works, with a minimal example — the smallest fragment that shows the mechanism.]

## Sightings

At least two, each with an exact path or URL.

- **[Tool one]** — [exact path or URL] — [what this instance does]
- **[Tool two]** — [exact path or URL] — [what this instance does]

## Porting Cost and Risk

[What adopting this in a Claude Code plugin would take, and what could go wrong.]

## Proposed Change

[[<candidate-slug>|<Candidate Title>]] ([<Candidate Title>](../../inventory/candidates/<candidate-slug>.md))

[One paragraph on what specifically changes in which plugin.]

## See Also

- [[<tool-slug>|<Tool Name>]] ([<Tool Name>](../topics/<tool-slug>.md)) — where this was first sighted

## Sources

- [<Note Title>](../../raw/notes/[first-tool]-extension-model.md) — first sighting
- [<Note Title>](../../raw/notes/[second-tool]-extension-model.md) — second sighting
```

The `sources:` list must name at least two notes. One source means the two-sighting rule was not met and the card should not exist.

- [ ] **Step 4: Write one candidate per pattern card**

For each card, create `.wiki/inventory/candidates/<candidate-slug>.md`:

```markdown
---
title: "[Candidate Title]"
kind: task
status: proposed
priority: [p0 | p1 | p2 | p3 | p4]
created: 2026-08-04
updated: 2026-08-04
last_checked: 2026-08-04
next_action: "[The single next concrete step, in one sentence.]"
sources:
  - wiki/concepts/<pattern-slug>.md
tags: [candidate, [relevant-axis]]
confidence: medium
summary: "[One sentence: what changes, in which plugin.]"
---

# [Candidate Title]

## Why Track This

[Why this deserves to survive the session. One paragraph.]

## Change

[What to do, specifically enough to act on without rereading the card.]

## Rationale

[[<pattern-slug>|<Pattern Name>]] ([<Pattern Name>](../../wiki/concepts/<pattern-slug>.md))

[Why this is worth doing, and what it improves.]

## Cost

[Rough effort, and what it touches.]
```

**These field values are fixed by llm-wiki's inventory reference, not chosen here.**

- `kind` must be one of `item`, `ingest-candidate`, `entity`, `corpus`, `question`, `task`, `artifact`, `watch`. A proposed plugin change with a next action is a `task`.
- `status` must be one of `proposed`, `active`, `blocked`, `ingested`, `superseded`, `archived`. A newly opened candidate is `proposed`; it becomes `active` when work starts and `ingested` when the change ships.
- `priority` runs `p0` (highest leverage or urgent) through `p4` (keep for completeness). Assign from the porting cost and the size of the improvement; never leave it unset.
- `sources` is a block list, and its paths resolve from the wiki root.

- [ ] **Step 5: Add backlinks from the profiles**

Each pattern card links to the profiles where it was sighted. Those profiles must link back — the dual-link convention is bidirectional, and a one-way link leaves the Obsidian graph incomplete.

Add or extend a `## See Also` section in each profile that a card cites:

```markdown
## See Also

- [[<pattern-slug>|<Pattern Name>]] ([<Pattern Name>](../concepts/<pattern-slug>.md)) — technique sighted here
```

Bump each modified profile's `updated:` field to today.

- [ ] **Step 6: Verify cards, candidates, and links**

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki

# Zero cards is a valid outcome, but then Step 7 must record it.
cards=$(ls wiki/concepts/*.md 2>/dev/null | grep -v '_index.md' | wc -l | tr -d ' ')
echo "cards: $cards"

for f in $(ls wiki/concepts/*.md 2>/dev/null | grep -v '_index.md'); do
  for k in title category sources created updated tags confidence volatility verified summary; do
    grep -q "^$k:" "$f" || echo "$f MISSING frontmatter: $k"
  done
  for s in "## Problem" "## Technique" "## Sightings" "## Porting Cost and Risk" "## Proposed Change" "## Sources"; do
    grep -q "^$s" "$f" || echo "$f MISSING section: $s"
  done
  # Two-sighting rule: at least two bulleted sightings.
  n=$(sed -n '/^## Sightings/,/^## /p' "$f" | grep -c '^- \*\*')
  [ "$n" -ge 2 ] || echo "$f VIOLATION: $n sighting(s), two required"
  grep -q '\[Technique Name\]' "$f" && echo "$f FAIL: template placeholder remains"
done

# Every card names a candidate, and every candidate exists.
for f in $(ls wiki/concepts/*.md 2>/dev/null | grep -v '_index.md'); do
  grep -o 'inventory/candidates/[a-z0-9-]*\.md' "$f" | sort -u | while read -r c; do
    test -f "$c" || echo "$f references missing candidate: $c"
  done
done

# Every candidate links back to a card and uses valid inventory field values.
for f in $(ls inventory/candidates/*.md 2>/dev/null | grep -v '_index.md'); do
  grep -q 'wiki/concepts/' "$f" || echo "$f MISSING backlink to its pattern card"
  for k in title kind status priority created updated last_checked next_action tags confidence summary; do
    grep -q "^$k:" "$f" || echo "$f MISSING frontmatter: $k"
  done
  grep -qE '^kind: (item|ingest-candidate|entity|corpus|question|task|artifact|watch)$' "$f" \
    || echo "$f INVALID kind (see llm-wiki inventory reference)"
  grep -qE '^status: (proposed|active|blocked|ingested|superseded|archived)$' "$f" \
    || echo "$f INVALID status (see llm-wiki inventory reference)"
  grep -qE '^priority: p[0-4]$' "$f" || echo "$f MISSING or invalid priority"
  grep -q '^## Why Track This' "$f" || echo "$f MISSING section: Why Track This"
  grep -q 'p0 | p1' "$f" && echo "$f FAIL: priority placeholder remains"
done
```

Expected: the card count, and no other output.

- [ ] **Step 7: Update indexes, or record the empty result**

Add one row per card to the `## Contents` table in `.wiki/wiki/concepts/_index.md`, and one row per candidate to `.wiki/inventory/candidates/_index.md`. Bump `Last updated:` in both, and in `.wiki/inventory/_index.md`.

Add the inventory line to Quick Navigation in `.wiki/_index.md`, which now applies because the layer exists:

```markdown
- [Inventory](inventory/_index.md)
```

**If Step 1 produced no qualifying technique,** skip the card and candidate steps and add this line to the `## Recent Changes` section of `.wiki/_index.md` instead:

```markdown
- 2026-08-04: Reviewed four profiles for portable techniques; none reached two sightings. Rubric spread recorded in the scoreboard.
```

That is a real finding about the four seed tools, and it points at the next action: profile a fifth tool rather than weaken the rule.

- [ ] **Step 8: Append the log entry and commit**

Append to `.wiki/log.md`, substituting the actual counts:

```markdown
## [2026-08-04] compile | 4 profiles → N pattern cards, N backlog candidates
```

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
git add .wiki
git diff --cached --check
git commit -m "feat: promote portable patterns and open plugin candidates"
```

---

### Task 9: Close out the wiki state

**Files:**
- Modify: `.wiki/_index.md`, `.wiki/log.md`

**Interfaces:**
- Consumes: every file produced by Tasks 1–8.
- Produces: an accurate master index. Any later session resumes from these statistics, so a wrong count here misleads every future run.

- [ ] **Step 1: Recount and update the master index statistics**

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki
echo "Sources:  $(find raw -name '*.md' ! -name '_index.md' | wc -l | tr -d ' ')"
echo "Articles: $(find wiki -name '*.md' ! -name '_index.md' | wc -l | tr -d ' ')"
echo "Candidates: $(find inventory/candidates -name '*.md' ! -name '_index.md' 2>/dev/null | wc -l | tr -d ' ')"
echo "Outputs:  $(find output -name '*.md' ! -name '_index.md' | wc -l | tr -d ' ')"
```

Write those numbers into the `## Statistics` block of `.wiki/_index.md`, and set `Last compiled:` and `Last lint:` to today.

- [ ] **Step 2: Verify index discipline held**

The spec's index rule is the one that keeps this wiki from repeating the hub's 91 KB raw index. Check it:

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki
wc -c _index.md
grep -c 'raw/notes/2026' _index.md
```

Expected: `_index.md` under 4 KB, and zero individual raw-file references in the master index. Raw files belong in their own directory indexes.

- [ ] **Step 3: Run the wiki health check**

In an interactive session with command access:

```
/wiki:lint --local
```

Resolve every issue reported as critical. Warnings and suggestions are the owner's call. If the executing session has no command access, run the structural checks from Tasks 1 through 8 again as the substitute and say in the commit body that lint was deferred.

- [ ] **Step 4: Append the log entry and commit**

Append to `.wiki/log.md`:

```markdown
## [2026-08-04] lint | Structure, indexes, and links verified; statistics recounted
```

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
git add .wiki
git diff --cached --check
git commit -m "chore: recount wiki statistics and verify structure"
```

---

### Task 10: Document the repository and narrow the `AGENTS.md` rules

**Files:**
- Create: `README.md`
- Modify: `AGENTS.md`, `CLAUDE.md`

**Interfaces:**
- Consumes: the wiki tree from Task 1. Nothing else. **This task depends only on Task 1 and may run immediately after it** — it is numbered last so the earlier tasks' cross-references stay stable, not because it must run last.
- Produces: the root-level documentation that points readers into `.wiki/`, and the convention amendments that make the wiki tree legal under this repository's own rules.

**Why the rules need amending.** `AGENTS.md` was written for a conventional application repository. Two of its rules now conflict with a repository whose layout llm-wiki dictates, and the spec resolves both. Leaving the conflict unresolved would mean every future contributor reads a rule the repository visibly breaks.

**One deviation from the spec.** The spec listed the `CLAUDE.md` update as an obligation of the Python commit in milestone 4. That is too late: `CLAUDE.md` currently states that `AGENTS.md` is the only tracked file and that no source or docs exist. Task 1 falsifies that sentence, so the update moves here.

- [ ] **Step 1: Write `README.md`**

````markdown
# agentic-tools-audit

A personal knowledge library comparing how agentic tools express extensions, kept to inform Claude Code plugin development.

The content lives in `.wiki/`, an [llm-wiki](https://github.com/casentino) local wiki. Start at `.wiki/_index.md`.

## Layout

| Path | Holds |
|------|-------|
| `.wiki/schema.md` | The six-axis rubric, vocabulary, and evidence rules |
| `.wiki/wiki/topics/` | One profile per tool |
| `.wiki/wiki/concepts/` | Portable techniques, each sighted in two or more tools |
| `.wiki/wiki/references/` | The cross-tool scoreboard |
| `.wiki/inventory/candidates/` | Proposed plugin changes |
| `docs/superpowers/` | Design specs and implementation plans |

## Commands

All wiki commands run from the repository root with `--local`, which resolves the wiki to `.wiki/`:

```
/wiki:query "<question>" --local
/wiki:lint --local
/wiki:ingest <url|file> --local
```

Repository checks:

```bash
git status --short
git diff --check
```

No build or test toolchain exists yet. The measurement pipeline will add Python, `ruff`, and `pytest`; this file gains those commands at that point.

## Conventions

`AGENTS.md` governs the repository. Inside `.wiki/`, llm-wiki conventions govern.
````

- [ ] **Step 2: Narrow the `AGENTS.md` structure rule**

In `AGENTS.md`, replace the `## Project Structure & Module Organization` paragraph with:

```markdown
This repository holds a knowledge wiki, not an application. The wiki lives in `.wiki/` and follows [llm-wiki](https://github.com/casentino) conventions; inside that directory, those conventions govern layout, frontmatter, and indexes. Keep the root for project-wide files such as `README.md`, manifests, and tool configuration, and keep design specs and plans in `docs/`.

Outside `.wiki/`, place code in `src/`, tests in `tests/`, and sanitized test data in `tests/fixtures/`. Code that serves the wiki is the exception: it belongs in `.wiki/output/projects/<slug>/code/` with its tests alongside, matching how llm-wiki organizes project code.

Do not commit dependency directories or raw collection material. Measurement summaries under `.wiki/output/projects/*/data/` are the record and belong in version control; anything a re-ingest can reproduce stays ignored.
```

- [ ] **Step 3: Update `CLAUDE.md`**

`CLAUDE.md`'s `## Repository state` section says `AGENTS.md` is the only tracked file and that no source, docs, or build configuration exist. Replace that section with:

```markdown
## Repository state

This repository holds a knowledge wiki at `.wiki/`, an llm-wiki local wiki registered in the hub at `/Users/kikyeongoh/Documents/opterk/llm-wiki` under `local_wikis`. Start at `.wiki/_index.md`; the rubric and evidence rules live in `.wiki/schema.md`.

There is still no build or test toolchain. The only repository-level checks are `git status --short` and `git diff --check`. Wiki operations run through llm-wiki commands with `--local` from the repository root.

Design specs and implementation plans live in `docs/superpowers/`.
```

Keep the rest of `CLAUDE.md` — the `AGENTS.md`-is-authoritative section and the toolchain obligations still hold. Delete the `## Domain note` section: its guess about scope has been replaced by the spec.

- [ ] **Step 4: Verify the documentation matches reality**

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
test -f README.md || echo "MISSING README.md"
grep -q '\.wiki/' README.md || echo "README does not point into .wiki/"
grep -q 'bootstrap state' AGENTS.md && echo "FAIL: AGENTS.md still claims bootstrap state"
grep -q 'only tracked file' CLAUDE.md && echo "FAIL: CLAUDE.md still claims one tracked file"
grep -q '## Domain note' CLAUDE.md && echo "FAIL: CLAUDE.md still carries the superseded domain guess"
grep -q 'llm-wiki conventions govern' AGENTS.md || echo "FAIL: AGENTS.md lacks the .wiki/ exemption"
```

Expected: no output.

- [ ] **Step 5: Commit**

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit
git add README.md AGENTS.md CLAUDE.md
git diff --cached --check
git commit -m "docs: document the wiki layout and reconcile repository conventions"
```

---

## Out of Plan Scope

These spec elements belong to milestone 4 and have no task here. They are deferred, not forgotten:

- `.wiki/output/projects/ecosystem-metrics/` — the measurement project, including `WHY.md`, `code/measure.py`, and `data/`.
- `pyproject.toml`, `ruff`, and `pytest` — the Python toolchain arrives with the measurement pipeline.
- `inventory/corpora/` collection records and `raw/repos/` collection manifests — these come from `/wiki:ingest-collection`, which milestone 4 runs. Milestone 1–3 evidence is hand-written notes in `raw/notes/`.
- The `measured-by` relationship verb defined in `schema.md` — declared now, used once measurements exist.

## Definition of Done

- `.wiki/` exists, is tracked by git, and `.gitignore` does not ignore it.
- The hub's `wikis.json` lists this wiki under `local_wikis`, and its `wikis` map is unchanged.
- `schema.md` defines exactly six axes and a four-level score scale.
- Four profiles exist, each scoring all six axes with a justification per axis and a recorded version or SHA.
- Each profile cites an evidence note in `raw/notes/`.
- The scoreboard's cells match their profiles.
- Every pattern card has two or more sightings and a candidate; every candidate links back to its card.
- Every candidate's `kind`, `status`, and `priority` hold values llm-wiki's inventory reference allows.
- The master `_index.md` reports accurate counts, stays under 4 KB, and references no individual raw file.
- `README.md` exists and points into `.wiki/`; `AGENTS.md` and `CLAUDE.md` no longer describe a bootstrap repository.
- `git status` is clean in both repositories.
