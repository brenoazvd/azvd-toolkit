# Modo Orquestração

> Blocos citados (B1-B13) ficam em `../../prompt-blocks/blocks/` (catálogo: `../../prompt-blocks/SKILL.md`; blocos pessoais em `blocks/local/`).

Para pedidos **multi-agente/multi-etapa** (dashboard multi-aba, ETL, N agentes), **não** monte um
prompt gigante (buga a IA). Monte um **task graph** e **um prompt por ticket**.

Perguntas (uma por vez):

1. **Quantas frentes/agentes?** → fan-out (paralelo) ou diamond (converge depois)?
2. **Dependências entre elas?** → o que bloqueia o quê.
3. **Onde é o gate humano?** → ponto de revisão em que você aprova antes de seguir.
4. **Contrato entre agentes paralelos** → o que cada um entrega (formato/id), para não colidir.

Defaults automáticos:
- Run longo com agentes em loop → inclua o bloco B13 (orçamento e desperdício de tokens): paralelismo
  limitado, medição por volta e por bloco de cota, parada sem progresso.
- Use `graph-engineering` → `references/task-graphs.md` para desenhar o grafo (fan-out/diamond/
  human gate).
- **Um prompt por ticket**, cada um montado com o Modo correspondente (um ticket de código vira
  Modo Código; um de análise vira Modo Análise).
- Verificação separada → o **orquestrador** confere cada ticket antes de marcar verde.

Entregável (formato de saída): **1 bloco de código com o prompt do Orquestrador Mestre** (que
desenha o grafo e distribui os tickets) + **N blocos de código, um prompt por ticket**. Se o grafo
for complexo (fan-out/diamond), encaminhe para `graph-engineering` desenhar o task graph antes de
montar os prompts.

Critério de parada: **"todos os tickets verdes + verificação do orquestrador"**.
