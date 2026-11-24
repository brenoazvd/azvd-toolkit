# Modo Orquestração

> Blocos citados (B1-B14) ficam em `../../prompt-blocks/blocks/` (catálogo: `../../prompt-blocks/SKILL.md`; blocos pessoais em `blocks/local/`).

Para pedidos **multi-agente/multi-etapa** (dashboard multi-aba, ETL, N frentes), o formato padrão é
**UM prompt em prosa, na voz do usuário: 3 parágrafos curtos, alvo 150 a 350 palavras, teto ~450**. Molde de
tom e tamanho: o bloco de código do `b12-gauntlet-loop.md` (~170 palavras). O agente executor vira o
diretor: decide os itens sozinho e usa as skills que já tem. Frases imperativas, sem listas, sem títulos,
sem seção PREMISSAS dentro do prompt: premissas e caminhos a trocar vão FORA do bloco, na resposta ao usuário.

1. **O quê e a barra:** o que construir/resolver e a régua (referência real nomeada ou critério objetivo do
   domínio, ex.: "o número bate exato com o script de origem"), mais só os fatos que o agente não descobre
   sozinho, em 1 ou 2 frases.
2. **Como:** distribua sub-agentes, um por item; o diretor decide os itens (o prompt não os lista); verbo de
   loop do host em cada item; crítico separado e rígido que verifica [CHECK] (as verificações que o usuário
   pediu, ex.: visual, código, regra de negócio) e, se não passar, continua.
3. **Orquestrador.** Esqueleto: "Você é o orquestrador e diretor-chefe. Não pare até o crítico ficar
   impressionado (comparando às cegas lado a lado, quando há referência) e um juiz aprovar o conjunto.
   Registre o plano e o andamento em documento. Sem push, sem produção, sem apagar dado. Reporte no fim em
   [FORMATO]. Distribua os sub-agentes pelas regras de custo (B11). [LOOP_VERB] até ficar perfeito[FECHO]."
   Sem usuário para o ok, a parada é esse critério; com usuário, apresente o plano antes e siga depois do ok.

Perguntas (uma por vez):

1. **Quais são os itens/frentes?** → só para você saber a ordem de impacto; o prompt não os lista.
2. **Há usuário para dar o ok do plano?** Se não (rodar à noite, `-p`): a parada é o critério do §3 (juiz
   aprova o conjunto ou N voltas sem progresso); liste na resposta o que fica para o usuário decidir.
3. **Qual verbo de loop o host tem?** (nunca assuma `/loop`) Mais o fecho do host (modo intenso), se houver.
4. **O agente-alvo tem o azvd-toolkit instalado e sub-agentes?**

Regras:
- Toolkit instalado: cite bloco, skill e agente pelo nome (B11, B13, `critico`, `juiz`) e não cole o texto.
  Sem ele: cole o texto real dos blocos.
- Run longo com agentes em loop → cite/inclua o bloco B13 (orçamento de tokens).
- **Exceção:** task graph + um prompt por ticket (`graph-engineering` → `references/task-graphs.md`, cada
  ticket no Modo correspondente) só se o usuário pedir prompts separados ou o agente-alvo não tiver sub-agentes.

Entregável: **1 bloco de código com o prompt único em prosa**; premissas e caminhos a trocar, fora dele.

Critério de parada: **"cada item aprovado pelo crítico + juiz aprova o conjunto"**.
