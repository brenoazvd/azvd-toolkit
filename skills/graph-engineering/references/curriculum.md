# Curso Fonte — Currículo Traduzido

Curso de pós-graduação da Southeast University《知识图谱》(Knowledge Graphs), Prof. Peng Wang.
Repositório: https://github.com/npubird/KnowledgeGraphCourse (4.4K★, em andamento desde 2019, atualizado anualmente).
Todos os conjuntos de slides são PDFs em chinês na raiz do repositório; os slides de 2025 possuem o prefixo `2025-pub-`.
Este arquivo é o mapa em português da edição de 2025.

## Aula 1 — Knowledge Graphs: Teoria, Tecnologia, Prática, Desafios
*(2025-pub-1 …A.pdf / …B.pdf)*

- 1.1 A visão de KG sobre inteligência cognitiva: o que o "conhecimento" adiciona à cognição de máquina;
  a essência de um knowledge graph (grafo de conhecimento); como os KGs evoluíram (redes semânticas → sistemas especialistas →
  Web Semântica → Knowledge Graph do Google de 2012); KG vs aprendizado profundo (*deep learning*); KG vs bases de
  conhecimento tradicionais vs bancos de dados; cenários de aplicação; valor central.
- 1.2 A pilha de tecnologia, que o restante do curso expande:
  **extração de conhecimento → fusão de conhecimento → aprendizado de representação → raciocínio → armazenamento.**
- 1.3 Gargalos: aquisição de conhecimento (cobertura/custo), qualidade do conhecimento
  (ruído/consistência) e aplicação inteligente (tornar o grafo realmente útil).

## Aula 2 — Representação do Conhecimento (2025-pub-2)

Conceitos e inventário de métodos: redes semânticas, sistemas de produção (regras), sistemas
de frames (quadros), grafos conceituais, análise formal de conceitos, lógica de descrição, ontologias,
linguagens de ontologia (RDF/RDFS/OWL) e aprendizado de representação de KG (embeddings).
→ Destilado em [modeling.md](modeling.md).

## Aula 3 — Modelagem do Conhecimento (2025-pub-3)

Imersão em ontologias: metodologia de engenharia de ontologias, aprendizado de ontologia (indução
semi-automática de schema), ferramentas de modelagem (Protégé et al.) e prática de modelagem.
→ Destilado em [modeling.md](modeling.md).

## Aula 4 — Extração de Conhecimento: Problemas & Métodos (2025-pub-4)

Análise do problema (cenários, por que extrair é difícil) e métodos por tipo de fonte:
dados estruturados (mapeamento D2R), dados semiestruturados (wrappers, tabelas web) e
texto não estruturado (o pipeline de PNL/NLP que as três aulas seguintes cobrem).
→ Destilado em [extraction.md](extraction.md).

## Aula 5 — Reconhecimento de Entidades (2025-pub-5, mais slides de fronteira 2025 5-1)

A escada completa de métodos: baseado em regras/dicionário → ML clássico (HMM/CRF) → aprendizado profundo
(BiLSTM-CRF) → semi-supervisionado → aprendizado por transferência → modelos pré-treinados (era BERT) →
paradigma da era LLM. Slides de progresso na fronteira atualizados anualmente.
→ Destilado em [extraction.md](extraction.md).

## Aula 6 — Extração de Relações (2025-pub-6)

Relações semânticas, design de features (características), conjuntos de dados de benchmark; métodos: baseado em templates/padrões,
supervisionado, fracamente supervisionado, supervisão distante (*distant supervision*), não supervisionado (open IE) e
abordagens de aprendizado profundo/por reforço.
→ Destilado em [extraction.md](extraction.md).

## Aula 7 — Extração de Eventos (2024-pub-7; inclui aula da indústria com a Huawei "Dos Paradigmas Clássicos aos de LLM")

Conceitos de eventos (gatilho/*trigger*, argumentos, tipos de eventos), métodos de extração, walkthrough de um sistema
de extração de eventos no domínio financeiro e grafos de lógica de eventos (事理图谱) — grafos cujos nós são
eventos e as arestas são elos causais/temporais/condicionais.
→ Destilado em [extraction.md](extraction.md).

## Aula 8 — Fusão de Conhecimento (2024-pub-8, mais slides de progresso na fronteira)

Heterogeneidade do conhecimento; pareamento de ontologias (*ontology matching*); extração e ajuste de correspondências;
pareamento de instâncias (*instance matching*); pareamento de entidades em grande escala (*blocking*, escalonamento); estudos de caso reais de fusão.
→ Destilado em [fusion-and-llm.md](fusion-and-llm.md).

## Aula 9 — Knowledge Graphs × Modelos de Linguagem de Grande Porte (2025-pub-9)

Ambas as direções: **KG para LLM** (ancoragem/*grounding*, recuperação/*retrieval*, redução de alucinações, memória
estruturada) e **LLM para KG** (extração impulsionada por LLM, indução de schema, fusão). A edição
de 2024 também inclui slides sobre ChatGPT para extração de informação, engenharia de prompt e
avaliação de qualidade.
→ Destilado em [fusion-and-llm.md](fusion-and-llm.md).

## Como o curso se mapeia para o pipeline de 9 etapas desta skill

| Etapa da skill | Fonte no curso |
|---|---|
| 1 Escopo & valor | Aulas 1.1, 1.3 |
| 2 Escolha de representação | Aula 2 |
| 3 Modelagem de ontologia | Aula 3 |
| 4 Extração de entidades | Aulas 4-5 |
| 5 Extração de relações | Aula 6 |
| 6 Extração de eventos | Aula 7 |
| 7 Gate de qualidade | Aula 1.3 + material de avaliação na 9 |
| 8 Fusão de conhecimento | Aula 8 |
| 9 Servir aos LLMs | Aula 9 |
