# Agentic Ops for Network Engineers — BSc Study Guide

> A bachelor-level course on agentic AI systems applied to network operations.
> From LLM fundamentals to closed-loop, multi-agent network automation.

## Audience

Bachelor-degree students with:
- A working knowledge of networking (routing, switching, basic protocols)
- Python fundamentals
- Familiarity with REST APIs and JSON
- Optional: prior exposure to ML/AI basics

## Repository layout

| Folder | What's inside |
|:--|:--|
| [`handout/`](./handout) | 16 chapter handouts + 6 appendices (Markdown). |
| [`notebooks/`](./notebooks) | Runnable Jupyter notebooks illustrating key concepts. |
| [`slidev/`](./slidev) | Slidev presentation deck mirroring the handouts. |
| [`agentic-ops-bsc-toc.md`](./agentic-ops-bsc-toc.md) | Full table of contents. |

## Course structure

| Part | Theme | Chapters |
|:--|:--|:--|
| I — Foundations | AI / agent basics | 1, 2, 3 |
| II — Network Context | Data, tools, RAG | 4, 5, 6 |
| III — Workflows | Building agents | 7, 8, 9, 10 |
| IV — Production | Eval, safety, cost, ops | 11, 12, 13, 14 |
| V — Forward | Ethics, capstones | 15, 16 |
| Appendices | Reference material | A–F |

## Quick start

### Notebooks

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install jupyter scikit-learn numpy
jupyter lab notebooks/
```

The notebooks run **fully offline**. The OpenAI / Ollama sections are optional and clearly marked.

### Slidev deck

```bash
cd slidev
npm install
npm run dev          # live preview on http://localhost:3030
npm run build        # static SPA in slidev/dist/
npm run export       # PDF export (requires Playwright)
```

## License

[MIT](./LICENSE). Educational use encouraged — please cite the source if you re-use diagrams or text.
