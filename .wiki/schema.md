---
title: "Agentic Tools Audit Topic Guide"
schema_state: advisory
created: 2026-08-04
updated: 2026-08-04
summary: "Human-owned rubric, vocabulary, and evidence rules for comparing agentic tool extension mechanisms."
---

# Agentic Tools Audit Topic Guide

> This guide adds local vocabulary. It redefines no global llm-wiki primitive — raw source folders, article categories, and required frontmatter keep their standard meanings.

## State

- `schema_state`: `advisory`

## Rubric Axes

Each axis states a plugin design question. Every tool profile scores all six.

| Axis | Question |
|------|----------|
| `discoverability` | How does the agent learn that this extension applies now? |
| `context-budget` | How much context does an extension consume, and how does the tool defer or stage it? |
| `composition` | How does the tool order and reconcile overlapping extensions? |
| `state` | Where does state that outlives a session live? |
| `side-effect-control` | How does the tool gate permissions and isolate effects? |
| `observability` | What logs, metrics, and failure diagnostics exist? |

## Score Scale

| Score | Meaning |
|-------|---------|
| `0` | No such mechanism. |
| `1` | Implicit. The convention exists only in practice. |
| `2` | Documented convention. |
| `3` | The tool enforces or verifies it. |

## Entity Types

| Type | Meaning |
|------|---------|
| `tool` | An agentic tool or extension host. |
| `extension-unit` | Whatever the tool treats as an extension: skill, rule, agent, hook, node. |
| `mechanism` | The machinery that discovers, loads, or runs an extension unit. |
| `pattern` | A portable technique sighted in two or more places. |
| `axis` | A rubric evaluation axis. |
| `candidate` | A proposed change to the owner's plugins. |

## Relationship Verbs

Added to the global set:

- `evaluates` — a profile against an axis
- `implements` — a tool realizing a pattern
- `informs` — a pattern producing a candidate
- `measured-by` — a profile against measurement data

## Source Conventions

- Official documentation and source code carry the argument. Blog posts and social media support it.
- Record the version examined in every profile. When a tool publishes no version, record the commit SHA.
- A pattern card requires sightings in at least two places. A single sighting stays in its profile.
- Reference the owner's plugin sources by path. Never copy them into `raw/`.
- Mark inferences as inferences. Never present one as a recorded fact.

## Article Boundaries

- `wiki/topics/` holds one profile per tool. Profiles describe extension models; they do not teach techniques.
- `wiki/concepts/` holds pattern cards. Cards teach techniques; they do not introduce tools.
- `wiki/references/` holds the rubric scoreboard and any cross-tool index.
- Every factual claim traces to a source listed in the article's Sources section.
