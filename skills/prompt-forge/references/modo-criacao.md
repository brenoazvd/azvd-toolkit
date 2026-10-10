# Modo Criação/Design (Gauntlet Loop)

> Blocos citados (B1-B13) ficam em `../../prompt-blocks/blocks/` (catálogo: `../../prompt-blocks/SKILL.md`; blocos pessoais em `blocks/local/`).

Para pedidos de **criação**, o prompt **não é montado a mão** — é gerado pela sub-entrevista. Mas
primeiro **descubra se há referência**:

1. **Extraia do pedido inicial** o máximo de slots (`[THING]`, `[REFERENCE]`, `[STACK]`, `[TIER]`,
   `[AREA_1]`/`[AREA_2]`) que já vieram na mensagem do usuário. Pergunte **apenas o que faltar**,
   uma pergunta por vez — nunca re-pergunte o que já foi informado.
2. **Com referência real nomeada** (ex.: "nível Call of Duty", "como a Linear") → ative o **Modo
   Gauntlet Loop** (abaixo): pergunte os slots que faltam e **entregue o prompt B12 pronto**.
3. **Sem referência nomeada** (ou o usuário recusa dar uma) → siga o fluxo Tarefa/Método/Meta
   padrão para criação simples (sem o loop). Não force o Gauntlet Loop sem referência.

Perguntas do Modo Gauntlet Loop (só para os slots ainda em aberto):

1. **O que você quer criar?** → `[THING]` (jogo FPS, landing, dashboard, demo...).
2. **Contra qual referência real?** → `[REFERENCE]` (Call of Duty, linear.app, Hades, Brotato...).
   Sem isso o loop não roda — pergunte direto.
3. **Em qual stack?** → `[STACK]` (ThreeJS, Next.js+Tailwind, Godot...).
4. **Quão alta a barra?** (default: **AAA**) → `[TIER]`. Raramente faz sentido abaixar.
5. **Quais 2 áreas mais importam?** → `[AREA_1]`/`[AREA_2]`.

Defaults automáticos (não pergunte, preencha sozinho):
- Run longo (fan-out de vários itens com crítico e A/B) → anexe o bloco B13 (orçamento e desperdício de
  tokens): no máximo 2 voltas simultâneas, medição por volta, parada sem progresso.
- `[LOOK]` → derivado do tom da referência (ex.: `belo e fluido` para web/UI, `estilo AAA` para jogos).
- `[CHECK]` → **UI/web:** `visualmente, via crítico de design disponível no host (ex.: skill impeccable no Claude Code) ou blind A/B manual se o host não tiver um`; **jogo/demo:** `visualmente (frame no jogo vs referência)`.
- `[LOOP_VERB]` → **pergunte ao usuário** qual verbo/comando de iteração o CLI dele tem (ex.: `/loop` no Claude Code); se o host não tiver um comando nativo, use "repita o ciclo manualmente" — nunca assuma que `/loop` existe em todo lugar.
  `[CLOSING_TAIL]` → fecho do host se existir, senão vazio.

Depois, preencha o esqueleto do bloco
`../../prompt-blocks/blocks/b12-gauntlet-loop.md` com esses valores e **entregue o prompt final
pronto**. (O esqueleto fica no próprio arquivo B12 — diferente dos outros modos, que compõem
blocos inline, porque o B12 é um bloco único pronto para colar, não uma composição.)

**UI/web:** se a criação for site/landing/dashboard/UI, o "crítico harsh" de design deve ser: a
skill `impeccable` **se o host for Claude Code**; caso contrário, qualquer revisor de design que o
host tiver, ou — na ausência de um — o próprio usuário fazendo blind A/B manual contra a
`[REFERENCE]`. Nunca assuma `impeccable` disponível fora do Claude Code.
