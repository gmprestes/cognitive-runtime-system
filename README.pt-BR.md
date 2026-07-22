# CRS — Cognitive Runtime System

**[English](README.md)** · **Português (Brasil)**

**Orquestração com leis. Memória em grafo. Humanos como revisores do que a organização tem permissão de lembrar.**

> CRS não é “mais um multi-agent framework”.  
> É uma **proposta de arquitetura**: o ciclo de trabalho e o ciclo de memória são um **programa** (runtime); o LLM só **propõe**; commits de estado e de memória são **governados**.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## Resumo

**CRS (Cognitive Runtime System)** é uma arquitetura de orquestração multiagente:

- **O runtime controla** — fases, budgets, DAGs, condições de parada  
- **Modelos só propõem** — nunca mutam estado durável em silêncio  
- **Memória é um grafo** — episódios, lições, evidências, arestas de impacto  
- **Humanos revisam memória** — não cada keystroke; as *crenças* que a org reutiliza  

Tese de moat: *modelos de juízo commodity + orquestração proprietária + memória organizacional governada.*

---

## Por que o CRS existe

A indústria costuma confundir três coisas diferentes:

| Coisa | Prática comum | Separação no CRS |
|---|---|---|
| **Juízo** | “O modelo é inteligente” | LLMs/agentes **propõem** |
| **Controle** | Prompt + esperança | Runtime com **leis e budgets** |
| **Memória** | RAG / log de chat / dump vetorial | **Grafo** com proveniência e review |

Sem essa separação, um swarm de agentes:

- repete erros caros  
- “aprende” rumores em alta velocidade  
- não tem dono da verdade operacional  

Com CRS:

```text
objetivo → proposta de plano → commit do DAG → swarm executa
         → evidência → proposta de memória → review humano/verificador
         → commit no MemoryGraph → próxima orquestração reutiliza com critério
```

---

## Conceitos centrais

### 1. Runtime de orquestração

O CRS coordena agentes (e humanos) com:

- um **DAG de trabalho** (dependências, paralelismo, sem ciclos)  
- **fases** (`plan → spawn → observe → critique → replan | merge | stop`)  
- **budgets** (filhos, rounds, wall-clock, risco)  
- **autoridade** (quem propõe vs quem committa)

Leia: [docs/pt-BR/02-orquestracao-dag.md](docs/pt-BR/02-orquestracao-dag.md)

### 2. Memory Graph

Memória organizacional como grafo causal — não só similaridade vetorial:

- nós: Episode, Task, Agent, Evidence, Belief, Lesson, HumanDecision…  
- arestas: `SUPPORTS`, `DERIVED_FROM`, `SUPERSEDES`, `HELPED` / `HURT` / `UNUSED`…

Leia: [docs/pt-BR/03-memory-graph.md](docs/pt-BR/03-memory-graph.md)

### 3. Human Memory Review

Humanos não babysitam cada tool call.  
Eles **autorizam o que pode ser lembrado e reaplicado**.

Três planos de memória:

| Plano | Conteúdo | Quem escreve |
|---|---|---|
| Hot | estado da tarefa agora | runtime + agentes |
| Warm | lições/crenças candidatas | agentes propõem |
| Cold | memória canônica da org | **humano** (ou dual control) |

Leia: [docs/pt-BR/04-human-memory-review.md](docs/pt-BR/04-human-memory-review.md)

### 4. Contrato de autoridade

```text
JudgmentModel  → MAY propor, MUST NOT mutar estado durável
Runtime        → MAY validar/commitar/rejeitar, aplica leis
Host/World     → executa tools, devolve observações
Human          → revisa memória e gates de alto risco
```

Leia: [docs/pt-BR/05-contratos-autoridade.md](docs/pt-BR/05-contratos-autoridade.md)

---

## Arquitetura (visão geral)

```text
┌──────────────────────────────────────────────────────┐
│  Plano de Review Humano                              │
│  approve / amend / reject memória e gates de risco   │
└───────────────────────────┬──────────────────────────┘
┌───────────────────────────▼──────────────────────────┐
│  CRS Orchestration Runtime                           │
│  fases · budgets · autoridade · condições de parada  │
└───────┬─────────────────────────────┬────────────────┘
        │ proposals                   │ commits
┌───────▼──────────┐         ┌────────▼────────┐
│ Camada de juízo  │         │ Estado durável  │
│ LLMs / agentes   │         │ WorkGraph (DAG) │
│ (I/O sem estado) │         │ MemoryGraph     │
└───────┬──────────┘         │ EvidenceLedger  │
        │                    └────────▲────────┘
┌───────▼──────────┐                  │
│ Host / Mundo     │ observações ─────┘
│ git, CI, tools,  │
│ outros agentes   │
└──────────────────┘
```

Detalhe: [docs/pt-BR/01-arquitetura.md](docs/pt-BR/01-arquitetura.md) · Tese: [docs/pt-BR/00-tese.md](docs/pt-BR/00-tese.md)

---

## Fórmula

```text
CRS =
  OrchestrationRuntime
+ WorkGraph                 # DAG de tarefas
+ MemoryGraph               # memória organizacional causal
+ EvidenceLedger
+ JudgmentModels            # LLMs / agentes (só propõem)
+ HumanReviewPlane
```

---

## O que o CRS **não** é

- Não é AGI  
- Não substitui modelos de juízo melhores/maiores  
- Não é LangGraph com outro nome (o diferencial é **autoridade + memória governada**)  
- Não exige humano em toda ação — exige humanos na **memória cold** e em gates de risco reais  

---

## Status

Este repositório publica a **proposta de arquitetura** (conceito, contratos, schemas).

| Camada | Status |
|---|---|
| Tese e contratos | v0.3 (este repo) |
| JSON schemas (WorkGraph / MemoryGraph / Proposal) | rascunho em `schemas/` |
| Runtime de referência (implementação) | planejado |
| Integração com swarms reais (ex.: coding agents) | planejado |

---

## Documentação (pt-BR)

| Documento | Descrição |
|---|---|
| [00-tese.md](docs/pt-BR/00-tese.md) | Tese e aposta |
| [01-arquitetura.md](docs/pt-BR/01-arquitetura.md) | Componentes e fluxo |
| [02-orquestracao-dag.md](docs/pt-BR/02-orquestracao-dag.md) | Orquestração e DAG |
| [03-memory-graph.md](docs/pt-BR/03-memory-graph.md) | Memory Graph |
| [04-human-memory-review.md](docs/pt-BR/04-human-memory-review.md) | Review humano de memória |
| [05-contratos-autoridade.md](docs/pt-BR/05-contratos-autoridade.md) | Contratos de autoridade |

Versão em inglês: [README.md](README.md) e [docs/](docs/).

---

## Princípios (não negociáveis)

1. **Controle e invariantes são código** — não são amostrados pela rede.  
2. **Trabalho e memória são estado mutável** — não só prosa no contexto.  
3. **Crescimento exige experiência avaliada** — logar ≠ aprender.  
4. **Crença durável exige proveniência** — e, quando a política exige, review.  
5. **Budget finito** — swarm sem teto é loop, não inteligência.

---

## Contribuir / discussão

Issues e discussões são bem-vindas — especialmente sobre:

- schemas de memória e políticas de promoção cold  
- métricas `HELPED` / `HURT` / `UNUSED`  
- papéis reais de Memory Reviewer em empresas  

---

## Licença

MIT — veja [LICENSE](LICENSE).

---

## Citação curta

> O modelo é commodity.  
> Orquestração com leis e memória organizacional governada são o moat.
