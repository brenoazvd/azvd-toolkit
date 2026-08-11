---
type: PromptBlock
title: B11 · Roteamento de modelos (heurísticas)
description: Sugestão de qual modelo usar por tipo de tarefa (leve/forte/mais forte) com fallback — sem hardcode de modelos, sempre confirma com o usuário. Não entra no GitHub com preferência pessoal.
tags:
  - prompt
  - routing
  - modelos
  - custo
  - orquestracao
status: active
generated:
  by: brenoazvd
  at: 2026-08-11
stale_after: 2027-01-01
sources:
  - RouterPatterns OEP (leve analisa → forte revisa → humano confere)
  - awesome-model-routing (RouteLLM, ClawRouter, Agent-as-a-Router)
  - Prompt-Engineering-Guide (técnicas p/ esforço)
---

# B11 · Roteamento de modelos (heurísticas)

Texto pronto (cole no prompt do orquestrador/agent, ou use na hora de escolher o modelo):

```
ROTEAMENTO DE MODELOS (heurísticas — SUGESTÃO, confirme com o usuário):
┌─────────────────────┬──────────────────────────────────────┬─────────────────────┐
│ Tipo de tarefa      │ Modelo a usar                       │ Fallback            │
├─────────────────────┼──────────────────────────────────────┼─────────────────────┤
│ Análise/exploração  │ modelo leve/barato                  │ modelo médio        │
│ Execução/código     │ modelo forte                        │ médio + thinking    │
│ Revisão/verificação │ mais forte (contexto separado)      │ forte               │
│ Planejamento        │ modelo médio                        │ forte               │
│ Criação/design      │ modelo forte                        │ forte + thinking    │
└─────────────────────┴──────────────────────────────────────┴─────────────────────┘
Regras:
- Heurística é SUGESTÃO — SEMPRE pergunte ao usuário qual modelo/CLI ele prefere antes de escolher.
- Não hard-codar nomes de modelos no prompt; quem escolhe é o usuário (veja o bloco local/ dele).
- Tarefa repetitiva/leitura pesada → modelo leve (custo menor). Lógica de negócio/cálculo → forte.
- Verificação SEMPRE em contexto separado (uma IA não julga bem o próprio trabalho).
- Se um roteador (ex.: LiteLLM, ClawRouter, OmniRoute) estiver disponível, o usuário pode apontar o
  modelo para o proxy e o roteador decide — a heurística vira a política de fallback.
```

**Quando:** sempre que precisar escolher "qual modelo" para uma tarefa — especialmente em
orquestração multi-agente (um ticket de exploração não precisa do modelo caro).

**Outcome:** padrão validado em campo: a tarefa define o modelo (leve analisa → forte revisa →
humano confere); Routing como categoria de infra é referenciada (Link de referência:
[awesome-model-routing](https://github.com/yenanjing/awesome-model-routing)).
