# Representação do Conhecimento & Modelagem de Ontologia
*(Aulas 2-3 do curso, traduzidas e adaptadas)*

## Conteúdo
- [Escolhendo uma representação](#escolhendo-uma-representação)
- [Método de engenharia de ontologias](#método-de-engenharia-de-ontologias)
- [Regras de design de schema](#regras-de-design-de-schema)
- [Exemplo prático](#exemplo-prático)
- [Aprendizado de ontologia](#aprendizado-de-ontologia)

## Escolhendo uma representação

O curso analisa o inventário histórico — redes semânticas, regras de produção, frames (quadros),
grafos conceituais, análise formal de conceitos, lógica de descrição — convergindo em dois sobreviventes
mais um atalho pragmático:

| Representação | Escolha quando | Custo |
|---|---|---|
| **Property graph** (Neo4j, Kùzu, NetworkX) | Padrão para produtos e memória de agentes. Nós/arestas com propriedades arbitrárias de chave-valor. | Sem semântica formal; a consistência é responsabilidade sua. |
| **Triplas RDF/OWL** | Interoperabilidade com ontologias existentes, conformidade com padrões, necessidade de raciocínio de lógica de descrição (subsumção/*subsumption*, checagem de consistência). | Verboso; reificação (*reification* — codificar afirmações sobre afirmações) necessária para propriedades de aresta; ferramentas mais complexas. |
| **Arestas tipadas em JSON/SQLite** | <50K nós, aplicação única, memória local do agente. | Poder de consulta limitado; migre quando consultas multi-hop (múltiplos saltos) ficarem lentas ou frequentes. |

Decida nesta etapa (não mais tarde) como cada fato carrega:
- **Tempo** — intervalo de validade ou timestamp do evento (`since`, `until`).
- **Proveniência** (*provenance*) — documento/URL fonte + timestamp de extração + confiança.

Em property graphs, esses itens são propriedades da aresta; em RDF, use reificação ou RDF-star; em JSON, apenas
adicione os campos. Adicionar proveniência retroativamente após a fusão é praticamente impossível.

## Método de engenharia de ontologias

Condensado do processo de engenharia de ontologias do curso:

1. **Competency questions (perguntas de competência).** Escreva as 10-20 perguntas que o grafo precisa responder
   ("De quais fornecedores o produto X depende transitivamente?"). Elas são a especificação da ontologia
   E sua suíte de testes.
2. **Enumerar tipos centrais de entidades** a partir das perguntas de competência. Comece com 5-15. Cada tipo
   precisa de uma definição de uma linha e 2-3 exemplos reais.
3. **Enumerar tipos de relação** com **`domain` e `range`** (ex.: `EMPLOYED_BY: Person → Org`).
   Comece com 10-30. Adicione notas de cardinalidade onde importarem (uma Pessoa tem **um único** local de nascimento).
4. **Atributos vs entidades.** Se possui seus próprios relacionamentos, é uma entidade ("Cidade" — tem
   país, população). Se é um valor pelo qual você filtra, é um atributo ("ano de fundação").
5. **Hierarquia de tipos apenas quando as consultas exigirem.** `Company ⊂ Organization` vale a pena se
   algumas consultas abrangerem todas as organizações; caso contrário, evite criar subclasses (*subclassing*) — uma estrutura plana é mais fácil de extrair.
6. **Validar em relação às perguntas de competência** — percorra cada pergunta no schema no papel.
   Qualquer pergunta que você não consiga percorrer pelo schema = tipo ou relação ausente.

## Regras de design de schema

- Nomes de verbos precisos para relações: `ACQUIRED`, `CITES`, `DEPENDS_ON` — nunca `RELATED_TO`,
  `HAS_LINK`. Relações vagas tornam todas as consultas subsequentes ambíguas.
- Se dois tipos de entidade são sempre consultados juntos, mescle-os em um só.
- Se um tipo de entidade precisa constantemente de atributos qualificadores para desambiguar o uso
  (`role: "author" | "editor"`), divida-o em dois tipos de relação.
- Nomeie entidades canonicamente no momento da modelagem (defina a regra da forma canônica: nome legal completo?
  minúsculas? idioma?) — a fusão (etapa 8) impõe qualquer regra estabelecida aqui.
- Mantenha um arquivo `ontology.md` (ou `.yaml`) no projeto como a fonte única da verdade; todo
  prompt de extração o embute textualmente.

## Exemplo prático

Pergunta de competência: "Quais engenheiros contribuíram para serviços que tiveram incidentes no último trimestre?"

```yaml
entities:
  Person:   {desc: engineer or manager, ex: [Jane Doe]}
  Service:  {desc: deployable software unit, ex: [payments-api]}
  Incident: {desc: production failure event, ex: [INC-4012]}
relations:
  CONTRIBUTED_TO: {domain: Person,  range: Service}
  AFFECTED:       {domain: Incident, range: Service, attrs: [severity]}
events:
  Incident: {trigger: outage/alert, args: [service, start_time, resolved_time, severity]}
```

Três tipos, duas relações, um tipo de evento — responde totalmente à pergunta. Resista em adicionar mais
até que uma pergunta de competência o exija.

## Aprendizado de ontologia

O curso cobre a indução semi-automática de schema (termos → conceitos → hierarquia → relações).
Atalho da era LLM que preserva a mesma disciplina: forneça a um LLM de 3 a 5 documentos representativos,
peça para propor tipos de entidades/relações **com citações de evidência**, e depois filtre manualmente para o
conjunto mínimo que responde às perguntas de competência. Nunca aceite automaticamente um schema induzido —
ontologias induzidas sofrem de over-fitting aos documentos de amostra.
