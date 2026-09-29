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
   filial, data).
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
   `sql`, `inputs` (`key` e `type`: `string`, `number` ou `date`), `columns` (`name` e `type`) e
   `pageSize`. O retorno traz `problems`: corrija cada um e grave de novo.
9. **Monte o Painel.** `update_draft` com a definição inteira: `filters`, `queries` e `widgets`. Cada
   Consulta é `{ key, kind: "webservice", api, operation, params, columns }`. Deixe `version` de fora: a
   publicação usa a Versão que o seu Rascunho de API cria, ou a última no ar. O retorno também traz
   `problems`.
10. **Entregue.** Diga à pessoa que o Rascunho está pronto para revisar e publicar na tela do Atlas, e
    quais APIs vão junto.

## Regras do SQL (da operação)

- Um SELECT do SQL Server por operação. Sem `;`, comentário, colchete, aspas duplas nem CTE.
- `ORDER BY` só com `TOP`.
- Entrada entra como `:chave` e precisa estar em `inputs`. Toda entrada precisa de um filtro do Painel
  **com o mesmo nome**, listado em `params` da Consulta: a entrada não tem valor padrão.
- Escolha só as colunas que o widget usa; nomeie cada uma em `columns` da operação, com o mesmo tipo e
  a mesma grafia na Consulta.

## Widgets

- `kpi` usa `value`; `bar` e `line` usam `x` e `y`; `pie` usa `label` e `value`; `table` mostra a
  Consulta inteira. Os campos são nomes de colunas da Consulta.

## O que não fazer

- Não peça nem invente código de regra, webservice ou porta do Senior: o Atlas gera isso sozinho a
  partir das operações.
- Não coloque SQL na Consulta do Painel: ela só aponta para uma operação.
- Não amostre tabela de credencial (as `E070*`): o Atlas recusa.
- Não repita a mesma consulta pesada em laço; ela roda no ERP do cliente.
