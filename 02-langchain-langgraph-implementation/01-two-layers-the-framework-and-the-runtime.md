# Two layers: the framework and the runtime

[Guide index](README.md) · [LCEL and the Runnable protocol →](02-lcel-and-the-runnable-protocol.md)

---

> LangChain and LangGraph are not competitors and not alternatives. They are two layers of one stack, shipped by one team, and as of the 1.0 releases (October 2025) the relationship is explicit: **LangChain is the high-level agent framework; LangGraph is the low-level runtime it executes on.**

Understanding this split is the first prerequisite for building anything serious. The wrong mental model — “LangChain for simple things, LangGraph for complex things” — leads to rewrites. The right model is layered: you almost always run on LangGraph; the only question is whether you let LangChain's `create_agent` build the graph for you, or whether you author the `StateGraph` yourself.

## What each layer actually is

**LangChain 1.0** is the developer-experience layer. It collapses the common agent into a single construct, `create_agent()`, and adds *middleware* — composable hooks that wrap the agent loop (PII redaction, human approval, summarisation, todo tracking). It standardises model I/O through typed content blocks so a tool call looks the same across providers. Its job is to get a correct agent shipped with minimal boilerplate. Critically, `create_agent` does not implement its own execution loop any more: it compiles down to a LangGraph graph.

**LangGraph 1.0** is the runtime and the framework for everything `create_agent` cannot express: arbitrary cycles, typed shared state, parallel fan-out, persistence, time-travel, and human-in-the-loop pauses that survive a server restart. It models an application as a *graph of nodes over a shared state object*, and executes that graph on a Pregel-style engine (§3). It is deliberately lower-level: more concepts, more control, no hidden loop.

> **KEY — The rule**  
> Reach for `create_agent` when the shape is “one model, some tools, a loop.” Drop to `StateGraph` the moment you need explicit control flow, multiple actors, durable long-running state, or a custom approval gate. You are on LangGraph either way — so learning the runtime is not optional for production work.

## Package topology

The ecosystem is intentionally fragmented into small packages so that a breaking change in an integration cannot destabilise the core. Pin to the layers you import directly.

![**Fig. 1** — The stack. Your code targets either `create_agent` or a hand-built ](../assets/diagrams/langgraph/fig01.svg)

***Fig. 1** — The stack. Your code targets either `create_agent` or a hand-built `StateGraph`; both compile onto the LangGraph runtime, which leans on `langchain-core` for the `Runnable` contract and message types. Provider SDKs live in thin partner packages so the core stays stable.*

A minimal install for a production analytics agent therefore looks like this — note that `langgraph.prebuilt` was deprecated at 1.0 and its functionality moved into `langchain.agents`:

```bash
pip install "langchain>=1.3,<2" "langgraph>=1.2,<2" \
    "langgraph-checkpoint-postgres" "langchain-openai" "langchain-anthropic"
```

The version pins matter. The 1.0 line committed to semantic versioning with no breaking changes before 2.0, so `>=1.x,<2` is safe; the pre-1.0 satellite packages (for example `deepagents`) can still break on minor bumps and should be pinned tighter.


---

[Guide index](README.md) · [LCEL and the Runnable protocol →](02-lcel-and-the-runnable-protocol.md)
