---
{
  "type": "PromptBlock",
  "title": "B6 · Contrato de memória coletiva (OKF)",
  "description": "Bloco que liga o agente à memória coletiva versionada (ler índice antes, logar depois).",
  "tags": [
    "prompt",
    "memoria"
  ],
  "status": "active",
  "generated": {
    "by": "brenoazvd",
    "at": "2026-08-11"
  },
  "stale_after": "2027-01-01",
  "sources": [
    "Usuário que pediu contrato de memória coletiva"
  ]
}
---

# B6 · Contrato de memória coletiva (OKF)

```
Antes de começar: leia <index.md> (catálogo).
Ao terminar: registre no <log.md> (formato OKF) e rode <okf_check.py> (saída "erros: 0").
```

**Quando:** agente roda em repo com memória coletiva versionada.

**Outcome:** correção do usuário 2026-08-07 — agentes entravam sem contexto e o log ficava desatualizado; o contrato embutido nos repos (CLAUDE.md/AGENTS.md) é rede de segurança, o prompt é a garantia.
