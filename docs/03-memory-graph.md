# Memory Graph

## Ideia

Memória organizacional não é:

- só o context window  
- só um vector store de “textos parecidos”  
- só o log do chat  

É um **grafo** do que a organização:

- tentou (episodes)  
- observou (evidence)  
- acredita (beliefs)  
- aprendeu a fazer (lessons)  
- decidiu com humanos (reviews)  
- mediu como impacto (`HELPED` / `HURT` / `UNUSED`)

## Por que grafo

Orquestração precisa de perguntas que similaridade não responde bem:

- O que já tentamos para *este* tipo de falha?  
- Quem vetou este procedimento e por quê?  
- Esta lição ainda ajuda ou já dói?  
- Qual evidência sustenta esta crença?  
- O que supersede o que?

Isso é **estrutura + tempo + causalidade**.

## Tipos de nó (MVP)

| Tipo | Descrição |
|---|---|
| `Episode` | uma execução (run, PR attempt, heartbeat) |
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
| `SUPPORTS` | evidence sustenta belief/lesson |
| `CONTRADICTS` | evidence enfraquece |
| `DERIVED_FROM` | lesson/belief ← episodes |
| `EXECUTED_BY` | episode → agent |
| `PRODUCED` | episode → artifact |
| `BLOCKS` / `ENABLES` | relações de trabalho (podem espelhar DAG) |
| `SUPERSEDES` | versão nova substitui antiga |
| `REVIEWED_BY` | human decision → nó de memória |
| `APPLIED_IN` | lesson usada em episode posterior |
| `HELPED` / `HURT` / `UNUSED` | impacto medido |

## Belief vs Lesson

| | Belief | Lesson |
|---|---|---|
| Natureza | “o que é verdade *aqui*” | “como agir *aqui*” |
| Exemplo | “auth vive em `src/auth`” | “após schema change, rodar migrate + typecheck” |
| Risco se errada | fato operacional falso | procedimento ruim reaplicado |
| Preferência de promoção | evidência forte | evidência + outcome |

Beliefs de mundo aberto demais devem ser tratadas com ceticismo; o grafo brilha em **verdade situada** (deste repo, deste produto, deste processo).

## Planos de memória

| Plano | Volatilidade | Escrita |
|---|---|---|
| **Hot** | episódica | livre sob runtime |
| **Warm** | candidata | agents propõem |
| **Cold** | canônica | review / dual control |

Promover warm → cold sem porta (verifier ou humano) é **non-conformant** no CRS v0.3 para lessons de procedimento e constraints.

## Write policy (mínimo)

Uma lesson candidata **deve ser rejeitada** salvo se:

1. tem `DERIVED_FROM` ≥ 1 episode com outcome, **e**  
2. tem evidência ligada, **e**  
3. passa por pelo menos uma porta:

| Porta | Uso |
|---|---|
| Verifier | testes/CI/assert programático |
| Human memory review | preferência, política, estilo, risco organizacional |
| Dual control | produção / segurança / compliance |

## Retrieve policy

Na montagem da view:

- priorizar lessons com `HELPED` alto no escopo  
- incluir `HURT` recentes como **anti-padrões**  
- degradar `UNUSED` prolongado  
- respeitar escopo (repo, time, ambiente)  
- orçar tokens/nós (memória sem orçamento vira dump)

## Esquecimento (obrigatório no desenho)

Sem retirement, o grafo vira entulho.

Políticas exemplo:

- `UNUSED` por N períodos → `retired` automático  
- `HURT` acima de limiar → `contested` + review  
- `SUPERSEDES` marca ancestral como não-default  

## O que não fazer

- Gravar monólogo longo do LLM como lesson  
- Aceitar “o agente disse” sem evidência  
- Memória global sem escopo  
- Confundir embedding recall com aprendizado  

**Gravar ≠ aprender.**  
Aprender = mudança persistente que melhora decisões futuras *com avaliação*.
