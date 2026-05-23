# Animatronio

**A self-maintaining personal memory system for AI agents.**

> Most AI memory systems are archives. Animatronio is a living model.

---

## The problem

Every AI agent you talk to starts from zero. Even agents that persist memory across sessions do so by appending facts to a store — they have no mechanism for detecting contradictions, resolving conflicts, or knowing what they *don't* know. The result: stale facts accumulate, conflicts go unresolved, and every new agent or system you onboard has to rediscover who you are from scratch.

Three problems the field acknowledges but hasn't solved:
1. **Cross-session identity** — your context doesn't travel between agents or systems
2. **Memory staleness** — old facts linger without expiry or conflict resolution
3. **Temporal abstraction** — agents can't reason about how facts have changed over time

Animatronio attacks all three.

---

## What it is

Animatronio is a user-owned, agent-agnostic personal knowledge base with a standard API. Any agent or system can read from and write to it. It maintains coherence automatically and actively tracks its own gaps.

It is not another vector database wrapper. The architecture has four distinct layers that don't exist together anywhere else.

---

## Architecture

### 1. Knowledge Store

The core memory layer. Stores facts, preferences, relationships, and experiences about the user. Uses a hybrid retrieval approach:

- **Vector embeddings** — semantic similarity search for fuzzy recall
- **Cosine similarity ranking** — relevance scoring across candidates
- **LLM reranking** — final pass to surface contextually appropriate results, not just semantically close ones

Facts are typed and timestamped. The store knows when something was learned and can reason about recency.

### 2. Coherence Engine

The part most memory systems skip entirely.

When new information arrives, the coherence engine checks it against existing facts for conflicts. Three resolution paths:

- **Auto-resolve** — when one fact is clearly newer or more specific, update silently
- **Flag for review** — when the conflict is ambiguous, add to the Curiosity Ledger (see below)
- **Temporal versioning** — when a fact changed over time, preserve the history rather than overwriting ("used to prefer X, now prefers Y")

The goal is a knowledge base that stays internally consistent without requiring manual curation.

### 3. Curiosity Ledger

The novel layer. Most memory systems track what they know. The Curiosity Ledger tracks what the system *doesn't* know — and what it wants to find out.

Each entry in the Curiosity Ledger is:
- A knowledge gap (something the system is uncertain about or lacks data on)
- A priority score (how much filling this gap would improve recall quality)
- A suggested question or prompt to surface it naturally in conversation
- An optional trigger condition (surface this when topic X comes up, or when it's been N days)

Agents consuming Animatronio can query the Curiosity Ledger and surface questions at natural moments in conversation — not as a survey, but as genuine curiosity. Over time the knowledge base gets sharper without the user having to think about it.

The Curiosity Ledger is introspective: it knows its own gaps rather than pretending to be complete.

### 4. Reflection Engine

Background process that runs periodically (configurable cadence). Three phases, inspired by cognitive sleep cycles:

- **Light phase** — ingest recent signals, deduplicate, stage candidates
- **Deep phase** — score and promote durable facts into long-term storage, resolve pending conflicts
- **REM phase** — extract themes, update the Curiosity Ledger based on patterns, identify what's becoming stale

The reflection engine is what turns Animatronio from a passive store into something that actively maintains itself.

---

## API

Animatronio exposes a standard REST API so any agent or system can use it without tight coupling.

### Core endpoints (planned)

```
GET  /recall?q=<query>         — Hybrid search, returns ranked facts
GET  /recall/:id               — Fetch specific fact by ID
POST /ingest                   — Add new fact or experience
POST /ingest/batch             — Batch ingest

GET  /curiosity                — Get open Curiosity Ledger entries
POST /curiosity/resolve/:id    — Mark a gap as filled
POST /curiosity/dismiss/:id    — Dismiss a gap as not relevant

GET  /conflicts                — List unresolved conflicts
POST /conflicts/resolve/:id    — Resolve a conflict manually

GET  /health                   — System health + reflection status
POST /reflect                  — Trigger a reflection sweep manually
```

### Authentication

User-owned and self-hosted. Auth via API key. No vendor lock-in, no cloud dependency.

---

## Cold start solution

The cold start problem: every new agent has to rediscover who the user is. With Animatronio, any new agent points at the API on day one and gets immediate, coherent context — preferences, relationships, history, ongoing threads — without the user having to re-explain anything.

This is the portability layer that the current agent ecosystem is missing.

---

## What Animatronio is not

- **Not a replacement for agent-specific memory** — agents still maintain their own session context. Animatronio is the shared layer underneath.
- **Not a logging system** — it doesn't store raw conversation transcripts. It stores distilled, structured knowledge.
- **Not cloud-dependent** — designed to run locally or on your own infrastructure.

---

## Relationship to Crucible

[Crucible](https://github.com/vsharma8363/OpenClaw-Crucible) handles agent self-improvement — making the agent smarter about its own behavior over time.

Animatronio handles user knowledge portability — making sure any agent always knows who it's talking to.

They're complementary. Crucible improves the agent. Animatronio carries the user.

---

## Status

Early design phase. This README is the spec. Implementation in progress.

Contributions and feedback welcome.

---

## Related work

- [Mem0](https://mem0.ai) — managed cloud memory, good benchmarks, not user-owned
- [Zep / Graphiti](https://arxiv.org/abs/2501.13956) — temporal knowledge graph, strong on relational reasoning
- [Letta / MemGPT](https://github.com/cpacker/MemGPT) — paged context memory, good for long sessions
- [mnemos](https://github.com/Sohamp2809/mnemos) — typed conflict resolution for multi-agent systems

None of these combine user ownership, coherence enforcement, and the Curiosity Ledger as a first-class primitive.
