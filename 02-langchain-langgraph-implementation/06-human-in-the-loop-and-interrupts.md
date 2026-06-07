# Human-in-the-loop and interrupts

[← Durability: checkpointers, threads, time-travel](05-durability-checkpointers-threads-time-travel.md) · [Guide index](README.md) · [Streaming and observability →](07-streaming-and-observability.md)

---

> An agent that can spend money, send email, or modify production data must be pausable for human review — and the pause must survive arbitrarily long, because the human might respond in five seconds or five hours. LangGraph makes this a first-class primitive built directly on the checkpointer, not a bolted-on callback.

## Dynamic interrupts: pause from inside a node

Call `interrupt(payload)` anywhere inside a node. The engine checkpoints, surfaces the payload to the caller, and stops. The run is now durably parked. When you are ready, you resume by invoking the graph with a `Command(resume=value)`; that `value` becomes the return value of the `interrupt()` call, and the node continues.

```python
from langgraph.types import interrupt, Command

def approval_gate(state: AgentState) -> dict:
    decision = interrupt({                      # <-- pause + checkpoint here
        "question": "Approve this plan before execution?",
        "plan": state["plan"],
        "options": ["approve", "reject", "edit"],
    })
    # execution RESUMES here when Command(resume=...) is supplied.
    return {"approved": decision == "approve"}

# 1) run until the interrupt:
out = graph.invoke(initial_state, cfg)
print(out["__interrupt__"])     # the payload we passed to interrupt(...)

# 2) ... hours later, on any instance, with the same thread_id ...
graph.invoke(Command(resume="approve"), cfg)   # node resumes; decision="approve"
```

> **WARNING — The re-execution trap, again**  
> Recall §3: when the graph resumes, the interrupted node runs *from the top*. Every line before `interrupt()` executes a second time. So any side effect placed before the interrupt — a write, a charge, a notification — happens twice. Put side effects *after* the interrupt, or make them idempotent. This single rule prevents the majority of HITL bugs.

## Static breakpoints: pause at the edge

When you do not want to modify node code, compile with `interrupt_before` or `interrupt_after` on named nodes. The graph halts at that boundary; you inspect or edit state with `update_state` (§5), then resume with `invoke(None, cfg)`.

```python
graph = builder.compile(
    checkpointer=checkpointer,
    interrupt_before=["execute_trade"],   # always pause before this node
)
graph.invoke(state, cfg)                  # stops before execute_trade
graph.update_state(cfg, {"approved": True})
graph.invoke(None, cfg)                   # continue from the breakpoint
```

## The framework shortcut: HumanInTheLoopMiddleware

If you build with LangChain's `create_agent` rather than a hand-rolled graph, the same capability is available as middleware. It intercepts tool calls and requires approval for the ones you designate — the approval still rides on the LangGraph interrupt machinery underneath, so the durability guarantees are identical.

```python
from langchain.agents import create_agent
from langchain.agents.middleware import HumanInTheLoopMiddleware

agent = create_agent(
    model="gpt-5.3",
    tools=[search, run_sql, send_email],
    middleware=[HumanInTheLoopMiddleware(
        interrupt_on={"send_email": True, "run_sql": False},  # gate writes only
        interrupt_mode="approve_or_edit",
    )],
    checkpointer=checkpointer,
)
```

This is the bridge back to the safety boundary: a high-agency tool call (“excessive agency” in OWASP's taxonomy) should pass through an approval gate before it executes. The interrupt is the mechanism; the policy of *which* tools require approval is yours to set.


---

[← Durability: checkpointers, threads, time-travel](05-durability-checkpointers-threads-time-travel.md) · [Guide index](README.md) · [Streaming and observability →](07-streaming-and-observability.md)
