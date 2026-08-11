# Extração de Conhecimento: Entidades, Relações, Eventos
*(Aulas 4-7 do curso, traduzidas e adaptadas)*

## Conteúdo
- [Extração por tipo de fonte](#extração-por-tipo-de-fonte)
- [Extração de entidades](#extração-de-entidades)
- [Extração de relações](#extração-de-relações)
- [Extração de eventos](#extração-de-eventos)
- [Padrão de prompt de extração com LLM](#padrão-de-prompt-de-extração-com-llm)
- [Modos de falha](#modos-de-falha)

## Extração por tipo de fonte

(Aula 4.) Mapeie o método para a estrutura da fonte — usar PNL/NLP em dados que já são estruturados
é o desperdício clássico de iniciante:

- **Estruturados (bancos de dados, CSVs, APIs):** mapeamento direto, sem PNL. Escreva um mapeamento por fonte
  de colunas → tipos de ontologia (a ideia D2R do curso). Código determinístico, não LLM.
- **Semi-estruturados (tabelas HTML, infoboxes, wikis, blobs JSON):** wrappers/parsers por
  família de layout; LLM apenas para células desorganizadas.
- **Não estruturados (texto, transcrições, PDFs):** o pipeline NER → RE → EE abaixo (reconhecimento de entidades nomeadas, extração de relações e extração de eventos).

## Extração de entidades

(Escada de métodos da Aula 5, comprimida para a era LLM.)

O curso delineia: regras/dicionários → HMM/CRF → BiLSTM-CRF → semi-supervisionado → transferência →
pré-treinado (BERT) → LLM. O que sobrevive na prática:

1. **Extração por dicionário/regras primeiro** para vocabulários fechados que você já possui (seus nomes
   de produtos, lista de equipe, símbolos de ações, valores de enum da ontologia). Correspondência exata supera qualquer modelo —
   gratuito, determinístico, 100% de precisão.
2. **Extração com LLM** para tudo que for aberto, com os tipos de entidade da ontologia + definições
   + exemplos no prompt (padrão abaixo).
3. **Capture sempre:** forma de superfície (*surface form*), forma canônica (*canonical form* — melhor palpite), tipo, ponteiro de origem
   (ID do doc + span de caracteres ou sentença), confiança.

Lição clássica que ainda se aplica à saída de LLM: **menções aninhadas e descontínuas**
("O professor da University of California, Berkeley, John Smith" contém uma ORG dentro do contexto
de uma PERSON) e **ambiguidade de tipo** ("Apple") causam a maioria dos erros — exija que o modelo cite sua
sentença de evidência, o que força a desambiguação a partir do contexto.

## Extração de relações

(Aula 6.) Inventário do curso: baseado em templates → supervisionado → fracamente supervisionado → supervisão
distante (*distant supervision*) → open IE não supervisionado → métodos profundos/RL. Destilação moderna:

- Extraia relações **apenas entre entidades que passaram pela etapa 4** — nunca permita que a extração
  de relações invente novas entidades. Essa restrição única elimina a maioria dos erros em cascata.
- Restrinja a saída à lista de relações da ontologia; **valide domínio/alcance (domain/range) em código**
  (uma aresta `EMPLOYED_BY` de Org → Org é automaticamente rejeitada).
- A intuição central da supervisão distante ainda importa para a avaliação: se duas entidades ocorrem juntas,
  o modelo afirmará alegremente a relação sugerida pelo conhecimento prévio. Proteção: exija uma citação
  de evidência que *afirme* diretamente a relação, não apenas a co-ocorrência ("Musk discutiu o Twitter" não é
  `OWNS`).
- Mantenha relações não modeladas mas repetidas em uma lista secundária `candidate_relations` — revise semanalmente;
  promova as reais para a ontologia em vez de forçá-las em tipos incorretos.

## Extração de eventos

(Aula 7.) Use quando o domínio for dinâmico — notícias, incidentes, transações, rodadas de investimento.

Um evento = **gatilho (*trigger*)** (a palavra/frase que o sinaliza) + **argumentos tipados** (participantes,
tempo, local, valores) + **tipo de evento** da ontologia. Eventos são nós de primeira classe com
arestas para seus argumentos — nunca achate um evento de 4 argumentos em 6 arestas pareadas (você perde
qual aquisição ocorreu a qual preço).

O estudo de caso financeiro do curso se generaliza: defina schemas de argumentos por tipo de evento
(`Acquisition: {acquirer, target, price, date}`), extraia para esse schema, rejeite eventos
que não possuem a evidência do gatilho.

**Grafos de lógica de eventos (事理图谱):** uma ideia distinta do curso que vale a pena conhecer — grafos
cujos nós são eventos e as arestas são relações causais/temporais/condicionais entre eventos
("aumento de juros → liquidação de títulos"). Construa um quando o usuário perguntar "o que leva a quê", e não apenas
"o que está relacionado a quê".

## Padrão de prompt de extração com LLM

Uma passada por etapa (entidades, depois relações, depois eventos), não um mega-prompt único:

```
You are extracting knowledge for a graph with this ontology:
<ontology>   (verbatim from the project's ontology file)

From the text below, extract every entity matching the ontology types.
For each: {surface, canonical, type, evidence: "<exact sentence>", confidence: high|med|low}
Rules:
- Only types from the ontology. Unknown-but-recurring concepts → list separately under "candidates".
- Evidence must be a verbatim quote containing the mention.
- Do not merge distinct mentions; deduplication happens later.
<text>
```

A passada de relações adiciona: a lista de entidades reconhecidas, o inventário de relações com domínio/alcance (domain/range) e
"afirme apenas relações que a sentença de evidência declara diretamente."

Divisão em blocos (*chunking*): sobreponha blocos em 10-15% para que entidades nas fronteiras das sentenças não sejam perdidas; extraia por
bloco; a fusão (etapa 8) reconcilia.

## Modos de falha

| Sintoma | Causa | Correção |
|---|---|---|
| Grafo cheio de nós `Concept`/`Thing` | Extração sem ontologia | Etapa 3 primeiro; re-extraia |
| Mesma pessoa como 4 nós | Regra de forma canônica pulada | Defina a regra na ontologia; passada de fusão |
| Relações incorretas com alta confiança | Co-ocorrência tratada como afirmação | Exigência de citação de evidência + validação de domínio/alcance |
| Eventos achatados em sopa de arestas | Sem schema de evento | Nós de evento de primeira classe com schemas de argumentos |
| Precisão colapsa em escala | Desvio de prompt (*prompt drift*) entre tipos de doc | Prompts por tipo de fonte; gate da etapa 7 por fonte |
