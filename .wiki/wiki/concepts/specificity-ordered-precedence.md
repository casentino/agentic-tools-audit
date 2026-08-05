---
title: "Specificity-Ordered Precedence"
category: concept
sources: ["raw/notes/2026-08-04-codex-cli-extension-model.md", "raw/notes/2026-08-04-cursor-extension-model.md"]
created: 2026-08-04
updated: 2026-08-04
tags: [pattern, composition]
aliases: ["Closer-File-Wins", "Nested Instruction Precedence"]
confidence: high
volatility: warm
verified: 2026-08-04
summary: "Stating a deterministic rule that a more specific (closer-directory) instruction file overrides a broader one when the two conflict, rather than concatenating both and leaving the contradiction to model judgment."
---

# Specificity-Ordered Precedence

> Two of the four profiled tools state a deterministic rule for what happens when a nested, more-specific instruction file disagrees with a broader one above it: the closer file wins. Claude Code documents the opposite for its own nested `CLAUDE.md`/`.claude/rules/` files — concatenation with no override — and names the resulting failure mode directly in its own troubleshooting text.

## Problem

Two instruction files that both apply to the current task but give conflicting guidance are, by default, simply concatenated — both enter context, and nothing tells the agent which one should win. Claude Code's own documentation names this exact failure by direct quote: "if two rules contradict each other, Claude may pick one arbitrarily. Review your CLAUDE.md files, nested CLAUDE.md files in subdirectories, and `.claude/rules/` periodically to remove outdated or conflicting instructions." That sentence exists because the tool has no deterministic tie-break for this case — "All discovered files are concatenated into context rather than overriding each other" — leaving the outcome to whichever way the model happens to read the contradiction that session, which need not be consistent run to run.

## Technique

When a more specific instruction file (nested deeper in the directory tree, or otherwise scoped narrower) can overlap with a broader one, state a rule that the narrower file wins on conflict, and apply that rule deterministically rather than leaving both in context as an unresolved contradiction.

Codex CLI states the rule for its own `AGENTS.md` nesting as a direct consequence of concatenation order: "Codex concatenates files from the root down, joining them with blank lines. Files closer to your current directory override earlier guidance because they appear later in the combined prompt." Cursor states the identical rule for its nested `AGENTS.md` files: "Instructions from nested `AGENTS.md` files are combined with parent directories, with more specific instructions taking precedence" — the child directory's file is the more specific source and wins on conflict.

## Sightings

- **OpenAI Codex CLI** — https://learn.chatgpt.com/docs/agent-configuration/agents-md — "Codex concatenates files from the root down, joining them with blank lines. Files closer to your current directory override earlier guidance because they appear later in the combined prompt."
- **Cursor** — https://cursor.com/docs/context/rules — "Instructions from nested `AGENTS.md` files are combined with parent directories, with more specific instructions taking precedence."

## Porting Cost and Risk

This is a documentation/authoring practice, not a tool feature, so the cost is near zero: no code changes, no new Claude Code capability to request. When a plugin or personal configuration ships multiple rule or skill files whose `paths` globs (see [[path-scoped-activation|Path-Scoped Activation]]) can both match the same file — or when nested `CLAUDE.md`/`.claude/rules/` files could plausibly overlap — the more specific file should say, in one explicit sentence, that it overrides the broader guidance for matching files, since Claude Code will not infer that order itself. The risk in skipping this: exactly the failure Claude Code's own docs name, silent and non-reproducible arbitration that can pass code review because it "worked" in whatever session someone happened to test. The risk in over-applying it: writing override language for files that never actually overlap adds noise without benefit, so this is worth doing only where a genuine specificity relationship exists (a narrower rule scoped inside a broader one's domain), not as a blanket habit across every rule file.

## Proposed Change

[[state-explicit-rule-precedence|State Explicit Precedence in Overlapping Plugin Rule Files]] ([State Explicit Precedence in Overlapping Plugin Rule Files](../../inventory/candidates/state-explicit-rule-precedence.md))

When authoring plugin rule or skill files with overlapping `paths` scope, or nested memory files, add one explicit sentence stating that the more specific file overrides the broader one on conflict.

## See Also

- [[codex-cli|OpenAI Codex CLI]] ([OpenAI Codex CLI](../topics/codex-cli.md)) — closer-`AGENTS.md`-wins via concatenation order
- [[cursor|Cursor]] ([Cursor](../topics/cursor.md)) — nested `AGENTS.md` child-wins precedence

## Sources

- [OpenAI Codex CLI Extension Model](../../raw/notes/2026-08-04-codex-cli-extension-model.md) — first sighting; closer-file-wins via concatenation order
- [Cursor Extension Model](../../raw/notes/2026-08-04-cursor-extension-model.md) — second sighting; nested `AGENTS.md` child-wins precedence
