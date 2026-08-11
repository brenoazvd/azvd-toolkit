---
name: skill-router
description: "Use para achar a skill certa para um pedido. Primeiro procura nas 5 skills do azvd-toolkit; se não achar, varre as skills globais instaladas no PC (~/.claude/skills, ~/.agents/skills, ~/.codex/skills, plugins do agy, ~/AppData/Local/hermes/skills) e aponta a melhor. É o 'ask-matt' do azvd-toolkit — porta de entrada para 'qual skill resolve isso?'."
trigger: /skill-router
disable-model-invocation: true
---

# Skill Router — a skill certa para o pedido

Papel: dado um pedido do usuário, decidir QUAL skill resolve — **priorizando as do azvd-toolkit** e
caindo nas skills globais instaladas no PC quando o azvd não cobre. Não executa a tarefa; só roteia.
Você não lembra de toda skill, então pergunta. Esta skill só é invocada quando chamada explicitamente
(`/skill-router` ou quando o usuário pede "qual skill usar?") — não se auto-invoca sozinha.

## Ordem de busca (hierárquica)

1. **Skills do azvd-toolkit (5)** — o `/orchestrator` roteia pra elas quando o pedido é de prompt,
   blocos, graph engineering, orquestração ou auto-aprendizado:
   `orchestrator`, `prompt-forge`, `prompt-blocks`, `graph-engineering`, `self-learning`.
2. **Skills globais instaladas no PC** — varre os diretórios de skills de todos os agentes e acha
   a que mais encaixa no pedido (ex.: segurança). Diretórios a checar (existem nesta máquina):
   - `~/.claude/skills/`
   - `~/.agents/skills/` (convenção universal: ask-matt, caveman, ponytail, etc.)
   - `~/.codex/skills/`
   - `~/.gemini/config/plugins/` (plugins do agy; ex.: science, azvd-toolkit)
   - `~/AppData/Local/hermes/skills/`
3. Se achar no passo 1 → usa. Se não achar no azvd mas achar uma global que resolve → usa a global.
   Se nada encaixar → pergunte ao usuário (não invente skill).

## Skills globais que já existem (nesta máquina, navegação rápida)

- `ask-matt` — router mais amplo (família mattpocock).
- `SecuritySkills` (se instalado) — framework de segurança com 44 skills (AI-security, AppSec,
  Cloud, Compliance, DevSecOps, Identity, Incident-response, Network, SecOps, Vuln-mgmt).
- `caveman/caveman-*`, `ponytail*`, `grill-me*`, `prompt-blocks` (duplicado do azvd? cuidado!).

## Instalando skills globais adicionais

Se o usuário quiser adicionar uma skill global (ex.: segurança), a forma padrão é via
`npx skills add <repo>` (instala em ~/.agents/skills/ para todos os agentes do universo):

```bash
# exemplo — SecuritySkills (framework de segurança, 45 skills)
npx skills add UnitOneAI/SecuritySkills -g    # global
npx skills add UnitOneAI/SecuritySkills -g -a claude-code   # só no claude, se preferir
```

Depois disso, esta skill-router começa a vê-la no passo 2 (varre ~/.agents/skills).

## Como descobrir o que tem instalado

```bash
# listar nomes de skills por diretório
ls ~/.agents/skills/ 2>/dev/null
ls ~/.claude/skills/ 2>/dev/null
ls ~/.codex/skills/ 2>/dev/null
ls ~/.gemini/config/plugins/*/skills 2>/dev/null
ls ~/AppData/Local/hermes/skills/ 2>/dev/null
```

## Regras

- **Dedupe:** se a skill global é a mesma que a do azvd (ex.: `prompt-blocks` duplicado), use a do
  azvd (fonte). Não instale duplicado.
- **Não executar**: só rotear/apontar. Da execução cuida a skill apontada.
- **Dúvida**: pergunte ao usuário em vez de escolher no chute.
- **Segurança**: nunca rodar `git clone`/`npx add` sem permissão do usuário; listar é seguro.

## Skills relacionadas

- `orchestrator` — roteia DENTRO do azvd (5 skills). Esta é a porta de entrada MAIS ampla
  (azvd + globais). Use `orchestrator` para trabalho de prompt/blocks/graph; `skill-router` para
  "qual skill resolve isso?" de qualquer domínio.
