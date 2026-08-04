---
title: "Cursor Extension Model"
source: "https://docs.cursor.com"
type: notes
ingested: 2026-08-04
tags: [cursor, extension-unit, mechanism]
summary: "Evidence on Cursor's rule files, their scoping fields, and its MCP surface. Collected from the official documentation."
---

# Cursor Extension Model

## Version Examined

`3.14.7` — read from the installed application bundle on this machine via:

```
plutil -p /Applications/Cursor.app/Contents/Info.plist | grep -i CFBundleShortVersionString
```

Output: `"CFBundleShortVersionString" => "3.14.7"`. This is an installed-application version (not read from an About panel, which cannot be opened in this environment), obtained 2026-08-04. The official documentation pages consulted below carry no version number or "last updated" date of their own — a direct query of the rules page for any version/last-updated string returned no matches.

## Rule Files

Three storage tiers, each with its own persistence location, plus one file-based alternative:

- **Project Rules**: "Project rules live in `.cursor/rules` as `.mdc` files and are version-controlled." Plain `.md` files placed in that directory are ignored — only the `.mdc` extension is read.
- **AGENTS.md**: a simpler, single-file alternative — "Place it in your project root as an alternative to `.cursor/rules` for straightforward use cases." Nested `AGENTS.md` files in subdirectories are also supported: "You can place `AGENTS.md` files in any subdirectory of your project, and they will be automatically applied when working with files in that directory or its children." (See Overlap Resolution for how nested copies combine.)
- **User Rules**: not a file at all — "Global to your Cursor environment. Used by Agent (Chat)." They are app-local settings edited from Customize → Rules, apply across every project on one machine, are not version-controlled, and do not transfer between machines unless manually recreated.
- **Team Rules**: managed centrally, not per-repo — "Team and Enterprise plans can create and enforce rules across their entire organization from the Cursor dashboard." An admin can lock one on: "When enabled, the rule is required for all team members and cannot be disabled in Customize."

**Frontmatter fields.** A `.mdc` rule's YAML frontmatter is documented as exactly three fields, all introduced together under a "Rule anatomy" section: `description`, `globs`, `alwaysApply`. A direct re-query for any additional field (name, version, tags) confirmed none exists: "The only documented frontmatter properties are those three." The docs frame the combination as: "Control how rules are applied from the type dropdown which changes properties `description`, `globs`, `alwaysApply`."

Best-practice size guidance, not an enforced limit: "Keep rules under 500 lines." No maximum rule count or token/context-budget cap is documented anywhere on the page beyond this per-file line-count recommendation.

## Scoping and Selection

The documentation names four distinct application mechanisms for a `.mdc` rule (a rule can only use one, selected by which of `description`/`globs`/`alwaysApply` are set):

1. **Always Apply** — `alwaysApply: true` includes the rule in every chat session, unconditionally.
2. **Apply Intelligently** — `alwaysApply: false` with a `description` set but no `globs`; the doc's own label for the trigger is "when Agent decides it's relevant based on description." This is model judgment, not a mechanical check.
3. **Apply to Specific Files** — `alwaysApply: false` with `globs` set; the rule attaches "when file paths match patterns in `globs`," a mechanical, tool-computed path match. Example glob patterns given verbatim: `*`, `**`, `*.ts`, `**/*.ts`, `src/**`, `src/**/*.tsx`, `docs/**/*.md, docs/**/*.mdx`, `tailwind.config.*`.
4. **Apply Manually** — `alwaysApply: false` with both `description` and `globs` omitted; "Included only when you `@`-mention the rule in chat" (e.g. `@my-rule`).

Of these four, (1) always-apply and (3) glob match are deterministic, tool-executed checks — the tool itself decides inclusion, not the model. (4) manual mention is an explicit user action. Only (2) "Apply Intelligently" hands the relevance judgment to the model, with no verification that the judgment was correct — the same shape of mechanism Claude Code uses for all its skills, but here it is one of four options rather than the only one.

**Rubric-aggregation gap, recorded here as a finding, not resolved here.** `.wiki/schema.md` defines what each of the four score levels (`0`-`3`) means but says nothing about how to aggregate a single axis score when a tool exposes several mechanisms with different enforcement levels for that same axis — e.g. Cursor's four discoverability mechanisms split 3 enforced / 1 model-judged. Whether the axis score should reflect the strongest mechanism available (a ceiling read) or be capped by the weakest one still in play (a floor read) is left to the profile author's judgment; this note flags the gap for the scoreboard task rather than silently picking one convention.

Nested `AGENTS.md` files are scoped by directory: applied automatically to files in "that directory or its children," and combined with parent directories (see Overlap Resolution).

**Troubleshooting surface documented for this axis**: "Check the rule type. For `Apply Intelligently`, ensure a description is defined. For `Apply to Specific Files`, ensure the file pattern matches referenced files." This is manual, user-run diagnosis, not an automated indicator.

## Overlap Resolution

**Cross-scope precedence (documented, applies to Team/Project/User rules together).** "Rules are applied in this order: Team Rules → Project Rules → User Rules. All applicable rules are merged; earlier sources take precedence when guidance conflicts." This is a stated, deterministic concatenation order, not a hard override/replace: all applicable rules go into context together, ordered so that Team text precedes Project text precedes User text.

**Nested AGENTS.md precedence (documented).** "Instructions from nested `AGENTS.md` files are combined with parent directories, with more specific instructions taking precedence." — the child directory's file is the more specific source and wins on conflict.

**Same-scope overlap — the case this note was asked to check explicitly: what happens when two Project Rules (both `.mdc` files in `.cursor/rules`, same tier) have globs that both match the file currently being edited?** A direct, targeted query of the rules documentation for this exact scenario found nothing: "the page says nothing about ordering, precedence, or conflict resolution between multiple Project Rules with overlapping glob patterns that match the same file. The documentation addresses cross-scope precedence (Team → Project → User rules) but does not discuss how multiple Project Rules within the same scope would be arbitrated." Given the general framing that "all applicable rules are merged," the working assumption is that same-scope overlapping rules are all concatenated into context with no documented order between them — but this is an **inference from an absence**, not a recorded fact, and is labeled as such.

## Deferred Content and MCP

**Deferred content.** A rule body does not have to inline everything it needs: "Use `@filename.ts` to include files in your rule's context. You can also @mention rules in chat to apply them manually." This lets a rule point at canonical source files rather than duplicating their content into the rule body.

**MCP configuration.** Declared in a `mcpServers` JSON object, read from two locations: project-level `.cursor/mcp.json` and global `~/.cursor/mcp.json`. Two transport shapes: stdio (`type: "stdio"`, `command`, optional `args`/`env`/`envFile`) and SSE/HTTP (`url`, optional `headers`, static OAuth via an `auth` object with `CLIENT_ID`/`CLIENT_SECRET`/`scopes`). Config values support interpolation: `${env:NAME}`, `${userHome}`, `${workspaceFolder}`, `${workspaceFolderBasename}`.

**MCP permission model.** Default is approval-gated, per connection and per call: "Cursor asks for approval before using MCP tools by default" and, from the security overview page, "All MCP connections need your approval. After you approve an MCP connection, each tool call still needs individual approval before running."

**Run Modes** (https://cursor.com/docs/agent/security/run-modes) generalize approval across MCP tools, shell commands, and fetch calls. Three modes, quoted verbatim:
- **Auto-review** (default): "Allowlisted calls run immediately. Other shell commands run in the sandbox when possible. Calls that do not use the sandbox go to the Auto-review classifier."
- **Allowlist**: "Actions in your allowlist run without approval. With sandboxing enabled, supported shell commands can run in the sandbox."
- **Run Everything**: "Every tool call runs automatically" — no sandbox, no classifier.

The classifier's own documented limitation, under a section literally titled "Auto-review is not a security boundary": "The classifier can make mistakes. It can allow a call you would have blocked, or block a call you would have allowed." (This is the only self-limitation quote confirmed on the run-modes page itself; the broader "best-effort guardrails" framing, quoted below, was verified to live on the security overview page instead, not here.)

**Sandbox mechanics** (same page): a sandboxed process can read/write workspace files freely and write to temp directories, but cannot access protected paths (`.git/config`, `.vscode`, `.cursorignore`, sensitive config) and has network access blocked by default until domains are explicitly allowed. Platform implementation: macOS via Seatbelt (`sandbox-exec`); Linux via Landlock (filesystem) + seccomp (syscalls), requiring kernel 6.2+; AppArmor setup is needed for remote/CLI environments.

**`permissions.json`** (https://cursor.com/docs/reference/permissions): read from `~/.cursor/permissions.json` (global) and `<workspace>/.cursor/permissions.json` (per-repo); "when both exist, their arrays are concatenated rather than one replacing the other." Fields: `mcpAllowlist` (string array, `server:tool` syntax with `*` wildcards), `terminalAllowlist` (string array, prefix match, case-sensitive), `autoRun` (object with `allow_instructions`/`block_instructions` natural-language hints that steer the Auto-review classifier). Precedence, highest to lowest: Team admin controls > `permissions.json` > IDE settings. A file-based allowlist overrides and locks the in-app one: "When `permissions.json` defines an allowlist, it overrides the corresponding in-app allowlist in Cursor Settings," and "the in-app allowlist editor becomes read-only." Constraint on activation: "`permissions.json` only takes effect when Run Mode is enabled in Cursor Settings (Auto-review, Allowlist, or Run Everything)," and the `autoRun` field is consulted only in Auto-review mode.

**Enterprise/admin controls** (from the MCP page): an MCP allowlist keyed by command pattern (stdio) or URL pattern (HTTP/SSE); a per-server tool allowlist ("empty allowlist permits all tools"); network controls restricting remote URLs to configured patterns; and local-server modes (Allow all, Allowlist, Deny all, No sandbox). Admins can optionally let users configure their own MCP servers outside the admin-defined patterns.

**Other approval-gated surfaces** (https://cursor.com/docs/agent/security): "Reading files and searching code don't require approval... Actions that could expose sensitive data require your explicit approval." "Agents can modify workspace files without approval, except for configuration files... Configuration files (like workspace settings) need your approval first." "By default, terminal commands need your approval. To let trusted calls run without prompting, configure Run Modes. They range from a simple allowlist to the Auto-review classifier, and they're best-effort guardrails rather than a hard security boundary." **Re-attribution note (post-review fix):** this "best-effort guardrails rather than a hard security boundary" sentence was originally miscited in this note to the run-modes page; a direct fetch of `https://cursor.com/docs/agent/security` confirmed it verbatim in this exact paragraph, and a direct fetch of `https://cursor.com/docs/agent/security/run-modes` confirmed the phrase does not appear there. Network egress is restricted by default: "Agents cannot make arbitrary network requests with default settings," with permitted destinations limited to GitHub, direct link retrieval, and web search providers. Workspace trust is "disabled by default" and, when enabled, can restrict AI features entirely.

**Observability for MCP** (https://cursor.com/docs/context/mcp): "Open the Output panel in Cursor (Cmd+Shift+U)," then "Select 'MCP Logs' from the dropdown," to "Check for connection errors, authentication issues, or server crashes." "The logs show server initialization, tool calls, and error messages." On a server failure, "Cursor shows an error message in chat" and "The tool call is marked as failed," directing the user to check logs for details. This is a documented, concrete diagnostic surface. No equivalent documented log/metrics surface was found for rules themselves beyond the manual troubleshooting checklist quoted under Scoping and Selection, and a direct query of the Agent Security page for logging/diagnostics content returned none: "The document contains no information about logging, diagnostics, or observability surfaces for agent actions."

**Memories — explicitly not used as evidence here.** Community forum threads (e.g. https://forum.cursor.com/t/0-51-memories-feature/98509) describe a beta "Memories" feature (enable via Settings → Rules → "Generate Memories"; a background model proposes a memory, the user approves it, and entries are reviewable/editable/deletable from Settings) that would be relevant to the `state` axis. This note deliberately does not rely on that description: the documentation URL that should cover it, `https://cursor.com/docs/context/memories`, resolves to the same content as the Rules page (confirmed by fetching its H1 and opening paragraph, which read "Rules" / "Rules provide system-level instructions to Agent..."), and the current changelog listing (https://cursor.com/changelog, entries from 2026-07-17 through 2026-08-03 at the time of this research) contains no Memories entry. Per the schema's source rules, forum posts may only support an argument, not carry it, and no primary documentation confirming Memories' current behavior was found. This is recorded as an absence in the primary sources checked, not as a verified mechanism.

## Sources

- https://cursor.com/docs/context/rules — rule file format/location for all four tiers (Project/.mdc, AGENTS.md incl. nested, User, Team), the three-field frontmatter (`description`/`globs`/`alwaysApply`) and confirmation no other field exists, the four application mechanisms and their exact trigger text, the glob pattern examples, the 500-line guidance, the cross-scope precedence quote, the nested-AGENTS.md precedence quote, the confirmed absence of same-scope overlap resolution, the `@filename` deferred-content quote, and the rule-troubleshooting checklist.
- https://cursor.com/docs/context/mcp — `mcp.json` locations and schema (stdio/SSE transports, config interpolation), default per-tool approval requirement, and the Output-panel "MCP Logs" observability surface with its documented contents and failure-marking behavior.
- https://cursor.com/docs/agent/security/run-modes — the three Run Modes (Auto-review, Allowlist, Run Everything) with verbatim behavior, the classifier's self-described fallibility (confirmed under its "Auto-review is not a security boundary" heading), and sandbox mechanics (protected paths, default-deny network, macOS Seatbelt / Linux Landlock+seccomp). Does NOT contain the "best-effort guardrails" sentence — confirmed absent by direct re-fetch.
- https://cursor.com/docs/reference/permissions — `permissions.json` file locations, concatenation-not-replacement behavior across the two locations, its three fields (`mcpAllowlist`, `terminalAllowlist`, `autoRun`), the team-admin > permissions.json > IDE-settings precedence, the override-and-lock-UI behavior, and the constraint that it only takes effect when a Run Mode is enabled.
- https://cursor.com/docs/agent/security — approval-by-default behavior for file reads/edits/config changes/terminal commands/MCP connections, the "best-effort guardrails rather than a hard security boundary" framing (confirmed here, not on the run-modes page, by direct fetch), default-deny network egress with named exceptions, workspace-trust default-off state, and the confirmed absence of any logging/diagnostics content on that page.
- https://forum.cursor.com/t/0-51-memories-feature/98509 — forum-only description of the beta Memories feature; cited solely to explain why this note treats Memories as unconfirmed rather than as evidence, per the schema's rule that blog/forum posts may support but not carry an argument.
- https://cursor.com/changelog — checked for a Memories entry (2026-07-17 through 2026-08-03 window visible); none found, supporting the absence noted above.
