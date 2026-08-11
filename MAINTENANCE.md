# Manutenção do azvd-toolkit

Este arquivo é para **quem mantém** o repo (o `brenoazvd`). Usuários querem o `README.md`.

## Sync para as 4 IAs

```bash
bash sync.sh           # propaga skills/ → Hermes, agy, claude, dir universal em 1 comando
bash sync.sh --push    # git add/commit/push + propaga (cuidado: sobe o que estiver commitado)
```

O `sync.sh`:
- copia `skills/*` para Hermes (`~/AppData/Local/hermes/skills/`), agy (`~/.gemini/config/plugins/azvd-toolkit/skills/`), claude (marketplace `azvd`), e o dir universal (`npx skills`).
- **preserva** `skills/prompt-blocks/blocks/local/` (per-user, gitignored) em cada IA ao propagar.
- aplica `sed -i '/^trigger:/d'` no agy (frontmatter do agy não aceita `trigger`).

## Vigilância de upstream (cron `azvd-upstream-watch`)

Checa os 3 repos originais puxados: codejunkie99/graph-engineering, Kulaxyz/self-learning-skills,
withkynam/vibecode-pro-max-kit. Só alerta quando houve commit novo, mostrando o que mudou. Você
decide incorporar (preservando PT-BR + seções azvd). Silêncio = nada mudou.

Script guardião: `~/AppData/Local/hermes/scripts/azvd_upstream_guard.sh` (compara diff, sinaliza
marcas locais PT-BR).

## Branches e pastas

- `skills/` — código-fonte das 6 skills (fonte única).
- `skills/prompt-blocks/blocks/` — blocos B1-B11 (1 = 1 arquivo).
- `skills/prompt-blocks/blocks/local/` — blocos pessoais (gitignored, não comita).
- `.claude-plugin/` — apenas o wrapper do marketplace do Claude (não é o conteúdo).
- `AGENTS.md` — regras de contribuição para qualquer IA; `RULES.md` — regras operacionais do mantenedor.

## Bug conhecido: `claude plugin marketplace add`

Falha com `Invalid marketplace source format` no v2.1.224-226 (mesmo com repo oficial). Contorno:
editar `~/.claude/plugins/known_marketplaces.json` + `claude plugin marketplace update azvd`. Ver
`RULES.md`/histórico.

## Claude: ler skills/blocos de plugin em modo `-p` (headless)

Em modo `-p` (não interativo), o claude NÃO lê arquivos fora do working dir (ex.: as skills dos
blocos ficam em `~/.claude/plugins/cache/azvd/...`), e como não há como aprovar interativamente,
o run trava pedindo permissão. **Fix persistente**: adicionar permissão de leitura no
`~/.claude/settings.json`:

```json
{ "permissions": { "allow": ["Read(~/.claude/plugins/cache/azvd/**)"] } }
```

O `**` cobre qualquer versão do cache. Alternativas: `--add-dir <path>` (só por run) ou
`--dangerously-skip-permissions` (inseguro). Prova validada: após o fix, `claude -p` leu o bloco
B2 do cache do plugin sem `--add-dir`.

## Notas do agy (instalação manual)

- Plugin em `~/.gemini/config/plugins/azvd-toolkit/` — manifest completo + pastas REAIS (não junction) + frontmatter SEM trigger, porque o agy rejeita campos que não conhece.
- `npx skills add` instala no dir universal `~/.agents/skills/` — o agy NÃO lê de lá (usa plugins).
- Prova funcional: `agy -p "List skills"` lista as skills carregadas; o painel `/skills` é quirk de UI.

## Adicionar uma skill nova

1. Criar `skills/<nome>/SKILL.md` no padrão (frontmatter name+description, PT-BR, "Skills relacionadas").
2. Adicionar ao `SKILL_NAMES` do `sync.sh` e ao `.claude-plugin/plugin.json`.
3. Adicionar linha na tabela do `README.md` e no diagrama mermaid.
4. Rodar `bash sync.sh --push`.
