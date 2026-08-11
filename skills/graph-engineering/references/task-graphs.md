# Task Graphs: Orquestrando Agentes
*(A metade de execução do graph engineering — como agentes trabalham, em vez do que eles lembram)*

## Conteúdo
- [O que é um task graph](#o-que-é-um-task-graph)
- [Fake edges](#fake-edges)
- [O padrão diamond](#o-padrão-diamond)
- [A stop rule](#a-stop-rule)
- [O gate humano](#o-gate-humano)
- [Guardrails](#guardrails)

## O que é um task graph

Nós são trabalhos — cada um algo que você entregaria a um único assistente (pesquisar um
concorrente, escrever um rascunho, checar uma afirmação). Desenhe uma seta SÓ quando um trabalho
precisa do *resultado* de outro antes de começar. O desenho é o plano; os agentes fluem por ele.
Um pequeno objeto de estado (o que foi achado, o que foi decidido, o que falta) viaja com o trabalho.

Isto é um DAG — o padrão que roda a infraestrutura de dados há décadas (Airflow, Prefect,
Temporal) agora aplicado a agentes (LangGraph, CrewAI, AutoGen). A idade do padrão é uma
característica: confie seu negócio a maquinário com décadas de história em produção.

## Fake edges

A primeira otimização custa nada: para cada "e depois" num pipeline existente, pergunte se o
próximo trabalho realmente LÊ a saída do anterior. "Resuma este arquivo e depois cheque minha
agenda" — o passo da agenda nunca usa o resumo; a aresta é falsa. Delete fake edges e esses
trabalhos rodam em paralelo. A maioria dos pipelines feitos à mão contém duas ou três.

## O padrão diamond

A forma para onde sistemas sérios convergem:

```
        ┌─ worker 1 ─┐
plan ───┼─ worker 2 ─┼─→ verify ─→ merge ─→ result
        └─ worker 3 ─┘
```

Divida a tarefa em ângulos independentes, rode os workers em paralelo, **verifique em um contexto
separado**, mescle os sobreviventes. O nó de verificação é inegociável: um modelo avaliando o
próprio trabalho no próprio contexto perde a maioria dos próprios erros. Dê a cada verificador uma
pergunta diferente (está correto? está atual? a fonte é real?) — céticos diversos pegam o que
idênticos não pegam.

## A stop rule

Do estudo Google DeepMind × MIT "Towards a Science of Scaling Agent Systems"
(180 configurações controladas): times coordenados vencem um agente único em ~80% dos trabalhos
que se dividem em peças independentes — e **toda** configuração multi-agente perdeu em trabalho
sequencial onde cada passo precisa da visão completa (degradando 39-70%). Agentes não coordenados
amplificaram os erros uns dos outros em 17,2×; um único coordenador dono do merge cortou para 4,4×.

O procedimento de decisão:
1. Pergunte: *onde meu trabalho se divide em peças que nunca leem os resultados umas das outras?*
2. Divida só isso. Tudo que é sequencial fica com um agente.
3. Nunca deixe achados se fundirem sem um dono do merge.

Mais agentes não é estratégia. O formato do trabalho decide.

## O gate humano

O humano é um nó. Roteie toda aresta irreversível — enviar, publicar, reembolsar, deletar, dar
deploy — por aprovação explícita. Regra de colocação: **ponha o gate onde um erro é caro de
desfazer, não em todo passo.** Gate em tudo faz do humano o gargalo; gate em nada significa que
ninguém está vigiando. Julgue o sistema por números que não podem argumentar de volta (testes que
rodaram, dinheiro que caiu), nunca pelos auto-relatos dele.

## Guardrails

Quatro caps que impedem um grafo de virar um acidente caro:
1. Todo loop tem um número máximo de rodadas.
2. Um escritor por arquivo — dois trabalhos nunca mutam o mesmo artefato.
3. O roteamento vive em passos escritos; o modelo preenche os trabalhos, não o plano.
4. Um cap duro de quantos agentes podem nascer.
