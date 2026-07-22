# CRS Architecture v0.3

**English** · **[Português (Brasil)](pt-BR/01-arquitetura.md)**

## Components

```text
CRS =
  OrchestrationRuntime
+ WorkGraph
+ MemoryGraph
+ EvidenceLedger
+ JudgmentModels
+ HumanReviewPlane
```

### OrchestrationRuntime

State machine for collective work.

Responsible for:

- accepting goals  
- receiving plan proposals  
- validating and **committing a DAG** into the WorkGraph  
- spawning / joining agents  
- budgets and stop conditions  
- requesting human review when policy requires it  
- building the *view* (filtered context) for judges  

### WorkGraph

Durable state of tasks and dependencies (DAG).

See [02-orchestration-dag.md](02-orchestration-dag.md).

### MemoryGraph

Durable state of what the organization knows, believes, and has already suffered.

See [03-memory-graph.md](03-memory-graph.md).

### EvidenceLedger

Registry of verifiable observations:

- tool output  
- CI log  
- artifact hash  
- URL / snapshot  
- human attestation  

No durable belief/lesson should exist without a path to evidence (or an explicit policy rejection of that rule — the default is fail-closed).

### JudgmentModels

LLMs and specialized agents. **Stateless with respect to commit.**

They receive a view and emit typed **Proposals**.  
They never write directly to WorkGraph/MemoryGraph.

### HumanReviewPlane

Surface where humans:

- approve / edit / reject candidate memory  
- resolve belief conflicts  
- authorize high-risk actions  

See [04-human-memory-review.md](04-human-memory-review.md).

## End-to-end flow

```text
1. Human/API creates Goal
2. Runtime requests plan proposal (JudgmentModel)
3. Runtime validates and commits DAG into WorkGraph
4. Runtime spawns nodes whose dependencies are satisfied
5. Host executes; observations → EvidenceLedger
6. Runtime requests critique/merge proposals
7. Summarizer proposes Lessons/Beliefs (warm)
8. Verifier and/or Human Memory Review
9. Runtime commits into MemoryGraph (cold if approved)
10. Next Goal retrieves memory with a view budget
11. HELPED/HURT/UNUSED impact updates the graph
```

## Normative phases (v0.3)

| Phase | Input | Typical output |
|---|---|---|
| `frame` | goal | scope, constraints |
| `plan` | frame + memory | DAG proposal |
| `spawn` | committed DAG | child runs |
| `observe` | tool/agent results | evidence |
| `critique` | evidence + artifacts | accept/reject/rework proposals |
| `replan` | failures / new context | DAG patch |
| `consolidate` | episodes | memory proposals |
| `stop` | done / budget / halt | terminal state |

Silent phase jumps are **non-conformant**.  
Skips must be explicit in policy.

## View (the antidote to context dump)

On every judgment, the runtime projects:

1. goal + active constraints  
2. relevant DAG slice  
3. `active` lessons in scope  
4. `accepted` / `contested` beliefs  
5. recent failures (`HURT`) of the same pattern  
6. remaining budgets  
7. human decisions that veto paths  

The model does **not** receive the whole graph.  
It receives a budgeted, hashed view (`input_view_hash` on the Proposal).

## Conformance criteria

An implementation may only call itself CRS-like if:

1. models do not commit durable state alone  
2. there is a DAG (or equivalent) with explicit dependencies  
3. durable memory has provenance  
4. there is a reject/retract path  
5. budgets bound the swarm  
6. human review (or a formal verifier) governs cold promotion  

Without that, it is an agent framework with marketing.
