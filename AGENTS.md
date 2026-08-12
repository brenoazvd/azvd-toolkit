# AGENTS.md — para qualquer IA que mexa neste repo

Este é o **azvd-toolkit**: um toolkit open-source de skills para trabalhar com IA
(orquestração, prompt engineering, graph engineering, auto-aprendizado). Nome do repo:
`brenoazvd/azvd-toolkit`.

## O que é este repo

5 skills adaptativas que qualquer agente (Claude Code, agy, Codex, Cursor, Hermes) pode usar:

| Skill | Função |
|---|---|
| `skills/orchestrator` | Roteia pedidos → skill certa; encadeia as outras |
| `skills/prompt-forge` | Forja prompts por entrevista: cada tipo (Criação/Código/Análise/Orquestração/Texto) tem um Modo que gera o prompt pronto (filosofia Gauntlet Loop) |
| `skills/prompt-blocks` | Protocolo de colheita de blocos de prompt (filtro F1-F5, OKF) + biblioteca B1-B12 |
| `skills/graph-engineering` | Knowledge graphs + task graphs (orquestração multi-agente), PT-BR |
| `skills/self-learning` | Colhe lições de sessão e atualiza as skills (com gate de permissão) |

## Como as skills se organizam (arquitetura)

- `skills/<nome>/SKILL.md` — conteúdo de cada skill (fonte única).
- `skills/prompt-blocks/blocks/` — blocos de prompt em arquivos `.md` (1 bloco = 1 arquivo), estilo OKF.
- `skills/prompt-blocks/blocks/local/` — **blocos pessoais do usuário, GITIGNORED, não versionado**.
- `.claude-plugin/` — manifests do marketplace do Claude (só para instalação por marketplace; não é o conteúdo).
- `.agents/CLAUDE.md`, `AGENTS.md` (este), `CONTEXT.md` — contexto/instruções de agente.
- `RULES.md` — regras operacionais do mantenedor (sync, permissões) — **não confundir com este**.
- `sync.sh` — propaga habilidades do repo para as 4 IAs (Hermes, agy, claude, dir universal).

## Regras de conteúdo (para qualquer agente que editar)

1. **Nada de dados pessoais ou de projeto específico.** Sem nome de cliente, repositório interno,
   incidente com identificador (ex.: "T15.2"), e-mail, path de máquina local, ou credencial.
2. **Modelos de IA sempre genéricos.** Use "modelo mais forte" / "modelo rápido/leve" — nunca
   "claude opus", "Gemini Flash", etc. **Nunca citar nome de modelo/provedor em sugestão** —
   sugerir por categoria (leve/forte/médio). O usuário escolhe o nome.
3. **Idioma principal: PT-BR.** Skills, blocos e README em português brasileiro.
4. **Um bloco = um arquivo em `blocks/`.** `SKILL.md` é catálogo — nunca cresce com conteúdo de bloco novo.
5. **Blocos novos exigem permissão** — mesmo em `blocks/` quanto em `blocks/local/`. Editar
   conteúdo de bloco existente é automático.
6. **(Opcional, padrão do repo)** frontmatter em estilo OKF v0.2 quando for criar arquivo novo.

## Fonte de verdade

- **Repo**: https://github.com/brenoazvd/azvd-toolkit
- Conteúdos puxados de upstream: ver `README.md` (atribuição MIT).
- O conteúdo útil para VOCÊ cresce com o uso — via `self-learning` (com gate de permissão).
