---
name: orchestrator
description: "Use quando uma tarefa precisa de MAIS DE UMA peça: encadear skills do azvd-toolkit (prompt-forge, graph-engineering, prompt-blocks, self-learning) ou montar um time de sub-agentes (construtor, crítico, juiz, conferidores) com custo controlado. Para um prompt único use prompt-forge; para 'qual skill uso?' use skill-router. Em dúvida de intenção, pergunta em vez de chutar."
trigger: /orchestrator
---

# Orchestrator — encadeia skills e monta o time de agentes

Papel: planejar **como** a tarefa é dividida e **quem** faz cada parte. Não executa a tarefa em si.

## 1. Escolha a rota

| A tarefa é… | Rota |
|---|---|
| um prompt autocontido para outra IA | `prompt-forge` — e pare aí |
| multi-etapa, multi-repo ou N frentes | `graph-engineering` (task graph) → `prompt-forge` (um prompt por ticket) |
| construir algo com qualidade verificada | time: `construtor` + `critico` (+ `juiz` se houver versões a comparar) |
| conferir números, telas ou queries antes de afirmar algo | `conferidor-dados` / `conferidor-tela` / `revisor-query` |
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

Regras de custo:
- **Todo agente tem modelo e esforço definidos** — nunca deixe herdar o modelo da sessão por omissão.
  Os agentes do toolkit já trazem isso no frontmatter; ajuste localmente se precisar (ver README).
- **Busca e exploração ampla** → modelo leve. **Construir** e **conferir** (conferidores, `revisor-query`,
  `critico`) → forte: conferência errada custa mais que o modelo. **Julgar o todo** → o mais forte, com
  menos frequência (ex.: a cada N rodadas, não a cada item).
- **Saída curta** pedida a cada agente (formato fixo, sem colar arquivos inteiros de volta).
- **Teto de rodadas** em todo loop (ex.: 5). Bateu o teto sem passar → pare e reporte.
- Reaproveite: retome o agente que já tem o contexto em vez de abrir outro do zero.
- Cota perto do fim: **não troque modelo no meio da execução**; salve um ponto de retomada (bloco B10)
  e deixe parar limpo.

## 3. O time de agentes (`agents/` do toolkit)

| Agente | Faz | Nunca faz |
|---|---|---|
| `construtor` | implementa um item com escopo fechado | julgar o próprio trabalho; mexer fora do escopo |
| `critico` | nota com régua fixa + defeitos acionáveis | consertar; elogiar sem prova |
| `juiz` | compara versões em A/B cego (ordem trocada) ou julga o conjunto | decidir com uma ordem só |
| `conferidor-dados` | reproduz número/afirmação na fonte, só leitura | escrever na fonte; arredondar a conclusão |
| `conferidor-tela` | confere na tela do sistema, só olhando | clicar em ação que altera dado; abrir abas em paralelo |
| `revisor-query` | plano de execução, índice e resultado antes × depois | aprovar sem medir |

Loop padrão de qualidade: **construtor → crítico → (corrige) → crítico … até passar ou bater o teto**;
o `juiz` entra quando há duas versões ou a cada N itens para ver o conjunto.

## 4. Regras de parada

- Cabe em **um prompt** → `prompt-forge`. Precisa de **N prompts** → task graph primeiro.
- Dúvida de **intenção** → pergunte ao usuário. Dúvida de **conteúdo** → a entrevista do
  `prompt-forge` resolve.
- Afirmação que vai para fora (cliente, fornecedor, relatório) → passa por um conferidor antes.

## Auto-atualização

Encaminhamento novo descoberto (pedido → skill/agente)? Adicione na tabela da seção 1 (protocolo
`self-learning`, regra das 3 verificações).

## Skills relacionadas

- `skill-router` — "qual skill resolve isso?" (só aponta).
- `prompt-forge`, `graph-engineering`, `prompt-blocks`, `self-learning` — as peças que este encadeia.
- `impeccable` (global, só Claude Code) — crítico de UI quando a construção é interface.
