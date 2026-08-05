
FAQs : https://claude.ai/share/94066a68-dcec-45b4-b00e-910c5b205eff

*what we are building*
A GitHub App that watches a repository. When someone opens a Pull Request, it automatically reads the new code changes, finds similar code from the repo's past that caused bugs, and posts a comment on the PR explaining the risk.
### Concrete Example
Imagine a developer opens a PR with this code:
```python
user_input = request.args.get('id')
query = f"SELECT * FROM users WHERE id = {user_input}"
db.execute(query)
```
Your system:
1. Receives the PR via webhook
2. Embeds this code chunk into a vector
3. Searches your ChromaDB for similar historical code patterns that were later fixed
4. Finds PR #47 from 6 months ago — identical pattern, caused a SQL injection bug, fixed in PR #52
5. Posts this comment on the PR automatically:
> **⚠️ Semantic similarity to past bugfix detected**
> 
> This code pattern is semantically similar to code in PR #47 that caused a SQL injection vulnerability, fixed in PR #52. The risk is unparameterized user input directly in a query string. Consider using parameterized queries instead.

The developer sees this before merging. That's real value.

PIECE 1: The Ingestion System (you run this once)
GitHub Repo History → fetch all bugfix PRs → chunk diffs → embed with CodeBERT → store in ChromaDB

PIECE 2: The Webhook Listener (runs automatically on every new PR)
New PR opened → GitHub sends event to your FastAPI → extract diff → embed → query ChromaDB → find similar bugfixes

PIECE 3: The Review Generator (runs after retrieval)
Similar historical patterns → prompt LLM → generate human-readable risk explanation → post as PR comment via GitHub API

NEW PR comes in (unknown if it has bugs)
        ↓
Find similar code from PAST that was later FIXED
        ↓
Warn: "this pattern looks like something that caused a bug before"

OFFLINE (you do this once before anything)
─────────────────────────────────────────
All repo PRs
      ↓
Filter: which ones are bugfixes?
(title has "fix"/"bug"/"patch" OR has "bug" label)
      ↓
Extract the DIFF from those fix PRs
(this diff shows exactly what buggy code looked like)
      ↓
Embed those diffs → store in ChromaDB
with metadata: {pr_number, pr_title, pr_url}

ONLINE (happens automatically on every new PR)
──────────────────────────────────────────────
New PR opens → webhook fires
      ↓
Extract new PR's diff
      ↓
Embed it → query ChromaDB
      ↓
Returns: top 3 most similar historical bugfix diffs
      ↓
LLM generates explanation
      ↓
Post comment on new PR

### Why You Embed the Fix PR's Diff, Not the Original Buggy Code

This is subtle but important — and a great interview answer.

Fix PRs show you **both sides** of a diff:

diff

```diff
- user_input = request.args.get('id')              ← the buggy line (removed)
- query = f"SELECT * FROM users WHERE id = {user_input}"
+ query = "SELECT * FROM users WHERE id = ?"       ← the fix (added)
+ db.execute(query, (user_input,))
```

You embed the **removed lines** (the `-` lines) — because that's the buggy pattern you want to match against new code. The `+` lines tell you what the fix was, which you include in your comment to the developer.

So ChromaDB stores:

- **Vector of:** the buggy code pattern (removed lines)
- **Metadata of:** the fix PR title, URL, and what the fix looked like

---

### What the Developer Actually Sees

```
⚠️ Similar to a past bug (PR #52 — "fix SQL injection in user endpoint")

Your code:
  query = f"SELECT * FROM users WHERE id = {user_input}"

Similar past buggy pattern (from 2 months ago):
  query = f"SELECT * FROM orders WHERE id = {user_id}"

How it was fixed:
  query = "SELECT * FROM orders WHERE id = ?"
  db.execute(query, (user_id,))

Similarity score: 87%
```

Clean, useful, self-contained. The developer gets everything they need in one comment. No clicking around, no tracing PR history manually.

---

### One Sentence Summary of the Whole System

> You pre-process a repo's bugfix history offline, embed the buggy code patterns, and at PR time you check if new code looks like any of those patterns — if yes, you show the developer what happened last time and how it was fixed.

That's the whole thing. Everything you build is in service of that sentence.

Quick checkpoint — you should now be able to explain this to someone in 60 seconds without notes:

1. **What problem it solves** — developers repeat past mistakes because repo history is invisible
2. **How the offline part works** — ingest bugfix PRs, embed the buggy patterns, store in ChromaDB
3. **How the online part works** — new PR fires webhook, embed new diff, find similar past bugs, post comment
4. **What makes it different** — it's not rule-based like SonarQube, it learns from THIS repo's own history


Great question — and the answer to that is actually one of the most important engineering decisions in your whole project. Let me answer it first, then draw.

---

## Chunking — You Compare Parts, Not the Whole PR

You **never** embed an entire PR diff as one vector. Here's why:

A long PR might touch 10 different files — a database query, a UI component, an auth function, a config file. If you embed all of that as one vector, it becomes meaningless soup. The embedding tries to represent everything and ends up representing nothing well.

You split the diff into **hunks** and embed each hunk separately.

A hunk is one contiguous block of changes. Git already gives you this for free — every `@@` marker in a diff is a new hunk:

```diff
@@ -23,6 +23,7 @@ def get_user(user_id):    ← hunk 1 starts here
- query = f"SELECT * FROM users WHERE id = {user_id}"
+ query = "SELECT * FROM users WHERE id = ?"

@@ -45,4 +45,6 @@ def update_profile(data):  ← hunk 2 starts here
- if data['age'] > 0:
+ if isinstance(data['age'], int) and data['age'] > 0:
```

So a PR with 10 hunks becomes 10 separate vectors in ChromaDB. A new PR with 5 hunks becomes 5 queries. You compare **hunk vs hunk**, not **PR vs PR**.

This is your chunking strategy — and you should be able to defend it in interviews:

- Why not file-level? Too coarse. One file can have both safe and buggy code.
- Why not line-level? Too granular. One line has no context.
- Why hunk-level? It's the natural unit of a code change. It has context (surrounding lines) but stays focused on one logical modification.

Now the architecture:Every box is clickable — tap any one and it'll ask me to go deeper on that specific piece.

---

To directly answer your question: **you compare hunk vs hunk, never full PR vs full PR.** A long PR with 15 hunks fires 15 separate queries against ChromaDB. Each hunk is independently compared against every stored bugfix pattern. You might get 0 matches on 13 hunks and 2 matches on 2 hunks — and you only comment on those 2. That's the right behavior.

This is also one of your strongest interview answers. When they ask "how do you handle long PRs?" you say: _"I chunk by git hunk boundaries — the @@ markers — so each logical change is evaluated independently. A 500-line PR might have 15 hunks, each queried separately. I only surface a warning when a specific hunk crosses a similarity threshold, not the whole PR."_ That answer shows you actually thought about the problem.

Great edge case to think about. This is exactly the kind of thing interviewers ask.

---

## The Problem

One line has almost no context. If someone changes:

```python
- timeout = 30
+ timeout = 60
```

A CodeBERT embedding of that single line is nearly meaningless. It could match any line that changes a number. Your retrieval will either return garbage or nothing useful.

---

## What You Do — Context Window

You don't embed just the changed line. You embed the changed line **plus surrounding lines for context.**

Git diffs already give you this for free. Look at what a real hunk looks like even for a one-line change:

```diff
@@ -45,7 +45,7 @@ def authenticate_user(token, db):
     if not token:
         raise ValueError("Token required")
     
-    user = db.query(f"SELECT * FROM users WHERE token = '{token}'")
+    user = db.query(f"SELECT * FROM users WHERE token = '{token_clean}'")
     
     return user
```

The `@@` hunk includes 3 lines before and after the change by default. That context — `authenticate_user`, `db.query`, `token` — is what makes the embedding meaningful. Now CodeBERT understands this is a database query inside an auth function, not just a string change.

So your rule is simple: **always embed the full hunk, never just the changed lines.**

---

## What If the Hunk Context Is Still Too Small?

Sometimes even the full hunk is only 4-5 lines. You add a fallback — expand the context manually by fetching extra lines above and below from the file using the GitHub API.

```python
def get_hunk_with_context(file_content, hunk_start, hunk_end, extra=5):
    start = max(0, hunk_start - extra)
    end = min(len(file_content), hunk_end + extra)
    return file_content[start:end]
```

This is another strong interview answer. When they ask "what if the hunk is too small?" you say: _"I fetch additional surrounding lines from the file via GitHub API to pad the context window before embedding. I tested with extra=3 and extra=5 and found 5 gave better retrieval without introducing too much noise."_

Notice the last part — _"I tested and found"_. That's what separates someone who built it from someone who read about it.

---

## One More Edge Case to Think About

What if the one-line change is in a **config file** like `.env` or `settings.json`? Those have almost no semantic meaning for code review.

Simple fix: filter by file extension before embedding. Skip `.json`, `.yaml`, `.env`, `.md` files entirely. Only embed `.py`, `.js`, `.ts`, `.java`, `.go` etc.

That's another decision you made deliberately — and can explain.

----
-----
-----

## Week-by-Week Resources

### Week 1 — Foundations

**Embeddings & Semantic Search**

- Jay Alammar — "The Illustrated Word2Vec": https://jalammar.github.io/illustrated-word2vec/
- Jay Alammar — "The Illustrated BERT": https://jalammar.github.io/illustrated-bert/
- Vicki Boykis — "What are embeddings" (PDF, free): https://vickiboykis.com/what_are_embeddings/
- Hands-on: ChromaDB quickstart docs — https://docs.trychroma.com/getting-started

**GitHub Webhooks**

- GitHub Webhooks official docs: https://docs.github.com/en/webhooks/using-webhooks/creating-webhooks
- ngrok quickstart (expose localhost): https://ngrok.com/docs/getting-started

**FastAPI**

- Official tutorial (do Part 1 only — don't go deep yet): https://fastapi.tiangolo.com/tutorial/

---

### Week 2 — Ingestion Pipeline

**GitHub API**

- PyGithub library (easier than raw API): https://pygithub.readthedocs.io/en/latest/introduction.html
- GitHub REST API — List PR docs: https://docs.github.com/en/rest/pulls/pulls#list-pull-requests

**Understanding Git Diffs**

- "How to read a diff" — just this short explainer: https://www.git-tower.com/learn/git/ebook/en/command-line/advanced-topics/diffs

**CodeBERT + Sentence Transformers**

- Sentence Transformers library docs: https://www.sbert.net/docs/quickstart.html
- CodeBERT paper (read abstract + Section 3 only, skip math): https://arxiv.org/abs/2002.08155
- Specifically use this model: `microsoft/codebert-base` via sentence-transformers

**ChromaDB with metadata filtering**

- ChromaDB usage guide (read the "where" filter section): https://docs.trychroma.com/usage-guide

---

### Week 3 — Webhook + Review Engine

**Async FastAPI (you'll need this for webhook processing)**

- "Async in FastAPI" — official docs section: https://fastapi.tiangolo.com/async/
- Background tasks in FastAPI: https://fastapi.tiangolo.com/tutorial/background-tasks/

**Prompt Engineering for code review**

- OpenAI prompt engineering guide (read "Tactics" section): https://platform.openai.com/docs/guides/prompt-engineering
- Ollama local setup (free alternative): https://ollama.com/download — run `ollama pull mistral` and use it exactly like OpenAI API

**Posting GitHub PR Reviews via API**

- GitHub API — Create review: https://docs.github.com/en/rest/pulls/reviews#create-a-review-for-a-pull-request

---

### Week 4 — Running on Real Repos

**Handling GitHub Rate Limits**

- GitHub rate limit docs: https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api
- Use `time.sleep` + check `X-RateLimit-Remaining` header — no library needed

**Evaluating Retrieval Quality**

- Read just Section 4 of this (precision/recall for RAG): https://arxiv.org/abs/2312.10997
- This gives you vocabulary to talk about your evaluation in interviews

---

### Week 5 — Docker + Deployment

**Docker Compose**

- Official "Get Started" Part 8 (multi-container): https://docs.docker.com/get-started/workshop/08_using_compose/
- A good practical example with FastAPI + PostgreSQL + another service: https://testdriven.io/blog/fastapi-docker-traefik/ (read the Docker Compose part only)

**Railway deployment (free)**

- Railway quickstart: https://docs.railway.app/quick-start
- Deploying FastAPI specifically: https://docs.railway.app/guides/fastapi

**Writing a good README**

- Best README template (just use this): https://github.com/othneildrew/Best-README-Template

**Architecture diagrams**

- Use Excalidraw (free, browser-based, looks clean): https://excalidraw.com

---

### Week 6 — Interview Prep

**System Design basics (so you can answer "what breaks at scale?")**

- Read Chapter 1 of "System Design Interview" by Alex Xu — PDFs float around online, or borrow from your institute library. Just Chapter 1 is enough for intern level.

**ML System Design (for Warner Bros AI/ML interviews)**

- Chip Huyen's ML interviews book — free chapter: https://huyenchip.com/ml-interviews-book/
- Read the "ML in Production" section specifically

**Mock interview practice**

- Pramp (free peer mock interviews): https://www.pramp.com
- Do at least 2 mock interviews in Week 6 where you explain only this project

---

## The One Thing Most People Miss

Save every mistake you make while building. Every rate limit hit, every retrieval failure, every wrong chunk boundary — write them in a `DEVLOG.md` in your repo. Two reasons:

One, interviewers always ask "what was the hardest problem you faced?" and "I hit GitHub rate limits and implemented exponential backoff with jitter" is a far better answer than "it was pretty smooth."

Two, it proves you actually built it. Anyone can clone a tutorial. A devlog of real failures is yours alone.

Want me to also give you a starter project structure — the folders, files, and skeleton code to set up on Day 1?