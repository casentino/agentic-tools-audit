---
title: "Path-Scoped Activation"
category: concept
sources: ["raw/notes/2026-08-04-claude-code-extension-model.md", "raw/notes/2026-08-04-cursor-extension-model.md"]
created: 2026-08-04
updated: 2026-08-04
tags: [pattern, discoverability]
aliases: ["Glob-Scoped Auto-Attach", "Mechanical Applicability Metadata"]
confidence: high
volatility: warm
verified: 2026-08-04
summary: "Declaring a file-path or glob condition directly in an extension's own metadata so the tool computes applicability mechanically, instead of leaving relevance entirely to the model's reading of a free-text description."
---

# Path-Scoped Activation

> Two of the four profiled tools let an extension declare, in its own frontmatter, a glob pattern that the tool itself checks against the file currently being worked on — a mechanical, verifiable "does this apply" test that sits alongside (not instead of) the model-judged description. Cursor's `discoverability` score (3) does sit above Claude Code's and Codex CLI's (2 each), but this mechanism is not what separates them: Claude Code ships the identical glob check under the field name `paths`, on both skills and rules (see Technique). The gap is a product of how the axis was aggregated — which mechanism a profile let set the score — not of the mechanism being absent from one tool. The scoreboard carries this as an open finding rather than a settled verdict; see the `## Where the Rubric Strained` section of [[rubric-scoreboard|Rubric Scoreboard]] ([Rubric Scoreboard](../references/rubric-scoreboard.md)), whose paragraph on `paths` records that the field "is the same class of mechanical match this scoreboard credits for Cursor's `discoverability` = 3" yet is cited nowhere in Claude Code's `discoverability` row.

## Problem

When the only way an agent learns "this extension applies right now" is a free-text description the model reads and judges, relevance depends on the model correctly matching task to description every time — and nothing verifies it did. This shows up concretely on this very machine: all eight of the owner's `~/.claude/rules/*.md` files carry no `paths` frontmatter, and per Claude Code's own documentation "Rules without a `paths` field are loaded unconditionally and apply to all files" — including in this very audit repository, which is Markdown and shell, not TypeScript. Five of those eight are explicitly TypeScript/JavaScript-specific, each announcing it in its own title: `coding-style.md`, `hooks.md`, `patterns.md`, `security.md`, and `testing.md`. The other three — `agents.md`, `git-workflow.md`, and `performance.md` — are stack-neutral and legitimately belong in every session. The mechanism to scope the five exists and is documented; it simply is not used, so five TS/JS-specific rule files consume context budget in every non-TS/JS session on this machine.

## Technique

Give the extension's frontmatter a glob field the tool checks mechanically against files the agent is currently reading or editing, and treat that as an independent activation path alongside (not a replacement for) description-based matching.

Cursor's `.mdc` rules document this directly as one of four scoping mechanisms: `globs` alone drives "Apply to Specific Files," whose documented trigger reads "When file matches a specified pattern," and whose row in the frontmatter-behavior table (`alwaysApply: false`, no `description`, `globs` provided) reads "Auto-attached when a matching file is in context." Calling that a mechanical, tool-computed path match is this card's characterization, not the page's wording. The page gives its example patterns verbatim, among them `*.ts`, `**/*.ts`, `src/**`, and `src/**/*.tsx`.

Claude Code's own rule and skill frontmatter carry the identical mechanism under the field name `paths`, confirmed directly against the current docs:

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Development Rules
- All API endpoints must include input validation
```

"Rules can be scoped to specific files using YAML frontmatter with the `paths` field. These conditional rules only apply when Claude is working with files matching the specified patterns" (https://code.claude.com/docs/en/memory). The identical field exists for skills too: "`paths` ... Glob patterns that limit when this skill is activated ... When set, Claude loads the skill automatically only when working with files matching the patterns" (https://code.claude.com/docs/en/skills).

## Sightings

- **Cursor** — https://cursor.com/docs/context/rules — "Apply to Specific Files" (`globs` frontmatter field), documented trigger "When file matches a specified pattern," with the frontmatter-behavior table adding "Auto-attached when a matching file is in context." Example patterns `*`, `**`, `*.ts`, `**/*.ts`, `src/**`, `src/**/*.tsx` are given verbatim on the page. That this constitutes a mechanical, tool-computed path match is this card's reading of those two sentences, not a quotation from them.
- **Claude Code** — https://code.claude.com/docs/en/memory (Path-specific rules section) and https://code.claude.com/docs/en/skills (`paths` frontmatter row) — a `paths` field on both `.claude/rules/*.md` files and `SKILL.md` frontmatter, documented as glob-matched, tool-computed activation; confirmed unused on this machine's own eight rule files, all of which therefore load unconditionally regardless of project type.

## Porting Cost and Risk

There is nothing to build: the mechanism is already native to Claude Code, on both skills and rules, and requires no new tool capability — only adding a `paths:` list to frontmatter that currently omits it. The cost is an audit pass over existing rule/skill files to identify which ones are stack- or directory-specific rather than universal, plus picking correct glob patterns (an easy place to get subtly wrong — too narrow silently under-triggers, too broad defeats the point). The risk worth naming: Cursor's own documentation admits the parallel same-scope gap this technique does not close — two rules whose globs both match one file merge with no documented order between them, "an inference from absence, not a recorded fact" per that tool's own note — so scoping activation by path does not by itself solve overlap between two path-scoped rules that both match (see [[specificity-ordered-precedence|Specificity-Ordered Precedence]] for that separate, related gap).

## Proposed Change

[[add-path-scoped-rule-activation|Add Path-Scoped Activation to Stack-Specific Rules]] ([Add Path-Scoped Activation to Stack-Specific Rules](../../inventory/candidates/add-path-scoped-rule-activation.md))

Add `paths:` frontmatter to the owner's TypeScript/JavaScript-specific `~/.claude/rules/*.md` files so they stop loading unconditionally in repositories, like this one, that are not TypeScript/JavaScript projects.

## See Also

- [[claude-code|Claude Code]] ([Claude Code](../topics/claude-code.md)) — the `paths` frontmatter field on skills and rules
- [[cursor|Cursor]] ([Cursor](../topics/cursor.md)) — `globs`-driven "Apply to Specific Files"

## Sources

- [Cursor Extension Model](../../raw/notes/2026-08-04-cursor-extension-model.md) — first sighting; the four scoping mechanisms and the `globs` mechanical match
- [Claude Code Extension Model](../../raw/notes/2026-08-04-claude-code-extension-model.md) — second sighting; the `paths` field on skills and rules, verified directly against the current docs, plus confirmation the eight local rule files leave it unset
