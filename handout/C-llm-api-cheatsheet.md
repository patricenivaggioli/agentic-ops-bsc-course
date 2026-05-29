# Appendix C — LLM API Cheatsheet

> Minimum-viable code for calling LLMs from Python: OpenAI, Anthropic, Google, and local (Ollama / vLLM). Covers chat, streaming, tool calling, and structured output.

> **Never hardcode API keys.** Always read from environment variables or a secret manager.

---

## C.1 Common conventions

```python
import os
OPENAI_KEY = os.environ["OPENAI_API_KEY"]
ANTHROPIC_KEY = os.environ["ANTHROPIC_API_KEY"]
GOOGLE_KEY = os.environ["GOOGLE_API_KEY"]
```

Store in `.env`, load with `python-dotenv`, never commit.

---

## C.2 OpenAI (and OpenAI-compatible)

### Chat

```python
from openai import OpenAI
client = OpenAI()  # uses OPENAI_API_KEY

resp = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a NetOps assistant."},
        {"role": "user", "content": "Explain ECMP in one paragraph."},
    ],
    temperature=0.2,
    max_tokens=300,
)
print(resp.choices[0].message.content)
```

### Streaming

```python
stream = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hi"}],
    stream=True,
)
for chunk in stream:
    delta = chunk.choices[0].delta.content or ""
    print(delta, end="", flush=True)
```

### Tool calling

```python
tools = [{
    "type": "function",
    "function": {
        "name": "get_bgp_neighbors",
        "description": "Return BGP neighbors for a device.",
        "parameters": {
            "type": "object",
            "properties": {
                "device": {"type": "string"},
                "vrf": {"type": "string", "default": "default"},
            },
            "required": ["device"],
        },
    },
}]

resp = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Show BGP neighbors on r1"}],
    tools=tools,
)
msg = resp.choices[0].message
if msg.tool_calls:
    call = msg.tool_calls[0]
    print(call.function.name, call.function.arguments)
```

### Structured output (JSON schema)

```python
from pydantic import BaseModel

class Verdict(BaseModel):
    cause: str
    confidence: float

resp = client.chat.completions.parse(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Classify: BGP session flapping every 30s"}],
    response_format=Verdict,
)
print(resp.choices[0].message.parsed)
```

### Local OpenAI-compatible (Ollama / vLLM)

```python
client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")
resp = client.chat.completions.create(
    model="llama3.1:8b",
    messages=[{"role": "user", "content": "Hello"}],
)
```

---

## C.3 Anthropic (Claude)

```python
import anthropic
client = anthropic.Anthropic()

resp = client.messages.create(
    model="claude-3-5-sonnet-latest",
    max_tokens=300,
    system="You are a NetOps assistant.",
    messages=[{"role": "user", "content": "What is BFD?"}],
)
print(resp.content[0].text)
```

### Tool calling

```python
tools = [{
    "name": "get_bgp_neighbors",
    "description": "Return BGP neighbors for a device.",
    "input_schema": {
        "type": "object",
        "properties": {"device": {"type": "string"}},
        "required": ["device"],
    },
}]

resp = client.messages.create(
    model="claude-3-5-sonnet-latest",
    max_tokens=300,
    tools=tools,
    messages=[{"role": "user", "content": "Neighbors on r1?"}],
)
for block in resp.content:
    if block.type == "tool_use":
        print(block.name, block.input)
```

---

## C.4 Google Gemini

```python
from google import genai
client = genai.Client(api_key=os.environ["GOOGLE_API_KEY"])

resp = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="Explain OSPF in two paragraphs.",
)
print(resp.text)
```

---

## C.5 Embeddings

```python
# OpenAI
emb = client.embeddings.create(
    model="text-embedding-3-small",
    input=["BGP session is flapping", "OSPF neighbor down"],
).data
vectors = [e.embedding for e in emb]
```

| Provider | Common models |
|:--|:--|
| OpenAI | `text-embedding-3-small`, `text-embedding-3-large` |
| Cohere | `embed-multilingual-v3.0` |
| Local | `nomic-embed-text` (via Ollama), `bge-large-en` |

---

## C.6 Parameters quick reference

| Param | Effect | Typical for NetOps |
|:--|:--|:--|
| `temperature` | Higher = more random | 0.1–0.3 |
| `top_p` | Nucleus sampling | 0.9 (or omit) |
| `max_tokens` | Output cap | 256–1024 |
| `seed` | Reproducibility (OpenAI) | Set for evals |
| `stop` | Stop sequences | Useful for tool patterns |
| `frequency_penalty` / `presence_penalty` | Repetition control | Usually 0 |

---

## C.7 Cost & latency monitoring

Always log:

```python
resp = client.chat.completions.create(...)
usage = resp.usage  # prompt_tokens, completion_tokens, total_tokens
```

Compute cost from a price table (see Ch 13). Record per-trace into Phoenix / LangSmith.

---

## C.8 Safety checklist (boilerplate)

- [ ] API key read from env, never logged
- [ ] Timeouts on every call (e.g. `httpx` 30s; provider clients accept `timeout=`)
- [ ] Retries only on 429 / 5xx (use `tenacity`)
- [ ] Hard cap on max_tokens and tool-call loop length
- [ ] Redact PII before sending (Ch 12)
- [ ] Log model name + version with every call (Ch 14)
