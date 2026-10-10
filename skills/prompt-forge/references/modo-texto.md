# Modo Texto/Conteúdo

> Blocos citados (B1-B13) ficam em `../../prompt-blocks/blocks/` (catálogo: `../../prompt-blocks/SKILL.md`; blocos pessoais em `blocks/local/`).

Para pedidos de **texto** (artigo, email, resumo, thread), o prompt é um **brief de conteúdo** com
formato explícito.

Perguntas (uma por vez, extraia antes):

1. **Tom e público?** → formal/informal, técnico/leigo, para quem é.
2. **Estrutura?** → seções, tamanho, formato (artigo, thread, email, resumo).
3. **Idioma?** → PT-BR / EN / outro.
4. **O que evitar?** → AI-isms, jargão de blog de IA, buzzwords (ex.: "déficit de memória",
   "seamless", "stop rule").

Defaults automáticos:
- Crítica separada → releitura: a voz é humana, sem marcas de IA, pronto pra enviar.
- Se o usuário quiser "humanizar" um texto já escrito, aponte a skill `humanizer` **se o host for
  Claude Code**; noutro host, aplique a releitura manualmente (sem marcas de IA, tom humano).
- Paralelismo → raro aqui (texto é sequencial); se for uma thread/série com N peças independentes,
  um sub-agente por peça. Loop → `[LOOP_VERB]` na releitura até sair sem marca de IA.

Esqueleto de montagem (nesta ordem):
`[TAREFA: o que escrever + tom/público] + [estrutura/seções] + B7 (resumo objetivo, se for
resumo/relatório) + [formato de saída explícito]`.

Montagem do prompt (componha com blocos):
- **B7 (resumo objetivo)** — se for resumo/relatório.
- Formato de saída explícito (seções, tamanho, tom) — sempre.

Critério de parada: **"entrega no formato X com tom Y, sem AI-isms, pronto pra enviar"**.
