---
title: "Separate Plugin Event Log from Cross-Session Memory Store"
kind: task
status: proposed
priority: p3
created: 2026-08-04
updated: 2026-08-04
last_checked: 2026-08-04
next_action: "Pick one owner-authored stateful plugin (metrics or history) and document whether its storage already separates a raw per-event log from a distilled cross-session summary; if not, design that split before adding new accumulated state."
sources:
  - wiki/concepts/tiered-persistence-split.md
tags: [candidate, state]
confidence: medium
summary: "Check whether the owner's stateful plugins (metrics, history) separate their raw event log from a distilled cross-session summary, and split them if they do not."
---

# Separate Plugin Event Log from Cross-Session Memory Store

## Why Track This

The owner has at least two plugins that accumulate state across sessions by hooking lifecycle events — `metrics` (wiring `SessionStart`/`PostToolUse`/`Stop`/`PreCompact`/`SessionEnd`/`UserPromptSubmit` to accumulate token/cost data for a statusline and dashboard) and `history` (snapshot/Jira/Notion generation). Three profiled tools converge on keeping the raw record of what happened separate from a distilled, purpose-built memory store, each for the same reason: conflating the two makes the record untrustworthy and the summary slow or stale. This is worth tracking as a design check now, before either plugin's storage grows large enough that retrofitting the split becomes expensive.

## Change

For the `metrics` plugin first (it already writes on nearly every lifecycle event, making it the more time-sensitive case): read its current storage format and confirm whether raw per-event data and the distilled statusline/dashboard summary already live in structurally separate files or are computed by re-parsing one combined store on every read. If separate, no change needed — document that this pattern is already satisfied and note it. If combined, design a split: an append-only raw log the hooks write to directly, and a separate summary file the dashboard reads, updated by a distinct (possibly less frequent) distillation step — mirroring Codex CLI's choice to gate memory generation and memory use as two independently toggleable steps rather than one synchronous read-and-summarize path. Repeat the check for `history` if time allows.

## Rationale

[[tiered-persistence-split|Tiered Persistence Split]] ([Tiered Persistence Split](../../wiki/concepts/tiered-persistence-split.md))

LangGraph's checkpointer/store split, Codex CLI's session-rollout/Memories split, and Claude Code's own transcript/auto-memory split all separate "what literally happened" from "what's worth carrying forward" as two different jobs with different write disciplines. A plugin that accumulates state across many sessions (as `metrics` and `history` both do) benefits from the same separation before its combined store becomes large enough that a retrofit is costly, and gets a clearer read/write contract in the process.

## Cost

Medium: this is a design and possibly a refactor task, not a one-line config change. The investigation step (reading current storage format) is cheap; the split itself, if needed, touches the plugin's hook scripts and whatever reads them (statusline wrapper, dashboard skill), and should be tested against real accumulated data rather than a fresh install to avoid silently dropping history during migration.
