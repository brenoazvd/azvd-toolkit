# Modo Código

> Blocos citados (B1-B12) ficam em `../../prompt-blocks/blocks/` (catálogo: `../../prompt-blocks/SKILL.md`; blocos pessoais em `blocks/local/`).

Para pedidos de **código** (endpoint, componente, bugfix, refactor), o prompt final é um **contrato
cirúrgico** — nunca um "faça isso e veja no que dá".

Perguntas (uma por vez, extraia antes o que já veio):

1. **Onde?** → repo/arquivos/paths que o agente pode tocar (e os que NÃO pode).
2. **Qual o bug/feature?** → comportamento esperado vs atual (peça um caso de entrada→saída).
3. **Quais os gates?** → comandos de verificação (build, testes, tsc, lint, probe) que devem ficar
   verdes antes de considerar pronto.
4. **Escopo cirúrgico:** → o que está FORA (não refatorar código adjacente, não "melhorar" o que
   não foi pedido).

Defaults automáticos (preencha sozinho):
- Modelo → categoria **forte** para execução (bloco `b11-roteamento-modelos.md`).
- Crítica separada → um segundo agente (`critico`, modelo **forte**, contexto separado) revisa o diff (a própria IA que codou
  não julga o próprio trabalho).
- Autonomia → da abertura (não re-perguntar).
- Paralelismo → se o bug/feature quebra em arquivos/módulos independentes, distribua um sub-agente
  por arquivo (mesmo escopo cirúrgico cada um). Loop → repita ciclo (corrige → roda gates → repassa
  pro crítico) via `[LOOP_VERB]` do host até todos os gates ficarem verdes.

Esqueleto de montagem (nesta ordem):
`[TAREFA/contexto do bug] + B4 (karpathy) + B2 (PARE E REPORTE) + B5 (teste de decisão, se agir
sem volta) + B7 (resumo objetivo) + [META: gates verdes]`.

Montagem do prompt (componha com blocos reais — abra e cole):
- **B4 (karpathy)** — comportamento cirúrgico: pense antes, simplicidade, mudanças mínimas, objetivo.
- **B2 (PARE E REPORTE)** — se faltar arquivo/dado/contexto, parar e reportar em vez de inventar.
- **B5 (teste de decisão)** — se o agente vai agir sem volta ao usuário.
- **B7 (resumo objetivo)** — formato do relatório final (o que fez, o que passou, o que sobrou).

Critério de parada (sempre no prompt): **"build+tsc/lint verdes, `git diff` só toca os arquivos X,
probe confirma o comportamento"** — não "pareceu funcionar".
