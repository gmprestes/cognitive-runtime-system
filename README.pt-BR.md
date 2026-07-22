# CRS — Cognitive Runtime System

**[English](README.md)** · **Português (Brasil)**

**Orquestração com leis. Memória em grafo. Humanos como revisores do que a organização tem permissão de lembrar.**

> CRS não é “mais um multi-agent framework”.  
> É uma **proposta de arquitetura**: o ciclo de trabalho e o ciclo de memória são um **programa** (runtime); o LLM só **propõe**; commits de estado e de memória são **governados**.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## Diagrama de capa

![Arquitetura CRS: plano de review humano, runtime de orquestração, modelos de juízo propondo, WorkGraph e MemoryGraph commitando, host devolvendo evidência](docs/assets/crs-cover.svg)

*Modelo propõe → runtime valida e committa → organização assina o que pode ser lembrado.*

---

## A ideia em linguagem simples

A maior parte dos sistemas multiagente trata o modelo como a *mente* do sistema:

- o modelo planeja  
- o modelo decide quando chamar tools  
- o modelo “lembra” via histórico de chat ou embeddings  
- humanos ou aprovam cada ação ou somem do loop  

Isso funciona em demos. Quebra em organizações.

**O CRS parte de outra aposta:**

| O que parece “inteligência” | Quem deveria ser dono no CRS |
|---|---|
| Julgar sob incerteza | **Modelo** (LLM / agente) — *só propõe* |
| Processo, limites, condições de parada | **Runtime** — *leis em código* |
| O que a empresa reutiliza na semana que vem | **Memory Graph** + **review humano** |
| Agir no mundo (git, CI, APIs) | **Host** — tools e side effects |

Em uma frase:

> **O modelo propõe. O runtime committa. A organização assina o que lembra.**

Essa é a ideia inteira. O resto é estrutura em volta dela.

---

## Resumo

**CRS (Cognitive Runtime System)** é uma arquitetura de orquestração multiagente:

1. **O runtime controla** — fases, budgets, DAGs, condições de parada  
2. **Modelos só propõem** — nunca mutam estado durável em silêncio  
3. **Trabalho é um DAG** — dependências explícitas, paralelismo, sem ciclos  
4. **Memória é um grafo** — episódios, lições, evidências, arestas de impacto  
5. **Humanos revisam memória** — não cada keystroke; as *crenças* que a org reutiliza  

**Tese de moat:** modelos de juízo commodity + orquestração proprietária + memória organizacional governada.

Big labs escalam **juízo**.  
Quem opera empresa precisa escalar **processo** e **memória com dono**.

---

## O problema que estamos resolvendo

### 1. Swarm sem estrutura

“Rodar cinco agentes” é fácil. Coordená-los não é.

Sem um plano de trabalho de verdade:

- agentes refazem o mesmo trabalho  
- ninguém sabe o que está bloqueado vs pronto  
- paralelismo colide  
- a execução nunca termina de forma limpa  

O CRS trata trabalho como um **DAG** (grafo dirigido acíclico): tarefas com dependências, fronteiras claras e estado terminal.

### 2. Memória sem dono

Hoje “memória de agente” costuma ser:

- uma context window longa  
- um vector store de texto similar  
- um log de chat tratado como verdade  

Nada disso responde:

- *Quem aprovou isso como conhecimento da empresa?*  
- *Qual evidência sustenta?*  
- *Reutilizar ajudou ou atrapalhou?*  
- *O que supersede isso agora?*  

Sem dono, multiagente escala **rumor** na mesma velocidade em que escala produtividade.

### 3. Human-in-the-loop no lugar errado

Aprovar cada tool call é caro. As pessoas abandonam.

Ignorar humanos é pior: lições ruins viram “é assim que trabalhamos aqui”.

O CRS move o humano para o **ponto de alavancagem**:

- não cada keystroke  
- **memória canônica** e **gates de risco reais**

**Code review** melhora o artefato de hoje.  
**Memory review** melhora a organização de amanhã.

---

## Como o CRS pensa uma execução

```text
objetivo
  → modelo propõe um plano
  → runtime valida e committa um DAG           ← plano vira oficial
  → swarm executa nós ready em paralelo
  → host devolve evidência (testes, logs, diffs)
  → modelo propõe lições / merges / replans
  → verifier e/ou humano revisa memória
  → runtime committa memória cold              ← só o que pode ser reutilizado
  → próximo objetivo recupera memória com budget
  → arestas de impacto: HELPED / HURT / UNUSED
```

### Proposal vs commit (o coração do CRS)

Esta é a distinção mais importante do desenho inteiro.

| Passo | Quem | Significado |
|---|---|---|
| **Propor** | Modelo | “Acho que o plano / lição / merge deve ser X” |
| **Validar** | Runtime | Checa leis: sem ciclos, budget, schema, constraints |
| **Commitar** | Runtime | Torna **estado oficial durável** |
| **Rejeitar** | Runtime / humano | Mantém histórico; não promove lixo |

Analogia com git:

| Git | CRS |
|---|---|
| edições na working tree | proposta do modelo |
| `git commit` | **commit** do runtime (DAG ou memória) |
| history | WorkGraph + MemoryGraph auditáveis |

Se o modelo pode gravar “verdade” sozinho, você não tem runtime cognitivo.  
Tem monólogo com side effects.

---

## Por que grafo para memória (e não só RAG)

RAG responde: *“qual texto parece com esta pergunta?”*  
Orquestração precisa: *“o que já tentamos, o que funcionou, quem vetou o quê, e o que nunca deve acontecer de novo?”*

Isso exige **estrutura, tempo e causalidade**.

| Exemplos de nós | Exemplos de arestas |
|---|---|
| Episode, Evidence, Belief, Lesson | `SUPPORTS`, `DERIVED_FROM` |
| Task, Agent, Artifact | `EXECUTED_BY`, `PRODUCED` |
| HumanDecision | `REVIEWED_BY`, `SUPERSEDES` |
| | `HELPED` / `HURT` / `UNUSED` |

**Logar ≠ aprender.**  
Aprender = mudança persistente que melhora decisões futuras **com avaliação**.

### Três planos de memória

| Plano | O que vive ali | Escrita |
|---|---|---|
| **Hot** | estado desta tarefa | livre sob o runtime |
| **Warm** | lições/crenças candidatas | agentes propõem |
| **Cold** | memória canônica da organização | humano e/ou verificador formal |

Agentes podem ser prolíficos no **warm**.  
**Cold** é escasso, revisado e com escopo (repo, time, ambiente).

---

## Um walkthrough concreto

Imagine o objetivo: *“Corrigir testes de auth quebrados após mudança de schema.”*

1. **Frame** — runtime carrega constraints e lessons cold deste repo.  
2. **Proposta de plano** — modelo sugere: recon → fix migration → fix testes → abrir PR.  
3. **Commit do DAG** — runtime aceita o plano (ou rejeita se houver ciclo / estouro de budget).  
4. **Spawn** — recon roda; ao terminar, as tasks de fix ficam `ready`.  
5. **Observe** — CI falha; evidência é ligada ao nó.  
6. **Critique / replan** — modelo propõe patch no DAG; runtime committa ou rejeita.  
7. **Consolidate** — modelo propõe uma lesson:  
   *“Após mudar schema, rodar migrate + typecheck antes de abrir PR.”*  
8. **Memory review** — humano (ou política com verifier de CI) aceita, edita ou rejeita.  
9. **Na semana seguinte** — um goal parecido recupera essa lesson na view.  
   Se um agente tentar pular, o **runtime pode bloquear** — o modelo não precisa “lembrar de ter cuidado”.

Essa última frase é a diferença de produto:

> Rigor onde código consegue enforçar.  
> Juízo onde a incerteza permanece.

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

### Fórmula

```text
CRS =
  OrchestrationRuntime      # leis, fases, budgets
+ WorkGraph                 # DAG de tarefas commitado
+ MemoryGraph               # memória organizacional causal
+ EvidenceLedger            # o que pode ser verificado
+ JudgmentModels            # LLMs / agentes (só propõem)
+ HumanReviewPlane          # memória cold + gates de risco
```

### Autoridade em uma tabela

| Ator | Pode | Não pode |
|---|---|---|
| **JudgmentModel** | emitir proposals tipadas | mutar estado durável sozinho |
| **Runtime** | validar, rejeitar, commitar; mudar fase | tratar prosa livre como commit |
| **Host** | executar tools; devolver observações | escrever memória cold fora das APIs do runtime |
| **Human** | revisar memória; override mediado; halt | burlar auditoria em silêncio |

Docs mais profundos: [tese](docs/pt-BR/00-tese.md) · [arquitetura](docs/pt-BR/01-arquitetura.md) · [DAG](docs/pt-BR/02-orquestracao-dag.md) · [memory graph](docs/pt-BR/03-memory-graph.md) · [review humano](docs/pt-BR/04-human-memory-review.md) · [contratos](docs/pt-BR/05-contratos-autoridade.md)

---

## Como isso difere de abordagens comuns

| Abordagem | Força | Lacuna que o CRS ataca |
|---|---|---|
| Agente único em chat | UX simples | sem processo/memória durável da org |
| Swarms de prompts multiagente | paralelismo | autoridade fraca; memória-rumor |
| Tools de workflow (grafos de passos) | pipelines confiáveis | pouca governança de crença/memória |
| RAG sobre docs | retrieval barato | similaridade ≠ verdade operacional aprovada |
| Fine-tune de modelo da empresa | estilo/conhecimento nos pesos | opaco, lento, difícil de retract |
| Humano aprova cada ação | sensação de segurança | não escala; alavancagem errada |

O CRS pode sentar **em cima** de APIs de modelo, agent CLIs e engines de workflow existentes.  
O ponto é o **tipo de sistema** (autoridade + memória governada), não uma marca de biblioteca.

---

## O que o CRS **não** é

- Não é AGI, consciência, nem a afirmação de que grafos “criam mentes”  
- Não afirma que modelos menores batem frontier em juízo bruto  
- Não é “LangGraph com outro nome” — o diferencial é **quem pode commitar o quê**  
- Não é aprovação humana em cada tool call — humanos donos da **memória cold** e de risco real  
- Não é teatro de ontologia infinita — comece com memória situada, com escopo e impacto medido  

---

## Princípios (não negociáveis)

1. **Controle e invariantes são código** — não são amostrados pela rede.  
2. **Trabalho e memória são estado mutável** — não só prosa no contexto.  
3. **Crescimento exige experiência avaliada** — logar ≠ aprender.  
4. **Crença durável exige proveniência** — e, quando a política exige, review.  
5. **Budget finito** — swarm sem teto é loop, não inteligência.

---

## Status

Este repositório publica a **proposta de arquitetura** (conceito, contratos, schemas).

| Camada | Status |
|---|---|
| Tese e contratos | v0.3 (este repo) |
| JSON schemas (WorkGraph / MemoryGraph / Proposal) | rascunho em [`schemas/`](schemas/) |
| Runtime de referência (implementação) | planejado |
| Integração com swarms reais (ex.: coding agents) | planejado |

### Mapa da documentação

| Documento | Tema |
|---|---|
| [docs/pt-BR/00-tese.md](docs/pt-BR/00-tese.md) | Por que esta aposta existe |
| [docs/pt-BR/01-arquitetura.md](docs/pt-BR/01-arquitetura.md) | Componentes e fluxo ponta a ponta |
| [docs/pt-BR/02-orquestracao-dag.md](docs/pt-BR/02-orquestracao-dag.md) | DAG de trabalho e commit de plano |
| [docs/pt-BR/03-memory-graph.md](docs/pt-BR/03-memory-graph.md) | Modelo de memória em grafo |
| [docs/pt-BR/04-human-memory-review.md](docs/pt-BR/04-human-memory-review.md) | Humanos como governadores de memória |
| [docs/pt-BR/05-contratos-autoridade.md](docs/pt-BR/05-contratos-autoridade.md) | Regras normativas de autoridade |

Inglês: [README.md](README.md) · [docs/](docs/)

---


## FAQ

### O CRS é produto, biblioteca ou paper?

É uma **proposta de arquitetura**: princípios, contratos, schemas e um formato de referência para sistemas multiagente. Um runtime de referência está planejado; o núcleo normativo é o modelo de autoridade e memória, não o nome de um pacote.

### Em que isso difere de LangGraph / CrewAI / AutoGen / “agent swarms”?

Essas tools são bons **músculos de workflow**. O CRS faz uma pergunta mais afiada: **quem tem permissão de commitar estado durável?**  
Se o modelo grava planos e “verdade” organizacional sem validação, proveniência e regras de promoção, você tem *sintaxe* de orquestração sem *autoridade* cognitiva. O CRS pode ser implementado em cima de vários runtimes; o tipo é o ponto.

### Humanos precisam aprovar cada ação do agente?

Não. Esse padrão não escala.  
No CRS, humanos se concentram em:

1. **Memória cold** — o que a organização pode reutilizar  
2. **Gates de risco reais** — produção, segurança, ações irreversíveis  

O uso de tools no hot path fica sob policy e budgets do runtime.

### Por que grafo em vez de memória vetorial / RAG?

RAG é ótimo para “texto parecido.” Orquestração precisa de perguntas causais: o que tentamos, qual evidência sustentou, quem aprovou, se reutilizar ajudou ou doeu, e o que supersede.  
Vetores ainda podem indexar nós; não substituem **memória com dono, ligações e avaliação**.

### O que é “commit do DAG” em uma frase?

O modelo *sugere* um plano de tarefas; o runtime *torna esse plano estado oficial durável* só depois de checagens de lei (sem ciclos, budget, owners, constraints). Sem commit, plano é conversa.

### Um modelo pequeno funciona com CRS?

Muitas vezes melhor do que se espera **em trabalho com forma de processo**, porque o rigor vive no runtime.  
A qualidade bruta de juízo ainda depende do modelo: o CRS sobe honestidade e disciplina de reuso; não cria gênio de domínio por mágica.

### O que impede o swarm de rodar para sempre?

**Budgets e condições de parada** explícitos: max filhos, rounds de replan, wall-clock, custo, writes candidatas de memória. Budget esgotado → `stop` com razão. Swarm sem teto é loop de custo.

### O que significa “logar ≠ aprender”?

Escrever texto em um store não é aprender.  
O CRS trata aprendizado como: episode + evidence + mudança de estado de memória + impacto posterior (`HELPED` / `HURT` / `UNUSED`). Acúmulo sem avaliação não pode ser reportado como sucesso.

### Isso reivindica AGI?

Não. CRS é uma **arquitetura para trabalho multiagente confiável e memória organizacional**. É um contraponto a “escalar o monólogo,” não uma tese de consciência.

### Por onde começar a ler?

1. Este README (ideia + walkthrough)  
2. [Contratos de autoridade](docs/pt-BR/05-contratos-autoridade.md)  
3. [Memory graph](docs/pt-BR/03-memory-graph.md)  
4. [Human memory review](docs/pt-BR/04-human-memory-review.md)

### Posso implementar só parte do CRS?

Sim. Um caminho parcial útil:

1. Proposal vs commit para planos (WorkGraph)  
2. Budgets e condições de parada  
3. Lessons ligadas a evidência (warm)  
4. Promoção humana para memória cold  

Conformidade completa precisa dos seis componentes da fórmula, mas valor incremental começa no passo 1.


## Contribuir / discussão

Issues e discussões são bem-vindas — especialmente sobre:

- schemas de memória e políticas de promoção cold  
- métricas `HELPED` / `HURT` / `UNUSED` que sobrevivem ao contato com produção  
- papéis reais de Memory Reviewer (quem é dono do runbook?)  
- condições de parada e budgets que não matem trabalho produtivo  

---

## Licença

MIT — veja [LICENSE](LICENSE).

---

## Citação curta

> O modelo é commodity.  
> Orquestração com leis e memória organizacional governada são o moat.
