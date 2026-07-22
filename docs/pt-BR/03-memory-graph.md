# Memory Graph

**[English](../03-memory-graph.md)** · **Português (Brasil)**

## Ideia

Memória organizacional não é:

- só a context window  
- só um vector store de “textos parecidos”  
- só o log do chat  

É um **grafo** do que a organização:

- tentou (episodes)  
- observou (evidence)  
- acredita (beliefs)  
- aprendeu a fazer (lessons)  
- decidiu com humanos (reviews)  
- mediu como impacto (`HELPED` / `HURT` / `UNUSED`)

## Por que um grafo

Orquestração precisa de perguntas que similaridade responde mal:

- O que já tentamos para *este* padrão de falha?  
- Quem vetou este procedimento e por quê?  
- Esta lição ainda ajuda ou dói?  
- Qual evidência sustenta esta crença?  
- O que supersede o quê?

Isso é **estrutura + tempo + causalidade**.

## Tipos de nó (MVP)

| Tipo | Descrição |
|---|---|
| `Episode` | uma execução (run, tentativa de PR, heartbeat) |
| `Task` / `Goal` | objetivo de negócio ou issue |
| `Agent` | papel ou instância |
| `Artifact` | diff, relatório, binário, doc |
| `Evidence` | observação verificável |
| `Belief` | afirmação revisável |
| `Lesson` | procedimento / estratégia / constraint reutilizável |
| `Entity` | repo, serviço, API, cliente, política |
| `HumanDecision` | approve / reject / amend |

## Tipos de aresta (MVP)

| Aresta | Significado |
|---|---|
| `SUPPORTS` | evidência sustenta belief/lesson |
| `CONTRADICTS` | evidência enfraquece |
| `DERIVED_FROM` | lesson/belief ← episodes |
| `EXECUTED_BY` | episode → agent |
| `PRODUCED` | episode → artifact |
| `BLOCKS` / `ENABLES` | relações de trabalho (podem espelhar o DAG) |
| `SUPERSEDES` | versão nova substitui a antiga |
| `REVIEWED_BY` | decisão humana → nó de memória |
| `APPLIED_IN` | lesson usada em episode posterior |
| `HELPED` / `HURT` / `UNUSED` | impacto medido |

## Belief vs Lesson

| | Belief | Lesson |
|---|---|---|
| Natureza | “o que é verdade *aqui*” | “como agir *aqui*” |
| Exemplo | “auth vive em `src/auth`” | “após mudar schema, rodar migrate + typecheck” |
| Risco se errada | fato operacional falso | procedimento ruim reaplicado |
| Preferência de promoção | evidência forte | evidência + outcome |

Beliefs de mundo aberto devem ser tratadas com ceticismo; o grafo brilha em **verdade situada** (este repo, este produto, este processo).

## Planos de memória

| Plano | Volatilidade | Escrita |
|---|---|---|
| **Hot** | episódica | livre sob o runtime |
| **Warm** | candidata | agentes propõem |
| **Cold** | canônica | review / dual control |

Promover warm → cold sem porta (verifier ou humano) é **non-conformant** no CRS v0.3 para lessons de procedimento e constraints.

## Write policy (mínimo)

Uma lesson candidata **deve ser rejeitada** a menos que:

1. tenha `DERIVED_FROM` ≥ 1 episode com outcome, **e**  
2. tenha evidência ligada, **e**  
3. passe por pelo menos uma porta:

| Porta | Uso |
|---|---|
| Verifier | testes/CI/assert programáticos |
| Human memory review | preferência, política, estilo, risco organizacional |
| Dual control | produção / segurança / compliance |

## Retrieve policy

Ao montar a view:

- priorizar lessons com alto `HELPED` no escopo  
- incluir `HURT` recentes como **anti-padrões**  
- degradar `UNUSED` prolongado  
- respeitar escopo (repo, time, ambiente)  
- orçar tokens/nós (memória sem budget vira dump)

## Esquecimento (obrigatório no desenho)

Sem retirement, o grafo vira aterro.

Políticas de exemplo:

- `UNUSED` por N períodos → `retired` automático  
- `HURT` acima do limiar → `contested` + review  
- `SUPERSEDES` marca o ancestral como não-default  

## O que não fazer

- Guardar monólogos longos do LLM como lessons  
- Aceitar “o agente disse” sem evidência  
- Memória global sem escopo  
- Confundir recall por embedding com aprendizado  

**Logar ≠ aprender.**  
Aprender = mudança persistente que melhora decisões futuras *com avaliação*.
