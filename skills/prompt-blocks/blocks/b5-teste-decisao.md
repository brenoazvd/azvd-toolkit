---
{
  "type": "PromptBlock",
  "title": "B5 · Teste de decisão",
  "description": "Bloco de autonomia: passo 1 com teste que decide o caminho sem voltar ao usuário.",
  "tags": [
    "prompt",
    "autonomia"
  ],
  "status": "active",
  "generated": {
    "by": "brenoazvd",
    "at": "2026-08-11"
  },
  "stale_after": "2027-01-01",
  "sources": [
    "Agente autônomo que precisava decidir caminho sem voltar ao usuário"
  ]
}
---

# B5 · Teste de decisão (passo 1 autônomo)

```
PASSO 1 — TESTE DE DECISÃO: rode <comando/teste>. Se <resultado X>, faça <caminho A>. Se <resultado Y>, faça <caminho B>.
```

**Quando:** agente externo/autônomo que não pode voltar ao usuário (o teste de decisão dá autonomia sem risco).

**Outcome:** Agente autônomo que, com timeout de 600s num teste de decisão, resolveu "re-disparar" vs "skip-list" sem depender do usuário.
