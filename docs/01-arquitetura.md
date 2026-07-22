# Arquitetura CRS v0.3

## Componentes

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

Máquina de estados do trabalho coletivo.

Responsável por:

- aceitar goals  
- receber proposals de plano  
- validar e **commitar DAG** no WorkGraph  
- spawn / join de agentes  
- budgets e stop conditions  
- pedir review humano quando a política exige  
- montar a *view* (contexto filtrado) para juízes  

### WorkGraph

Estado durável das tarefas e dependências (DAG).

Ver [02-orquestracao-dag.md](02-orquestracao-dag.md).

### MemoryGraph

Estado durável do que a organização sabe, acredita e já sofreu.

Ver [03-memory-graph.md](03-memory-graph.md).

### EvidenceLedger

Registro de observações verificáveis:

- saída de tool  
- log de CI  
- hash de artifact  
- URL / snapshot  
- atestação humana  

Nenhuma belief/lesson durável deveria existir sem caminho até evidência (ou rejeição explícita da política — o padrão é fail-closed).

### JudgmentModels

LLMs e agentes especializados. **Stateless do ponto de vista do commit.**

Eles recebem uma view e emitem **Proposal** tipada.  
Nunca escrevem direto em WorkGraph/MemoryGraph.

### HumanReviewPlane

Superfície onde humanos:

- aprovam / editam / rejeitam memória candidata  
- resolvem conflitos de crença  
- autorizam ações de alto risco  

Ver [04-human-memory-review.md](04-human-memory-review.md).

## Fluxo ponta a ponta

```text
1. Human/API cria Goal
2. Runtime pede proposal de plano (JudgmentModel)
3. Runtime valida e committa DAG no WorkGraph
4. Runtime spawna nós liberados (dependências satisfeitas)
5. Host executa; observações → EvidenceLedger
6. Runtime pede critique/merge proposals
7. Summarizer propõe Lessons/Beliefs (warm)
8. Verifier e/ou Human Memory Review
9. Runtime committa no MemoryGraph (cold se aprovado)
10. Próximo Goal recupera memória com orçamento de view
11. Impacto HELPED/HURT/UNUSED atualiza o grafo
```

## Fases normativas (v0.3)

| Fase | Entrada | Saída típica |
|---|---|---|
| `frame` | goal | escopo, constraints |
| `plan` | frame + memória | proposal de DAG |
| `spawn` | DAG commitado | runs filhos |
| `observe` | tool/agent results | evidence |
| `critique` | evidence + artifacts | accept/reject/rework proposals |
| `replan` | falhas / novo contexto | DAG patch |
| `consolidate` | episodes | memory proposals |
| `stop` | done / budget / halt | terminal state |

Saltos de fase silenciosos são **non-conformant**.  
Skip deve ser explícito na policy.

## View (o antídoto ao context dump)

A cada juízo, o runtime projeta:

1. goal + constraints ativas  
2. trecho do DAG relevante  
3. lessons `active` no escopo  
4. beliefs `accepted` / `contested`  
5. falhas recentes (`HURT`) do padrão  
6. budgets restantes  
7. decisões humanas que vetam caminhos  

O modelo **não** recebe o grafo inteiro.  
Recebe uma view orçada e hasheada (`input_view_hash` na Proposal).

## Critérios de conformidade

Uma implementação só pode se chamar CRS-like se:

1. modelos não commitam estado durável sozinhos  
2. há DAG (ou equivalente) com dependências explícitas  
3. memória durável tem proveniência  
4. existe caminho de reject/retract  
5. budgets limitam o swarm  
6. review humano (ou verifier formal) governa promoção cold  

Sem isso, é agent framework com marketing.
