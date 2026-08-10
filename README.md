# azvd-toolkit

Toolkit pessoal de skills para trabalhar com IA — a versão **versionada em repo** do que foi validado em campo (orquestração agêntica claude CLI + agy, graph engineering, padrões de prompt com lição de incidente).

## As 4 skills

| Skill | Função | Trigger |
|---|---|---|
| `orchestrator` | Roteador: decide qual skill usar e encadeia | `/orchestrator` |
| `prompt-forge` | Monta prompts por entrevista interativa (anatomia Tarefa/Método/Meta) | `/prompt-forge` |
| `prompt-blocks` | Biblioteca de blocos comprovados, cada um com a lição de incidente que o originou | `/prompt-blocks` |
| `graph-engineering` | Task graphs (orquestração multi-agente) + knowledge graphs (pipeline de 9 etapas) | `/graph-engineering` |

## Como as skills se conversam

```mermaid
flowchart LR
  U[Usuário] --> O[orchestrator]
  O --> PF[prompt-forge]
  O --> GE[graph-engineering]
  O --> PB[prompt-blocks]
  PF --> PB
  GE --> PF
  PB --> PF
```

- `orchestrator` roteia e encadeia (primeira parada de qualquer pedido).
- `prompt-forge` monta o prompt e compõe com blocos de `prompt-blocks`.
- `graph-engineering` planeja a orquestração (fan-out, diamond, human gate); cada ticket vira prompt no `prompt-forge`.
- `prompt-blocks` guarda as lições (outcome) que alimentam todas.

**Regra central:** prompt único gigante buga a IA — tarefa multi-etapa = task graph + um prompt por ticket.

## Origem

Validado em produção no ecossistema SaaS OEP (2026-08): tickets autocontidos, padrão analyzer→reviewer→verify (agy analisa → claude opus revisa → orquestrador confere), contrato de memória OKF, teste de decisão autônomo. `graph-engineering` é réplica do repo [codejunkie99/graph-engineering](https://github.com/codejunkie99/graph-engineering) (MIT), com a metade de task graphs vinda do trabalho DeepMind×MIT.

## Instalar como plugin (o jeito "marketplace" — como os caras fazem)

Este repo já é um **plugin + marketplace** válido para o Claude Code (manifests em
`.claude-plugin/plugin.json` e `.claude-plugin/marketplace.json`, no mesmo formato dos
marketplaces oficiais). Depois de publicado no GitHub:

```bash
# 1. No Claude Code — adicionar o marketplace e instalar
claude plugin marketplace add azvd https://github.com/brenoazvd/azvd-toolkit
claude plugin install azvd-toolkit@azvd

# 2. Receber atualizações depois de mudanças no repo
claude plugin marketplace update azvd

# 3. Antigravity (agy) — instalar o plugin direto
agy plugin install azvd-toolkit@https://github.com/brenoazvd/azvd-toolkit
#    ou ponte: agy plugin import claude   (importa plugins do Claude Code)
```

### ⚠️ Bug conhecido: `claude plugin marketplace add` (v2.1.224–226)

O comando está quebrado nesta versão — falha com `Invalid marketplace source format`
mesmo para o marketplace oficial (`anthropics/claude-plugins-official`), em todas as
formas (`owner/repo`, `https://...`, `./path`). Contorno **validado**:

```bash
# 1. Registrar o marketplace manualmente (com backup)
cp ~/.claude/plugins/known_marketplaces.json ~/.claude/plugins/known_marketplaces.json.bak
# adicionar no JSON (mesmo shape das entradas existentes):
#   "azvd": {
#     "source": {"source": "github", "repo": "brenoazvd/azvd-toolkit"},
#     "installLocation": "C:\\Users\\<user>\\.claude\\plugins\\marketplaces\\azvd",
#     "lastUpdated": "<ISO-8601>"
#   }
# 2. Clonar/validar + instalar
claude plugin marketplace update azvd
claude plugin install azvd-toolkit@azvd
```

### Antigravity (agy) — status

Instalação validada como **plugin** (formato nativo do agy):

```bash
# 1. Criar o plugin: ~/.gemini/config/plugins/azvd-toolkit/
#    plugin.json  → {"name": "azvd-toolkit"}   (UTF-8 SEM BOM — BOM quebra o parser)
#    skills/      → as 4 skills (pastas REAIS, não junction — o painel /skills não
#                   atravessa junction; o validator atravessa, o painel não)
# 2. Validar (opcional):
agy plugin validate 'C:\Users\<user>\.gemini\config\plugins\azvd-toolkit'   # espera "skills: 4 processed"
# 3. Habilitar (escreve em ~/.gemini/config/config.json: {"azvd-toolkit": {"enabled": true}})
agy plugin enable azvd-toolkit
```

Descobertas do caminho (2026-08-10):
- O painel `/skills` do agy lista skills de plugin no formato `plugin:skill` (ex. `science:quickgo-database`) e **escaneia pastas reais** — junction é visível pro `agy plugin validate` mas NÃO pro painel.
- `~/.gemini/config/skills/` (junctions → `.agents/skills/<nome>`) é onde ficam as skills "avulsas" do agy; o `skills-lock.json` registra fontes github (hash `computedHash` não é sha256 do arquivo — algoritmo proprietário, não replicar à mão).
- `npx skills add` instala no dir universal `~/.agents/skills/` (agentes `.agents/skills` = codex/cursor/antigravity-cli) — o agy NÃO lê de lá.

### Hermes (este assistente)

```bash
# Cópia direta para o diretório de skills do Hermes (fica ativo na próxima sessão):
cp -r .claude/skills/{orchestrator,prompt-forge,prompt-blocks,graph-engineering} \
      ~/AppData/Local/hermes/skills/
```

## Instalar localmente (sem GitHub — junction ou cópia)

```bash
# opção 1 — junction (fonte = este repo; edita aqui, vale no ~/.claude)
# cmd (Windows), uma por skill:
mklink /J "%USERPROFILE%\.claude\skills\orchestrator"       "%~dp0.claude\skills\orchestrator"
mklink /J "%USERPROFILE%\.claude\skills\prompt-forge"       "%~dp0.claude\skills\prompt-forge"
mklink /J "%USERPROFILE%\.claude\skills\prompt-blocks"      "%~dp0.claude\skills\prompt-blocks"
mklink /J "%USERPROFILE%\.claude\skills\graph-engineering"  "%~dp0.claude\skills\graph-engineering"

# opção 2 — cópia (simples, mas dessincroniza):
cp -r .claude/skills/* ~/.claude/skills/
```
