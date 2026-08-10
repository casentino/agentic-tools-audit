# Task 4 Report: Profile OpenAI Codex CLI

## Version recorded

`codex --version` on this machine → **`codex-cli 0.144.1`**. Codex CLI is installed locally, so per the brief's ordering this is the version recorded (not a release tag or commit SHA). Obtained by running `codex --version` directly (Bash).

Documentation was read from the `openai/codex` GitHub repository's `main` branch and the hosted docs site its `docs/` stub files point to (`developers.openai.com/codex/...`, 308-redirecting to `learn.chatgpt.com/docs/...`), which is not pinned to the 0.144.1 tag. To catch any drift, I cross-checked the documented mechanisms against the installed binary's own `--help` output (`codex --help`, `codex plugin --help`, `codex mcp --help`, `codex sandbox --help`, `codex resume --help`) and against two directories that exist on disk (`~/.codex/skills`, `~/.codex/memories/`). No contradiction was found — every mechanism the hosted docs describe (mcp, mcp-server, plugin/marketplace, three sandbox modes, hook-trust gate, resume/fork/archive/unarchive/delete) is present verbatim in the installed 0.144.1 binary's own help text. This is recorded explicitly in the evidence note's "Version Examined" section.

## Sources consulted and what each established

- `github.com/openai/codex` (repo root) — orientation; confirmed the `docs/` directory is a set of one-line stubs, not the substantive documentation.
- `docs/agents_md.md`, `docs/config.md`, `docs/sandbox.md`, `docs/skills.md`, `docs/slash_commands.md`, `docs/execpolicy.md`, `docs/exec.md`, `docs/getting-started.md` (raw GitHub) — each confirmed to be a redirect stub; `docs/config.md` additionally established the `allow_managed_hooks_only` admin override.
- `learn.chatgpt.com/docs/agent-configuration/agents-md` — AGENTS.md discovery (three scopes, `AGENTS.override.md` precedence), merge order (concatenate root-down, closer overrides), and the `project_doc_max_bytes` size cap.
- `learn.chatgpt.com/docs/config-file/config-basic` and `config-reference` — config file location/format, `mcp_servers.*` fields, `approval_policy`/`sandbox_mode` values, profiles path, `[otel]` section, `log_dir`, `hooks` table.
- `learn.chatgpt.com/codex/sandboxing` and `codex/agent-approvals-security` — the three sandbox modes and three approval modes, verbatim, plus the granular approval-policy object and `auto_review`.
- `learn.chatgpt.com/docs/build-skills` — skill file layout, five discovery locations, progressive-disclosure loading and its numeric budget (~2% context / 8,000 chars).
- `learn.chatgpt.com/docs/hooks` — full event list, hook config discovery locations, the union (non-override) merge rule, concurrent execution, and handler capabilities (`decision`, `additionalContext`, `updatedInput`).
- `learn.chatgpt.com/docs/plugins` — plugin definition, what it bundles (skills/connectors/MCP/browser extensions/hooks/scheduled tasks), marketplace-based discovery.
- `learn.chatgpt.com/docs/agent-configuration/rules` — the execution-policy rules engine (`prefix_rule`, decision values, storage path, "most restrictive result wins," the `git add . && rm -rf /` verification example).
- `learn.chatgpt.com/docs/cli/reference` — CLI subcommand descriptions for `mcp`, `mcp-server`, `resume`, `fork`, `archive`, `unarchive`, `delete`, `doctor`.
- `learn.chatgpt.com/docs/config-file/environment-variables` — `RUST_LOG` behavior, `codex-tui.log` opt-in status.
- `learn.chatgpt.com/docs/customization/memories?surface=cli` — the Memories system: storage path, off-by-default state, background generation, and its three config keys.
- `learn.chatgpt.com/docs/developer-commands?surface=cli` — interactive slash-command reference (`/status`, `/mcp`, `/hooks`, `/memories`, `/compact`, `/feedback`, etc.).
- Local install (`codex-cli 0.144.1`) — `codex --help` and subcommand `--help` output, used only to corroborate that documented mechanisms exist in the installed version; not used as a documentation source in its own right.

## The six scores

| Axis | Score | One-line evidence |
|------|-------|--------------------|
| `discoverability` | 2 | Skill relevance is model-judged from an always-resident name+description under a fixed budget; nothing verifies the model picked correctly, and AGENTS.md/hooks aren't "discovered" at all — they fire unconditionally. |
| `context-budget` | 3 | Two numeric, tool-enforced caps found: `project_doc_max_bytes` for AGENTS.md concatenation, and ~2%-of-context/8,000-char for the resident skill list. |
| `composition` | 2 | AGENTS.md and the rules engine both have tool-verified deterministic reconciliation, but hooks are the documented counterexample — all matching hooks run, none override. |
| `state` | 3 | Two tool-managed, documented persistence mechanisms with defined locations/config gates: session rollouts (resume/fork/archive/unarchive/delete) and an explicitly off-by-default Memories system. |
| `side-effect-control` | 3 | OS-level sandbox (3 modes) + approval policy (3 modes + granular form) + a rules engine tool-verified against compound-command bypass, plus a hook-trust gate. |
| `observability` | 2 | Rich documented surfaces (`codex doctor`, `RUST_LOG`, opt-in `codex-tui.log`, native `[otel]` section, `/status`/`/mcp`/`/hooks`) but none of them run or verify by default — all user-invoked. |

## Axes scored 0 or 1

None. Every axis found a real, documented mechanism (2 or 3). This is itself a notable finding — see Concerns below.

## Check block run and output

```bash
cd /Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki
f=wiki/topics/codex-cli.md
for k in title category sources created updated tags aliases confidence volatility verified summary; do
  grep -q "^$k:" "$f" || echo "MISSING frontmatter: $k"
done
for a in discoverability context-budget composition state side-effect-control observability; do
  grep -q "\`$a\`" "$f" || echo "MISSING axis: $a"
done
for s in "## Extension Model" "## Rubric Scores" "## Portability" "## Sources"; do
  grep -q "^$s" "$f" || echo "MISSING section: $s"
done
grep -qE '^\| `[a-z-]+` \| [0-3] \|' "$f" || echo "FAIL: no scored axis rows"
grep -q '\[0-3\]' "$f" && echo "FAIL: unfilled score placeholder remains"
test -f raw/notes/2026-08-04-codex-cli-extension-model.md || echo "MISSING evidence note"
```

Output: **(none)** — matches the brief's "Expected: no output."

`git diff --cached --check` before commit also produced no output (exit 0).

## Files changed

- Created: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/raw/notes/2026-08-04-codex-cli-extension-model.md`
- Created: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/wiki/topics/codex-cli.md`
- Modified: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/raw/notes/_index.md` (added Contents row)
- Modified: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/wiki/topics/_index.md` (added Contents row)
- Modified: `/Users/kikyeongoh/Documents/opterk/agentic-tools-audit/.wiki/log.md` (appended log line)
- `.wiki/_index.md` was not touched.
- `.wiki/inventory/` was not created.
- Commit: `b0c8f04` on `feat/agentic-tools-wiki` — "feat: profile OpenAI Codex CLI against the audit rubric" (no attribution/co-author trailer, as instructed).

## Self-review

Walked each of the six axes back to the evidence note:

- **discoverability (2):** rests on the note's "Skills — progressive disclosure" quote and the general "read before doing any work" / hook handler-capabilities language, both present with URLs (`learn.chatgpt.com/docs/build-skills`, `learn.chatgpt.com/docs/agent-configuration/agents-md`, `learn.chatgpt.com/docs/hooks`).
- **context-budget (3):** rests on `project_doc_max_bytes` (agents-md URL) and the "2% / 8,000 characters" figure (build-skills URL), both quoted in the note.
- **composition (2):** rests on AGENTS.md override-order quote (agents-md URL), the rules-engine "most restrictive result wins" quote (agent-configuration/rules URL), and the hooks union-merge quote (hooks URL) — all three present in the note.
- **state (3):** rests on the resume/fork/archive/unarchive/delete command descriptions (cli/reference URL) and the Memories off-by-default/config-key quotes (customization/memories URL), both in the note.
- **side-effect-control (3):** rests on the sandbox-mode table (codex/sandboxing URL), approval-mode table (codex/agent-approvals-security URL), the rules-engine compound-command example (agent-configuration/rules URL), and the `--dangerously-bypass-hook-trust` local corroboration — all in the note.
- **observability (2):** rests on `codex doctor`/RUST_LOG/otel quotes (cli/reference, environment-variables, config-reference URLs) and the slash-command list (developer-commands URL) — all in the note.

I also re-checked every quoted string in the profile against the note (the AGENTS.md merge-order quote, the hooks union-merge quote, the "most restrictive result wins" quote, the memories "off by default" quote, the plugin post-install-availability quote, the sandbox/approval mode names, the rules-engine example, the `--dangerously-bypass-hook-trust` flag, the config keys for Memories) — none appears in the profile without a matching, URL-cited appearance in the note first.

## Concerns

The brief frames Codex CLI as "deliberately thin" and "the useful low end of the scale" so that not every tool scores 2-3. My research of the actual current documentation (main-branch docs + the hosted `learn.chatgpt.com` site) found a **materially richer surface than that framing anticipated**: Codex now has skills with progressive disclosure, an event-keyed hooks system whose event names closely mirror Claude Code's own (`SessionStart`, `PreToolUse`, `PostToolUse`, `Stop`, ...), a marketplace-distributed plugin/bundling layer, and an explicit background "Memories" system — none of which score 0 or 1. I did not stretch to manufacture these; each is directly documented and, where checkable, corroborated by the installed 0.144.1 binary's own `--help` output. I flagged rather than suppressed this: no axis was forced low to match the brief's expectation, and the profile's body explicitly calls out where Codex genuinely is loosely-governed (hook merge is a documented non-arbitrating union) versus where it is unusually strict (the sandbox + rules-engine + hook-trust-gate side-effect layer). If the task's downstream consumers (Task 7's scoreboard, Task 8's pattern-sighting) specifically need a rubric floor example, Codex CLI may not supply one as cleanly as the brief assumed — worth flagging to the plan owner, since a different/older tool or an older Codex release might fit the "thin" framing better.

Second, minor: exact on-disk paths for session rollout files (e.g., the `~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl` pattern reported consistently by several third-party sources) could not be confirmed on an official docs page I could reach, so I deliberately left that specific path detail out of the note/profile rather than cite a blog/DeepWiki claim as primary — the state axis score instead rests on the officially documented command surface (`resume`/`fork`/`archive`/`unarchive`/`delete`) and the officially documented Memories path (`~/.codex/memories/`), both of which I could verify directly.
