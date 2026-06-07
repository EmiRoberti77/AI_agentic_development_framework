# StateGraph: state, reducers, nodes, edges

[← The engine underneath: Pregel super-steps](03-the-engine-underneath-pregel-super-steps.md) · [Guide index](README.md) · [Durability: checkpointers, threads, time-travel →](05-durability-checkpointers-threads-time-travel.md)

---

> A LangGraph application is built in four moves: define the **state**, add **nodes**, wire **edges**, then **compile**. The state schema is the single most consequential decision in the whole project — teams routinely rewrite it twice before an agent is stable — so we start there.

## 1. The state schema and the role of reducers

State is a typed schema, either a `TypedDict` (lightweight, the common choice) or a Pydantic model (when you want validation at the boundary). Each field is a channel. By default a field is an *overwrite* channel: a node's update replaces the value. To make a field *accumulate*, you annotate it with a reducer.

A reducer has the signature `(current_value, update) -> new_value`. The engine calls it to merge each node's partial into the channel. This is exactly the mechanism that makes parallel writes and resumable accumulation well-defined.

```python
import operator
from typing import Annotated, TypedDict
from langchain_core.messages import AnyMessage
from langgraph.graph.message import add_messages

class AgentState(TypedDict):
    # ACCUMULATE: add_messages is a smart reducer -- it appends, and also
    # de-duplicates / updates messages by id (needed for streaming + tools).
    messages: Annotated[list[AnyMessage], add_messages]

    # ACCUMULATE: operator.add concatenates lists across nodes and across
    # parallel branches without a race.
    findings: Annotated[list[str], operator.add]

    # OVERWRITE (no annotation): last write wins. Use for "current" values.
    plan: str
    step_count: int
    approved: bool
```

> **KEY — Accumulator vs. overwrite — the rule of thumb**  
> Use a reducer (`Annotated[list, add]`) for anything that should *grow* over the agent's life: messages, findings, errors, completed steps. Use a plain type for *current* values: which step you are on, the active plan, whether approval was granted. Accumulator fields survive and merge across checkpoints and parallel branches; overwrite fields take their last written value. Getting this wrong produces either silent data loss (you used overwrite where you needed accumulate) or unbounded state growth (the reverse).

If you ever need a single node to bypass a reducer and replace a reduced channel wholesale, wrap the value:

```python
from langgraph.types import Overwrite

def reset_node(state: AgentState):
    # normally `messages` appends; Overwrite replaces the entire channel.
    return {"messages": Overwrite([system_message])}
```

## 2. Nodes are functions over state

A node is just a callable that receives the current state and returns a partial update — a dict of the channels it wants to write. It must not mutate the input; it returns only the keys it changes. The body can be anything: a plain function, an LCEL chain, a model call, a tool invocation, even another compiled graph (a subgraph).

```python
from langchain_openai import ChatOpenAI
model = ChatOpenAI(model="gpt-5.3", temperature=0)

def planner(state: AgentState) -> dict:
    msg = model.invoke(state["messages"])
    # return ONLY the channels we touch; the engine merges via reducers.
    return {"messages": [msg], "plan": msg.content, "step_count": state["step_count"] + 1}

def researcher(state: AgentState) -> dict:
    hits = vector_store.similarity_search(state["plan"], k=5)
    return {"findings": [d.page_content for d in hits]}   # operator.add appends
```

## 3. Edges: static, conditional, and the entry point

Edges schedule the next node(s). There are two kinds. A **static edge** always fires: after `A`, run `B`. A **conditional edge** runs a router function over the state and uses its return value to pick the next node — this is how branching and looping are expressed. `START` and `END` are sentinel nodes marking the graph's entry and exit.

```python
from langgraph.graph import StateGraph, START, END

def route_after_plan(state: AgentState) -> str:
    # the router returns a KEY; the path_map translates it to a node name.
    if state["step_count"] > 6:
        return "stop"                 # circuit breaker: bound the cycle
    if "need_research" in state["plan"].lower():
        return "research"
    return "stop"

builder = StateGraph(AgentState)
builder.add_node("planner", planner)
builder.add_node("research", researcher)

builder.add_edge(START, "planner")          # entry point
builder.add_conditional_edges(
    "planner", route_after_plan,
    {"research": "research", "stop": END},  # path_map: key -> destination
)
builder.add_edge("research", "planner")     # the CYCLE: loop back to plan
```

That single `add_edge("research", "planner")` is the whole point of LangGraph: it closes a loop that LCEL (Fig. 2) structurally cannot. The router's circuit breaker on `step_count` is not optional — an agentic cycle with no bound is an outage waiting for a prompt that never converges.

## 4. Compile

Compilation validates the topology (no orphan nodes, reachable `END`) and binds runtime concerns: the checkpointer (§5), static breakpoints, and the long-term store. `StateGraph` is a builder; only the compiled graph is executable and exposes `invoke / stream / ainvoke / astream`.

```python
graph = builder.compile()                 # executable; add checkpointer in S5

result = graph.invoke({
    "messages": [("user", "Draft a market-entry brief for the EU.")],
    "findings": [], "plan": "", "step_count": 0, "approved": False,
})
print(result["plan"], len(result["findings"]), "findings")
```

> **NOTE — Keep state small and typed**  
> The most common scaling failure is unbounded state: messages and findings grow every cycle until context blows up or the checkpoint balloons. Prune aggressively — summarise old messages into one, cap `findings`, and keep “current” scalars on overwrite channels. Small typed state is the signature of a healthy LangGraph project.


---

[← The engine underneath: Pregel super-steps](03-the-engine-underneath-pregel-super-steps.md) · [Guide index](README.md) · [Durability: checkpointers, threads, time-travel →](05-durability-checkpointers-threads-time-travel.md)
