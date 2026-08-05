Here are the three paragraphs — written so you can refine them into your own words, and eventually directly into resume bullets or interview answers.

---

**Project 1: Semantic Code Review Assistant**

Large codebases accumulate recurring bug patterns that senior reviewers catch through experience, but this knowledge is never systematically captured — new PRs repeat old mistakes because there's no memory across reviews. My solution ingests a repository's historical bugfix PRs, embeds code hunks using CodeBERT, and builds a searchable index of known problematic patterns. When a new PR is opened, the system compares incoming code hunks against this index and automatically posts review comments flagging similar historical bugs with context on what went wrong before. The concrete metric: **pattern recall rate** — what percentage of known bug-pattern reintroductions in a held-out test set of PRs does the system correctly flag, measured against manual labeling.

---

**Project 2: LLM Inference Request Scheduler**

Serving LLMs to multiple concurrent users is inefficient by default — naive FCFS scheduling causes short requests to wait behind long ones, wasting throughput and increasing average latency across all users. My solution implements a scheduling layer in front of a simulated inference backend, comparing multiple policies: FCFS, shortest-job-first, and priority-aware batching using real request length distributions from the ShareGPT dataset. The system exposes a FastAPI interface, logs per-request latency and queue wait time, and produces a comparative analysis across policies. The concrete metric: **throughput improvement of best policy over FCFS baseline** — measured in requests/second at equivalent average latency, targeting a demonstrable 30-40% gain.

---

**Project 3: Streaming ML Feature Pipeline**

ML models serving real-time applications degrade when fed stale features — a recommendation model using batch-computed features from hours ago misses recent user behavior that would meaningfully change its output. My solution builds a streaming feature pipeline simulating a video platform (directly analogous to Warner Bros. Max) where user events (views, ratings, searches) flow through Kafka, features are computed in real-time (rolling watch time, content affinity scores, session signals), and served from Redis to a FastAPI feature-serving endpoint that a downstream model can query at inference time. The concrete metric: **feature freshness** — median age of features served at inference time under streaming vs. batch pipeline, targeting sub-30-second freshness vs. hours for batch.

---

These three paragraphs together are your north star while building. Any time you feel scope creep pulling you toward something fancy, come back to these and ask — does this addition improve my stated metric? If no, cut it.

Now do you want the consolidated week-by-week timeline for all three?