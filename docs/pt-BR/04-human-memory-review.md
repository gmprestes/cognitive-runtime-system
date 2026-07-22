# Human Memory Review

**[English](../04-human-memory-review.md)** · **Português (Brasil)**

## Tese

> Humanos não deveriam revisar *cada resposta* do swarm.  
> Deveriam revisar **o que a organização tem permissão de lembrar e reutilizar**.

Isso concentra a atenção humana no ponto de alavancagem: a **memória cold**.

## Por que isso fortalece a tese CRS

| Abordagem | Efeito |
|---|---|
| Humano aprova cada tool call | lento, caro, abandonado |
| Humano só olha o resultado final | memória podre já foi escrita |
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

1. aceitar / editar / rejeitar propostas warm→cold  
2. restringir escopo (este repo, este time, só staging…)  
3. resolver conflitos entre beliefs  
4. forçar retract quando o mundo mudou  
5. (opcional) assinar gates de alto risco  

## UX mínima (cards, não um grafo assustador)

Revisores não precisam “desenhar grafo”. Veem cards:

```text
PROPOSTA DE MEMÓRIA
Tipo: procedure
Claim: "Após mudanças de schema, rodar migrate + typecheck antes do PR"
Escopo: repo veluacode
Baseado em: 3 episodes (2 ok, 1 falha quando pulado)
Evidência: CI #4821, #4829
Impacto estimado: menos retrabalho em PRs de schema

[ Aceitar ]  [ Aceitar com edições ]  [ Rejeitar ]  [ Adiar ]
```

Por baixo, o runtime grava nós, arestas `REVIEWED_BY` e status cold.

## Estados de decisão humana

| Decisão | Efeito |
|---|---|
| `accept` | promove para cold; lesson/belief `active` |
| `accept_with_edits` | conteúdo/escopo alterados; proveniência mantém origem + emenda |
| `reject` | não promove; motivo fica no grafo |
| `defer` | permanece warm com SLA |
| `retract` | remove do active; `SUPERSEDES` ou `retired` |

## Matriz de autoridade (exemplo)

| Tipo de memória | Só verifier | Humano obrigatório | Dual control |
|---|---|---|---|
| Fato local confirmado por teste | ✅ | | |
| Procedimento de dev | ✅ ou humano | preferível humano na 1ª vez | |
| Constraint de produção | | | ✅ |
| Preferência de produto/estilo | | ✅ | |
| Segurança / privacidade | | | ✅ |

Cada empresa instancia a matriz. O CRS exige que a matriz **exista** e seja enforced no runtime.

## Loop completo de feedback

```text
swarm trabalha
  → evidência
  → proposta de memória (warm)
  → review humano/verificador
  → commit no MemoryGraph (cold)
  → retrieve na próxima orquestração
  → HELPED / HURT / UNUSED
  → supersede / retire / re-review
```

## Métricas que importam para o revisor

- tempo humano / semana em memory review (meta: baixo e estável)  
- taxa de reject (writer ruim vs rigor saudável)  
- taxa de repetição de erros já cobertos por lesson  
- % de memória cold com evidência incompleta (deve tender a 0)  

## Anti-padrões

- transformar review em “chat com o agente de novo”  
- aceitar tudo para limpar a fila  
- rejeitar tudo sem motivo (quebra o writer)  
- um único humano gargalo sem delegação por escopo  
- misturar code review e memory review sem distinção  

## Frase de produto

**Code review** melhora o artefato de hoje.  
**Memory review** melhora a organização de amanhã.
