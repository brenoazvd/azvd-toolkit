# B2 · PARE E REPORTE (stop & report)

```
Se <condição>, PARE e reporte — NÃO improvise nem invente alternativa.
```

**Quando:** risco de escopo, dado/config ausente, decisão de negócio, token/infra que não deve ser inventado ("não invente o token; se não achar, PARE e reporte").

**Outcome:** incidente T15.2 (builder reescreveu `repository.py` inteiro mesmo com instrução explícita) e agy T13 ("plano ≠ execução": exit 0 com `git diff --stat` vazio = analisou, não executou).
