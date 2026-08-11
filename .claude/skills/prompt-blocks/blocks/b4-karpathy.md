---
{
  "type": "PromptBlock",
  "title": "B4 · Regras karpathy",
  "description": "Bloco de execução cirúrgica: só os arquivos do ticket, validação no final, conflito → PARE.",
  "tags": [
    "prompt",
    "escopo",
    "execucao"
  ],
  "status": "active",
  "generated": {
    "by": "brenoazvd",
    "at": "2026-08-11"
  },
  "stale_after": "2027-01-01",
  "sources": [
    "Incidente T15.2 (OEP 2026-08-06)"
  ]
}
---

# B4 · Regras karpathy (execução cirúrgica)

```
REGRAS DE EXECUÇÃO:
- Mexa SÓ nos arquivos deste ticket; não refatore o que não está quebrado.
- Não commite; deixe o WIP.
- Valide ao final: <tsc --noEmit / npm run build / python -m compileall>.
- Se algo conflitar com o plano, PARE e reporte em vez de improvisar.
```

**Quando:** todo builder (claude/agy/subagente).

**Outcome:** incidente T15.2 (2026-08-06) — reduziu a classe de violação de escopo.
