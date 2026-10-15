---
name: skill-router
description: "Use SOMENTE quando a mensagem do usuário contiver /skill-router (em qualquer posição: início, meio ou fim) ou quando ele perguntar QUAL skill resolve um pedido ('qual skill uso pra isso?', 'tem skill pra X?'). Quebra o pedido em partes e aponta, para cada parte, a skill do azvd-toolkit, a skill instalada no ambiente ou o tipo de artefato do host que resolve — não executa a tarefa."
trigger: /skill-router
---

# Skill Router — qual skill resolve isso

Papel único: dado um pedido, **apontar** o que resolve cada parte dele. Não executa, não encadeia —
encadear é do `orchestrator`. Só roda quando o usuário cita `/skill-router`, em qualquer parte da mensagem; não se dispara sozinha em
outros pedidos.

## Método

0. **Quebre o pedido em partes.** Um pedido raramente é uma coisa só: "pesquisa de preço + proposta
   visual + contra-argumentos" são três partes (pesquisar, montar a peça, argumentar). Uma skill para
   o pedido inteiro costuma ser a mais óbvia, não a certa.
1. **Para cada parte, busque nesta ordem**, e pare só *naquela parte*:
   1. **Skills e agentes do azvd-toolkit** (tabela abaixo).
   2. **Skills instaladas no ambiente.** Use a lista que o host expõe na sessão; se não houver, liste os
      diretórios de skills do host (ex.: `~/.claude/skills/`, `~/.agents/skills/`, a pasta de plugins do
      agente em uso). Procure por **palavra da parte** (pesquisa, web, slide, gráfico, planilha, PDF,
      texto, revisão…) nas `description`, não pela primeira que parecer servir.
   3. **Tipos de artefato do host** (no Claude: Slides, Dashboard, Docs, Design). Não são skills, mas
      costumam ser o melhor caminho quando a parte é "visual", "apresentação", "deck" ou "relatório".
2. **Parte sem nada que encaixe** → diga "sem skill: faço direto" naquela linha. Nunca invente skill.

| Pedido | Aponte para |
|---|---|
| montar/refinar um prompt para uma IA | `prompt-forge` |
| blocos de prompt prontos, ou guardar uma lição como bloco | `prompt-blocks` |
| tarefa que precisa de várias skills ou de sub-agentes (construir + criticar + julgar) | `orchestrator` |
| mapear código/docs em grafo, ou desenhar task graph | `graph-engineering` |
| "aprendi algo", "lembra disso", lição da sessão | `self-learning` |
| bug relatado ("está bugando", "não aparece", "veio errado", "não funciona") | `conferidor-dados` (reproduzir na fonte) → `verificador-previo` (com varredura de **defeito irmão**) → `construtor` → `testador-web` na tela do relato **e nas vizinhas** → `critico` |
| conferir um pedido/plano antes de construir (precisa? já existe? critério de aceite) | agente `verificador-previo` |
| implementar um item com escopo fechado | agente `construtor` |
| avaliar uma entrega contra uma régua (nota + defeitos) | agente `critico` |
| decidir entre versões / julgar o conjunto inteiro | agente `juiz` |
| conferir número ou afirmação contra a fonte (banco, API, planilha) | agente `conferidor-dados` |
| conferir algo numa tela de sistema de terceiros (só olhar) | agente `conferidor-tela` |
| testar a sua aplicação: fluxos, erros, acessibilidade, visual, marcas de IA | agente `testador-web` |
| revisar mudança de query SQL (plano, índice, resultado) | agente `revisor-query` |
| qual modelo e nível de esforço usar em cada agente/papel; vale sub-agente?; reduzir custo de tokens | `orchestrator` §2 → bloco `prompt-blocks/blocks/b11-roteamento-modelos.md` (e `b13-orcamento-tokens.md` para medir) |
| classificar, filtrar ou ranquear muitos itens (≥ 20) gastando pouco | agente `triador` (regra: bloco `b11-roteamento-modelos.md`, pergunta 1) |
| acrescentar item novo num loop de construção que já roda | bloco `b12-gauntlet-loop.md`, seção "Item novo no meio do loop" |

## Regras

- **Dedupe:** se a mesma skill existe no toolkit e no ambiente, use a do toolkit (é a fonte).
- **Instalar algo novo** (`npx skills add`, `git clone`): só com permissão do usuário. Listar é seguro.
- **Resposta em tabela curta**, uma linha por parte: `parte → skill/agente/artefato → por quê`. Nada de
  "nenhuma skill do toolkit cobre isso" sozinho: diga o que resolve fora do toolkit.
- **Chamado junto com uma tarefa** (`faça X /skill-router`): entregue a tabela em poucas linhas e siga a
  tarefa usando o que apontou. O router não trava a tarefa nem a troca por uma escolha só.

## Skills relacionadas

- `orchestrator` — quando a resposta é "precisa de mais de uma peça".
- `self-learning` — rota nova descoberta (pedido → skill) entra na tabela acima.
