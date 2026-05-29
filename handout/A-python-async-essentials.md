# Appendix A — Python & Async Essentials for Agents

> A focused refresher of the Python features you will use most often when building agents. Assumes you know basic Python (functions, classes, dicts, lists).

---

## A.1 Environment

Use **Python 3.11+** and an isolated virtual environment.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install langgraph langchain-openai pydantic httpx rich
```

For reproducibility, pin dependencies:

```bash
pip freeze > requirements.txt
```

Or use **uv** / **poetry** for faster, lock-file-based workflows.

---

## A.2 Type hints and dataclasses

Type hints document intent and unlock validation tools.

```python
from dataclasses import dataclass

@dataclass
class Device:
    name: str
    mgmt_ip: str
    role: str = "leaf"
```

Use `typing` for richer types:

```python
from typing import Optional, Literal

Severity = Literal["info", "warn", "error", "critical"]

def open_ticket(title: str, severity: Severity, owner: Optional[str] = None) -> str:
    ...
```

---

## A.3 Pydantic — for tool schemas

Agents need typed inputs/outputs. Pydantic v2 is the de-facto standard.

```python
from pydantic import BaseModel, Field

class GetBgpNeighborsInput(BaseModel):
    device: str = Field(..., description="Device hostname")
    vrf: str = Field("default", description="VRF name")

class BgpNeighbor(BaseModel):
    peer: str
    asn: int
    state: Literal["Established", "Idle", "Active"]
```

The model's JSON schema (`GetBgpNeighborsInput.model_json_schema()`) is what frameworks send to the LLM for tool calling.

---

## A.4 Async / await

LLM calls and HTTP calls are I/O bound. Use `async`:

```python
import asyncio, httpx

async def fetch(url: str) -> dict:
    async with httpx.AsyncClient(timeout=10) as client:
        r = await client.get(url)
        r.raise_for_status()
        return r.json()

async def main():
    urls = ["https://api.example.com/a", "https://api.example.com/b"]
    results = await asyncio.gather(*(fetch(u) for u in urls))
    print(results)

asyncio.run(main())
```

Rules of thumb:

| Do | Don't |
|:--|:--|
| Use `async` for network/disk I/O | Use `async` for CPU-bound work |
| `await` every coroutine | Mix `time.sleep` with async code |
| Bound parallelism with `asyncio.Semaphore` | Fire 1000 concurrent calls |
| Always set timeouts | Trust default infinite waits |

---

## A.5 Retries and backoff

Use `tenacity`:

```python
from tenacity import retry, wait_exponential, stop_after_attempt

@retry(wait=wait_exponential(min=1, max=10), stop=stop_after_attempt(3))
async def safe_fetch(url): ...
```

Retry only on transient errors (timeouts, 5xx). Never retry on 4xx auth failures.

---

## A.6 Logging (structured)

```python
import logging, json

class JsonFormatter(logging.Formatter):
    def format(self, record):
        return json.dumps({
            "ts": self.formatTime(record),
            "level": record.levelname,
            "msg": record.getMessage(),
            **getattr(record, "extra", {}),
        })

logger = logging.getLogger("agent")
h = logging.StreamHandler(); h.setFormatter(JsonFormatter())
logger.addHandler(h); logger.setLevel(logging.INFO)

logger.info("tool_call", extra={"extra": {"tool": "get_bgp_neighbors", "device": "r1"}})
```

Pair with OpenTelemetry for traces (see Ch 11).

---

## A.7 Errors and timeouts

```python
class ToolError(Exception):
    """Raised when a tool execution fails in a recoverable way."""

async def run_tool(...):
    try:
        return await asyncio.wait_for(do_work(), timeout=30)
    except asyncio.TimeoutError as e:
        raise ToolError("tool_timeout") from e
```

Agents should turn tool exceptions into **observations** the LLM can reason about, not crash.

---

## A.8 Testing

Use `pytest` + `pytest-asyncio`:

```python
import pytest

@pytest.mark.asyncio
async def test_fetch(monkeypatch):
    async def fake_get(self, url): ...
    monkeypatch.setattr("httpx.AsyncClient.get", fake_get)
    ...
```

Snapshot tests are useful for prompts (assert that the rendered prompt matches a stored fixture).

---

## A.9 Project layout (suggested)

```
agent/
  __init__.py
  config.py        # env, model names, autonomy
  prompts/         # versioned prompts (.j2 or .md)
  tools/
    bgp.py
    netflow.py
  rag/
    indexer.py
    retriever.py
  graph.py         # LangGraph / state machine
  app.py           # entry point
tests/
  test_tools.py
eval/
  dataset.jsonl
  runner.py
```

---

## A.10 Cheatsheet

| Need | Library |
|:--|:--|
| HTTP | `httpx` |
| Async | stdlib `asyncio` |
| Validation | `pydantic` |
| Retries | `tenacity` |
| Logging | stdlib `logging` + JSON |
| Tracing | `opentelemetry-sdk`, `arize-phoenix` |
| Vector DB | `chromadb`, `qdrant-client` |
| Agent framework | `langgraph`, `crewai`, `pydantic-ai`, NeMo AT |
| LLM clients | `openai`, `anthropic`, `ollama` |
| CLI | `typer` or `click` |
| Pretty output | `rich` |
