---
name: prompt-blocks
description: "Use para colher lições em blocos de prompt comprovados. NÃO é biblioteca pré-cheia: é protocolo — filtra se a lição é útil para o usuário, pede permissão antes de criar, e grava cada bloco em arquivo .md no estilo OKF. Alimenta prompt-forge e qualquer ticket."
trigger: /prompt-blocks
---

# Prompt Blocks — protocolo de colheita em blocos

Esta skill **não vem cheia de blocos**: ela ensina o processo de **decidir, criar e manter**
blocos de prompt validados em campo — um bloco = um arquivo `.md` em `blocks/`, no **estilo OKF**
(frontmatter v0.2), com a lição que o originou. O conteúdo útil para VOCÊ cresce aqui com o uso.

## Filtro de utilidade (o que merece virar bloco)

Antes de criar QUALQUER bloco, rode o filtro. **Vai** exige **F1 E F3** + pelo menos um de **F2/F4**:

| # | Pergunta | Resposta boa (vai) | Resposta ruim (recusa) |
|---|---|---|---|
| **F1 · Recorrência** | Isso vai acontecer de novo? | sim, com certeza / já aconteceu 2x | cenário único, 1x só |
| **F2 · Custo** | O que custa repetir? | horas de retrabalho, risco de negócio | 5 minutos, sem consequência |
| **F3 · Evidência** | Tem check que passou? | teste/probe/incidente com número | "pareceu funcionar" |
| **F4 · Especificidade** | É do JEITO do usuário? | regra que ele corrigiu, padrão dele | genérico que qualquer guia ensina |
| **F5 · Tamanho** | Cabe em 1 linha? | procedimento multi-passo → bloco | 1 fato → memória (OKF `log.md`) |

**Decisão:** vai → cria (com permissão). Recusa → **registra a recusa no OKF `log.md` como lição
negativa** (beco descartado + motivo) ou descarta — a recusa com motivo também vale (evita
redescobrir o beco).

## Padrões comprovados (pesquisa 2026-08: Prompt-Engineering-Guide, prompts.chat)

Ao avaliar uma lição, confira se ela toca um destes padrões — **não duplique** o que já está coberto:

| Padrão | Quando usar | Já coberto por |
|---|---|---|
| **Few-shot** (exemplo entrada→saída no prompt) | saída ambígua; mostrar o formato esperado | prompt-forge (regra "peça exemplo") |
| **Delimitadores** (separar contexto/instrução/saída) | prompt longo; evitar mistura de seções | — (novo) |
| **Chain-of-thought** ("pense antes de responder") | análise/diagnóstico; raciocínio antes da resposta | B3/B7 parcial |
| **Formato de saída explícito** (JSON/tabela/estrutura) | qualquer entrega | B7 (resumo objetivo) |
| **Papel/persona** ("você é revisor sênior") | foco e critério de julgamento | padrão analyzer/reviewer OEP |
| **Prompt chaining** (encadear prompts) | tarefa complexa → dividir | task graph (um prompt por ticket) |
| **Auto-crítica** (revisar o próprio output) | qualidade; revisor separado | reviewer gate OEP |
| **ReAct** (raciocinar + agir) | agentes com ferramentas | nativo nos agentes |

## Estilo do usuário (detecte ANTES de colher)

Cada usuário armazena conhecimento do seu jeito. **No primeiro uso (ou quando for criar algo),
descubra o estilo** — pergunte ou detecte pelos arquivos existentes:

| Estilo | Sinais | Onde guardar fatos de 1 linha / lições negativas |
|---|---|---|
| **OKF** (bundle com índice) | `index.md` + `log.md` + `okf_check.py` | `log.md` do bundle, concepts com frontmatter v0.2 |
| **MEMORY.md** / notas no repo | `MEMORY.md` no repo | `MEMORY.md` (append) |
| **Instruções permanentes** | `CLAUDE.md` / `AGENTS.md` | bloco novo no arquivo de instruções |
| **Auto-memory do agente** | `~/.claude/projects/.../memory/` | memória automática do agente |
| **Nenhum** | nada encontrado | proponha o mais simples (MEMORY.md) |

Regras:
- **Blocos novos sempre em `blocks/` desta skill** (são portáveis entre agentes e estilos) — o
  frontmatter usa o estilo do usuário; **default: OKF v0.2** (mostrado abaixo).
- Fatos de 1 linha e recusas do filtro vão para o local do estilo detectado — **nunca assuma OKF
  sem confirmar**.

## Criação no estilo do usuário (default: OKF)

Cada bloco = **1 arquivo** em `blocks/` com frontmatter OKF v0.2:

```yaml
---
type: PromptBlock
title: B11 · <nome do bloco>
description: <1 linha: o que o bloco faz>
tags: [prompt, <tema>]
status: active
generated:
  by: <quem criou>
  at: <ISO-8601>
stale_after: <data de revisão ou "nunca">
sources:
  - <origem: incidente / repo / usuário>
---
```

Corpo: **texto pronto para colar** (com `<placeholders>` marcados) + **Quando** usar + **Outcome**
(a lição que originou). O catálogo abaixo ganha 1 linha nova a cada bloco criado.

## Gate de permissão (sempre)

- **Arquivo NOVO** (bloco novo): **AVISE o usuário e PEÇA permissão ANTES de criar** — proposta de
  1-2 linhas (a lição, onde vai ficar, por quê, e qual padrão da pesquisa ela toca). Só cria com OK.
- **Arquivo EXISTENTE** (atualizar texto/quando/outcome): automático, sem pedir.

## Catálogo (blocos existentes)

| Bloco | Arquivo | Quando usar |
|---|---|---|
| B1 · LEIA PRIMEIRO | `blocks/b1-leia-primeiro.md` | contexto obrigatório antes de agir |
| B2 · PARE E REPORTE | `blocks/b2-pare-e-reporte.md` | risco de escopo, dado/token ausente |
| B3 · O QUE JÁ FOI VERIFICADO | `blocks/b3-ja-verificado.md` | rodar em cima de investigação feita |
| B4 · Regras karpathy | `blocks/b4-karpathy.md` | execução cirúrgica de builders |
| B5 · Teste de decisão | `blocks/b5-teste-decisao.md` | agente autônomo sem volta ao usuário |
| B6 · Contrato de memória (OKF) | `blocks/b6-contrato-okf.md` | repo com memória coletiva |
| B7 · Resumo objetivo | `blocks/b7-resumo-objetivo.md` | relatório de retorno |
| B8 · Entrevista A/B | `blocks/b8-entrevista-ab.md` | skills interativas |
| B9 · Memória do projeto | `blocks/b9-memoria-projeto.md` | agentes recorrentes (docs nunca stale) |
| B10 · Checkpoint | `blocks/b10-checkpoint.md` | runs longos com retomada |
| KG: blocos `/kg-*` (tutor, scope, schema, extract, relations, events, fuse, eval, rag) | `graph-engineering/references/workflows.md` | prompts prontos para colar do domínio de knowledge graph |

## Como usar

1. Identifique a seção do prompt que precisa (contexto / método / meta / saída).
2. Abra o arquivo do bloco no catálogo, cole o texto + preencha os `<placeholders>` com o domínio real.
3. Registre no final do bloco usado a lição nova que você aprendeu — a biblioteca cresce com outcome, não com teoria.

## Skills relacionadas

- `prompt-forge` — entrevista que monta o prompt e compõe estes blocos (lê o catálogo).
- `orchestrator` — roteador do toolkit.
- `self-learning` — protocolo de colheita que alimenta esta biblioteca (usa este filtro).
- `graph-engineering` — tickets de task graph usam os blocos B2-B7.
