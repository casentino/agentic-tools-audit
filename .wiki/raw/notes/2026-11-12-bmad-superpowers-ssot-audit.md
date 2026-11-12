---
title: "BMAD-METHOD, Superpowers, and SSOT-Manager Audit Note"
source: "MANUAL"
type: notes
ingested: 2026-11-12
tags: [bmad-method, superpowers, ssot-manager, prd-validation, context-budget, extension-unit]
summary: "Evidence collected from BMAD-METHOD (6.11.0), Superpowers (6.3.0), and the owner's ssot-manager (0.1.0) on PRD analysis/validation pipelines, enterprise customization surfaces, and CLAUDE.md/AGENTS.md context-budget hygiene."
---

# BMAD-METHOD, Superpowers, and SSOT-Manager Audit Note

## Versions Examined

- **BMAD-METHOD**: `6.11.0` (`package.json`), commit `67d876f1c317a82123e429d0c078ff3c601a99a8` (`git -C clones/BMAD-METHOD rev-parse HEAD`).
- **Superpowers**: `6.3.0` (`package.json`), commit `b36e0829c6d0140e93cfef2ca599b1b07d4a7797` (`git -C clones/superpowers rev-parse HEAD`).
- **ssot-manager** (owner workspace `/Users/casentino/Developer/hwahae/workspace/ssot-manager`): `0.1.0` (`package.json`), commit `069821292115cd961428287518c340df618056dc` (`git -C /Users/casentino/Developer/hwahae/workspace/ssot-manager rev-parse HEAD`).

---

## 1. BMAD-METHOD: PRD Analysis, Validation, and Enterprise Customization

### Phase 1 Analysis Upstream (`docs/explanation/analysis-phase.md`)

BMAD separates Phase 1 (Analysis) from Phase 2 (Planning) so PRD authoring does not rest on unexamined assumptions: "A PRD answers 'what should we build and why?' If you feed it vague thinking, you get a vague PRD — and every downstream document inherits that vagueness." (`clones/BMAD-METHOD/docs/explanation/analysis-phase.md`, lines 12–14).

Four optional upstream skills feed `bmad-prd`:
1. **Brainstorming** (`bmad-brainstorming`): facilitated creative exploration.
2. **Research / Deep Recon** (`bmad-deep-recon`): typed research packs covering market, domain, technical, competitive teardowns, user-voice, and academic literature (`clones/BMAD-METHOD/docs/explanation/analysis-phase.md`, line 30).
3. **Product Brief** (`bmad-product-brief`): 1–2 page collaborative discovery summary (`clones/BMAD-METHOD/docs/explanation/analysis-phase.md`, line 40).
4. **PRFAQ** (`bmad-prfaq`): Working Backwards press release and FAQ gauntlet (`clones/BMAD-METHOD/docs/explanation/analysis-phase.md`, lines 48–50).

The Business Analyst persona (`Mary`, `src/bmm-skills/agents/bmad-agent-analyst/SKILL.md` and `customize.toml`) dispatches these upstream skills from a structured menu (`BP`, `MR`, `DR`, `TR`, `TS`, `CR`, `UV`, `CB`, `WB`, `PC`) and carries `identity = "Channels Michael Porter's strategic rigor and Barbara Minto's Pyramid Principle discipline."` (`clones/BMAD-METHOD/src/bmm-skills/agents/bmad-agent-analyst/customize.toml`, line 41).

### Core PRD Skill (`src/bmm-skills/plan/bmad-prd/SKILL.md`)

`bmad-prd` supports three intent modes (`Create`, `Update`, `Validate`) and splits state across three distinct artifacts in `{doc_workspace}` (`clones/BMAD-METHOD/src/bmm-skills/plan/bmad-prd/SKILL.md`, line 14):
- `prd.md`: the requirements document itself (capabilities, not implementation).
- `.memlog.md`: "the run's canonical memory and audit trail — every decision, change, and override (including headless overrides) lands as one append-only line as the conversation unfolds. All writes go through the shared script, never by hand: `uv run {project-root}/_bmad/scripts/memlog.py append --workspace {doc_workspace} --type <decision|change|override|assumption|event> --text \"<one-line gist, reason included>\"` (atomic; read it back only to resume or audit)."
- `addendum.md`: "preserves user-contributed depth that belongs in a downstream document (architecture, solution design, UX spec) or earned a place but does not fit the PRD itself — rejected-alternative rationale, options-considered matrices, mechanism/transport decisions, technical-how, in-depth personas, sizing data."

To protect the parent context budget, `bmad-prd` enforces subagent extraction and compact review returns:
- "Source documents go to subagents for extraction; the parent assembles from extracts. Only load source documents into the parent context wholesale when no subagents are available." (`clones/BMAD-METHOD/src/bmm-skills/plan/bmad-prd/SKILL.md`, line 67).
- At the Reviewer Gate: "Each writes its full review to `{doc_workspace}/review-{slug}.md` and returns ONLY a compact summary (verdict, top 2-5 findings, file path) — the parent never holds full review text." (`clones/BMAD-METHOD/src/bmm-skills/plan/bmad-prd/SKILL.md`, line 77).

### PRD Validation Playbook and Seven-Dimension Rubric

When invoked under `Validate` intent (`src/bmm-skills/plan/bmad-prd/references/validate.md`), the rubric-walker subagent evaluates `prd.md` (and `addendum.md`) against `src/bmm-skills/plan/bmad-prd/assets/prd-validation-checklist.md` ("PRD Quality Rubric"). Each dimension receives a qualitative judgment (`strong | adequate | thin | broken`) backed by specific citations (`clones/BMAD-METHOD/src/bmm-skills/plan/bmad-prd/assets/prd-validation-checklist.md`, line 10):

1. **Decision-readiness**: "Decisions that are stated as decisions, not buried as 'considerations.' Trade-offs named with what was given up, not just what was chosen." (lines 47–48).
2. **Substance over theater**: flags **Persona theater** ("Personas that don't drive a single decision in the PRD"), **Innovation theater**, **NFR theater** ("copied boilerplate ('system must be scalable / secure / reliable') without product-specific thresholds"), and **Vision theater** (lines 58–61).
3. **Strategic coherence**: checks for a unified thesis, feature prioritization following from the thesis, Success Metrics validating the thesis, and named counter-metrics (lines 67–74).
4. **Done-ness clarity**: "FRs with at least one testable consequence per FR — verifiable condition, measurable outcome." Flags vague phrases like `"System handles X gracefully," "reasonable performance," "user-friendly"` (lines 83–84).
5. **Scope honesty**: explicit `Non-Goals`, `[ASSUMPTION: …]` tags indexed at the end, and `[NOTE FOR PM]` callouts (lines 95–98).
6. **Downstream usability**: glossary consistency, contiguous unique IDs (`FR`, `UJ`, `SM`), and named protagonists in User Journeys (lines 107–110).
7. **Shape fit**: calibrates structure to product shape (consumer vs. single-operator internal tool vs. regulatory update) and flags over- or under-formalization (lines 118–125).

The synthesis pass (`references/validate.md`, lines 31–48) consolidates `review-rubric.md` and any extra `review-*.md` files into both `{doc_workspace}/validation-report.html` (filled from `assets/validation-report-template.html`) and `{doc_workspace}/validation-report.md`.

### Enterprise Customization Surface (`docs/how-to/expand-bmad-for-your-org.md`)

BMAD adapts to organizational standards without editing installed skill files or forking by merging TOML overrides across three layers (`clones/BMAD-METHOD/docs/how-to/expand-bmad-for-your-org.md`, lines 25–30, and `src/bmm-skills/plan/bmad-prd/customize.toml`, lines 5–12):
- **Base defaults**: `{skill-root}/customize.toml`
- **Team overrides (committed)**: `{project-root}/_bmad/custom/{skill-name}.toml`
- **Personal overrides (gitignored)**: `{project-root}/_bmad/custom/{skill-name}.user.toml`
- **Central config**: `{project-root}/_bmad/custom/config.toml`

Merge semantics (`uv run {project-root}/_bmad/scripts/resolve_customization.py`): scalars override, tables deep-merge, arrays of tables keyed by `code` or `id` replace matching entries and append new ones, and other arrays append (`clones/BMAD-METHOD/src/bmm-skills/agents/bmad-agent-analyst/SKILL.md`, line 31).

In `bmad-prd/customize.toml`, organizations override:
- `prd_template` and `validation_checklist_template` (lines 43, 52) to swap in regulated-industry or company-specific templates and rubrics.
- `persistent_facts` (line 33) for standing constraints loaded at activation (literal strings, `skill:`, or `file:` globs).
- `external_sources` (line 108) for on-demand MCP/knowledge-base directives consulted only when a matching need arises during Discovery or drafting.
- `doc_standards` (line 87, default `["skill:bmad-review lenses=structure,prose"]`) applied at Finalize step 5.
- `external_handoffs` (line 127) and `on_complete` (line 38) to publish finalized PRDs to Confluence, Notion, or Jira at Finalize step 6.

---

## 2. Context-Budget Hygiene for `CLAUDE.md` and `AGENTS.md`

### BMAD's `bmad-project-context` Skill (`src/bmm-skills/plan/bmad-project-context/`)

BMAD treats instruction-file bloat as an explicit reliability defect:
- "Every line is paid in every session, and instruction-following degrades as the loaded set grows. Count what other always-loaded files add. Over budget means cut the weakest lines or move them behind a trigger — never raise the budget." (`clones/BMAD-METHOD/src/bmm-skills/plan/bmad-project-context/references/best-practices.md`, line 45).
- "Keep the IDE file **succinct**. A dozen well-chosen lines are more effective than a sprawling list. Models read it every turn, and noise crowds out signal." (`clones/BMAD-METHOD/docs/how-to/expand-bmad-for-your-org.md`, line 228).

Key mechanisms in `bmad-project-context`:
1. **Single-source consolidation (`@AGENTS.md`)**: "The proposal states what remains of every file instructions were moved out of — commonly a `CLAUDE.md` reduced to `@AGENTS.md`, once that import is verified for every harness in use, like any loading mechanism. No instruction lives in two loaded files, where it is paid for twice" (`clones/BMAD-METHOD/src/bmm-skills/plan/bmad-project-context/SKILL.md`, line 91).
2. **Explicit Exclusion Rules**: `references/best-practices.md` (lines 26–35) excludes repo overviews, directory trees, stack lists ("Derived fresh, more accurately; stored copies rot"), style rules enforceable by a linter/hook ("Belongs in a formatter, linter, hook, or CI check — propose the check instead"), and transcribed command lists whose obvious invocation already works.
3. **Prose-to-Automation Routing**: "For each candidate, ask first whether a hook, lint rule, or CI check enforces it better than prose; if so propose the check, and the line becomes the fallback if they decline. A ledger entry marked `automate` keeps its instruction until its check is in place (a later run deletes the line under ground 2 once the check is live)." (`SKILL.md`, line 66).
4. **Ledger-governed Deletion and Pruning (`Audit`)**: Every existing human-written instruction enters a ledger (`retain | rewrite | relocate | automate | delete`). Deletion requires one of four grounds (`references/best-practices.md`, lines 77–80): (1) Stale or incorrect, (2) Mechanically enforced by a live hook/linter/CI check, (3) Harmful or contradictory, or (4) Explicit line-item user approval. `Audit` mode re-verifies every path and command claim against `git log --diff-filter=DR --name-only` since the recorded provenance SHA and guarantees: "Audit ends smaller or equal." (`SKILL.md`, lines 85, 111).
5. **Observed-Mistake Gate (`Record`)**: "Capture one observed agent mistake as it happens — the only admissible source for a pitfall... One occurrence is noted; a recurring or costly mistake earns a line now" (`SKILL.md`, lines 103–105).

### Superpowers' Instruction Architecture (`clones/superpowers/`)

1. **Skill-First Bootstrap (`using-superpowers`)**: Rather than packing procedural workflows into `CLAUDE.md`, Superpowers injects a 63-line bootstrap skill (`clones/superpowers/skills/using-superpowers/SKILL.md`) that mandates checking and invoking task-specific skills before any action, while preserving explicit user hierarchy: "User instructions (CLAUDE.md, AGENTS.md, GEMINI.md, etc, direct requests) take precedence over skills, which in turn override default behavior." (line 63).
2. **Where Conventions Live (`writing-skills`)**: `clones/superpowers/skills/writing-skills/SKILL.md` (lines 58–59) separates reusable techniques (which belong in skills) from "Project-specific conventions (put in your instructions file)" and "Mechanical constraints (if it's enforceable with regex/validation, automate it—save documentation for judgment calls)." Heavy reference material (`100+ lines`) is split into separate files (line 85).
3. **Platform-Neutral Instruction References**: In `clones/superpowers/docs/superpowers/specs/2026-05-05-platform-neutral-config-refs-design.md` (lines 5–7, 37), Superpowers replaced hardcoded `CLAUDE.md` assumptions inside skills with generic references ("your instructions file") mapped per harness (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`), and symlinked `AGENTS.md` to `CLAUDE.md` at the repo root (`RELEASE-NOTES.md`, line 242).
4. **Contributor Gate in Repo `CLAUDE.md`**: Superpowers uses its own repository `CLAUDE.md` (`clones/superpowers/CLAUDE.md`, lines 3–20, `## If You Are an AI Agent`) as a high-salience contributor gate against automated PR slop ("This repo has a 94% PR rejection rate..."), requiring human diff review, environment disclosure, and end-to-end session transcripts for new harness integrations.

---

## 3. Owner Workspace Comparison: `ssot-manager`

Examined at `/Users/casentino/Developer/hwahae/workspace/ssot-manager` (`context.md`, `interview.md`, `AGENTS.md`, `CLAUDE.md`):

- **Purpose & Architecture (`context.md`, lines 7–19)**: A local Next.js (`127.0.0.1:6427`) Claude Code plugin that turns markdown PRDs into a verified React Flow Single Source of Truth (SSOT) graph (`AnalysisJobFlowGraphSchema`), a design-conformance coverage matrix (`CoverageJob`, lines 127–139), and a derived Domain Entity Relationship Diagram (`DiagramJob`, lines 78–98).
- **Zero-API-Key Local CLI Delegation (`context.md`, lines 14–19, 35–43)**: Authentication reuses `gh auth token` via local `gh` CLI; LLM inference delegates to the user's logged-in `claude` CLI session either via manual slash-command handoff (`/ssot-manager run <jobId>`) or headless spawn (`execFile` running `claude --dangerously-skip-permissions -p "/ssot-manager run <jobId>"`).
- **Deterministic Pre-LLM PRD Detection (`context.md`, lines 20–25)**: `prd-detection.ts` (`scorePrdCandidate`) scores repository markdown files deterministically (0–1 confidence from path, frontmatter, headings, keywords, structure) without an LLM call, pre-selecting up to 3 PRD candidates.
- **Boundary Schema Validation & Verbatim Citation Grounding (`context.md`, lines 48–55, 85, 132)**:
  - Graph outputs are validated with Zod (`AnalysisJobFlowGraphSchema`, `ErdGraphSchema`) at job completion; node detail fields and edge relationship types remain `.optional()` with render fallbacks (`normalizeRelationshipType` → `"related"`) so minor LLM omissions do not blank the canvas (`status: "invalid"`).
  - Both `DiagramJob` (`normalizeErdGraph` in `diagram-normalize.ts`) and `CoverageJob` (`normalizeCoverageRecord` in `coverage-normalize.ts`) enforce **fail-closed verbatim citation anchoring** via `hasVerbatimCitation` (`evidence-verbatim.ts`): every entity, attribute, relationship, or coverage claim must carry an `evidence.quotedSpan` that is a verbatim substring of `job.sourceDocuments`, or the item is dropped/downgraded (`invalid`/`absent-candidate`).
- **Instruction Partitioning in `ssot-manager`**:
  - `CLAUDE.md` holds the Ouroboros specification-first workflow block plus project-local autonomous PR/merge gates (`pnpm typecheck && pnpm lint && pnpm test`).
  - `AGENTS.md` delegates all frontend coding rules to a dedicated rulebook (`docs/frontend-conventions.md`, 20,862 bytes) with structured violation reporting (`RULE-ID @ path:line — <violation> → <fix direction>`), while architectural rationale lives in `context.md` (40,151 bytes).
