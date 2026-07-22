# Orchestration and DAG

**English** · **[Português (Brasil)](pt-BR/02-orquestracao-dag.md)**

## What a DAG is

**DAG** = *Directed Acyclic Graph*.

In CRS, the DAG describes **work**:

- nodes = tasks / subtasks  
- edges = “must finish before”  
- acyclic = no impossible circular dependency  

### Example

```text
        [frame/plan]
             │
             ▼
      [repo recon]
             │
       ┌─────┴─────┐
       ▼           ▼
 [implement A] [implement B]   ← parallel
       │           │
       └─────┬─────┘
             ▼
          [tests]
             │
             ▼
      [PR + summary]
```

## Why multi-agent orchestration needs a DAG

Without a DAG, swarms tend to:

- serialize too much (slow)  
- parallelize without order (chaos)  
- re-enter loops of “redo what has not finished yet”  

With a DAG the runtime answers deterministically:

| Question | Answer |
|---|---|
| What can run now? | nodes with satisfied deps |
| What is parallel? | nodes on the same frontier |
| What is blocked? | incomplete deps |
| Is the goal finished? | terminals `done` / `cancelled` |
| Is the plan invalid? | cycle, budget exceeded, missing owner |

## Proposal vs DAG commit

This is the heart of CRS orchestration.

### 1. Proposal (model)

The JudgmentModel suggests a plan:

```json
{
  "opening": "propose_plan",
  "payload": {
    "nodes": [
      {"id": "t1", "title": "Recon", "owner_role": "explorer"},
      {"id": "t2", "title": "Implement fix", "owner_role": "implementer", "depends_on": ["t1"]},
      {"id": "t3", "title": "Tests", "owner_role": "tester", "depends_on": ["t2"]}
    ]
  }
}
```

This is **not** yet the official plan.

### 2. Validation (runtime)

The runtime checks, in code:

- acyclicity  
- `depends_on` references  
- known owners  
- budget (node count, depth, estimated cost)  
- constraints / lessons that forbid the path  
- whether a human gate is required  

Failure → reject (with reason). The model may re-propose.

### 3. Commit (runtime)

**Committing the DAG** = writing the plan into the WorkGraph as durable state.

After commit:

- spawns are real  
- progress is measured node by node  
- replan is an explicit patch (not “changing your mind in prose”)  

Analogy:

| Git | CRS |
|---|---|
| working tree | plan proposal |
| `git commit` | **DAG commit** |
| history | auditable WorkGraph |

## WorkGraph — minimum node fields

```text
WorkNode {
  id
  title
  goal_id
  depends_on[]
  owner_role | owner_agent_id
  status: pending | ready | running | blocked | done | failed | cancelled
  budget: { max_minutes?, max_cost?, max_tool_calls? }
  evidence_ids[]
  artifact_ids[]
  created_at, updated_at
}
```

## Replan

Replan does **not** erase history.

- cancels or supersedes nodes  
- adds nodes with new deps  
- records `reason` + evidence  
- respects the goal’s global budget  

## Stop conditions (example laws)

The runtime **must** be able to stop when:

- all terminals are done  
- global budget is exhausted  
- a blocking contradiction has no resolve path  
- human halt  
- risk is above the ceiling without approval  

A swarm without stop conditions is not orchestration — it is cost.

## Relationship to Memory Graph

The DAG is **this run’s work**.  
The Memory Graph is **what persists across runs**.

DAG execution episodes feed Lesson/Belief proposals.  
Active lessons influence the *next* plan (via view); they do not silently rewrite the past.
