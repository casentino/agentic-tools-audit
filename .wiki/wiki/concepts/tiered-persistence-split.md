---
title: "Tiered Persistence Split"
category: concept
sources: ["raw/notes/2026-08-04-claude-code-extension-model.md", "raw/notes/2026-08-04-codex-cli-extension-model.md", "raw/notes/2026-08-04-langgraph-extension-model.md"]
created: 2026-08-04
updated: 2026-08-04
tags: [pattern, state]
aliases: ["Session Record vs Cross-Session Memory", "Checkpoint/Store Split"]
confidence: high
volatility: warm
verified: 2026-08-04
summary: "Keeping the literal record of what happened in one mechanism, and a separate, purpose-built store of distilled facts that outlive it, in another — rather than one undifferentiated place for both."
---

# Tiered Persistence Split

> Three of the four profiled tools split state that outlives a single interaction into two structurally distinct mechanisms: one that is (or closely tracks) the raw record of the conversation or run, resumable and ordered, and a second, separate mechanism holding distilled facts meant to be pulled into future, unrelated sessions. Keeping these apart is a design choice each tool makes deliberately, not an accident of two features shipping separately.

## Problem

A single undifferentiated persistence layer conflates two different jobs with different failure modes. A raw conversation/run record needs to preserve exactly what happened, in order, resumable from any point — polluting it with synthesized summaries makes it untrustworthy as a record. A cross-session memory store needs to hold only what is worth carrying forward, pruned and current — leaving it entangled with the full transcript makes it slow to read and stale the moment the source conversation is edited or resumed differently. A plugin author who builds one store to serve both jobs (e.g., grepping a transcript file to reconstruct "what does the user usually want") ends up with something that is neither a reliable record nor a useful summary.

## Technique

Maintain two mechanisms with different scopes and different write disciplines: a thread/session-scoped record of what actually happened, and a separate, namespaced store the extension writes to only when it has something durable and cross-cutting worth keeping.

LangGraph states the split as a design principle before naming either half: "LangGraph provides two complementary persistence systems: Checkpointers persist a thread's graph state as checkpoints... Stores persist application-defined data outside the graph state... Use them for long-term, cross-thread memory." A node accesses the store explicitly, by namespace, never automatically:

```python
memories = runtime.store.asearch(namespace, query="user preferences")
```

## Sightings

- **Claude Code** — `~/.claude/projects/<encoded-project-path>/*.jsonl` (session transcripts) vs. `~/.claude/projects/<project>/memory/` (auto memory's `MEMORY.md` + topic files) — the note documents these as two separate mechanisms: transcripts are "each session's JSONL history," while auto memory is described as its own system with its own size cap ("the first 200 lines of `MEMORY.md`, or the first 25KB") and its own write-time `modified` timestamp stamped into frontmatter — distinct from, and not derived by re-reading, the transcript.
- **OpenAI Codex CLI** — https://learn.chatgpt.com/docs/cli/reference (session rollouts: `resume`/`fork`/`archive`/`unarchive`/`delete`) vs. https://learn.chatgpt.com/docs/customization/memories?surface=cli (Memories) — the note states the two are deliberately separated: "a separate 'Memories' system background-summarizes past sessions into `~/.codex/memories/`... so tool-authored persistent memory is opt-in and kept apart from the record of what happened."
- **LangGraph** — https://docs.langchain.com/oss/python/langgraph/persistence — "LangGraph provides two complementary persistence systems: Checkpointers persist a thread's graph state as checkpoints... Stores persist application-defined data outside the graph state... Unlike checkpointers, which save the full graph state scoped to one thread, stores hold arbitrary key-value data accessible from any thread" (https://docs.langchain.com/oss/python/langgraph/stores).

## Porting Cost and Risk

Claude Code already implements one instance of this split at the host level (transcripts vs. auto memory), so the porting cost falls on plugin authors building their *own* stateful features on top of the host, not on the host itself. A plugin that accumulates data across sessions (the owner's `metrics` and `history` plugins are both this shape) should keep its raw per-event log — the append-only record a `PostToolUse`/`Stop` hook writes on every firing — structurally separate from a distilled, queryable summary file, rather than having the dashboard or snapshot generator re-parse the raw log on every read. The cost is a real design/refactor effort: defining what belongs in the raw log versus the summary, and writing (or scheduling) the distillation step, mirroring Codex CLI's explicit choice to make distillation an opt-in, config-gated, background process rather than synchronous per-session work. The risk: two stores drift if the distillation step silently stops running (Codex's own design accepts this risk explicitly, gating memory *use* and memory *generation* as two independently toggleable settings so a stale-but-present memory store never blocks the raw record from being trusted).

## Proposed Change

[[separate-plugin-log-from-memory-store|Separate Plugin Event Log from Cross-Session Memory Store]] ([Separate Plugin Event Log from Cross-Session Memory Store](../../inventory/candidates/separate-plugin-log-from-memory-store.md))

Pick one owner-authored stateful plugin and separate its raw per-event log from a distilled, cross-session summary file, instead of one undifferentiated store serving both.

## See Also

- [[claude-code|Claude Code]] ([Claude Code](../topics/claude-code.md)) — session transcripts vs. auto memory
- [[codex-cli|OpenAI Codex CLI]] ([OpenAI Codex CLI](../topics/codex-cli.md)) — session rollouts vs. the opt-in Memories system
- [[langgraph|LangGraph]] ([LangGraph](../topics/langgraph.md)) — checkpointer vs. store, the clearest engineered version of the split

## Sources

- [Claude Code Extension Model](../../raw/notes/2026-08-04-claude-code-extension-model.md) — first sighting; transcripts vs. auto memory
- [OpenAI Codex CLI Extension Model](../../raw/notes/2026-08-04-codex-cli-extension-model.md) — second sighting; session rollouts vs. the opt-in Memories system
- [LangGraph Extension Model](../../raw/notes/2026-08-04-langgraph-extension-model.md) — third sighting; checkpointer vs. store stated explicitly as "two complementary persistence systems"
