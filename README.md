# Agentic AI Field Guides

Two companion technical guides for engineers and architects building production LLM
systems — from the patterns that shape the whole system down to the runtime code that
makes it actually run. Authored by **Emiliano Roberti**.

Both guides are written to be read in a browser on GitHub: prose, code, tables, and
architecture **diagrams** all render inline. The original single-file HTML and
print-ready PDF versions live in the project outputs.

## The two guides

### 📐 [Architecting Agentic AI](01-architecting-agentic-ai/README.md)
A practitioner's field guide to the data structures, retrieval systems, frameworks, and safety patterns behind production LLM applications.

The **architecture-level** view: how a modern AI application is layered, how each
storage and reasoning layer works, and how to choose between options. Covers data
structures and embeddings, the data layer (medallion, cubes, semantic layer), vector
and graph databases, RAG patterns, structured outputs, agent frameworks, multi-agent
orchestration, model customization (RAG vs. fine-tuning, LoRA), and the safety boundary.
14 sections · 11 diagrams.

### 🛠️ [LangChain & LangGraph at the Implementation Level](02-langchain-langgraph-implementation/README.md)
Runnables, state machines, durable execution, data cubes, and a catalogue of agentic design patterns — built from the primitives upward, for engineers who need to know how the machine actually runs.

The **implementation-level** view of the LangChain 1.0 / LangGraph 1.0 stack: the
Runnable protocol and LCEL, the Pregel super-step engine, `StateGraph` internals,
checkpointers and durable execution, interrupts and human-in-the-loop, streaming, tools
and the ReAct loop, a catalogue of design patterns, and data cubes / the semantic layer
as a governed agent tool — ending in a full reference implementation.
12 sections · 5 diagrams.

## How they fit together

Read the **architecture guide first** to decide *what* to build and *why*; read the
**implementation guide** to learn *how* to build it on LangGraph. The first answers
"which database / pattern / customization strategy fits my problem"; the second answers
"how do I wire that into a durable, observable, safe agent."

## Repository layout

```
agentic-ai-field-guides/
├── README.md                              ← you are here
├── 01-architecting-agentic-ai/
│   ├── README.md                          ← guide overview + contents
│   └── NN-*.md                            ← one navigable file per section
├── 02-langchain-langgraph-implementation/
│   ├── README.md
│   └── NN-*.md
└── assets/
    └── diagrams/
        ├── agentic/*.svg
        └── langgraph/*.svg
```

Each section file carries a `← previous · index · next →` navigation bar at the top and
bottom, so you can read a guide straight through or jump around.

## Who this is for

Engineers, ML/AI engineers, data and solution architects, and technical leads who need
more than a quickstart — people responsible for systems that have to be *correct,
observable, recoverable, and safe* under production load.

---

*© 2026 Emiliano Roberti. Living documents — patterns are intended to outlive the specific tools that implement them.*
