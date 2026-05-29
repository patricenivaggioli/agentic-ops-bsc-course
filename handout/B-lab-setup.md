# Appendix B — Lab Setup

> Practical recipes to spin up a network lab for your capstone. All open-source. Tested on Linux & macOS (Apple Silicon — see notes).

---

## B.1 Choosing a lab platform

| Tool | Best for | Notes |
|:--|:--|:--|
| **Containerlab** | Recommended. Container-based topologies, very fast | Linux-native; on macOS use Lima or a Linux VM |
| **GNS3** | GUI-driven, mixes images | Heavier; needs GNS3 VM |
| **EVE-NG** | Multi-vendor IOS-XR/JunOS labs | Free CE edition; needs Intel/AMD CPU |
| **Cisco Modeling Labs (CML)** | Vendor-supported, official IOS images | Commercial licence |
| **Pure Docker** | Lightweight protocol-only labs (FRR) | Quickest for BGP/OSPF demos |

We'll use **Containerlab** as the default.

---

## B.2 Containerlab quick start

### Install (Linux)

```bash
bash -c "$(curl -sL https://get.containerlab.dev)"
```

### Install (macOS — via Lima Linux VM)

```bash
brew install lima
limactl start --name=clab template://ubuntu-lts
limactl shell clab
# inside the VM:
bash -c "$(curl -sL https://get.containerlab.dev)"
```

### Verify

```bash
containerlab version
docker version
```

---

## B.3 Sample topology — 6-router BGP lab

`bgp-lab.clab.yml`:

```yaml
name: bgp-lab
topology:
  nodes:
    r1: { kind: linux, image: frrouting/frr:latest }
    r2: { kind: linux, image: frrouting/frr:latest }
    r3: { kind: linux, image: frrouting/frr:latest }
    r4: { kind: linux, image: frrouting/frr:latest }
    rr1: { kind: linux, image: frrouting/frr:latest }
    rr2: { kind: linux, image: frrouting/frr:latest }
  links:
    - endpoints: [r1:eth1, r2:eth1]
    - endpoints: [r2:eth2, rr1:eth1]
    - endpoints: [r3:eth1, rr1:eth2]
    - endpoints: [r3:eth2, rr2:eth1]
    - endpoints: [r4:eth1, rr2:eth2]
```

Deploy / destroy:

```bash
sudo containerlab deploy -t bgp-lab.clab.yml
sudo containerlab destroy -t bgp-lab.clab.yml
```

Access a node:

```bash
docker exec -it clab-bgp-lab-r1 vtysh
```

### Tip — vendor images

| Image | Notes |
|:--|:--|
| `frrouting/frr` | Free, BGP/OSPF/IS-IS |
| `ghcr.io/nokia/srlinux` | Free, modern model-driven (gNMI) |
| `ceos:latest` (Arista cEOS) | Free with Arista account |
| Cisco XRd, Juniper cRPD | Vendor account required |

---

## B.4 Telemetry side-car

Add Prometheus + Grafana:

```yaml
prometheus:
  kind: linux
  image: prom/prometheus
  binds: [./prom.yml:/etc/prometheus/prometheus.yml]
grafana:
  kind: linux
  image: grafana/grafana
  ports: ["3000:3000"]
```

For gNMI streaming, run a collector (e.g. `gnmic` or `telegraf` with `gnmi` input) and write to Prometheus.

---

## B.5 NetBox (source of truth)

```bash
git clone -b release https://github.com/netbox-community/netbox-docker.git
cd netbox-docker
docker compose up -d
# UI: http://localhost:8000 — admin / admin
```

Seed it via the NetBox API or `pynetbox`.

---

## B.6 Observability for the agent

| Need | Tool |
|:--|:--|
| Traces | Phoenix (`pip install arize-phoenix && phoenix serve`) |
| Metrics on agent | Prometheus client |
| Logs | Loki or just `journalctl` |
| Eval runs | LangSmith (SaaS) or Phoenix datasets |

---

## B.7 Local LLMs

### Ollama (easiest)

```bash
brew install ollama   # or: curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.1:8b
ollama pull qwen2.5:7b
ollama run llama3.1:8b "Explain BGP in two sentences."
```

OpenAI-compatible endpoint at `http://localhost:11434/v1`.

### vLLM (server, GPU)

```bash
pip install vllm
python -m vllm.entrypoints.openai.api_server \
  --model meta-llama/Meta-Llama-3.1-8B-Instruct \
  --max-model-len 8192
```

---

## B.8 Fault injection cheat-sheet (FRR)

| Fault | Command |
|:--|:--|
| Neighbor down | `shutdown` under `router bgp` |
| Slow link | `tc qdisc add dev eth1 root netem delay 100ms` |
| Loss | `tc qdisc add dev eth1 root netem loss 5%` |
| Black-hole route | `ip route 10.0.0.0/24 Null0` |
| Filter | add `route-map` denying the prefix |

Always **document** what you injected — the agent's job is to find it; yours is to grade correctly.

---

## B.9 Topology presets to start from

| Project (Ch 16) | Suggested topology |
|:--|:--|
| BGP troubleshooting | §B.3 (6-router) |
| NetFlow anomaly | 2 routers + 3 hosts + 1 flow collector (`goflow2`) |
| Multi-agent IR | §B.3 + Wi-Fi mock as host scripts |
| Config drift | Any 3-router lab + NetBox |
| Wi-Fi roaming | Pure mock (Python script generating events) |
| Compliance | Pure config-file lab (no live devices needed) |
| Synthetic probes | 4 hosts + traffic-control delays |
| Knowledge concierge | No topology — corpus only |

---

## B.10 Troubleshooting tips

- "Permission denied" → containerlab needs `sudo` or membership in `docker`.
- On macOS, ARM images differ; many vendor images are amd64 only — use Lima with x86_64 emulation or run on a Linux x86 box.
- If Phoenix or Grafana isn't reachable, check that ports are mapped to the host in the clab YAML.
- Always `clab destroy` between sessions to free interfaces.
