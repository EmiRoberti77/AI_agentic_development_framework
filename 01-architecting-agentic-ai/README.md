# Architecting Agentic AI

> A practitioner's field guide to the data structures, retrieval systems, frameworks, and safety patterns behind production LLM applications.

**Author:** Emiliano Roberti · **Level:** Architecture & implementation · **Updated:** May 2026

Building an AI system that is *correct, observable, recoverable, and safe* under production load is an architecture problem — and it is the problem this paper addresses. We treat the modern AI application as a layered system and work bottom-up: from how a language model actually represents data (tokens, embeddings, vector spaces), through the storage and retrieval substrate (vector databases, graph databases, semantic and data layers), the reasoning and control layer (RAG, structured outputs, agent frameworks, multi-agent orchestration), the model-customization layer (RAG vs. fine-tuning, LoRA/QLoRA, custom datasets), and finally the cross-cutting **safety and trust boundary** that must wrap all of it.

The intent is pedagogical: each section explains the *mechanism* first, then the *engineering decision* — when to pick a pattern, what it costs, and how it fails. Diagrams show how the pieces actually fit together. The closing reference architecture assembles every layer into one deployable blueprint. Where the field moves fast, the paper reflects the state of practice as of mid-2026; the patterns are intended to outlive the specific tools that implement them.

## Contents

1. [The agentic stack — a mental model](01-the-agentic-stack-a-mental-model.md)
2. [How an LLM represents data: tokens, embeddings, vectors](02-how-an-llm-represents-data-tokens-embeddings-vectors.md)
3. [The data layer: cubes, medallion, semantic layer, feature stores](03-the-data-layer-cubes-medallion-semantic-layer-feature-stores.md)
4. [Vector databases & semantic search](04-vector-databases-semantic-search.md)
5. [Graph databases & GraphRAG](05-graph-databases-graphrag.md)
6. [RAG patterns: naïve → advanced → agentic](06-rag-patterns-na-ve-advanced-agentic.md)
7. [Structured LLM outputs & constrained decoding](07-structured-llm-outputs-constrained-decoding.md)
8. [Agent frameworks: LangChain vs. LangGraph](08-agent-frameworks-langchain-vs-langgraph.md)
9. [Multi-agent orchestration patterns](09-multi-agent-orchestration-patterns.md)
10. [Customizing the model: RAG, fine-tuning, LoRA & custom data](10-customizing-the-model-rag-fine-tuning-lora-custom-data.md)
11. [Safety-conscious architecture & the trust boundary](11-safety-conscious-architecture-the-trust-boundary.md)
12. [Reference architecture: putting it together](12-reference-architecture-putting-it-together.md)
13. [Decision matrices & selection guide](13-decision-matrices-selection-guide.md)
14. [References & further reading](14-references-further-reading.md)

---

*Part of the [Agentic AI Field Guides](../README.md). Diagrams render inline on GitHub; the source HTML/PDF lives alongside this repo.*
