---
title: "Deferred Reference Loading"
category: concept
sources: ["raw/notes/2026-08-04-claude-code-extension-model.md", "raw/notes/2026-08-04-codex-cli-extension-model.md", "raw/notes/2026-08-04-cursor-extension-model.md"]
created: 2026-08-04
updated: 2026-08-04
tags: [pattern, context-budget]
aliases: ["Progressive Disclosure", "Description-Resident, Body-Deferred Loading"]
confidence: high
volatility: warm
verified: 2026-08-04
summary: "Keeping the always-loaded part of an extension short by moving procedural detail into separate files the tool only reads when the task actually needs them."
---

# Deferred Reference Loading

> Three of the four profiled tools split an extension into a short, always-resident trigger (a name, a description, a one-line pointer) and a larger body of procedural detail that only enters context once something concrete needs it — an invocation, a file touch, an explicit `@`-mention. The always-resident part answers "does this apply and what is it," never "how exactly do I do it."

## Problem

An extension author who inlines everything — every edge case, every worked example, every troubleshooting step — into the one file the tool loads at every session start pays that file's full token cost in every session, whether or not the content is ever used. Multiply this across a dozen installed skills or a long CLAUDE.md and the always-loaded context fills with material that is relevant to at most one task in twenty. The failure mode is concrete, not abstract: a skill author keeps adding detail to `SKILL.md` because that is the only file they know the tool reads, until the file itself becomes the budget problem it was meant to solve.

## Technique

Split the extension into two layers. The thin layer — a name plus a one- or two-sentence description — stays resident so the tool (or the model reading it) can decide relevance. The thick layer — the actual steps, scripts, edge cases — lives in separate files that load only on demand: when the extension is invoked, when a matching file is touched, or when explicitly referenced.

Minimal shape, taken directly from an installed skill on the audit machine (`~/.claude/plugins/cache/llm-wiki/wiki/0.16.0/skills/wiki-manager/SKILL.md`): the description frontmatter field carries the full triggering detail, and each body section is reduced to a pointer:

```markdown
### Ingestion
See [references/ingestion.md](references/ingestion.md).
```

The 17 files under that skill's own `references/` directory hold the actual procedures; `SKILL.md` itself holds only the map.

## Sightings

- **Claude Code** — `~/.claude/plugins/cache/llm-wiki/wiki/0.16.0/skills/wiki-manager/SKILL.md` — a skill whose body "defers detail to 17 files under its own `references/` directory ... rather than inlining everything in `SKILL.md` itself," with each workflow section reduced to one line plus a link; more generally, every Claude Code skill keeps only its `description` resident and loads the full body "only when invoked" (https://code.claude.com/docs/en/skills).
- **OpenAI Codex CLI** — https://learn.chatgpt.com/docs/build-skills — "ChatGPT and Codex start with each skill's name and description, then load the full `SKILL.md` instructions when they decide to use that skill," with the resident list "constrained to approximately 2% of context window or 8,000 characters"; the documented directory layout itself includes optional `scripts/`, `references/`, `assets/` alongside the required `SKILL.md`, the same references-directory shape as the Claude Code skill above.
- **Cursor** — https://cursor.com/docs/context/rules — "Use `@filename.ts` to include files in your rule's context," documented so "a rule body does not have to inline everything it needs" and can "point at canonical source files rather than duplicating their content into the rule body."

## Porting Cost and Risk

Claude Code already hosts this mechanism natively for skills (resident description, deferred body) and for `.claude/rules/*.md` (loaded unconditionally unless path-scoped, see [[path-scoped-activation|Path-Scoped Activation]]) — nothing needs inventing at the host level. The cost is entirely authoring discipline: auditing existing `SKILL.md`/`CLAUDE.md` files for length and moving procedural detail that is not needed on every invocation into `references/*.md`, mirroring the `wiki-manager` skill's own pattern. The risk is under-splitting (a `references/` file so critical to the mechanism's correctness that deferring it causes silent misbehavior when it isn't read) or over-splitting (so many one-line pointers that a reader loses the shape of the workflow before ever opening a reference file). Neither risk is a new tool capability to build — both are ordinary authoring judgment calls the technique does not obviate.

## Proposed Change

[[audit-skill-progressive-disclosure|Audit Plugin Skills for Progressive Disclosure]] ([Audit Plugin Skills for Progressive Disclosure](../../inventory/candidates/audit-skill-progressive-disclosure.md))

Audit the owner's installed and authored plugins' `SKILL.md` files for length, and apply the same split the `wiki-manager` skill already uses to any that inline detail unnecessarily.

## See Also

- [[claude-code|Claude Code]] ([Claude Code](../topics/claude-code.md)) — skill description/body split; the `wiki-manager` reference-file example
- [[codex-cli|OpenAI Codex CLI]] ([OpenAI Codex CLI](../topics/codex-cli.md)) — skill progressive disclosure with a numeric resident-budget cap
- [[cursor|Cursor]] ([Cursor](../topics/cursor.md)) — `@filename.ts` deferred file reference inside a rule body

## Sources

- [Claude Code Extension Model](../../raw/notes/2026-08-04-claude-code-extension-model.md) — first sighting; the `wiki-manager` skill's references-directory pattern and the general description/body split
- [OpenAI Codex CLI Extension Model](../../raw/notes/2026-08-04-codex-cli-extension-model.md) — second sighting; progressive disclosure with a numeric context-budget cap
- [Cursor Extension Model](../../raw/notes/2026-08-04-cursor-extension-model.md) — third sighting; `@filename.ts` deferred content inside a rule
