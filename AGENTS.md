# AGENTS.md — para qualquer IA que mexa neste repo

Este é o **azvd-toolkit**: um toolkit open-source de skills para trabalhar com IA
(orquestração, prompt engineering, graph engineering, auto-aprendizado). Nome do repo:
`brenoazvd/azvd-toolkit`.

## O que é este repo

6 skills adaptativas que qualquer agente (Claude Code, agy, Codex, Cursor, Hermes) pode usar, e 9 sub-agentes (`agents/`) para o Claude Code:

| Skill | Função |
|---|---|
| `skills/skill-router` | Aponta qual skill/agente resolve um pedido (roda quando o usuário cita /skill-router) |
| `skills/orchestrator` | Encadeia skills e monta o time de agentes com custo controlado |
| `skills/prompt-forge` | Forja prompts por entrevista: cada tipo (Criação/Código/Análise/Orquestração/Texto) tem um Modo que gera o prompt pronto (filosofia Gauntlet Loop) |
| `skills/prompt-blocks` | Protocolo de colheita de blocos de prompt (filtro F1-F5, OKF) + biblioteca B1-B13 |
| `skills/graph-engineering` | Knowledge graphs + task graphs (orquestração multi-agente), PT-BR |
| `skills/self-learning` | Colhe lições de sessão e atualiza as skills (com gate de permissão) |

| Agente (`agents/`) | Papel | Categoria de modelo · esforço |
|---|---|---|
| `verificador-previo` | antes de construir: precisa existir? já existe? premissas, critério de aceite | forte · high |
| `construtor` | implementa um item com escopo fechado | forte · high |
| `critico` | nota com régua + defeitos, sem consertar | forte · high |
| `juiz` | A/B cego nas duas ordens ou julgamento do conjunto | mais forte · medium |
| `conferidor-dados` | reproduz número/afirmação na fonte, só leitura | forte · high |
| `conferidor-tela` | confere na tela de um sistema web, só olhando | forte · medium |
| `testador-web` | testa a sua aplicação no navegador: fluxos, erros, dados, larguras, acessibilidade, visual e marcas de IA, com prova | forte · high |
| `revisor-query` | plano, índice e resultado antes × depois de uma query | forte · high |
| `triador` | classifica, filtra ou ranqueia muitos itens (≥ 20) contra rótulos fechados, com máscara de dado pessoal e lista de revisão | leve · high |

## Como as skills se organizam (arquitetura)

- `skills/<nome>/SKILL.md` — conteúdo de cada skill (fonte única). Curto (< 500 linhas); detalhe vai em
  `references/` (um nível só), como os Modos do `prompt-forge`.
- `agents/<nome>.md` — sub-agentes do plugin (frontmatter: `model`, `effort`, `maxTurns`, ferramentas).
- `skills/prompt-blocks/blocks/` — blocos de prompt em arquivos `.md` (1 bloco = 1 arquivo), estilo OKF.
- `skills/prompt-blocks/blocks/local/` — **blocos pessoais do usuário, GITIGNORED, não versionado**.
- `.claude-plugin/` — manifests do marketplace do Claude (só para instalação por marketplace; não é o conteúdo).
- `AGENTS.md` (este), `CONTEXT.md` — contexto/instruções de agente.
- `hooks/hooks.json`: no início de toda sessão imprime o resumo de custo do B11 (trecho entre `<!-- resumo-sessao: ... -->` e `<!-- /resumo-sessao -->`). Mudou a regra de custo? Mude ali.
- `RULES.md` — regras operacionais do mantenedor (sync, permissões) — **não confundir com este**.
- `sync.sh` — propaga habilidades do repo para as 4 IAs (Hermes, agy, claude, dir universal).

## Regras de conteúdo (para qualquer agente que editar)

1. **Nada de dados pessoais ou de projeto específico.** Sem nome de cliente, repositório interno,
   incidente com identificador (ex.: "T15.2"), e-mail, path de máquina local, ou credencial.
2. **Modelos de IA sempre genéricos.** Use "modelo mais forte" / "modelo rápido/leve" — nunca
   "claude opus", "Gemini Flash", etc. **Nunca citar nome de modelo/provedor em sugestão** —
   sugerir por categoria (leve/forte/médio). O usuário escolhe o nome.
   **Exceção:** o campo `model:` do frontmatter em `agents/` usa o apelido que o host exige
   (`haiku`/`sonnet`/`opus` = leve/forte/mais forte), porque sem ele o agente herda o modelo da sessão
   e o custo dispara. No texto dos agentes continua genérico. Quem quiser outro modelo sobrescreve com
   um arquivo de mesmo nome em `~/.claude/agents/` (tem precedência sobre o plugin).
3. **Idioma principal: PT-BR.** Skills, blocos e README em português brasileiro.
4. **Um bloco = um arquivo em `blocks/`.** `SKILL.md` é catálogo — nunca cresce com conteúdo de bloco novo.
5. **Blocos novos exigem permissão** — mesmo em `blocks/` quanto em `blocks/local/`. Editar
   conteúdo de bloco existente é automático.
6. **(Opcional, padrão do repo)** frontmatter em estilo OKF v0.2 quando for criar arquivo novo.
7. **Skill ou agente ajustado: teste antes e depois num caso real** (o mesmo pedido, 3 vezes com a versão
   velha e 3 com a nova, por exemplo `claude -p` com e sem `--plugin-dir`, ou o eval da skill `skill-creator`,
   que compara a skill velha com a nova). Regra de conduta (o que o agente responde numa conversa longa) se
   testa com o contexto real: cópia da sessão cortada antes do erro, reaberta com `--resume --fork-session`;
   prompt curto costuma passar onde a sessão longa falha e anote o resultado no commit. Mais longo
   e não melhor = desfaça. Mudança só de texto que não muda comportamento (contagem, link, erro de digitação)
   dispensa o teste.

## Fonte de verdade

- **Repo**: https://github.com/brenoazvd/azvd-toolkit
- Conteúdos puxados de upstream: ver `README.md` (atribuição MIT).
- O conteúdo útil para VOCÊ cresce com o uso — via `self-learning` (com gate de permissão).
