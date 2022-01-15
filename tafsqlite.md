# Teste de uso de um banco de dados SQLite para registros financeiros

Neste projeto, vamos testar o uso de um banco de dados em SQLite para a gerência de informações financeiras no âmbito pessoal ou doméstico, tais como receitas de salários ou serviços prestados, despesas com alimentação ou transporte, etc. Os passos necessários para o teste são:

1. Projeto de banco de dados de registro de finanças
2. Inserção de dados fictícios
3. Análise de dados
4. Avaliação da estrutura do banco de dados

## Parte 1 - O projeto de banco de dados
Inspirado nas ideias apresentadas em [^1] e [^2], o esquema inicial para as tabelas poderá ficar assim:
[^1]: https://stackoverflow.com/questions/2494343/database-schema-design-for-a-double-entry-accounting-system
[^2]: https://medium.com/@RobertKhou/double-entry-accounting-in-a-relational-database-2b7838a5d7f8

* Tabela: **`transacao`**
  - `id`
  - `data`
  - `valor`
  - `historico`
  - `referente`
* Tabela: **`conta`**
  - `id`
  - `descricao`
  - `codigo`
* Tabela: **`partimento`**
  - `id`
  - `conta_id`
  - `transacao_id`
  - `valor`
  - `dc`

A tabela **`partimento`** contém os registros que ligam uma transação a uma ou mais contas na tabela **`conta`**, à semelhança do método contábil chamado "partidas dobradas". Portanto, a tabela **`partimento`** representa o relacionamento "muitos-para-muitos" conforme o esquema abaixo:

**`transacao ----< partimento >---- conta`**

### Criação do banco de dados e tabelas

No SQLite, a criação do banco de dados é feita simplesmente com o comando:
<pre>
<b>$ sqlite3 taf.db3</b>
<b>sqlite></b>
</pre>
Para a criação das tabelas, são executadas as seguintes DDLs:
</pre>
```sqlite
CREATE TABLE transacao (
    id        INTEGER         PRIMARY KEY AUTOINCREMENT
                              NOT NULL,
    data      DATE            NOT NULL,
    valor     DECIMAL (15, 2) NOT NULL,
    historico VARCHAR (100)   NOT NULL,
    referente VARCHAR (50)
);

CREATE TABLE conta (
    id        INTEGER       PRIMARY KEY AUTOINCREMENT
                            NOT NULL,
    descricao VARCHAR (100) NOT NULL,
    codigo    VARCHAR (5)   NOT NULL,
    dc        VARCHAR (1)   NOT NULL
                            CHECK (dc IN ("C", "D", "*") ) 
);

CREATE TABLE partimento (
    id           INTEGER         PRIMARY KEY AUTOINCREMENT
                                 NOT NULL,
    transacao_id INTEGER         REFERENCES transacao (id) ON DELETE CASCADE
                                                           ON UPDATE CASCADE
                                 NOT NULL,
    conta_id     INTEGER         REFERENCES conta (id) ON DELETE CASCADE
                                                       ON UPDATE CASCADE
                                 NOT NULL,
    valor        DECIMAL (15, 2) NOT NULL,
    dc           VARCHAR (1)     NOT NULL
                                 CHECK (dc IN ("C", "D") ) 
);
```

## Parte 2 - Inserção de dados nas tabelas
### Transações de receitas e despesas
Começando pela tabela **`transacao`**, vamos registrar ocorrências de receitas e despesas fictícias:

```sqlite
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/05','   1940.42','Proventos','dezembro/2019');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/06','    967.12','Compras no supermercado XXX',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/09','     80.78','Pagto de conta de energia eletrica','06/12/2019 a 05/01/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/10','     98.99','Pagto de conta de telefone','dezembro/2019');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/10','    135.40','Compra de gas de cozinha',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/12','    201.40','Compras na farmacia YYY',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/28','    150.00','Renda de servico de montagem',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/30','     29.89','Pagto de imposto territorial','2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/31','     17.32','Rendimento de aplicacao financeira','31/01/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/02/05','   1950.11','Proventos','janeiro/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/02/06','   1267.12','Compras no supermercado XXX',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/02/09','     84.71','Pagto de conta de energia eletrica','06/01/2020 a 05/02/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/02/10','     98.99','Pagto de conta de telefone','janeiro/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/02/14','    140.00','Venda de maquina usada',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/02/16','     41.27','Despesas na farmacia YYY',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/02/28','     16.27','Rendimento de aplicacao financeira','28/02/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/05','   1940.42','Proventos','fevereiro/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/06','    807.20','Compras no supermercado ZZZ',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/09','     80.44','Pagto de conta de energia eletrica','06/02/2020 a 05/03/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/10','     98.99','Pagto de conta de telefone','fevereiro/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/11','    132.20','Compra de gas de cozinha',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/16','    158.00','Vendas de doces e salgados',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/26','    407.70','Compras no supermercado XXX',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/28','     16.22','Rendimento de aplicacao financeira','31/03/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/05','   1950.68','Proventos','marco/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/06','    902.40','Compras no supermercado SSS',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/09','     80.44','Pagto de conta de energia eletrica','06/03/2020 a 05/04/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/10','     98.90','Pagto de conta de telefone','marco/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/26','     50.40','Despesa de transporte',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/26','    107.00','Pagto de servico de limpeza',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/30','     17.86','Rendimento de aplicacao financeira','30/04/2020');

```

> Observe-se que os valores monetários são sempre informados com ponto decimal (.). Já as datas devem estar no formato YYYY-MM-DD para maior compatibilidade.

Após a inserção dos registros, a listagem da tabela **`transacao`** poderá ficar como abaixo:
<pre>
<b>sqlite></b> .mode table
<b>sqlite></b> SELECT id, data, printf('%15.2f', valor), historico, referente FROM transacao;
+----+------------+-------------------------+------------------------------------+-------------------------+
| id |    data    | printf('%15.2f', valor) |             historico              |        referente        |
+----+------------+-------------------------+------------------------------------+-------------------------+
| 1  | 2020/01/05 |         1940.42         | Proventos                          | dezembro/2019           |
| 2  | 2020/01/06 |          967.12         | Compras no supermercado XXX        |                         |
| 3  | 2020/01/09 |           80.78         | Pagto de conta de energia eletrica | 06/12/2019 a 05/01/2020 |
| 4  | 2020/01/10 |           44.51         | Pagto de conta de agua             | dezembro/2019           |
| 5  | 2020/01/10 |           98.99         | Pagto de conta de telefone         | dezembro/2019           |
| 6  | 2020/01/10 |          135.40         | Compra de gas de cozinha           |                         |
| 7  | 2020/01/12 |          201.40         | Compras na farmacia YYY            |                         |
| 8  | 2020/01/28 |          150.00         | Renda de servico de montagem       |                         |
| 9  | 2020/01/30 |           29.89         | Pagto de imposto territorial       | 2020                    |
| 10 | 2020/01/31 |           17.32         | Rendimento de aplicacao financeira | 31/01/2020              |
| 11 | 2020/02/05 |         1950.11         | Proventos          | janeiro/2020            |
| 12 | 2020/02/06 |         1267.12         | Compras no supermercado XXX        |                         |
| 13 | 2020/02/09 |           84.71         | Pagto de conta de energia eletrica | 06/01/2020 a 05/02/2020 |
| 14 | 2020/02/10 |           44.51         | Pagto de conta de agua             | janeiro/2020            |
| 15 | 2020/02/10 |           98.99         | Pagto de conta de telefone         | janeiro/2020            |
| 16 | 2020/02/14 |          140.00         | Venda de maquina usada             |                         |
| 17 | 2020/02/16 |           41.27         | Despesas na farmacia YYY           |                         |
| 18 | 2020/02/28 |           16.27         | Rendimento de aplicacao financeira | 28/02/2020              |
| 19 | 2020/03/05 |         1940.42         | Proventos          | fevereiro/2020          |
| 20 | 2020/03/06 |          807.20         | Compras no supermercado ZZZ        |                         |
| 21 | 2020/03/09 |           80.44         | Pagto de conta de energia eletrica | 06/02/2020 a 05/03/2020 |
| 22 | 2020/03/10 |           44.32         | Pagto de conta de agua             | fevereiro/2020          |
| 23 | 2020/03/10 |           98.99         | Pagto de conta de telefone         | fevereiro/2020          |
| 24 | 2020/03/11 |          132.20         | Compra de gas de cozinha           |                         |
| 25 | 2020/03/16 |          158.00         | Vendas de doces e salgados         |                         |
| 26 | 2020/03/26 |          407.70         | Compras no supermercado XXX        |                         |
| 27 | 2020/03/28 |           16.22         | Rendimento de aplicacao financeira | 31/03/2020              |
| 28 | 2020/04/05 |         1950.68         | Proventos          | marco/2020              |
| 29 | 2020/04/06 |          902.40         | Compras no supermercado SSS        |                         |
| 30 | 2020/04/09 |           80.44         | Pagto de conta de energia eletrica | 06/03/2020 a 05/04/2020 |
| 31 | 2020/04/10 |           46.70         | Pagto de conta de agua             | marco/2020              |
| 32 | 2020/04/10 |           98.90         | Pagto de conta de telefone         | marco/2020              |
| 33 | 2020/04/11 |          130.33         | Compra de gas de cozinha           |                         |
| 34 | 2020/04/26 |           50.40         | Despesa de transporte              |                         |
| 35 | 2020/04/30 |           17.86         | Rendimento de aplicacao financeira | 30/04/2020              |
+----+------------+-------------------------+------------------------------------+-------------------------+

</pre>
### As contas

Os registros históricos de transações mostram a necessidade de classificar os eventos financeiros em categorias fixas, chamadas de "contas". Nesse contexto, algumas contas recorrentes são: salários, compras em lojas, vendas de itens, ganhos com serviços prestados, contas de energia, água e telefone, pagamento de impostos, etc.

As contas devem possuir um código para facilitar o uso e a classificação, como mostrado a seguir. Dependendo da necessidade, este esquema de contas poderá ser expandido para ajustar-se a novos eventos.
<pre>
R     - Receitas
R-100 - Salários e Proventos
R-200 - Serviços Diversos
R-500 - Vendas de Itens Diversos
R-900 - Rendimentos de Aplicações Financeiras

D     - Despesas
D-010 - Energia Elétrica
D-020 - Água e Esgoto
D-030 - Gás de Cozinha
D-040 - Telefone
D-050 - Transportes
D-510 - Supermercados
D-520 - Farmácias
D-530 - Lojas Diversas
D-800 - Serviços Diversos
D-900 - Impostos
</pre>

### Os principais ativos

O patrimônio pessoal é representado por diversos elementos: dinheiro, contas bancárias, imóveis, veículos, direitos autoriais, obrigações financeiras, etc.

Nesta simulação, vamos considerar apenas o recebimento de dinheiro em espécie ou através de uma conta bancária. Quanto ao passivo, que são obrigações a cumprir, como contas a pagar (prestações), empréstimos, etc. este não será abordado, para simplificar a demonstração. O "dinheiro" vai ser representado pelos códigos:

<pre>
A     - Ativo
A-100 - Dinheiro em Carteira
A-210 - Conta Bancária
A-220 - Aplicação Financeira
</pre>

### Dados para a tabela `contas`


# Referências