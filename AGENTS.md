# Repository Guidelines

## Project Structure & Module Organization

This repository is in its bootstrap state: it contains no tracked source, tests, assets, or build configuration. Keep the root for project-wide files such as `README.md`, manifests, and tool configuration. When implementation begins, place code in `src/`, tests in `tests/`, documentation in `docs/`, and sanitized test data in `tests/fixtures/`. Organize modules by responsibility, and avoid committing generated output, dependency directories, or local audit results.

## Build, Test, and Development Commands

No build or test toolchain exists yet. Use these repository checks now:

- `git status --short` — review staged, modified, and untracked files.
- `git diff --check` — detect whitespace errors before committing.

The first change that adds a language or framework must also add reproducible scripts and document them in `README.md` (for example, `npm test` or `pytest`). Update this guide when canonical commands become available; do not rely on undocumented local aliases.

## Coding Style & Naming Conventions

Use UTF-8, final newlines, and spaces instead of tabs unless the selected language requires otherwise. Indent Markdown, YAML, and JSON with two spaces. Adopt the ecosystem's standard formatter and linter with the first source module, commit their configuration, and run them before opening a pull request. Prefer descriptive names: `kebab-case` for documentation and shell files, `snake_case` for Python modules, and `PascalCase` for exported types or classes where the language convention supports it.

## Testing Guidelines

Add tests with every feature and bug fix. Mirror `src/` organization under `tests/`, and use the framework's conventional names, such as `test_*.py` or `*.test.ts`. Cover success, failure, and boundary cases. Audit fixtures must be minimal, deterministic, and stripped of credentials or customer data. Until a coverage target is adopted, prioritize meaningful assertions over a numeric threshold.

## Commit & Pull Request Guidelines

The repository has no commit history from which to infer a house style. Use concise Conventional Commit subjects, such as `feat: add tool inventory scanner` or `docs: define audit workflow`. Keep each commit focused. Pull requests should explain the problem and solution, list validation performed, link related issues, and note follow-up work. Include screenshots only for user-visible changes and call out configuration or security implications explicitly.

## Security & Configuration

Never commit secrets, tokens, private keys, raw customer data, or populated `.env` files. Provide `.env.example` with placeholder values for required settings, and redact sensitive fields from logs and audit artifacts before sharing them.
