---
type: PromptBlock
title: B13 · Orçamento e desperdício de tokens
description: Regras para runs longos com vários agentes caberem na cota — medir antes/durante/depois, limitar paralelismo, parar volta sem progresso e contabilizar o desperdício (voltas perdidas, desfeitas ou cortadas pelo limite). Cole no prompt do orquestrador.
tags:
  - prompt
  - custo
  - tokens
  - orquestracao
  - loop
status: active
generated:
  by: brenoazvd
  at: 2026-10-09
stale_after: 2027-04-01
sources:
  - Run noturno de landing com até 7 workflows simultâneos: a cota acabou em ~1h30 por janela e cada corte reiniciou todos os agentes que estavam no meio
  - https://dev.to/maximsaplin/the-ai-bill-grows-in-the-agent-loop-87n (custo = tentativas × turnos × paralelismo)
  - https://ccusage.com/guide/blocks-reports (consumo por bloco de 5 h, ritmo de queima, previsão)
  - https://www.ml4devs.com/what-is/agent-loops (orçamento, parada sem progresso)
  - https://github.com/anthropics/claude-code/issues/22625 (sem medição nativa por sub-agente)
---

# B13 · Orçamento e desperdício de tokens

Texto pronto (cole no prompt do orquestrador de um run longo com vários agentes):

```
ORÇAMENTO DE TOKENS (vale durante todo o run):

O custo de um run com agentes é TENTATIVAS × TURNOS POR AGENTE × PARALELISMO. Enxugar texto de prompt
muda pouco; o que estoura a cota é volta repetida sem progresso e muita coisa em paralelo.

1. Gargalo é a cota, não o relógio. Paralelismo não termina antes: gasta a mesma cota mais rápido e,
   quando o limite corta, TODO agente que estava no meio recomeça do zero (só os já terminados voltam do
   cache). Por isso: no máximo [N_PARALELO] workflows/voltas ao mesmo tempo (padrão 2). Exceção: o
   julgamento periódico do conjunto pode rodar como extra, sem travar as voltas.
2. Meça antes, durante e depois:
   - antes de disparar: estime o custo de UMA volta (some os agentes dela) e quantas voltas cabem no
     bloco de cota atual;
   - durante: acompanhe o bloco atual (ex.: `npx ccusage@latest blocks --live`: ritmo de queima e
     previsão de quando acaba) e o total de tokens que cada workflow/volta reporta;
   - depois de cada volta: registre no STATUS os tokens dela e se ela foi APROVADA, PERDIDA (A/B ou
     crítico não melhorou), DESFEITA (piorou) ou CORTADA (limite de uso no meio);
   - a cada ~20 voltas: some o gasto POR PAPEL (construtor, crítico, captura, A/B, juiz), pelos rótulos
     dos agentes (a tela de progresso do workflow mostra os tokens de cada agente; os transcripts dos
     sub-agentes trazem o `usage`). O gasto se concentra num papel; ajuste esse primeiro, pelo bloco
     B11 · Roteamento por tarefa (nível de esforço antes do modelo, sempre com uma volta de teste).
3. Pare a volta que não progride: [N_SEM_PROGRESSO] voltas seguidas sem melhora no mesmo item (padrão 2)
   → o item vai para o fim da fila e a próxima volta dele precisa mudar a abordagem, não repetir.
4. Perto do fim do bloco (pela previsão do passo 2): não dispare volta nova que não termina antes do
   corte; salve o ponto de retomada (bloco B10) e pare limpo.
5. Na retomada: retome primeiro o que já estava mais adiantado (já construído/capturado), no máximo
   [N_PARALELO] por vez.
6. Ao fim de cada bloco, informe uma linha de DESPERDÍCIO: tokens em voltas perdidas + desfeitas +
   cortadas, sobre o total do bloco. Se passar de [LIMITE_DESPERDICIO] (padrão 30%), reduza paralelismo
   ou mude a abordagem dos itens que mais perderam antes de continuar.
```

**Quando:** qualquer run longo com vários agentes em loop (construção iterativa com crítico, A/B,
juiz), principalmente com assinatura de cota por janela de tempo.

**Outcome:** num run de construção de landing com até 7 workflows simultâneos, a cota acabou em cerca
de 1h30 por janela; o limite cortou tudo que estava no meio (6 workflows numa só parada) e várias voltas
consumiram a rodada inteira para no fim serem perdidas no A/B ou desfeitas por falha grande nova. Limitar a
2 voltas simultâneas não muda a qualidade (mesmos modelos e régua) nem o tempo total, que já é limitado
pela cota — só reduz o que se perde em cada corte.
