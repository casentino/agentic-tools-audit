---
title: "LangGraph Extension Model"
source: "https://langchain-ai.github.io/langgraph/"
type: notes
ingested: 2026-08-04
tags: [langgraph, extension-unit, state]
summary: "Evidence on LangGraph's graph composition, checkpointers, store, and interrupts. Collected from the official documentation."
---

# LangGraph Extension Model

## Version Examined

`1.2.10` — the package is not installed on this machine (`python3 -m pip show langgraph` returned nothing, exit code 1), so the version is the current published release read from the Python Package Index:

```bash
curl -s https://pypi.org/pypi/langgraph/json | python3 -c "import json,sys; print(json.load(sys.stdin)['info']['version'])"
```

Output: `1.2.10`. The frontmatter `source` URL above (`https://langchain-ai.github.io/langgraph/`) is the legacy docs root; fetching it returns an HTTP 200 redirect page whose body reads "The LangGraph documentation has moved to docs.langchain.com," with canonical target `https://docs.langchain.com/oss/python/langgraph/overview`. All documentation quotes below were fetched from that current site (`docs.langchain.com/oss/python/langgraph/*`), verified against the raw HTML response rather than recalled, since a Task 6 briefing note specifically flagged two prior profiles in this series for citation defects.

## Unit of Composition

**Nodes.** "In LangGraph, nodes are Python functions (either synchronous or asynchronous)" registered onto a graph builder with `add_node`; example: `builder.add_node("plain_node", plain_node)`. A node is not a declarative file the framework discovers — it is a Python callable the developer writes and wires in at graph-construction time. (https://docs.langchain.com/oss/python/langgraph/graph-api)

**Edges.** Plain edges are registered with `add_edge("node_a", "node_b")`; conditional edges with `add_conditional_edges("node_a", routing_function)`, where "This method accepts the name of a node and a 'routing function' to call after that node is executed," and by default "the return value `routing_function` is used as the name of the node (or list of nodes) to send the state to next." Routing is therefore developer-written code evaluated by the framework, not a framework-supplied relevance or pattern match. (https://docs.langchain.com/oss/python/langgraph/graph-api)

**Parallel fan-out.** "A node can have multiple outgoing edges. If a node has multiple outgoing edges, **all** of those destination nodes will be executed in parallel as a part of the next superstep." (https://docs.langchain.com/oss/python/langgraph/graph-api)

**Reducers govern state-key composition.** "Each key in the state has an independent reducer function. If no reducer is specified, the system defaults to an override behavior where the new update replaces the existing value" — confirmed on the page itself as: "the default reducer discards the left argument and keeps only the right." A custom reducer is declared by annotating a `TypedDict` field, e.g. `messages: Annotated[list[AnyMessage], add_messages]`, so that concurrent writes to the same key (e.g. two parallel nodes both updating `aggregate`) accumulate via `operator.add` instead of overwriting. No error-on-conflict was found documented for the no-reducer, concurrent-write case — the default overwrite behavior applies uniformly and silently; getting correct accumulation under parallel writes is the developer's responsibility to configure via the right reducer, not something the framework detects or warns about. (https://docs.langchain.com/oss/python/langgraph/graph-api)

**Subgraphs — two distinct composition shapes, table-quoted verbatim:**
- "Add a subgraph as a node" — "Parent and subgraph **share state keys**—the subgraph reads from and writes to the same channels as the parent," and "You pass the compiled subgraph directly to `add_node`—no wrapper function needed."
- "Call a subgraph inside a node" — used when "Parent and subgraph have **different state schemas** (no shared keys), or you need to transform state between them," in which case "You write a wrapper function that maps parent state to subgraph input and subgraph output back to parent state."
(https://docs.langchain.com/oss/python/langgraph/use-subgraphs)

**Dynamic dispatch.** For map-reduce-shaped fan-out where "the number of edges may not be known ahead of time," a conditional edge can return `Send` objects, e.g. `[Send("generate_joke", {"subject": s}) for s in state["subjects"]]`, each carrying its own per-branch input state. (https://docs.langchain.com/oss/python/langgraph/graph-api)

## Persistence

LangGraph documents two separate, complementary persistence systems, stated plainly: "LangGraph provides two complementary persistence systems:
- **Checkpointers** persist a thread's graph state as checkpoints. Use them for short-term, thread-scoped memory, including conversation continuity, human-in-the-loop workflows, time travel, and fault tolerance.
- **Stores** persist application-defined data outside the graph state. Use them for long-term, cross-thread memory, including user preferences, facts, and shared knowledge." (https://docs.langchain.com/oss/python/langgraph/persistence)

**Checkpointer — what it saves, and at what granularity.** "A checkpointer saves a snapshot of graph state at each super-step, organized into **threads**. Compile a graph with a checkpointer to enable human-in-the-loop workflows, time travel debugging, fault-tolerant execution, and conversational memory." A super-step is defined precisely: "A super-step is a single 'tick' of the graph where all nodes scheduled for that step execute (potentially in parallel). For a sequential graph like `START -> A -> B -> END`, there are separate super-steps for the input, node A, and node B — producing a checkpoint after each one." (https://docs.langchain.com/oss/python/langgraph/checkpointers)

**Threads.** "A thread is a unique ID or thread identifier assigned to each checkpoint saved by a checkpointer. It contains the accumulated state of a sequence of runs." A `thread_id` in the `configurable` portion of the run config is the primary key the checkpointer uses to store and retrieve checkpoints; `graph.get_state(config)` and `checkpointer.get_tuple(config)` return the full `StateSnapshot`/`CheckpointTuple` for a thread, optionally at a specific historical `checkpoint_id` — the time-travel capability. (https://docs.langchain.com/oss/python/langgraph/checkpointers, https://docs.langchain.com/oss/python/langgraph/add-memory)

**Backends.** Named checkpointer implementations: `InMemorySaver` (built in, process-local), `SqliteSaver`/`AsyncSqliteSaver`, `PostgresSaver`/`AsyncPostgresSaver`, and Redis/CosmosDB variants shown in examples (`AsyncRedisSaver`). (https://docs.langchain.com/oss/python/langgraph/checkpointers, https://docs.langchain.com/oss/python/langgraph/add-memory)

**Durability modes — a tool-enforced consistency/performance tradeoff, set per call.** Three modes, quoted verbatim: `"exit"`: "LangGraph persists changes only when graph execution exits — successfully, with an error, or due to a human-in-the-loop interrupt. This provides the best performance for long-running graphs but means intermediate state is not saved, so you cannot recover from system failures (like process crashes) mid-execution." `"async"`: "LangGraph persists changes asynchronously while the next step [runs]" — good performance, small crash-window risk. `"sync"`: persists changes synchronously before each step, for high durability at a performance cost. Set via `graph.stream({"input": "test"}, durability="sync")`. (https://docs.langchain.com/oss/python/langgraph/checkpointers)

**Store — long-term, cross-thread memory, structurally distinct from a checkpointer.** "Stores let agents persist information across threads, including user preferences, accumulated knowledge, and facts that should survive beyond a single conversation. Unlike checkpointers, which save the full graph state scoped to one thread, stores hold arbitrary key-value data accessible from any thread." Data is organized as key-value items inside a `namespace` (a tuple of strings), each item also carrying `created_at`/`updated_at` timestamps. Named backends: "For production, use a persistent store like `PostgresStore`, `MongoDBStore`, or `RedisStore`. All implementations extend `BaseStore`," while `InMemoryStore` "is suitable for development and testing." Stores additionally "support semantic search, allowing you to find memories based on meaning rather than exact matches," when configured with an embedding model. (https://docs.langchain.com/oss/python/langgraph/stores)

**Context-window management is a documented pattern set, not an automatic cap.** "Most LLMs have a maximum context window... To address this, common solutions include trimming messages, deleting messages from the LangGraph state, summarizing earlier messages, or managing checkpoints" — four named solutions in that sentence. Of those, three are demonstrated as concrete utilities/patterns elsewhere on the same page: the `trim_messages` utility trims by token count and strategy (e.g. `strategy="last"`, `max_tokens=128`); `RemoveMessage(id=...)` deletes specific messages from state; a `summarize_conversation` node pattern replaces old messages with a running summary. Each of these three requires the developer to write and wire in the corresponding node/logic — none is applied by the framework automatically or by default. (https://docs.langchain.com/oss/python/langgraph/add-memory)

## Interrupts and Gates

**Three requirements to pause a run, quoted verbatim as a numbered list:** "To use `interrupt`, you need:
1. A **checkpointer** to persist the graph state (use a durable checkpointer in production)
2. A **thread ID** in your config so the runtime knows which state to resume from
3. To call `interrupt()` where you want to pause (payload must be JSON-serializable)"
(https://docs.langchain.com/oss/python/langgraph/interrupts)

**Resume mechanics.** A paused run resumes by invoking the graph again with `Command(resume=<value>)` against the same `thread_id`; the value returned by `interrupt()` inside the node becomes that resume value. Multiple simultaneous interrupts (parallel branches) are resumed by id: `Command(resume={interrupt_id: answer, ...})`.

**Re-execution semantics — a real, documented gotcha.** "the runtime restarts the entire node from the beginning—it does not resume from the exact line where `interrupt` was called. This means any code that ran before the `interrupt` will execute again." Any side effect placed before an `interrupt()` call inside the same node will therefore re-run on resume unless the developer makes it idempotent or moves it after the interrupt.

**Serialization constraint.** The interrupt payload and the resume value must be JSON-serializable; passing a class instance is documented as failing ("The instance cannot be serialized"), with the recommended pattern being plain dictionaries of simple values.

**Dynamic interrupts are per-call-site, developer-placed code**, most commonly wrapping a tool: `interrupt({"action": "send_email", ..., "message": "Approve sending this email?"})` inside the tool body, gating that specific side effect only if the developer chose to instrument that tool. Nothing in the framework auto-detects which tool calls are "dangerous" or requires approval by default — every gate is opt-in, one call site at a time.

**Static interrupts are the one part of this mechanism the framework itself enforces without per-call-site code.** "You can use static interrupts as breakpoints to step through the graph execution one node at a time. Static interrupts are triggered at defined points either before or after a node executes. You can set these by specifying `interrupt_before` and `interrupt_after` when compiling the graph" — e.g. `builder.compile(interrupt_before=["node_a"], interrupt_after=["node_b", "node_c"], checkpointer=checkpointer)`. This is a compile-time, tool-checked list of node names; the framework itself pauses before/after the named node runs, with no code required inside that node. It is still binary (a full stop before/after a whole node, not a scoped gate on one specific side-effecting action within it) and still requires the developer to name the nodes.

(https://docs.langchain.com/oss/python/langgraph/interrupts)

## Tracing

**Built-in, local, no external account required.** `graph.stream(...)`/`graph.astream(...)` accept a `stream_mode` argument with several documented values — `"values"` (full state after each step), `"updates"` (only the state changes per node), `"messages"` (LLM token/metadata tuples), `"custom"` (developer-emitted data), `"checkpoints"` and `"tasks"` (require a checkpointer; track state history and execution events), and `"debug"`. "Use the `debug` streaming mode to stream as much information as possible throughout the execution of the graph. The streamed outputs include the name of the node as well as the full state." Precisely: "The `debug` mode combines `checkpoints` and `tasks` events with additional metadata. Use `checkpoints` or `tasks` directly if you only need a subset of the debug information." (https://docs.langchain.com/oss/python/langgraph/streaming)

**LangSmith — the fuller tracing/visualization product, explicitly separate and opt-in.** "Traces are a series of steps that your application takes to go from input to output. Each of these individual steps is represented by a run." Using it requires an external account: "Sign up (for free) or log in at smith.langchain.com" and "A LangSmith API key." Once enabled, it lets a developer "visualize these execution steps" to "Debug a locally running application," "Evaluate the application performance," and "Monitor the application." LangSmith Studio additionally "provides a UI to set static interrupts in a graph before execution and to inspect the graph state at any point during its run." The graph-api page itself frames this division of labor: "For observability, tracing, and evaluation of agents, LangSmith is the recommended tool for monitoring LangGraph workflows" — i.e. the richest inspection experience is a separate hosted product layered on top of the framework's own stream-mode primitives, not something the open-source framework provides natively end-to-end. (https://docs.langchain.com/oss/python/langgraph/observability, https://docs.langchain.com/oss/python/langgraph/interrupts, https://docs.langchain.com/oss/python/langgraph/graph-api)

## Sources

- https://docs.langchain.com/oss/python/langgraph/graph-api — node/edge registration (`add_node`, `add_edge`, `add_conditional_edges`), routing-function semantics, parallel-outgoing-edge/superstep behavior, reducers (default-overwrite and custom), the `Send` API for dynamic dispatch, and the pointer to LangSmith for observability.
- https://docs.langchain.com/oss/python/langgraph/use-subgraphs — the two documented subgraph-composition shapes (shared-state-key node embedding vs. wrapper-function invocation) and their exact trigger conditions.
- https://docs.langchain.com/oss/python/langgraph/checkpointers — checkpointer definition, super-step definition, thread definition, the three durability modes (`exit`/`async`/`sync`) quoted verbatim, and named backend implementations.
- https://docs.langchain.com/oss/python/langgraph/persistence — the checkpointer-vs-store framing ("two complementary persistence systems"), quoted in full.
- https://docs.langchain.com/oss/python/langgraph/stores — store definition, namespace/key/value/timestamp structure, named backends, and semantic search.
- https://docs.langchain.com/oss/python/langgraph/interrupts — the three requirements for `interrupt()`, the node-restarts-from-the-beginning resume semantics, the serialization constraint, and static `interrupt_before`/`interrupt_after` breakpoints.
- https://docs.langchain.com/oss/python/langgraph/add-memory — `trim_messages`, `RemoveMessage`, and the summarization pattern for context-window management; thread/checkpoint state-inspection APIs (`get_state`, `checkpointer.get_tuple`).
- https://docs.langchain.com/oss/python/langgraph/observability — the trace/run definitions, the LangSmith sign-up/API-key requirement, and the visualize/debug/evaluate/monitor capability list.
- https://docs.langchain.com/oss/python/langgraph/streaming — the `stream_mode` values, and the `debug` mode's exact relationship to `checkpoints`/`tasks`.
