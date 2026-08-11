# CONTEXT.md — contexto do repositório azvd-toolkit

O **azvd-toolkit** é um toolkit open-source de skills para trabalhar com IA: orquestração,
prompt engineering, graph engineering e auto-aprendizado. Repo: `brenoazvd/azvd-toolkit`.

## Visão de 1 parágrafo

Criado a partir de experiências reais de orquestração multi-agente (em um contexto de dados
educacionais), o toolkit transforma o que funciona em **skills reutilizáveis e adaptativas**.
São 5 skills que "se conversam": um roteador (`orchestrator`) distribui pedidos; o
`prompt-forge` monta prompts por entrevista (anatomia Tarefa/Método/Meta); o `prompt-blocks`
colhe blocos de prompt validados; o `graph-engineering` cobre knowledge graphs + task graphs;
e o `self-learning` faz o toolkit se adaptar com base no uso, com gate de permissão.

## Arquitetura do repo

```
azvd-toolkit/
├── skills/                    ← conteúdo real das skills (fonte única; não é .claude/skills/)
│   ├── orchestrator/SKILL.md
│   ├── prompt-forge/SKILL.md
│   ├── prompt-blocks/SKILL.md + blocks/ (B1-B11) + blocks/local/ (gitignored)
│   ├── graph-engineering/SKILL.md + references/
│   └── self-learning/SKILL.md
├── .claude-plugin/            ← só o wrapper do marketplace do Claude (plugin.json, marketplace.json)
├── AGENTS.md                  ← regras de contribuição para qualquer IA
├── RULES.md                   ← regras operacionais do mantenedor (sync/permissões)
├── CONTEXT.md                 ← este arquivo
├── sync.sh                    ← propaga skills para as 4 IAs
└── README.md
```

## Nota sobre o nome `.claude/`

Antes as skills ficavam em `.claude/skills/`. Agora vivem em `skills/` — nome neutro e universal
(lido por qualquer agente e pelo `npx skills`). O `.claude-plugin/` permanece apenas como o
pacote de instalação do marketplace do Claude Code, **não** como o conteúdo das skills.

## Distribuição (sync)

- `sync.sh` copia `skills/` para as 4 IAs: Hermes (`~/AppData/Local/hermes/skills/`), agy
  (`~/.gemini/config/plugins/azvd-toolkit/skills/`), claude (marketplace `azvd`), e o dir
  universal (`npx skills`, p/ codex/cursor/antigravity-cli).
- `skills/prompt-blocks/blocks/local/` é **gitignored e preservado** em cada IA pelo sync
  (não propaga, não apaga). É por-usuário.
- `RULES.md` tem as regras operacionais de sync/permissões; `AGENTS.md` as de conteúdo.
