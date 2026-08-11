---
name: prompt-blocks
description: "Use para compor/refinar prompts com blocos comprovados (LEIA PRIMEIRO, PARE E REPORTE, regras karpathy, teste de decisão, contrato OKF, resumo objetivo), cada um com lição de incidente real. Alimenta prompt-forge e qualquer ticket."
trigger: /prompt-blocks
---

# Prompt Blocks — blocos comprovados com outcome

Cada bloco: texto pronto para colar + quando usar + a lição que o originou. Compor um prompt = escolher blocos + preencher as lacunas com o domínio real. **Nunca colar bloco sem preencher os placeholders** — bloco com lacuna vira instrução vazia.

## B1 · LEIA PRIMEIRO (read-first)
```
LEIA PRIMEIRO:
- <arquivo1> — <por quê>
- <arquivo2> — <por quê>
```
**Quando:** o agente precisa de contexto antes de agir (docs de regra, código existente, índice de memória).
**Outcome:** padrão agy 2026-08 — sem leitura obrigatória, o agente age sobre suposição e erra campo/join.

## B2 · PARE E REPORTE (stop & report)
```
Se <condição>, PARE e reporte — NÃO improvise nem invente alternativa.
```
**Quando:** risco de escopo, dado/config ausente, decisão de negócio, token/infra que não deve ser inventado ("não invente o token; se não achar, PARE e reporte").
**Outcome:** incidente T15.2 (builder reescreveu `repository.py` inteiro mesmo com instrução explícita) e agy T13 ("plano ≠ execução": exit 0 com `git diff --stat` vazio = analisou, não executou).

## B3 · O QUE JÁ FOI VERIFICADO (não refaça)
```
O QUE JÁ FOI VERIFICADO (não refaça isso):
- <fato> — <evidência>
```
**Quando:** agente roda em cima de investigação/diagnóstico já feito (padrão analyzer→reviewer).
**Outcome:** 2026-08-10 — revisor Opus 5 validou em cima da análise do agy sem redescobrir; sem a seção, trabalho duplicado e re-verificação de tudo.

## B4 · Regras karpathy (execução cirúrgica)
```
REGRAS DE EXECUÇÃO:
- Mexa SÓ nos arquivos deste ticket; não refatore o que não está quebrado.
- Não commite; deixe o WIP.
- Valide ao final: <tsc --noEmit / npm run build / python -m compileall>.
- Se algo conflitar com o plano, PARE e reporte em vez de improvisar.
```
**Quando:** todo builder (claude/agy/subagente).
**Outcome:** incidente T15.2 (2026-08-06) — reduziu a classe de violação de escopo.

## B5 · Teste de decisão (passo 1 autônomo)
```
PASSO 1 — TESTE DE DECISÃO: rode <comando/teste>. Se <resultado X>, faça <caminho A>. Se <resultado Y>, faça <caminho B>.
```
**Quando:** agente externo/autônomo que não pode voltar ao usuário (o teste de decisão dá autonomia sem risco).
**Outcome:** `ticket_etl_retry_agent.txt` (2026-08-10) — testar `DIM_EDU_ETP_TUR` com timeout 600s decide "re-disparar" vs "skip-list", sem depender do usuário.

## B6 · Contrato de memória coletiva (OKF)
```
Antes de começar: leia <index.md> (catálogo).
Ao terminar: registre no <log.md> (formato OKF) e rode <okf_check.py> (saída "erros: 0").
```
**Quando:** agente roda em repo com memória coletiva versionada.
**Outcome:** correção do usuário 2026-08-07 — agentes entravam sem contexto e o log ficava desatualizado; o contrato embutido nos repos (CLAUDE.md/AGENTS.md) é rede de segurança, o prompt é a garantia.

## B7 · Resumo objetivo (formato de saída)
```
RESUMO FINAL (objetivo, sem narrativa):
- O que foi feito, com evidência (exit code, git diff --stat, probes HTTP, hashes)
- O que NÃO foi feito e por quê
- Pendências / decisões que dependem do usuário
```
**Quando:** todo agente que reporta de volta ao orquestrador.
**Outcome:** regra do usuário "nunca confiar no resumo" — o resumo é ponto de partida da verificação, não a verdade.

## B8 · Entrevista A/B (interativa)
```
Pergunte UMA coisa por vez, com opções A/B/C e uma recomendada. Só avance quando o usuário responder.
```
**Quando:** qualquer skill interativa (a `prompt-forge` usa isto).

## B9 · Memória do projeto (auto-atualizada)
```
MEMÓRIA DO PROJETO — leia no início da sessão, atualize ao final de cada entrega:
- Fatos e decisões desta sessão que valem pra próxima
- O que mudou (rotas, comandos, schema, arquivos-chave)
- Becos sem saída conhecidos (e o porquê)
```
**Quando:** projetos com agentes recorrentes; o arquivo (MEMORY.md no repo, OKF `log.md`, ou CLAUDE.md/AGENTS.md) é lido no start e recebe append no fim.
**Outcome:** vibecode-pro-max-kit (2026) — "self-improving project memory": docs nunca ficam stale, a próxima sessão começa sabendo o que a anterior aprendeu. Mesmo princípio do contrato OKF já validado no ecossistema OEP (2026-08-07).

## B10 · Checkpoint de retomada (não perca o lugar)
```
CHECKPOINT — escreva ao fim de cada fase, leia ao retomar:
- Fase concluída / próxima fase
- Estado REAL (commits, arquivos, probes) — não o que "pretendia fazer"
- Decisões pendentes que dependem do usuário
```
**Quando:** runs longos (multi-ticket, multi-fase) que podem ser cortados por timeout/reset de contexto.
**Outcome:** vibecode-pro-max-kit — "never loses its place": progress notes em disco a cada fase; sessão nova retoma exatamente onde parou. Lição do ecossistema OEP: agy exit 0 com diff vazio (plano ≠ execução) — o checkpoint registra a verdade do que foi FEITO, não do que foi planejado.

## Como usar

1. Identifique a seção do prompt que precisa (contexto / método / meta / saída).
2. Cole o bloco + preencha os `<placeholders>` com o domínio real.
3. Registre no final do bloco usado a lição nova que você aprendeu — esta biblioteca cresce com outcome, não com teoria.

## Auto-atualização

Descobriu uma lição nova durante o uso (correção do usuário, incidente, padrão que se repetiu)?
Ela vira um **bloco novo** aqui (B9, B10...) com: o texto pronto + quando usar + a lição que originou
(protocolo `self-learning`, regra de promoção de 3 verificações — check que passou + padrão de falha
nomeado + beco descartado). O prompt-forge passa a compor com o bloco automaticamente.

## Skills relacionadas

- `prompt-forge` — entrevista que monta o prompt e compõe estes blocos.
- `orchestrator` — roteador do toolkit.
- `graph-engineering` — tickets de task graph usam os blocos B2-B7.
