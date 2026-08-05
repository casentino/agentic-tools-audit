# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository state

This repository holds a knowledge wiki at `.wiki/`, an llm-wiki local wiki registered in the hub at `/Users/kikyeongoh/Documents/opterk/llm-wiki` under `local_wikis`. Start at `.wiki/_index.md`; the rubric and evidence rules live in `.wiki/schema.md`.

There is still no build or test toolchain. The only repository-level checks are `git status --short` and `git diff --check`. Wiki operations run through llm-wiki commands with `--local` from the repository root.

Design specs and implementation plans live in `docs/superpowers/`.

## AGENTS.md is authoritative

Read `AGENTS.md` before making changes — it is the repository's own contributor guide and takes precedence over generic defaults. Its binding decisions:

- **Planned layout** (create these only when implementation actually begins): code in `src/`, tests mirroring `src/` under `tests/`, sanitized fixtures in `tests/fixtures/`. `docs/` is not planned — it already exists and holds design specs and plans at `docs/superpowers/`. Keep the root for project-wide files (`README.md`, manifests, tool config).
- **Never commit** generated output, dependency directories, or local audit results.
- **Indentation**: two spaces for Markdown, YAML, and JSON. UTF-8, final newline, spaces over tabs unless the language demands otherwise.
- **Naming**: `kebab-case` for docs and shell files, `snake_case` for Python modules, `PascalCase` for exported types/classes.
- **Commits**: Conventional Commit subjects, one focused change per commit.
- **Secrets**: no tokens, private keys, raw customer data, or populated `.env` files. Ship `.env.example` with placeholders instead, and redact sensitive fields from audit artifacts before sharing.

## The first real change carries extra obligations

`AGENTS.md` requires that whichever change first introduces a language or framework must, in the same change:

1. Add the ecosystem's standard formatter and linter, with their config committed.
2. Add reproducible scripts (e.g. `npm test`, `pytest`) — not undocumented local aliases.
3. Document those commands in `README.md`.
4. Update `AGENTS.md`'s "Build, Test, and Development Commands" section to replace the placeholder git-only checks.
5. Update this file's **Repository state** section, and replace it with actual architecture notes once there is architecture to describe.

Tests ship with every feature and bug fix, covering success, failure, and boundary cases. Fixtures must be minimal, deterministic, and stripped of credentials or customer data.
