# RULES.md — regras para quem mexe neste repo

Este é um repositório **público**. Qualquer pessoa ou IA que editar aqui deve seguir estas regras.

## Publicação

1. **Nada de dados pessoais ou de projeto específico.** Sem nomes de cliente, repositório
   interno, incidente com identificador (ex.: "T15.2"), e-mail, path de máquina local, ou
   credencial.
2. **Modelos de IA sempre genéricos.** Use "modelo mais forte" ou "modelo rápido/leve" — nunca
   "claude opus", "Gemini Flash", etc. O usuário escolhe o seu. Se precisar de recomendação,
   **pergunte ao usuário qual agente/modelo ele prefere**.
3. **Idioma principal: PT-BR.** Skills, blocos e README em português brasileiro.
4. **Frontmatter no estilo do usuário (default: OKF v0.2).** Pergunte/detecte o estilo antes
   de criar arquivo novo.

## Conteúdo

5. **Filtro F1-F5 antes de criar bloco novo.** Recorrência + evidência + custo/especificidade —
   se não passar, não cria. Arquivo novo = permissão do usuário antes.
6. **Skills relacionadas:** aponte para skills que EXISTEM neste toolkit. Não referencie skills
   externas como se fossem parte dele.
7. **Um bloco = um arquivo em `blocks/`.** SKILL.md é só catálogo — nunca cresce com conteúdo
   de bloco novo. `blocks/local/` é espaço PESSOAL do usuário (gitignored, não versionado):
   quem clona o repo cria o próprio `local/` vazio e o evolve com o uso; nunca propagar no sync.
8. **Traduções:** todo conteúdo público em PT-BR. Referências acadêmicas originais podem ficar
   no idioma fonte (com nota).

## Operação

9. **Não commitar arquivos de distribuição local** (sync.sh, backups, temporários).
10. **Antes de alterar skills, validar** com as ferramentas nativas de cada agente e propagar.
11. **sync.sh só roda localmente** — está no .gitignore, não vai pro repo público.
12. **Ao atualizar conteúdo puxado de upstream** (graph-engineering, self-learning, vibecode):
    preservar traduções PT-BR e seções adaptadas ao toolkit; nunca substituir cegamente.
