# Human Memory Review

## Tese

> Humanos não deveriam revisar *cada resposta* do swarm.  
> Deveriam revisar **o que a organização tem permissão de lembrar e reutilizar**.

Isso concentra atenção humana no ponto de alavancagem: o **cold memory**.

## Por que isso fortalece a tese CRS

| Abordagem | Efeito |
|---|---|
| Humano aprova cada tool call | lento, caro, abandono |
| Humano só olha o resultado final | memória podre já foi gravada |
| Humano revisa **memória candidata** | swarm rápido + aprendizado com dono |

O swarm continua autônomo no hot path.  
A empresa governa o que vira capital operacional.

## Papel: Memory Reviewer

Não é “prompt engineer”.  
É mais perto de:

- editor de runbook vivo  
- curador de verdade operacional  
- auditor de lições de postmortem  

Responsabilidades:

1. aceitar / editar / rejeitar proposals warm→cold  
2. restringir escopo (este repo, este time, só staging…)  
3. resolver conflitos entre beliefs  
4. forçar retract quando o mundo mudou  
5. (opcional) assinar gates de alto risco  

## UX mínima (card, não grafo assustador)

O revisor não precisa “desenhar grafo”. Ele vê cards:

```text
PROPOSTA DE MEMÓRIA
Tipo: procedure
Claim: "Após mudar schema, rodar migrate + typecheck antes do PR"
Escopo: repo veluacode
Baseado em: 3 episodes (2 ok, 1 falha se pulado)
Evidência: CI #4821, #4829
Impacto estimado: menos retrabalho em PRs de schema

[ Aceitar ]  [ Aceitar com edição ]  [ Rejeitar ]  [ Adiar ]
```

Por baixo, o runtime grava nós, arestas `REVIEWED_BY`, e status cold.

## Estados de decisão humana

| Decisão | Efeito |
|---|---|
| `accept` | promove cold; lesson/belief `active` |
| `accept_with_edits` | conteúdo/escopo alterados; provenance mantém origem + emenda |
| `reject` | não promove; motivo fica no grafo |
| `defer` | permanece warm com SLA |
| `retract` | tira do active; `SUPERSEDES` ou `retired` |

## Matriz de autoridade (exemplo)

| Tipo de memória | Verifier only | Human required | Dual control |
|---|---|---|---|
| Fato local confirmado por teste | ✅ | | |
| Procedimento de dev | ✅ ou human | preferível human na 1ª vez | |
| Constraint de produção | | | ✅ |
| Preferência de estilo/produto | | ✅ | |
| Segurança / privacidade | | | ✅ |

Cada empresa instancia a matriz. O CRS exige que a matriz **exista** e seja enforced no runtime.

## Feedback loop completo

```text
swarm trabalha
  → evidence
  → memory proposal (warm)
  → human/verifier review
  → MemoryGraph commit (cold)
  → retrieve na próxima orquestração
  → HELPED / HURT / UNUSED
  → supersede / retire / re-review
```

## Métricas que importam para o revisor

- tempo humano / semana em memory review (meta: baixo e estável)  
- % reject (writer ruim vs rigor saudável)  
- taxa de repetição de erros já cobertos por lesson  
- % de cold memory com evidence incompleta (deve tender a 0)  

## Anti-padrões

- transformar review em “chat com o agente de novo”  
- aceitar tudo para limpar a fila  
- rejeitar tudo sem motivo (quebra o writer)  
- um único humano gargalo sem delegação por escopo  
- misturar code review com memory review sem distinção  

## Frase de produto

**Code review** melhora o artefato de hoje.  
**Memory review** melhora a organização de amanhã.
