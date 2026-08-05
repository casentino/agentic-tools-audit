---
title: "Audit Plugin Skills for Progressive Disclosure"
kind: task
status: proposed
priority: p2
created: 2026-08-04
updated: 2026-08-04
last_checked: 2026-08-04
next_action: "List every installed plugin's SKILL.md line count (wc -l) and, for any that exceed roughly 100-150 lines, move procedural detail into a references/*.md file the skill links to, following the wiki-manager skill's own pattern."
sources:
  - wiki/concepts/deferred-reference-loading.md
tags: [candidate, context-budget]
confidence: medium
summary: "Audit installed SKILL.md files for unnecessary bulk and split detail into references/ files, matching the pattern the owner's own wiki-manager skill already uses."
---

# Audit Plugin Skills for Progressive Disclosure

## Why Track This

The audit found the owner already has one skill (`wiki-manager`, part of the `llm-wiki` plugin) applying this pattern well — description-driven activation with all procedural detail pushed into 17 `references/*.md` files, leaving `SKILL.md` itself as a one-line-per-section index. Not every installed skill necessarily follows the same discipline. This is worth tracking because it is a cheap, mechanical check (a line count) that surfaces a real, if modest, context-budget cost every session pays for any skill that inlines detail unnecessarily, and it deserves a survey pass rather than being assumed fine because one example does it well.

## Change

Run a line-count sweep over every installed skill's `SKILL.md` (`find ~/.claude/plugins/cache/*/*/*/skills/*/SKILL.md` per the evidence note's own glob, 87 files were found at audit time). For any file whose body — excluding frontmatter — is long relative to its actual invocation-time need, move the bulk into one or more `references/*.md` files and replace the inlined content with a short pointer, mirroring the `wiki-manager` skill's `### Ingestion\nSee [references/ingestion.md](references/ingestion.md).` shape. Skills that are already short, or whose body is genuinely needed in full on every invocation, need no change — this is a targeted split, not a blanket rewrite.

## Rationale

[[deferred-reference-loading|Deferred Reference Loading]] ([Deferred Reference Loading](../../wiki/concepts/deferred-reference-loading.md))

Three profiled tools (Claude Code, Codex CLI, Cursor) converge on the same shape — a thin, always-resident trigger plus a thick, deferred body — as the mechanism behind their strongest context-budget scores. Applying it consistently across the owner's own installed skills, not just the one example already doing it, keeps every session's resident context proportional to what a task actually needs rather than to how many skills happen to be installed.

## Cost

Low-to-medium: the survey itself is a single `wc -l` sweep over ~87 files, a few minutes of scripting. The per-file split work varies — a skill that is already lean needs nothing, while a genuinely bloated one requires judgment about what is safe to defer versus what must stay inline for correctness. No tool or plugin code changes; this only touches skill authoring, and only for skills the owner controls or can propose changes to.
