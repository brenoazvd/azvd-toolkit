# Workflows de Knowledge Graph — prontos para colar

Nove blocos, uma tarefa por bloco. O bloco 1 é a âncora: ele entrega o curso inteiro a um modelo e
faz o modelo te ensinar. Os outros oito são ferramentas de propósito único. Cada um é
autossuficiente; rode em ordem e cada um come a saída do anterior.

Curso fonte: [npubird/KnowledgeGraphCourse](https://github.com/npubird/KnowledgeGraphCourse)
Versão em skill (entregue a disciplina inteira ao seu agente): [`graph-engineering/`](graph-engineering/)

---

## 1 · A ÂNCORA — `/kg-tutor`

```
Você vai me ensinar o curso de pós-graduação de Knowledge Graph da Southeast University. Quero
terminar capaz de CONSTRUIR um, não capaz de descrever um.

COMO VOCÊ RODA ISSO

Pergunte-me três coisas e espere minhas respostas:
- o que estou construindo, ou quero construir
- meu nível: nunca toquei em um / já usei banco de grafo / li os papers
- horas por semana que eu realmente tenho

Depois proponha um caminho pelos módulos e deixe eu aprovar. Nunca ensine dois módulos numa mesma
mensagem.

Por módulo: explique a ideia em termos simples usando MEU domínio como exemplo corrente — nunca um
exemplo genérico de filmes-e-atores. Nomeie o único erro que iniciantes cometem aqui. Depois me dê
UMA tarefa de construção e PARE. Não continue até eu mostrar a saída. Quando eu mostrar, critique
antes de avançar: me diga o que quebra em 100x o volume.

OS MÓDULOS
01 conceitos — o que é um KG e quando ele é a ferramenta errada
02 representação — redes semânticas, frames, lógica descritiva, embeddings
03 ontologia — design de schema, domínios e ranges. o mais difícil, o mais durável
04 extração — roteamento de fontes por tipo
05 entidades · 06 relações · 07 eventos
08 fusão — deduplicação, alinhamento, bloqueio
09 embeddings — família TransE, e como link prediction é realmente avaliado
10 KG x LLM — GraphRAG, grounding, modelos construindo grafos

O QUE VOCÊ NÃO PODE FAZER
Não ensine métodos de 2016 como prática atual. NER por feature engineering e embeddings de
tradução são alfabetização, não ferramentas — diga isso quando chegarmos lá. Não me deixe pular
os módulos 03 ou 08; é onde projetos reais morrem. Não aceite "faz sentido" como evidência de
entendimento — me faça aplicar. Se meu projeto não precisa de grafo de verdade, diga no módulo 01
e pare o curso.

FIM DE TODA SESSÃO
Me dê uma linha que eu possa colar de volta na próxima vez para retomar: módulos cobertos, o que
construí, o que errei, o que vem a seguir.

Faça suas três perguntas agora.
```

## 2 · `/kg-scope`

```
Aja como arquiteto de knowledge graph. Quero modelar um domínio antes de escrever qualquer código.

Domínio: [DESCREVA EM 2 FRASES]
O que quero responder com ele: [3 PERGUNTAS REAIS]

Retorne:
1. 8-12 tipos de entidade, cada um com os 3-5 atributos que importam e uma nota sobre o que
   identifica uma instância de forma única
2. 5-8 tipos de relação como (tipo sujeito, predicado, tipo objeto), com cardinalidade
3. Minhas 3 perguntas reescritas como travessias sobre esses tipos
4. Qualquer coisa que minhas perguntas precisem e o schema não responda, e o que está faltando

Não escreva código. Se uma pergunta precisar de agregação em vez de travessia, diga — isso é
banco de dados, não grafo.
```

## 3 · `/kg-schema`

```
Aja como engenheiro de ontologias. Transforme este rascunho de schema numa ontologia real.

Rascunho: [COLE A SAÍDA DO /kg-scope]

Retorne:
1. Uma hierarquia de classes com relações de subclasse explícitas, no máximo 3 níveis de profundidade
2. Toda propriedade com domínio, range, e se é funcional ou inversa-funcional
3. Serialização Turtle que eu possa carregar direto no Protégé
4. Cada decisão de modelagem onde você escolheu entre duas opções defensáveis, e por quê

Reutilize schema.org ou um vocabulário existente para qualquer coisa genérica — só crie IRIs novas
para o que é específico do meu domínio. Sinalize qualquer coisa que você modelou como classe mas
que deveria ter sido instância.
```

## 4 · `/kg-extract`

```
Aja como engenheiro de extração. Desenhe o pipeline antes de eu construir.

Fontes: [LISTE-AS — ex.: 400 PDFs, uma tabela Postgres, HTML raspado]
Schema alvo: [COLE A SAÍDA DO /kg-schema]

Retorne:
1. Divida minhas fontes em estruturadas / semiestruturadas / não estruturadas, e o método de cada
   uma — as duas primeiras não deveriam precisar de modelo
2. Para o conjunto não estruturado: o prompt, o schema JSON de saída, a estratégia de chunking
3. Os 5 modos de falha mais prováveis para ESTE dado específico, com um teste de detecção para cada
4. Um protocolo de verificação manual de 50 documentos: o que amostro, o que registro, qual número
   me diz para parar de ajustar

Não proponha fine-tuning até a linha de base com prompt ter uma taxa de erro medida.
```

## 5 · `/kg-relations`

```
Aja como engenheiro de extração de relações.

Relações do schema: [COLE-AS]
Corpus: [DESCREVA-O]

Retorne:
1. Um prompt que emita apenas triplas tipadas válidas contra o meu schema, cada uma com score de
   confiança e um span de evidência verbatim
2. Uma linha de base de distant supervision: qual tabela ou lista existente eu posso alinhar ao
   meu texto para gerar pares de treino de graça, e o ruído que isso introduz
3. Regras de rejeição — as triplas a descartar antes de chegarem ao grafo
4. Como testar as duas abordagens uma contra a outra em 100 frases

Toda tripla carrega proveniência. Uma tripla sem span de evidência é uma alucinação com passos extras.
```

## 6 · `/kg-events`

```
Aja como engenheiro de extração de eventos. Quero um grafo de coisas que ACONTECERAM, não de
coisas que são.

Domínio e corpus: [DESCREVA]

Retorne:
1. Um schema de tipo de evento: gatilho, argumentos e seus papéis, âncora de tempo
2. O prompt de extração, um registro por evento, com spans de argumento
3. As arestas entre eventos — causal, temporal, condicional — e como distinguir "relatado como
   causando" de "apenas co-ocorreu"
4. Como armazenar isso para que uma consulta possa caminhar uma cadeia para trás a partir de um
   desfecho

Mantenha nós de evento separados de nós de entidade. Nunca colapse uma causa em um atributo.
```

## 7 · `/kg-fuse`

```
Aja como engenheiro de resolução de entidades. Meu grafo tem duplicatas.

Tipo de entidade e volume: [ex.: 40k registros de empresas]
Campos disponíveis: [LISTE-OS]

Retorne:
1. Uma estratégia de bloqueio para não fazer comparações n-quadrado, com a redução esperada
2. A função de matching: quais campos, qual medida de similaridade, quais pesos, qual limiar
3. Uma faixa de revisão — a faixa de score onde um HUMANO decide em vez da máquina
4. Uma política de merge: em conflito, qual fonte vence, e o que sobrevive como alias em vez de
   ser descartado
5. 10 casos difíceis do meu campo onde a abordagem ingênua falha

Merges devem ser reversíveis. Diga-me o que logar para eu poder desfazer um.
```

## 8 · `/kg-eval`

```
Aja como revisor cético do meu knowledge graph.

O que construí: [DESCREVA]
Números que estou prestes a alegar: [COLE-OS]

Retorne:
1. Precisão e recall no nível de tripla — como amostrar e estimá-los com intervalo de confiança
   declarado, não vibe
2. Onde meu conjunto de teste vaza para o meu conjunto de treino ou de desenvolvimento de prompt
3. Se estou reportando link prediction: se o cenário filtrado foi usado, e o que uma linha de base
   trivial pontuaria
4. As três alegações que um revisor ataca primeiro, e o experimento que defende cada uma

Assuma que meus números estão inflados até o método de amostragem provar o contrário.
```

## 9 · `/kg-rag`

```
Aja como engenheiro de recuperação. Ligue meu grafo a um agente e prove que ele vence busca vetorial.

Grafo: [DESCREVA]
Tipos de pergunta: [3 EXEMPLOS]

Retorne:
1. A estratégia de recuperação por tipo de pergunta — lookup de entidade, travessia k-hop, extração
   de subgrafo, ou vetor puro. Diga quais perguntas não precisam do grafo
2. Como um subgrafo recuperado é serializado em contexto sem estourar a janela
3. Uma linha de base só-vetor sobre o mesmo texto fonte
4. Um conjunto de eval de 30 perguntas escrito ANTES de qualquer sistema rodar, com gabarito e a
   métrica que os separa

Se o grafo não vencer em perguntas multi-hop, ele não está ganhando seu custo de manutenção.
```
