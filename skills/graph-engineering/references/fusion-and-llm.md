# Fusão de Conhecimento & Servindo o Grafo para LLMs
*(Aulas 8-9 do curso, traduzidas e adaptadas)*

## Conteúdo
- [Por que a fusão é a etapa decisiva](#por-que-a-fusão-é-a-etapa-decisiva)
- [O pipeline de fusão](#o-pipeline-de-fusão)
- [Pareamento de ontologias](#pareamento-de-ontologias)
- [KG para LLM](#kg-para-llm)
- [LLM para KG](#llm-para-kg)
- [Loop de grafo como memória](#loop-de-grafo-como-memória)

## Por que a fusão é a etapa decisiva

(Aula 8.) Toda passada de extração produz duplicatas: "SEU", "Southeast University",
"东南大学" são a mesma universidade; "Bob Smith (doc 3)" e "Robert Smith (doc 41)" podem ou não
ser a mesma pessoa. Um grafo sem fusão responde a consultas multi-hop (múltiplos saltos) incorretamente *com confiança* — os caminhos
se quebram nas fronteiras das duplicatas. Essa é a razão nº 1 pela qual projetos de KG do mundo real produzem algo
inútil.

## O pipeline de fusão

Três passos, a partir do material de pareamento de entidades em grande escala (*large-scale entity matching*) do curso:

1. **Blocking (bloqueio/agrupamento)** — nunca compare todos os pares (O(n²)). Agrupe candidatos de forma barata primeiro: mesmo tipo +
   (token compartilhado | expansão de sigla correspondente | similaridade de embedding acima do limiar | mesma
   chave normalizada). Apenas pares dentro de um bloco passam por comparação completa.
2. **Matching (pareamento/correspondência)** — pontue pares candidatos com base em evidências em camadas:
   - Camada de string: correspondência normalizada/alias/sigla.
   - Camada de atributos: atributos compatíveis (mesmo ano de fundação, mesmo domínio de e-mail).
   - **Camada de estrutura** (a ênfase do curso, e o que a deduplicação ingênua perde): compare
     vizinhanças — dois nós "J. Smith" que compartilham 3 coautores e uma afiliação são a mesma
     pessoa; nomes idênticos com vizinhanças disjuntas não são.
   - Adjudicação por LLM apenas para a faixa intermediária ambígua (heurísticas baratas para os casos claros;
     o modelo vê atributos + vizinhanças + citações de evidência de ambos os nós).
3. **Política de merge (fusão)** — código determinístico, não julgamento do modelo: mantenha o nome canônico, una
   aliases e arestas, mantenha valores de atributos por fonte com proveniência quando houver conflito
   (NÃO sobrescreva silenciosamente — valores conflitantes são sinais), registre `merged_from` para desfazer.

Limiares: auto-merge apenas acima de alta confiança; rejeição automática abaixo do limiar de baixa confiança; enfileire a faixa intermediária para
adjudicação por LLM ou revisão humana. Um merge errôneo é muito mais prejudicial do que um merge omitido —
ela funde silenciosamente todo o conjunto de arestas de duas entidades.

## Pareamento de ontologias

Ao fundir dois grafos (não apenas instâncias), alinhe os schemas primeiro: mapeie tipos de entidades e
tipos de relações entre as fontes (curso: 本体匹配 + ajuste de correspondência). Ordem prática — alinhe
os tipos por nome+definição com um LLM, verifique com a sobreposição de instâncias (se os nós `Firm` da fonte A
corresponderem em sua maioria aos nós `Company` da fonte B, o mapeamento de tipos está confirmado), e então traduza
as arestas da fonte B por meio do mapeamento antes da fusão de instâncias.

## KG para LLM

(Aula 9, direção 1.) Fazendo o grafo reduzir alucinações e expandir o contexto:

- **Recuperação GraphRAG:** vincule entidades da consulta (*entity-linking*) → expanda k saltos (k=1-2; além de 2 torna-se ruído
  sem re-ranqueamento) → serialize o subgrafo como triplas/caminhos compactos com proveniência →
  esse é o contexto do LLM. As respostas citam fatos do grafo, não palpites.
- **Serialização que funciona:** linhas no formato `(head)-[REL {time, source}]->(tail)`, agrupadas por entidade
  principal (*head*), deduplicadas. Tabelas de triplas superam resumos em prosa — o LLM pode citar fatos exatos.
- **Perguntas multi-hop:** recupere *caminhos* entre as entidades da consulta, não vizinhanças
  ao redor de cada uma. O caminho É o esqueleto da resposta; o LLM o narra.
- **Resumos de comunidade** para perguntas do tipo "quais são os grandes temas": agrupe o grafo em clusters,
  resuma por cluster offline, recupere os resumos no momento da consulta.

## LLM para KG

(Aula 9, direção 2.) Já embutido nas etapas 3-8 do pipeline: indução de schema
(com filtragem humana), extração (com restrições de ontologia + citações de evidência), adjudicação de fusão
(apenas na faixa intermediária). O enquadramento do curso a ser mantido: o LLM é um componente dentro
de cada etapa com validação ao seu redor — não um substituto para o pipeline.

## Loop de grafo como memória

Para agentes que acumulam conhecimento ao longo de sessões:

1. Após cada sessão/tarefa, execute a extração (etapas 4-6) sobre novas informações com a mesma
   ontologia.
2. Mescle novos fatos no grafo existente (etapa 8) — mesmo mecanismo de blocking/matching/merge,
   incremental.
3. No início da sessão ou sob demanda, recupere via GraphRAG (acima) em vez de despejar todo
   o grafo no contexto.
4. **Tratamento de contradições:** quando um novo fato conflitar com um armazenado, mantenha ambos com
   tempo + proveniência e prefira o mais recente no momento da recuperação — fatos mudam ("trabalha em X"
   torna-se desatualizado); o grafo deve registrar a mudança, não resistir a ela.
5. Passada periódica de higiene: re-execute a fusão em todo o grafo e re-pontue confianças desatualizadas.
   Grafos de memória sem manutenção deterioram da mesma forma que extrações sem fusão.
