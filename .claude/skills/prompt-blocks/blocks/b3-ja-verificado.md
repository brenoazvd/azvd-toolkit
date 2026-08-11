---
{
  "type": "PromptBlock",
  "title": "B3 · O QUE JÁ FOI VERIFICADO",
  "description": "Bloco anti-retrabalho: declara o que já foi provado para o agente não refazer.",
  "tags": [
    "prompt",
    "eficiencia"
  ],
  "status": "active",
  "generated": {
    "by": "brenoazvd",
    "at": "2026-08-11"
  },
  "stale_after": "2027-01-01",
  "sources": [
    "Agente leve analisando e agente forte revisando a mesma investigação"
  ]
}
---

# B3 · O QUE JÁ FOI VERIFICADO (não refaça)

```
O QUE JÁ FOI VERIFICADO (não refaça isso):
- <fato> — <evidência>
```

**Quando:** agente roda em cima de investigação/diagnóstico já feito (padrão analyzer→reviewer).

**Outcome:** 2026-08-10 — agente forte revisor validou em cima da análise do agente leve sem redescobrir; sem a seção, trabalho duplicado e re-verificação de tudo.
