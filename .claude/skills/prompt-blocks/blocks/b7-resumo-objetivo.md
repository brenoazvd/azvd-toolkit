# B7 · Resumo objetivo (formato de saída)

```
RESUMO FINAL (objetivo, sem narrativa):
- O que foi feito, com evidência (exit code, git diff --stat, probes HTTP, hashes)
- O que NÃO foi feito e por quê
- Pendências / decisões que dependem do usuário
```

**Quando:** todo agente que reporta de volta ao orquestrador.

**Outcome:** regra do usuário "nunca confiar no resumo" — o resumo é ponto de partida da verificação, não a verdade.
