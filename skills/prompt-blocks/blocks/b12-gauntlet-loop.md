---
type: PromptBlock
title: B12 · Gauntlet Loop (Criação de alta fidelidade)
description: Prompt final comprovado para criação/design de alta qualidade contra uma referência NOMEADA (COD, Hades, Linear...). Paralelismo entre itens (sub-agentes se o host suportar, senão sequencial), crítico separado e HARSH, blind A/B lado a lado contra a referência real, META invertida (barra inatingível, o humano é o brake). Cole e preencha [THING]/[REFERENCE]/[STACK].
tags:
  - prompt
  - criacao
  - design
  - qualidade
  - referencia
  - loop
status: active
generated:
  by: brenoazvd
  at: 2026-08-12
stale_after: 2027-01-01
sources:
  - Matt Shumer (@mattshumer_) — Gauntlet Loop / Claude-of-Duty (MIT)
  - github.com/duolahypercho/gauntlet-loop (skill derivada, MIT)
  - github.com/mshumer/Claude-of-Duty (prompt original)
---

# B12 · Gauntlet Loop (Criação de alta fidelidade)

Este é o **bloco de prompt para criação/design de alta qualidade** — a versão comprovada do que a
linha "Criação/Design" da matriz do `prompt-forge` entrega quando a barra é alta e há uma
referência real para comparar. Texto pronto para colar (preencha os `[...]`) no prompt do agente
de execução:

```
Construa [THING] no nível de [REFERENCE]. Deve ser absolutamente perfeito, [LOOK], com cada
coisa feita em qualidade [TIER], de [AREA_1] a [AREA_2] a qualquer coisa que você puder imaginar.

Se o seu ambiente suportar múltiplas instâncias/sub-agentes em paralelo, distribua-os e faça
cada um atacar um item individualmente; se não suportar, processe item por item, sequencialmente,
sem pular nenhum. De qualquer forma, o [THING] deve ficar absolutamente perfeito. Use [LOOP_VERB]
em cada item e tenha uma instância SEPARADA verificando [CHECK] para garantir que esteja em
[TIER]. Essa instância separada deve ser um crítico REALMENTE rígido, e se não estiver em [TIER],
ele deve continuar.

Não pare até que o crítico fique absolutamente impressionado com a qualidade ao comparar com
[REFERENCE]. Ele deve literalmente comparar lado a lado às cegas e dizer qual das duas fica
melhor. Faça isso em [STACK]. [LOOP_VERB] até ficar absolutamente perfeito, repetindo o ciclo
(em paralelo se possível, senão sequencial)[CLOSING_TAIL].
```

### Placeholders (nouns)

| Slot | Preencher com | Default para criação/design |
|---|---|---|
| `[THING]` | o que construir | um jogo FPS, um site, uma demo, um infográfico |
| `[REFERENCE]` | **uma referência real e nomeada** | COD, Hades, Brotato, linear.app, um deck famoso |
| `[LOOK]` | descrição visual | `visualmente belo` |
| `[TIER]` | nível de qualidade | `AAA` / `qualidade de estúdio top` |
| `[AREA_1]` / `[AREA_2]` | duas frentes de trabalho | `texturas`/`física`, `combate`/`iluminação`, `tipografia`/`movimento` |
| `[CHECK]` | como o crítico verifica | `visualmente` (frame no jogo vs referência) |
| `[STACK]` | stack/ferramenta de execução | ThreeJS, Godot, Next.js + Tailwind... |
| `[LOOP_VERB]` | **verbo do CLI escolhido pelo usuário** para iterar (agente usado) | pergunte ao usuário; ex.: `/loop` no Claude Code — nomes variam por host, nunca assuma que existe; use "repita o ciclo manualmente" se o host não tiver um |
| `[CLOSING_TAIL]` | **fecho opcional do host** (verbo de "modo intenso", se existir) | espaço + verbo de intensidade (ex.: ` e <modo-intenso>`), ou vazio por padrão |

### A META invertida (difere das outras linhas)

Regra de ouro deste bloco — **contrasta com a regra geral do prompt-forge** ("a META define quando
PODE parar"):

- A barra é **inatingível por construção**. Comparado contra `[REFERENCE]`, a comparação às cegas
  sempre vai seguir falhando, e é isso que empurra a qualidade para cima.
- **O humano é o brake.** Nenhuma parada automática, nenhum "N ciclos limpos", nenhum "bom o
  suficiente". O agente só para quando o humano parar (ou um orçamento explícito bater).
- **Nunca** perguntar "quer que eu continue?" após um ciclo — só seguir.
- Critério subjetivo **não** é aceite; o único juiz válido é o crítico separado comparando às
  cegas com `[REFERENCE]`.

### Regras do crítico

- Crítico **separado do executor** (nunca a mesma IA que fez julga o próprio trabalho).
- Deve ser **realmente rígido/harsh** — sem versão suave, sem abaixar a barra.
- Julga **frame no jogo / resultado real** contra a referência real — não a sua própria intenção.
- Se não estiver em `[TIER]`, **continua** (não entrega como está).

### O que NÃO fazer

- Não virar o problema em ferramenta (suites de captura, máquinas de estado, contratos de
  arquitetura, ajudantes de script). O prompt é o método.
- Não gastar o run construindo tooling em vez do `[THING]`.
- Não abaixar `[REFERENCE]` nem suavizar o crítico.
- **Blind A/B é decisivo**: comparar lado a lado às cegas e declarar "qual fica melhor" — não
  "ambas estão boas".

**Quando usar:** tipo de pedido **Criação/Design** (matriz do `prompt-forge`) em que o usuário traz
ou aceita uma referência de qualidade (ex.: "nível Call of Duty", "como o site da Linear"). É o
prompt final padrão que o `prompt-forge` entrega nesse caso — em vez de um prompt genérico de
criação. Para código/análise/texto, use os blocos correspondentes (B4 karpathy, B7, B5...).

**Outcome:** técnica validada em campo pelo próprio Matt Shumer — o prompt de 152 palavras gerou
de um único prompt um jogo FPS completo (visualmente indistinguível de real, 3.8M views) sem
nenhum asset externo. A skill `gauntlet-loop` (duolahypercho) é a mesma técnica empacotada como
slash-command. A META invertida ("humano é o brake") é o diferencial que a matriz atual do
prompt-forge não captura — esta é a correção.
