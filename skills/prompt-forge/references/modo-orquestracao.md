# Modo Orquestração

> Blocos citados (B1-B14) ficam em `../../prompt-blocks/blocks/` (catálogo: `../../prompt-blocks/SKILL.md`; blocos pessoais em `blocks/local/`).

Para pedidos **multi-agente/multi-etapa** (dashboard multi-aba, ETL, N frentes), o formato padrão é
**UM prompt em prosa, na voz do usuário** (3 parágrafos, cerca de 600 a 700 palavras). O agente
executor vira o diretor: quebra o trabalho sozinho e usa as skills que já tem instaladas.

1. **O quê e por quê:** objetivo, contexto que o agente não tem, o que o usuário já percebeu, onde
   procurar, e "toda conclusão aponta a fonte".
2. **Como:** um sub-agente por item; skills e agentes chamados pelo nome (ex.: `/azvd-toolkit:skill-router`
   em cada item, `karpathy-guidelines` em toda alteração, o verbo de loop do host em cada item); crítico
   separado e rigoroso, com a régua escrita (o que é grave, o que é defeito); teste antes e depois num caso real.
3. **Orquestrador**, começando por "Você é o orquestrador e diretor-chefe". Esqueleto: registre achados
   e plano em documento, não só no chat; apresente o plano antes, em ordem de impacto; depois do meu ok,
   um item por vez com o mesmo crítico; um juiz avalia o conjunto no fim; limites (sem push, sem
   produção, sem apagar dado); reporte em [FORMATO]; distribua os sub-agentes pelas regras de custo (B11).

Perguntas (uma por vez):

1. **Quais são os itens/frentes?** → um sub-agente por item, em ordem de impacto.
2. **Há usuário para dar o ok do plano?** Se não (rodar à noite, `-p`): no §3, troque "depois do meu ok"
   por um critério de parada (ex.: juiz aprova o conjunto ou N voltas sem progresso) e liste o que fica
   para o usuário decidir.
3. **Qual verbo de loop o host tem?** (nunca assuma `/loop`)
4. **O agente-alvo tem o azvd-toolkit instalado e sub-agentes?**

Regras:
- Toolkit instalado: cite bloco, skill e agente pelo nome (B11, B13, `critico`, `juiz`) e não cole o texto.
  Sem ele: cole o texto real dos blocos.
- Run longo com agentes em loop → cite/inclua o bloco B13 (orçamento de tokens).
- **Exceção:** task graph + um prompt por ticket (`graph-engineering` → `references/task-graphs.md`, cada
  ticket no Modo correspondente) só se o usuário pedir prompts separados ou o agente-alvo não tiver sub-agentes.

Entregável: **1 bloco de código com o prompt único em prosa**.

Critério de parada: **"cada item aprovado pelo crítico + juiz aprova o conjunto"**.
