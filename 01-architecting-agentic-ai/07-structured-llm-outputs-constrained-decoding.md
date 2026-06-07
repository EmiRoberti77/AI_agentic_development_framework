# Structured LLM outputs & constrained decoding

[← RAG patterns: naïve → advanced → agentic](06-rag-patterns-na-ve-advanced-agentic.md) · [Guide index](README.md) · [Agent frameworks: LangChain vs. LangGraph →](08-agent-frameworks-langchain-vs-langgraph.md)

---

> An agent is software calling software. The moment an LLM's output feeds a function, a database, or another agent, "mostly valid JSON" is a production incident. Structured outputs turn schema compliance from a prompt-engineering hope into an infrastructure guarantee.

## The problem

At a thousand requests a day, "the model usually returns the right format" becomes hundreds of malformed outputs — a missing brace, a string where an integer was expected, a date in the wrong format. Each one crashes the downstream step. There are three escalating defences.

### ① Schema-as-contract with Pydantic

Define the desired shape once as a typed Python model. It serves triple duty: documentation, runtime validation, and JSON-Schema generation for the API.

```python
from pydantic import BaseModel, Field
from typing import Literal

class SupportTicket(BaseModel):
    category: Literal["billing", "bug", "feature", "other"]
    priority: Literal["low", "medium", "high", "urgent"]
    summary: str = Field(max_length=200)
    needs_human: bool
```

### ② Constrained decoding — the guarantee

This is the mechanism that makes it reliable. The provider compiles your JSON Schema into a grammar (a context-free grammar / regex) and, *at each generation step, masks every token that would violate the schema* — their probability is forced to zero. The model literally cannot emit an invalid structure. This moved schema compliance from prompt engineering to a generation-level guarantee; on hard JSON-schema evals it took compliance from sub-40% to effectively 100%.

```python
# OpenAI / Anthropic SDKs accept the Pydantic model directly
resp = client.responses.parse(
    model="<model>",
    input=[{"role":"user", "content": email_text}],
    text_format=SupportTicket,          # schema enforced by constrained decoding
)
ticket = resp.output_parsed            # a typed SupportTicket instance, guaranteed valid
route(ticket.category, ticket.priority) # safe to call downstream code
```

Open-source serving stacks (vLLM, SGLang) implement the same idea via libraries like Outlines and XGrammar, building a regex/FSM from the schema. For cross-provider portability, **Instructor** wraps Pydantic over 15+ providers with a unified API and automatic retry-on-validation-failure.

> **NOTE — Watch-outs**  
> Structured-output modes support only a *subset* of JSON Schema: no recursive schemas, limited enums, and `additionalProperties:false` required on objects. Numeric constraints like `minimum` are often stripped into field descriptions rather than enforced — so keep semantic validation in Pydantic validators, not only in the schema. The first request with a new schema pays a one-time grammar-compilation latency; subsequent calls use a cached grammar.

## Function / tool calling — structured output as an action

Tool calling is structured output pointed at code. You expose a tool with a typed signature; the model, instead of replying in prose, emits a *structured call* — `{"name":"get_weather","arguments":{"city":"London"}}` — which your runtime validates, executes, and feeds back. This is the atom of every agent: the loop of *reason → emit a validated tool call → execute → observe → repeat*. Constrained decoding guarantees the call is syntactically valid; your code must still guarantee it is *safe* (§11).

## The three-layer reliability stack

| Layer | Mechanism | Guarantees | Does NOT guarantee |
| --- | --- | --- | --- |
| Prompt instructions | "Respond as JSON matching…" | nothing (best-effort) | validity, types, completeness |
| Constrained decoding | grammar/FSM token masking | **syntactic** validity + schema shape | that values are *correct* or safe |
| Pydantic validation + retry | parse, validate, re-ask on failure | **semantic** validity (ranges, cross-field rules) | factual correctness |

Stack all three. Constrained decoding gives you a parseable object; Pydantic validators enforce business rules; a bounded retry loop salvages the rare miss. Factual correctness is still the model's job — and the reason §11 exists.


---

[← RAG patterns: naïve → advanced → agentic](06-rag-patterns-na-ve-advanced-agentic.md) · [Guide index](README.md) · [Agent frameworks: LangChain vs. LangGraph →](08-agent-frameworks-langchain-vs-langgraph.md)
