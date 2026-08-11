---
{
  "type": "PromptBlock",
  "title": "B2 · PARE E REPORTE (stop & report)",
  "description": "Bloco que corta improviso: condição de parada com reporte ao orquestrador.",
  "tags": [
    "prompt",
    "escopo",
    "seguranca"
  ],
  "status": "active",
  "generated": {
    "by": "brenoazvd",
    "at": "2026-08-11"
  },
  "stale_after": "2027-01-01",
  "sources": [
    "Incidente Builder que reescreveu arquivo inteiro sem permissão",
    "Agente que reportou plano como execução (diff vazio)"
  ]
}
---

# B2 · PARE E REPORTE (stop & report)

```
Se <condição>, PARE e reporte — NÃO improvise nem invente alternativa.
```

**Quando:** risco de escopo, dado/config ausente, decisão de negócio, token/infra que não deve ser inventado ("não invente o token; se não achar, PARE e reporte").

**Outcome:** Builder que reescreveu arquivo inteiro mesmo com instrução explícita e agente que reportou plano como execução ("plano ≠ execução": exit 0 com `git diff --stat` vazio = analisou, não executou).
