# Chapter 13 — Cost, Performance and Model Selection

> **Learning objectives:** Understand the economics of LLM-powered agents, choose between small and frontier models, weigh local vs. SaaS deployment, design for latency budgets, and apply concrete techniques (caching, routing, distillation) to cut cost without losing quality.

---

## 13.1 The cost equation

For a single agent run:

$$
\text{Cost} = \sum_{i=1}^{N_{\text{LLM}}} \big( p_{\text{in}}^{(i)} \cdot T_{\text{in}}^{(i)} + p_{\text{out}}^{(i)} \cdot T_{\text{out}}^{(i)} \big) \;+\; \sum_{j=1}^{N_{\text{tool}}} C_{\text{tool}}^{(j)}
$$

| Variable | Meaning |
|:--|:--|
| $N_{\text{LLM}}$ | Number of LLM calls in the run |
| $p_{\text{in}}, p_{\text{out}}$ | Per-token prices (input/output) |
| $T_{\text{in}}, T_{\text{out}}$ | Tokens (input/output) per call |
| $C_{\text{tool}}^{(j)}$ | External cost of tool $j$ (search API, ThousandEyes, etc.) |

Three levers to pull:

1. **Cheaper tokens** (smaller model, prompt caching, local)
2. **Fewer tokens** (concise prompts, summarised tool outputs, shorter chains)
3. **Fewer calls** (better tools, early stopping, parallelism)

---

## 13.2 The model landscape

```mermaid
flowchart LR
    SS["🪶 Small<br/>(< 8B)<br/>local · cheap · narrow"] --> SM["🐦 Medium<br/>(8\u201370B)<br/>strong general"]
    SM --> SL["🦅 Large frontier<br/>(100B+)<br/>best reasoning · costly"]

    style SS fill:#74b9ff,stroke:#0984e3,color:#000
    style SM fill:#a29bfe,stroke:#6c5ce7,color:#000
    style SL fill:#fab1a0,stroke:#e17055,color:#000
```

| Tier | Examples (2026 era) | Sweet spot |
|:--|:--|:--|
| Small | Llama 3.2 3B, Phi-4, Qwen 2.5 7B | Routing, classification, structured extraction |
| Medium | Llama 3.3 70B, Mistral Large, Qwen 2.5 72B | Most tool-calling agents |
| Large / frontier | GPT-4.1, Claude Opus 4, Gemini Ultra | Hard reasoning, judge, complex multi-step |

> Names and tiers move quickly — what matters is the **principle**: don't pay frontier prices for a task a 7B model can handle.

### Where to spend the big model

| Phase | Suggested tier |
|:--|:--|
| Intent classification / routing (supervisor) | Small |
| Tool selection in a 5–10 tool agent | Small/Medium |
| Diagnosis with reasoning over evidence | Medium → Large |
| Final synthesis & user-facing answer | Medium |
| LLM-as-judge in eval | Different model than the agent, often Large |

---

## 13.3 Local vs. SaaS

### Decision factors

| Factor | Lean SaaS | Lean Local |
|:--|:--|:--|
| Data sensitivity | Low (public) | High (configs, customer IPs) |
| Latency requirement | Forgiving | Sub-200 ms / on-device |
| Cost at scale (very high QPS) | Low/medium QPS | High sustained QPS |
| Ops capacity | Small team | DevOps capacity to run GPUs |
| Need bleeding-edge model | Yes | No |
| Air-gapped / regulated | No | Yes |

### Local runtimes

| Runtime | Notes |
|:--|:--|
| **Ollama** | Easy local serving, good for dev |
| **vLLM** | High-throughput inference (PagedAttention), production |
| **TGI** (HF Text Generation Inference) | Production, good ecosystem |
| **llama.cpp** | CPU/edge inference; quantised models |
| **NVIDIA NIM** | Packaged enterprise inference microservices |

### Hybrid is normal

```mermaid
flowchart LR
    R["📨 Request"] --> CL["📋 Classifier<br/>(small local)"]
    CL -->|simple| LO["🪶 Local 7B<br/>(structured extraction)"]
    CL -->|hard| SA["🦅 SaaS frontier<br/>(complex reasoning)"]
    LO & SA --> A["💬 Answer"]

    style R fill:#74b9ff,stroke:#0984e3,color:#000
    style CL fill:#a29bfe,stroke:#6c5ce7,color:#000
    style LO fill:#81ecec,stroke:#00cec9,color:#000
    style SA fill:#fab1a0,stroke:#e17055,color:#000
    style A fill:#55efc4,stroke:#00b894,color:#000
```

This **router pattern** typically captures 60–80 % of requests on the cheap path while preserving frontier quality where it matters.

---

## 13.4 Latency budgets

Decompose end-to-end latency:

| Component | Typical |
|:--|:--|
| Network round-trip to LLM | 50–250 ms |
| Prompt processing (input tokens) | linear in $T_{\text{in}}$ |
| Generation (output tokens) | linear in $T_{\text{out}}$ (often the dominant term) |
| Tool call (device, API) | 100 ms – 5 s |
| Re-rank / retrieval | 50–500 ms |

> Generation latency scales with **output** tokens. Asking for shorter answers is the cheapest win.

### Patterns to cut latency

| Pattern | Effect |
|:--|:--|
| **Streaming** | First token in < 500 ms; UX feels fast |
| **Parallel tool calls** | When tools are independent |
| **Speculative decoding** | Draft + verify model |
| **Early stopping** | Stop as soon as enough evidence (Ch 7 budgets) |
| **Caching** (prompt, embedding, retrieval, tool) | Skip work entirely |

---

## 13.5 Cost-reduction techniques (practical)

### Prompt caching

Most providers (OpenAI, Anthropic) cache repeated prefixes — system prompt + tool definitions can be reused for ~10× cheaper. Keep the **stable** part of the prompt at the top.

### Smaller models for "boring" calls

Use a small model for:

- Routing / classification
- Summarising tool outputs (§4.7)
- Extracting fields from text
- LLM-as-judge for cheap structural checks

### Distillation

Run frontier model offline to label data, fine-tune a small model on those labels. Common for high-volume, narrow tasks (e.g. log-line classification).

### Quantisation (local)

INT8 / INT4 (e.g. AWQ, GPTQ) cut GPU memory by 2–4× with marginal quality loss for many tasks.

### Truncation and summarisation of context

Don't ship 200 log lines — ship a 10-line summary. Don't ship full traces — ship the last N spans.

### Batch and cache embeddings

Embedding the same query twice is waste. Cache by `sha256(text + model + version)`.

### Tool result memoisation

Within a run (or short TTL), cache `(tool, args)` → result. Re-asking "is the link up?" five times is common.

---

## 13.6 A decision flow for model choice

```mermaid
flowchart TB
    S["🎯 Task"] --> Q1{"Privacy<br/>restricts SaaS?"}
    Q1 -->|yes| L["🏠 Local pipeline"]
    Q1 -->|no| Q2{"High volume<br/>(>>1M/day)?"}
    Q2 -->|yes| Q3{"Specialised<br/>narrow task?"}
    Q3 -->|yes| DI["🧪 Distil small model"]
    Q3 -->|no| RT["🔀 Router (small\u2192large)"]
    Q2 -->|no| Q4{"Hard reasoning<br/>required?"}
    Q4 -->|yes| FR["🦅 Frontier SaaS"]
    Q4 -->|no| MD["🐦 Medium SaaS"]

    style S fill:#74b9ff,stroke:#0984e3,color:#000
    style L fill:#81ecec,stroke:#00cec9,color:#000
    style DI fill:#a29bfe,stroke:#6c5ce7,color:#000
    style RT fill:#ffeaa7,stroke:#fdcb6e,color:#000
    style FR fill:#fab1a0,stroke:#e17055,color:#000
    style MD fill:#55efc4,stroke:#00b894,color:#000
```

---

## 13.7 Cost dashboards and budgets

Track in production:

| Metric | Alert when |
|:--|:--|
| $ per run (p50, p95) | p95 > 2× baseline |
| Daily spend per agent | > budget for the day |
| Tokens per run | > 2× baseline (prompt bloat?) |
| Tool-call count per run | spikes (loop?) |
| Cache hit rate (prompt, retrieval, tool) | drops sharply |
| Model mix (% frontier vs. small) | unexpected shift |

> Cost is a **quality signal**: a sudden spike usually means a bug (loop, runaway, regression in routing).

---

## 13.8 Quality / cost trade-off

```mermaid
xychart-beta
    title "Quality vs $/run (illustrative)"
    x-axis "$ per run" 0 --> 0.20
    y-axis "Quality score (0\u20131)" 0 --> 1
    line "Frontier-only" [0.65, 0.78, 0.85, 0.90, 0.92]
    line "Router (small\u2192large)" [0.70, 0.83, 0.88, 0.90, 0.91]
    line "Small-only" [0.55, 0.60, 0.62, 0.62, 0.63]
```

Two important shapes:

- **Diminishing returns**: doubling spend rarely doubles quality.
- **Routers** often dominate frontier-only at the same cost.

---

## 13.9 Anti-patterns

| Anti-pattern | Symptom |
|:--|:--|
| "Use the biggest model everywhere" | Cost explodes; quality plateaus |
| Long system prompts re-sent every call | Tokens wasted; fix with prompt cache |
| Dumping raw `show` output to the LLM | Token bloat; summarise first |
| Ignoring streaming | UX feels broken even on fast backends |
| No cost dashboard | Bills surprise you |
| Hard-coding model names everywhere | Painful to swap; use a config layer |

---

## Summary

```mermaid
mindmap
  root((Chapter 13<br/>Recap))
    Cost equation
      Cheaper · Fewer · Smarter
    Model tiers
      Small · Medium · Frontier
      Spend big only where it matters
    Local vs SaaS
      Privacy · Scale · Ops
      Hybrid router common
    Latency
      Output tokens dominate
      Stream · Parallel · Cache
    Cost techniques
      Prompt cache · Distillation
      Quantisation · Memoise
    Decision flow
      Privacy \u2192 Volume \u2192 Reasoning
    Dashboards
      $/run · tokens · cache hit
    Trade-off
      Routers > frontier at same cost
```

---

## Exercises

1. **Cost calc.** A run uses 8k input + 1k output tokens at $3/M in + $15/M out, plus 3 tool calls at $0.002 each. Total cost?
2. **Routing design.** Sketch a router for a NetOps agent that splits "lookup" (small), "triage" (medium), "RCA" (frontier).
3. **Latency budget.** 5-second p95 budget — allocate it across LLM, retrieval, tools, network. Justify.
4. **Cache plan.** What would you cache and at what TTL: tool calls, embeddings, prompt prefixes, retrieved chunks?
5. **Distillation.** Identify a NetOps task narrow and high-volume enough to justify distilling a small model. Outline the data labelling step.
6. **Anti-pattern hunt.** Read a teammate's prompt that resends a 4k-token system message every call. Propose two fixes.
