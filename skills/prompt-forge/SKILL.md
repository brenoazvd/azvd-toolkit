---
name: prompt-forge
description: "Use quando o usuário quer um PROMPT para entregar a uma IA/agente: criar, refinar ou corrigir um prompt ('monta um prompt pra…', 'como peço X pra IA', 'melhora esse prompt'). Entrevista curta (uma pergunta por vez) e entrega o prompt pronto para colar, com crítica separada e critério objetivo de parada. Não use para executar a tarefa em si."
trigger: /prompt-forge
---

# Prompt Forge — forja prompts por entrevista

Você não escreve a tarefa: você **monta o prompt** que outro agente vai executar. O usuário responde
slots curtos; a skill identifica o **Modo** do pedido, abre o roteiro dele e entrega o prompt pronto.

## Fluxo (siga nesta ordem)

0. **Revise o contexto antes de tudo.** Releia o pedido e abra de verdade cada arquivo, skill ou repo
   citado — não confie na memória nem no nome do arquivo. Se o pedido depende de um estado que você
   não conferiu (schema, contrato, versão), confira antes.
1. **Identifique o Modo** pela tabela abaixo e **abra o arquivo do Modo** (`references/modo-*.md`).
   Leia só o Modo escolhido. Exceção: no Modo Orquestração, abra também o Modo de cada ticket.
2. **Extraia antes de perguntar.** Preencha com o que já veio no pedido os slots da abertura e do Modo.
   Abertura (pergunte **só o que faltar**):
   - agente-alvo (qual CLI/host vai rodar o prompt);
   - categoria de modelo — sugira pela tarefa (bloco B11): leve para explorar, forte para executar
     e revisar, mais forte só para julgar o conjunto — e pergunte qual ele prefere. **Nunca cite nome de modelo/provedor**; o
     usuário escolhe o nome;
   - autonomia (executa tudo / executa e reporta / só analisa).
3. **Siga o roteiro do Modo**, uma pergunta por vez.
4. **Monte o prompt com os blocos reais.** Os blocos citados nos Modos (B1-B12) ficam em
   `../prompt-blocks/blocks/`; leia o catálogo `../prompt-blocks/SKILL.md`, cheque `blocks/` **e**
   `blocks/local/` (blocos pessoais do usuário) e cole o texto real — não reescreva de memória.
5. **Entregue** em um bloco de código, pronto para colar no agente escolhido (ex.: `claude -p
   "$(cat prompt.txt)"`; outros CLIs colam o mesmo texto — o prompt é o artefato portável). Liste as
   premissas que você assumiu, se houver.

## Os 5 Modos

| Tipo do pedido | Exemplos | Roteiro | Crítica separada | Critério de parada |
|---|---|---|---|---|
| **Criação/Design** | jogo, site, landing, dashboard | [`references/modo-criacao.md`](references/modo-criacao.md) (bloco B12) | crítico harsh + blind A/B vs referência nomeada | "crítico impressionado vs [REFERENCE] — o humano para o loop" |
| **Código** | endpoint, componente, bugfix | [`references/modo-codigo.md`](references/modo-codigo.md) (B4+B2+B5+B7) | outro agente revisa o diff | "build/testes verdes, diff só toca os arquivos X, probe confirma" |
| **Análise/Diagnóstico** | "por que o KPI erra?", review | [`references/modo-analise.md`](references/modo-analise.md) (B3+B7) | leve analisa → forte revisa → humano confere | "causa raiz com evidência + fix mínimo proposto" |
| **Orquestração** | N agentes, ETL, multi-fase | [`references/modo-orquestracao.md`](references/modo-orquestracao.md) (graph-engineering) | orquestrador confere cada ticket | "todos os tickets verdes + verificação do orquestrador" |
| **Texto/Conteúdo** | artigo, email, resumo | [`references/modo-texto.md`](references/modo-texto.md) (B7) | releitura separada (voz humana) | "formato X, tom Y, sem marcas de IA" |

Nenhum Modo encaixa → **anatomia de fallback**: todo prompt tem **TAREFA** (o que fazer: objetivo,
contexto mínimo, entregáveis), **MÉTODO** (como: passos, ferramentas, restrições, formato) e **META**
(quando pode parar: critério de aceite objetivo + o que não fazer). Sem META a IA define o próprio
critério de parada — e erra; sem MÉTODO ela improvisa o caminho.

## Princípios (valem em todo Modo)

1. **A entrevista gera o prompt.** O usuário responde slots; nunca escreve o prompt na mão.
2. **Crítica separada.** Quem constrói não julga o próprio trabalho — sempre há um verificador
   separado (outro agente forte em contexto separado, skill especializada ou o humano).
3. **Critério objetivo de parada.** Nunca "pareceu funcionar": build verde, probe, print, blind A/B.
   O agente não para com "bom o suficiente" auto-declarado — só quando o check passa (ou o humano para).
4. **Paralelismo + loop quando decompõe.** Itens independentes → sub-agentes em paralelo; se o host
   tiver verbo de iteração (`[LOOP_VERB]`, ex.: `/loop`), repita o ciclo até o critério bater.
   Pergunte qual verbo o host tem — nunca assuma que `/loop` existe.
5. **Prompt gigante = não.** Multi-etapa ou multi-agente → Modo Orquestração (task graph, um prompt
   por ticket).

Referência visual nomeada e "o humano é o brake" valem **só** em Criação/Design. Fora dela o
critério é objetivo do domínio — forçar referência visual ali é o que quebra.

## Regras da entrevista

1. **Uma pergunta por vez**, de preferência A/B ou 2-3 opções objetivas, com uma recomendada.
2. **Nunca re-pergunte** o que já veio. **Dúvida = pergunte**, não suponha em silêncio.
3. **Exemplo no domínio do usuário**, nunca genérico. Peça "um exemplo de entrada e a saída que você
   espera" em vez de aceitar "entendeu?".
4. **Só entregue** quando todos os slots do Modo estiverem preenchidos (defaults aplicados).

**Sem usuário para responder** (modo `-p`, script, chamado por outro agente): preencha cada slot
obrigatório com a opção mais segura e **declare a premissa numa seção "PREMISSAS" do prompt**. Se
qualquer escolha segura puder estar errada num ponto crítico, não entregue: reporte "faltou contexto".

## Contrato do prompt entregue

- **Autocontido:** o agente não viu esta conversa — embuta os fatos. Não referencie caminhos fora do
  repo do agente (ou passe-os com `--add-dir`).
- **Componha com blocos do `prompt-blocks`** quando houver bloco aplicável (B1 LEIA PRIMEIRO antes de agir, B2 PARE
  E REPORTE, B3 JÁ VERIFICADO sobre trabalho já feito, B7 resumo no fim, B13 orçamento de tokens em run longo com agentes…). Abra o arquivo e cole o texto real.
- **Papéis de sub-agente:** se o prompt pede construtor, crítico, juiz ou conferidor, use os agentes do
  toolkit (`agents/`) pelo nome em vez de descrever o papel de novo — ver `orchestrator`.

## Auto-atualização

A entrevista revelou uma pergunta, ferramenta ou tipo de pedido que faltava? Adicione no arquivo do
Modo (protocolo `self-learning`, regra das 3 verificações).

## Skills relacionadas

- `prompt-blocks` — blocos comprovados que os Modos compõem.
- `orchestrator` — encadeia skills e agentes do toolkit em tarefas multi-etapa.
- `graph-engineering` — desenha o task graph do Modo Orquestração.
- `skill-router` — só aponta a skill certa quando o usuário não sabe qual usar (chamado manualmente).
