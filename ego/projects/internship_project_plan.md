# Semantic Code Review Assistant — Internship Project Plan
A GitHub App that watches a repository. When someone opens a Pull Request, it automatically reads the new code changes, finds similar code from the repo's past that caused bugs, and posts a comment on the PR explaining the risk.

**One sentence:** You pre-process a repo's bugfix history offline, embed the buggy code patterns, and at PR time you check if new code looks like any of those patterns — if yes, you show the developer what happened last time and how it was fixed.

---

## Why This Is a Strong Project

- Maps directly to all 8 roles in your eligibility list (PEGA AI/ML, Warner Bros AI/ML, Warner Bros Data Platform, Carousell Backend, Vivriti DevSecOps)
- Covers your electives: NLP with LLMs, Distributed Systems, Cloud Computing, ML
- Has a live demo moment: push a PR → comment appears automatically in under 30 seconds
- Genuine engineering decisions you can defend, not a tutorial project
- "Institutional memory for codebases" — a real problem no existing tool solves for repo-specific history

---
## The Architecture (Two Phases)

### Phase 1 — Offline Ingestion (run once)

```
GitHub repo → fetch all closed PRs → filter bugfix PRs (title has "fix"/"bug"/"patch")
→ extract diff (removed lines = buggy pattern)
→ split by @@ hunk markers
→ embed each hunk with CodeBERT (sentence-transformers)
→ store vector in ChromaDB + metadata in PostgreSQL
```

### Phase 2 — Online (fires on every new PR)

```
New PR opens → GitHub sends webhook → FastAPI receives it
→ extract diff → split into hunks → embed each hunk
→ query ChromaDB → retrieve top 3 similar historical bugfix hunks
→ LLM generates review comment (Mistral free / OpenAI)
→ POST comment to PR via GitHub API
```

### Infrastructure

- Docker Compose: 3 services (FastAPI, PostgreSQL, ChromaDB)
- Deployed on Railway (free tier) — live webhook URL, no localhost needed
- GitHub repo with README, DEVLOG.md, architecture diagram

---

## Key Design Decisions (memorise these for interviews)

| Decision          | What you chose                           | Why                                                                                           |
| ----------------- | ---------------------------------------- | --------------------------------------------------------------------------------------------- |
| Chunking level    | Hunk-level (@@ markers)                  | File = too coarse. Line = too fine. Hunk = one logical change with surrounding context.       |
| What to embed     | Removed lines from fix PRs               | The `-` lines in a bugfix diff ARE the buggy pattern. The `+` lines show how it was fixed.    |
| Embeddings model  | CodeBERT via sentence-transformers       | Code semantics differ from natural language. CodeBERT was pre-trained on code+docs pairs.     |
| Vector store      | ChromaDB                                 | Free, runs locally, supports metadata filtering. Pinecone costs money. FAISS has no metadata. |
| LLM               | Mistral via Ollama (dev) / OpenAI (demo) | Ollama is completely free for development. Switch to OpenAI only for the final demo.          |
| Why not SonarQube | SonarQube uses generic rules             | Your system learns from THIS repo's own history — catches project-specific patterns.          |
| One-line hunks    | Expand context                           | Fetch 5 extra lines above and below from file via GitHub API before embedding.                |
| File filtering    | Skip config files                        | Skip .json, .yaml, .env, .md — only embed .py, .js, .ts, .java, .go etc.                      |

---

## Week-by-Week Build Plan

### Week 1 — Foundations (don't touch the project yet)

**Goal:** By end of week, you can embed a piece of code and retrieve similar code from a toy dataset.

**Daily split:** 1.5 hr learn + 2 hr hands-on

**Tasks:**
- Read Jay Alammar's Illustrated BERT: https://jalammar.github.io/illustrated-bert/
- Install: Python 3.11, Docker, Cursor (cursor.com), create test GitHub repo
- Run ChromaDB locally, embed 20 random sentences, query them — see what comes back
- Load `microsoft/codebert-base` via sentence-transformers, embed 5 code snippets
- Write a tiny script: given a code string, find the most similar from a list of 10 — print cosine similarity scores
- Set up DEVLOG.md — write your first entry tonight

**End state:** A Python script that takes a code snippet, embeds it with CodeBERT, queries ChromaDB, returns top 3 similar snippets with scores.

**Resources:**
- Jay Alammar Illustrated Word2Vec: https://jalammar.github.io/illustrated-word2vec/
- Jay Alammar Illustrated BERT: https://jalammar.github.io/illustrated-bert/
- ChromaDB quickstart: https://docs.trychroma.com/getting-started
- Sentence Transformers quickstart: https://www.sbert.net/docs/quickstart.html

---

### Week 2 — Ingestion Pipeline

**Goal:** ChromaDB is populated with real bugfix patterns from a real GitHub repo. The knowledge base exists.

**Daily split:** 0.5 hr learn + 3 hr build

**Tasks:**
- Install PyGithub, get a GitHub personal access token
- Write `fetcher.py`: fetch all closed PRs from `psf/requests` (small, well-labelled repo)
- Filter PRs: keep only those with "fix", "bug", "patch" in title or labels
- Write `chunker.py`: parse raw diff strings, split by `@@` markers, extract removed lines + 5 lines context
- Handle rate limits: check `X-RateLimit-Remaining` header, sleep when needed — write in DEVLOG
- Write `embedder.py`: embed each hunk, store in ChromaDB with metadata (pr_number, pr_title, pr_url, fix_preview)
- Store raw PR data in PostgreSQL using psycopg2
- Run end to end — verify ChromaDB has 100+ entries

**End state:** Run `python ingest.py psf/requests` and ChromaDB fills with real bugfix hunks. Query manually and get relevant results.

**Resources:**
- PyGithub docs: https://pygithub.readthedocs.io/en/latest/introduction.html
- GitHub REST API — PRs: https://docs.github.com/en/rest/pulls/pulls#list-pull-requests
- Understanding git diffs: https://www.git-tower.com/learn/git/ebook/en/command-line/advanced-topics/diffs
- CodeBERT paper (read abstract + Section 3): https://arxiv.org/abs/2002.08155

---

### Week 3 — Webhook + Retrieval (hardest week)

**Goal:** Opening a PR triggers your FastAPI server, retrieves similar bugfixes, prints them to terminal. No LLM yet — just raw retrieval working end to end.

**Budget an extra day here. Expect things to break. Write every failure in DEVLOG.**

**Tasks:**
- Build `main.py` in FastAPI: one POST endpoint `/webhook` that receives GitHub events
- Install ngrok, expose localhost:8000, register as GitHub webhook on test repo
- Parse PR payload: extract diff URL, fetch actual diff, run through chunker
- For each hunk: query ChromaDB, collect matches above similarity threshold (start 0.75)
- Print matches to terminal with PR number and similarity score
- Move processing to FastAPI BackgroundTask so webhook responds in under 1 second

**End state:** Open a PR on test repo → terminal shows retrieved bugfix matches within 5 seconds.

**Resources:**
- GitHub Webhooks docs: https://docs.github.com/en/webhooks/using-webhooks/creating-webhooks
- ngrok quickstart: https://ngrok.com/docs/getting-started
- FastAPI async: https://fastapi.tiangolo.com/async/
- FastAPI background tasks: https://fastapi.tiangolo.com/tutorial/background-tasks/
- ChromaDB where filters: https://docs.trychroma.com/usage-guide

---

### Week 4 — LLM Review Generation + GitHub Comment

**Goal:** A real comment appears on a real PR automatically. First full end-to-end demo moment.

**Tasks:**
- Install Ollama locally, pull Mistral: `ollama pull mistral` — test with a basic prompt
- Write `reviewer.py`: takes retrieved hunks, formats prompt, calls Mistral, gets review text
- Prompt: new code + 3 similar past bugfixes + "explain the risk in 3 bullet points"
- Write `commenter.py`: POST to GitHub API `/repos/{owner}/{repo}/pulls/{n}/reviews`
- Wire everything in `main.py`: webhook → chunk → retrieve → review → comment
- Open a PR with obviously suspicious code — watch the comment appear
- Tune: if comment too verbose, tighten prompt. If retrieval misses, lower threshold.

**End state:** Push a PR → automatic comment appears on GitHub with risk explanation and reference to past bugfix PR.

**Resources:**
- OpenAI prompt engineering guide: https://platform.openai.com/docs/guides/prompt-engineering
- Ollama setup: https://ollama.com/download
- GitHub API — Create review: https://docs.github.com/en/rest/pulls/reviews#create-a-review-for-a-pull-request

---

### Week 5 — Run on Real Repo + Measure Precision

**Goal:** System runs on a real open source repo with 6 months of history. You have a precision number to cite in interviews.

**Tasks:**
- Re-run ingestion on larger repo: `fastapi/fastapi` or `django/django`
- Handle edge cases: empty diffs, binary files, config-only changes — add file extension filter
- Handle one-line hunks: expand context by fetching surrounding file lines via GitHub API
- Manually evaluate precision: take 20 retrieval results, check if genuinely relevant — write the number down
- Adjust similarity threshold based on findings — document before/after in DEVLOG
- Test edge cases: PR with 0 code changes, PR with 50 hunks, PR in non-Python file

**End state:** System ingested a real repo, retrieval precision measured and documented, edge cases handled.

**Your interview answer:** "I measured retrieval precision at X% on 20 manual test cases, then adjusted the similarity threshold from 0.75 to 0.70 and improved it to Y%."

**Resources:**
- GitHub rate limits: https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api
- RAG evaluation (read Section 4): https://arxiv.org/abs/2312.10997

---

### Week 6 — Docker, Deploy, README, Demo Video

**Goal:** Anyone can run your project with one command. It is live on the internet. You have a 3-minute demo video.

**Tasks:**
- Write `docker-compose.yml`: three services — FastAPI app, PostgreSQL, ChromaDB
- Test: `docker-compose up` from scratch on a clean folder — fix everything that breaks
- Deploy FastAPI to Railway (free): https://docs.railway.app/guides/fastapi
- Point GitHub webhook to live Railway URL, not ngrok
- Draw architecture diagram in Excalidraw (excalidraw.com) — save as PNG, add to repo
- Write README: problem → architecture → design decisions → how to run → example output
- Record Loom demo (3 min): show repo, push PR live, watch comment appear, explain what happened
- Add demo video link to README and resume

**End state:** Public GitHub repo, live deployed system, 3-min Loom demo, clean README with architecture diagram.

**Resources:**
- Docker Compose tutorial: https://docs.docker.com/get-started/workshop/08_using_compose/
- Railway quickstart: https://docs.railway.app/quick-start
- Best README template: https://github.com/othneildrew/Best-README-Template
- Excalidraw (free diagrams): https://excalidraw.com

---

### Week 7 — Interview Prep

**Goal:** You can talk about this project for 30 minutes without notes.

**Tasks:**
- Write answers to all 8 interview questions below — then say them out loud
- Do 2 mock interviews on Pramp (pramp.com) explaining only this project
- Apply to all 8 roles if not done already — check all deadlines
- Update resume: add project with 3 bullet points (what it does, stack, one measurable result)
- Fix anything you cannot explain — if you don't understand why a line exists, rewrite it

**End state:** You can answer "walk me through your project" cold, in under 2 minutes, clearly.

---

## The 8 Interview Questions to Prepare

1. Walk me through your architecture
2. Why CodeBERT over regular BERT or text-embedding-ada?
3. Why hunk-level chunking? Why not file-level or line-level?
4. Why not just use SonarQube or GitHub Copilot?
5. What was the hardest bug you hit while building this?
6. How did you measure retrieval quality?
7. What would break at 100x scale?
8. What would you do differently with 3 more months?

---

## Your Answer to "Tell Me About Your Project" (2 minutes, memorise this)

> "I built a semantic code review system that watches a GitHub repository for new pull requests. It works in two phases. First, offline, it ingests the repo's entire PR history, identifies bugfix PRs by their labels and titles, extracts the buggy code patterns from the diffs, and stores them as vector embeddings in ChromaDB using CodeBERT.
>
> Then online, whenever a new PR is opened, a GitHub webhook fires, my FastAPI server receives it, chunks the diff by hunk boundaries, embeds each hunk, and queries ChromaDB for semantically similar historical bugfix patterns. If it finds a match above a similarity threshold, it generates a review comment using Mistral explaining the risk and referencing the original bugfix PR — and posts that automatically to the PR.
>
> The hardest part was the chunking strategy. I tried file-level first, found retrieval was too coarse, moved to hunk-level which improved precision from around 40% to 68% on my manual test set. It's deployed on Railway with a Docker Compose setup — anyone can run it with one command."

---

## Project File Structure

```
semantic-code-reviewer/
├── docker-compose.yml
├── README.md
├── DEVLOG.md
├── architecture.png
├── app/
│   ├── main.py          # FastAPI app, webhook endpoint
│   ├── fetcher.py       # GitHub API, PR fetching
│   ├── chunker.py       # Diff parsing, hunk splitting
│   ├── embedder.py      # CodeBERT embeddings, ChromaDB storage
│   ├── retriever.py     # ChromaDB querying
│   ├── reviewer.py      # LLM prompt, review generation
│   └── commenter.py     # GitHub API, posting PR comment
├── scripts/
│   └── ingest.py        # Run once: ingest a repo's history
└── requirements.txt
```

---

## AI Tools to Use (All Free)

- Claude.ai — architecture discussions, debugging, understanding concepts
- Cursor (cursor.com) — AI in your editor, sees your whole codebase
- Perplexity.ai — fast answers from docs
- Ollama — run Mistral locally for free during development

**The golden rule:** Never copy-paste AI code without reading every line. If you can't explain it, you can't defend it in an interview.

---

## Daily Routine

**Night before (15 min):** Write exactly what function you're building tomorrow. Not "work on ingestion" — "write the GitHub PR fetcher that handles pagination and rate limits."

**Start of session (10 min):** Paste your plan into Claude. Ask: "Is there anything I'm missing about this approach?"

**During build:** Cursor for code generation. Run immediately. Break it on purpose. Fix it.

**End of session (15 min):** Write 3 lines in DEVLOG.md — what you built, what broke, what you learned.

---

## Realistic Expectations

This project significantly improves your chances. It does not guarantee an internship.

What it does: gets you past resume screening, gives you 20-30 minutes of solid interview material, compensates partially for 7.44 CPI by showing practical ability.

What it doesn't fix: hard CPI cutoffs applied before humans see your resume.

The actual win: you go from "probably screened out" to "has a real shot at interview calls." The project gets you in the room. Your ability to explain it clearly gets you the offer.

**Start applying now — don't wait until the project is done.**
