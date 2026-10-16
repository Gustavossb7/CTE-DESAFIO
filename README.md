# Exercícios de SQL

Página única em HTML com **128 exercícios de SQL** em oito trilhas. Você escreve a consulta, clica em **Verificar resposta** e a página diz se está correta. Não precisa de servidor, banco de dados nem instalação: é só abrir o arquivo no navegador.

## Como usar

1. Baixe ou clone o repositório.
2. Abra `exercicios-sql-completo.html` no navegador.
3. Use os botões no topo para escolher a trilha (**Todos**, **Fácil**, **Médio**, **Difícil**, **Joins**, **CASE WHEN**, **Subconsulta**, **CTE**, **Window function**).
4. Escreva a consulta na caixa de cada exercício e clique em **Verificar resposta**.
5. Se travar, o botão **Ver resposta** mostra a solução.

A barra de progresso soma os acertos, e cada botão mostra o andamento da própria trilha. O progresso **não é salvo**: ao recarregar a página, a contagem volta a zero.

## Trilhas

| Trilha | Exercícios | O que cobre |
|---|---:|---|
| Fácil | 20 | `SELECT`, `WHERE`, `ORDER BY`, agregações, `GROUP BY` e um `JOIN` simples |
| Médio | 20 | JOINs, subconsultas, `CASE`, `UNION`, funções de janela, `INSERT`, `DELETE` |
| Difícil | 20 | Risco de crédito: CTEs, janelas, subconsulta correlacionada, self join, inadimplência |
| Joins | 20 | `INNER`, `LEFT`, `RIGHT`, `FULL`, `CROSS`, anti-join, self join, tabela derivada |
| CASE WHEN | 12 | Classificação, pivot com `SUM`/`COUNT`, `CASE` no `ORDER BY` e no `UPDATE` |
| Subconsulta | 12 | Escalar, `IN`, `NOT IN`, `EXISTS`, `ALL`, correlacionada, no `SELECT` e no `FROM` |
| CTE | 12 | CTE simples, encadeadas, com `JOIN`, com janela e recursiva |
| Window function | 12 | `ROW_NUMBER`, `DENSE_RANK`, `LAG`/`LEAD`, `FIRST_VALUE`, `NTILE`, `OVER` |

## Como a correção funciona

A página **não executa** o SQL. Ela compara o texto da sua consulta com as respostas aceitas, depois de normalizar os dois. A comparação ignora:

- maiúsculas e minúsculas;
- espaços e quebras de linha extras;
- o ponto e vírgula final;
- `INNER JOIN` ou `JOIN`, `LEFT OUTER JOIN` ou `LEFT JOIN`, e `ASC` no `ORDER BY`;
- a ordem dos dois lados de uma igualdade (`c.id = p.cliente_id` ou `p.cliente_id = c.id`).

Por isso, uma consulta diferente da esperada, mas também correta, pode ser marcada como errada. Os enunciados indicam apelidos de tabela, nomes de coluna e a ordem esperada para reduzir esse problema. Para conferir o resultado de verdade, rode a consulta no seu banco.

## Esquemas das tabelas

Cada trilha usa um destes cenários. Os scripts abaixo são para **PostgreSQL**.

### Fácil e Médio

```sql
CREATE TABLE clientes (
    id     SERIAL PRIMARY KEY,
    nome   VARCHAR(100) NOT NULL,
    cidade VARCHAR(100),
    idade  INTEGER
);

CREATE TABLE produtos (
    id        SERIAL PRIMARY KEY,
    nome      VARCHAR(100) NOT NULL,
    categoria VARCHAR(50),
    preco     NUMERIC(10, 2) NOT NULL
);

CREATE TABLE pedidos (
    id         SERIAL PRIMARY KEY,
    cliente_id INTEGER NOT NULL REFERENCES clientes (id),
    produto_id INTEGER NOT NULL REFERENCES produtos (id),
    quantidade INTEGER NOT NULL,
    data       DATE NOT NULL
);
```

### Joins

```sql
CREATE TABLE agencias (
    id     SERIAL PRIMARY KEY,
    nome   VARCHAR(100) NOT NULL,
    cidade VARCHAR(100)
);

CREATE TABLE clientes (
    id           SERIAL PRIMARY KEY,
    nome         VARCHAR(100) NOT NULL,
    agencia_id   INTEGER,
    indicado_por INTEGER,
    CONSTRAINT fk_agencia       FOREIGN KEY (agencia_id)   REFERENCES agencias (id),
    CONSTRAINT fk_indicado_por  FOREIGN KEY (indicado_por) REFERENCES clientes (id)
);

CREATE TABLE produtos (
    id   SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL
);

CREATE TABLE contratos (
    id         SERIAL PRIMARY KEY,
    cliente_id INTEGER NOT NULL REFERENCES clientes (id),
    produto_id INTEGER NOT NULL REFERENCES produtos (id),
    valor      NUMERIC(12, 2) NOT NULL
);
```

### Difícil, CASE WHEN, Subconsulta, CTE e Window function (risco de crédito)

```sql
CREATE TABLE clientes (
    id       SERIAL PRIMARY KEY,
    nome     VARCHAR(100) NOT NULL,
    renda    NUMERIC(12, 2),
    segmento VARCHAR(50)
);

CREATE TABLE contratos (
    id               SERIAL PRIMARY KEY,
    cliente_id       INTEGER NOT NULL REFERENCES clientes (id),
    produto          VARCHAR(50),
    valor            NUMERIC(12, 2) NOT NULL,
    data_contratacao DATE NOT NULL
);

CREATE TABLE parcelas (
    id             SERIAL PRIMARY KEY,
    contrato_id    INTEGER NOT NULL REFERENCES contratos (id),
    numero         INTEGER NOT NULL,
    vencimento     DATE NOT NULL,
    valor          NUMERIC(12, 2) NOT NULL,
    data_pagamento DATE,
    dias_atraso    INTEGER NOT NULL DEFAULT 0
);
```

Os três cenários reutilizam o nome `clientes` com colunas diferentes. Para praticar mais de um, crie cada cenário em um schema separado (por exemplo `CREATE SCHEMA joins;`) ou apague as tabelas com `DROP TABLE ... CASCADE` antes de trocar.

## Compatibilidade entre bancos

Os exercícios foram pensados para PostgreSQL. Alguns recursos variam em outros bancos:

- `RIGHT JOIN` e `FULL JOIN` (trilha Joins): o MySQL não tem `FULL JOIN`, e o SQLite só aceita os dois nas versões mais recentes.
- `WITH RECURSIVE` (trilha CTE): no SQL Server, a mesma ideia é escrita sem a palavra `RECURSIVE`.
- `EXTRACT(YEAR FROM ...)` (trilha Difícil): não funciona no SQL Server.

## Estrutura sugerida do repositório

```
.
├── README.md
└── exercicios-sql-completo.html
```

## Como adicionar ou editar exercícios

Todos os exercícios ficam no próprio HTML, dentro da lista `SECTIONS` no bloco `<script>`. Cada trilha tem `id`, `nome`, `desc`, `schema` e `data`. Em `data`, cada exercício segue este formato:

```js
["Enunciado do exercício.",
 "SELECT ...;",                       // resposta mostrada em "Ver resposta" e aceita na correção
 ["select ... outra forma aceita"]]   // opcional: variações também aceitas
```

Para criar uma trilha nova, acrescente um objeto à lista `SECTIONS` com `fmt:"new"` (o formato acima). O botão de filtro, a contagem e a barra de progresso são gerados automaticamente.

## Ideias para evoluir

- Datas (`CURRENT_DATE`, `DATE_TRUNC`, intervalos) e funções de texto.
- Criação e alteração de tabelas (`CREATE`, `ALTER`, restrições) e transações.
- Qualidade de dados: duplicados, `NULLIF` e a armadilha do `NOT IN` com `NULL`.
- Trilha de indicadores de crédito: roll rate, safras e perda esperada (PD × LGD × EAD).
- Executar o SQL de verdade no navegador (por exemplo com SQLite em WebAssembly), em vez de comparar texto.
- Completar as trilhas temáticas, hoje com 12 exercícios, até 20.
