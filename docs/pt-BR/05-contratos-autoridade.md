# Contratos de autoridade (CRS v0.3)

**[English](../05-authority-contracts.md)** · **Português (Brasil)**

## Princípio

Se o modelo pode sozinho mutar o que a organização acredita e reutiliza, você não tem runtime cognitivo — tem monólogo com side effects.

## Contrato I — Estado

Tipos lógicos mínimos:

- `Goal`, `WorkNode` (DAG)  
- `Evidence`  
- `Belief`  
- `Lesson`  
- `Episode`  
- `HumanDecision`  
- `Proposal`  

Estados de belief (exemplo):

`candidate | provisional | accepted | contested | suspended | retracted | superseded`

## Contrato II — Autoridade

| Ator | MAY | MUST NOT |
|---|---|---|
| **JudgmentModel** | emitir Proposal tipada | mutar WorkGraph, MemoryGraph, EvidenceLedger, budgets, fase |
| **Runtime** | validar, reject, commit; mudar fase; enforçar budget | tratar prosa livre como commit |
| **Host** | executar ações no mundo; devolver observations | escrever memória cold fora das APIs do runtime |
| **Human** | memory review; override mediado; halt | bypass do audit log (SHOULD NOT) |

## Contrato III — Proposal

Todo output de modelo que possa afetar cognição/orquestração **deve** ser envelopado:

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

Openings normativas v0.3 (conjunto mínimo):

| Opening | Uso |
|---|---|
| `propose_plan` | DAG inicial |
| `propose_replan` | patch do DAG |
| `rank_actions` | próximas ações |
| `split_work` | fan-out |
| `accept_child_result` | merge de filho |
| `propose_belief` | belief candidata |
| `propose_lesson` | lesson candidata |
| `request_human_review` | escalada |
| `halt` | parar com motivo |

Openings desconhecidas → **reject**.

## Contrato IV — Aprendizado

Aprendizado válido exige:

1. episode com outcome  
2. evidence  
3. mudança de estado de memória (belief/lesson)  
4. caminho de avaliação posterior (`HELPED`/`HURT`/`UNUSED`)  

Acumular texto sem (3)+(4) **não pode** ser reportado como learning success.

## Contrato V — Vida finita (budget)

Todo goal/swarm tem budget explícito:

- max children  
- max replan rounds  
- max wall-clock  
- max cost  
- max writes candidatas de memória  

Esgotado → fase `stop` com razão.  
Sem budget, o sistema não é CRS-conforme.

## Leis curtas (para lembrar)

1. Controle é código.  
2. Trabalho e memória são estado.  
3. Modelo propõe; runtime committa.  
4. Crença durável exige proveniência.  
5. Swarm sem teto é loop.
