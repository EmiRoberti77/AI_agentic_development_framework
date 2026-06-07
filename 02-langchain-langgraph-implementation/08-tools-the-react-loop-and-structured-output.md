# Tools, the ReAct loop, and structured output

[← Streaming and observability](07-streaming-and-observability.md) · [Guide index](README.md) · [A catalogue of agentic design patterns →](09-a-catalogue-of-agentic-design-patterns.md)

---

> An agent is a loop: the model reasons, optionally calls a tool, observes the result, and reasons again until it answers. That loop is the ReAct pattern, and in LangGraph it is just a two-node cycle. Structured output is the same machinery pointed at a schema instead of a function.

## Defining tools

The `@tool` decorator turns a typed Python function into a tool: its signature and docstring become the schema the model sees, and its type hints drive argument validation. The model never runs your code — it emits a request to call it, and the runtime dispatches.

```python
from langchain_core.tools import tool

@tool
def query_metric(metric: str, dimensions: list[str], time_range: str) -> dict:
    """Query a governed business metric from the semantic layer.

    metric: a defined metric name, e.g. 'net_revenue'.
    dimensions: grouping fields, e.g. ['region', 'month'].
    time_range: ISO interval, e.g. '2026-01-01/2026-03-31'.
    """
    return semantic_layer.run(metric, dimensions, time_range)

model_with_tools = model.bind_tools([query_metric])
```

## The ReAct loop as a graph

Two nodes and one conditional edge. The model node calls the LLM; if the response contains tool calls, route to the tool node, which executes them and appends `ToolMessage`s; then loop back to the model. If there are no tool calls, the model has answered — route to `END`.

```python
from langgraph.graph import StateGraph, START, END
from langchain_core.messages import ToolMessage

def call_model(state):
    return {"messages": [model_with_tools.invoke(state["messages"])]}

def call_tools(state):
    last = state["messages"][-1]
    out = []
    for tc in last.tool_calls:
        result = TOOLS_BY_NAME[tc["name"]].invoke(tc["args"])
        out.append(ToolMessage(content=str(result), tool_call_id=tc["id"]))
    return {"messages": out}

def should_continue(state) -> str:
    return "tools" if state["messages"][-1].tool_calls else "end"

g = StateGraph(AgentState)
g.add_node("model", call_model)
g.add_node("tools", call_tools)
g.add_edge(START, "model")
g.add_conditional_edges("model", should_continue, {"tools": "tools", "end": END})
g.add_edge("tools", "model")           # observe -> reason again (the loop)
agent = g.compile(checkpointer=checkpointer)
```

This is exactly what `create_agent` builds for you. Authoring it by hand is worthwhile once, because every advanced pattern in §9 is a variation on this skeleton — add a reflection node, a supervisor, a validation gate, and you have moved from a single agent to an orchestrated system without leaving the model.

## Structured output: a schema instead of prose

When you need typed data rather than text, bind a schema. Under the hood the model is constrained — via tool-calling or grammar-constrained decoding — so the response is guaranteed to *parse*; you still validate that it is *correct*. Define the contract with Pydantic and let the runtime enforce it.

```python
from pydantic import BaseModel, Field

class RiskAssessment(BaseModel):
    severity: int = Field(ge=1, le=5, description="1=low, 5=critical")
    rationale: str
    mitigations: list[str]

structured = model.with_structured_output(RiskAssessment)
risk = structured.invoke("Assess supply-chain risk for EU market entry.")
assert isinstance(risk, RiskAssessment)        # parsed + field-validated
```

> **KEY — Syntactic vs. semantic correctness**  
> Constrained decoding guarantees the output *parses* into your schema — the brackets always close, the enum is always valid. It does *not* guarantee the values are *true*. Keep the Pydantic validators (ranges, cross-field rules) as your semantic gate, and on failure feed the validation error back to the model for one retry. Structure is enforced by the decoder; meaning is enforced by you.


---

[← Streaming and observability](07-streaming-and-observability.md) · [Guide index](README.md) · [A catalogue of agentic design patterns →](09-a-catalogue-of-agentic-design-patterns.md)
