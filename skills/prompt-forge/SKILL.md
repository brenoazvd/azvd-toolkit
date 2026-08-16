---
name: prompt-forge
description: "Use para criar/refinar prompts de IA por entrevista interativa. Cada tipo de pedido (Criação, Código, Análise, Orquestração, Texto) tem um MODO que gera o prompt pronto para colar — você responde slots curtos, a skill monta o prompt com crítica separada e critério objetivo de parada (filosofia Gauntlet Loop). Anatomia Tarefa/Método/Meta é o fallback. Entrega em qualquer CLI de agente."
trigger: /prompt-forge
---

# Prompt Forge — forja prompts por entrevista (modos prontos)

Todo pedido de prompt tem um **MODO** que o resolve. Você não descreve objetivo/método/meta em
prosa: você responde **slots curtos**, e a skill monta o prompt final **pronto para colar**, com a
técnica comprovada do domínio.

## A filosofia (herdada do Gauntlet Loop)

Cinco princípios atravessam todos os modos. São a "alma" da técnica que gera jogo nível AAA de um
prompt, adaptada por domínio — **universais em todo Modo**, não só em Criação/Design:

1. **A entrevista gera o prompt pronto.** O usuário responde slots; a skill monta. Ele nunca escreve
   o prompt na mão.
2. **Crítica separada e rigorosa.** Nenhuma IA julga o próprio trabalho — sempre há um verificador
   separado (modelo mais forte, skill especializada, ou você).
3. **Critério objetivo de parada.** Nunca "pareceu funcionar" — sempre um check verificável (build
   verde, probe, print, blind A/B).
4. **Não pare sem evidência.** Sem "bom o suficiente" auto-declarado; o agente só para quando o
   check objetivo passa (ou o humano para).
5. **Paralelismo + loop quando o pedido decompõe.** Se o trabalho quebra em itens independentes
   (arquivos, hipóteses, tickets), distribua em sub-agentes/instâncias paralelas; se o host tiver
   verbo de iteração (`[LOOP_VERB]`, ex.: `/loop`), use-o para repetir o ciclo até o critério do
   Modo bater. Cada Modo abaixo já embute essa mecânica no seu esqueleto de montagem.

> O que **não** é transferível fica só em Criação/Design: blind A/B contra **referência visual
> nomeada** e "humano é o brake" (barra inatingível por construção). Fora de Criação, o critério de
> parada é sempre objetivo do domínio (build verde, causa raiz com evidência, tickets verdes) — nunca
> estético; forçar referência visual ali é o que quebra.

## Anatomia de fallback (Tarefa/Método/Meta)

Quando nenhum Modo se encaixa (pedido atípico), use a anatomia base. Todo prompt tem 3 partes;
sem as 3, a IA inventa o que falta:

| Seção | Pergunta que responde | Conteúdo |
|---|---|---|
| **A TAREFA** | O que a IA deve fazer? | Objetivo concreto, contexto mínimo, entregáveis, escopo |
| **O MÉTODO** | Como ela deve fazer? | Passos, ferramentas, restrições, formato de saída, idioma |
| **A META** | Quando ela PODE PARAR? | Critério de aceite objetivo + o que NÃO fazer |

Sem a META a IA define o próprio critério de parada (e erra). Sem o MÉTODO ela improvisa caminho.

## Regras de ouro da entrevista

1. **UMA pergunta por vez.** Nunca despeje um questionário inteiro.
2. **Extraia antes de perguntar.** Leia o pedido inicial e preencha os slots que já vieram; pergunte
   **apenas o que faltar**. Nunca re-pergunte o que já foi informado.
3. **Dúvida = PERGUNTAR.** Se algo ficou ambíguo (agente alvo, autonomia, o que fazer com o
   resultado, idioma…), pare e abra uma caixa de texto com a pergunta. Nunca preencha com suposição
   silenciosa.
4. **A/B quando der.** Ofereça 2-3 opções objetivas, com uma recomendada.
5. **Exemplo no domínio do usuário**, nunca genérico ("uma empresa", "um sistema").
6. **Não aceite "entendeu?".** Peça aplicação: "me dá um exemplo de entrada e a saída que você
   espera".
7. **Só entrega o prompt final quando todos os slots do Modo estiverem preenchidos** (com os
   defaults automáticos aplicados nos que não precisam ser perguntados).

### Modo headless / sem usuário (`-p`, script, subagente)

Se NÃO houver usuário interativo para responder a entrevista (modo `-p`, automação, ou quando o
agente é chamado por outro agente), a regra 3 muda:

- **NÃO invente respostas em silêncio.** Se uma resposta for obrigatória e não houver usuário,
  preencha com a opção padrão mais segura e **declare a premissa explicitamente no prompt final**
  (ex.: "PREMISSA: agente-alvo = CLI do usuário; autonomia = analisa+reporta; modelo = leve/forte
  conforme B11").
- O prompt final deve sempre listar as **premissas assumidas** em uma seção própria, para o usuário
  revisar depois. É a forma de "perguntar" quando não dá pra perguntar.
- Se o pedido for ambíguo a ponto de qualquer escolha segura poder estar errada, e não houver como
  perguntar, prefira **não_entregar** e reportar "faltou contexto" — nunca adivinhe sentido crítico.

## Entrevista adaptativa — os 5 Modos

Na abertura, **identifique o TIPO do pedido** e siga o Modo da linha (sub-seções abaixo). A tabela
é o índice; o roteiro completo de cada Modo está na sub-seção correspondente.

| Tipo | Exemplos | Modo (sub-seção) | Crítica separada | Critério de parada |
|---|---|---|---|---|
| **Criação/Design** | jogo, site, landing, dashboard, demo, infográfico | **Modo Gauntlet Loop** (bloco B12) | crítico harsh + blind A/B vs referência; `impeccable` p/ UI | "crítico impressionado vs [REFERENCE] — e o humano para o loop" |
| **Código** | endpoint, componente, bugfix | **Modo Código** (B4 karpathy + B2 + B5 + B7) | revisão separada (modelo mais forte / code review) | "build+tsc verdes, git diff só toca os arquivos X, probe confirma" |
| **Análise/Diagnóstico** | "por que o KPI erra?", review de diff | **Modo Análise** (B3 + B7) | leve analisa → forte revisa → você confere | "causa raiz com evidência (probe/print) + fix mínimo proposto" |
| **Orquestração** | dashboard multi-aba, ETL, N agentes | **Modo Orquestração** (graph-engineering) | verificação do orquestrador por ticket | "todos os tickets verdes + verificação do orquestrador" |
| **Texto/Conteúdo** | artigo, email, resumo, thread | **Modo Texto** (B7 + humanizer) | releitura separada (voz humana / formato) | "entrega no formato X com tom Y, pronto pra enviar" |

### Modo Criação/Design (Gauntlet Loop)

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
- `[LOOK]` → derivado do tom da referência (ex.: `belo e fluido` para web/UI, `estilo AAA` para jogos).
- `[CHECK]` → **UI/web:** `visualmente, via crítico de design disponível no host (ex.: skill impeccable no Claude Code) ou blind A/B manual se o host não tiver um`; **jogo/demo:** `visualmente (frame no jogo vs referência)`.
- `[LOOP_VERB]` → **pergunte ao usuário** qual verbo/comando de iteração o CLI dele tem (ex.: `/loop` no Claude Code); se o host não tiver um comando nativo, use "repita o ciclo manualmente" — nunca assuma que `/loop` existe em todo lugar.
  `[CLOSING_TAIL]` → fecho do host se existir, senão vazio.

Depois, preencha o esqueleto do bloco
`skills/prompt-blocks/blocks/b12-gauntlet-loop.md` com esses valores e **entregue o prompt final
pronto**. (O esqueleto fica no próprio arquivo B12 — diferente dos outros modos, que compõem
blocos inline, porque o B12 é um bloco único pronto para colar, não uma composição.)

**UI/web:** se a criação for site/landing/dashboard/UI, o "crítico harsh" de design deve ser: a
skill `impeccable` **se o host for Claude Code**; caso contrário, qualquer revisor de design que o
host tiver, ou — na ausência de um — o próprio usuário fazendo blind A/B manual contra a
`[REFERENCE]`. Nunca assuma `impeccable` disponível fora do Claude Code.

### Modo Código

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
- Crítica separada → um segundo agente/modelo **mais forte** revisa o diff (a própria IA que codou
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

### Modo Análise/Diagnóstico

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
- Modelo → **leve** para exploração, **mais forte** para a revisão (bloco `b11` — usado pela
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

### Modo Orquestração

Para pedidos **multi-agente/multi-etapa** (dashboard multi-aba, ETL, N agentes), **não** monte um
prompt gigante (buga a IA). Monte um **task graph** e **um prompt por ticket**.

Perguntas (uma por vez):

1. **Quantas frentes/agentes?** → fan-out (paralelo) ou diamond (converge depois)?
2. **Dependências entre elas?** → o que bloqueia o quê.
3. **Onde é o gate humano?** → ponto de revisão em que você aprova antes de seguir.
4. **Contrato entre agentes paralelos** → o que cada um entrega (formato/id), para não colidir.

Defaults automáticos:
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

### Modo Texto/Conteúdo

Para pedidos de **texto** (artigo, email, resumo, thread), o prompt é um **brief de conteúdo** com
formato explícito.

Perguntas (uma por vez, extraia antes):

1. **Tom e público?** → formal/informal, técnico/leigo, para quem é.
2. **Estrutura?** → seções, tamanho, formato (artigo, thread, email, resumo).
3. **Idioma?** → PT-BR / EN / outro.
4. **O que evitar?** → AI-isms, jargão de blog de IA, buzzwords (ex.: "déficit de memória",
   "seamless", "stop rule").

Defaults automáticos:
- Crítica separada → releitura: a voz é humana, sem marcas de IA, pronto pra enviar.
- Se o usuário quiser "humanizar" um texto já escrito, aponte a skill `humanizer` **se o host for
  Claude Code**; noutro host, aplique a releitura manualmente (sem marcas de IA, tom humano).
- Paralelismo → raro aqui (texto é sequencial); se for uma thread/série com N peças independentes,
  um sub-agente por peça. Loop → `[LOOP_VERB]` na releitura até sair sem marca de IA.

Esqueleto de montagem (nesta ordem):
`[TAREFA: o que escrever + tom/público] + [estrutura/seções] + B7 (resumo objetivo, se for
resumo/relatório) + [formato de saída explícito]`.

Montagem do prompt (componha com blocos):
- **B7 (resumo objetivo)** — se for resumo/relatório.
- Formato de saída explícito (seções, tamanho, tom) — sempre.

Critério de parada: **"entrega no formato X com tom Y, sem AI-isms, pronto pra enviar"**.

## Fluxo

0. **Revisão de contexto (sempre, antes de perguntar ou montar).** Releia a mensagem do usuário e
   abra de verdade qualquer arquivo/path/skill/repo que ele citar — não assuma pela memória de
   sessões anteriores nem pelo nome do arquivo. Se algo citado mudou, sumiu, ou o pedido depende de
   um estado que você não conferiu ainda (schema, contrato de API, versão de skill), confira antes
   de seguir. É aqui que se evita errar em coisas básicas por montar o prompt em cima de suposição
   estale.
1. **Abertura (até 4 perguntas):**
   - O que você vai pedir? (1 frase) → já **identifica o TIPO/Modo** aqui.
   - Para qual agente? (claude CLI / agy / codex / cursor / outro)
   - **Qual modelo você prefere?** — agora que o TIPO é conhecido, sugira pela categoria (use o
     bloco `skills/prompt-blocks/blocks/b11-roteamento-modelos.md`): análise/exploração → modelo
     "leve", execução/código → "forte", revisão → "mais forte". **Sugira por CATEGORIA
     (leve/forte/médio), NUNCA por nome de modelo/provedor** (não cite "Gemini", "Claude", "GPT"...
     — quem escolhe o nome é o usuário). Ofereça a categoria e pergunte qual ele prefere.
   - Nível de autonomia? (executa tudo / executa e reporta / só analisa)
2. **Siga o Modo** da matriz correspondente ao TIPO identificado (Criação/Código/Análise/
   Orquestração/Texto). Só caia no fallback Tarefa/Método/Meta se nenhum Modo encaixar.
3. **Monte o prompt** com os blocos do Modo (abra os arquivos reais — não improvise da memória).
4. **Entrega** — prompt final em bloco de código, pronto para colar no CLI escolhido na abertura
   (ex.: `claude -p "$(cat prompt.txt)"` no Claude Code; `agy`/`codex`/`cursor` colam o mesmo texto
   na sua própria interface — o prompt é o artefato portável, não o comando de invocação).

## Seções extras (opcionais, só se fizerem sentido)

- **"O QUE JÁ FOI VERIFICADO (não refaça)"** — para agente rodando em cima de trabalho/diagnóstico
  existente (bloco B3; já é padrão no Modo Análise).
- **"LEIA PRIMEIRO"** — lista de arquivos obrigatórios antes de agir (bloco B1).
- **"Resumo objetivo no final"** — formato do relatório de saída do agente (bloco B7).

## Regras do prompt resultante (contrato com o agente)

- **Autocontido**: o agente não conhece a conversa — embuta os fatos do contexto. Não referencie
  caminhos fora do repo sem `--add-dir`.
- **Componha seções com blocos da skill `prompt-blocks`** quando houver bloco aplicável (PARE E
  REPORTE, karpathy, teste de decisão…). Para **Criação/Design com referência nomeada**, o prompt
  final **é** o bloco `b12-gauntlet-loop.md`.
- **Prompt único gigante = NÃO.** Se o pedido for multi-etapa ou multi-agente, use o **Modo
  Orquestração** (`graph-engineering`, task graph, um prompt por ticket) — juntar tudo num prompt
  buga a IA.

## Skills relacionadas

- `skill-router` — porta de entrada: encaminha para esta skill quando o pedido é "montar prompt".
- `prompt-blocks` — blocos comprovados (com lição de incidente) que os Modos compõem.
- `orchestrator` — roteador do toolkit (quando usar esta skill).
- `graph-engineering` — se o pedido virar orquestração multi-agente (task graph).
- `impeccable` (global, **disponível apenas no Claude Code**) — crítico/iteração de design; o
  "crítico harsh" do Modo Criação quando é UI. Noutro host, use o revisor de design disponível ali
  ou blind A/B manual.
- `humanizer` (global, **disponível apenas no Claude Code**) — tirar AI-isms de texto (Modo Texto).
  Noutro host, aplique a releitura manualmente.

### Como usar os blocos do prompt-blocks (obrigatório)

Quando o fluxo disser "componha com blocos", **abra de verdade os arquivos** em
`skills/prompt-blocks/blocks/` e **cole o texto do bloco real** (B1-B12) no prompt que está
montando — não improvise a partir da memória. Leia o `skills/prompt-blocks/SKILL.md` (catálogo) e
abra o(s) bloco(s) aplicável(eis) antes de preencher o placeholder. B12 (Gauntlet Loop) é o bloco
do tipo **Criação/Design com referência nomeada** — use-o como esqueleto do prompt final nesse caso.

## Auto-atualização

A entrevista revelou uma pergunta, ferramenta, bloco ou tipo de pedido que faltava em algum Modo?
Adicione uma linha/slot nova (protocolo `self-learning`, regra de promoção de 3 verificações) — os
Modos se adaptam aos pedidos que você realmente faz.
