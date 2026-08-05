---
title: "State Explicit Precedence in Overlapping Plugin Rule Files"
kind: task
status: proposed
priority: p3
created: 2026-08-04
updated: 2026-08-04
last_checked: 2026-08-04
next_action: "When authoring or reviewing plugin rule/skill files whose paths globs can overlap, or nested CLAUDE.md files, add one sentence to the narrower file stating it overrides the broader guidance for matching files."
sources:
  - wiki/concepts/specificity-ordered-precedence.md
tags: [candidate, composition]
confidence: medium
summary: "Add explicit override language to narrower plugin rule/skill files whenever their scope can overlap a broader one, since Claude Code will not infer precedence itself."
---

# State Explicit Precedence in Overlapping Plugin Rule Files

## Why Track This

Claude Code's own documentation names, by direct quote, the failure mode this closes: "if two rules contradict each other, Claude may pick one arbitrarily." Nested `CLAUDE.md` files are explicitly concatenated rather than override-resolved, unlike Codex CLI's and Cursor's documented closer-file-wins conventions for their own nested instruction files. The owner's current rule set is flat (no nesting) so this is not an active bug today, but it becomes relevant the moment [[add-path-scoped-rule-activation|Add Path-Scoped Activation to Stack-Specific Rules]] or any future plugin work introduces multiple `paths`-scoped rules whose globs can overlap on the same file. Tracking it now means the authoring convention exists before the first real overlap, not after a silent, hard-to-reproduce contradiction is debugged.

## Change

Adopt a one-sentence authoring convention for any plugin rule, skill, or memory file whose scope is nested inside, or can overlap with, a broader one: state directly in the narrower file that it overrides the broader guidance for matching files (e.g. "This overrides the general TypeScript style guide above for files under `src/api/`."). Apply this specifically when reviewing the outcome of the path-scoping candidate above, and any time a new plugin ships more than one rule/skill file with `paths` globs that could both match the same file.

## Rationale

[[specificity-ordered-precedence|Specificity-Ordered Precedence]] ([Specificity-Ordered Precedence](../../wiki/concepts/specificity-ordered-precedence.md))

Codex CLI and Cursor both state a deterministic "closer/more-specific file wins" rule for their own nested instruction files; Claude Code documents the opposite (concatenation, arbitrary resolution on contradiction) and names the resulting problem directly in its own troubleshooting text. Since Claude Code will not compute this precedence itself, stating it explicitly in the file is the only way to get the same deterministic outcome the other two tools provide natively.

## Cost

Very low: a documentation/authoring habit, not a code or config change. The only ongoing cost is remembering to apply it, which is why it is tracked here rather than left to memory — and it only matters where a genuine overlap exists, so it should not be applied as blanket boilerplate to every rule file regardless of whether overlap is possible.
