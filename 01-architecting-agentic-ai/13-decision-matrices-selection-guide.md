# Decision matrices & selection guide

[← Reference architecture: putting it together](12-reference-architecture-putting-it-together.md) · [Guide index](README.md) · [References & further reading →](14-references-further-reading.md)

---

> Condensed rules of thumb for the recurring choices. Treat as starting points; validate with a proof-of-concept on your own data and query distribution.

### Retrieval

| If… | Then |
| --- | --- |
| < ~5M vectors, already on Postgres | **pgvector** + HNSW; add app-side BM25 for hybrid |
| Need fastest filtered self-hosted search | **Qdrant** |
| Want native hybrid + auto-vectorization | **Weaviate** |
| Billion-scale | **Milvus** (budget K8s ops) |
| Zero-ops managed | **Pinecone** |
| Multi-hop / relational questions proven | add a **knowledge graph** (Neo4j) → GraphRAG hybrid |

### Orchestration

| If… | Then |
| --- | --- |
| Single agent, 1–2 tools | vendor SDK (**OpenAI Agents** / **Claude Agent SDK**) — skip frameworks |
| Cycles, retries, human-in-loop, audit | **LangGraph** StateGraph |
| Fast prototype, role-split work | **CrewAI** |
| Multi-party debate / code execution | **AG2** (AutoGen successor) |
| .NET / Azure shop | **Microsoft Agent Framework** |
| Multimodal / GCP-native | **Google ADK** |

### Customization

| If… | Then |
| --- | --- |
| Facts change / must cite / private | **RAG** |
| Need consistent format/tone/behaviour | **LoRA** fine-tune (+ RAG for facts) |
| GPU-memory constrained | **QLoRA** |
| Tried prompt + RAG, still short | fine-tune — write evals first |
| LoRA underperforms after tuning | consider distillation / full FT (rare) |

## Closing

The tools in this paper will be renamed and rewritten — they always are. What persists is the architecture: a governed source of truth; retrieval that finds what's relevant and connected; reasoning made reliable through structure and durable state; customization applied in the right order; and a trust boundary that assumes the model and the world are both untrusted. Master the layers and the patterns, and the next framework is just an implementation detail.

— Emiliano Roberti
AI / Data Architecture · May 2026


---

[← Reference architecture: putting it together](12-reference-architecture-putting-it-together.md) · [Guide index](README.md) · [References & further reading →](14-references-further-reading.md)
