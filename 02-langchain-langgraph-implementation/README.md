# LangChain & LangGraph at the Implementation Level

> Runnables, state machines, durable execution, data cubes, and a catalogue of agentic design patterns — built from the primitives upward, for engineers who need to know how the machine actually runs.

**Author:** Emiliano Roberti · **Level:** Architecture & implementation · **Updated:** May 2026

Most teams reach for an agent framework, wire up a few examples, and ship something that demos well and breaks in production. The break is rarely the model — it is the *runtime*: state that silently overwrites itself, workflows that cannot survive a restart, cycles with no brakes, and tools that hallucinate numbers because nobody put a governed surface between the model and the warehouse.

This document is the engineer's view. It takes the LangChain 1.0 / LangGraph 1.0 stack apart and explains how each layer executes: the **Runnable protocol** and LCEL composition; the **Pregel-style super-step engine** that drives every LangGraph; **StateGraph** state schemas, reducers, nodes and edges; **checkpointers** and durable, resumable execution; **interrupts** and human-in-the-loop; streaming and tracing; tools, the ReAct loop, and structured output. It then presents a **catalogue of agentic design patterns** as graphs you can build, and a deep treatment of **data cubes and the semantic layer** as the governed tool that keeps analytical agents honest. A full reference implementation assembles the pieces into one deployable agent.

Versions track the state of the libraries in mid-2026 (`langchain 1.3.x`, `langgraph 1.2.x`, `langgraph-checkpoint 4.x`). Where the API surface moves, the underlying execution model does not — the patterns are written to outlast the function names.

## Contents

1. [Two layers: the framework and the runtime](01-two-layers-the-framework-and-the-runtime.md)
2. [LCEL and the Runnable protocol](02-lcel-and-the-runnable-protocol.md)
3. [The engine underneath: Pregel super-steps](03-the-engine-underneath-pregel-super-steps.md)
4. [StateGraph: state, reducers, nodes, edges](04-stategraph-state-reducers-nodes-edges.md)
5. [Durability: checkpointers, threads, time-travel](05-durability-checkpointers-threads-time-travel.md)
6. [Human-in-the-loop and interrupts](06-human-in-the-loop-and-interrupts.md)
7. [Streaming and observability](07-streaming-and-observability.md)
8. [Tools, the ReAct loop, and structured output](08-tools-the-react-loop-and-structured-output.md)
9. [A catalogue of agentic design patterns](09-a-catalogue-of-agentic-design-patterns.md)
10. [Data cubes and the semantic layer as a governed tool](10-data-cubes-and-the-semantic-layer-as-a-governed-tool.md)
11. [Reference implementation: a governed analytics agent](11-reference-implementation-a-governed-analytics-agent.md)
12. [Production notes & decision matrix](12-production-notes-decision-matrix.md)

---

*Part of the [Agentic AI Field Guides](../README.md). Diagrams render inline on GitHub; the source HTML/PDF lives alongside this repo.*
