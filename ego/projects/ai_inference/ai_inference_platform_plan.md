# 🚀 AI Inference Platform — Build Plan

> A mini OpenAI/Anthropic-style LLM serving platform targeting **AI Engineering / AI Infra / MLOps internships**.

---

## 🗺️ Project Overview

You are building a platform that:
- Accepts API requests (like `/v1/chat/completions`)
- Routes them to LLM backends (local or remote models)
- Manages queuing, batching, and concurrency
- Auto-scales replicas based on load
- Exposes a monitoring dashboard with metrics

**End result**: A deployable, demo-able system that mirrors what companies like Anyscale, Modal, Together AI, and Replicate build internally.

---

## 🏗️ Architecture at a Glance

```
Client (curl / SDK / UI)
        │
        ▼
  [API Gateway]          ← FastAPI / Kong
        │
        ▼
  [Request Queue]        ← Redis / Celery / Ray
        │
        ▼
  [Inference Workers]    ← vLLM / llama.cpp / HuggingFace
        │
        ▼
  [Model Registry]       ← local weights / HuggingFace Hub
        │
  [Auto-Scaler]          ← Kubernetes HPA / Ray Autoscaler
        │
  [Monitoring]           ← Prometheus + Grafana
```

---

## 📋 Build Phases

---

### Phase 1 — Core API Server
**Goal**: Stand up an OpenAI-compatible REST API.

#### Steps
1. Initialize a FastAPI project with a `/v1/chat/completions` endpoint
2. Define request/response Pydantic schemas matching OpenAI's format
3. Implement streaming responses using `StreamingResponse` + SSE
4. Add API key auth middleware (simple token check to start)
5. Write basic unit tests for the endpoint contract

#### What to Learn
| Topic | Resource |
|---|---|
| FastAPI fundamentals | [FastAPI Docs](https://fastapi.tiangolo.com/) |
| OpenAI API schema | [OpenAI API Reference](https://platform.openai.com/docs/api-reference) |
| Pydantic v2 | [Pydantic Docs](https://docs.pydantic.dev/) |
| Server-Sent Events (SSE) | [MDN SSE Guide](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) |

---

### Phase 2 — LLM Backend Integration
**Goal**: Connect the API to real LLM inference.

#### Steps
1. Set up **vLLM** locally to serve an open model (Mistral 7B or Llama 3.1 8B)
2. Wire your FastAPI gateway to forward requests to vLLM's OpenAI-compatible server
3. Add model selection: parse the `model` field in the request and route to the right backend
4. Implement a model config file (YAML/JSON) to register available models
5. Test with `curl` and a simple Python client

#### What to Learn
| Topic | Resource |
|---|---|
| vLLM quickstart | [vLLM Docs](https://docs.vllm.ai/en/latest/) |
| LLM inference concepts (KV cache, batching, quantization) | [vLLM Blog](https://blog.vllm.ai/) |
| HuggingFace model loading | [HF Transformers Docs](https://huggingface.co/docs/transformers) |
| GGUF / llama.cpp (CPU fallback) | [llama.cpp GitHub](https://github.com/ggerganov/llama.cpp) |

---

### Phase 3 — Request Queue & Concurrency
**Goal**: Handle multiple simultaneous requests without crashing.

#### Steps
1. Add **Redis** as a broker
2. Use **Celery** (or Ray tasks) to offload inference to async workers
3. Implement a request queue with configurable max concurrency per model
4. Add a `/v1/queue/status` endpoint to expose queue depth
5. Load test with `locust` to find your breaking point

#### What to Learn
| Topic | Resource |
|---|---|
| Redis fundamentals | [Redis University](https://university.redis.io/) |
| Celery task queues | [Celery Docs](https://docs.celeryq.dev/) |
| Ray for distributed inference | [Ray Docs](https://docs.ray.io/) |
| Async Python (asyncio) | [Python asyncio Docs](https://docs.python.org/3/library/asyncio.html) |
| Load testing with Locust | [Locust Docs](https://locust.io/) |

---

### Phase 4 — Containerization & Orchestration
**Goal**: Make the whole system portable and reproducible.

#### Steps
1. Write a `Dockerfile` for the API gateway
2. Write a `Dockerfile` for the inference worker
3. Create a `docker-compose.yml` that wires: API + Redis + Worker + Grafana + Prometheus
4. Add health check endpoints (`/health`, `/ready`) to each service
5. Test the full stack locally with `docker compose up`

#### What to Learn
| Topic | Resource |
|---|---|
| Docker fundamentals | [Docker Getting Started](https://docs.docker.com/get-started/) |
| Docker Compose | [Compose Docs](https://docs.docker.com/compose/) |
| Multi-stage Docker builds | [Docker Multi-stage Guide](https://docs.docker.com/build/building/multi-stage/) |
| NVIDIA Container Toolkit (GPU in Docker) | [NVIDIA Docs](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) |

---

### Phase 5 — Kubernetes Deployment & Auto-Scaling
**Goal**: Deploy on Kubernetes with automatic replica scaling.

#### Steps
1. Write K8s `Deployment` + `Service` manifests for each component
2. Add a `ConfigMap` for model configs and a `Secret` for API keys
3. Configure **Horizontal Pod Autoscaler (HPA)** based on CPU/GPU utilization
4. Use `minikube` or `kind` locally, or deploy to a free GKE/EKS tier
5. Write a `Helm chart` to package the whole platform

#### What to Learn
| Topic | Resource |
|---|---|
| Kubernetes basics | [Kubernetes.io Tutorials](https://kubernetes.io/docs/tutorials/) |
| HPA & autoscaling | [HPA Docs](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) |
| Helm charts | [Helm Docs](https://helm.sh/docs/) |
| GPU scheduling in K8s | [K8s GPU Docs](https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/) |
| KServe (ML serving on K8s) | [KServe Docs](https://kserve.github.io/website/) |

---

### Phase 6 — Monitoring & Observability
**Goal**: Know what your platform is doing at all times.

#### Steps
1. Instrument FastAPI with **Prometheus** metrics (requests/sec, latency p50/p95/p99, queue depth, token throughput)
2. Deploy **Grafana** and build a dashboard
3. Add structured JSON logging with request IDs for traceability
4. Set up alerting rules (e.g., alert if p99 latency > 2s)
5. Implement distributed tracing with **OpenTelemetry**

#### What to Learn
| Topic | Resource |
|---|---|
| Prometheus + Python | [prometheus-fastapi-instrumentator](https://github.com/trallnag/prometheus-fastapi-instrumentator) |
| Grafana dashboards | [Grafana Docs](https://grafana.com/docs/) |
| LLM-specific metrics (TTFT, TPS, ITL) | [vLLM Metrics Docs](https://docs.vllm.ai/en/latest/serving/metrics.html) |
| OpenTelemetry Python | [OTel Python Docs](https://opentelemetry.io/docs/languages/python/) |
| Structured logging | [structlog](https://www.structlog.org/en/stable/) |

---

### Phase 7 — Developer-Facing UI & SDK
**Goal**: Make it usable and demo-able.

#### Steps
1. Build a simple **Playground UI** (React or plain HTML) with model selector + chat interface
2. Write a minimal Python SDK (`pip install your-sdk`) wrapping the API
3. Add a `/v1/models` endpoint listing available models
4. Write a `README.md` with quickstart, architecture diagram, and demo GIF
5. Deploy to a cloud VM and make it publicly accessible

#### What to Learn
| Topic | Resource |
|---|---|
| Building a chat UI | [Vercel AI SDK](https://sdk.vercel.ai/docs) |
| Writing Python SDKs | [Python Packaging Guide](https://packaging.python.org/en/latest/) |
| OpenAI Python SDK (for reference) | [openai-python GitHub](https://github.com/openai/openai-python) |

---

## 🧰 Full Tech Stack

| Layer | Technology |
|---|---|
| **API Gateway** | FastAPI, Uvicorn, Pydantic |
| **Inference Engine** | vLLM, llama.cpp |
| **Queue / Broker** | Redis, Celery (or Ray) |
| **Containerization** | Docker, Docker Compose |
| **Orchestration** | Kubernetes, Helm |
| **Auto-Scaling** | K8s HPA, KEDA |
| **Monitoring** | Prometheus, Grafana, OpenTelemetry |
| **Models** | HuggingFace Hub (Mistral 7B, Llama 3.1 8B) |
| **Testing** | pytest, Locust |
| **Cloud** | GCP / AWS free tier, or Hetzner VPS |

---

## 💡 What to Add to Stand Out for Internships

These additions will make your project genuinely impressive and map to real interview topics:

### 🔥 High-Impact Additions

| Feature | Why It Matters | Difficulty |
|---|---|---|
| **Continuous Batching** | Core vLLM optimization; shows you understand inference throughput | ⭐⭐⭐ |
| **Token Streaming with backpressure** | Real production systems need this; most toy projects skip it | ⭐⭐ |
| **Multi-model routing with load balancing** | Mirrors what Anyscale/Together do; great talking point | ⭐⭐ |
| **Request prioritization queue** (paid vs free tier) | Shows systems design thinking | ⭐⭐⭐ |
| **Quantized model support** (GPTQ, AWQ, GGUF) | Infra cost reduction; every company cares about this | ⭐⭐ |
| **KEDA autoscaling on queue depth** | More realistic than CPU-based HPA for LLM workloads | ⭐⭐⭐ |
| **Benchmark dashboard** (TTFT, TPS comparisons across models) | Demonstrates MLOps metric awareness | ⭐⭐ |
| **Model warm-up & eviction policy** | Real infra challenge; great to explain in interviews | ⭐⭐⭐ |
| **OpenTelemetry tracing** (trace one request end-to-end) | Shows observability maturity | ⭐⭐ |

### 📝 Things to Write Up
- An architecture decision record (ADR) explaining why you chose vLLM over TGI
- A benchmarking writeup: throughput vs latency tradeoffs at different batch sizes
- A blog post or LinkedIn post with your Grafana screenshots

---

## 🎯 Interview Alignment

| Role | What They'll Ask | Where in This Project |
|---|---|---|
| **AI Infra Intern** | How do you serve LLMs efficiently? What is KV cache? | Phase 2, Phase 3 |
| **MLOps Intern** | How do you monitor model performance? How do you deploy updates? | Phase 5, Phase 6 |
| **AI Eng Intern** | How would you scale this? What bottlenecks did you hit? | Phase 3, Phase 5 |

**Prepare to answer**:
- "Walk me through your architecture"
- "What was the hardest thing to build?"
- "How did you handle concurrent requests?"
- "What metrics did you track and why?"
- "What would you change if you had GPU budget?"

---

## 📅 Suggested Timeline

| Week | Focus |
|---|---|
| 1 | Phase 1 + Phase 2 (working API + LLM) |
| 2 | Phase 3 (queue + concurrency) |
| 3 | Phase 4 (Docker + Compose) |
| 4 | Phase 5 (Kubernetes + HPA) |
| 5 | Phase 6 (Prometheus + Grafana) |
| 6 | Phase 7 + polish + write-up + deploy |

---

## 📚 Essential Background Reading

- [**Efficient Memory Management for Large Language Model Serving with PagedAttention**](https://arxiv.org/abs/2309.06180) — the vLLM paper
- [**How continuous batching enables 23x throughput**](https://www.anyscale.com/blog/continuous-batching-llm-inference) — Anyscale blog
- [**LLM Inference Performance Engineering**](https://www.databricks.com/blog/llm-inference-performance-engineering-best-practices) — Databricks
- [**Serving ML Models at Scale**](https://huyenchip.com/2020/12/27/real-time-machine-learning.html) — Chip Huyen's blog

---

> **Tip**: Host this publicly on GitHub with a great README, a live demo link, and a Grafana screenshot. Recruiters and hiring managers at infra-focused teams look at GitHub before the interview. Make it easy to understand in 60 seconds.
