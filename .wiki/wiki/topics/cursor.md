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

> Cursor treats an extension as a rule — a `.mdc` file, a plain `AGENTS.md` file, or an app/dashboard-managed User or Team rule — and decides whether it applies now through three mostly-mechanical checks (an always-on flag, a file-glob match, or an explicit `@`-mention) plus one model-judged check (a description), rather than through description matching alone; that mix of deterministic and judged triggers is the thing that makes Cursor's scoping mechanism genuinely different from a pure description-matching tool.

## Extension Model

**Unit:** A rule. Four storage shapes exist: a `.mdc` file with three-field YAML frontmatter (`description`, `globs`, `alwaysApply`) under `.cursor/rules` (version-controlled, project scope); a plain `AGENTS.md` file at the project root or nested in any subdirectory; an app-local **User Rule** (no file — a setting under Customize → Rules, machine-local, not version-controlled); and a dashboard-managed **Team Rule** (org scope; an admin can enforce one so it "cannot be disabled in Customize").

**Loading:** Project `.mdc` files and `AGENTS.md` are read straight from the repository the same way any tracked file is; User Rules load from local app settings; Team Rules sync down from the Cursor dashboard. Which of these actually attach to a given turn is decided per-rule by which of the three frontmatter fields is set (see below), not by loading everything unconditionally.

**Injection point:** All rules judged applicable are merged into context together in a fixed order — Team Rules first, then Project Rules, then User Rules — with "earlier sources take precedence when guidance conflicts." Nested `AGENTS.md` files combine similarly, parent then child, with the child (more specific) directory winning on conflict. A rule body is not required to inline everything it needs: `@filename.ts` inside a rule pulls a file into context by reference.

The scoping fields, by exact name, drive four distinct mechanisms: `alwaysApply: true` (unconditional, every session); a `description` alone drives **Apply Intelligently**, where the model — not the tool — decides relevance from the text, exactly the same shape of mechanism as description-matching skills elsewhere; `globs` alone drives **Apply to Specific Files**, a mechanical path-pattern match the tool itself computes (`**/*.ts`, `src/**`, etc.); and omitting both drives **Apply Manually**, triggered only by an explicit `@rule-name` mention in chat. Of these four, two — `alwaysApply: true` and the `globs` match — are deterministic checks the tool itself executes, deciding inclusion without the model. `@`-mention is deterministic as well, but it is an explicit user action rather than anything the tool infers. Only the description-driven path hands the relevance judgment to the model, and the documentation itself points users at a manual troubleshooting checklist — "Check the rule type. For `Apply Intelligently`, ensure a description is defined. For `Apply to Specific Files`, ensure the file pattern matches referenced files." — rather than any automated confirmation.

Overlap resolution is documented at the tier level — Team/Project/User merge in a stated order, and nested `AGENTS.md` files merge parent-to-child — but the documentation is silent on the finer case of two same-tier Project Rules whose `globs` both match one file; no order or arbitration is described for that case, so the reconciliation is presumed to be simple concatenation, not a resolved fact.

## Rubric Scores

Version examined: 3.14.7 (installed Cursor.app, via `plutil -p .../Info.plist`)

| Axis | Score | Justification |
|------|-------|---------------|
| `discoverability` | 3 | Two of the four documented apply-mechanisms (`alwaysApply`, `globs`) are mechanical, tool-executed checks rather than model judgment, and a third (manual `@`-mention) is deterministic but user-initiated rather than tool-inferred; only the description-driven "Apply Intelligently" path is left to unverified model judgment. Under the ceiling read stated below, the tool-computed `globs` path sets the score. |
| `context-budget` | 2 | Loading is staged by the same three/four mechanisms (not everything is always resident), and a "keep rules under 500 lines" guideline is documented, but neither a rule count nor a context-token cap is enforced anywhere in the docs. |
| `composition` | 2 | Cross-scope order (Team → Project → User) and nested-`AGENTS.md` parent/child precedence are explicitly documented, but the docs are silent on arbitration between two same-tier rules whose globs both match one file — a documented convention with a confirmed, undocumented gap. |
| `state` | 2 | Each rule tier has a defined, documented persistence location (git-tracked `.mdc`/`AGENTS.md`, local app settings for User Rules, dashboard storage for Team Rules with an admin-lock), but no enforced size cap or write-time integrity marker (e.g. a stamped timestamp) is documented for any of them. |
| `side-effect-control` | 3 | Terminal commands, file-config edits, and MCP connections/tool-calls all require approval by default, Run Modes add an OS-level sandbox (macOS Seatbelt / Linux Landlock+seccomp) plus a classifier, and a `permissions.json` allowlist demonstrably overrides and read-only-locks the in-app UI — enforced at multiple layers, despite the security overview docs' own "best-effort guardrails rather than a hard security boundary" caveat. |
| `observability` | 2 | MCP has a concrete, documented diagnostic surface (Output panel → "MCP Logs", showing "server initialization, tool calls, and error messages", plus chat-level failure marking); rules have only a manual troubleshooting checklist, and the general Agent Security docs name no logging/diagnostics surface at all. |

`discoverability` is scored on a ceiling read (the strongest documented mechanism — tool-computed `globs`/`alwaysApply`/manual-mention — sets the score, even though the fourth, description-driven path remains model-judged) while `context-budget`, `composition`, `state`, and `observability` are scored on a floor read (one confirmed, documented gap caps the score even where other parts of that same axis are enforced); the asymmetry is deliberate rather than an inconsistency, because `discoverability` asks whether an agent has *any* enforced way to learn relevance, whereas the other four ask how completely the tool's enforcement covers the mechanism's full scope — a question a single confirmed gap genuinely answers "no, not completely" for.

## Portability

Cursor's clearest lesson for a Claude Code plugin author is deterministic, glob-based auto-attach as a companion to description matching: a skill that mechanically checks file globs needs no model judgment call for the common "does this apply to the file I'm touching" case. Its two-location, concatenated `permissions.json` (user + workspace, with a documented precedence and a UI that locks read-only once a file-based allowlist is set) is a reusable pattern for layering permission config. Its "Output panel → dedicated log channel" is a concrete, low-effort observability primitive worth copying for MCP-heavy plugins.

## See Also

- [[deferred-reference-loading|Deferred Reference Loading]] ([Deferred Reference Loading](../concepts/deferred-reference-loading.md)) — technique sighted here (`@filename.ts` deferred file reference inside a rule body)
- [[path-scoped-activation|Path-Scoped Activation]] ([Path-Scoped Activation](../concepts/path-scoped-activation.md)) — technique sighted here (`globs`-driven "Apply to Specific Files")
- [[specificity-ordered-precedence|Specificity-Ordered Precedence]] ([Specificity-Ordered Precedence](../concepts/specificity-ordered-precedence.md)) — technique sighted here (nested `AGENTS.md` child-wins precedence)

## Sources

- [Cursor Extension Model](../../raw/notes/2026-08-04-cursor-extension-model.md) — rule format, scoping fields, MCP surface
