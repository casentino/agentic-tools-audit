# Agentic Tools Audit Wiki Design

## Purpose

Build a personal knowledge library and benchmark of agentic tools that feeds the owner's Claude Code plugin development. The library exists to answer design questions, not to survey the market. Every record must earn its place by informing a plugin decision.

The wiki answers questions such as:

- What does each tool treat as its unit of extension, and when does it load one?
- How does an agent learn that a given extension applies right now?
- How do tools budget context across many extensions?
- How do they resolve conflicts when several extensions apply at once?
- Which of these techniques can move into my own plugins, and at what cost?

## Scope

The wiki covers agentic CLIs and any tool that exposes an extension mechanism, including IDE extensions and agent frameworks. Two methods produce its records: static measurement of ecosystem implementations, and qualitative scoring against a fixed rubric.

Three things stay out.

- **Live task experiments.** Running the same task across tools would produce the strongest evidence and the largest bill. Tool accounts, replay fixtures, and per-run variance all cost more than the conclusions are worth here.
- **Self-measurement of the owner's plugins.** Session logs and token counts belong to the metrics plugin, not to this wiki.
- **Adoption recommendations.** Nothing here decides what a team should use.

The first pass covers three to five seed tools. Breadth stays available; the work does not attempt it at once.

## Repository Boundary

This repository owns the wiki and its Git history. The wiki lives at `.wiki/` as an llm-wiki local wiki, and the hub registers it in `local_wikis` with an absolute path:

```json
{
  "hub_path": "/Users/kikyeongoh/Documents/opterk/llm-wiki"
}
```

### Why a separate repository

The hub already holds `topics/experience-knowledge`, whose raw layer contains company tickets and Notion snapshots. That material must stay private forever. This wiki analyzes public tools and may itself become public — the repository already has the remote `casentino/agentic-tools-audit`.

Merging the two closes the public option permanently. Separating them now costs a directory move if the owner later wants one repository. The asymmetry decides it: prefer the reversible arrangement.

Repository size reinforces the choice. Ingesting ecosystem repositories adds raw files by the hundred, and those clone and pull costs have no reason to fall on private work records.

### What the separation costs

- **Bulk commands skip this wiki.** `--wiki all` iterates registered topic wikis under the hub. A local wiki is not among them, so refresh runs once per wiki instead.
- **`local_wikis` support is thin.** llm-wiki 0.16.0 mentions the field in exactly one place, the registration step in `commands/wiki.md`. Registration records the wiki; no command promises to traverse it.
- **Dual links cannot cross the boundary.** Relative paths break between repositories. Cross-references to `experience-knowledge` take the form of source notes, not links.
- **Commands depend on the working directory.** Every `/wiki` invocation runs inside this repository with `--local`.

These costs are acceptable. The wiki count stays at two, so running commands twice replaces bulk iteration.

### Why `.wiki/` and not the repository root

Every command resolves `--local` to `.wiki/` in the current directory. Placing the wiki at the repository root would break that resolution and silence the tooling. `README.md` points readers from the root into `.wiki/`.

## Structure

```text
agentic-tools-audit/
├── AGENTS.md                      contributor guidelines
├── CLAUDE.md                      agent guidance; points into .wiki/
├── README.md                      what this repository is, and its commands
├── pyproject.toml                 ruff and pytest configuration
├── docs/superpowers/specs/        this design and its successors
└── .wiki/
    ├── _index.md                  statistics, navigation, recent changes
    ├── config.md                  title, scope, conventions
    ├── schema.md                  rubric axes, entity types, evidence rules
    ├── log.md                     append-only activity log
    ├── inbox/
    ├── raw/
    │   ├── repos/                 collection manifests
    │   └── articles/              ingested documents
    ├── wiki/
    │   ├── topics/<tool>.md       tool profiles
    │   ├── concepts/<pattern>.md  pattern cards
    │   └── references/            rubric scoreboard
    ├── inventory/
    │   ├── corpora/               one record per ingested collection
    │   └── candidates/            plugin backlog items
    └── output/projects/ecosystem-metrics/
        ├── WHY.md
        ├── code/                  measurement scripts and tests
        └── data/                  measurement results
```

## Record Types

| Record | Location | Frontmatter | Answers |
|--------|----------|-------------|---------|
| Tool profile | `wiki/topics/<tool>.md` | `category: topic`, `volatility: hot` | How does this tool express extensions? |
| Pattern card | `wiki/concepts/<pattern>.md` | `category: concept`, `volatility: warm` | How does this technique move into my plugins? |
| Scoreboard | `wiki/references/rubric-scoreboard.md` | `category: reference` | How do tools compare across axes? |
| Collection | `inventory/corpora/<slug>.md` | — | What did this ingest bring in? |
| Backlog item | `inventory/candidates/<slug>.md` | — | What should I change in my plugins? |

Each type answers one question and leaves the others alone. Profiles describe; they do not teach techniques. Pattern cards teach; they do not introduce tools.

Profiles carry five sections: the extension model (unit, loading, injection point), rubric scores with a one-line justification per axis, a link to measurements when they exist, a portability note of three lines or fewer, and Sources.

Pattern cards carry six: the problem, the technique with a minimal example, at least two sightings with exact paths, porting cost and risk, the proposed change with a link to its backlog item, and Sources.

The scoreboard is written by hand from the profiles. Generating it requires measurement coverage the early milestones do not have, and a hand-written table of five tools costs less than the script that would build it.

## Rubric

`schema.md` defines six axes. Each axis states a plugin design question.

| Axis | Question |
|------|----------|
| `discoverability` | How does the agent learn that this extension applies now? |
| `context-budget` | How much context does an extension consume, and how does the tool defer or stage it? |
| `composition` | How does the tool order and reconcile overlapping extensions? |
| `state` | Where does state that outlives a session live? |
| `side-effect-control` | How does the tool gate permissions and isolate effects? |
| `observability` | What logs, metrics, and failure diagnostics exist? |

Scores run 0 to 3:

- **0** — no such mechanism
- **1** — implicit; the convention exists only in practice
- **2** — documented convention
- **3** — the tool enforces or verifies it

## Vocabulary

`schema.md` adds local entity types: `tool`, `extension-unit` (whatever the tool treats as an extension — skill, rule, agent, hook, node), `mechanism`, `pattern`, `axis`, and `candidate`. It adds four relationship verbs to the global set: `evaluates` (profile to axis), `implements` (tool to pattern), `informs` (pattern to candidate), and `measured-by` (profile to measurement data). It redefines no global primitive.

## Evidence Rules

- Official documentation and source code carry the argument. Blog posts and social media support it.
- Every profile records the version examined. When a tool publishes no version, record the commit SHA.
- **A pattern card requires sightings in at least two places.** A single sighting stays in its profile. This rule is the only defense against unbounded card growth.
- The owner's plugin sources stay in their own repositories. Reference their paths; never copy them into `raw/`.
- Mark inferences as inferences. Never present one as a recorded fact.

## Index Discipline

The master index lists collections, never individual raw files. One `inventory/corpora/<slug>.md` represents each ingested collection, so hundreds of raw files occupy a few lines of index.

The hub shows why this matters. In `topics/experience-knowledge`, 242 sources have grown `raw/_index.md` to 91 KB while the master `_index.md` holds at 1.8 KB. Cross-wiki peeks read only the master index, so the cost stays hidden until a workflow walks the raw index. Ingesting ecosystem repositories would reproduce the same growth here within a few collections.

## Measurement Pipeline

Measurement reads raw files that `/wiki:ingest-collection --adapter git` has already ingested with `revision`, `sha`, and `canonical_url` provenance. The scripts touch no network and clone nothing, so the same raw layer always yields the same numbers.

The pipeline measures only what maps to a rubric axis: extension-unit counts, token-length distribution per unit, description length and trigger phrasing (`discoverability`), separation of reference files (`context-budget`), and declared hooks and permissions (`side-effect-control`). The remaining axes resist measurement and stay with the rubric.

Two failures can occur: a raw file whose structure differs from expectation, and a metric that cannot be derived. Both record `null` with a reason and continue. Ecosystem repositories break conventions routinely, and one irregular repository must not halt the run. Results carry a `skipped` list so omissions stay visible.

Tests fix the input and output of each measurement function against minimal raw samples under `code/tests/fixtures/`. No test reaches a real ecosystem repository.

## Toolchain Obligations

`AGENTS.md` requires that the change introducing a language also make its commands reproducible. The commit that adds Python therefore:

1. Commits `ruff` and `pytest` configuration in `pyproject.toml` at the repository root, with both tools pointed at `.wiki/output/projects/ecosystem-metrics/code/`.
2. Adds the single entry point `.wiki/output/projects/ecosystem-metrics/code/measure.py`, invoked as `python3 <path>/measure.py <subcommand>` after the pattern the hub's `session_corpus.py` already sets.
3. Documents `ruff check .`, `pytest`, and that entry point in `README.md`.
4. Replaces the placeholder command section in `AGENTS.md`.
5. Updates the bootstrap section of `CLAUDE.md`.

### Two `AGENTS.md` rules need narrowing

Both conflicts arise because `AGENTS.md` was written for a conventional application repository, and this one hosts a wiki whose layout llm-wiki dictates.

**Generated output.** `AGENTS.md` forbids committing generated output and local audit results. Measurement results are the record, and llm-wiki commits `output/projects/<slug>/data/` by convention. Discarding them would erase the wiki's time axis. Narrow the rule to raw collection material: under `.wiki/output/projects/ecosystem-metrics/data/`, measurement summaries stay committed, and anything re-derivable by re-ingesting stays ignored.

**Code and test locations.** `AGENTS.md` places code in `src/` and tests in `tests/`. llm-wiki places project code in `output/projects/<slug>/code/`, and the hub's own `session-corpus` project follows that layout. Two competing trees would leave every future reader guessing. Grant the wiki subtree an explicit exemption: inside `.wiki/`, llm-wiki conventions govern, and `src/` and `tests/` apply only to code outside it. This repository expects no such code.

## Milestones

1. **Initialize.** Create `.wiki/` with `/wiki init --local`, write `config.md` and `schema.md` including the six axes, and register the wiki in the hub.
2. **Score three to five seed tools.** Write profiles with rubric scores and no measurement. This step tests whether the axes survive contact with real tools.
3. **Promote patterns.** Convert techniques sighted in two or more profiles into pattern cards, each linked to a backlog candidate.
4. **Measure.** Build the measurement pipeline once the axes hold still.

Step 2 precedes step 4 deliberately. Measurement code depends on the axes, and axes shift when they first meet real tools. Writing the pipeline early would mean writing it twice.

### Plan boundary

The implementation plan that follows this spec covers steps 1 through 3. Step 4 gets its own spec, written once the seed profiles show which axes survived and which measurements they actually need. Planning the pipeline now would commit to axis definitions that step 2 exists to test.
