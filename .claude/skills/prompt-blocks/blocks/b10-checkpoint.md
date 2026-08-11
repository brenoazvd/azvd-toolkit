---
{
  "type": "PromptBlock",
  "title": "B10 · Checkpoint de retomada",
  "description": "Bloco de resiliência: estado REAL em disco a cada fase para retomada.",
  "tags": [
    "prompt",
    "resiliencia"
  ],
  "status": "active",
  "generated": {
    "by": "brenoazvd",
    "at": "2026-08-11"
  },
  "stale_after": "2027-01-01",
  "sources": [
    "vibecode-pro-max-kit (2026)",
    "agy T13 (OEP)"
  ]
}
---

# B10 · Checkpoint de retomada (não perca o lugar)

```
CHECKPOINT — escreva ao fim de cada fase, leia ao retomar:
- Fase concluída / próxima fase
- Estado REAL (commits, arquivos, probes) — não o que "pretendia fazer"
- Decisões pendentes que dependem do usuário
```

**Quando:** runs longos (multi-ticket, multi-fase) que podem ser cortados por timeout/reset de contexto.

**Outcome:** vibecode-pro-max-kit — "never loses its place": progress notes em disco a cada fase; sessão nova retoma exatamente onde parou. Lição do ecossistema OEP: agy exit 0 com diff vazio (plano ≠ execução) — o checkpoint registra a verdade do que foi FEITO, não do que foi planejado.
