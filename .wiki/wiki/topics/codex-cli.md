---
title: "OpenAI Codex CLI"
category: topic
sources: ["raw/notes/2026-08-04-codex-cli-extension-model.md"]
created: 2026-08-04
updated: 2026-08-04
tags: [codex-cli, agentic-cli, extension-unit]
aliases: ["Codex CLI"]
confidence: high
volatility: hot
verified: 2026-08-04
summary: "Codex CLI's extension model and its scores across the six audit axes."
---

# OpenAI Codex CLI

> Codex CLI treats "extension" as five separate things — `AGENTS.md` instruction files, `SKILL.md` skill packages, MCP servers, event-keyed hooks, and a marketplace-distributed plugin layer that bundles any of the previous four into one install — and the most distinctive thing about it is the split in how rigorously each is governed: the instruction/skill/hook surfaces are documented but loosely reconciled (hooks explicitly run as an unarbitrated union, "one hook can't prevent another matching hook from starting"), while the side-effect surface is unusually strict, combining an OS-level sandbox, a granular approval policy, and a rules engine that resolves conflicting command patterns by "most restrictive wins."

## Extension Model

**Unit:** Five kinds — `AGENTS.md` instruction files (plain markdown, no frontmatter), Skills (`SKILL.md` + optional `scripts/`/`references/`/`assets/`), MCP servers (`mcp_servers.<id>` tables in `config.toml`), hooks (event-keyed handlers in `hooks.json` or an inline `[hooks]` table), and plugins (a marketplace-installed bundle of any of the other four plus connectors and scheduled-task templates).

**Loading:** `AGENTS.md` files are discovered by a directory walk from the project root down to the current working directory and always read "before doing any work"; skills are discovered from up to five scopes (`$CWD/.agents/skills`, `$REPO_ROOT/.agents/skills`, `$HOME/.agents/skills`, `/etc/codex/skills`, and bundled system skills) but only their name and description stay resident; hooks are discovered by scanning fixed paths (`~/.codex/hooks.json`, `~/.codex/config.toml`, and repo-local equivalents) at every layer with no override between layers; plugins are installed from a shared ChatGPT/Codex marketplace catalog and become available "when you start a new chat or CLI session after installation."

**Injection point:** `AGENTS.md` content enters as concatenated markdown, closer files appended later so they "override earlier guidance because they appear later in the combined prompt," capped by `project_doc_max_bytes`. Skill bodies enter only once the model chooses to invoke that skill, after living as a name+description line within a documented ~2%-of-context / 8,000-character budget. Hook effects enter at the exact named lifecycle event they matched (`PreToolUse`, `PostToolUse`, `SessionStart`, etc.) as a block decision, injected `additionalContext`, or (for `PreToolUse` only) rewritten tool input — the one unit whose effect does not depend on model judgment.

`AGENTS.md` and skills are the closest thing here to Claude Code's memory-file/skill split, and the mechanics are close enough to name directly: unconditional instruction concatenation with a byte cap, versus conditional bodies gated by an always-resident description under its own budget. Where Codex diverges from a "thin" extension surface is the hook and plugin layers added on top — hooks mirror Claude Code's own event names (`SessionStart`, `PreToolUse`, `PostToolUse`, `Stop`, ...) almost exactly, but the merge policy is looser: "if more than one hook source exists, Codex loads all matching hooks. Higher-precedence config layers don't replace lower-precedence hooks" — a documented refusal to arbitrate, not an oversight report of one.

The side-effect layer is where Codex is least thin. Three sandbox modes (`read-only`, `workspace-write`, `danger-full-access`) are backed by a real OS-level sandbox invoked directly via `codex sandbox`; three approval modes (`on-request`, `never`, `untrusted`) plus a granular per-category object sit on top; and a separate rules engine (`prefix_rule`, decision `allow`/`prompt`/`forbidden`, stored in `~/.codex/rules/default.rules`) resolves overlapping rules with a verified "most restrictive result wins" — explicitly proven against a compound command (`git add . && rm -rf /`) that a naive prefix match would have let through. Hooks add their own gate on top of all this: they require a persisted "trust" before Codex will run them, bypassable only through an explicitly named `--dangerously-bypass-hook-trust` flag.

State is split cleanly into two mechanisms with different defaults. Session rollouts have a full lifecycle command surface (`resume`, `fork`, `archive`, `unarchive`, `delete`) for the literal conversation record. A separate "Memories" system background-summarizes past sessions into `~/.codex/memories/` — but it is "off by default," gated by its own `[features] memories`, `memories.generate_memories`, and `memories.use_memories` config keys, so tool-authored persistent memory is opt-in and kept apart from the record of what happened.

## Rubric Scores

Version examined: codex-cli 0.144.1 (local install; documentation examined is the `openai/codex` `main` branch and its linked hosted docs, not pinned to a tag)

| Axis | Score | Justification |
|------|-------|---------------|
| `discoverability` | 2 | Skill relevance is a documented convention — the model judges from an always-resident name+description within a fixed budget — but nothing verifies it picked the right skill; hooks and `AGENTS.md` are not "discovered" at all, they fire/load unconditionally. |
| `context-budget` | 3 | The tool enforces numeric caps at two points: `AGENTS.md` concatenation stops once `project_doc_max_bytes` is reached, and the resident skill list is capped at "approximately 2% of context window or 8,000 characters," abbreviating descriptions once that's exceeded. |
| `composition` | 2 | `AGENTS.md` has a deterministic, tool-executed override order (closer file wins) and the rules engine tool-verifiably resolves overlapping command patterns ("most restrictive result wins"), but hooks are the documented counterexample — "higher-precedence config layers don't replace lower-precedence hooks," all matching hooks just run concurrently with no arbitration. |
| `state` | 3 | Two tool-managed, documented persistence mechanisms exist with defined locations and defined config gates: session rollouts (`resume`/`fork`/`archive`/`unarchive`/`delete`) and a separate, explicitly off-by-default Memories system (`~/.codex/memories/`, `[features] memories`). |
| `side-effect-control` | 3 | Layered and tool-enforced: an OS-level sandbox with three modes invoked via a dedicated `codex sandbox` subcommand, an approval-policy layer with a granular per-category form, and a rules engine that is tool-verified to resolve compound commands correctly rather than just documented as a convention. |
| `observability` | 2 | Rich, well-documented surfaces exist — `codex doctor`, `RUST_LOG` verbosity control, an opt-in `codex-tui.log`, a native `[otel]` export section, and `/status`/`/mcp`/`/hooks`/`/feedback` introspection commands — but every one of them is something a user must invoke or configure; none runs or verifies by default. |

## Portability

Two things worth lifting directly: a numeric, enforced context-budget cap on concatenated instruction files (not just "keep it short" guidance), and a rules engine that resolves overlapping permission patterns by evaluating sub-commands separately and taking the most restrictive result, rather than trusting the first matching prefix. The hook merge policy is a cautionary counter-example — "all matching hooks run, none override" is documented but leaves real conflicts unresolved; a Claude Code plugin author gets more mileage from Claude Code's own dedup-and-precedence hook merge than from copying this one.

## Sources

- [OpenAI Codex CLI Extension Model](../../raw/notes/2026-08-04-codex-cli-extension-model.md) — instruction files, configuration, approval modes
