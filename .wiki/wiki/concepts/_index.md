# Concepts Index

> Portable techniques extracted from two or more tools.

Last updated: 2026-08-04

## Contents

| File | Summary | Tags | Updated |
|------|---------|------|---------|
| [deferred-reference-loading.md](deferred-reference-loading.md) | Keeping the always-loaded part of an extension short by moving procedural detail into separate files the tool only reads when the task actually needs them. | pattern, context-budget | 2026-08-04 |
| [path-scoped-activation.md](path-scoped-activation.md) | Declaring a file-path or glob condition directly in an extension's own metadata so the tool computes applicability mechanically, instead of leaving relevance entirely to the model. | pattern, discoverability | 2026-08-04 |
| [tiered-persistence-split.md](tiered-persistence-split.md) | Keeping the literal record of what happened in one mechanism, and a separate, purpose-built store of distilled facts that outlive it, in another. | pattern, state | 2026-08-04 |
| [specificity-ordered-precedence.md](specificity-ordered-precedence.md) | Stating a deterministic rule that a more specific instruction file overrides a broader one on conflict, rather than concatenating both and leaving the contradiction to model judgment. | pattern, composition | 2026-08-04 |

## Considered, Not Promoted

- **Gating side effects behind an explicit approval step** — sighted in Claude Code (permission modes, deny-always-wins, protected paths, auto-mode classifier), OpenAI Codex CLI (sandbox modes, approval policies, a "most restrictive wins" rules engine), and Cursor (approval-gated MCP/terminal/file edits, Run Modes, `permissions.json`) — cleared the two-sighting rule but was not promoted to a card. Reason: Claude Code's own profile already scores `side-effect-control` at 3, its ceiling, and its Portability section states directly that "a porting target does not need to invent any of those" — there is no gap left for a plugin author to close. This is a third exclusion reason distinct from the two named gates (failing the two-sighting count, or failing portability because Claude Code's architecture cannot host the technique): the technique is portable in principle, and Claude Code already implements it better than every tool it was sighted in.

## Recent Changes

- 2026-08-04: Directory created.
- 2026-08-04: Added 4 pattern cards, each sighted in 2-3 of the four profiled tools' Extension Model sections and traced to a backlog candidate in `inventory/candidates/`.
