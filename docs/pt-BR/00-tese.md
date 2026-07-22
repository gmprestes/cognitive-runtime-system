# Tese CRS

**[English](../00-thesis.md)** · **Português (Brasil)**

## Em uma frase

**O ciclo de orquestração e o ciclo de memória devem ser um programa com leis; o modelo só julga sob incerteza; o que a organização reutiliza precisa de proveniência e, quando importa, de review humano.**

## O problema

Sistemas multiagente hoje costumam empilhar:

- prompts longos  
- tools  
- “memória” como logs ou embeddings  
- esperança de que o modelo se comporte  

Isso falha de formas previsíveis:

1. **Erros se repetem** — nada governa o que virou lição.  
2. **Rumores viram política** — um agente “aprende” errado e contamina o swarm.  
3. **Sem accountability** — ninguém é dono da crença operacional.  
4. **Custo sem alavancagem** — mais tokens, mais filhos, menos estrutura.

A indústria investe pesado em **juízo** (modelos maiores).  
Subinveste em **processo** e **memória com dono** como *tipo de sistema*.

## A aposta

| Camada | Commodity / mercado | Diferenciação CRS |
|---|---|---|
| Juízo (LLM) | sim — melhora sozinho com o tempo | pluggable |
| Músculos (tools, CI, browsers) | sim | host |
| **Orquestração com leis** | subinvestida | **runtime** |
| **Memória organizacional governada** | subinvestida | **Memory Graph + review** |

Frase de posicionamento:

> Trilhões de parâmetros melhoram o que o sistema *pode julgar*.  
> CRS melhora *como* o sistema é *obrigado* a trabalhar e a lembrar.

## Dois andares (não misture no slogan)

| Andar | Nome | Função |
|---|---|---|
| 1 | Deliberar / orquestrar | fazer a tarefa *agora* com estrutura |
| 2 | Tornar-se | o que **permanece** para a próxima vez (memória) |

- Andar 1 sem 2 = time competente com amnésia.  
- Andar 2 sem 1 = diário caótico.  
- Os dois = organização que executa e aprende com critério.

## Humanos no ponto de alavancagem

O erro clássico de “human in the loop” é pedir confirmação em toda ação barata.

O CRS move o humano para:

- **memória cold** — o que pode ser reaplicado  
- **gates de risco** — produção, segurança, compliance, ações irreversíveis  

O swarm continua rápido; a empresa não automatiza o próprio esquecimento nem o próprio moinho de rumores.

## Como o sucesso parece

Não: “o agente conversou bem”.  
Sim:

- menos retrabalho no mesmo padrão de erro  
- lições rejeitadas quando são lixo (sinal de writer honesto)  
- arestas `HELPED` / `HURT` com sinal real  
- auditabilidade: por que o sistema escolheu X ontem  

## Não-objetivos

- AGI / consciência  
- substituir modelos frontier  
- grafo por grafo (ontologia infinita)  
- humano em cada keystroke  

## Pitch interno (30 segundos)

Temos agentes. Eles são rápidos e baratos de escalar.  
O que quebra é coordenação e memória.  
CRS trata orquestração como runtime (DAG, budgets, autoridade) e memória como grafo com review humano.  
O modelo propõe; o sistema committa; a empresa assina o que lembra.
