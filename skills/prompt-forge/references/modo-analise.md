# Modo Análise/Diagnóstico

> Blocos citados (B1-B13) ficam em `../../prompt-blocks/blocks/` (catálogo: `../../prompt-blocks/SKILL.md`; blocos pessoais em `blocks/local/`).

Para pedidos de **análise** ("por que o KPI erra?", review de diff, investigação), o prompt é uma
**investigação guiada** — nunca um "me explica isso" aberto.

Perguntas (uma por vez, extraia antes):

1. **O que já foi verificado?** → para não refazer trabalho (vira a seção B3 no prompt).
2. **Qual a fonte?** → onde está a verdade (DB, logs, código, dashboard, print do RM...).
3. **Quais hipóteses testar?** → 2-3 suspeitas iniciais, ou "descubra" se não houver.
4. **O que entregar?** → causa raiz + evidência + fix mínimo proposto, ou só o diagnóstico.

Defaults automáticos:
- Crítica separada → **modelo leve analisa → modelo forte revisa → você confere** (o analista não
  valida o próprio achado).
- Modelo → **leve** para exploração, **forte** para a revisão (bloco `b11` — usado pela
  entrevista para configurar o pipeline, NÃO colado no corpo do prompt do analista).
- Paralelismo → se houver 2-3 hipóteses independentes, um sub-agente por hipótese, todos correndo
  antes da revisão. Loop → `[LOOP_VERB]` até a causa raiz ter evidência real (não "acho que é isso").

Esqueleto de montagem (nesta ordem):
`[TAREFA/pergunta] + B3 (O QUE JÁ FOI VERIFICADO) + [fonte + hipóteses] + B7 (resumo objetivo) +
[META: causa raiz + evidência]`.

Montagem do prompt (componha com blocos):
- **B3 (O QUE JÁ FOI VERIFICADO)** — o que não refazer.
- **B7 (resumo objetivo)** — formato do relatório.

Critério de parada: **"causa raiz com evidência (probe/print/query que reproduz) + fix mínimo
proposto"** — não "acho que é isso".
