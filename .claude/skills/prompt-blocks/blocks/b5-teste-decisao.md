# B5 · Teste de decisão (passo 1 autônomo)

```
PASSO 1 — TESTE DE DECISÃO: rode <comando/teste>. Se <resultado X>, faça <caminho A>. Se <resultado Y>, faça <caminho B>.
```

**Quando:** agente externo/autônomo que não pode voltar ao usuário (o teste de decisão dá autonomia sem risco).

**Outcome:** `ticket_etl_retry_agent.txt` (2026-08-10) — testar `DIM_EDU_ETP_TUR` com timeout 600s decide "re-disparar" vs "skip-list", sem depender do usuário.
