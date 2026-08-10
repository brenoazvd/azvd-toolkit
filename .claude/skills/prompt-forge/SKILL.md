---
name: prompt-forge
description: "Use para criar/refinar prompts de IA por entrevista interativa (anatomia Tarefa/Método/Meta). O agente pergunta UMA coisa por vez; em dúvida, para e pergunta em vez de inventar. Entrega prompt final pronto para colar (claude CLI, agy, ChatGPT, subagente)."
trigger: /prompt-forge
---

# Prompt Forge — forja prompts por entrevista

Todo prompt de qualidade tem 3 partes (anatomia conceitual). Sem as 3, a IA inventa o que falta:

| Seção | Pergunta que responde | Conteúdo |
|---|---|---|
| **A TAREFA** | O que a IA deve fazer? | Objetivo concreto, contexto mínimo, entregáveis, escopo |
| **O MÉTODO** | Como ela deve fazer? | Passos, ferramentas, restrições, formato de saída, idioma |
| **A META** | Quando ela PODE PARAR? | Critério de aceite objetivo + o que NÃO fazer |

Sem a META a IA define o próprio critério de parada (e erra). Sem o MÉTODO ela improvisa caminho (e viola escopo).

## Regras de ouro da entrevista

1. **UMA pergunta por vez.** Nunca despeje um questionário inteiro.
2. **Dúvida = PERGUNTAR.** Se algo ficou ambíguo (agente alvo, nível de autonomia, o que fazer com o resultado, idioma…), pare e abra uma caixa de texto com a pergunta. Nunca preencha com suposição silenciosa.
3. **A/B quando der.** Ofereça 2-3 opções objetivas, com uma recomendada.
4. **Exemplo no domínio do usuário**, nunca genérico ("uma empresa", "um sistema").
5. **Não aceite "entendeu?".** Peça aplicação: "me dá um exemplo de entrada e a saída que você espera".
6. **Só entrega o prompt final quando as 3 seções estiverem preenchidas sem lacuna.**

## Fluxo

1. **Abertura (até 3 perguntas):**
   - O que você vai pedir? (1 frase)
   - Para qual agente? (claude CLI / agy / ChatGPT / subagente / outro)
   - Nível de autonomia? (executa tudo / executa e reporta / só analisa)
2. **A TAREFA** — objetivo verificável, contexto mínimo, entregáveis, o que está FORA do escopo.
3. **O MÉTODO** — passos numerados, arquivos/ferramentas permitidos, formato de saída, idioma.
4. **A META** — critério de aceite objetivo (ex.: "build passa, git diff só toca os arquivos X"), regra de parada ("se Y, PARE e reporte"), o que NÃO fazer.
5. **Entrega** — prompt final em bloco de código, pronto para colar, com sugestão de comando (ex.: `claude -p "$(cat prompt.txt)" --model sonnet`).

## Seções extras (opcionais, só se fizerem sentido)

- **"O QUE JÁ FOI VERIFICADO (não refaça)"** — para agente rodando em cima de trabalho/diagnóstico existente.
- **"LEIA PRIMEIRO"** — lista de arquivos obrigatórios antes de agir.
- **"Resumo objetivo no final"** — formato do relatório de saída do agente.

## Regras do prompt resultante (contrato com o agente)

- **Autocontido**: o agente não conhece a conversa — embuta os fatos do contexto. Não referencie caminhos fora do repo sem `--add-dir`.
- **Componha seções com blocos da skill `prompt-blocks`** quando houver bloco aplicável (PARE E REPORTE, karpathy, teste de decisão…).
- **Prompt único gigante = NÃO.** Se o pedido for multi-etapa ou multi-agente, sugira `graph-engineering` (task graph) e um prompt por ticket — juntar tudo num prompt buga a IA.

## Skills relacionadas

- `prompt-blocks` — blocos comprovados (com lição de incidente) para compor as seções.
- `orchestrator` — roteador do toolkit (quando usar esta skill).
- `graph-engineering` — se o pedido virar orquestração multi-agente (task graph).
