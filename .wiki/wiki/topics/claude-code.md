---
title: "Claude Code"
category: topic
sources: ["raw/notes/2026-08-04-claude-code-extension-model.md"]
created: 2026-08-04
updated: 2026-08-04
tags: [claude-code, agentic-cli, extension-unit]
aliases: ["Claude Code CLI"]
confidence: high
volatility: hot
verified: 2026-08-04
summary: "Claude Code's extension model and its scores across the six audit axes."
---

# Claude Code

> Claude Code treats an extension as one of several typed units — skill, plugin, hook, subagent, MCP server, or memory file — each with its own frontmatter/manifest, its own trigger, and its own place in context; the single most distinctive thing about how it loads them is the split between units whose *content* is staged (skills: description always in context, body only on invocation) and units that are *mechanically* triggered regardless of the model's judgment (hooks: fire deterministically on a named lifecycle event, merged and deduplicated across scopes by the tool itself).

## Extension Model

**Unit:** Six distinct kinds — skills (`SKILL.md` + frontmatter), plugins (`.claude-plugin/plugin.json` bundling skills/commands/hooks), hooks (event-keyed shell commands), subagents (frontmatter'd markdown under `agents/`), MCP servers (`mcpServers` entries in `.claude.json`/`.mcp.json`), and memory files (`CLAUDE.md`, `.claude/rules/*.md`, and tool-written auto memory under `~/.claude/projects/<project>/memory/`).

**Loading:** Memory files and skill *descriptions* load at session start and stay in context for the whole session; skill *bodies*, subagent contexts, and MCP tool schemas load lazily — a skill body only when invoked, a subagent's context only when dispatched, nested-directory skills/rules only once a file in that directory is touched. Hooks are not "loaded" into context at all; they are registered at start and executed as external processes when their event fires.

**Injection point:** CLAUDE.md content enters as a user message after the system prompt, not inside it — "there's no guarantee of strict compliance." Skill content enters as a single message at invocation and persists for the session (deduplicated on identical re-invocation, budget-capped after compaction). Hook output can inject `additionalContext` or block the triggering action outright; it is the one unit whose effect on the agent is not mediated by "does the model choose to read this."

Plugins are the packaging layer, not a new mechanism: a plugin's `plugin.json` simply points at directories of skills/commands and declares a `hooks` object using the exact same event-keyed schema a user or project could declare directly in `settings.json`. Enabling a plugin (`enabledPlugins` in `~/.claude/settings.json`) merges its skills and hooks into the session; disabling one (several are toggled off on this machine, e.g. `"ouroboros@ouroboros": false`) removes them. Because hooks from user settings, project settings, and every enabled plugin all merge into one event table — deduplicated by command string and run in parallel — a single Claude Code session on this machine has, in practice, several plugins' `SessionStart`/`Stop`/`PreToolUse` hooks all firing on the same events without any one of them overriding another.

The permission layer is the deepest and most mechanically enforced part of the model: a small set of modes (`default`/Manual, `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions`) set a baseline, `permissions.allow`/`deny`/`ask` rules of the form `ToolName(pattern)` layer on top and merge (never override) across managed/local/project/user scopes with `deny` always winning, and a fixed list of "protected paths" (`.git`, `.claude`, shell rc files, `.mcp.json`, etc.) is checked *before* any allow rule, so no combination of settings can silently re-enable writes to the tool's own configuration.

## Rubric Scores

Version examined: 2.1.221 (Claude Code)

| Axis | Score | Justification |
|------|-------|---------------|
| `discoverability` | 2 | Skill/subagent applicability is a documented convention — the tool keeps `description` text in context and the model judges relevance (`code-claude.com/docs/en/skills`: "the `description` helps Claude decide when to load the skill automatically") — but nothing verifies the model picked the *right* one; hooks are the exception, matching mechanically on event+matcher. |
| `context-budget` | 3 | The tool actively enforces budget, not just documents it: auto memory's `MEMORY.md` is capped at "the first 200 lines... or the first 25KB" and an over-limit write triggers a Claude Code-generated error forcing a rewrite, while skills keep only descriptions resident and stage full bodies (and cap re-attachment to a 25,000-token shared budget after compaction). |
| `composition` | 2 | Hook and permission-rule merging across scopes (managed/project/user/plugin) is documented and deterministic (dedup, parallel run, deny-always-wins, managed-cannot-be-overridden), but arbitration among multiple applicable skills or subagents is left to model judgment with no tool-enforced tie-break — a documented convention overall, not a uniformly enforced one. |
| `state` | 3 | Auto memory is a first-class, tool-managed persistence mechanism with a defined location (`~/.claude/projects/<project>/memory/`), a defined entrypoint (`MEMORY.md`), enforced size limits, and a `modified` timestamp the tool itself stamps into frontmatter on write — this is enforced/verified, not merely conventional. |
| `side-effect-control` | 3 | A tiered, mode-based permission system with an explicit precedence order, a `deny`-always-wins rule engine, a hard-coded protected-path list checked ahead of allow rules, and (in `auto` mode) a separate classifier model that blocks named dangerous categories by default — this is the tool actively gating and verifying effects, the clearest 3 of the six axes. |
| `observability` | 2 | Session transcripts (`~/.claude/projects/.../*.jsonl`) and dedicated introspection commands (`/context`, `/status`, `/permissions`, `/doctor`, `/memory`) are documented, built-in surfaces, but hook failure diagnosis still relies on plain exit code/stderr with no dedicated failure log found; richer dashboards (e.g. the owner's own `metrics` plugin) are built by plugin authors on top of the hook/transcript primitives, not shipped natively. |

## Portability

Claude Code already gives a plugin author enforced permission gating, enforced state-size limits, and deterministic hook merging — a porting target does not need to invent any of those. What a plugin author still has to work around by hand: there is no tool-enforced way to arbitrate two skills/subagents whose descriptions both plausibly match a prompt (must write descriptions defensively instead), and there is no built-in structured log of *why* a hook blocked something beyond its own exit code and stderr.

## Sources

- [Claude Code Extension Model](../../raw/notes/2026-08-04-claude-code-extension-model.md) — extension units, loading order, permission surface
