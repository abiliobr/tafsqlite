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
* Tabela: **`entrada`**
  - `id`
  - `conta_id`
  - `transacao_id`
  - `valor`
  - `dc`

A tabela **`entrada`** contém os registros que ligam uma transação a uma ou mais contas na tabela **`conta`**, à semelhança do método contábil chamado "partidas dobradas". Portanto, a tabela **`entrada`** representa o relacionamento "muitos-para-muitos" conforme o esquema abaixo:

**`transacao ----< entrada >---- conta`**

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
    codigo    INTEGER (5)   NOT NULL
                            UNIQUE,
    natureza  VARCHAR (1)   CHECK (natureza IN ("C", "D") )
                            NOT NULL,
    sintetica BOOLEAN       DEFAULT (0)
                            NOT NULL
);

CREATE TABLE entrada (
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
| 4  | 2020/01/10 |           98.99         | Pagto de conta de telefone         | dezembro/2019           |
| 5  | 2020/01/10 |          135.40         | Compra de gas de cozinha           |                         |
| 6  | 2020/01/12 |          201.40         | Compras na farmacia YYY            |                         |
| 7  | 2020/01/28 |          150.00         | Renda de servico de montagem       |                         |
| 8  | 2020/01/30 |           29.89         | Pagto de imposto territorial       | 2020                    |
| 9  | 2020/01/31 |           17.32         | Rendimento de aplicacao financeira | 31/01/2020              |
| 10 | 2020/02/05 |         1950.11         | Proventos                          | janeiro/2020            |
| 11 | 2020/02/06 |         1267.12         | Compras no supermercado XXX        |                         |
| 12 | 2020/02/09 |           84.71         | Pagto de conta de energia eletrica | 06/01/2020 a 05/02/2020 |
| 13 | 2020/02/10 |           98.99         | Pagto de conta de telefone         | janeiro/2020            |
| 14 | 2020/02/14 |          140.00         | Venda de maquina usada             |                         |
| 15 | 2020/02/16 |           41.27         | Despesas na farmacia YYY           |                         |
| 16 | 2020/02/28 |           16.27         | Rendimento de aplicacao financeira | 28/02/2020              |
| 17 | 2020/03/05 |         1940.42         | Proventos                          | fevereiro/2020          |
| 18 | 2020/03/06 |          807.20         | Compras no supermercado ZZZ        |                         |
| 19 | 2020/03/09 |           80.44         | Pagto de conta de energia eletrica | 06/02/2020 a 05/03/2020 |
| 20 | 2020/03/10 |           98.99         | Pagto de conta de telefone         | fevereiro/2020          |
| 21 | 2020/03/11 |          132.20         | Compra de gas de cozinha           |                         |
| 22 | 2020/03/16 |          158.00         | Vendas de doces e salgados         |                         |
| 23 | 2020/03/26 |          407.70         | Compras no supermercado XXX        |                         |
| 24 | 2020/03/28 |           16.22         | Rendimento de aplicacao financeira | 31/03/2020              |
| 25 | 2020/04/05 |         1950.68         | Proventos                          | marco/2020              |
| 26 | 2020/04/06 |          902.40         | Compras no supermercado SSS        |                         |
| 27 | 2020/04/09 |           80.44         | Pagto de conta de energia eletrica | 06/03/2020 a 05/04/2020 |
| 28 | 2020/04/10 |           98.90         | Pagto de conta de telefone         | marco/2020              |
| 29 | 2020/04/26 |           50.40         | Despesa de transporte              |                         |
| 30 | 2020/04/26 |          107.00         | Pagto de servico de limpeza        |                         |
| 31 | 2020/04/30 |           17.86         | Rendimento de aplicacao financeira | 30/04/2020              |
+----+------------+-------------------------+------------------------------------+-------------------------+
</pre>
### As contas

Os registros históricos de transações mostram a necessidade de classificar os eventos financeiros em categorias fixas, chamadas de "contas". Nesse exemplo, algumas contas recorrentes são: salários, compras em lojas, vendas de itens, ganhos com serviços prestados, contas de energia, água e telefone, pagamento de impostos, etc.

O patrimônio pessoal é representado por diversos elementos: dinheiro, contas bancárias, imóveis, veículos, direitos autoriais, obrigações financeiras, etc. Nesta simulação, para as contas do Ativo, vamos considerar apenas a existência de dinheiro em espécie ou disponível em uma conta bancária ou aplicação financeira.

Quanto ao passivo, que são as obrigações a cumprir (prestações, empréstimos contraídos, etc) vamos usar apenas contas a pagar e o patrimônio líquido (ou equidade) para simplificar a demonstração.

As contas devem possuir um código para facilitar o uso e a classificação, como mostrado a seguir. Dependendo da necessidade, este esquema de contas poderá ser expandido para ajustar-se a novas ocorrências.
<pre>
1      - Ativo
111100 - Dinheiro em Carteira
111210 - Conta Bancária
111220 - Aplicação Financeira
2      - Passivo
211    - Contas a Pagar
22     - Patrimonio Liquido
3      - Despesas
311110 - Energia Elétrica
311120 - Água e Esgoto
311130 - Gás de Cozinha
311140 - Telefone
311150 - Transportes
311510 - Supermercados
311520 - Farmácias
311530 - Lojas Diversas
311800 - Serviços Diversos
311900 - Impostos
4      - Receitas
411100 - Salários e Proventos
411200 - Serviços Diversos
411500 - Vendas de Itens Diversos
411900 - Rendimentos de Aplicações Financeiras
</pre>

Os registros na tabela `contas` inicialmente serão estes:
```sqlite
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Ativo',1,'D',1);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Dinheiro em Carteira',11110,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Conta Bancaria',11120,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Aplicacao Financeira',11130,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Passivo',2,'C',1);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Contas a Pagar',211,'C',1);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Patrimonio Liquido',22,'C',1);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Despesas',3,'D',1);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Energia Eletrica',31111,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Agua e Esgoto',31112,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Gas de Cozinha',31113,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Telefone',31114,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Transportes',31150,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Supermercados',31161,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Farmacias',31162,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Lojas Diversas',31163,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Servicos Diversos',31180,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Impostos',31190,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Receitas',4,'C',1);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Salarios e Proventos',41110,'C',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Servicos Diversos',41120,'C',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Vendas de Itens Diversos',41150,'C',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Rendimentos de Aplicacao Financeira',41190,'C',0);

```
<pre>
<b>sqlite></b> .mode table
<b>sqlite></b> SELECT * FROM conta;
+----+-------------------------------------+--------+----------+-----------+
| id |              descricao              | codigo | natureza | sintetica |
+----+-------------------------------------+--------+----------+-----------+
| 1  | Ativo                               | 1      | D        | 1         |
| 2  | Dinheiro em Carteira                | 11110  | D        | 0         |
| 3  | Conta Bancaria                      | 11120  | D        | 0         |
| 4  | Aplicacao Financeira                | 11130  | D        | 0         |
| 5  | Passivo                             | 2      | C        | 1         |
| 6  | Contas a Pagar                      | 211    | C        | 1         |
| 7  | Patrimonio Liquido                  | 22     | C        | 1         |
| 8  | Despesas                            | 3      | D        | 1         |
| 9  | Energia Eletrica                    | 31111  | D        | 0         |
| 10 | Agua e Esgoto                       | 31112  | D        | 0         |
| 11 | Gas de Cozinha                      | 31113  | D        | 0         |
| 12 | Telefone                            | 31114  | D        | 0         |
| 13 | Transportes                         | 31150  | D        | 0         |
| 14 | Supermercados                       | 31161  | D        | 0         |
| 15 | Farmacias                           | 31162  | D        | 0         |
| 16 | Lojas Diversas                      | 31163  | D        | 0         |
| 17 | Servicos Diversos                   | 31180  | D        | 0         |
| 18 | Impostos                            | 31190  | D        | 0         |
| 19 | Receitas                            | 4      | C        | 1         |
| 20 | Salarios e Proventos                | 41110  | C        | 0         |
| 21 | Servicos Diversos                   | 41120  | C        | 0         |
| 22 | Vendas de Itens Diversos            | 41150  | C        | 0         |
| 23 | Rendimentos de Aplicacao Financeira | 41190  | C        | 0         |
+----+-------------------------------------+--------+----------+-----------+
</pre>

# Referências