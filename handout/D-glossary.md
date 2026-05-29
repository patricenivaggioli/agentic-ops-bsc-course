# Appendix D — Glossary

> Short definitions of every key term in this course. Cross-references in *italics*.

| Term | Definition |
|:--|:--|
| **A2A (Agent-to-Agent protocol)** | Open spec for *agents* to discover and call each other across vendors. Complements *MCP*. |
| **Action tool** | A *tool* that changes system state (e.g. push config). Requires safety controls. |
| **Agent** | A system that uses an *LLM* to decide which *tools* to call to achieve a goal. |
| **AgentOps** | DevOps practices applied to agents: versioning, CI/CD, eval, monitoring. (Ch 14) |
| **Autonomy level (L0–L4)** | Spectrum of human oversight, from advisor (L0) to fully autonomous (L4). (Ch 9) |
| **Batfish** | Open-source network configuration analyser; a form of digital twin. |
| **BGP** | Border Gateway Protocol — Internet routing. |
| **Canary** | Rolling out a new agent version to a small slice of traffic first. (Ch 14) |
| **Chain-of-thought (CoT)** | Prompting style where the model reasons step by step. |
| **Charter** | Written scope, capabilities, and limits of an agent. (Ch 7) |
| **Chunking** | Splitting a document into smaller pieces for retrieval. (Ch 6) |
| **Closed loop** | Detect → decide → act → verify cycle that runs without humans for each iteration. (Ch 10) |
| **Containerlab** | Tool to spin up network topologies in containers. (Appendix B) |
| **Cosine similarity** | Common metric for comparing embedding vectors. |
| **Cross-encoder** | Re-ranking model that scores a (query, doc) pair jointly. (Ch 6) |
| **Debate pattern** | Multi-agent pattern where agents argue then converge. (Ch 8) |
| **Drift** | Divergence between intended and actual configuration. |
| **Embedding** | Vector representation of text used for semantic search. |
| **Eval set** | Curated dataset used to score agent quality. (Ch 11) |
| **Faithfulness** | Whether an answer is supported by the cited sources. (Ch 6 / Ch 11) |
| **Few-shot** | Including example input/outputs in the prompt. |
| **Forward / backward verification** | Checking state and impact after an action. (Ch 10) |
| **Function calling** | LLM produces a structured call to a *tool*. |
| **gNMI** | gRPC Network Management Interface — streaming telemetry & config. |
| **Guardrail** | Policy enforced around the LLM (input/output checks, allowlists). (Ch 12) |
| **Hallucination** | Confident but incorrect LLM output. |
| **HITL (Human-in-the-Loop)** | Human approval/edit step inside the agent loop. (Ch 9) |
| **Hybrid search** | Combining lexical (BM25) and vector retrieval, often via RRF. (Ch 6) |
| **IBN (Intent-Based Networking)** | Specifying *what* the network should do, not *how*. (Ch 10) |
| **Idempotent** | Same effect whether run once or N times — safer for retries. |
| **Indirect prompt injection** | Malicious instructions hidden inside data the agent reads (e.g. logs). |
| **Inference** | Running a trained model to produce output. |
| **Kill switch** | Mechanism to instantly disable an agent or revert to manual. |
| **Knowledge tool** | A *tool* that returns information from a corpus (often RAG). |
| **LangGraph** | Open-source state-machine framework for agent orchestration. |
| **LLM** | Large Language Model. |
| **MCP (Model Context Protocol)** | Open protocol for tools/resources/prompts exposed to LLMs. (Ch 5) |
| **MTTR** | Mean Time To Repair. |
| **Multi-agent** | Architecture with multiple specialised agents. (Ch 8) |
| **NeMo Guardrails** | NVIDIA framework for policy enforcement around LLMs. |
| **NETCONF / RESTCONF** | Standard protocols for device configuration. |
| **NetBox** | Open-source DCIM / IPAM — common source of truth. |
| **NetFlow / IPFIX / sFlow** | Flow telemetry protocols. |
| **Observability** | Ability to understand a system from its outputs (logs/metrics/traces). |
| **OpenConfig** | Vendor-neutral YANG data models. |
| **OpenTelemetry (OTel)** | Open standard for traces/metrics/logs. |
| **Pairwise judge** | LLM-as-judge variant comparing two candidate outputs. (Ch 11) |
| **Phoenix** | Open-source LLM observability (Arize). |
| **Plan-and-Execute** | Pattern: first plan a sequence, then execute. (Ch 3) |
| **Prompt injection** | Attack where attacker text tries to override system instructions. (Ch 12) |
| **RAG (Retrieval-Augmented Generation)** | Retrieving documents to ground LLM answers. (Ch 6) |
| **RBAC / ABAC / ReBAC** | Access-control models (role/attribute/relationship-based). |
| **ReAct** | Reason + Act loop pattern. (Ch 3) |
| **Re-ranker** | Second-stage retrieval model improving top results. (Ch 6) |
| **Reflection** | Pattern where agent critiques and revises its own output. (Ch 3) |
| **RFC 9272 / 9273** | YANG and gNMI-related references (illustrative). |
| **Router (model router)** | Picks the cheapest model that meets quality bar. (Ch 13) |
| **Runbook** | Documented procedure to handle a recurring incident. |
| **Shadow mode** | New agent runs in parallel without acting, for comparison. (Ch 14) |
| **Single source of truth (SSoT)** | Authoritative inventory of intended state (often NetBox/Git). |
| **SLI / SLO / SLA** | Service-level indicators / objectives / agreements. |
| **Sliced eval** | Computing metrics per sub-population (vendor, severity, …). (Ch 11) |
| **Streaming telemetry** | Push-based device telemetry (gNMI/Kafka), opposite of polling. |
| **Supervisor agent** | Top-level orchestrator delegating to specialists. (Ch 8) |
| **Synthetic probe** | Active test (ping/HTTP) used to measure SLOs. |
| **Telemetry pipeline** | Collector → transport → store → query. (Ch 4) |
| **Tool** | A function/HTTP endpoint the agent can call. (Ch 5) |
| **Trace** | Tree of spans capturing a single agent run. (Ch 11) |
| **Vector store** | Database optimised for similarity search (Chroma, Qdrant, pgvector). |
| **YANG** | Data modelling language for network management. |
