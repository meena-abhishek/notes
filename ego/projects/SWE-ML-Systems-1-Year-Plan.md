# One-Year Plan: Software Engineer, ML Systems (Google / Microsoft / Amazon)

**Starting point assumed:** basic programming only, no prior ML/systems work.
**Target:** SWE roles on ML-systems-adjacent teams — ML infra, model serving, feature platforms, training pipelines.

This is a from-scratch plan. Total time budget: ~12 months, roughly 25-30 hrs/week alongside coursework.

---

## How the year is split

| Phase | Months | Focus |
|---|---|---|
| 1 | 1–2 | Programming depth + CS fundamentals refresh |
| 2 | 3–5 | ML fundamentals + PyTorch, hands-on |
| 3 | 6–8 | Systems + one real ML infra project |
| 4 | 9–10 | Second project + MLOps/serving depth |
| 5 | 11–12 | Interview prep: DSA, system design, ML system design, applications |

Two threads run in parallel from month 1 onward and never stop: **DSA practice** and **building something**. Everything else slots around these.

---

## Phase 1 (Months 1–2): Foundations

**Goal:** get comfortable enough in Python + CS fundamentals that ML material doesn't fight you on syntax.

- **Python for the level you need:** if "basic programming" means another language, spend 1–2 weeks on Python specifically — [Python Crash Course](https://ehmatthes.github.io/pcc/) or the official [Python tutorial](https://docs.python.org/3/tutorial/). Don't linger — you learn Python fastest by using it in Phase 2.
- **CS fundamentals refresh:** Harvard's [CS50x](https://cs50.harvard.edu/x/) — free, and it directly covers memory, data structures, and algorithms you'll need in interviews. If you already have solid DSA (arrays, trees, hashmaps, recursion), skip this and go straight to LeetCode Easy problems instead.
- **Start DSA now, don't wait:** begin daily practice on [LeetCode](https://leetcode.com/) — Easy problems in month 1, Medium by month 2. Use the [NeetCode 150](https://neetcode.io/practice) list as your syllabus; NeetCode's YouTube explanations are the fastest way to learn patterns (two pointers, sliding window, BFS/DFS, DP) from zero.
- **Git/GitHub basics** if not already comfortable — you'll be shipping projects publicly all year: [GitHub's own guide](https://docs.github.com/en/get-started).

**Exit criteria for Phase 1:** you can write Python comfortably, solve LeetCode Easy problems without major struggle, and know what a hash map, tree, and graph traversal are.

---

## Phase 2 (Months 3–5): ML Fundamentals

**Goal:** understand how models actually work, not just how to call `.fit()`.

- **Core course:** [Andrew Ng's Machine Learning Specialization](https://www.coursera.org/specializations/machine-learning-introduction) (Coursera, audit for free) — linear/logistic regression, neural nets, from first principles.
- **Then go deep and practical:** [Practical Deep Learning for Coders](https://course.fast.ai/) (fast.ai, free) — top-down, code-first, uses PyTorch. This is the single best free resource for someone who wants to *build* rather than sit through theory. Pair it with the free book at [fastai book](https://fastai.github.io/fastbook2e/).
- **PyTorch itself:** [PyTorch official 60-minute blitz](https://docs.pytorch.org/tutorials/beginner/deep_learning_60min_blitz.html), then build 2–3 small end-to-end projects (image classifier, sentiment classifier) so PyTorch becomes muscle memory, not tutorial-following.
- **Neural nets from first principles (do this — it pays off hugely in interviews):** [Andrej Karpathy's "Neural Networks: Zero to Hero"](https://karpathy.ai/zero-to-hero.html) — builds backprop and a GPT from scratch in code. This is the highest-signal free resource for understanding what's actually happening inside a model, which is exactly what ML systems interviews probe.
- **Keep LeetCode going:** Medium-level, aim for consistency (contest-style, timed) over volume.

**Exit criteria:** you can explain gradient descent, backprop, overfitting, and train a basic CNN or transformer from scratch without copy-pasting.

---

## Phase 3 (Months 6–8): Systems + First Project

This is where "ML" and "systems" merge — the actual content of the role you're targeting.

- **Distributed systems fundamentals:** [Designing Data-Intensive Applications](https://dataintensive.net/) (Kleppmann) — the standard reference for how real data/ML infra is built. Read chapters on replication, partitioning, and stream processing first; those map directly to ML infra work.
- **Systems papers that ML infra interviewers reference:** [Google's Bigtable paper](https://research.google/pubs/pub27898/), [Google's Borg paper](https://research.google/pubs/pub43438/), [Spanner](https://research.google/pubs/pub39966/). Skim for architecture and tradeoffs, not memorization.
- **LLM serving specifics (high-leverage for 2026 hiring):** [vLLM / PagedAttention paper](https://arxiv.org/abs/2309.06180) — read this properly, it's the most commonly referenced paper in ML infra interviews right now.
- **Build project 1 — a code review / retrieval tool, or similar:** something with a real ingestion pipeline, vector store, and API layer (e.g., CodeBERT-style embeddings + a vector DB like ChromaDB + FastAPI + async workers via Celery/Redis + Postgres for metadata + Prometheus for observability). The point isn't the specific stack — it's demonstrating you can design and ship a real system, not a notebook.

**Exit criteria:** one deployed/deployable project with a clear architecture doc (a running "devlog" of decisions you made and why — this becomes your interview story).

---

## Phase 4 (Months 9–10): Second Project + MLOps

- **MLOps depth:** [MLOps Zoomcamp](https://github.com/DataTalksClub/mlops-zoomcamp) (DataTalksClub, free) — covers model serving, monitoring, and pipelines with hands-on labs. This maps directly onto SWE-for-ML-systems interview questions.
- **Model serving tools:** get hands-on with [Triton Inference Server](https://github.com/triton-inference-server/server) or [TorchServe](https://pytorch.org/serve/) — build a minimal serving setup and load-test it.
- **Streaming basics:** [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide-v2/) (free from Confluent) — enough to build a basic streaming feature pipeline.
- **Build project 2 — an LLM inference scheduler or streaming feature pipeline:** something that touches batching, latency/throughput tradeoffs, or real-time feature computation. This is your strongest differentiator if you get it right — pair it with what you learned from the vLLM paper.

**Exit criteria:** two shipped projects, both with clear write-ups, both defensible in a 45-minute deep-dive interview.

---

## Phase 5 (Months 11–12): Interview Prep

- **DSA:** by now you should be at LeetCode Medium/Hard consistently. Push toward contest rating via [LeetCode Contests](https://leetcode.com/contest/) weekly.
- **System design (general):** [Alex Xu's System Design Interview Vol 1 & 2](https://bytebytego.com/) — standard prep material, also has an ML-systems-specific track.
- **ML system design specifically:** [Chip Huyen's "Designing Machine Learning Systems"](https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/) — the best single resource for ML-specific system design interviews (feature stores, training pipelines, serving at scale).
- **Mock interviews:** [Pramp](https://www.pramp.com/) (free peer mocks) or paid platforms if budget allows.
- **Resume + applications:** target SWE roles with "ML," "AI Platform," "Applied," or "ML Infra" in the title at Google, Microsoft, Amazon. Start applying from month 9 onward — big-company pipelines are slow, don't wait until everything is "ready."

---

## Running threads (all year, don't skip)

- **DSA:** daily, escalating difficulty, contest practice from month 6 onward.
- **Build in public:** keep a devlog for every project — this is what interviewers actually want to hear about, more than the tech stack itself.
- **Read one paper a week** starting month 4 — arXiv, company engineering blogs (Google AI blog, Meta engineering, Uber engineering) — builds the vocabulary you need to sound credible fast.

---

## Reality check

A year gets you to a strong **entry-level SWE / ML-systems** candidate — not a research scientist, not a senior infra engineer. The realistic path into Google/Microsoft/Amazon at this stage is via general SWE or MLE hiring pipelines, with ML-systems experience as your specialization and differentiator, not a separate job ladder you're hired directly into. Once inside, moving toward ML platform teams is a well-trodden internal path.
