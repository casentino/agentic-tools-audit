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
| `discoverability` | 3 | 2 | 3 | 1 | 2 |
| `context-budget` | 3 | 3 | 2 | 2 | 1 |
| `composition` | 3 | 3 | 2 | 3 | 1 |
| `state` | 3 | 3 | 2 | 3 | 1 |
| `side-effect-control` | 3 | 3 | 3 | 2 | 1 |
| `observability` | 2 | 2 | 2 | 2 | 0 |

Copy each score from its profile. A cell that disagrees with its profile is a bug in this table, not a revision of the score.

## Reading the Spread

`discoverability` has the widest spread (2): LangGraph scores 1 because a compiled graph has no host-side moment that judges whether a node "applies now" at all — topology is fixed developer code, not something discovered — while Cursor and Claude Code score 3 because both support tool-computed mechanical activation checks (Cursor's `alwaysApply` and `globs`; Claude Code's documented `paths` glob field on both skills and rules); Codex CLI sits in between at 2, where skill relevance is a documented convention the model judges but nothing verifies. Two axes tied at spread 1 are worth naming for what they disagree about rather than how much: `composition` splits on whether conflict resolution rules exist — Cursor is capped at 2 because its hooks lack arbitration rules, while Claude Code, Codex CLI, and LangGraph each resolve overlap cases via specified precedence rules (e.g. deny-wins arbitration documented for both Codex CLI and Claude Code); and `side-effect-control` splits on default enforcement — Claude Code, Codex CLI, and Cursor all gate effects with a default-on layer (permission modes, sandboxes, approval policies), while LangGraph's only gating mechanism, `interrupt()`, is opt-in code a developer must add at each call site, with no default-deny anywhere in the framework.

## Where the Rubric Strained

`observability` never discriminates: every tool scores 2. Each has a real, documented introspection surface (Claude Code's transcripts and `/context`/`/status`/`/doctor`; Codex CLI's `codex doctor`, `RUST_LOG`, `[otel]`; Cursor's MCP "Output panel → MCP Logs"; LangGraph's `stream_mode` values), and each also has a named gap that keeps it off 3 (no dedicated hook-failure log; nothing runs by default; rules have only a manual checklist; the fullest tracing story is the separate, sign-up-gated LangSmith). Four very different tools landing on the identical score is a strong candidate for revising this axis — either its four-level scale is too coarse to separate "built-in but must be invoked" from "runs by default," or the axis is measuring something these four tools genuinely converge on and a fifth, more divergent tool is needed to test it.

These open questions were resolved in the 2026-08-15 session:
1. **Claude Code `discoverability` raised to 3:** Acknowledging the documented `paths` frontmatter field for both skills and rules as a mechanical, tool-computed activation check matching Cursor's `alwaysApply`/`globs` mechanism.
2. **Codex CLI `composition` raised to 3:** Resolving the 3-vs-2 split by recognizing Codex CLI's documented deny-wins hook arbitration rules ("If multiple matching hooks return decisions, any `deny` wins. Otherwise, an `allow` lets the request proceed...") as functional equivalents to Claude Code's precedence rules.
3. **`observability` confirmed at 2:** Deferring any change to this axis until a fifth tool can be profiled to provide sufficient comparative spread.

The rubric (`.wiki/schema.md`, Score Scale) never states how to score an axis whose several mechanisms have mixed enforcement, and only two of the four profiles say which read they used. Cursor's Rubric Scores section states its convention directly: `discoverability` gets a ceiling read ("does *any* enforced path exist"), the other five get a floor read ("one confirmed, documented gap caps the score even where other parts of that same axis are enforced"). LangGraph's section states the identical convention by name ("Reads used, stated per this series' aggregation convention... discoverability ceiling, the other five floor"). Claude Code's and Codex CLI's Rubric Scores sections contain no comparable sentence anywhere — no use of "ceiling," "floor," or "aggregation" in either file — because both were profiled before the convention was named mid-series. Reading their five non-`discoverability` justifications for implicit method, both are consistent with the floor-read-with-*confirmed*-gap convention Cursor later stated in words: Claude Code's `composition` explicitly distinguishes a merely *inferred* absence ("still an inference from absence rather than a confirmed one") from a confirmed gap and scores it 3 rather than capping it, the same distinction Cursor draws by contrast in scoring its own `composition` a 2 for a gap it calls "confirmed"; and Codex CLI's `composition` (now 3) and `observability` (2) each cap on a plainly confirmed, stated gap (such as a lack of default verification for observability), matching the pattern.

The seed set does not exercise the rubric's low end the way the plan intended. The plan picked OpenAI Codex CLI to anchor the scale's floor on the stated grounds that its extension surface was "deliberately thin." Research falsified that: Codex CLI scores 2/3/2/3/3/2, matching or beating Cursor's 3/2/2/2/3/2 on five of six axes (`context-budget` 3>2, `composition` 2=2, `state` 3>2, `side-effect-control` 3=3, `observability` 2=2 — it loses only on `discoverability`, 2<3) and tying Claude Code's 3s on `context-budget`, `state`, and `side-effect-control`. No cell in the entire 4×6 matrix reads `0` ("no such mechanism") — the floor the scale defines is never touched by any of the four tools profiled. The one `1` in the matrix, LangGraph's `discoverability`, did not come from thinness at all; LangGraph's own profile attributes it to a structural mismatch — a compiled graph has no host-side relevance-judgment moment for the rubric to score, not an absent or weak version of one. So the low end this scoreboard actually has came from a library whose extension unit is ordinary application code, not from the CLI the plan expected to be thin. Before a fifth tool is profiled, it's worth choosing one that can plausibly land a genuine `0` on at least one axis, or accepting that this rubric's four levels compress everything above "no mechanism" into a narrow 1–3 band that this seed set has barely spread.

## See Also

- [[claude-code|Claude Code]] ([Claude Code](../topics/claude-code.md)) — the porting target
- [[codex-cli|OpenAI Codex CLI]] ([OpenAI Codex CLI](../topics/codex-cli.md)) — a second CLI, scoring high across the board
- [[cursor|Cursor]] ([Cursor](../topics/cursor.md)) — glob-scoped rules
- [[langgraph|LangGraph]] ([LangGraph](../topics/langgraph.md)) — first-class persistence

## Sources

- [Claude Code Extension Model](../../raw/notes/2026-08-04-claude-code-extension-model.md) — Claude Code scores
- [OpenAI Codex CLI Extension Model](../../raw/notes/2026-08-04-codex-cli-extension-model.md) — Codex CLI scores
- [Cursor Extension Model](../../raw/notes/2026-08-04-cursor-extension-model.md) — Cursor scores
- [LangGraph Extension Model](../../raw/notes/2026-08-04-langgraph-extension-model.md) — LangGraph scores
