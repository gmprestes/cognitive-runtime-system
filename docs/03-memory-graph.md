# Memory Graph

**English** · **[Português (Brasil)](pt-BR/03-memory-graph.md)**

## Idea

Organizational memory is not:

- only the context window  
- only a vector store of “similar texts”  
- only the chat log  

It is a **graph** of what the organization:

- tried (episodes)  
- observed (evidence)  
- believes (beliefs)  
- learned to do (lessons)  
- decided with humans (reviews)  
- measured as impact (`HELPED` / `HURT` / `UNUSED`)

## Why a graph

Orchestration needs questions that similarity answers poorly:

- What have we already tried for *this* failure pattern?  
- Who vetoed this procedure and why?  
- Does this lesson still help or does it hurt?  
- Which evidence supports this belief?  
- What supersedes what?

That is **structure + time + causality**.

## Node types (MVP)

| Type | Description |
|---|---|
| `Episode` | one execution (run, PR attempt, heartbeat) |
| `Task` / `Goal` | business objective or issue |
| `Agent` | role or instance |
| `Artifact` | diff, report, binary, doc |
| `Evidence` | verifiable observation |
| `Belief` | revisable claim |
| `Lesson` | reusable procedure / strategy / constraint |
| `Entity` | repo, service, API, customer, policy |
| `HumanDecision` | approve / reject / amend |

## Edge types (MVP)

| Edge | Meaning |
|---|---|
| `SUPPORTS` | evidence supports belief/lesson |
| `CONTRADICTS` | evidence weakens |
| `DERIVED_FROM` | lesson/belief ← episodes |
| `EXECUTED_BY` | episode → agent |
| `PRODUCED` | episode → artifact |
| `BLOCKS` / `ENABLES` | work relations (may mirror the DAG) |
| `SUPERSEDES` | new version replaces old |
| `REVIEWED_BY` | human decision → memory node |
| `APPLIED_IN` | lesson used in a later episode |
| `HELPED` / `HURT` / `UNUSED` | measured impact |

## Belief vs Lesson

| | Belief | Lesson |
|---|---|---|
| Nature | “what is true *here*” | “how to act *here*” |
| Example | “auth lives in `src/auth`” | “after schema change, run migrate + typecheck” |
| Risk if wrong | false operational fact | bad procedure reapplied |
| Promotion preference | strong evidence | evidence + outcome |

Open-world beliefs should be treated with skepticism; the graph shines on **situated truth** (this repo, this product, this process).

## Memory planes

| Plane | Volatility | Write path |
|---|---|---|
| **Hot** | episodic | free under runtime |
| **Warm** | candidate | agents propose |
| **Cold** | canonical | review / dual control |

Promoting warm → cold without a gate (verifier or human) is **non-conformant** in CRS v0.3 for procedure lessons and constraints.

## Write policy (minimum)

A candidate lesson **must be rejected** unless:

1. it has `DERIVED_FROM` ≥ 1 episode with an outcome, **and**  
2. it has linked evidence, **and**  
3. it passes at least one gate:

| Gate | Use |
|---|---|
| Verifier | programmatic tests/CI/assert |
| Human memory review | preference, policy, style, organizational risk |
| Dual control | production / security / compliance |

## Retrieve policy

When building the view:

- prioritize lessons with high `HELPED` in scope  
- include recent `HURT` as **anti-patterns**  
- degrade prolonged `UNUSED`  
- respect scope (repo, team, environment)  
- budget tokens/nodes (memory without a budget becomes a dump)

## Forgetting (required in the design)

Without retirement, the graph becomes landfill.

Example policies:

- `UNUSED` for N periods → automatic `retired`  
- `HURT` above threshold → `contested` + review  
- `SUPERSEDES` marks the ancestor as non-default  

## What not to do

- Store long LLM monologues as lessons  
- Accept “the agent said so” without evidence  
- Global memory without scope  
- Confuse embedding recall with learning  

**Logging ≠ learning.**  
Learning = persistent change that improves future decisions *with evaluation*.
