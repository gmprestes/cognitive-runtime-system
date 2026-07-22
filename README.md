# CRS — Cognitive Runtime System

**Orchestration with laws. Memory as a graph. Humans as reviewers of what the organization is allowed to remember.**

> CRS is not “yet another multi-agent framework.”  
> It is an **architecture proposal**: the work cycle and the memory cycle are a **program** (runtime); the LLM only **proposes**; commits to state and memory are **governed**.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## TL;DR

**CRS (Cognitive Runtime System)** is an architecture for multi-agent orchestration:

- **Runtime owns control** — phases, budgets, DAGs, stop conditions  
- **Models only propose** — they never silently mutate durable state  
- **Memory is a graph** — episodes, lessons, evidence, impact edges  
- **Humans review memory** — not every keystroke; the *beliefs* the org reuses  

Moat thesis: *commodity judgment models + proprietary orchestration + governed organizational memory.*

---

## Why CRS exists

The industry often confuses three different things:

| Thing | Common practice | CRS separation |
|---|---|---|
| **Judgment** | “The model is smart” | LLMs/agents **propose** |
| **Control** | Prompt + hope | Runtime with **laws and budgets** |
| **Memory** | RAG / chat log / vector dump | **Graph** with provenance and review |

Without that separation, an agent swarm:

- repeats expensive mistakes  
- “learns” rumors at high speed  
- has no owner for operational truth  

With CRS:

```text
goal → plan proposal → DAG commit → swarm executes
     → evidence → memory proposal → human/verifier review
     → MemoryGraph commit → next orchestration reuses with criteria
```

---

## Core concepts

### 1. Orchestration runtime

CRS coordinates agents (and humans) with:

- a **work DAG** (dependencies, parallelism, no cycles)  
- **phases** (`plan → spawn → observe → critique → replan | merge | stop`)  
- **budgets** (children, rounds, wall-clock, risk)  
- **authority** (who proposes vs who commits)

Read: [docs/02-orchestration-dag.md](docs/02-orchestration-dag.md)

### 2. Memory Graph

Organizational memory as a causal graph — not only vector similarity:

- nodes: Episode, Task, Agent, Evidence, Belief, Lesson, HumanDecision…  
- edges: `SUPPORTS`, `DERIVED_FROM`, `SUPERSEDES`, `HELPED` / `HURT` / `UNUSED`…

Read: [docs/03-memory-graph.md](docs/03-memory-graph.md)

### 3. Human Memory Review

Humans do not babysit every tool call.  
They **authorize what may be remembered and reapplied**.

Three memory planes:

| Plane | Content | Who writes |
|---|---|---|
| Hot | task state now | runtime + agents |
| Warm | candidate lessons/beliefs | agents propose |
| Cold | canonical org memory | **human** (or dual control) |

Read: [docs/04-human-memory-review.md](docs/04-human-memory-review.md)

### 4. Authority contract

```text
JudgmentModel  → MAY propose, MUST NOT mutate durable state
Runtime        → MAY validate/commit/reject, enforces laws
Host/World     → executes tools, returns observations
Human          → reviews memory & high-risk gates
```

Read: [docs/05-authority-contracts.md](docs/05-authority-contracts.md)

---

## Architecture (overview)

```text
┌──────────────────────────────────────────────────────┐
│  Human Review Plane                                  │
│  approve / amend / reject memory and risk gates      │
└───────────────────────────┬──────────────────────────┘
┌───────────────────────────▼──────────────────────────┐
│  CRS Orchestration Runtime                           │
│  phases · budgets · authority · stop conditions      │
└───────┬─────────────────────────────┬────────────────┘
        │ proposals                   │ commits
┌───────▼──────────┐         ┌────────▼────────┐
│ Judgment layer   │         │ Durable state   │
│ LLMs / agents    │         │ WorkGraph (DAG) │
│ (stateless I/O)  │         │ MemoryGraph     │
└───────┬──────────┘         │ EvidenceLedger  │
        │                    └────────▲────────┘
┌───────▼──────────┐                  │
│ Host / World     │ observations ────┘
│ git, CI, tools,  │
│ other agents     │
└──────────────────┘
```

Detail: [docs/01-architecture.md](docs/01-architecture.md) · Thesis: [docs/00-thesis.md](docs/00-thesis.md)

---

## Formula

```text
CRS =
  OrchestrationRuntime
+ WorkGraph                 # task DAG
+ MemoryGraph               # causal organizational memory
+ EvidenceLedger
+ JudgmentModels            # LLMs / agents (propose only)
+ HumanReviewPlane
```

---

## What CRS is **not**

- Not AGI  
- Not a replacement for better/larger judgment models  
- Not LangGraph under another name (the differentiator is **authority + governed memory**)  
- Does not require a human on every action — it requires humans on **cold memory** and real risk gates  

---

## Status

This repository publishes the **architecture proposal** (concept, contracts, schemas).

| Layer | Status |
|---|---|
| Thesis and contracts | v0.3 (this repo) |
| JSON schemas (WorkGraph / MemoryGraph / Proposal) | draft in `schemas/` |
| Reference runtime (implementation) | planned |
| Integration with real swarms (e.g. coding agents) | planned |

---

## Principles (non-negotiable)

1. **Control and invariants are code** — they are not sampled by the network.  
2. **Work and memory are mutable state** — not only prose in context.  
3. **Growth requires evaluated experience** — logging ≠ learning.  
4. **Durable belief requires provenance** — and, when policy requires it, review.  
5. **Finite budget** — a swarm without a ceiling is a loop, not intelligence.

---

## Contributing / discussion

Issues and discussions are welcome — especially on:

- memory schemas and cold-promotion policies  
- `HELPED` / `HURT` / `UNUSED` metrics  
- real-world Memory Reviewer roles in companies  

---

## License

MIT — see [LICENSE](LICENSE).

---

## Short citation

> The model is commodity.  
> Orchestration with laws and governed organizational memory are the moat.
