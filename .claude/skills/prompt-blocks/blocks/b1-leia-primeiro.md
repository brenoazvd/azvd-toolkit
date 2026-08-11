---
{
  "type": "PromptBlock",
  "title": "B1 · LEIA PRIMEIRO (read-first)",
  "description": "Bloco que obriga o agente a ler arquivos de contexto antes de agir.",
  "tags": [
    "prompt",
    "contexto"
  ],
  "status": "active",
  "generated": {
    "by": "brenoazvd",
    "at": "2026-08-11"
  },
  "stale_after": "2027-01-01",
  "sources": [
    "Padrão agy 2026-08 (OEP)"
  ]
}
---

# B1 · LEIA PRIMEIRO (read-first)

```
LEIA PRIMEIRO:
- <arquivo1> — <por quê>
- <arquivo2> — <por quê>
```

**Quando:** o agente precisa de contexto antes de agir (docs de regra, código existente, índice de memória).

**Outcome:** padrão agy 2026-08 — sem leitura obrigatória, o agente age sobre suposição e erra campo/join.
