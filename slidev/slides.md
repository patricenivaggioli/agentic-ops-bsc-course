---
theme: default
highlighter: shiki
transition: slide-left
mdc: true
---

# AgenticOps for Networks

<div class="pt-10">
  <span class="text-orange-400 font-bold text-xl">AI Foundations for Networks - Module 8</span>
</div>

<div class="pt-6 text-left inline-block">
  <p class="text-sm text-gray-300">Guillaume Ladhuie, Patrice Nivaggioli</p>
</div>

<div class="abs-br m-6 text-sm text-gray-400">
  ESGI, 2026
</div>

---

# Agenda

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### Part I — Foundations
1. **From Automation to Agentic Ops**
2. **LLM & Agent Fundamentals**
3. **Anatomy of an AI Agent**

### Part II — Inputs & Tools
4. **Network Data Sources**
5. **Tools an Agent Needs**
6. **RAG for NetOps**

### Part III — Designing Agents
7. **Designing a Troubleshooting Agent**
8. **Multi-Agent Architectures**
9. **Human-in-the-Loop**
10. **Closed-Loop Automation**

</div>

<div class="space-y-1">

### Part IV — Production
11. **Observability & Evaluation**
12. **Safety, Security, Guardrails**
13. **Cost, Performance, Model Selection**
14. **AgentOps Lifecycle**
15. **Ethics, Limitations, Future**

### Part V — Project
16. **Capstone Project Ideas**

### Appendices
A. Python & Async · B. Lab Setup · C. LLM APIs · D. Glossary · E. Further Reading · F. Datasets

</div>

</div>

---
layout: section
---

# Part I
## Foundations

---

# From Automation to Agentic Ops

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### The maturity ladder

| Level | Style | Example |
|:--|:--|:--|
| **M0** | Manual CLI | telnet, copy-paste |
| **M1** | Scripted | Bash, Python loops |
| **M2** | Orchestrated | Ansible, Terraform |
| **M3** | Intent-based | NetBox + GitOps |
| **M4** | **Agentic** | LLM reasons + acts |

Each level **reduces** human toil and **raises** the level of abstraction.

</div>

<div class="space-y-1">

### Why now?

- **LLMs** can read logs, configs, runbooks
- **Tool calling** turns talk into action
- **Telemetry** is finally machine-readable (gNMI)
- **Cost** of inference dropped 10× / year

### What's an agent?

> An LLM that **plans** a task, **calls tools** to act on the world, **observes** the result, and **iterates**.

Not a chatbot. Not a script. A **loop**.

</div>

</div>

---

# Why NetOps is a good fit (and why it's hard)

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### Good fit
- Rich **text data** (configs, RFCs, runbooks)
- Many **APIs** (NETCONF, gNMI, REST)
- Repetitive **patterns** (BGP RCA, drift)
- **High toil** worth automating

### Hard parts
- Mistakes can take **production down**
- Vendor sprawl & legacy CLI
- Strict **change windows**
- **Auditability** required

</div>

<div class="space-y-1">

```mermaid {scale: 0.55}
graph LR
    H["👤 Engineer"] --> A["🤖 Agent"]
    A --> T["🔧 Tools<br/>(read/diag/act)"]
    T --> N["🌐 Network"]
    N --> O["📡 Observation"]
    O --> A
    A --> H
    style H fill:#7c3aed,color:#fff
    style A fill:#ea580c,color:#fff
    style T fill:#2563eb,color:#fff
    style N fill:#16a34a,color:#fff
    style O fill:#7c3aed,color:#fff
```

The agent **closes the loop** between intent and observation.

</div>

</div>

---

# LLM & Agent Fundamentals

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### What an LLM really is

A **next-token predictor** trained on huge text corpora.

$$ P(t_{n+1} \mid t_1, \dots, t_n) $$

- Strong at **patterns** in text
- Weak at **exact math** and **fresh facts**
- Confident even when **wrong** (hallucination)

### Key parameters

| Param | Effect | NetOps |
|:--|:--|:--|
| `temperature` | Randomness | 0.1–0.3 |
| `max_tokens` | Output cap | 256–1024 |
| `seed` | Reproducible | Set for evals |

</div>

<div class="space-y-1">

### From LLM → agent

```mermaid {scale: 0.55}
graph LR
    P["Prompt"] --> L["LLM"]
    L --> R["Reasoning"]
    R --> TC["Tool call"]
    TC --> EX["Execute"]
    EX --> OBS["Observation"]
    OBS --> L
    L --> ANS["Answer"]
    style P fill:#2563eb,color:#fff
    style L fill:#7c3aed,color:#fff
    style R fill:#ea580c,color:#fff
    style TC fill:#ea580c,color:#fff
    style EX fill:#16a34a,color:#fff
    style OBS fill:#7c3aed,color:#fff
    style ANS fill:#16a34a,color:#fff
```

The loop is what turns words into **work**.

</div>

</div>

---

# Anatomy of an AI Agent

<div class="mt-1">

```mermaid {scale: 0.65}
graph LR
    G["🎯 Goal"] --> PL["📋 Planner<br/>(LLM)"]
    PL --> AC["🔧 Tool call"]
    AC --> EX["⚙️ Executor"]
    EX --> OBS["📡 Observation"]
    OBS --> MEM["🧠 Memory"]
    MEM --> PL
    PL --> END["✅ Answer"]
    style G fill:#2563eb,color:#fff
    style PL fill:#7c3aed,color:#fff
    style AC fill:#ea580c,color:#fff
    style EX fill:#ea580c,color:#fff
    style OBS fill:#2563eb,color:#fff
    style MEM fill:#7c3aed,color:#fff
    style END fill:#16a34a,color:#fff
```

</div>

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### Core patterns
- **ReAct** — Reason + Act loop
- **Plan-and-Execute** — plan first, then run
- **Reflection** — critique own output
- **Multi-agent** — specialists + supervisor

</div>

<div class="space-y-1">

### State of an agent
- **Working memory** — current task
- **Long memory** — past episodes / RAG
- **Tools** — typed callables
- **Policy** — what is allowed

</div>

</div>

---
layout: section
---

# Part II
## Inputs & Tools

---

# Network Data Sources

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### The data pyramid

```mermaid {scale: 0.55}
graph TB
    SoT["📚 Source of Truth<br/>(NetBox, Git)"] --> CFG["⚙️ Configs"]
    CFG --> ST["📊 State<br/>(gNMI, SNMP)"]
    ST --> FL["🌊 Flows<br/>(NetFlow/IPFIX)"]
    FL --> PKT["📦 Packets<br/>(pcap)"]
    style SoT fill:#7c3aed,color:#fff
    style CFG fill:#2563eb,color:#fff
    style ST fill:#ea580c,color:#fff
    style FL fill:#16a34a,color:#fff
    style PKT fill:#6b7280,color:#fff
```

Up = **semantic richness** · Down = **raw volume**

</div>

<div class="space-y-1">

### SNMP vs gNMI

| | SNMP | gNMI |
|:--|:--|:--|
| Model | UDP polling | gRPC streaming |
| Schema | MIBs (vendor) | YANG (OpenConfig) |
| Push | No | Yes |
| LLM-friendly | Hard | Easy (JSON) |

### Reference stack

NetBox · Prometheus · Loki · Tempo · OTel · Batfish (twin)

</div>

</div>

---

# Tools an Agent Needs

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### Four tool families

| Family | Example | Risk |
|:--|:--|:--|
| **Read** | `get_bgp_neighbors` | Low |
| **Diagnostic** | `path_analysis`, `dry_run` | Low |
| **Action** | `push_config`, `clear_bgp` | High |
| **Knowledge** | `search_runbook` (RAG) | Low |

### Safe action design
- **Least privilege** scope
- **Idempotent** when possible
- **Dry-run** default
- **Approval** gate (HITL)
- **Auditable** + **reversible**

</div>

<div class="space-y-1">

### MCP — Model Context Protocol

> Open standard exposing **tools / resources / prompts** to any LLM client.

```mermaid {scale: 0.6}
graph LR
    AG["🤖 Agent"] -->|MCP| S1["📦 NetBox<br/>server"]
    AG -->|MCP| S2["📦 Prom<br/>server"]
    AG -->|MCP| S3["📦 Lab<br/>server"]
    style AG fill:#7c3aed,color:#fff
    style S1 fill:#2563eb,color:#fff
    style S2 fill:#ea580c,color:#fff
    style S3 fill:#16a34a,color:#fff
```

One protocol, many backends — vendor-neutral.

</div>

</div>

---

# RAG for NetOps

<div class="mt-1">

```mermaid {scale: 0.6}
graph LR
    Q["❓ Query"] --> EMB["🧮 Embed"]
    EMB --> VS[("📚 Vector DB")]
    EMB --> BM[("🔤 BM25")]
    VS --> RRF["🔀 RRF fusion"]
    BM --> RRF
    RRF --> RR["🥇 Cross-encoder<br/>re-rank"]
    RR --> CTX["📝 Context"]
    CTX --> LLM["🦅 LLM"]
    LLM --> ANS["✅ Answer + citations"]
    style Q fill:#2563eb,color:#fff
    style VS fill:#2563eb,color:#fff
    style BM fill:#ea580c,color:#fff
    style RRF fill:#7c3aed,color:#fff
    style RR fill:#7c3aed,color:#fff
    style CTX fill:#ea580c,color:#fff
    style LLM fill:#7c3aed,color:#fff
    style ANS fill:#16a34a,color:#fff
```

</div>

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### Useful corpora
- Internal runbooks & post-mortems
- Vendor docs (Cisco/Juniper/Arista)
- RFCs · OpenConfig YANG · MIBs
- Design docs & topology drawings

</div>

<div class="space-y-1">

### Eval metrics
- **Recall@k**, **MRR**, **nDCG** (retrieval)
- **Faithfulness** + **citation correctness** (answer)
- Always require **citations**

</div>

</div>

---

# RAG vs Fine-tune vs Tools

<div class="grid grid-cols-3 gap-4 mt-4 text-sm">

<div class="border border-purple-400 rounded-lg p-3">

### RAG
**Inject knowledge**

Fresh docs, citable, cheap to update.

✅ Runbooks, RFCs
❌ Real-time state

</div>

<div class="border border-orange-400 rounded-lg p-3">

### Fine-tune
**Shape behaviour**

Style, format, narrow tasks.

✅ Consistent output
❌ Expensive, stale

</div>

<div class="border border-blue-400 rounded-lg p-3">

### Tools
**Read/act in real time**

APIs to the live network.

✅ Live state, actions
❌ Need safe design

</div>

</div>

<div class="mt-3 text-sm text-gray-500 text-center">
In practice: combine all three — RAG for knowledge, tools for state/action, fine-tuning for style.
</div>

---
layout: section
---

# Part III
## Designing Agents

---

# Designing a Troubleshooting Agent

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### The charter (1 page)
- **Goal** — *site A → site B unreachable*
- **Scope** — IP/OSPF/BGP only
- **Tools allowed** — read + diagnostic
- **Autonomy** — L1 (suggest, no write)
- **Budgets** — ≤ 8 tool calls, ≤ 30 s
- **Success** — RCA + recommended fix

### OSI-style decomposition

L1 link → L2 ARP/MAC → L3 routes → L4 reachability → app

</div>

<div class="space-y-1">

```mermaid {scale: 0.55}
graph TB
    R["Report:<br/>A→B unreachable"] --> P1["Check L1/L2<br/>(interface up?)"]
    P1 --> P2["Check L3<br/>(route present?)"]
    P2 --> P3["Check BGP<br/>(neighbor up?)"]
    P3 --> P4["Check ACL<br/>(deny match?)"]
    P4 --> RCA["💡 RCA + fix"]
    style R fill:#ea580c,color:#fff
    style P1 fill:#2563eb,color:#fff
    style P2 fill:#2563eb,color:#fff
    style P3 fill:#2563eb,color:#fff
    style P4 fill:#2563eb,color:#fff
    style RCA fill:#16a34a,color:#fff
```

Each step is a **tool call** producing an observation.

</div>

</div>

---

# Multi-Agent Architectures

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### Patterns

| Pattern | When |
|:--|:--|
| **Supervisor** | Dispatch to specialists |
| **Hierarchical** | Nested supervisors |
| **Pipeline** | Linear stages |
| **Swarm** | Peers + shared memory |
| **Debate** | Cross-check answers |

**MCP** = tools · **A2A** = agent-to-agent

</div>

<div class="space-y-1">

```mermaid {scale: 0.5}
graph TB
    SUP["🧭 Supervisor"] --> OBS["👁 Observer"]
    SUP --> DIAG["🔍 Diagnoser"]
    SUP --> PLAN["📋 Planner"]
    SUP --> EXE["⚙️ Executor"]
    SUP --> REV["✅ Reviewer"]
    style SUP fill:#7c3aed,color:#fff
    style OBS fill:#2563eb,color:#fff
    style DIAG fill:#2563eb,color:#fff
    style PLAN fill:#ea580c,color:#fff
    style EXE fill:#ea580c,color:#fff
    style REV fill:#16a34a,color:#fff
```

Specialised prompts + smaller context = better answers.

</div>

</div>

---

# Human-in-the-Loop

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### Autonomy spectrum

| Level | Role |
|:--|:--|
| **L0** | Advisor (reads) |
| **L1** | Suggests, human acts |
| **L2** | Approve & run |
| **L3** | Acts in narrow scope |
| **L4** | Fully autonomous |

Promotion requires **evidence** (Ch 11).

</div>

<div class="space-y-1">

### Approval gate (6 parts)
1. Change **summary** in plain English
2. **Diff** of proposed action
3. **Impact** estimate (devices, traffic)
4. **Rollback** plan
5. **TTL** (auto-cancel if no decision)
6. **Audit** record (who, when, why)

UI: Slack / ticket / web — choose the one operators already live in.

</div>

</div>

---

# Closed-Loop Automation

<div class="mt-1">

```mermaid {scale: 0.6}
graph LR
    DET["📡 Detect<br/>(symptom)"] --> DEC["🧠 Decide<br/>(LLM + RAG)"]
    DEC --> APP{"🛡 Guardrails<br/>OK?"}
    APP -->|Yes| ACT["⚙️ Act"]
    APP -->|No| HITL["👤 HITL"]
    ACT --> VER["✅ Verify"]
    VER -->|Pass| DONE["🟢 Done"]
    VER -->|Fail| ROLL["↩️ Rollback"]
    style DET fill:#2563eb,color:#fff
    style DEC fill:#7c3aed,color:#fff
    style APP fill:#ea580c,color:#fff
    style ACT fill:#ea580c,color:#fff
    style HITL fill:#7c3aed,color:#fff
    style VER fill:#16a34a,color:#fff
    style DONE fill:#16a34a,color:#fff
    style ROLL fill:#ea580c,color:#fff
```

</div>

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### Self-heal rubric
- Symptom is **observable**
- Cause is **bounded**
- Action is **scoped + reversible**
- Verification is **automatic**

</div>

<div class="space-y-1">

### Kill switch
- One command stops all agents
- Documented · tested · drilled
- Logs preserved for post-mortem

</div>

</div>

---
layout: section
---

# Part IV
## Production

---

# Observability & Evaluation

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### Trace tree

```mermaid {scale: 0.55}
graph TB
    R["🌐 Run"] --> P["📋 Plan"]
    R --> T1["🔧 tool: bgp"]
    R --> T2["🔧 tool: route"]
    R --> RAG["📚 RAG search"]
    R --> ANS["✅ Answer"]
    style R fill:#7c3aed,color:#fff
    style P fill:#ea580c,color:#fff
    style T1 fill:#2563eb,color:#fff
    style T2 fill:#2563eb,color:#fff
    style RAG fill:#2563eb,color:#fff
    style ANS fill:#16a34a,color:#fff
```

**Tools:** OpenTelemetry · Phoenix · LangSmith

</div>

<div class="space-y-1">

### Four metric families
- **Quality** — accuracy, faithfulness
- **Efficiency** — latency, tool count
- **Cost** — $ / run
- **Reliability** — success rate

### Eval styles
- **Offline** — fixed dataset, LLM judge
- **Shadow** — run silently in prod
- **Canary** — 1% → 10% → 100%
- **A/B** — compare variants

</div>

</div>

---

# Safety, Security, Guardrails

<div class="grid grid-cols-2 gap-4 mt-2 text-xs leading-tight">

<div class="space-y-1">

### OWASP LLM Top 10 → NetOps

| Risk | NetOps example |
|:--|:--|
| Prompt injection | Malicious log line |
| Insecure output | Unchecked CLI string |
| Training data leak | Secrets in prompts |
| Model DoS | Tool-call loop |
| Supply chain | Rogue MCP server |
| Excessive agency | Wildcard ACL push |

</div>

<div class="space-y-1">

### Layered defence

```mermaid {scale: 0.5}
graph LR
    IN["📥 Input"] --> F1["🛡 Sanitise"]
    F1 --> POL["📜 Policy"]
    POL --> LLM["🦅 LLM"]
    LLM --> F2["🛡 Validate output"]
    F2 --> SBX["📦 Sandbox tool"]
    SBX --> NET["🌐 Network"]
    style IN fill:#2563eb,color:#fff
    style F1 fill:#ea580c,color:#fff
    style POL fill:#ea580c,color:#fff
    style LLM fill:#7c3aed,color:#fff
    style F2 fill:#ea580c,color:#fff
    style SBX fill:#ea580c,color:#fff
    style NET fill:#16a34a,color:#fff
```

NeMo Guardrails · Llama Guard · Presidio · Lakera · Rebuff

</div>

</div>

---

# Cost, Performance, Model Selection

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### Cost equation

$$ C = N_\text{runs} \cdot (T_\text{in} \cdot p_\text{in} + T_\text{out} \cdot p_\text{out}) $$

| Tier | Use case |
|:--|:--|
| **Small** (7–8B local) | Classify, extract |
| **Medium** (mini SaaS) | RCA, summarise |
| **Frontier** | Complex multi-step |

### Routing pattern

Small first → escalate only if **confidence low** or **task hard**.

</div>

<div class="space-y-1">

### Quality vs cost

```mermaid {scale: 0.55}
graph LR
    S["🐣 Small"] --> M["🐤 Medium"]
    M --> F["🦅 Frontier"]
    style S fill:#16a34a,color:#fff
    style M fill:#2563eb,color:#fff
    style F fill:#7c3aed,color:#fff
```

### Reduce cost
- Prompt **caching**
- **Quantised** local models (Ollama)
- **Memoise** tool results
- **Distil** repeating tasks

Local: Ollama · vLLM · TGI · NIM

</div>

</div>

---

# AgentOps Lifecycle

<div class="mt-1">

```mermaid {scale: 0.6}
graph LR
    DEV["✏️ Dev"] --> LINT["🧪 Lint"]
    LINT --> SMOKE["💨 Smoke evals"]
    SMOKE --> FULL["📊 Full eval"]
    FULL --> SHA["👤 Shadow"]
    SHA --> CAN["🐤 Canary 1-10%"]
    CAN --> PROD["🌐 100%"]
    PROD -.->|regression| ROLL["↩️ Auto-rollback"]
    style DEV fill:#2563eb,color:#fff
    style LINT fill:#ea580c,color:#fff
    style SMOKE fill:#ea580c,color:#fff
    style FULL fill:#ea580c,color:#fff
    style SHA fill:#7c3aed,color:#fff
    style CAN fill:#7c3aed,color:#fff
    style PROD fill:#16a34a,color:#fff
    style ROLL fill:#ea580c,color:#fff
```

</div>

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### Version everything
Prompts · tools · models · RAG index · graph · guardrails · eval set

</div>

<div class="space-y-1">

### Improvement flywheel
Trace → curate failures → add to eval → fix → re-evaluate → promote autonomy

</div>

</div>

---

# Ethics, Limitations, Future

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### Accountability
Humans + organisations remain responsible. The agent is **not** a defence.

### Today's limits
- Hallucination · bias · drift
- No true reasoning
- Persuasibility (prompt injection)
- Cost & latency tax

### Regulation (EU)
GDPR · **EU AI Act** (high-risk) · NIS2 · DORA

</div>

<div class="space-y-1">

### Workforce shift

| Shrinks | Grows |
|:--|:--|
| Manual CLI | Designing safe automation |
| Tier-1 triage | Supervising agent fleets |
| Vendor memorisation | Reasoning + data + safety |

### Where it's heading
HITL agents → self-healing scoped loops → intent-driven networks → largely autonomous (with humans authoring policy)

</div>

</div>

---
layout: section
---

# Part V
## Capstone Projects

---

# Capstone Project Ideas

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### Project menu
- **A** — BGP troubleshooter (MCP)
- **B** — NetFlow anomaly + RAG
- **C** — Multi-agent incident response
- **D** — Config drift + HITL
- **E** — Wi-Fi roaming RCA
- **F** — Compliance auditor
- **G** — Synthetic probe orchestrator
- **H** — NOC knowledge concierge

</div>

<div class="space-y-1">

### Deliverables (all projects)
- 1-page **charter**
- Architecture **diagram**
- Reproducible **repo**
- **Toolset** spec
- **Eval set** ≥ 30 cases
- Eval **report** + safety review
- 5-min **demo** + post-mortem

### Grading weights
Correctness 25 · Safety 20 · Engineering 20 · Eval rigour 15 · Demo 10 · Originality 10

</div>

</div>

---

# Suggested 12-Week Timeline

<div class="mt-1">

```mermaid {scale: 0.55}
graph LR
    W1["W1<br/>Charter+lab"] --> W2["W2<br/>Tools+data"]
    W2 --> W3["W3<br/>First agent"]
    W3 --> W4["W4<br/>RAG/multi"]
    W4 --> W5["W5<br/>Guardrails"]
    W5 --> W6["W6<br/>Eval set"]
    W6 --> W7["W7<br/>Iterate"]
    W7 --> W8["W8<br/>HITL/loop"]
    W8 --> W9["W9<br/>Observ."]
    W9 --> W10["W10<br/>Shadow/canary"]
    W10 --> W11["W11<br/>Polish"]
    W11 --> W12["W12<br/>Defence"]
    style W1 fill:#2563eb,color:#fff
    style W6 fill:#ea580c,color:#fff
    style W10 fill:#ea580c,color:#fff
    style W12 fill:#16a34a,color:#fff
```

</div>

<div class="grid grid-cols-2 gap-4 mt-2 text-sm leading-tight">

<div class="space-y-1">

### Defence checklist
- Live end-to-end demo
- Every tool call appears in trace UI
- Name every metric on eval report
- One bug + post-mortem

</div>

<div class="space-y-1">

### Show maturity
- One threat you defended against
- Justify your autonomy level
- Articulate **why** + **where** it fails

</div>

</div>

---

# Reference Lab Stack

<div class="mt-1">

```mermaid {scale: 0.55}
graph LR
    CL["🧪 Containerlab"] --> NW["🌐 Routers<br/>(SR Linux/FRR)"]
    NW --> EX["📥 Exporters<br/>(gNMI/SNMP)"]
    EX --> PR[("📈 Prometheus")]
    NW --> LK[("📝 Loki")]
    PR & LK --> AG["🤖 Agent<br/>(LangGraph)"]
    AG --> MCP["🔌 MCP servers"]
    AG --> LL["🦅 LLM<br/>(SaaS/Ollama)"]
    AG --> TR[("🔭 Phoenix")]
    style CL fill:#2563eb,color:#fff
    style NW fill:#2563eb,color:#fff
    style EX fill:#ea580c,color:#fff
    style PR fill:#ea580c,color:#fff
    style LK fill:#ea580c,color:#fff
    style AG fill:#7c3aed,color:#fff
    style MCP fill:#7c3aed,color:#fff
    style LL fill:#7c3aed,color:#fff
    style TR fill:#16a34a,color:#fff
```

</div>

<div class="mt-3 text-center text-sm text-gray-500">
All open source · runs on a laptop · matches every capstone project
</div>

---

# Appendices

<div class="grid grid-cols-3 gap-4 mt-4 text-sm">

<div class="border border-purple-400 rounded-lg p-3">

### A — Python & Async
asyncio · pydantic · tenacity · logging · pytest

### B — Lab Setup
Containerlab · NetBox · Prom/Loki · Phoenix · Ollama

</div>

<div class="border border-orange-400 rounded-lg p-3">

### C — LLM APIs
OpenAI · Anthropic · Gemini · Ollama compat · tool calling · structured output

### D — Glossary
Every key term cross-referenced to chapters

</div>

<div class="border border-blue-400 rounded-lg p-3">

### E — Further Reading
Foundational papers · safety · standards · books · blogs

### F — Sample Datasets
CAIDA · RIPE RIS · LogHub · Batfish examples · synthetic recipes

</div>

</div>

---
layout: center
class: text-center
---

# Summary

<div class="grid grid-cols-2 gap-6 mt-6 text-left">

<div>

### What we covered

✅ From scripts to **agents**
✅ Tools, RAG, multi-agent
✅ HITL & closed loops
✅ Eval, safety, AgentOps
✅ Ethics + the road ahead

</div>

<div>

### Key takeaways

- Agents = **LLM + tools + loop**
- **Safety** beats cleverness
- **Evaluate** before you promote autonomy
- Humans stay **accountable**
- Network engineers run the **agents** of tomorrow

</div>

</div>

---
layout: center
class: text-center
---

# Thank you

<div class="pt-8">
  <span class="text-orange-400 font-bold text-xl">Agentic Ops for Network Engineers</span>
</div>

<div class="pt-6 text-sm text-gray-400">
Questions? · Full handout in <code>handout/</code>
</div>
