# Appendix E — Further Reading

> A curated list of references, organised by theme. Mix of foundational papers, practitioner books, blogs, and standards. URLs intentionally minimal — search by title.

---

## E.1 Foundational LLM & agent papers

- Vaswani et al., *Attention Is All You Need* (2017) — the original transformer.
- Brown et al., *Language Models are Few-Shot Learners* (GPT-3, 2020).
- Wei et al., *Chain-of-Thought Prompting Elicits Reasoning* (2022).
- Yao et al., *ReAct: Synergizing Reasoning and Acting in Language Models* (2022).
- Shinn et al., *Reflexion: Language Agents with Verbal Reinforcement Learning* (2023).
- Schick et al., *Toolformer* (2023).
- Lewis et al., *Retrieval-Augmented Generation for Knowledge-Intensive NLP* (2020).
- Gao et al., *Retrieval-Augmented Generation for Large Language Models: A Survey* (2023+).
- Park et al., *Generative Agents* (2023).
- Madaan et al., *Self-Refine* (2023).

## E.2 Multi-agent & orchestration

- Wu et al., *AutoGen* (Microsoft, 2023).
- Hong et al., *MetaGPT* (2023).
- Du et al., *Improving Factuality and Reasoning via Multi-Agent Debate* (2023).
- *LangGraph* documentation and tutorials.
- *CrewAI* documentation.
- *Pydantic AI* documentation.
- NVIDIA *NeMo Agent Toolkit* documentation.

## E.3 RAG & embeddings

- Karpukhin et al., *Dense Passage Retrieval* (2020).
- Izacard & Grave, *Atlas / Fusion-in-Decoder*.
- Reimers & Gurevych, *Sentence-BERT* (2019).
- *BGE* and *E5* embedding family papers.
- *ColBERT v2* (late-interaction retrieval).
- Cormack, Clarke, Büttcher, *Reciprocal Rank Fusion* (2009).

## E.4 Evaluation & observability

- Liu et al., *G-Eval* (LLM-as-judge, 2023).
- Zheng et al., *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena* (2023).
- *RAGAS* — RAG evaluation framework.
- *TruLens* docs and examples.
- *Phoenix* (Arize) docs.
- *LangSmith* documentation.
- *DeepEval* documentation.
- OpenTelemetry GenAI conventions (in-progress spec).

## E.5 Safety, security, guardrails

- OWASP *LLM Top 10* (2025 version).
- NIST AI Risk Management Framework (AI RMF 1.0).
- Greshake et al., *Not what you've signed up for* (indirect prompt injection, 2023).
- Anthropic, *Constitutional AI* (2022).
- NVIDIA *NeMo Guardrails* technical report.
- Meta *Llama Guard* model card.
- *Guardrails AI* documentation.
- *Presidio* (Microsoft) for PII redaction.

## E.6 Standards & protocols

- *Model Context Protocol* (modelcontextprotocol.io).
- *Agent2Agent (A2A) Protocol*.
- OpenAI *Function Calling* docs; Anthropic *Tool use*; Google Gemini *Function calling*.
- IETF *YANG* (RFC 7950), *NETCONF* (RFC 6241), *RESTCONF* (RFC 8040).
- *OpenConfig* models.
- IETF *gNMI* / *gNOI* specs.
- IETF *IPFIX* (RFC 7011).
- TM Forum *Autonomous Networks* levels.
- ETSI *Zero-touch Service Management* (ZSM).

## E.7 NetOps automation & telemetry

- Edelman, Lowe, Oswalt, *Network Programmability and Automation*, O'Reilly.
- Tischer & Gooley, *Cisco Certified DevNet Associate DEVASC 200-901*.
- *NetBox* and *Nautobot* documentation.
- *Batfish* docs and academic papers.
- Cloudflare, Meta, LinkedIn engineering blogs on network automation.

## E.8 SRE / DevOps foundations

- Beyer et al., *Site Reliability Engineering* (Google, 2016) — free online.
- *The SRE Workbook* (Google).
- Forsgren, Humble, Kim, *Accelerate*.
- Humble & Farley, *Continuous Delivery*.

## E.9 Ethics, policy, society

- Mitchell et al., *Model Cards for Model Reporting* (2019).
- Gebru et al., *Datasheets for Datasets* (2018).
- EU *AI Act* official text (2024).
- EU *NIS2 Directive* official text.
- AI Now Institute reports.
- Bender et al., *On the Dangers of Stochastic Parrots* (2021).

## E.10 Blogs & newsletters worth following

| Source | Topic |
|:--|:--|
| *Simon Willison's Weblog* | Pragmatic LLM engineering |
| *Eugene Yan* | Applied ML/LLM |
| *LangChain* / *LlamaIndex* / *Anthropic* / *OpenAI* engineering blogs | Frameworks, patterns |
| *Latent Space* (podcast + newsletter) | Industry trends |
| *Cloudflare blog* — Network category | Real-world large-scale NetOps |
| *Meta engineering* (network infra posts) | Datacentre operations |
| *RIPE Labs*, *APNIC blog* | BGP, routing, measurements |

## E.11 Courses and tutorials (free / freemium)

- DeepLearning.AI short courses on agents and RAG.
- LangChain Academy.
- *Hugging Face Agents Course*.
- Stanford *CS25: Transformers United*.
- Karpathy's *Neural Networks: Zero to Hero* (YouTube).
- Cisco DevNet learning labs (network automation focus).

---

> The half-life of LLM tooling is short — always check publication dates and prefer official docs over blog snippets for code you put in production.
