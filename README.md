# CRS — Cognitive Runtime System

**Orquestração com leis. Memória em grafo. Humanos como revisores do que a organização pode lembrar.**

> CRS não é “mais um multi-agent framework”.  
> É uma **proposta de arquitetura**: o ciclo de trabalho e de memória é um **programa** (runtime); o LLM só **propõe**; o commit de estado e de memória é **governado**.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## TL;DR (EN)

**CRS (Cognitive Runtime System)** is an architecture for multi-agent orchestration:

- **Runtime owns control** — phases, budgets, DAGs, stop conditions  
- **Models only propose** — they never silently mutate durable state  
- **Memory is a graph** — episodes, lessons, evidence, impact edges  
- **Humans review memory** — not every keystroke; the *beliefs* the org reuses  

Moat thesis: *commodity judgment models + proprietary orchestration + governed organizational memory.*

---

## Por que CRS existe

Hoje a indústria confunde três coisas:

| Coisa | O que costuma ser | O que CRS separa |
|---|---|---|
| **Juízo** | “O modelo é inteligente” | LLM/agentes **propõem** |
| **Controle** | Prompt + esperança | Runtime com **leis e budgets** |
| **Memória** | RAG / chat log / vector dump | **Grafo** com proveniência e review |

Sem essa separação, um swarm de agentes:

- repete erros caros  
- “aprende” rumores em alta velocidade  
- não tem dono para o que virou verdade operacional  

Com CRS:

```text
goal → proposal de plano → commit de DAG → swarm executa
     → evidências → proposal de memória → review (humano/verificador)
     → MemoryGraph commit → próxima orquestração reutiliza com critério
```

---

## Conceitos centrais

### 1. Runtime de orquestração

O CRS coordena agentes (e humanos) com:

- **DAG de trabalho** (dependências, paralelismo, sem ciclos)  
- **fases** (`plan → spawn → observe → critique → replan | merge | stop`)  
- **budgets** (filhos, rounds, wall-clock, risco)  
- **autoridade** (quem propõe vs quem committa)

Leitura: [docs/02-orquestracao-dag.md](docs/02-orquestracao-dag.md)

### 2. Memory Graph

Memória organizacional como grafo causal — não só similaridade vetorial:

- nós: Episode, Task, Agent, Evidence, Belief, Lesson, HumanDecision…  
- arestas: `SUPPORTS`, `DERIVED_FROM`, `SUPERSEDES`, `HELPED` / `HURT` / `UNUSED`…

Leitura: [docs/03-memory-graph.md](docs/03-memory-graph.md)

### 3. Human Memory Review

Humanos não babysitam cada tool call.  
Eles **autorizam o que pode ser lembrado e reaplicado**.

Três planos:

| Plano | Conteúdo | Quem escreve |
|---|---|---|
| Hot | estado da tarefa agora | runtime + agentes |
| Warm | candidatas a lição/crença | agentes propõem |
| Cold | memória canônica da org | **humano** (ou dual control) |

Leitura: [docs/04-human-memory-review.md](docs/04-human-memory-review.md)

### 4. Contrato de autoridade

```text
JudgmentModel  → MAY propose, MUST NOT mutate durable state
Runtime        → MAY validate/commit/reject, enforces laws
Host/World     → executes tools, returns observations
Human          → reviews memory & high-risk gates
```

Leitura: [docs/05-contratos-autoridade.md](docs/05-contratos-autoridade.md)

---

## Arquitetura (visão)

```text
┌──────────────────────────────────────────────────────┐
│  Human Review Plane                                  │
│  approve / amend / reject memória e gates de risco   │
└───────────────────────────┬──────────────────────────┘
┌───────────────────────────▼──────────────────────────┐
│  CRS Orchestration Runtime                           │
│  phases · budgets · authority · stop conditions      │
└───────┬─────────────────────────────┬────────────────┘
        │ proposals                   │ commits
┌───────▼──────────┐         ┌────────▼────────┐
│ Judgment layer   │         │ Durable state   │
│ LLMs / agents    │         │ WorkGraph (DAG) │
│ (stateless I/O)  │         │ MemoryGraph     │
└───────┬──────────┘         │ EvidenceLedger  │
        │                    └────────▲────────┘
┌───────▼──────────┐                  │
│ Host / World     │ observations ────┘
│ git, CI, tools,  │
│ other agents     │
└──────────────────┘
```

Detalhe: [docs/01-arquitetura.md](docs/01-arquitetura.md) · Tese: [docs/00-tese.md](docs/00-tese.md)

---

## Fórmula

```text
CRS =
  OrchestrationRuntime
+ WorkGraph                 # DAG de tarefas
+ MemoryGraph               # memória causal da organização
+ EvidenceLedger
+ JudgmentModels            # LLMs / agentes (só propõem)
+ HumanReviewPlane
```

---

## O que CRS **não** é

- Não é AGI  
- Não substitui modelos maiores/melhores de juízo  
- Não é LangGraph com outro nome (o diferencial é **autoridade + memória governada**)  
- Não exige humano em cada ação — exige humano no **cold memory** e em riscos reais  

---

## Status

Este repositório publica a **proposta de arquitetura** (conceito, contratos, schemas).

| Camada | Status |
|---|---|
| Tese e contratos | v0.3 (este repo) |
| Schemas JSON (WorkGraph / MemoryGraph / Proposal) | rascunho em `schemas/` |
| Runtime de referência (implementação) | planejado |
| Integração com swarms reais (ex.: coding agents) | planejado |

Linha de pesquisa anterior (L1 Rust, evals A/B, proxy OpenAI-like) validou partes do contrato de autoridade e foi arquivada. A v0.3 **reorienta o CRS para orquestração multi-agente + memória-grafo com review humano**.

---

## Conteúdo para divulgação

- [content/linkedin.md](content/linkedin.md) — post LinkedIn  
- [content/x-thread.md](content/x-thread.md) — thread no X  

---

## Princípios (não negociáveis)

1. **Controle e invariantes são código** — não são amostrados pela rede.  
2. **Trabalho e memória são estado mutável** — não só prosa no contexto.  
3. **Crescimento exige experiência avaliada** — gravar ≠ aprender.  
4. **Crença durável exige proveniência** — e, quando a política manda, review.  
5. **Budget finito** — swarm sem teto é loop, não inteligência.

---

## Contribuir / conversar

Issues e discussões são bem-vindas — especialmente:

- schemas de memória e políticas de promoção cold  
- métricas de `HELPED` / `HURT` / `UNUSED`  
- papéis humanos de memory reviewer em empresas reais  

---

## Licença

MIT — ver [LICENSE](LICENSE).

---

## Citação curta

> O modelo é commodity.  
> A orquestração com leis e a memória governada da organização são o moat.
