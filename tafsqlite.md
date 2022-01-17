# Teste de uso de um banco de dados SQLite para registros financeiros

Neste projeto, vamos testar o uso de um banco de dados em SQLite para a gerência de informações financeiras no âmbito pessoal ou doméstico, tais como receitas de salários ou serviços prestados, despesas com alimentação ou transporte, etc. Os passos necessários para o teste são:

1. Projeto de banco de dados de registro de finanças
2. Inserção de dados fictícios
3. Análise de dados
4. Avaliação da estrutura do banco de dados

## Parte 1 - O projeto de banco de dados
Inspirado nas ideias apresentadas em [^1], [^2] e [^3], o esquema inicial para as tabelas poderá ficar assim:
[^1]: https://stackoverflow.com/questions/2494343/database-schema-design-for-a-double-entry-accounting-system
[^2]: https://medium.com/@RobertKhou/double-entry-accounting-in-a-relational-database-2b7838a5d7f8
[^3]: https://hledger.org/accounting.html

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
    dcs          VARCHAR (1)     NOT NULL
                                 CHECK (dcs IN ("S", "D", "C") ) 
);
```

## Parte 2 - Inserção de dados nas tabelas

### Plano de contas

As ocorrências financeiras precisam ser organizadas em categorias fixas, chamadas de "contas". Nesse estudo, algumas contas recorrentes serão: salários, compras em lojas, vendas de itens, ganhos com serviços prestados, contas de energia, água e telefone, pagamento de impostos, etc.

O patrimônio pessoal é representado por diversos elementos: dinheiro físico, contas bancárias, imóveis, veículos, direitos autoriais, obrigações financeiras, etc. Neste estudo, vamos considerar apenas a existência de dinheiro em espécie ou disponível em uma conta bancária ou aplicação financeira, para o patrimônio atual. Quanto ao passivo, que são as obrigações a cumprir (prestações, empréstimos contraídos, etc) vamos abordar apenas contas a pagar e o patrimônio líquido (ou equidade) para simplificar a demonstração.

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

### Transações

Na tabela **`transacao`**, vamos registrar ocorrências financeira fictícias ao longo de alguns meses, como receitas, despesas, saques e depósitos.

Os primeiros registros serão os saldos iniciais de contas do ativo: dinheiro em espécie, saldo da conta bancária e saldo da aplicação financeira. Inicialmente, não haverá obrigações pendentes (passivos).

```sqlite
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/01',     75.40,'Saldo Anterior em Especie','31/12/2019');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/01',    482.86,'Saldo Anterior em Conta Corrente','31/12/2019');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/01',   4877.14,'Saldo Anterior em Aplicacao Financeira','31/12/2019');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/05',   1940.42,'Proventos','dezembro/2019');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/06',    967.12,'Compras no supermercado XXX',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/09',     80.78,'Pagto de conta de energia eletrica','06/12/2019 a 05/01/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/10',     98.99,'Pagto de conta de telefone','dezembro/2019');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/10',    135.40,'Compra de gas de cozinha',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/12',    201.40,'Compras na farmacia YYY',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/28',    150.00,'Renda de servico de montagem',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/29',    250.00,'Saque em conta corrente',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/30',     29.89,'Pagto de imposto territorial','2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/30',    100.00,'Deposito em aplicacao financeira',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/01/31',     17.32,'Rendimento de aplicacao financeira','31/01/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/02/05',   1950.11,'Proventos','janeiro/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/02/06',   1267.12,'Compras no supermercado XXX',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/02/09',     84.71,'Pagto de conta de energia eletrica','06/01/2020 a 05/02/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/02/10',     98.99,'Pagto de conta de telefone','janeiro/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/02/14',    140.00,'Venda de maquina usada',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/02/16',     41.27,'Despesas na farmacia YYY',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/02/28',     16.27,'Rendimento de aplicacao financeira','28/02/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/05',   1940.42,'Proventos','fevereiro/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/06',    807.20,'Compras no supermercado ZZZ',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/09',     80.44,'Pagto de conta de energia eletrica','06/02/2020 a 05/03/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/10',     98.99,'Pagto de conta de telefone','fevereiro/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/11',    132.20,'Compra de gas de cozinha',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/16',    158.00,'Vendas de doces e salgados',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/26',    407.70,'Compras no supermercado XXX',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/27',    140.00,'Deposito em conta corrente',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/03/28',     16.22,'Rendimento de aplicacao financeira','31/03/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/05',   1950.68,'Proventos','marco/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/06',    150.00,'Saque em conta corrente',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/06',    902.40,'Compras no supermercado SSS',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/09',     80.44,'Pagto de conta de energia eletrica','06/03/2020 a 05/04/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/10',     98.90,'Pagto de conta de telefone','marco/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/26',     50.40,'Despesa de transporte',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/26',    107.00,'Pagto de servico de limpeza',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/30',     17.86,'Rendimento de aplicacao financeira','30/04/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020/04/30',   1000.00,'Saque em aplicacao financeira',NULL);

```

> Observe-se que os valores monetários são sempre informados com ponto decimal (.). Já as datas devem estar no formato YYYY-MM-DD para maior compatibilidade.

Após a inserção dos registros, a listagem da tabela **`transacao`** poderá ficar como abaixo:
<pre>
<b>sqlite></b> .mode table
<b>sqlite></b> SELECT id, data, printf('%15.2f', valor), historico, referente FROM transacao;
+----+------------+-------------------------+----------------------------------------+-------------------------+
| id |    data    | printf('%15.2f', valor) |               historico                |        referente        |
+----+------------+-------------------------+----------------------------------------+-------------------------+
| 1  | 2020/01/01 |           75.40         | Saldo Anterior em Especie              | 31/12/2019              |
| 2  | 2020/01/01 |          482.86         | Saldo Anterior em Conta Corrente       | 31/12/2019              |
| 3  | 2020/01/01 |         4877.14         | Saldo Anterior em Aplicacao Financeira | 31/12/2019              |
| 4  | 2020/01/05 |         1940.42         | Proventos                              | dezembro/2019           |
| 5  | 2020/01/06 |          967.12         | Compras no supermercado XXX            |                         |
| 6  | 2020/01/09 |           80.78         | Pagto de conta de energia eletrica     | 06/12/2019 a 05/01/2020 |
| 7  | 2020/01/10 |           98.99         | Pagto de conta de telefone             | dezembro/2019           |
| 8  | 2020/01/10 |          135.40         | Compra de gas de cozinha               |                         |
| 9  | 2020/01/12 |          201.40         | Compras na farmacia YYY                |                         |
| 10 | 2020/01/28 |          150.00         | Renda de servico de montagem           |                         |
| 11 | 2020/01/29 |          250.00         | Saque em conta corrente                |                         |
| 12 | 2020/01/30 |           29.89         | Pagto de imposto territorial           | 2020                    |
| 13 | 2020/04/30 |          100.00         | Deposito em aplicacao financeira       |                         |
| 14 | 2020/01/31 |           17.32         | Rendimento de aplicacao financeira     | 31/01/2020              |
| 15 | 2020/02/05 |         1950.11         | Proventos                              | janeiro/2020            |
| 16 | 2020/02/06 |         1267.12         | Compras no supermercado XXX            |                         |
| 17 | 2020/02/09 |           84.71         | Pagto de conta de energia eletrica     | 06/01/2020 a 05/02/2020 |
| 18 | 2020/02/10 |           98.99         | Pagto de conta de telefone             | janeiro/2020            |
| 19 | 2020/02/14 |          140.00         | Venda de maquina usada                 |                         |
| 20 | 2020/02/16 |           41.27         | Despesas na farmacia YYY               |                         |
| 21 | 2020/02/28 |           16.27         | Rendimento de aplicacao financeira     | 28/02/2020              |
| 22 | 2020/03/05 |         1940.42         | Proventos                              | fevereiro/2020          |
| 23 | 2020/03/06 |          807.20         | Compras no supermercado ZZZ            |                         |
| 24 | 2020/03/09 |           80.44         | Pagto de conta de energia eletrica     | 06/02/2020 a 05/03/2020 |
| 25 | 2020/03/10 |           98.99         | Pagto de conta de telefone             | fevereiro/2020          |
| 26 | 2020/03/11 |          132.20         | Compra de gas de cozinha               |                         |
| 27 | 2020/03/16 |          158.00         | Vendas de doces e salgados             |                         |
| 28 | 2020/03/26 |          407.70         | Compras no supermercado XXX            |                         |
| 29 | 2020/03/27 |          140.00         | Deposito em conta corrente             |                         |
| 30 | 2020/03/28 |           16.22         | Rendimento de aplicacao financeira     | 31/03/2020              |
| 31 | 2020/04/05 |         1950.68         | Proventos                              | marco/2020              |
| 32 | 2020/04/06 |          150.00         | Saque em conta corrente                |                         |
| 33 | 2020/04/06 |          902.40         | Compras no supermercado SSS            |                         |
| 34 | 2020/04/09 |           80.44         | Pagto de conta de energia eletrica     | 06/03/2020 a 05/04/2020 |
| 35 | 2020/04/10 |           98.90         | Pagto de conta de telefone             | marco/2020              |
| 36 | 2020/04/26 |           50.40         | Despesa de transporte                  |                         |
| 37 | 2020/04/26 |          107.00         | Pagto de servico de limpeza            |                         |
| 38 | 2020/04/30 |           17.86         | Rendimento de aplicacao financeira     | 30/04/2020              |
| 39 | 2020/04/30 |         1000.00         | Saque em aplicacao financeira          |                         |
+----+------------+-------------------------+----------------------------------------+-------------------------+
</pre>

### Saldos iniciais

Na tabela **`entrada`** serão registrados 3 saldos iniciais:
```sqlite
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(1,2,  75.40,'S');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(2,3, 482.86,'S');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(3,4,4877.14,'S');

```
<pre>
<b>sqlite></b> .mode table
<b>sqlite></b> SELECT select id, transacao_id, conta_id, printf('%15.2f', valor), dcs FROM entrada;
+----+--------------+----------+-------------------------+-----+
| id | transacao_id | conta_id | printf('%15.2f', valor) | dcs |
+----+--------------+----------+-------------------------+-----+
| 1  | 1            | 2        |           75.40         | S   |
| 2  | 2            | 3        |          482.86         | S   |
| 3  | 3            | 4        |         4877.14         | S   |
+----+--------------+----------+-------------------------+-----+
</pre>

Depois, a primeira demonstração pode ser obtida assim:
<pre>
<b>sqlite></b> SELECT conta.codigo as Codigo, conta.descricao as Conta, conta.natureza AS 'D/C', printf('%15.2f',SUM(IIF(entrada.dcs="D" OR (entrada.dcs="S" and conta.natureza="D"),-1*entrada.valor,IIF(entrada.dcs="C" OR (entrada.dcs="S" and conta.natureza="C"),entrada.valor,0)))) AS Saldo FROM entrada INNER JOIN conta ON entrada.conta_id = conta.id INNER JOIN transacao ON entrada.transacao_id = transacao.id WHERE entrada.dcs = "S" and transacao.data = '2020/01/01' GROUP BY conta_id HAVING transacao.data <= '2020/01/01' ORDER BY Codigo;
+--------+----------------------+-----+-----------------+
| Codigo |        Conta         | D/C |      Saldo      |
+--------+----------------------+-----+-----------------+
| 11110  | Dinheiro em Carteira | D   |          -75.40 |
| 11120  | Conta Bancaria       | D   |         -482.86 |
| 11130  | Aplicacao Financeira | D   |        -4877.14 |
+--------+----------------------+-----+-----------------+

</pre>

## Lançamentos
Agora, os lançamentos para o restante próximas transações, com algumas observações:

* O salário é depositado na conta bancária; as rendas extras foram recebidas em dinheiro físico;
* As compras em supermercado geralmente são feitas através de débito automático na conta corrente;
* As contas de energia elétrica, de telefone e os impostos são pagos através da conta corrente no banco;
* O pagamento na compra do gás de cozinha e as compras na farmácia foram feitos com dinheiro em espécie.
```sqlite
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 4,20, 1940.42,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 4, 3, 1940.42,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 5, 3,  967.12,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 5,14,  967.12,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 6, 3,   80.78,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 6, 9,   80.78,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 7, 3,   98.99,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 7,12,   98.99,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 8, 2,  135.40,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 8,11,  135.40,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 9, 2,  201.40,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 9,15,  201.40,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(10,21,  150.00,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(10, 2,  150.00,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(11, 3,  250.00,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(11, 2,  250.00,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(12, 3,   29.89,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(12,18,   29.89,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(13, 2,  100.00,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(13, 4,  100.00,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(14, 4,   17.32,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(14, 3,   17.32,'D');

```
<pre>
<b>sqlite></b> SELECT id, transacao_id, conta_id, printf('%15.2f',valor, dcs) FROM entrada;
+----+--------------+----------+-----------------------------+
| id | transacao_id | conta_id | printf('%15.2f',valor, dcs) |
+----+--------------+----------+-----------------------------+
| 1  | 1            | 2        |           75.40             |
| 2  | 2            | 3        |          482.86             |
| 3  | 3            | 4        |         4877.14             |
| 4  | 4            | 20       |         1940.42             |
| 5  | 4            | 3        |         1940.42             |
| 6  | 5            | 3        |          967.12             |
| 7  | 5            | 14       |          967.12             |
| 8  | 6            | 3        |           80.78             |
| 9  | 6            | 9        |           80.78             |
| 10 | 7            | 3        |           98.99             |
| 11 | 7            | 12       |           98.99             |
| 12 | 8            | 2        |          135.40             |
| 13 | 8            | 11       |          135.40             |
| 14 | 9            | 2        |          201.40             |
| 15 | 9            | 15       |          201.40             |
| 16 | 10           | 21       |          150.00             |
| 17 | 10           | 2        |          150.00             |
| 18 | 11           | 3        |          250.00             |
| 19 | 11           | 2        |          250.00             |
| 20 | 12           | 3        |           29.89             |
| 21 | 12           | 18       |           29.89             |
| 22 | 13           | 2        |          100.00             |
| 23 | 13           | 4        |          100.00             |
| 24 | 14           | 4        |           17.32             |
| 25 | 14           | 3        |           17.32             |
+----+--------------+----------+-----------------------------+

<b>sqlite></b> SELECT conta.codigo as Codigo, conta.descricao as Conta, conta.natureza AS 'D/C', printf('%15.2f',SUM(IIF(entrada.dcs="D" OR (entrada.dcs="S" and conta.natureza="D"),(-1)*entrada.valor,IIF(entrada.dcs="C" OR (entrada.dcs="S" and conta.natureza="C"),entrada.valor,0)))) AS Saldo FROM entrada INNER JOIN conta ON entrada.conta_id = conta.id INNER JOIN transacao ON entrada.transacao_id = transacao.id GROUP BY conta_id HAVING transacao.data <= '2020/01/31' ORDER BY Codigo;
+--------+----------------------+-----+-----------------+
| Codigo |        Conta         | D/C |      Saldo      |
+--------+----------------------+-----+-----------------+
| 11110  | Dinheiro em Carteira | D   |          -38.60 |
| 11120  | Conta Bancaria       | D   |        -1013.82 |
| 11130  | Aplicacao Financeira | D   |        -4959.82 |
| 31111  | Energia Eletrica     | D   |          -80.78 |
| 31113  | Gas de Cozinha       | D   |         -135.40 |
| 31114  | Telefone             | D   |          -98.99 |
| 31161  | Supermercados        | D   |         -967.12 |
| 31162  | Farmacias            | D   |         -201.40 |
| 31190  | Impostos             | D   |          -29.89 |
| 41110  | Salarios e Proventos | C   |         1940.42 |
| 41120  | Servicos Diversos    | C   |          150.00 |
+--------+----------------------+-----+-----------------+

</pre>
# Referências