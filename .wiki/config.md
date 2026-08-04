---
title: "Agentic Tools Audit"
description: "Extension mechanisms, rubric scores, and portable patterns across agentic tools."
created: 2026-08-04
freshness_threshold: 70
---

# Wiki Configuration

## Scope

This wiki covers agentic CLIs and any tool exposing an extension mechanism, including IDE extensions and agent frameworks. It exists to inform the owner's Claude Code plugin development. Records earn their place by answering a plugin design question.

Live task experiments, self-measurement of the owner's plugins, and adoption recommendations stay out of scope.

## Conventions

- Score every profile against all six axes in [schema.md](schema.md).
- Record the version examined, or the commit SHA when the tool publishes no version.
- Promote a technique to a pattern card only after sighting it in two or more places.
- Reference the owner's plugin sources by path. Never copy them into `raw/`.
- List collections in the master index, never individual raw files.
