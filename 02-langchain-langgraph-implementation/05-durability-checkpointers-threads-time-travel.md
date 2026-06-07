# Durability: checkpointers, threads, time-travel

[← StateGraph: state, reducers, nodes, edges](04-stategraph-state-reducers-nodes-edges.md) · [Guide index](README.md) · [Human-in-the-loop and interrupts →](06-human-in-the-loop-and-interrupts.md)

---

> The core promise of LangGraph 1.0 is deceptively simple: *your agent should survive a server restart.* A long-running workflow that gets interrupted — by a crash, a deploy, or a human approval that arrives the next morning — resumes exactly where it stopped, with no custom database code. That promise is delivered by the checkpointer.

## What a checkpoint is

At every super-step barrier (§3) the engine serialises the full channel state plus the scheduling frontier and writes it to a **checkpointer**, namespaced by a `thread_id`. A thread is one conversation / one workflow run; its checkpoint history is the complete, replayable timeline of that run. Pass the same `thread_id` again and the graph loads the latest checkpoint and continues.

```python
from langgraph.checkpoint.postgres import PostgresSaver

# Choose the backend by deployment shape:
#   InMemorySaver  -> dev / tests only; lost on restart.
#   SqliteSaver    -> single-process / single-server production.
#   PostgresSaver  -> multi-instance, horizontally scaled production.
checkpointer = PostgresSaver.from_conn_string(POSTGRES_URL)
checkpointer.setup()                        # one-time: creates tables

graph = builder.compile(checkpointer=checkpointer)

cfg = {"configurable": {"thread_id": "user-42-session-7"}}

graph.invoke({"messages": [("user", "Start the brief.")], ...}, cfg)
# ... process dies here, machine reboots, new pod comes up ...
graph.invoke({"messages": [("user", "Add a risks section.")]}, cfg)
#  ^ same thread_id -> state from the first call is already there.
```

> **KEY — Backend selection**  
> `InMemorySaver` (aka `MemorySaver`) for development and unit tests. `SqliteSaver` when one process owns the workflow. `PostgresSaver` the moment you run more than one instance behind a load balancer — checkpoints must be visible to whichever pod picks up the next request. Use a connection pool; the checkpointer is on the hot path of every super-step.

## Inspecting and rewriting history

Because the full timeline is persisted, you get three operations that are otherwise hard to build: read the current state, walk the history, and — the powerful one — *fork* the run by editing a past checkpoint and resuming from it. This is “time-travel,” and it is how you implement debugging, what-if analysis, and human edits to an agent's trajectory.

```python
# Current state snapshot for a thread.
snap = graph.get_state(cfg)
print(snap.values["step_count"], snap.next)   # state + scheduled-next nodes

# Full ordered history (newest first): every super-step boundary.
for chk in graph.get_state_history(cfg):
    print(chk.config["configurable"]["checkpoint_id"], chk.values["step_count"])

# TIME-TRAVEL: rewind to an earlier checkpoint and fork the timeline.
old = list(graph.get_state_history(cfg))[3]          # some prior checkpoint
graph.update_state(old.config, {"plan": "corrected plan"})  # edit the past
graph.invoke(None, old.config)   # resume from the edited fork (input=None)
```

Note the `invoke(None, config)` idiom: passing `None` as input means “do not add new input, just continue executing from the loaded checkpoint.” You will see it again in §6, because resuming from a human interrupt uses the same mechanism. `update_state` respects reducers — an update to a reduced channel is merged, not blindly replaced — which is why editing history is safe.

## Long-term memory vs. thread state

Checkpoints are *thread-scoped*: they hold the state of one run. For knowledge that must persist *across* threads — user preferences, learned facts, profile data — LangGraph provides a separate `BaseStore` abstraction, bound at compile time and namespaced by an arbitrary key (commonly the user id). Keep the two distinct: thread state is the conversation; the store is the memory the conversation draws on.

```python
from langgraph.store.postgres import PostgresStore

store = PostgresStore.from_conn_string(POSTGRES_URL)
graph = builder.compile(checkpointer=checkpointer, store=store)

# inside a node, the store is injected and namespaced per-user:
def remember(state, *, store):
    ns = ("prefs", state["user_id"])
    store.put(ns, "tone", {"value": "concise"})
    tone = store.get(ns, "tone")
    return {...}
```


---

[← StateGraph: state, reducers, nodes, edges](04-stategraph-state-reducers-nodes-edges.md) · [Guide index](README.md) · [Human-in-the-loop and interrupts →](06-human-in-the-loop-and-interrupts.md)
