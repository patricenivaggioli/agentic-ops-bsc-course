# Chapter 14 — AgentOps: CI/CD and Lifecycle for Agents

> **Learning objectives:** Treat agents as living systems with their own SDLC: version prompts/tools/models, build CI/CD pipelines (regression tests + canary), run continuous evaluation, and feed production signals back into improvement.

---

## 14.1 What is AgentOps?

AgentOps = **DevOps for agentic systems**. It extends MLOps with the specifics of LLM-powered agents:

| MLOps | AgentOps |
|:--|:--|
| Versions models + datasets | + prompts, tools, RAG indices, agent graphs |
| Trains + deploys models | + prompt rollouts, A/B, canary by traffic % |
| Monitors data drift | + tool-call drift, prompt-injection rate |
| Re-trains on schedule | + re-tunes prompts; refreshes RAG; updates eval set |

---

## 14.2 What needs to be versioned

```mermaid
flowchart TB
    A["🤖 Agent release vX.Y.Z"] --> P["📝 System & tool prompts"]
    A --> T["🔧 Tool catalog (schemas)"]
    A --> M["🧠 Model ids (pinned)"]
    A --> R["📚 RAG index version"]
    A --> G["🕸️ Agent graph / orchestration"]
    A --> GD["✅ Guardrail rules"]
    A --> EV["📋 Eval dataset version"]
    A --> CFG["⚙️ Config (budgets, autonomy levels)"]

    style A fill:#a29bfe,stroke:#6c5ce7,color:#000
    style P fill:#74b9ff,stroke:#0984e3,color:#000
    style T fill:#74b9ff,stroke:#0984e3,color:#000
    style M fill:#74b9ff,stroke:#0984e3,color:#000
    style R fill:#74b9ff,stroke:#0984e3,color:#000
    style G fill:#74b9ff,stroke:#0984e3,color:#000
    style GD fill:#ffeaa7,stroke:#fdcb6e,color:#000
    style EV fill:#55efc4,stroke:#00b894,color:#000
    style CFG fill:#fab1a0,stroke:#e17055,color:#000
```

A reproducible release = **all of these pinned together**, in one Git commit (or one manifest).

### Example release manifest

```yaml
agent: nettriage
version: 1.5.0
model:
  planner: anthropic/claude-opus-4.7-2026-05
  worker:  openai/gpt-4.1-mini-2026-03
prompts:
  system: prompts/nettriage_system_v12.md
  worker: prompts/diagnoser_v7.md
tools: tools/catalog.v9.yaml
rag:
  collection: runbooks
  index_version: 2026-05-14T12:00Z
graph: graphs/nettriage_v3.py
guardrails: rails/v4.colang
config:
  budgets: {calls: 12, tokens: 30000, wall_s: 60}
  autonomy: L2
eval_dataset: evals/nettriage.v2026-05.jsonl
```

---

## 14.3 The CI/CD pipeline

```mermaid
flowchart LR
    PR["📥 PR (prompt/tool/model bump)"] --> L["🧹 Lint + unit tests"]
    L --> SM["💨 Smoke evals (fast)"]
    SM --> FE["🧪 Full eval suite"]
    FE --> SH["👻 Shadow on prod traffic"]
    SH --> CN["🐦 Canary (5\u201310 %)"]
    CN --> RO["🚀 Rollout (50 %\u2192100 %)"]
    RO --> MO["📡 Continuous monitoring"]
    MO -->|regression| RB["↩️ Auto-rollback"]

    style PR fill:#74b9ff,stroke:#0984e3,color:#000
    style L fill:#81ecec,stroke:#00cec9,color:#000
    style SM fill:#ffeaa7,stroke:#fdcb6e,color:#000
    style FE fill:#a29bfe,stroke:#6c5ce7,color:#000
    style SH fill:#a29bfe,stroke:#6c5ce7,color:#000
    style CN fill:#fab1a0,stroke:#e17055,color:#000
    style RO fill:#55efc4,stroke:#00b894,color:#000
    style MO fill:#55efc4,stroke:#00b894,color:#000
    style RB fill:#fab1a0,stroke:#e17055,color:#000
```

### Gate criteria per stage

| Stage | Pass criteria |
|:--|:--|
| Lint | Prompts compile, tool schemas valid |
| Unit | Each tool wrapper passes its tests |
| Smoke (10–30 cases) | ≥ baseline – ε on critical slices |
| Full eval | No slice regressed > 5 %; cost p95 not worse > 10 % |
| Shadow | Disagreement with prod < threshold; no safety violation |
| Canary | Online metrics (approval, rollback) ≥ baseline |
| Rollout | Sustained for 24 h before next ramp |

### Auto-rollback triggers

| Trigger | Action |
|:--|:--|
| Error rate > 2× baseline | Rollback |
| Rollback-by-operator rate > 2× | Rollback |
| Cost p95 > 3× baseline | Rollback |
| Safety violation detected | Immediate rollback + page |
| Latency p95 > SLO | Rollback if sustained > 10 min |

---

## 14.4 Prompt and tool versioning

### Prompts

- One file per prompt; include `prompt_id` + `version` in metadata.
- Embed `prompt_version` in every LLM call → traceable in logs.
- Diff-review prompts like code; require approval.
- Keep an archive of all deployed versions for replay.

### Tools

| Change | Rule |
|:--|:--|
| New tool | Minor bump; full eval |
| New required arg | **Major** bump; backward-incompatible |
| Description change | Minor — but eval (LLM tool selection is sensitive to descriptions) |
| Bugfix in implementation | Patch; eval still recommended |

> Tool descriptions are part of the **prompt** for the LLM. Treat them as such.

---

## 14.5 Continuous evaluation

Three layers in production.

```mermaid
flowchart TB
    L1["🟢 Live request"] --> L2["🤖 Agent run"] --> L3["📡 Trace span"]
    L3 --> Q1["⚡ Cheap online checks<br/>(schema, citation present, refused safely)"]
    L3 --> Q2["🧠 LLM-judge (sampled, async)"]
    L3 --> Q3["👤 Human spot-check<br/>(daily / weekly)"]
    Q1 & Q2 & Q3 --> D["📊 Dashboards + alerts"]

    style L1 fill:#74b9ff,stroke:#0984e3,color:#000
    style L2 fill:#a29bfe,stroke:#6c5ce7,color:#000
    style L3 fill:#81ecec,stroke:#00cec9,color:#000
    style Q1 fill:#ffeaa7,stroke:#fdcb6e,color:#000
    style Q2 fill:#ffeaa7,stroke:#fdcb6e,color:#000
    style Q3 fill:#ffeaa7,stroke:#fdcb6e,color:#000
    style D fill:#55efc4,stroke:#00b894,color:#000
```

| Layer | Latency | Coverage | Cost |
|:--|:--|:--|:--|
| Online checks | < 100 ms | 100 % | ~0 |
| LLM-judge sampled | seconds (async) | 5–20 % | $$ |
| Human spot-check | hours/days | 1–2 % | $$$ |

Combine all three: cheap broad coverage + deep but narrow human eyes.

---

## 14.6 The improvement flywheel

```mermaid
flowchart LR
    P["🚀 Prod traffic"] --> T["📋 Traces + signals"]
    T --> M["🔬 Mine failures<br/>(low score, rejected, rolled back)"]
    M --> D["📚 Add to eval dataset"]
    D --> H["💡 Hypothesise fix<br/>(prompt · tool · rag · model)"]
    H --> PR["🔧 PR + CI eval"]
    PR --> RO["🚀 Release"]
    RO --> P

    style P fill:#74b9ff,stroke:#0984e3,color:#000
    style T fill:#81ecec,stroke:#00cec9,color:#000
    style M fill:#a29bfe,stroke:#6c5ce7,color:#000
    style D fill:#55efc4,stroke:#00b894,color:#000
    style H fill:#ffeaa7,stroke:#fdcb6e,color:#000
    style PR fill:#fab1a0,stroke:#e17055,color:#000
    style RO fill:#55efc4,stroke:#00b894,color:#000
```

A healthy team ships **small, reviewed changes weekly**, not big rewrites yearly.

### Sources of fix hypotheses

| Signal | Common fix |
|:--|:--|
| Wrong tool chosen | Better tool description; reduce tool count |
| Hallucinated answer | Stricter prompt; add `say "I don't know"` rule |
| Stale RAG hit | Refresh index; add date filter |
| Long traces (>10 tool calls) | Lower budget; pre-summarise outputs |
| High rollback rate on a class | Tighten preconditions; add HITL gate |

---

## 14.7 Promoting autonomy safely

Autonomy (Ch 9) should be promoted **per action**, evidence-driven.

| Stage | Required evidence |
|:--|:--|
| L2 → L3 (timed veto) | 200+ runs at L2 with ≥ 95 % approval; rollback < 1 % |
| L3 → L4 (autonomous) | 1000+ runs at L3 with no safety incident; verify step in place |
| Any promotion | Documented post-mortem of *all* failures in the window |

Demotions must be just as easy: a single safety incident → demote.

---

## 14.8 Team practices

| Practice | Why |
|:--|:--|
| **Weekly eval review** | Look at regressions and qualitative samples together |
| **"Bad trace of the week"** | Pick one ugly trace, agree on a fix |
| **Prompt PR template** | Force "what changed / why / eval delta" |
| **Shared eval set** | Single source of truth across teams |
| **On-call rotation for the agent** | Like any production service |
| **Post-mortems for AI incidents** | Same rigour as for outages |

---

## 14.9 Anti-patterns

| Anti-pattern | Fix |
|:--|:--|
| Tweak prompt in production directly | Force everything through CI |
| Eval only on success rate | Add cost, latency, safety, slices |
| One person "owns the prompt" | Treat prompts as shared code |
| Big-bang release ("now with GPT-5!") | Shadow + canary, even for "obvious wins" |
| No replay capability | Build trace replay from day one |
| Manual rollbacks only | Automate trigger + rollback |

---

## Summary

```mermaid
mindmap
  root((Chapter 14<br/>Recap))
    Version everything
      Prompts · Tools · Models
      RAG · Graph · Guardrails
    CI/CD
      Lint \u2192 Smoke \u2192 Full \u2192 Shadow \u2192 Canary \u2192 Rollout
      Auto-rollback triggers
    Continuous eval
      Online checks (100 %)
      LLM-judge (sampled)
      Human spot-check
    Flywheel
      Mine failures \u2192 add to eval
      Hypothesise \u2192 PR \u2192 ship
    Autonomy promotion
      Evidence-driven
      Demotion fast
    Team
      Weekly review
      AI incident post-mortems
```

---

## Exercises

1. **Manifest.** Write a release manifest (per §14.2) for an imaginary "config-drift agent" you maintain.
2. **CI gates.** Define 5 quantitative pass/fail gates for your full eval stage.
3. **Tool versioning.** Renaming `query_logs` → `search_logs` and adding required arg `since`. What kind of version bump? What do you migrate?
4. **Rollback rule.** Design an auto-rollback rule combining error rate, rollback rate, and safety violations.
5. **Flywheel.** Choose one failure mode from production traces and walk through the §14.6 flywheel for it.
6. **Promotion.** Your `clear_arp` tool has run 300 times at L2 with 97 % approval, 0 rollbacks. Promote to L3? What additional checks would you add before doing so?
