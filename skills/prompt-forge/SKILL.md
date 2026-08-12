---
name: prompt-forge
description: "Use para criar/refinar prompts de IA por entrevista interativa (anatomia Tarefa/Método/Meta). O agente pergunta UMA coisa por vez; em dúvida, para e pergunta em vez de inventar. Entrega prompt final pronto para colar (qualquer CLI de agente (claude, agy, codex, cursor...))."
trigger: /prompt-forge
---

# Prompt Forge — forja prompts por entrevista

Todo prompt de qualidade tem 3 partes (anatomia conceitual). Sem as 3, a IA inventa o que falta:

| Seção | Pergunta que responde | Conteúdo |
|---|---|---|
| **A TAREFA** | O que a IA deve fazer? | Objetivo concreto, contexto mínimo, entregáveis, escopo |
| **O MÉTODO** | Como ela deve fazer? | Passos, ferramentas, restrições, formato de saída, idioma |
| **A META** | Quando ela PODE PARAR? | Critério de aceite objetivo + o que NÃO fazer |

Sem a META a IA define o próprio critério de parada (e erra). Sem o MÉTODO ela improvisa caminho (e viola escopo).

## Regras de ouro da entrevista

1. **UMA pergunta por vez.** Nunca despeje um questionário inteiro.
2. **Dúvida = PERGUNTAR.** Se algo ficou ambíguo (agente alvo, nível de autonomia, o que fazer com o resultado, idioma…), pare e abra uma caixa de texto com a pergunta. Nunca preencha com suposição silenciosa.
3. **A/B quando der.** Ofereça 2-3 opções objetivas, com uma recomendada.
4. **Exemplo no domínio do usuário**, nunca genérico ("uma empresa", "um sistema").
5. **Não aceite "entendeu?".** Peça aplicação: "me dá um exemplo de entrada e a saída que você espera".
6. **Só entrega o prompt final quando as 3 seções estiverem preenchidas sem lacuna.**

### Modo headless / sem usuário (`-p`, script, subagente)

Se NÃO houver usuário interativo para responder a entrevista (modo `-p`, automação, ou quando o
agente é chamado por outro agente), a regra 2 muda:

- **NÃO invente respostas em silêncio.** Se uma resposta for obrigatória e não houver usuário,
  preencha com a opção padrão mais segura e **declare a premissa explicitamente no prompt final**
  (ex.: "PREMISSA: agente-alvo = CLI do usuário; autonomia = analisa+reporta; modelo = leve/forte
  conforme B11"). 
- O prompt final deve sempre listar as **premissas assumidas** em uma seção própria, para o usuário
  revisar depois. É a forma de "perguntar" quando não dá pra perguntar.
- Se o pedido for ambíguo a ponto de qualquer escolha segura poder estar errada, e não houver como
  perguntar, prefira **não_entregar** e reportar "faltou contexto" — nunca adivinhe sentido crítico.

## Entrevista adaptativa (matriz por tipo de pedido)

Na abertura, **identifique o TIPO do pedido** e use a linha da matriz para guiar as perguntas do
MÉTODO e da META — nunca entrevista genérica. Quando o usuário já traz uma referência de qualidade
(ex.: "nível Call of Duty"), transforme-a em **benchmark objetivo** (alvo comparável, quem julga, critério).

| Tipo | Exemplos | O que o MÉTODO deve puxar | Ferramentas/verbos que o prompt ganha | META típica |
|---|---|---|---|---|
| **Criação/Design** | jogo FPS, site, landing, dashboard, demo, infográfico | **Modo Gauntlet Loop** (ver sub-seção abaixo): o que + **referência real NOMEADA** + stack + tier + 2 áreas. Para **UI/web**, o crítico de design é a skill `impeccable` | **Gera automaticamente o prompt estilo Gauntlet Loop** (bloco B12): fan-out por item + `[LOOP_VERB]` + crítico separado e HARSH + blind A/B vs referência real + META invertida (humano é o brake). Você **não escreve** o prompt — a entrevista monta | "o crítico separado, comparando às cegas com [REFERENCE], fica impressionado — e o humano para o loop (sem parada automática)" |
| **Código** | endpoint, componente, bugfix | repo/arquivos, agente alvo e modelo, gates (tsc/build/probes), escopo cirúrgico | `seu CLI de código (com as flags de modelo/tempo do agente)`, `--allowedTools`, PARE E REPORTE, regras karpathy (cirúrgico), testes | "build+tsc verdes, git diff só toca os arquivos X, probe confirma" |
| **Orquestração** | dashboard multi-aba, ETL, N agentes | quantos agentes/CLIs, dependências, onde é o gate humano | task graph (fan-out/diamond/human gate), **um prompt por ticket** (prompt único buga), contrato entre agentes paralelos | "todos os tickets verdes + verificação do orquestrador" |
| **Análise/Diagnóstico** | "por que o KPI erra?", review de diff | o que JÁ foi verificado (não refaça), fontes, hipóteses a testar | agente leve analisa → agente forte revisa → você confere; probes; seção "O QUE JÁ FOI VERIFICADO" | "causa raiz com evidência (probe/print) + fix mínimo proposto" |
| **Texto/Conteúdo** | artigo, email, resumo, thread | tom, público, estrutura, tamanho, idioma | formato de saída explícito, seções, "sem AI-isms / voz humana" | "entrega no formato X com tom Y, pronto pra enviar" |

Regra: cada linha vira **perguntas específicas** na entrevista e **seções concretas** no prompt final.

### Modo Criação/Design (Gauntlet Loop)

Para pedidos na linha **Criação/Design**, o prompt **não é montado a mão** — é gerado pela
sub-entrevista. Mas primeiro **descubra se há referência**:

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
- `[CHECK]` → **UI/web:** `visualmente via skill impeccable (ou render)`; **jogo/demo:** `visualmente (frame no jogo vs referência)`.
- `[LOOP_VERB]` → verbo do CLI escolhido na abertura (ex.: `/loop`, `/goal`);
  `[CLOSING_TAIL]` → fecho do host se existir, senão vazio.

Depois, preencha o esqueleto do bloco
`skills/prompt-blocks/blocks/b12-gauntlet-loop.md` com esses valores e **entregue o prompt final
pronto**.

**UI/web:** se a criação for site/landing/dashboard/UI, além de gerar o prompt Gauntlet Loop,
aponte que a **crítica e a iteração de design** são feitas pela skill `impeccable` (critique/audit/
polish) — ela é o "crítico harsh" especializado em design, com benchmark próprio. Ela entra na fase
de construção/revisão da UI, não na geração do prompt.

Observação de escopo: o Gauntlet Loop (blind A/B contra referência visual) é o método certo para
**Criação** (algo novo com referência comparável). Ele **não substitui** o método de Código
(cirúrgico, gates), Análise (leve→forte→confere) nem Texto (tom/formato) — esses continuam com a
anatomia Tarefa/Método/Meta.

## Fluxo

1. **Abertura (até 4 perguntas):**
   - O que você vai pedir? (1 frase)
   - Para qual agente? (claude CLI / agy / codex / cursor / outro)
   - **Qual modelo você prefere?** — sugerir pelo TIPO de pedido (use o bloco `skills/prompt-blocks/blocks/b11-roteamento-modelos.md`): análise/exploração → modelo "leve", execução/código → "forte", revisão → "mais forte". **Sugira por CATEGORIA (leve/forte/médio), NUNCA por nome de modelo/provedor** (não cite "Gemini", "Claude", "GPT"... — quem escolhe o nome é o usuário). Ofereça a categoria e pergunte qual ele prefere.
   - Nível de autonomia? (executa tudo / executa e reporta / só analisa)
2. **A TAREFA** — objetivo verificável, contexto mínimo, entregáveis, o que está FORA do escopo.
3. **O MÉTODO** — passos numerados, arquivos/ferramentas permitidos, formato de saída, idioma.
4. **A META** — critério de aceite objetivo (ex.: "build passa, git diff só toca os arquivos X"), regra de parada ("se Y, PARE e reporte"), o que NÃO fazer.
5. **Entrega** — prompt final em bloco de código, pronto para colar, com sugestão de comando (ex.: `claude -p "$(cat prompt.txt)" --model sonnet`).

## Seções extras (opcionais, só se fizerem sentido)

- **"O QUE JÁ FOI VERIFICADO (não refaça)"** — para agente rodando em cima de trabalho/diagnóstico existente.
- **"LEIA PRIMEIRO"** — lista de arquivos obrigatórios antes de agir.
- **"Resumo objetivo no final"** — formato do relatório de saída do agente.

## Regras do prompt resultante (contrato com o agente)

- **Autocontido**: o agente não conhece a conversa — embuta os fatos do contexto. Não referencie caminhos fora do repo sem `--add-dir`.
- **Componha seções com blocos da skill `prompt-blocks`** quando houver bloco aplicável (PARE E REPORTE, karpathy, teste de decisão…). Para o tipo **Criação/Design com referência nomeada**, o prompt final **é** o bloco `b12-gauntlet-loop.md` — use-o pronto em vez de montar um prompt de criação genérico. Sem referência nomeada, monte o fluxo flexível de criação.
- **Prompt único gigante = NÃO.** Se o pedido for multi-etapa ou multi-agente, sugira `graph-engineering` (task graph) e um prompt por ticket — juntar tudo num prompt buga a IA.

## Skills relacionadas

- `skill-router` — porta de entrada: encaminha para esta skill quando o pedido é "montar prompt".
- `prompt-blocks` — blocos comprovados (com lição de incidente) para compor as seções.
- `orchestrator` — roteador do toolkit (quando usar esta skill).
- `graph-engineering` — se o pedido virar orquestração multi-agente (task graph).

### Como usar os blocos do prompt-blocks (obrigatório)

Quando o fluxo disser "componha com blocos", **abra de verdade os arquivos** em
`skills/prompt-blocks/blocks/` e **cole o texto do bloco real** (B1-B12) no prompt que está
montando — não improvise a partir da memória. Leia o `skills/prompt-blocks/SKILL.md` (catálogo) e
abra o(s) bloco(s) aplicável(eis) antes de preencher o placeholder. B12 (Gauntlet Loop) é o bloco
do tipo **Criação/Design com referência nomeada** — use-o como esqueleto do prompt final nesse caso
(ver matriz).

## Auto-atualização

A entrevista revelou uma pergunta, ferramenta ou tipo de pedido que faltava na matriz? Adicione uma
linha nova (protocolo `self-learning`, regra de promoção de 3 verificações) — a entrevista se adapta
aos pedidos que você realmente faz.
