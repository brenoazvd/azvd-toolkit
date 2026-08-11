---
name: prompt-blocks
description: "Use para compor/refinar prompts com blocos comprovados (LEIA PRIMEIRO, PARE E REPORTE, karpathy, teste de decisão, memória do projeto, checkpoint), cada um em arquivo próprio com a lição de incidente. Alimenta prompt-forge e qualquer ticket."
trigger: /prompt-blocks
---

# Prompt Blocks — blocos comprovados com outcome

Biblioteca de blocos de prompt **validados em campo**, cada um com: texto pronto para colar +
quando usar + a lição que o originou. **Um bloco = um arquivo** em `blocks/` (a SKILL.md é só o
catálogo e nunca cresce). Compor um prompt = escolher blocos + preencher as lacunas com o domínio
real. **Nunca colar bloco sem preencher os placeholders** — bloco com lacuna vira instrução vazia.

## Catálogo

| Bloco | Arquivo | Quando usar |
|---|---|---|
| B1 · LEIA PRIMEIRO | `blocks/b1-leia-primeiro.md` | agente precisa de contexto antes de agir |
| B2 · PARE E REPORTE | `blocks/b2-pare-e-reporte.md` | risco de escopo, dado/token ausente, decisão de negócio |
| B3 · O QUE JÁ FOI VERIFICADO | `blocks/b3-ja-verificado.md` | agente roda em cima de investigação já feita |
| B4 · Regras karpathy | `blocks/b4-karpathy.md` | todo builder (execução cirúrgica) |
| B5 · Teste de decisão | `blocks/b5-teste-decisao.md` | agente autônomo que não pode voltar ao usuário |
| B6 · Contrato de memória (OKF) | `blocks/b6-contrato-okf.md` | repo com memória coletiva versionada |
| B7 · Resumo objetivo | `blocks/b7-resumo-objetivo.md` | todo agente que reporta de volta |
| B8 · Entrevista A/B | `blocks/b8-entrevista-ab.md` | skills interativas (prompt-forge) |
| B9 · Memória do projeto | `blocks/b9-memoria-projeto.md` | projetos com agentes recorrentes (docs nunca ficam stale) |
| B10 · Checkpoint | `blocks/b10-checkpoint.md` | runs longos que podem ser cortados (retomada) |

## Como usar

1. Identifique a seção do prompt que precisa (contexto / método / meta / saída).
2. Abra o arquivo do bloco no catálogo, cole o texto + preencha os `<placeholders>` com o domínio real.
3. Registre no final do bloco usado a lição nova que você aprendeu — a biblioteca cresce com outcome, não com teoria.

## Auto-atualização

Descobriu uma lição nova durante o uso (correção do usuário, incidente, padrão que se repetiu)?

1. **Arquivo NOVO** (bloco novo B11+): **AVISE o usuário e PEÇA permissão ANTES de criar** —
   proposta de 1-2 linhas (o que a lição é, onde vai ficar, por quê). Só cria com OK do usuário.
   Depois de criar: 1 linha nova no catálogo acima (protocolo `self-learning`, regra de promoção
   de 3 verificações — check que passou + padrão de falha nomeado + beco descartado).
2. **Arquivo EXISTENTE** (atualizar texto/quando-usar/outcome): automático, sem pedir.

O prompt-forge passa a compor com o bloco novo automaticamente (lê o catálogo).

## Skills relacionadas

- `prompt-forge` — entrevista que monta o prompt e compõe estes blocos.
- `orchestrator` — roteador do toolkit.
- `graph-engineering` — tickets de task graph usam os blocos B2-B7.
- `self-learning` — protocolo de colheita que alimenta esta biblioteca.
