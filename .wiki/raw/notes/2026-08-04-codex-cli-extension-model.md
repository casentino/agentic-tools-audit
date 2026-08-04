---
title: "OpenAI Codex CLI Extension Model"
source: "https://github.com/openai/codex"
type: notes
ingested: 2026-08-04
tags: [codex-cli, extension-unit, mechanism]
summary: "Evidence on Codex CLI's instruction files, configuration surface, and approval modes. Collected from the repository documentation."
---

# OpenAI Codex CLI Extension Model

## Version Examined

Codex CLI is installed on this machine. `codex --version` → `codex-cli 0.144.1`. This is the version recorded.

The `openai/codex` repository's `docs/` directory (as of the `main` branch at time of research) no longer contains the substantive documentation itself — every file checked (`docs/agents_md.md`, `docs/config.md`, `docs/sandbox.md`, `docs/skills.md`, `docs/slash_commands.md`, `docs/execpolicy.md`, `docs/exec.md`, `docs/getting-started.md`) is a one-line stub pointing at a hosted docs site (`developers.openai.com/codex/...`, which 308-redirects to `learn.chatgpt.com/docs/...`). That hosted site is not pinned to the 0.144.1 tag — it documents current/`main` behavior. Where this matters, local `--help` output from the installed 0.144.1 binary is used to corroborate that a documented mechanism actually exists in the installed version (see below); no contradiction was found between the two.

## Extension Units

Four distinct kinds of extension unit are documented:

**1. `AGENTS.md` instruction files.** "Codex reads `AGENTS.md` files before doing any work." (https://learn.chatgpt.com/docs/agent-configuration/agents-md)

**2. Skills.** "A skill packages instructions, resources, and optional scripts so either product can follow a workflow reliably." Directory layout: `SKILL.md` (required, "Contains instructions and metadata"), plus optional `scripts/`, `references/`, `assets/`, and `agents/openai.yaml`. Discovery locations: `$CWD/.agents/skills` (repository-specific), `$REPO_ROOT/.agents/skills` (organization-wide repository skills), `$HOME/.agents/skills` (user-scoped), `/etc/codex/skills` (system/admin), and bundled system skills from OpenAI. (https://learn.chatgpt.com/docs/build-skills). Locally, `~/.codex/skills` exists on this machine, consistent with the documented user-scope location.

**3. MCP servers.** Declared via `mcp_servers.<id>` tables in `config.toml`. Fields: `command` ("Launcher command for an MCP stdio server"), `args` ("Arguments passed to the MCP stdio server command"), `env` ("Environment variables forwarded to the MCP stdio server"), `url` ("Endpoint for an MCP streamable HTTP server"), plus authentication, timeouts, tool-approval-mode, and tool-filtering options. (https://learn.chatgpt.com/docs/config-file/config-reference). CLI surface confirmed locally: `codex mcp {list, get, add, remove, login, logout}` and `codex mcp-server` ("Start Codex as an MCP server (stdio)") — Codex can act as an MCP server itself, not only a client. (`codex --help`, `codex mcp --help`, local install)

**4. Hooks.** "Hooks are an extensibility framework for Codex. They allow you to inject your own scripts into the agentic loop" — for "custom logging/analytics", "scanning your team's prompts to block accidentally pasting API keys", "summarizing chats to create persistent memories automatically", or "running a custom validation check when a chat turn stops." Event names: `SessionStart`, `SessionEnd`, `SubagentStart`, `SubagentStop`, `PreToolUse`, `PostToolUse`, `PermissionRequest`, `PreCompact`, `PostCompact`, `UserPromptSubmit`, `Stop`. (https://learn.chatgpt.com/docs/hooks)

**5. Plugins (bundling layer).** "[Plugins] bundle capabilities into reusable workflows in ChatGPT and Codex." A plugin can bundle: "Skills: reusable instructions for specific kinds of work", "Connectors: connections to tools like GitHub, Slack, or Google Drive", "MCP servers: services that give ChatGPT and Codex access to more tools", "Browser extensions: browser capabilities that a plugin needs", "Hooks: commands that run at configured lifecycle points", and "Scheduled task templates: reusable starting points". "ChatGPT and Codex use the same public plugin catalog" — discovery is via a marketplace/"Plugins Directory" with OpenAI, workspace, and personal tabs. "Bundled skills become available when you start a new chat or CLI session after installation." (https://learn.chatgpt.com/docs/plugins). CLI surface confirmed locally: `codex plugin {add, list, marketplace, remove}` and `codex plugin marketplace {add, list, upgrade, remove}` — `add` "Install a plugin from a configured marketplace snapshot"; `marketplace add` "Add a local or Git marketplace to the configured marketplace sources". (`codex plugin --help`, `codex plugin marketplace --help`, local install)

## Loading and Injection

**AGENTS.md merge order.** "Codex follows a hierarchical search pattern across three scopes": global (`~/.codex` or `$CODEX_HOME`, reads `AGENTS.override.md` if present, else `AGENTS.md`), then project scope — "Starting at the project root (typically the Git root), Codex walks down to your current working directory... In each directory along the path, it checks for `AGENTS.override.md`, then `AGENTS.md`, then any fallback names in `project_doc_fallback_filenames`." Combination: "Codex concatenates files from the root down, joining them with blank lines. Files closer to your current directory override earlier guidance because they appear later in the combined prompt." Stopping/cap rule: "Codex stops searching once it reaches your current directory," and "Codex skips empty files and stops adding files once the combined size reaches the limit defined by `project_doc_max_bytes`." (https://learn.chatgpt.com/docs/agent-configuration/agents-md)

**Skills — progressive disclosure.** "ChatGPT and Codex start with each skill's name and description, then load the full `SKILL.md` instructions when they decide to use that skill." The initial resident list (names + descriptions only) is "constrained to approximately 2% of context window or 8,000 characters maximum, ensuring descriptions are abbreviated when many skills exist." (https://learn.chatgpt.com/docs/build-skills)

**Hooks — discovery and merge.** "Codex discovers hooks next to active config layers in either of these forms: `hooks.json` [or] inline `[hooks]` tables inside `config.toml`," at `~/.codex/hooks.json`, `~/.codex/config.toml`, `<repo>/.codex/hooks.json`, `<repo>/.codex/config.toml`. Merge behavior is explicitly a union, not an override: "If more than one hook source exists, Codex loads all matching hooks. Higher-precedence config layers don't replace lower-precedence hooks." Execution is concurrent and non-blocking of siblings: "Multiple matching command hooks for the same event are launched concurrently, so one hook can't prevent another matching hook from starting." Handler capabilities: a hook can return `"decision": "block"` with a reason to stop the triggering action, inject `"additionalContext"`, and (for `PreToolUse` only) rewrite input via `"updatedInput"`. Hook output defaults to a 2,500-token cap before spillover to disk. (https://learn.chatgpt.com/docs/hooks). Admin override: "Admins can set top-level `allow_managed_hooks_only = true` in `requirements.toml` to ignore user, project, and session hook configs while still allowing managed hooks from requirements and managed config layers." (https://raw.githubusercontent.com/openai/codex/main/docs/config.md). Local corroboration: the installed binary exposes a top-level flag `--dangerously-bypass-hook-trust` — "Run enabled hooks without requiring persisted hook trust for this invocation. DANGEROUS." — meaning hooks require an established, persisted "trust" before Codex will run them at all, and bypassing that gate is a named, explicitly-dangerous escape hatch. (`codex --help`, local install)

**Config layering generally.** "The CLI and IDE extension share the same configuration layers," and settings "follow a precedence order, with CLI flags taking highest priority." Profiles: "Config profile files live next to `config.toml` as `$CODEX_HOME/profile-name.config.toml`; select one with `--profile profile-name`." (https://learn.chatgpt.com/docs/config-file/config-basic, https://learn.chatgpt.com/docs/config-file/config-reference)

## Approval and Sandbox Modes

**Sandbox modes** (`sandbox_mode` config field / `-s`/`--sandbox` CLI flag):

| Mode | Description |
|------|-------------|
| `read-only` | "The agent can inspect files, but it can't edit files or run commands without approval." |
| `workspace-write` | "The agent can read files, edit within the workspace, and run routine local commands inside that boundary." (documented default for local work) |
| `danger-full-access` | "The agent runs without sandbox restrictions. This removes the filesystem and network boundaries and should be used only when you want the agent to act with full access." |

(https://learn.chatgpt.com/codex/sandboxing). Local `codex --help` confirms the exact same three values for `-s, --sandbox <SANDBOX_MODE>`: "[possible values: read-only, workspace-write, danger-full-access]".

**Approval modes** (`approval_policy` config field):

| Mode | Description |
|------|-------------|
| `on-request` | "Codex requires approval to edit outside the workspace or to access network." |
| `never` | Codex "never asks for approval" (non-interactive). |
| `untrusted` | "Codex runs only known-safe read operations automatically. Commands that can mutate state or trigger external execution paths require approval." |

Granular form: `approval_policy = { granular = { sandbox_approval = bool, rules = bool, mcp_elicitations = bool, request_permissions = bool, skill_approval = bool } }` lets an operator "keep specific approval prompt categories interactive while automatically rejecting others." There is also `auto_review`, a reviewer mode that "routes eligible approval requests through a reviewer agent before Codex runs the request." (https://learn.chatgpt.com/codex/agent-approvals-security, https://learn.chatgpt.com/docs/config-file/config-reference)

**`codex sandbox` subcommand.** A dedicated CLI verb, not just a config flag: "Run commands within a Codex-provided sandbox" (macOS "seatbelt" per its `--sandbox-state-json`/`--log-denials` flags — "While the command runs, capture macOS sandbox denials via `log stream` and print them after exit"). (`codex sandbox --help`, local install)

**Execution-policy rules engine** (`~/.codex/rules/default.rules`, distinct from sandbox/approval modes). "Use rules to control which commands Codex can run outside the sandbox." Rules use `prefix_rule()` to match command patterns; `decision` is `"allow"` (run without prompting), `"prompt"` (ask before each invocation), or `"forbidden"` (block the request). "When you add a command to the allow list in the TUI, Codex writes to the user layer at `~/.codex/rules/default.rules` so future runs can skip the prompt." "When Smart approvals are enabled (the default), Codex may propose a `prefix_rule` for you during escalation requests." Conflict/composition rule, tool-verified: "Codex then evaluates each command against your rules, and the most restrictive result wins. Even if you allow `pattern=["git", "add"]`, Codex won't auto allow `git add . && rm -rf /`, because the `rm -rf /` portion is evaluated separately and prevents the whole invocation from being auto allowed." (https://learn.chatgpt.com/docs/agent-configuration/rules)

## State and Observability

**Sessions (rollout persistence).** Documented CLI command surface for session lifecycle: `codex resume` ("Continue a previous interactive session by ID or resume the most recent chat"), `codex fork` ("Fork a previous interactive session into a new chat"), `codex archive` ("Archive a saved interactive session by session ID or session name"), `codex unarchive` ("Restore an archived interactive session"), `codex delete` ("Permanently delete a saved interactive session"). (https://learn.chatgpt.com/docs/cli/reference). Locally confirmed as real top-level subcommands with matching help text (`codex --help`; `codex resume --help` shows `[SESSION_ID]`, `--last`, `--all`). Interactive equivalents also exist as slash commands: `/rename`, `/archive`, `/delete`, `/clear`. (https://learn.chatgpt.com/docs/developer-commands?surface=cli)

**Memories (tool-generated persistence layer, distinct from sessions).** "Memories let ChatGPT and Codex carry useful context from earlier work into future work." Storage: "Codex stores memories under your Codex home directory. By default, that's `~/.codex`" and "The main memory files live under `~/.codex/memories/`." Default state: "Local Codex memories are off by default." Mechanism: "After you enable memories, Codex can turn useful context from eligible prior chats into local memory files," and it "updates memories in the background instead of immediately at the end of every chat." Config gate: `[features] memories = true` in `config.toml`, plus `memories.generate_memories` ("whether newly created chats can be stored as memory-generation inputs") and `memories.use_memories` ("whether Codex injects existing memories into future sessions"). (https://learn.chatgpt.com/docs/customization/memories?surface=cli). A `/memories` slash command exists to "Configure memory use and generation." (https://learn.chatgpt.com/docs/developer-commands?surface=cli). Local corroboration: a `~/.codex/memories/` directory exists on this machine, consistent with the documented path.

**Profiles.** Named config layers persisted at `$CODEX_HOME/profile-name.config.toml` (see Loading and Injection). Locally, several such profile files exist under `~/.codex/` (e.g. distinct `*.config.toml` files matching the documented naming pattern), corroborating that this is a real, used mechanism rather than a documented-but-dormant one.

**Logging.** `RUST_LOG` — "Controls Rust log filtering and verbosity. `codex exec` defaults to `error` output unless you set a more verbose value," and "accepts values such as `error`, `warn`, `info`, `debug`, and `trace`. It also accepts more targeted Rust logging filters, such as `codex_core=debug,codex_tui=debug`." Plaintext session log: "The interactive CLI records diagnostics in bounded local stores by default, but the plaintext `codex-tui.log` file is opt-in," enabled via the `log_dir` config field ("Directory where Codex writes log files; defaults to `$CODEX_HOME/log`"). (https://learn.chatgpt.com/docs/config-file/environment-variables, https://learn.chatgpt.com/docs/config-file/config-reference)

**OpenTelemetry.** A dedicated `[otel]` config section: `otel.environment` ("Environment tag applied to emitted OpenTelemetry events (default: dev)"), `otel.exporter`, `otel.metrics_exporter` ("defaults to statsig"), `otel.trace_exporter`, and `otel.log_user_prompt` ("Opt in to exporting raw user prompts with OpenTelemetry logs"). Exporters support `otlp-http` and `otlp-grpc`. (https://learn.chatgpt.com/docs/config-file/config-reference)

**Diagnostics.** `codex doctor` — "Diagnose local Codex installation, config, auth, and runtime health" (local `--help`); official CLI reference describes it as generating "a diagnostic report for local installation, config, auth, runtime, Git, terminal, app-server, and thread inventory issues." (https://learn.chatgpt.com/docs/cli/reference). Slash-command surfaces for live introspection: `/status` ("displays current session state and agent details"), `/mcp` ("List configured Model Context Protocol (MCP) tools"), `/hooks` ("View and manage lifecycle hooks"), `/feedback` ("Send logs to the Codex maintainers"), `/compact` ("Summarize the visible chat to free tokens"). (https://learn.chatgpt.com/docs/developer-commands?surface=cli)

## Sources

- https://github.com/openai/codex — repository root; primary source for this profile.
- https://raw.githubusercontent.com/openai/codex/main/docs/agents_md.md — confirms `docs/AGENTS.md` doc is a stub pointing to the hosted guide.
- https://learn.chatgpt.com/docs/agent-configuration/agents-md — AGENTS.md discovery locations, merge/override order, size cap (`project_doc_max_bytes`).
- https://raw.githubusercontent.com/openai/codex/main/docs/config.md — confirms `docs/config.md` is a stub; establishes the `allow_managed_hooks_only` admin override in `requirements.toml`.
- https://learn.chatgpt.com/docs/config-file/config-basic — config file path/format, profile files, CLI-flags-highest-precedence statement.
- https://learn.chatgpt.com/docs/config-file/config-reference — `mcp_servers.*` fields, `approval_policy` values (incl. granular form), `sandbox_mode` values, `[otel]` section, `log_dir`, `hooks` table.
- https://raw.githubusercontent.com/openai/codex/main/docs/sandbox.md — confirms `docs/sandbox.md` is a stub pointing to the hosted security guide.
- https://learn.chatgpt.com/codex/sandboxing — sandbox mode table (`read-only`, `workspace-write`, `danger-full-access`).
- https://learn.chatgpt.com/codex/agent-approvals-security — approval mode table (`on-request`, `never`, `untrusted`), granular approval policy, `auto_review`.
- https://raw.githubusercontent.com/openai/codex/main/docs/skills.md — confirms `docs/skills.md` is a stub pointing to the hosted skills guide.
- https://learn.chatgpt.com/docs/build-skills — skill definition, file layout, discovery locations, progressive-disclosure loading and its numeric budget cap.
- https://learn.chatgpt.com/docs/hooks — hook event list, config discovery locations, union (non-override) merge behavior, concurrent execution, handler capabilities (`decision`, `additionalContext`, `updatedInput`), output token cap.
- https://learn.chatgpt.com/docs/plugins — plugin definition, what a plugin can bundle, marketplace-based discovery/installation, post-install availability.
- https://learn.chatgpt.com/docs/agent-configuration/rules — execution-policy rules engine (`prefix_rule`, decision values, storage path, most-restrictive-wins composition, compound-command safety example).
- https://learn.chatgpt.com/docs/cli/reference — CLI subcommand descriptions: `mcp`, `mcp-server`, `resume`, `fork`, `archive`, `unarchive`, `delete`, `doctor`.
- https://learn.chatgpt.com/docs/config-file/environment-variables — `RUST_LOG` behavior and accepted values, `codex-tui.log` opt-in behavior.
- https://learn.chatgpt.com/docs/customization/memories?surface=cli — Memories feature: storage path, default-off state, background generation, `[features] memories`, `memories.generate_memories`, `memories.use_memories`.
- https://learn.chatgpt.com/docs/developer-commands?surface=cli — interactive slash-command reference (`/status`, `/mcp`, `/hooks`, `/memories`, `/permissions`, `/compact`, `/feedback`, `/agent`, session-management commands).
- Local install (`codex-cli 0.144.1`): `codex --version`, `codex --help`, `codex plugin --help`, `codex plugin marketplace --help`, `codex mcp --help`, `codex sandbox --help`, `codex resume --help` — corroborates that the mechanisms above (mcp, mcp-server, plugin/marketplace, sandbox modes, hook-trust gate, resume/fork/archive/unarchive/delete) exist as real, working commands in the installed version, and confirms the presence of `~/.codex/skills` and `~/.codex/memories/` directories consistent with the documented locations.
