---
type: PromptBlock
title: B11 · Roteamento por tarefa (modelo, esforço, modelo de decisão e sub-agente)
description: Pré-regras e roteiro de 5 perguntas para cada tarefa ou papel de um time de agentes — precisa de LLM ou basta um modelo de decisão? vale sub-agente? qual categoria de modelo (leve/forte/mais forte)? qual nível de esforço? como testar antes de baixar? Sem nome de modelo; as fontes oficiais ficam em sources e se atualizam sozinhas. O nome de cada categoria fica no bloco local/ do usuário.
tags:
  - prompt
  - routing
  - modelos
  - esforco
  - custo
  - orquestracao
status: active
generated:
  by: brenoazvd
  at: 2026-08-11
updated:
  at: 2026-10-09
  why: "Run de construção com time de agentes: medir o consumo por papel mostrou o crítico com 57% do gasto; o nível de esforço moveu mais a conta que o modelo"
stale_after: 2027-04-01
sources:
  - https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence (ordem das alavancas; custo por tarefa concluída; varrer 2-3 níveis de esforço)
  - https://platform.claude.com/docs/en/build-with-claude/effort (níveis por modelo; nível baixo em instrução longa pula checagem)
  - https://code.claude.com/docs/en/sub-agents (modelo e esforço por sub-agente)
  - https://code.claude.com/docs/en/workflows (tokens por agente na tela de progresso; testar numa fatia antes)
  - https://code.claude.com/docs/en/model-config (modelo padrão de sub-agente, troca por fase, modelo de reserva, esforço por sub-agente)
  - https://code.claude.com/docs/en/advisor (consultor nos pontos de decisão; vale na assinatura; sub-agentes herdam)
  - https://docs.typesafe.ai (exemplo de modelo de decisão tipado — escolha, sim/não e nota com probabilidade)
  - RouterPatterns OEP (leve analisa → forte revisa → humano confere)
  - awesome-model-routing (RouteLLM, ClawRouter, Agent-as-a-Router)
  - Prompt-Engineering-Guide (técnicas p/ esforço)
  - https://docs.typesafe.ai/patterns/confidence-routing.md (limiar de confiança por risco da ação)
  - hermes-jev-skills, jev-router e agent-router (ideias copiadas sem instalar — pré-regras, escalada por evidência, cache; auditoria de 09/10/2026)
---

# B11 · Roteamento por tarefa (modelo, esforço, modelo de decisão e sub-agente)

Texto pronto (cole no prompt do orquestrador, ou use para decidir cada papel de um time):

```
ROTEAMENTO POR TAREFA — responda as 5 perguntas para cada tarefa ou papel, nesta ordem.

0. PRÉ-REGRAS (valem antes de tudo):
   - O usuário nomeou o modelo ou o nível → use o dele.
   - Tarefa que toca senha, chave, permissão, pagamento ou exclusão de dado → começa em forte, nível alto.
   - Falhou 2 vezes seguidas no mesmo item → sobe UM degrau, nunca dois: primeiro o nível (até o máximo),
     depois a categoria. As 2 falhas já são a prova de ganho que o nível máximo exige. Subir só com
     evidência (teste falhando, crítica, confiança baixa), nunca "porque existe um mais forte".
   - Antes de pedir opinião a um modelo, rode a conferência determinística que existir (teste, linter,
     tipo, diff). Ela vence opinião.
   - Confiança baixa = abaixo de 0,6. Ação de alto risco (bloquear, apagar, cobrar, publicar) só com
     confiança de 0,85 ou mais; abaixo disso, pergunte ao humano em vez de adivinhar. Vale também para
     quem só classifica, quando a ação sai automática da classificação.

1. PRECISA DE LLM?
   - Mecânico e determinístico (rodar script, mover, formatar, capturar, buscar por padrão exato) → o
     SCRIPT faz; um LLM leve só orquestra e confere. Achar algo pelo SENTIDO (onde está a regra X) → LLM leve.
     Papel mecânico no modelo leve leva um ARQUIVO DE LIÇÕES: lê antes de começar, conserta sozinho até 2
     vezes e anota "sintoma → causa → o que resolveu" (todo conserto, inclusive o primeiro; timeout conta como
     falha); não resolveu → refaz no forte em nível baixo (reserva).
   - Decisão fechada sobre MUITOS itens (classificar, filtrar, ranquear, sim/não, dar nota a cada item), a partir de
     20 itens ou ~20 mil caracteres → MODELO DE DECISÃO (classificador que devolve escolha + probabilidade),
     não LLM. Um script monta os itens a partir dos arquivos (o LLM não digita item por item) e só o
     resumo volta ao contexto. Confiança baixa → LLM forte revisa; se continuar baixa, humano.
   - Dado pessoal (nome, documento, contato, saúde) NUNCA vai a serviço externo — e o serviço do modelo de
     decisão É externo, salvo se rodar local. Tire a coluna e confira o texto livre (nome citado no meio do
     comentário) antes, por script local ou modelo local, nunca mandando o texto a outro serviço para
     descobrir; se não dá para tirar, não envie. Número agregado sem identificar ninguém pode ir.
   - Criticar ou dar nota a UM item com nuance (régua de qualidade, tela, texto) → LLM, não modelo de decisão.
   - Escrever, criar, explicar, julgar com nuance → LLM. Quando os dois aparecem juntos, o padrão é o
     modelo de decisão decidir e o LLM só ESCREVER.

2. PRECISA DE SUB-AGENTE?
   - Sim, para: isolar leitura pesada do contexto principal, paralelizar partes independentes, ou julgar
     em contexto separado (quem constrói não julga o próprio trabalho). Os papéis de um time (construtor,
     crítico, captura, A/B, juiz) são sempre sub-agentes. A triagem em lote roda por script num sub-agente
     leve, para os itens brutos não entrarem no contexto principal. Escrever o texto final fica com o
     agente principal, salvo se exigir leitura pesada.
   - Não, quando a tarefa é uma cadeia só, cabe num contexto e não tem cauda de custo: faça direto.
     Tarefa pequena (corrigir um texto, um ajuste pontual) nunca vira orquestração.

3. QUAL CATEGORIA DE MODELO E QUAL NÍVEL? (tabela de consulta; a pergunta 4 explica os níveis)
   ┌───────────────────────────────────────────────┬──────────────────────┬──────────┬─────────────────────┐
   │ Papel                                         │ Categoria            │ Nível    │ Fallback            │
   ├───────────────────────────────────────────────┼──────────────────────┼──────────┼─────────────────────┤
   │ Mecânico (rodar script, capturar, mover,      │ leve                 │ médio;   │ forte, nível baixo  │
   │ formatar, buscar e ler muito)                 │                      │ alto se  │                     │
   │                                               │                      │ longo ou │                     │
   │                                               │                      │ estrito  │                     │
   │ Construir / programar                         │ forte                │ alto     │ forte, nível maior  │
   │ Primeira versão de algo sem modelo pronto     │ mais forte (só nela) │ médio    │ forte               │
   │ Criticar / conferir (contexto separado)       │ forte                │ alto     │ forte, nível maior  │
   │ Escolha binária às cegas (A/B de uma volta)   │ forte                │ médio    │ forte, nível maior  │
   │ Julgar o conjunto                             │ mais forte, raro     │ médio    │ forte               │
   │ Planejar                                      │ médio/forte          │ alto     │ mais forte          │
   │ Escrever o texto final (a partir da decisão)  │ forte                │ alto     │ mais forte          │
   │ Revisar o que o modelo de decisão marcou como │ forte                │ alto     │ humano              │
   │ confiança baixa                               │                      │          │                     │
   │ Ajuste pontual (um texto, um erro)            │ o agente principal,  │ o da     │ —                   │
   │                                               │ sem time             │ sessão   │                     │
   └───────────────────────────────────────────────┴──────────────────────┴──────────┴─────────────────────┘
   - Agente com modelo fixo no arquivo (ex.: um construtor forte) e a linha pede outra categoria (primeira
     versão no mais forte, A/B no forte quando o agente de julgamento é o mais forte)? Troque SÓ naquela
     chamada, pelo parâmetro de modelo do host; não precisa de cópia local do agente.

4. QUAL NÍVEL DE ESFORÇO? (o nível costuma mover a conta mais que o modelo — ajuste ele primeiro)
   - Leve: médio por padrão; ALTO quando a tarefa é longa (dezenas de arquivos ou passos) ou tem regra que não pode falhar (ex.: nunca
     matar processo alheio, conferir se o print saiu preto). Nível baixo em instrução longa tende a pular
     checagem e parar cedo.
   - Construir (fora a primeira versão no mais forte), escrever o texto final e revisar confiança baixa:
     alto. Ajuste pontual: o nível da sessão. Criticar: alto (o máximo só se um
     teste mostrar ganho). Escolha A/B: médio. Mais forte (primeira versão e julgar o conjunto): médio.
   - "Raro" para o julgamento do conjunto = a cada 3 a 5 voltas ou ao fim de uma etapa; o usuário ajusta.
   - Nível máximo só com prova de ganho: custa muito mais para pouco a mais. Medição de terceiros num kit
     de roteamento: nível alto custou ~1,8× o médio sem acertar mais de primeira; o alto fica para quem
     constrói, critica e escreve; o resto, médio.

5. COMO MUDAR SEM PERDER QUALIDADE?
   - Meça antes: gasto POR PAPEL (some os tokens de cada agente pelo rótulo). Ataque o papel que mais gasta.
   - Baixar modelo ou nível = teste antes (inclusive descer do máximo para o padrão): rode UMA volta com a
     configuração nova no mesmo item e compare nota e falhas com a volta anterior. Não piorou → vira
     padrão. Piorou → volta atrás e registre. ("Padrão" de cada papel = o nível da pergunta 4.) Mudou duas
     coisas juntas (ex.: nível e o que o crítico vê)? O teste vale para o pacote; o julgamento periódico do
     conjunto confere depois e, se achar o que o papel deixou passar, ele volta ao antigo.
   - Compare custo por TAREFA CONCLUÍDA, não por token: o barato que precisa de mais voltas sai caro.
   - Régua de um modelo de decisão: classificação humana feita sem regra escrita NÃO serve de gabarito.
     Meça contra um LLM forte com as mesmas regras + uma amostra conferida à mão. Se o LLM forte também
     discorda do gabarito humano, o problema é o gabarito, não o modelo.
   - Não troque modelo nem nível no meio de uma execução (perde o cache); troque entre voltas.
   - A cada lançamento de modelo, reveja a tabela: preços e diferenças mudam (fontes oficiais no topo).

ATUALIZAÇÃO (para o bloco não envelhecer em silêncio):
- Antes de aplicar, olhe a data `updated.at` do topo. Passou de 60 dias, ou saiu modelo novo desde então?
  Releia as fontes oficiais listadas em `sources` (níveis de esforço, custo, sub-agentes, recursos do host),
  ajuste a tabela e os níveis se algo mudou e atualize a data. Recurso novo do host entra na seção acima.
- O nível que cada papel usa de verdade (medido) fica no bloco local/ do usuário, com a data da medição.

USE O QUE O HOST JÁ FAZ ANTES DE INVENTAR (recursos nativos, que se atualizam com o host):
- Modelo padrão de sub-agente: o host costuma ter uma variável que define o modelo de todo sub-agente
  sem modelo declarado. Ponha a categoria forte (ou leve) ali: é a rede de segurança contra herdar o
  modelo mais caro da sessão.
- Consultor (advisor): o modelo principal trabalha e consulta um modelo mais forte só nos pontos de
  decisão (antes de escolher o caminho, quando um erro se repete, antes de declarar "terminei"). Vale em
  tarefa longa com muitos passos; em tarefa curta, não compensa. Alternativa ao "mais forte na primeira
  versão": forte executando + mais forte consultando. Teste numa volta antes de adotar (pergunta 5).
- Troca por fase: modelo mais forte só no planejamento, forte na execução.
- Modelo de reserva: quando o principal estiver sobrecarregado, o host troca sozinho só naquele turno.

Regras:
- Isto é SUGESTÃO por categoria. O nome do modelo de cada categoria é escolha do usuário (bloco local/).
- Todo agente sai com modelo e nível declarados; nunca herda o da sessão por omissão.
- Roteador que escolhe o modelo a cada mensagem não substitui esta decisão por papel; vale como
  fallback. Proxy que vê todo o tráfego (e o login) exige auditoria antes de instalar.
```

**Quando:** ao montar qualquer time de agentes ou loop (construtor, crítico, captura, juiz), ao decidir
se uma triagem em lote vai para LLM ou para um modelo de decisão, e sempre que o consumo apertar.

**Outcome:** num run de construção com time de agentes, medir o gasto por papel mostrou o crítico com 57%
do consumo (relia dezenas de prints por passo), o construtor com 18%, a escolha A/B com 8% e a captura
com 5%. A captura passou para o modelo leve em nível ALTO (é longa e tem regra que não pode falhar, com um
arquivo de lições para consertar sozinha), a escolha A/B baixou para nível médio e o nível do crítico foi
testado numa volta antes de trocar (em nível alto, vendo só as tiras montadas: nota igual ou melhor,
achou regressões reais e custou cerca de 1/3); nada disso mexeu na régua; o modelo mais forte ficou só na primeira versão de itens novos e no julgamento
do conjunto. Padrão anterior mantido: a tarefa define o modelo (leve analisa → forte revisa → humano
confere). Referência de roteamento como infra:
[awesome-model-routing](https://github.com/yenanjing/awesome-model-routing).
