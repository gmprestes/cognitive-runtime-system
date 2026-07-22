# Orquestração e DAG

## O que é um DAG

**DAG** = *Directed Acyclic Graph* = grafo **dirigido** e **acíclico**.

No CRS, o DAG descreve o **trabalho**:

- nós = tarefas / subtarefas  
- setas = “precisa terminar antes”  
- sem ciclos = não existe dependência circular impossível  

### Exemplo

```text
        [frame/plan]
             │
             ▼
      [recon do repo]
             │
       ┌─────┴─────┐
       ▼           ▼
 [implement A] [implement B]   ← paralelo
       │           │
       └─────┬─────┘
             ▼
         [testes]
             │
             ▼
      [PR + resumo]
```

## Por que orquestração multi-agente precisa de DAG

Sem DAG, o swarm tende a:

- serializar demais (lento)  
- paralelizar sem ordem (caos)  
- reentrar em loops de “refazer o que ainda não terminou”  

Com DAG o runtime responde de forma determinística:

| Pergunta | Resposta |
|---|---|
| O que pode rodar agora? | nós com deps satisfeitas |
| O que é paralelo? | nós no mesmo frontier |
| O que está bloqueado? | deps incompletas |
| O goal acabou? | terminais `done` / `cancelled` |
| Plano inválido? | ciclo, budget estourado, dono ausente |

## Proposal vs commit de DAG

Este é o coração do CRS na orquestração.

### 1. Proposal (modelo)

O JudgmentModel sugere um plano:

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

Ainda **não** é o plano oficial.

### 2. Validação (runtime)

O runtime checa, em código:

- aciclicidade  
- referências `depends_on`  
- owners conhecidos  
- budget (nº nós, profundidade, custo estimado)  
- constraints / lessons que proíbem o caminho  
- necessidade de gate humano  

Falha → reject (com motivo). O modelo pode re-propor.

### 3. Commit (runtime)

**Commitar o DAG** = gravar o plano no WorkGraph como estado durável.

Depois do commit:

- spawns são reais  
- progresso é medido nó a nó  
- replan é patch explícito (não “mudança de ideia em prosa”)  

Analogia:

| Git | CRS |
|---|---|
| working tree | proposal de plano |
| `git commit` | **commit do DAG** |
| history | WorkGraph auditável |

## WorkGraph — campos mínimos de um nó

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

Replan **não** apaga história.

- cancela ou supersede nós  
- adiciona nós com novas deps  
- registra `reason` + evidence  
- respeita budget global do goal  

## Stop conditions (exemplos de lei)

O runtime **deve** poder parar quando:

- todos terminais done  
- budget global esgotado  
- contradição bloqueante sem path de resolve  
- humano halt  
- risco acima do teto sem aprovação  

Swarm sem stop condition não é orquestração — é custo.

## Relação com Memory Graph

O DAG é o **trabalho desta vez**.  
O Memory Graph é o **que permanece entre vezes**.

Episodes de execução do DAG alimentam proposals de Lesson/Belief.  
Lessons ativas influenciam o *próximo* plan (via view), não reescrevem o passado em silêncio.
