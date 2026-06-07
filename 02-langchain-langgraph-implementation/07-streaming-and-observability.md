# Streaming and observability

[← Human-in-the-loop and interrupts](06-human-in-the-loop-and-interrupts.md) · [Guide index](README.md) · [Tools, the ReAct loop, and structured output →](08-tools-the-react-loop-and-structured-output.md)

---

> A durable agent that you cannot watch is a black box. LangGraph exposes several stream modes, each answering a different question, and integrates with LangSmith for full-trace inspection. Picking the right stream mode is a UX and bandwidth decision, not a formality.

## Stream modes

| mode | emits | use it for |
| --- | --- | --- |
| **values** | the full state after each super-step | simple UIs that re-render the whole state; debugging |
| **updates** | only the channel diffs each node wrote | efficient progress UIs; logging what changed |
| **messages** | LLM tokens as they generate, per node | chat token-streaming (time-to-first-token UX) |
| **custom** | whatever a node emits via the stream writer | tool progress bars, intermediate artefacts |
| **debug** | checkpoint + task events | deep diagnostics, replay analysis |

```python
# Combine modes; each chunk is tagged with its mode.
for mode, chunk in graph.stream(state, cfg, stream_mode=["updates", "messages"]):
    if mode == "messages":
        token, meta = chunk
        print(token.content, end="", flush=True)     # live tokens
    elif mode == "updates":
        print("\n[step]", chunk)                       # what each node wrote

# astream_events gives a single structured feed across the whole run:
async for ev in graph.astream_events(state, cfg):
    if ev["event"] == "on_tool_start":
        log.info("tool %s args=%s", ev["name"], ev["data"]["input"])
```

> **NOTE — Tracing**  
> Set `LANGSMITH_TRACING=true` and every `invoke` / `stream` records a full tree: each node, each model call with token counts and latency, each tool call with inputs and outputs, and the state at every step. In production this is the difference between “the agent gave a wrong answer” and “node `research` retrieved the wrong chunk at step 4 because the query was mis-rewritten.”


---

[← Human-in-the-loop and interrupts](06-human-in-the-loop-and-interrupts.md) · [Guide index](README.md) · [Tools, the ReAct loop, and structured output →](08-tools-the-react-loop-and-structured-output.md)
