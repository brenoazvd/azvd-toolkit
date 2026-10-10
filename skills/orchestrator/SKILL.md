---
name: orchestrator
description: "Use quando uma tarefa precisa de MAIS DE UMA peça: encadear skills do azvd-toolkit (prompt-forge, graph-engineering, prompt-blocks, self-learning) ou montar um time de sub-agentes (construtor, crítico, juiz, conferidores) com custo controlado (qual modelo e nível de esforço em cada papel, se vale sub-agente, como medir e baixar o gasto de tokens). Para um prompt único use prompt-forge; para 'qual skill uso?' o usuário chama /skill-router (manual). Em dúvida de intenção, pergunta em vez de chutar."
trigger: /orchestrator
---

# Orchestrator — encadeia skills e monta o time de agentes

Papel: planejar **como** a tarefa é dividida e **quem** faz cada parte. Não executa a tarefa em si.

## 1. Escolha a rota

| A tarefa é… | Rota |
|---|---|
| um prompt autocontido para outra IA | `prompt-forge` — e pare aí |
| multi-etapa, multi-repo ou N frentes | `graph-engineering` (task graph) → `prompt-forge` (um prompt por ticket) |
| construir algo com qualidade verificada | time: `verificador-previo` + `construtor` + `critico` (+ `juiz` se houver versões a comparar) |
| conferir números, telas de sistema de terceiros (só olhar) ou queries antes de afirmar algo | `conferidor-dados` / `conferidor-tela` / `revisor-query` |
| testar a sua aplicação usando de verdade (fluxos, erros, acessibilidade, visual, marcas de IA) | `testador-web` |
| registrar uma lição da sessão | `self-learning` |
| intenção ambígua | **pergunte** — uma pergunta, A/B com recomendação |

Saída de uma peça é entrada da próxima. Nunca refaça o que a anterior já entregou.

## 2. Dimensione antes de disparar (custo)

Multi-agente custa muito mais que um agente só — use quando o valor justifica.

| Complexidade | Time |
|---|---|
| busca ou conferência simples | **1 agente**, poucas chamadas |
| comparar 2-4 opções, ou 2-4 itens independentes | **2-4 agentes** em paralelo |
| construção grande (vários itens + crítica) | construtores em paralelo **com teto** (ex.: 3) + 1 crítico por item + juiz periódico |

Regras de custo (run longo com loop: cole o bloco B13 · Orçamento e desperdício de tokens no prompt do
orquestrador — custo = tentativas × turnos × paralelismo; o limite de cota reinicia todo agente que estava no meio):
- **Todo agente tem modelo e esforço definidos** — nunca deixe herdar o modelo da sessão por omissão.
  Os agentes do toolkit já trazem isso no frontmatter; ajuste localmente se precisar (ver README).
- **Para cada papel, responda as 5 perguntas do bloco B11 · Roteamento por tarefa**
  (`skills/prompt-blocks/blocks/b11-roteamento-modelos.md`) antes de disparar:
  precisa de LLM ou basta script / modelo de decisão (triagem em lote, a partir de ~20 itens)? vale
  sub-agente? qual categoria (leve/forte/mais forte)? qual nível de esforço? como testar antes de baixar?
  Resumo: **mecânico e busca ampla** → leve (nível alto se a tarefa é longa ou tem regra estrita);
  **construir** e **conferir** (conferidores, `revisor-query`, `testador-web`, `critico`) → forte: conferência
  errada custa mais que o modelo; **julgar o todo** → o mais forte, com menos frequência (a cada N rodadas).
- **Meça o gasto por papel** (bloco B13, `skills/prompt-blocks/blocks/b13-orcamento-tokens.md`) e baixe nível ou modelo só depois de uma volta de teste que não piore
  a nota nem traga falha nova.
- **Saída curta** pedida a cada agente (formato fixo, sem colar arquivos inteiros de volta).
- **Teto de rodadas** em todo loop (ex.: 5). Bateu o teto sem passar → pare e reporte. Exceção: o Gauntlet
  Loop (bloco B12) não tem teto, quem para é o humano; lá o limite é o orçamento (bloco B13: item sem progresso
  vai para o fim da fila, e o run para limpo perto do fim da cota).
- Reaproveite: retome o agente que já tem o contexto em vez de abrir outro do zero.
- Cota perto do fim: **não troque modelo no meio da execução**; salve um ponto de retomada (bloco B10)
  e deixe parar limpo.

## 3. O time de agentes (`agents/` do toolkit)

| Agente | Faz | Nunca faz |
|---|---|---|
| `verificador-previo` | antes de construir: precisa existir, já existe, premissas, critério de aceite | construir; decidir gosto; escolher entre leituras ambíguas |
| `construtor` | implementa um item com escopo fechado | julgar o próprio trabalho; mexer fora do escopo |
| `critico` | nota com régua fixa + defeitos acionáveis | consertar; elogiar sem prova |
| `juiz` | compara versões em A/B cego (ordem trocada) ou julga o conjunto; no A/B de cada volta, chame com modelo forte (B11) | decidir com uma ordem só |
| `conferidor-dados` | reproduz número/afirmação na fonte, só leitura | escrever na fonte; arredondar a conclusão |
| `conferidor-tela` | confere na tela do sistema, só olhando | clicar em ação que altera dado; abrir abas em paralelo |
| `testador-web` | usa a sua aplicação de verdade: fluxos, erros, dados, larguras, acessibilidade, visual e marcas de IA, com prova | gravar em produção; consertar código; dar nota; rodar 2 no mesmo navegador em paralelo |
| `revisor-query` | plano de execução, índice e resultado antes × depois | aprovar sem medir |
| `triador` | classifica muitos itens (≥ 20) contra rótulos fechados, com máscara e lista de revisão; usa o modelo de decisão do usuário se houver | escrever o texto final; mandar dado pessoal sem máscara; decidir caso de confiança baixa |

Loop padrão de qualidade: **verificador-previo (item não trivial) → construtor → testador-web (se for web) → crítico → (corrige) → testador-web → crítico … até passar ou bater o teto**
(INCONCLUSIVO do testador não conta como passou; se for falta de login ou ambiente, pergunte ao usuário
em vez de girar outra rodada);
o `juiz` entra quando há duas versões ou a cada N itens para ver o conjunto.
Item novo entrando num loop que já roda: antes do construtor, faça o arranque do bloco B12
(`skills/prompt-blocks/blocks/b12-gauntlet-loop.md`, seção "Item novo no meio do loop").

## 4. Regras de parada

- Cabe em **um prompt** → `prompt-forge`. Precisa de **N prompts** → task graph primeiro.
- Dúvida de **intenção** → pergunte ao usuário. Dúvida de **conteúdo** → a entrevista do
  `prompt-forge` resolve.
- Afirmação que vai para fora (cliente, fornecedor, relatório) → passa por um conferidor antes.

## Auto-atualização

Encaminhamento novo descoberto (pedido → skill/agente)? Adicione na tabela da seção 1 (protocolo
`self-learning`, regra das 3 verificações).

## Skills relacionadas

- `skill-router` — "qual skill resolve isso?" (só aponta; manual, o usuário chama /skill-router).
- `prompt-forge`, `graph-engineering`, `prompt-blocks`, `self-learning` — as peças que este encadeia.
- `impeccable` (global, só Claude Code) — crítico de UI quando a construção é interface.
