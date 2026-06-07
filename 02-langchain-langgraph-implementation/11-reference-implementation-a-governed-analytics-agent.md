# Reference implementation: a governed analytics agent

[← Data cubes and the semantic layer as a governed tool](10-data-cubes-and-the-semantic-layer-as-a-governed-tool.md) · [Guide index](README.md) · [Production notes & decision matrix →](12-production-notes-decision-matrix.md)

---

> Everything assembled into one deployable graph: typed accumulating state, the constrained semantic-layer tool, a ReAct loop, a human approval gate on any action that leaves the read-only path, durable Postgres checkpointing, and a validated structured answer. Read it as the canonical skeleton to copy.

```python
import operator
from typing import Annotated, TypedDict
from pydantic import BaseModel, Field
from langchain_core.messages import AnyMessage, ToolMessage
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langgraph.types import interrupt
from langgraph.checkpoint.postgres import PostgresSaver
from langchain_openai import ChatOpenAI

# ---- 1. STATE: accumulate messages/findings; overwrite the scalars (S4) ----
class AnalyticsState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
    findings: Annotated[list[dict], operator.add]
    steps: int

class Answer(BaseModel):                       # validated final output (S8)
    headline: str
    figures: dict[str, float]
    caveats: list[str] = Field(default_factory=list)

# query_cube is the constrained tool from S10 (read-only, deterministic).
READ_TOOLS  = {"query_cube": query_cube}
WRITE_TOOLS = {"publish_report": publish_report}   # high-agency: gated
ALL_TOOLS   = {**READ_TOOLS, **WRITE_TOOLS}

model = ChatOpenAI(model="gpt-5.3", temperature=0).bind_tools(list(ALL_TOOLS.values()))

# ---- 2. NODES (S4, S8) ----
def reason(state: AnalyticsState) -> dict:
    return {"messages": [model.invoke(state["messages"])], "steps": state["steps"] + 1}

def act(state: AnalyticsState) -> dict:
    out = []
    for tc in state["messages"][-1].tool_calls:
        if tc["name"] in WRITE_TOOLS:                 # HITL gate (S6)
            ok = interrupt({"approve_action": tc["name"], "args": tc["args"]})
            if ok != "approve":
                out.append(ToolMessage("Action rejected by reviewer.",
                                       tool_call_id=tc["id"]))
                continue
        result = ALL_TOOLS[tc["name"]].invoke(tc["args"])  # side effect AFTER gate
        out.append(ToolMessage(str(result), tool_call_id=tc["id"]))
        if tc["name"] == "query_cube":
            out_findings = {"findings": [{"q": tc["args"], "r": result}]}
    return {"messages": out, **(out_findings if "query_cube" in
            [t["name"] for t in state["messages"][-1].tool_calls] else {})}

def finalize(state: AnalyticsState) -> dict:
    ans = model.with_structured_output(Answer).invoke(
        state["messages"] + [("user", "Summarise findings as the Answer schema.")])
    return {"messages": [("assistant", ans.model_dump_json())]}

# ---- 3. CONTROL FLOW: bounded ReAct + finalize (S4, S8) ----
def route(state: AnalyticsState) -> str:
    if state["steps"] > 8:                             # circuit breaker
        return "finalize"
    return "act" if state["messages"][-1].tool_calls else "finalize"

g = StateGraph(AnalyticsState)
g.add_node("reason", reason); g.add_node("act", act); g.add_node("finalize", finalize)
g.add_edge(START, "reason")
g.add_conditional_edges("reason", route, {"act": "act", "finalize": "finalize"})
g.add_edge("act", "reason")                            # the loop
g.add_edge("finalize", END)

# ---- 4. COMPILE with durability (S5) ----
checkpointer = PostgresSaver.from_conn_string(POSTGRES_URL); checkpointer.setup()
agent = g.compile(checkpointer=checkpointer)

# ---- 5. RUN, streaming progress (S7) ----
cfg = {"configurable": {"thread_id": "analytics-001"}}
for mode, chunk in agent.stream(
        {"messages": [("user", "Q1 net revenue by region, then publish it.")],
         "findings": [], "steps": 0},
        cfg, stream_mode=["updates", "messages"]):
    ...   # render tokens / progress; pauses at the publish approval interrupt
```

Trace the layers in that one file: the **state** separates accumulating channels from overwrite scalars (§4); the **cube tool** makes wrong numbers unrepresentable (§10); the **ReAct loop** is a bounded two-node cycle (§8) with an explicit circuit breaker (§4); the **write gate** places the side effect strictly after the interrupt so a resume cannot double-publish (§3, §6); the **Postgres checkpointer** makes the whole run survive a restart mid-approval (§5); and the **structured finalize** returns a validated object, not prose (§8). Nothing here is exotic — it is the primitives, composed with discipline.


---

[← Data cubes and the semantic layer as a governed tool](10-data-cubes-and-the-semantic-layer-as-a-governed-tool.md) · [Guide index](README.md) · [Production notes & decision matrix →](12-production-notes-decision-matrix.md)
