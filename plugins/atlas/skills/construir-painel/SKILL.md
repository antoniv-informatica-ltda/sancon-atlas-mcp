---
name: construir-painel
description: Monta um Rascunho de Painel do Atlas (Sancon) sobre o ERP ou o HCM Senior da empresa conectada, usando as ferramentas do servidor MCP do Atlas. Use quando pedirem um painel, indicador, dashboard ou relatório com dados do Senior.
---

# Construir um Painel no Atlas

O Atlas guarda o dicionário de dados do ERP e do HCM Senior da empresa conectada e monta Painéis
(indicadores, gráficos e tabelas) sobre o dado vivo. Você monta o **Rascunho**; quem publica é uma
pessoa, na tela do Atlas. Nenhuma ferramenta publica, e não tente contornar isso.

## Fluxo

1. **Entenda o pedido.** Pergunte o que falta: qual indicador, qual período, quais filtros (empresa,
   filial, data).
2. **Ache as tabelas.** `search_tables` com palavras do negócio ("pedido de venda", "cliente"). Os
   códigos seguem o padrão Senior: `E120PED` (pedidos), `E085CLI` (clientes), `R034FUN` (colaboradores).
3. **Entenda os campos.** `describe_table` traz tipo, chave primária, relacionamentos e o significado
   de cada valor de domínio. Tabela grande vem em partes: siga `nextOffset`.
4. **Junte.** `join_path` dá a condição de junção pronta entre duas tabelas.
5. **Abra o Rascunho.** `list_environments` e `create_draft` (ou `read_draft`, se ele já existe).
   Amostra e SELECT de teste só funcionam com Rascunho aberto.
6. **Olhe o dado.** `sample_table` com poucos campos, antes de escrever SQL.
7. **Teste cada consulta.** `run_select` com o SELECT exato que vai para o Painel. Corrija pela
   mensagem de erro, que vem do próprio ERP.
8. **Monte.** `update_draft` com a definição inteira: filtros, Consultas e widgets. Passe a versão
   lida; se o Rascunho mudou, leia de novo.
9. **Entregue.** Diga à pessoa que o Rascunho está pronto para revisar e publicar na tela do Atlas.

## Regras do SQL

- Um SELECT do SQL Server por Consulta. Sem `;`, comentário, colchete, aspas duplas nem CTE.
- `ORDER BY` só com `TOP`.
- Filtro entra como `:chave` e precisa estar em `params` da Consulta e em `filters` da definição.
- Escolha só as colunas que o widget usa; nomeie cada uma em `columns`.

## Widgets

- `kpi` usa `value`; `bar` e `line` usam `x` e `y`; `pie` usa `label` e `value`; `table` mostra a
  Consulta inteira. Os campos são nomes de colunas da Consulta.

## O que não fazer

- Não peça nem invente código de regra, webservice ou porta do Senior: o Atlas gera isso sozinho a
  partir da definição.
- Não amostre tabela de credencial (as `E070*`): o Atlas recusa.
- Não repita a mesma consulta pesada em laço; ela roda no ERP do cliente.
