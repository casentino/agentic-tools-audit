---
title: "LangGraph"
category: topic
sources: ["raw/notes/2026-08-04-langgraph-extension-model.md"]
created: 2026-08-04
updated: 2026-08-04
tags: [langgraph, agent-framework, state]
aliases: ["LangGraph framework"]
confidence: high
volatility: hot
verified: 2026-08-04
summary: "LangGraph's graph-based extension model and its scores across the six audit axes."
---

# LangGraph

> LangGraph treats an extension as a node — a Python function wired into a graph with `add_node`/`add_edge`/`add_conditional_edges` and fixed at `compile()` time — plus two separate, first-class persistence primitives (a thread-scoped checkpointer and a cross-thread store) that a plugin author would otherwise have to bolt on themselves; its answer to `state` differs from the three CLI-shaped tools in this audit because persistence here is not a bounded file the tool manages on the developer's behalf (an auto-memory cap, a rule file) but a versioned, super-step-granular snapshot system with named durable backends and a tunable consistency/performance tradeoff (`durability="exit"|"async"|"sync"`) — a mechanism engineered for long-running, resumable, human-gated runs rather than for a single chat session.

## Extension Model

**Unit:** A node — a Python function (sync or async) registered with `add_node`. Nodes are connected by edges (`add_edge`, or `add_conditional_edges` with a developer-written routing function) and can themselves be compiled subgraphs, embedded either directly (sharing state keys with the parent) or through a wrapper function (when state schemas differ).

**Loading:** Nothing is discovered or loaded at runtime the way a skill description or hook config is. The entire topology — every node, every edge, every routing function — is Python code the developer writes and wires together before calling `.compile()`; once compiled, the graph's shape is fixed for that process. What does happen at runtime is state flowing through channels: each node receives the current state and returns a partial update, and a per-key reducer function (custom, or the documented default of "discards the left argument and keeps only the right") decides how that update merges into accumulated state at each super-step.

**Injection point:** A node's own code decides what enters an LLM's context — there is no framework-level analog to "skill body attaches at invocation." Long-term memory is explicit and pull-based: a node calls `runtime.store.asearch(namespace, query=...)` and formats the result into a prompt itself; the store does not push anything into context automatically. Human-in-the-loop payloads surface through `interrupt()`/`Command(resume=...)`, and are similarly whatever JSON-serializable payload the developer chose to pass.

This is the central shape mismatch this audit has to name plainly: LangGraph is a library, not a host with a fixed extension surface, and its "extension unit" is ordinary application code rather than a declarative file a runtime discovers and judges the relevance of. Three axes read almost unrecognizably as a result. `discoverability` — "how does the agent learn that this extension applies now?" — presumes a host deciding whether some external, author-supplied unit is relevant to the current turn; LangGraph has no such moment for nodes at all, because the developer's own code *is* the routing decision (a conditional edge's routing function, evaluated deterministically), not something the framework judges on the agent's behalf. `context-budget` and `side-effect-control` likewise resolve to "here is a well-documented utility function/primitive (`trim_messages`, `interrupt()`) you may call," not "the framework enforces a bound or a gate by default" — and per this audit's own rule against inflating scores by treating "you can write code that does this" as the framework enforcing it, that difference is scored, not glossed over. `state` is the one axis where the library shape is a genuine advantage rather than a mismatch: persistence here is not incidental scaffolding bolted onto a chat loop but an engineered subsystem (checkpointer + store, both with defined schemas, named durable backends, and a documented consistency knob) that exists specifically because LangGraph is built for runs that pause, crash, resume, and outlive a single conversation.

## Rubric Scores

Version examined: 1.2.10 (PyPI current release; not installed locally — `python3 -m pip show langgraph` returned nothing, so the version was read via `curl -s https://pypi.org/pypi/langgraph/json | python3 -c "import json,sys; print(json.load(sys.stdin)['info']['version'])"`)

| Axis | Score | Justification |
|------|-------|---------------|
| `discoverability` | 1 | No framework mechanism judges whether a node "applies now" — topology is developer-written code fixed at `compile()` time, not something the tool discovers or scores relevance for; the closest analog, an LLM choosing among bound tools inside a node, is ordinary unverified model judgment and not a LangGraph-specific mechanism. |
| `context-budget` | 2 | `trim_messages`, `RemoveMessage`, and a summarization node pattern are specifically documented for managing context size, but all three are opt-in code the developer must wire into a node — nothing is trimmed, deleted, or summarized by the framework automatically or by default. |
| `composition` | 3 | Every state key has a framework-executed reconciliation rule — a custom reducer or the documented default overwrite — applied deterministically at each super-step, and subgraph key-sharing is an explicit, schema-checked choice (shared keys vs. wrapper-function translation); the residual gap is that omitting a reducer for concurrent same-key writes is silently accepted with no error, a configuration footgun rather than a case the mechanism fails to resolve. |
| `state` | 3 | Two engineered, precisely defined persistence systems — a checkpointer (thread-scoped, super-step-granular snapshots, named durable backends, a tunable `exit`/`async`/`sync` durability mode) and a store (cross-thread, namespaced key-value with timestamps, named durable backends) — with `interrupt()` itself runtime-requiring a checkpointer and thread ID before it will even run. |
| `side-effect-control` | 2 | `interrupt()` plus compile-time `interrupt_before`/`interrupt_after` breakpoints are a real, checkpoint-backed pause/resume/approval primitive, but every dynamic gate is single-call-site opt-in code the developer must add — there is no default-deny, no built-in classifier of risky actions, and no sandboxing, so nothing is gated unless a developer explicitly instruments that exact call site. |
| `observability` | 2 | Built-in, local, no-account `stream_mode` values (`debug`, `checkpoints`, `tasks`, `values`, `updates`, `messages`, `custom`) give real structured execution visibility without any external tool, but the fullest tracing/visualization/evaluation experience (LangSmith) is an explicitly separate, sign-up-gated product the docs themselves point to as "the recommended tool for monitoring LangGraph workflows," not something the open-source framework provides natively end-to-end. |

Reads used, stated per this series' aggregation convention (`discoverability` ceiling, the other five floor): `discoverability` is a ceiling read, and even the best-case mechanism available (an LLM's ordinary tool-calling judgment inside a node) is unverified model judgment, so the ceiling still lands at `1`, not higher — there is no stronger mechanism to take the ceiling from. `context-budget`, `side-effect-control`, and `observability` are floor reads: each has a real, well-documented primitive, but each also has a confirmed, stated gap (no automatic enforcement; no default gating; the deepest tooling is an external product) that caps the score at `2` rather than `3`. `composition` and `state` are also floor reads, but in both cases no confirmed gap was found in the mechanism's own coverage — every overlapping write always resolves to something, and both persistence systems are fully specified with named durable backends — so the floor read still lands at `3`.

## Portability

The persistence pair — a thread-scoped, super-step-checkpointed history plus a separately namespaced cross-thread store, with a documented `exit`/`async`/`sync` durability knob — is the clearest thing worth porting: a Claude Code plugin author building anything that must survive a crash or a long pause has a concrete schema to copy rather than inventing one. The static `interrupt_before`/`interrupt_after` compile-time breakpoint list is a second candidate: a tool-checked "pause before/after this named unit runs" primitive that needs no per-call-site code, closer in kind to a hook matcher than to `interrupt()`'s opt-in, per-call-site approval pattern.

## Sources

- [LangGraph Extension Model](../../raw/notes/2026-08-04-langgraph-extension-model.md) — graph composition, checkpointers, store, interrupts
