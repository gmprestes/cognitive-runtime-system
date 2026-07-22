# Orquestração e DAG

**[English](../02-orchestration-dag.md)** · **Português (Brasil)**

## O que é um DAG

**DAG** = *Directed Acyclic Graph* (grafo dirigido acíclico).

No CRS, o DAG descreve o **trabalho**:

- nós = tarefas / subtarefas  
- arestas = “precisa terminar antes”  
- acíclico = sem dependência circular impossível  

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

## Por que orquestração multiagente precisa de DAG

Sem DAG, swarms tendem a:

- serializar demais (lento)  
- paralelizar sem ordem (caos)  
- reentrar em loops de “refazer o que ainda não terminou”  

Com DAG o runtime responde de forma determinística:

| Pergunta | Resposta |
|---|---|
| O que pode rodar agora? | nós com deps satisfeitas |
| O que é paralelo? | nós na mesma fronteira |
| O que está bloqueado? | deps incompletas |
| O goal terminou? | terminais `done` / `cancelled` |
| O plano é inválido? | ciclo, budget excedido, owner ausente |

## Proposal vs commit do DAG

Este é o coração da orquestração no CRS.

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

Isso **ainda não** é o plano oficial.

### 2. Validação (runtime)

O runtime checa, em código:

- aciclicidade  
- referências `depends_on`  
- owners conhecidos  
- budget (nº de nós, profundidade, custo estimado)  
- constraints / lessons que proíbem o caminho  
- se um gate humano é necessário  

Falha → reject (com motivo). O modelo pode re-propor.

### 3. Commit (runtime)

**Commitar o DAG** = gravar o plano no WorkGraph como estado durável.

Depois do commit:

- spawns são reais  
- progresso é medido nó a nó  
- replan é um patch explícito (não “mudar de ideia em prosa”)  

Analogia:

| Git | CRS |
|---|---|
| working tree | proposta de plano |
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
- respeita o budget global do goal  

## Condições de parada (exemplos de leis)

O runtime **deve** poder parar quando:

- todos os terminais estão done  
- o budget global se esgotou  
- contradição bloqueante sem path de resolve  
- halt humano  
- risco acima do teto sem aprovação  

Swarm sem condições de parada não é orquestração — é custo.

## Relação com o Memory Graph

O DAG é o **trabalho desta execução**.  
O Memory Graph é o **que persiste entre execuções**.

Episodes de execução do DAG alimentam propostas de Lesson/Belief.  
Lessons ativas influenciam o *próximo* plan (via view); não reescrevem o passado em silêncio.
