---
title: "Add Path-Scoped Activation to Stack-Specific Rules"
kind: task
status: proposed
priority: p1
created: 2026-08-04
updated: 2026-08-04
last_checked: 2026-08-04
next_action: "Add a paths: frontmatter glob (e.g. paths: [\"**/*.ts\", \"**/*.tsx\"]) to each TypeScript/JavaScript-specific file under ~/.claude/rules/ so it stops loading unconditionally in non-TS/JS repositories."
sources:
  - wiki/concepts/path-scoped-activation.md
tags: [candidate, discoverability]
confidence: high
summary: "Scope the owner's TypeScript/JavaScript-specific personal rules to matching file paths instead of loading them unconditionally in every repository."
---

# Add Path-Scoped Activation to Stack-Specific Rules

## Why Track This

This is the clearest, cheapest, most directly evidenced finding in this audit: `~/.claude/rules/coding-style.md`, `testing.md`, `hooks.md`, `patterns.md`, and `security.md` are explicitly TypeScript/JavaScript-scoped (their own headers say so — e.g. "TypeScript/JavaScript Coding Style"), carry no `paths` frontmatter, and per Claude Code's own documentation therefore "are loaded unconditionally at every session start." This very audit session, in a repository that is Markdown and shell rather than TypeScript, loaded all of them anyway. The fix is a documented, native Claude Code mechanism the owner is simply not using yet, with no host-level change required.

## Change

For each stack-specific file under `~/.claude/rules/` (starting with `coding-style.md`, `testing.md`, `hooks.md`, `patterns.md`, `security.md`), add a `paths:` YAML frontmatter block scoping it to the relevant file extensions, e.g.:

```markdown
---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---
```

Leave genuinely universal rules (e.g. `git-workflow.md`, `agents.md`) without `paths` so they continue loading everywhere. Verify with `/context` in a non-TS/JS repository (such as this one) that the scoped files no longer appear under loaded memory files, and in a TS/JS repository that they still do.

## Rationale

[[path-scoped-activation|Path-Scoped Activation]] ([Path-Scoped Activation](../../wiki/concepts/path-scoped-activation.md))

Cursor's `globs` mechanism and Claude Code's own `paths` field are the identical mechanical, tool-computed applicability check — the rubric scoreboard names this exact mechanism as the reason Cursor's `discoverability` axis outscores Claude Code's. Claude Code already has the mechanism; using it removes five TS/JS-specific rule files' worth of context-budget cost from every session run in a non-TS/JS repository, with no loss of coverage in repositories where they do apply.

## Cost

Very low: a frontmatter addition to five existing files, no new tooling, no plugin code. The only judgment call is picking correct glob patterns per file (risk of too-narrow under-triggering or too-broad over-triggering), verifiable directly with `/context` after the change.
