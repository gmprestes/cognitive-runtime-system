# Tese CRS

## Em uma frase

**O ciclo de orquestração e de memória deve ser um programa com leis; o modelo só julga sob incerteza; o que a organização reutiliza precisa de proveniência e, quando importa, de review humano.**

## O problema

Sistemas multi-agente atuais costumam empilhar:

- prompts longos  
- tools  
- “memória” como log ou embedding  
- esperança de que o modelo se comporte  

Isso falha de formas previsíveis:

1. **Erro se repete** — nada governa o que virou lição.  
2. **Rumores viram política** — um agente “aprende” errado e contamina o swarm.  
3. **Sem accountability** — ninguém assina a crença operacional.  
4. **Custo sem alavancagem** — mais tokens, mais filhos, menos estrutura.

A indústria investe pesado em **juízo** (modelos maiores).  
Investe pouco em **processo** e **memória com dono** como *tipo* de sistema.

## A aposta

| Camada | Commodity / mercado | Diferenciação CRS |
|---|---|---|
| Juízo (LLM) | sim — melhora sozinho com o tempo | pluggable |
| Músculos (tools, CI, browsers) | sim | host |
| **Orquestração com leis** | sub-investida | **runtime** |
| **Memória organizacional governada** | sub-investida | **Memory Graph + review** |

Frase de posicionamento:

> Trilhões de parâmetros melhoram o que o sistema *pode julgar*.  
> CRS melhora o *como* o sistema é *obrigado* a trabalhar e a lembrar.

## Dois andares (não misturar no slogan)

| Andar | Nome | Função |
|---|---|---|
| 1 | Deliberar / orquestrar | fazer a tarefa *agora* com estrutura |
| 2 | Tornar-se | o que **fica** para a próxima vez (memória) |

- Andar 1 sem 2 = time competente com amnésia.  
- Andar 2 sem 1 = diário caótico.  
- Os dois = organização que executa e aprende com critério.

## Humanos no ponto de alavancagem

O erro clássico de “human in the loop” é pedir confirmação a cada ação barata.

CRS move o humano para:

- **cold memory** — o que pode ser reaplicado  
- **gates de risco** — produção, segurança, compliance, irreversível  

Assim o swarm permanece rápido, e a empresa não automatiza o próprio esquecimento nem o próprio rumor.

## O que sucesso parece

Não: “o agente conversou bem”.  
Sim:

- menos retrabalho no mesmo padrão de erro  
- lessons rejeitadas quando são lixo (writer honesto)  
- arestas `HELPED` / `HURT` com sinal real  
- auditoria: por que o sistema escolheu X ontem  

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
