# Production notes & decision matrix

[← Reference implementation: a governed analytics agent](11-reference-implementation-a-governed-analytics-agent.md) · [Guide index](README.md)

---

> The failures that take agents down in production are boringly consistent. This is the checklist worth taping to the wall, followed by the “which tool” matrix.

### The recurring production failures

- **Unbounded state.** Messages and findings grow every cycle until context overflows or checkpoints bloat. Summarise and cap. Small typed state is non-negotiable.
- **Non-idempotent nodes.** Side effects before a barrier or interrupt run twice on resume. Upserts, idempotency keys, effects after the commit point.
- **Unbounded cycles.** Every loop needs a step counter and a “no progress” exit. An agent that cannot converge must be made to stop.
- **Wrong checkpointer.** In-memory in production loses everything on deploy; single-file SQLite cannot serve multiple pods. Use Postgres with a pool the moment you scale out.
- **Ungoverned data access.** A raw SQL tool will eventually report a confident wrong number. Route analytics through the semantic layer.
- **Ungated agency.** Any tool that writes, sends, pays, or deletes goes behind a human interrupt until you have earned the trust to automate it.

| if you need… | reach for |
| --- | --- |
| a fixed retrieval / extraction pipeline | **LCEL chain** (no graph) |
| one model, some tools, a loop | **create_agent** (LangChain) |
| cycles, branching, custom control flow | **StateGraph** (LangGraph) |
| resume after restart / long pauses | **checkpointer** + thread_id |
| human approval before an action | **interrupt()** / HITL middleware |
| same work over N items | **Send** map–reduce |
| several specialised skills, audited | **supervisor** pattern |
| quantitative answers from data | **semantic-layer tool**, never raw SQL |
| typed output, not prose | **with_structured_output** + validators |

> **NOTE — Closing**  
> The framework is not the hard part; the runtime discipline is. State design, idempotency, bounded cycles, durable checkpoints, gated agency, and a governed data surface are what separate a demo from a system. Learn the engine (§3), respect the barrier, and the rest is composition.


---

[← Reference implementation: a governed analytics agent](11-reference-implementation-a-governed-analytics-agent.md) · [Guide index](README.md)
