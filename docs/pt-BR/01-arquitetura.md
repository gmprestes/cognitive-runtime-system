# Arquitetura CRS v0.3

**[English](../01-architecture.md)** · **Português (Brasil)**

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
- receber propostas de plano  
- validar e **commitar um DAG** no WorkGraph  
- spawn / join de agentes  
- budgets e condições de parada  
- solicitar review humano quando a política exige  
- montar a *view* (contexto filtrado) para os juízes  

### WorkGraph

Estado durável de tarefas e dependências (DAG).

Veja [02-orquestracao-dag.md](02-orquestracao-dag.md).

### MemoryGraph

Estado durável do que a organização sabe, acredita e já sofreu.

Veja [03-memory-graph.md](03-memory-graph.md).

### EvidenceLedger

Registro de observações verificáveis:

- saída de tool  
- log de CI  
- hash de artifact  
- URL / snapshot  
- atestação humana  

Nenhuma belief/lesson durável deveria existir sem caminho até evidência (ou rejeição explícita dessa regra na política — o padrão é fail-closed).

### JudgmentModels

LLMs e agentes especializados. **Sem estado em relação ao commit.**

Recebem uma view e emitem **Proposals** tipadas.  
Nunca escrevem diretamente no WorkGraph/MemoryGraph.

### HumanReviewPlane

Superfície onde humanos:

- aprovam / editam / rejeitam memória candidata  
- resolvem conflitos de crença  
- autorizam ações de alto risco  

Veja [04-human-memory-review.md](04-human-memory-review.md).

## Fluxo ponta a ponta

```text
1. Humano/API cria Goal
2. Runtime pede proposta de plano (JudgmentModel)
3. Runtime valida e committa o DAG no WorkGraph
4. Runtime spawna nós com dependências satisfeitas
5. Host executa; observações → EvidenceLedger
6. Runtime pede propostas de critique/merge
7. Summarizer propõe Lessons/Beliefs (warm)
8. Verifier e/ou Human Memory Review
9. Runtime committa no MemoryGraph (cold se aprovado)
10. Próximo Goal recupera memória com budget de view
11. Impacto HELPED/HURT/UNUSED atualiza o grafo
```

## Fases normativas (v0.3)

| Fase | Entrada | Saída típica |
|---|---|---|
| `frame` | goal | escopo, constraints |
| `plan` | frame + memória | proposta de DAG |
| `spawn` | DAG commitado | runs filhos |
| `observe` | resultados de tool/agente | evidência |
| `critique` | evidência + artifacts | propostas accept/reject/rework |
| `replan` | falhas / novo contexto | patch do DAG |
| `consolidate` | episodes | propostas de memória |
| `stop` | done / budget / halt | estado terminal |

Saltos silenciosos de fase são **non-conformant**.  
Skips devem ser explícitos na policy.

## View (o antídoto ao dump de contexto)

Em todo juízo, o runtime projeta:

1. goal + constraints ativas  
2. fatia relevante do DAG  
3. lessons `active` no escopo  
4. beliefs `accepted` / `contested`  
5. falhas recentes (`HURT`) do mesmo padrão  
6. budgets restantes  
7. decisões humanas que vetam caminhos  

O modelo **não** recebe o grafo inteiro.  
Recebe uma view com budget e hash (`input_view_hash` na Proposal).

## Critérios de conformidade

Uma implementação só pode se chamar CRS-like se:

1. modelos não commitam estado durável sozinhos  
2. existe um DAG (ou equivalente) com dependências explícitas  
3. memória durável tem proveniência  
4. existe caminho de reject/retract  
5. budgets limitam o swarm  
6. review humano (ou verificador formal) governa a promoção cold  

Sem isso, é agent framework com marketing.
