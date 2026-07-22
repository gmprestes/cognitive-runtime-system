# Authority contracts (CRS v0.3)

**English** · **[Português (Brasil)](pt-BR/05-contratos-autoridade.md)**

## Principle

If the model can alone mutate what the organization believes and reuses, you do not have a cognitive runtime — you have a monologue with side effects.

## Contract I — State

Minimum logical types:

- `Goal`, `WorkNode` (DAG)  
- `Evidence`  
- `Belief`  
- `Lesson`  
- `Episode`  
- `HumanDecision`  
- `Proposal`  

Belief states (example):

`candidate | provisional | accepted | contested | suspended | retracted | superseded`

## Contract II — Authority

| Actor | MAY | MUST NOT |
|---|---|---|
| **JudgmentModel** | emit typed Proposal | mutate WorkGraph, MemoryGraph, EvidenceLedger, budgets, phase |
| **Runtime** | validate, reject, commit; change phase; enforce budget | treat free-form prose as commit |
| **Host** | execute world actions; return observations | write cold memory outside runtime APIs |
| **Human** | memory review; mediated override; halt | bypass audit log (SHOULD NOT) |

## Contract III — Proposal

Every model output that can affect cognition/orchestration **must** be enveloped:

```json
{
  "id": "prop_…",
  "opening": "propose_plan",
  "created_at": "…",
  "model_id": "…",
  "input_view_hash": "…",
  "uncertainty": 0.0,
  "payload": {}
}
```

Normative openings v0.3 (minimum set):

| Opening | Use |
|---|---|
| `propose_plan` | initial DAG |
| `propose_replan` | DAG patch |
| `rank_actions` | next actions |
| `split_work` | fan-out |
| `accept_child_result` | child merge |
| `propose_belief` | candidate belief |
| `propose_lesson` | candidate lesson |
| `request_human_review` | escalation |
| `halt` | stop with reason |

Unknown openings → **reject**.

## Contract IV — Learning

Valid learning requires:

1. episode with outcome  
2. evidence  
3. memory state change (belief/lesson)  
4. a later evaluation path (`HELPED`/`HURT`/`UNUSED`)  

Accumulating text without (3)+(4) **must not** be reported as learning success.

## Contract V — Finite life (budget)

Every goal/swarm has an explicit budget:

- max children  
- max replan rounds  
- max wall-clock  
- max cost  
- max candidate memory writes  

Exhausted → `stop` phase with reason.  
Without budget, the system is not CRS-conformant.

## Short laws (to remember)

1. Control is code.  
2. Work and memory are state.  
3. Model proposes; runtime commits.  
4. Durable belief requires provenance.  
5. A swarm without a ceiling is a loop.
