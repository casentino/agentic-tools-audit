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
| `discoverability` | 2 | 2 | 3 | 1 | 2 |
| `context-budget` | 3 | 3 | 2 | 2 | 1 |
| `composition` | 3 | 2 | 2 | 3 | 1 |
| `state` | 3 | 3 | 2 | 3 | 1 |
| `side-effect-control` | 3 | 3 | 3 | 2 | 1 |
| `observability` | 2 | 2 | 2 | 2 | 0 |

Copy each score from its profile. A cell that disagrees with its profile is a bug in this table, not a revision of the score.

## Reading the Spread

`discoverability` has the widest spread (2): LangGraph scores 1 because a compiled graph has no host-side moment that judges whether a node "applies now" at all — topology is fixed developer code, not something discovered — while Cursor scores 3 because two of its four apply-mechanisms (`alwaysApply`, `globs`) are mechanical, tool-executed checks and a third (manual `@`-mention) is deterministic but user-initiated rather than tool-inferred; Claude Code and Codex CLI sit in between at 2, where skill relevance is a documented convention the model judges but nothing verifies. Two axes tied at spread 1 are worth naming for what they disagree about rather than how much: `composition` splits on whether an unarbitrated overlap case is confirmed to exist — Codex CLI's hooks explicitly run as an unarbitrated union ("higher-precedence config layers don't replace lower-precedence hooks"), capping it at 2, while Claude Code and LangGraph each found every collision case they checked resolved by a specified rule; and `side-effect-control` splits on default enforcement — Claude Code, Codex CLI, and Cursor all gate effects with a default-on layer (permission modes, sandboxes, approval policies), while LangGraph's only gating mechanism, `interrupt()`, is opt-in code a developer must add at each call site, with no default-deny anywhere in the framework.

## Where the Rubric Strained

`observability` never discriminates: every tool scores 2. Each has a real, documented introspection surface (Claude Code's transcripts and `/context`/`/status`/`/doctor`; Codex CLI's `codex doctor`, `RUST_LOG`, `[otel]`; Cursor's MCP "Output panel → MCP Logs"; LangGraph's `stream_mode` values), and each also has a named gap that keeps it off 3 (no dedicated hook-failure log; nothing runs by default; rules have only a manual checklist; the fullest tracing story is the separate, sign-up-gated LangSmith). Four very different tools landing on the identical score is a strong candidate for revising this axis — either its four-level scale is too coarse to separate "built-in but must be invoked" from "runs by default," or the axis is measuring something these four tools genuinely converge on and a fifth, more divergent tool is needed to test it.

The rubric (`.wiki/schema.md`, Score Scale) never states how to score an axis whose several mechanisms have mixed enforcement, and only two of the four profiles say which read they used. Cursor's Rubric Scores section states its convention directly: `discoverability` gets a ceiling read ("does *any* enforced path exist"), the other five get a floor read ("one confirmed, documented gap caps the score even where other parts of that same axis are enforced"). LangGraph's section states the identical convention by name ("Reads used, stated per this series' aggregation convention... discoverability ceiling, the other five floor"). Claude Code's and Codex CLI's Rubric Scores sections contain no comparable sentence anywhere — no use of "ceiling," "floor," or "aggregation" in either file — because both were profiled before the convention was named mid-series. Reading their five non-`discoverability` justifications for implicit method, both are consistent with the floor-read-with-*confirmed*-gap convention Cursor later stated in words: Claude Code's `composition` explicitly distinguishes a merely *inferred* absence ("still an inference from absence rather than a confirmed one") from a confirmed gap and scores it 3 rather than capping it, the same distinction Cursor draws by contrast in scoring its own `composition` a 2 for a gap it calls "confirmed"; and Codex CLI's `composition` (2) and `observability` (2) each cap on a plainly confirmed, stated gap (unarbitrated hooks; nothing verifies by default), matching the pattern exactly. `discoverability` is the one axis where the two undated profiles cannot be confirmed to follow the stated convention, and Claude Code is the clearer case of possible drift: its justification describes hooks as matching "mechanically on event+matcher" — language that reads as an enforced path, which under Cursor's stated ceiling rule ("does any enforced path exist") would argue for a 3 — yet the axis is scored 2, with hooks characterized as "the exception" rather than as the mechanism that sets the ceiling. Codex CLI's `discoverability` is not a useful test case either way, because its text explicitly excludes hooks and `AGENTS.md` from counting as discovery at all ("they fire/load unconditionally"), leaving only one candidate mechanism — so ceiling and floor reads would agree by default. This is reported as an open finding, not a fixed one: it cannot be told from the text alone whether Claude Code's `discoverability` would move under the now-stated convention, only that the convention itself was unstated when that score was set, and an unstated derivation rule is exactly what makes a comparison table like this one hard to trust at the cell level.

A second, related tension surfaced after this scoreboard was written, during pattern promotion (Task 8), and is recorded here rather than left silent in a downstream file: Claude Code's own `paths` frontmatter field, documented for both skills and `.claude/rules/*.md` files as a glob-gated, tool-computed activation check (`raw/notes/2026-08-04-claude-code-extension-model.md`, the "Path-scoped skill activation" paragraph at line 94 and the "Path-specific rules" paragraph at line 108, both added post-review against the primary docs), is the same class of mechanical match this scoreboard credits for Cursor's `discoverability` = 3. That field existed at the time Claude Code was profiled but is not cited anywhere in `claude-code.md`'s `discoverability` row, which names only hooks as the mechanical exception to model-judged relevance. This is reported as a finding, not a verdict: whether accounting for `paths` would move Claude Code's `discoverability` score, or whether it changes nothing because the field is undocumented as to how consistently it's applied across the owner's own skills, is a question for whoever next revisits that profile — this task promotes patterns from existing scores, it does not re-score them.

A third tension, in the same "open finding, not a fixed one" spirit as the `paths` note above, sits in the `composition` axis's 3-vs-2 split between Claude Code and Codex CLI. Codex CLI is capped at `2` on the stated grounds that its hooks are "the documented counterexample" to tool-enforced composition — "all matching hooks just run concurrently with no arbitration" (`wiki/topics/codex-cli.md`, `composition` row). But Codex CLI's own hook documentation states a deny-wins arbitration rule for exactly this case: "If multiple matching hooks return decisions, any `deny` wins. Otherwise, an `allow` lets the request proceed…" and, for `Stop` hooks specifically, "If any matching `Stop` hook returns `continue: false`, that takes precedence over continuation decisions from other matching `Stop` hooks" (learn.chatgpt.com/docs/hooks, fetched 2026-08-05). Claude Code, scored `3` on the same axis, documents the same underlying shape for its own hooks — union execution ("All matching hooks run in parallel") reconciled by a stated precedence order rather than by deduplication: "When multiple PreToolUse hooks return different decisions, precedence is `deny` > `defer` > `ask` > `allow`," and a hook's `continue: false` "[t]akes precedence over any event-specific decision fields" (code.claude.com/docs/en/hooks, fetched 2026-08-05). So both tools document the same two things for overlapping hooks: every matching hook runs (a union, not a precedence-ordered subset), and conflicting decisions are reconciled by a stated deny-wins rule. The 3-vs-2 split does not rest on the difference either profile's `composition` justification names — Codex CLI's justification calls its union "no arbitration," but its own docs describe arbitration for the conflict case that would matter. This is recorded as an open finding, not resolved here: whether either score should move, and if so which way, is a re-score decision for whoever next revisits the rubric. No score is changed by this paragraph.

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
