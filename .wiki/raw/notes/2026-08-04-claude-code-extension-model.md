---
title: "Claude Code Extension Model"
source: "MANUAL"
type: notes
ingested: 2026-08-04
tags: [claude-code, extension-unit, mechanism]
summary: "Evidence on Claude Code's extension units, their loading path, and its permission and hook surface. Collected from the local installation and official documentation."
---

# Claude Code Extension Model

## Version Examined

```
2.1.221 (Claude Code)
```

Output of `claude --version`, run on this machine on 2026-08-04.

## Extension Units

### Skills

Declared by a `SKILL.md` file with YAML frontmatter. Documented frontmatter fields (https://code.claude.com/docs/en/skills): `name`, `description`, `when_to_use`, `disable-model-invocation`, `user-invocable`, `allowed-tools`, `disallowed-tools`, `paths`, `context`, `background`, `arguments`. The directory name normally supplies the invocable command; `description` is what the tool keeps in context so Claude can decide whether the skill applies.

Locations observed on this machine: personal (`~/.claude/skills/`, none present), project (`.claude/skills/`, none in this repo), and — the dominant source — bundled inside installed plugins under `~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/skills/*/SKILL.md`. A `find ~/.claude/plugins/cache/*/*/*/skills/*/SKILL.md` glob matched 87 files across marketplaces `casentino`, `hwahae-hds`, `llm-wiki`, and `ouroboros`.

Concrete frontmatter examples read directly:
- `~/.claude/plugins/cache/casentino/wf/2.26.0/skills/wf-guide/SKILL.md` — `name: wf-guide`, `description: wf 플러그인 사용법을 안내합니다. 사용자가 worktree 관련 질문을 하거나, wf 명령어를 처음 사용하거나, 어떤 명령어를 써야 할지 모를 때 트리거됩니다.`
- `~/.claude/plugins/cache/casentino/metrics/1.40.0/skills/cost-alert/SKILL.md` — `description: 비용 관련 의사결정이 필요할 때 자동 활성화됩니다. 대규모 코드 생성, 반복적 에이전트 호출, 컨텍스트 윈도우 90% 초과 시 트리거됩니다.`
- `~/.claude/plugins/cache/llm-wiki/wiki/0.16.0/skills/wiki-manager/SKILL.md` — a long `description` frontmatter field (dozens of trigger phrases) that drives ambient activation; the body defers detail to 17 files under its own `references/` directory (`ls -1 references/ | wc -l` → 17; e.g. `references/ingestion.md`, `references/linting.md`) rather than inlining everything in `SKILL.md` itself. Each workflow section in `SKILL.md` is one line plus a link, e.g. `### Ingestion\nSee [references/ingestion.md](references/ingestion.md).`

Trigger: the tool keeps skill descriptions in context at session start; Claude compares the current task against them and loads the full body only when it decides a skill applies, or the user explicitly types `/name`. Per official docs (https://code.claude.com/docs/en/skills): "Every skill needs a SKILL.md file with two parts: YAML frontmatter between --- markers that tells Claude when to use the skill, and markdown content with the instructions Claude follows when the skill runs. The directory name becomes the command you type, and the description helps Claude decide when to load the skill automatically."

**Same-name precedence (composition).** The same docs page states an explicit, deterministic conflict-resolution order for skills that share a name across scopes: "When skills share the same name across levels, enterprise overrides personal, and personal overrides project. A skill at any of these levels also overrides a bundled skill with the same name. For example, a `code-review` skill in your project's `.claude/skills/` replaces the bundled `/code-review`. Plugin skills use a `plugin-name:skill-name` namespace, so they cannot conflict with other levels. If you have files in `.claude/commands/`, those work the same way, but if a skill and a command share the same name, the skill takes precedence." (https://code.claude.com/docs/en/skills). This is a tool-enforced rule, not a convention Claude follows by choice — the precedence is fixed (enterprise > personal > project > bundled) and plugin skills are namespaced (`plugin-name:skill-name`) so they cannot collide with any other level at all. What this does *not* cover: two differently-named extensions (e.g. two skills, or two subagents) whose descriptions both plausibly match the same prompt. No source found — local or documented — describes a tool-enforced tie-break for that case; which one (if either) gets used is model judgment. That narrower gap remains an inference from absence, not a confirmed one.

### Plugins

Declared by a `.claude-plugin/plugin.json` manifest with `name`, `description`, `version`, and pointers to `commands`, `skills`, and (optionally) a `hooks` object. Read directly:

- `~/.claude/plugins/cache/casentino/history/1.17.1/.claude-plugin/plugin.json`:
  ```json
  {"name": "history", "description": "Development history — snapshot, Jira ticket update, Notion career history. Depends on: metrics (optional, for obsidian git sync)", "version": "1.17.1", "commands": ["./commands/history/"], "skills": ["./skills/"]}
  ```
- `~/.claude/plugins/cache/casentino/metrics/1.40.0/.claude-plugin/plugin.json` additionally declares a `hooks` object keyed by event name (`UserPromptSubmit`, `Stop`, `PreCompact`, `PostToolUse`, `SessionStart`, `SessionEnd`), each an array of `{"type": "command", "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/<script>.sh"}` entries.
- `~/.claude/plugins/cache/casentino/wf/2.26.0/.claude-plugin/plugin.json` declares `SessionStart` and `SessionEnd` hook arrays for worktree-context capture.

Plugin marketplaces are registered separately and enumerate available plugins:
- `~/.claude/plugins/marketplaces/casentino/.claude-plugin/marketplace.json` — `{"name": "casentino", "plugins": [{"name": "metrics", "source": "./plugins/metrics", "description": "...", "version": "1.40.0"}, ...]}`.
- `~/.claude/plugins/marketplaces/team-attention-plugins/.claude-plugin/marketplace.json` — same shape, third-party marketplace.

Whether an installed plugin is active is a per-user toggle: `~/.claude/settings.json` → `enabledPlugins`, e.g. `"wiki@llm-wiki": true`, `"ouroboros@ouroboros": false`, `"hds-figma-design@hwahae-hds": false`. Enabling a plugin merges its skills/commands/hooks into the session (per docs, see Loading section).

### Hooks

Declared by a `hooks` object keyed by lifecycle event name, either inline in a settings file (`~/.claude/settings.json`, `.claude/settings.json`) or inside a plugin's `plugin.json`. Each event maps to an array of `{matcher, hooks: [{type: "command", command}]}` entries.

Local example, `~/.claude/settings.json`:
```json
"hooks": {
  "PreToolUse": [
    {"matcher": "Bash", "hooks": [{"type": "command", "command": "rtk hook claude"}]}
  ]
}
```
This is a real, currently-active hook: every `Bash` tool call on this machine is first rewritten by `rtk hook claude` (per `~/.claude/RTK.md`, which documents `rtk` as a token-optimized CLI proxy).

Plugin-declared hooks observed: `metrics` (1.40.0) wires `UserPromptSubmit`, `Stop`, `PreCompact`, `PostToolUse`, `SessionStart`, `SessionEnd`; `wf` (2.26.0) wires `SessionStart`/`SessionEnd`. Script paths only are cited here (not copied): `~/.claude/plugins/cache/casentino/metrics/1.40.0/hooks/*.sh` and `~/.claude/plugins/cache/casentino/wf/2.26.0/hooks/*.sh`.

Per official docs (https://code.claude.com/docs/en/hooks), the documented event surface is large; core events confirmed both by local usage and by doc quotes: `SessionStart`, `SessionEnd`, `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `Stop`, `PreCompact`. A WebFetch summary additionally listed further events (`SubagentStart`/`SubagentStop`, `PostCompact`, `Notification`, `PermissionRequest`, `PermissionDenied`, and others); these are plausible for v2.1.221 but were not independently re-verified with direct quotes here, so they are noted as **likely but unconfirmed by direct quote** rather than settled fact.

### Subagents

Declared by a markdown file with YAML frontmatter under `~/.claude/agents/` (personal scope — the only scope this task was asked to inspect). Observed frontmatter fields: `name`, `description`, `tools`, `model`, `permissionMode`, `maxTurns`, `skills`.

- `~/.claude/agents/plan-validator.md`: `name: plan-validator`, `description: Pre-dispatch validator for implementation plans...`, `tools: Read, Grep, Glob, Bash`, `model: haiku`.
- `~/.claude/agents/prompt-analyzer.md`: `tools: [Read, Grep, Glob]`, `permissionMode: plan`, `maxTurns: 4`, `skills: [prompt-optimizer]`.
- `~/.claude/agents/prompt-scorer.md`: same shape, `maxTurns: 3`.

Trigger: a `description` field the orchestrating agent reads to decide whether to dispatch that subagent for a task, plus explicit dispatch via the Task/Agent tool. `tools`/`skills` frontmatter scope what is available inside the subagent's own context — a distinct budget from the parent conversation.

### MCP servers

Declared by an `mcpServers` object, either globally in `~/.claude.json` or per-project in a `.mcp.json` file (none exists in this repository: `find ... -maxdepth 1 -iname ".mcp.json"` returned nothing).

Global config read from `~/.claude.json` (`mcpServers` key), values other than secrets shown:
- `sequential-thinking` → `{"type": "stdio", "command": "npx", "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]}`
- `context7` → `{"type": "stdio", "command": "npx", "args": ["-y", "@upstash/context7-mcp"]}`
- `figma-desktop` → `{"type": "http", "url": "http://127.0.0.1:3845/mcp"}`
- `airbridge` → `{"type": "http", "url": "https://mcp.airbridge.io/mcp"}`

MCP servers register at session start; their tools enter the model's tool list namespaced as `mcp__<server>__<tool>` (seen in this session's own deferred-tool list, e.g. `mcp__plugin_context7_context7__query-docs`).

**Path-scoped skill activation (`paths` frontmatter, verified post-review against primary source).** The skills reference page documents a `paths` field not elaborated on in the earlier pass above: "`paths` | No | Glob patterns that limit when this skill is activated. Accepts a comma-separated string or a YAML list. When set, Claude loads the skill automatically only when working with files matching the patterns. Uses the same format as [path-specific rules](/docs/en/memory#path-specific-rules)." (https://code.claude.com/docs/en/skills, fetched 2026-08-05 to confirm this field's exact mechanics after the first pass only listed its name). This is a mechanical, tool-computed path/glob match gating automatic activation — distinct from, and layered on top of, the model-judged `description` relevance check.

### Memory files (CLAUDE.md, `.claude/rules/`, auto memory)

Declared by plain markdown at several scopes. Per https://code.claude.com/docs/en/memory, in load order (broadest to most specific): Managed policy (`/Library/Application Support/ClaudeCode/CLAUDE.md` etc.), User (`~/.claude/CLAUDE.md`), Project (`./CLAUDE.md` or `./.claude/CLAUDE.md`), Local (`./CLAUDE.local.md`).

Confirmed locally:
- `~/.claude/CLAUDE.md` exists (2711 bytes) and contains, at line 44, a bare import: `@RTK.md` (confirmed via `grep -n "^@" ~/.claude/CLAUDE.md`), which the docs say is expanded and loaded at launch, recursively up to 4 hops.
- `~/.claude/rules/*.md` — six files (`coding-style.md`, `git-workflow.md`, `testing.md`, `performance.md`, `patterns.md`, `hooks.md`, `agents.md`, `security.md` — actually eight) present with no `paths:` frontmatter, so per docs they load unconditionally at every session start alongside `CLAUDE.md`, at the same priority.
- `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/CLAUDE.md` — a project-level memory file, present in this repo and read this session.
- No `~/.claude/settings.local.json`-scoped `CLAUDE.local.md` was found in this repo.

Docs quotes: "CLAUDE.md files can import additional files using `@path/to/import` syntax... Imported files can recursively import other files, with a maximum depth of four hops." / "User-level rules are loaded before project rules, giving project rules higher priority." Auto memory (a separate, tool-written mechanism) stores a `MEMORY.md` index plus topic files under `~/.claude/projects/<project>/memory/`; only "the first 200 lines of `MEMORY.md`, or the first 25KB, whichever comes first" load at session start, and the tool actively enforces this: "If the file is over a limit, the write still succeeds, but Claude Code returns an error telling Claude to rewrite the index."

**Path-specific rules (`.claude/rules/*.md`, verified post-review against primary source).** "Rules can be scoped to specific files using YAML frontmatter with the `paths` field. These conditional rules only apply when Claude is working with files matching the specified patterns... Path-scoped rules trigger when Claude reads files matching the pattern, not on every tool use." Example glob patterns given verbatim: `**/*.ts`, `src/**/*`, `*.md`, `src/components/*.tsx`. "Rules without a `paths` field are loaded unconditionally and apply to all files." (https://code.claude.com/docs/en/memory, fetched 2026-08-05). This confirms the eight unconditional rule files observed locally (`~/.claude/rules/*.md`) are unconditional specifically because none sets `paths` — the mechanism for scoping them exists and is documented, but is unused on this machine.

**Composition gap for concatenated instruction files, confirmed by direct quote (verified post-review).** Unlike same-name skill precedence (a tool-enforced order, above), CLAUDE.md/rules files that merely *overlap in applicability* without sharing a name have no tool-enforced tie-break: "Claude treats them as context, not enforced configuration." The docs name the resulting failure mode directly, under a "Consistency" heading: "if two rules contradict each other, Claude may pick one arbitrarily. Review your CLAUDE.md files, nested CLAUDE.md files in subdirectories, and `.claude/rules/` periodically to remove outdated or conflicting instructions." Nested CLAUDE.md files are explicitly not override-resolved either: "All discovered files are concatenated into context rather than overriding each other." (https://code.claude.com/docs/en/memory, fetched 2026-08-05). This contrasts with Codex CLI's and Cursor's documented closer/child-file-wins conventions for their own nested instruction files (see those tools' notes) — Claude Code concatenates and leaves conflicts to model judgment where those two tools state a deterministic specificity order.

## Loading and Injection

- **CLAUDE.md / rules**: "Each Claude Code session begins with a fresh context window... Both [CLAUDE.md and auto memory] are loaded at the start of every conversation." All discovered CLAUDE.md/CLAUDE.local.md files along the directory tree from filesystem root down to the working directory are concatenated (not override-merged) into context at launch; nested subdirectory CLAUDE.md files load lazily "when Claude reads files in those subdirectories." (https://code.claude.com/docs/en/memory)
- **Skills**: "In a regular session, skill descriptions are loaded into context so Claude knows what's available, but full skill content only loads when invoked." Nested `.claude/skills/` below the start directory load "the first time Claude reads or edits a file inside that subdirectory." When invoked, "the rendered `SKILL.md` content enters the conversation as a single message and stays there for the rest of the session"; re-invocation with identical content is deduplicated rather than re-appended. Under compaction, "Claude Code re-attaches the most recent invocation of each skill after the summary, keeping the first 5,000 tokens of each," shared across a 25,000-token budget. (https://code.claude.com/docs/en/skills)
- **Hooks**: fire synchronously at the named lifecycle event. The docs state the execution and dedup rules as three consecutive sentences, quoted separately here: "All matching hooks run in parallel." / "If you define the same handler in more than one settings file, it runs once." / "A plugin's or skill's copy of the same handler stays separate." Hook entries from different scopes (user/project/local/plugin) "merge across settings levels rather than replacing each other." (https://code.claude.com/docs/en/hooks). **Correction (post-review, 2026-08-05):** an earlier draft rendered the first two of those sentences as one quotation, "All matching hooks run in parallel, and identical handlers are deduplicated automatically," which is not text on the page and over-generalized the dedup rule — deduplication applies to the *same handler defined in more than one settings file*, and explicitly does **not** collapse a plugin's or skill's copy of that same handler, which stays separate and runs on its own.
- **MCP servers**: register at session start; tool schemas enter the model's tool list the same way built-in tools do.
- **Subagents**: dispatched mid-session by the Agent/Task tool; a fresh subagent gets a new context window with only its own frontmatter-scoped tools/skills, except a `fork`, which "inherits the parent conversation and system prompt" (docs, memory page, re: auto memory and forks).
- **Auto memory**: `MEMORY.md` (bounded to 200 lines/25KB) loads at every session start; topic files referenced from it load on demand via normal file-read tools.

## Permission and Hook Surface

**Permission modes** (https://code.claude.com/docs/en/permission-modes), selectable via `permissions.defaultMode` in a settings file or `--permission-mode`:

| Mode | What runs without asking |
|------|---------------------------|
| `default` (labeled "Manual") | Reads only |
| `acceptEdits` | Reads, file edits, and common filesystem Bash commands (`mkdir`, `touch`, `mv`, `cp`, `rm`, `rmdir`, `sed`) inside the working directory |
| `plan` | Reads, plus classifier-approved commands when auto mode is available |
| `auto` | Everything, gated by a background classifier model |
| `dontAsk` | Only pre-approved (`permissions.allow`) tools and read-only Bash |
| `bypassPermissions` | Everything, including writes to protected paths |

This machine's own `~/.claude/settings.json` sets `"permissions": {"defaultMode": "acceptEdits"}`.

**Rule syntax**: `permissions.allow` / `permissions.deny` / `permissions.ask` arrays of `ToolName(pattern)` strings (e.g. `"Bash(git commit *)"`, `"Read(//Users/.../fe-monorepo-core/**)"` — both present in the local `~/.claude/settings.json` `allow` array). Rules merge across scopes rather than overriding; a `deny` match always wins over an `allow` match. **Settings precedence**, highest to lowest: Managed (cannot be overridden) → command-line arguments → Local (`.claude/settings.local.json`) → Project (`.claude/settings.json`) → User (`~/.claude/settings.json`).

**Protected paths** (writes never auto-approved except in `bypassPermissions`, or in planning sessions with bypass available): a fixed list including `.git`, `.claude` (except `.claude/worktrees`), `.mcp.json`, `.claude.json`, shell rc files, etc. — enforced *before* `permissions.allow` rules are even evaluated, so an `Edit(.claude/**)` allow rule does not bypass the prompt.

**Auto mode** layers a separate classifier model (Sonnet 5 by default) in front of every non-read, non-working-directory action; it blocks categories such as `curl | bash`, force-push, `terraform destroy`, secret-manager writes, etc., by default, and falls back to prompting "If the classifier blocks an action 3 times in a row or 20 times total."

**Hook events** confirmed active on this machine via plugin manifests: `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop`, `PreCompact`, `SessionStart`, `SessionEnd`. Hooks can block (`decision: "block"`), inject `additionalContext`, or (for `PreToolUse`) return a `permissionDecision`.

## Observability

- **Transcripts**: each session's JSONL history is written under `~/.claude/projects/<encoded-project-path>/*.jsonl`. Confirmed by directory listing of `~/.claude/projects/` (e.g. `-Users-kikyeongoh--claude-plugins-cache-superpowers-marketplace-episodic-memory-1-0-15`, an encoded absolute path). Docs state writes to these transcript files are themselves blocked-by-default under auto mode: "A transcript is session state that Claude Code writes, not a working file, and a tampered entry reaches every later check once you resume the session, so auto mode blocks these writes as defense in depth."
- **Slash commands for introspection**: `/context` ("check the list under **Memory files** to verify your CLAUDE.md and CLAUDE.local.md files loaded"), `/status` (login/mode state), `/memory` (browse/edit CLAUDE.md, rules, and auto-memory files), `/permissions` (a "Recently denied" tab where a blocked auto-mode action can be retried), `/doctor` (checkup, including a CLAUDE.md trim proposal).
- **Auto memory provenance**: Claude Code stamps a `modified` ISO-8601 field into memory-file frontmatter on write, so both the user and Claude can see how current a saved fact is.
- **Hook failures**: a hook is a shell command; its exit code and stdout/stderr are the diagnostic surface (no separate hook-failure log was found locally or in the fetched docs beyond this).
- **Third-party observability built on the primitives above**: the owner's own `metrics` plugin (not copied into this wiki; path only) wires `SessionStart`, `PostToolUse`, `Stop`, `PreCompact`, `SessionEnd`, and `UserPromptSubmit` hooks to accumulate token/cost data and render it via a custom statusline (`~/.claude/settings.json` → `"statusLine": {"command": "bash /Users/kikyeongoh/.claude/metrics/statusline-wrapper.sh"}`) and a `metrics:dashboard` skill. This shows Claude Code's own observability primitives are the hook/transcript surface; richer dashboards are plugin-built on top rather than native to the CLI.

## Sources

- `claude --version` (executed locally 2026-08-04) — established the installed version, `2.1.221 (Claude Code)`.
- `~/.claude/plugins/cache/*/*/*/skills/*/SKILL.md` (87 files, globbed) — established skill frontmatter shape and volume of installed skills.
- `~/.claude/plugins/cache/llm-wiki/wiki/0.16.0/skills/wiki-manager/SKILL.md` and its `references/` directory — established the progressive-disclosure pattern (short SKILL.md body, detail deferred to reference files).
- `~/.claude/plugins/cache/casentino/history/1.17.1/.claude-plugin/plugin.json`, `.../metrics/1.40.0/.claude-plugin/plugin.json`, `.../wf/2.26.0/.claude-plugin/plugin.json` — established the plugin manifest schema and hook declarations.
- `~/.claude/plugins/marketplaces/casentino/.claude-plugin/marketplace.json`, `~/.claude/plugins/marketplaces/team-attention-plugins/.claude-plugin/marketplace.json` — established the marketplace manifest schema.
- `~/.claude/settings.json` — established live hook config (`PreToolUse`/`rtk hook claude`), `permissions.defaultMode: "acceptEdits"`, `allow` rule syntax, `enabledPlugins` map, and statusline hook.
- `~/.claude/settings.local.json` — established local-scope permission overrides (`WebFetch(domain:...)`, `WebSearch`).
- `~/.claude/agents/plan-validator.md`, `prompt-analyzer.md`, `prompt-scorer.md` — established subagent frontmatter fields.
- `~/.claude.json` (`mcpServers` key) — established global MCP server registration and transport types.
- `~/.claude/CLAUDE.md` (and its `@RTK.md` import) and `~/.claude/rules/*.md` — established the memory-file import mechanism and unconditional rule loading.
- `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/CLAUDE.md` — established a project-scope memory file exists and is distinct from user-scope.
- `~/.claude/projects/` directory listing — established the transcript-storage naming convention.
- https://code.claude.com/docs/en/memory — established CLAUDE.md scope/load-order table, `@import` syntax and depth limit, auto-memory size limits and enforcement, rule-loading precedence; re-fetched 2026-08-05 (post-review) to confirm the `paths` field's glob syntax and confirm, by direct quote, that neither CLAUDE.md nor `.claude/rules/` resolves cross-file contradictions deterministically ("Claude may pick one arbitrarily").
- https://code.claude.com/docs/en/skills — established SKILL.md frontmatter fields, description-driven auto-invocation (verbatim quote), progressive disclosure, compaction behavior for invoked skills, and the enterprise > personal > project > bundled same-name precedence order plus plugin `plugin-name:skill-name` namespacing; re-fetched 2026-08-05 (post-review) to confirm the `paths` field's exact mechanics (glob-gated automatic activation).
- https://code.claude.com/docs/en/hooks — established parallel hook execution, cross-scope merge/dedup behavior, and plugin-hook merging.
- https://code.claude.com/docs/en/permission-modes — established the full permission-mode table, protected-path rules, and auto-mode classifier behavior.
- https://code.claude.com/docs/en/permissions (preview only, large page) — established the tiered permission system (read-only vs. Bash vs. file-modification approval tiers) and allow/deny rule syntax basics.
- https://code.claude.com/docs/en/settings (WebFetch summary) — established the settings-scope precedence order (managed > CLI > local > project > user).
