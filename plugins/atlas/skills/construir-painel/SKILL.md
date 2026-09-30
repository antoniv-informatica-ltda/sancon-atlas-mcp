---
name: construir-painel
description: Monta um Rascunho de Painel do Atlas (Sancon) sobre o ERP ou o HCM Senior da empresa conectada, usando as ferramentas do servidor MCP do Atlas. Use quando pedirem um painel, indicador, dashboard ou relatório com dados do Senior.
---

# Construir um Painel no Atlas

O Atlas guarda o dicionário de dados do ERP e do HCM Senior da empresa conectada e monta Painéis
(indicadores, gráficos e tabelas) sobre o dado vivo. Você monta o **Rascunho** do Painel e o das **APIs**
que ele usa; quem publica é uma pessoa, na tela do Atlas, e a publicação leva junto as APIs que o
Rascunho abriu. Nenhuma ferramenta publica, e não tente contornar isso.

## Como as peças se encaixam

- Uma **API** é um conjunto de **operações**: cada uma é um SELECT fixo, com **entradas** (parâmetros) e
  **colunas** de saída. O SQL mora na operação.
- Uma **Consulta** do Painel só **aponta** para uma operação de uma API (`api`, `operation`) e diz quais
  filtros do Painel preenchem as entradas (`params`) e quais colunas ela mostra (`columns`, com o rótulo).
- Um **widget** mostra uma Consulta.

## Fluxo

1. **Entenda o pedido.** Pergunte o que falta: qual indicador, qual período, quais filtros (empresa,
   filial, data). Pedido curto ("um KPI e uma tabela") não é teto: proponha a composição abaixo.
2. **Ache as tabelas.** `search_tables` com palavras do negócio ("pedido de venda", "cliente"). Os
   códigos seguem o padrão Senior: `E120PED` (pedidos), `E085CLI` (clientes), `R034FUN` (colaboradores).
3. **Entenda os campos.** `describe_table` traz tipo, chave primária, relacionamentos e o significado
   de cada valor de domínio. Tabela grande vem em partes: siga `nextOffset`.
4. **Junte.** `join_path` dá a condição de junção pronta entre duas tabelas.
5. **Abra o Rascunho do Painel.** `list_environments` e `create_draft` (ou `read_draft`, se ele já existe).
   Amostra, SELECT de teste e API só funcionam com Rascunho aberto.
6. **Olhe o dado.** `sample_table` com poucos campos, antes de escrever SQL.
7. **Reaproveite antes de criar.** `list_apis` mostra as APIs do Ambiente com a Versão no ar e as
   operações dela (entradas e colunas). Se uma operação já devolve o que o widget precisa, use-a: a
   Consulta aponta para ela e nada mais é publicado. `draft: "other"` é API que outro Painel (ou a tela
   de APIs) está mudando: não mexa nela, use a Versão no ar.
8. **Crie a API, se preciso.** `create_api_draft` (com o `panelId`, um `slug` novo e o nome) abre o
   Rascunho dela. Teste cada SELECT com `run_select` e grave as operações com `update_api_draft`
   (passe a `version` lida; se mudou, leia de novo com `read_api_draft`). Uma operação tem `key`,
   `sql`, `inputs` (`key` e `type`: `string`, `number` ou `date`), `columns` (`name` e `type`),
   `pageSize` e, num ranking, `orderBy`. Entrada `string` ou `number` que pode ficar vazia ("todas as
   empresas") leva `optional: true`. O retorno traz `problems`, que impedem publicar: corrija cada um
   e grave de novo. `hints` não impedem nada, mas leia.
9. **Monte o Painel.** `update_draft` com a definição inteira: `filters`, `queries` e `widgets`. Cada
   Consulta é `{ key, kind: "webservice", api, operation, params, columns }`. Deixe `version` de fora: a
   publicação usa a Versão que o seu Rascunho de API cria, ou a última no ar. O retorno também traz
   `problems` e `hints`.
10. **Entregue.** Diga à pessoa que o Rascunho está pronto para revisar e publicar na tela do Atlas, e
    quais APIs vão junto.

## Regras do SQL (da operação)

- Um SELECT do SQL Server por operação. Sem `;`, comentário, colchete, aspas duplas nem CTE.
- `ORDER BY` só com `TOP`, e ele só escolhe as linhas: a página sai na ordem de `orderBy`
  (`[{ "column": "total", "desc": true }]`), e sem ele na ordem das colunas declaradas. Ranking sem
  `orderBy` aparece ordenado pela primeira coluna.
- Entrada entra como `:chave` e precisa estar em `inputs`. Toda entrada precisa de um filtro do Painel
  **com o mesmo nome**, listado em `params` da Consulta: a entrada não tem valor padrão.
- Entrada sem valor chega à operação como `0` (number) ou `''` (string), e `WHERE CodEmp = :empresa`
  devolve zero linhas. Se "todas" faz sentido (empresa, filial, cliente), marque a entrada com
  `optional: true` e use-a **só** como `coluna = :chave` ou `coluna LIKE :chave`: o Atlas troca por
  `(vazio OR coluna = :chave)`. Piso ou faixa (`>= :minimo`, `BETWEEN`) não é opcional; `date` também
  não (tem `default` relativo).
- Escolha só as colunas que o widget usa; nomeie cada uma em `columns` da operação, com o mesmo tipo e
  a mesma grafia na Consulta.

## Widgets

- `kpi` usa `value`, e `previous` (opcional) para mostrar a variação contra ela; `bar` e `line` usam
  `x` e `y`; `pie` usa `label` e `value`; `table` mostra a Consulta inteira, e quem lê ordena pelo
  cabeçalho. Os campos são nomes de colunas da Consulta.
- `bar` e `pie` aceitam `filterOnClick: { filter, column }`: clicar na barra preenche o filtro com o
  valor da coluna naquela linha (o código do cliente, não o nome), e o resto do Painel relê.
- Coluna `number` da Consulta leva `format`: `money` (R$), `integer`, `percent` (o valor já em pontos
  percentuais) ou `code` (código, sem separador de milhar).
- Filtro com `options` vira lista. Empresa e filial vêm do catálogo do Atlas, sem consulta ao ERP:
  filtro `number` com `options: { catalog: "company" }` e, para a filial,
  `options: { catalog: "branch", parent: "empresa" }` (a filial segue a empresa escolhida). Qualquer
  outra lista é `options: { query, value, label }`: `query` é uma Consulta sem `params`, `value` a
  coluna com o tipo do filtro e `label` a que aparece.
- Todo widget aceita `cols` (3 a 12, na grade de 12) e `color` (`navy`, `purple`, `teal`, `orange`,
  `green`, `rose`, `slate`, `gold`). Os widgets saem na ordem da lista. Sem `cols`: KPI ocupa 3,
  gráfico 6 (12 se for o único), tabela 12. `color` pinta o filete do KPI e a série de `bar` e `line`;
  `pie` usa a paleta inteira.

## Um Painel que se lê de relance

- 2 a 4 KPIs no topo. Para comparar com o período anterior, a operação devolve as duas colunas (atual e
  anterior) e o KPI usa `previous`.
- Um gráfico por pergunta: ranking vira `bar` (os 10 maiores, com `orderBy` desc), evolução no tempo
  vira `line` (uma linha por mês), participação de até 6 partes vira `pie`.
- A tabela por último, para o detalhe.
- Filtro de período sempre: dois filtros `date` (`dataInicial`, `dataFinal`), usados como entradas das
  operações, com `default` relativo — `startOfYear` e `today` (ou `startOfMonth`). Data fixa no
  `default` faz o Painel publicado abrir para sempre no mesmo período.
- Empresa e filial do catálogo (`options.catalog`), e o Painel tem de **abrir com dado**: ou o filtro
  tem `default` (a empresa principal), ou a entrada correspondente é `optional` e o vazio é "todas".
  Sem um dos dois, o Painel abre vazio até alguém escolher.
- `format` em toda coluna `number`.
- Layout: 3 ou 4 KPIs de `cols: 3` na primeira linha; ranking longo em `bar` com `cols: 12`; dois
  gráficos curtos lado a lado com `cols: 6`; a tabela em `cols: 12` no fim. Uma `color` diferente por
  KPI quando eles medem coisas diferentes.

## O que não fazer

- Não peça nem invente código de regra, webservice ou porta do Senior: o Atlas gera isso sozinho a
  partir das operações.
- Não coloque SQL na Consulta do Painel: ela só aponta para uma operação.
- Não amostre tabela de credencial (as `E070*`): o Atlas recusa.
- Não repita a mesma consulta pesada em laço; ela roda no ERP do cliente.
