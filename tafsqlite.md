# Estudo de uso de um banco de dados SQLite para registros financeiros

Neste projeto, vamos testar o uso de um banco de dados em SQLite para a gerência de informações financeiras no âmbito pessoal ou doméstico, tais como receitas de salários ou serviços prestados, despesas com alimentação ou transporte, etc. Os passos necessários para o teste são:

1. Conhecimento dos elementos financeiros
2. Projeto de banco de dados
3. Inserção de dados fictícios
4. Análise de dados
5. Avaliação da estrutura do banco de dados

## Parte 1 - Elementos financeiros

O patrimônio pessoal é representado por diversos elementos: dinheiro físico, contas bancárias, imóveis, veículos, direitos autorais, obrigações financeiras, etc. Neste estudo, vamos considerar apenas a existência de dinheiro em espécie ou disponível em uma conta bancária ou aplicação financeira no patrimônio atual.

Quanto ao passivo, que são as obrigações financeiras (prestações, empréstimos contraídos, etc) vamos considerar apenas contas a pagar e o patrimônio líquido (também chamado genericamente de "equidade") para simplificar a demonstração.

As ocorrências financeiras precisam ser organizadas em categorias fixas, chamadas de "contas". Nesse estudo, algumas contas recorrentes serão: recebimento de salário, compras em lojas, contas de energia e telefone, pagamento de impostos, despesa com transporte, etc.

## Parte 2 - O projeto de banco de dados
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

A tabela **`entrada`** contém os registros que ligam uma transação a uma ou mais contas na tabela **`conta`**, para atender ao método contábil chamado "partidas dobradas". Portanto, a tabela **`entrada`** representa o relacionamento "muitos-para-muitos" conforme o esquema abaixo:

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

## Parte 3 - Inserção de dados nas tabelas

### Um plano de contas

As contas devem possuir um código para facilitar a classificação, como mostrado a seguir. Dependendo da necessidade, este esquema de contas poderá ser expandido para ajustar-se a novas ocorrências.
<pre>
1     - Ativo
11110 - Dinheiro em Carteira
11120 - Conta Bancária
11130 - Aplicação Financeira
2     - Passivo
21150 - Contas a Pagar
23100 - Patrimonio Liquido
23199 - Saldos de Abertura
3     - Despesas
31111 - Energia Elétrica
31112 - Água e Esgoto
31113 - Telefone
31115 - Transportes
31161 - Supermercados
31162 - Saude
31163 - Lojas Diversas
31163 - Serviços Diversos
31190 - Impostos
4     - Receitas
41110 - Salários e Proventos
41120 - Serviços Prestados
41150 - Outras Receitas
41190 - Rendimentos de Aplicações Financeiras
</pre>

Os registros na tabela `contas` inicialmente serão estes:
```sqlite
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Ativo',1,'D',1);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Dinheiro em Carteira',11110,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Conta Bancaria',11120,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Aplicacao Financeira',11130,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Passivo',2,'C',1);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Contas a Pagar',21150,'C',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Patrimonio Liquido',23100,'C',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Saldos de Abertura',23199,'C',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Despesas',3,'D',1);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Energia Eletrica',31111,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Agua e Esgoto',31112,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Telefone',31113,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Transportes',31150,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Supermercados',31161,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Saude',31162,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Lojas Diversas',31163,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Servicos Diversos',31180,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Impostos',31190,'D',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Receitas',4,'C',1);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Salarios e Proventos',41110,'C',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Servicos Prestados',41120,'C',0);
INSERT INTO conta (descricao, codigo, natureza, sintetica) VALUES('Outras Receitas',41150,'C',0);
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
| 6  | Contas a Pagar                      | 21150  | C        | 0         |
| 7  | Patrimonio Liquido                  | 23100  | C        | 0         |
| 8  | Saldos de Abertura                  | 23199  | C        | 0         |
| 9  | Despesas                            | 3      | D        | 1         |
| 10 | Energia Eletrica                    | 31111  | D        | 0         |
| 11 | Agua e Esgoto                       | 31112  | D        | 0         |
| 12 | Telefone                            | 31113  | D        | 0         |
| 13 | Transportes                         | 31150  | D        | 0         |
| 14 | Supermercados                       | 31161  | D        | 0         |
| 15 | Saude                               | 31162  | D        | 0         |
| 16 | Lojas Diversas                      | 31163  | D        | 0         |
| 17 | Servicos Diversos                   | 31180  | D        | 0         |
| 18 | Impostos                            | 31190  | D        | 0         |
| 19 | Receitas                            | 4      | C        | 1         |
| 20 | Salarios e Proventos                | 41110  | C        | 0         |
| 21 | Servicos Prestados                  | 41120  | C        | 0         |
| 22 | Outras Receitas                     | 41150  | C        | 0         |
| 23 | Rendimentos de Aplicacao Financeira | 41190  | C        | 0         |
+----+-------------------------------------+--------+----------+-----------+
</pre>

### As transações

Na tabela **`transacao`**, vamos registrar ocorrências financeira fictícias ao longo de alguns meses, como receitas, despesas, saques e depósitos.

Os primeiros registros serão os saldos iniciais de contas do ativo: dinheiro em espécie, saldo da conta bancária e saldo da aplicação financeira. Inicialmente, não haverá obrigações pendentes (passivos).

```sqlite
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-01-01',     75.40,'Saldo Anterior em Carteira','31/12/2019');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-01-01',    482.86,'Saldo Anterior em Conta Corrente','31/12/2019');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-01-01',   4877.14,'Saldo Anterior em Aplicacao Financeira','31/12/2019');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-01-05',   1940.42,'Proventos','dezembro/2019');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-01-06',    967.12,'Compras no supermercado XXX',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-01-09',     80.78,'Pagto de conta de energia eletrica','06/12/2019 a 05-01-2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-01-10',     98.99,'Pagto de conta de telefone','dezembro/2019');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-01-12',    201.40,'Compras na farmacia YYY',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-01-28',    150.00,'Renda de servico de montagem',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-01-29',    250.00,'Saque em conta corrente',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-01-30',     29.89,'Pagto de imposto territorial','2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-01-30',    100.00,'Deposito em aplicacao financeira',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-01-31',     17.32,'Rendimento de aplicacao financeira','31-01-2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-02-05',   1950.11,'Proventos','janeiro/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-02-06',   1267.12,'Compras no supermercado XXX',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-02-09',     84.71,'Pagto de conta de energia eletrica','06/01/2020 a 05/02/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-02-10',     98.99,'Pagto de conta de telefone','janeiro/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-02-14',    140.00,'Venda de maquina usada',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-02-16',     41.27,'Despesa na farmacia YYY',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-02-28',     16.27,'Rendimento de aplicacao financeira','28/02/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-03-05',   1940.42,'Proventos','fevereiro/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-03-06',    807.20,'Compras no supermercado ZZZ',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-03-09',     80.44,'Pagto de conta de energia eletrica','06/02/2020 a 05/03/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-03-10',     98.99,'Pagto de conta de telefone','fevereiro/2020');
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-03-26',    407.70,'Compras no supermercado XXX',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-03-27',    140.00,'Deposito em conta corrente',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-03-28',     50.40,'Despesa de transporte',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-03-29',    107.00,'Pagto de servico de limpeza',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-03-30',   1000.00,'Saque em aplicacao financeira',NULL);
INSERT INTO transacao (data, valor, historico, referente) VALUES('2020-03-31',     16.22,'Rendimento de aplicacao financeira','31/03/2020');

```

> Note-se que, no SQLite, os valores monetários são sempre informados com ponto decimal (.). Já as datas devem estar no formato YYYY-MM-DD para maior compatibilidade.

Após a inserção dos registros, a listagem da tabela **`transacao`** poderá ficar como abaixo:
<pre>
<b>sqlite></b> .mode table
<b>sqlite></b> SELECT id, data, printf('%15.2f', valor), historico, referente FROM transacao;
+----+------------+-------------------------+----------------------------------------+-------------------------+
| id |    data    | printf('%15.2f', valor) |               historico                |        referente        |
+----+------------+-------------------------+----------------------------------------+-------------------------+
| 1  | 2020-01-01 |           75.40         | Saldo Anterior em Carteira             | 31/12/2019              |
| 2  | 2020-01-01 |          482.86         | Saldo Anterior em Conta Corrente       | 31/12/2019              |
| 3  | 2020-01-01 |         4877.14         | Saldo Anterior em Aplicacao Financeira | 31/12/2019              |
| 4  | 2020-01-05 |         1940.42         | Proventos                              | dezembro/2019           |
| 5  | 2020-01-06 |          967.12         | Compras no supermercado XXX            |                         |
| 6  | 2020-01-09 |           80.78         | Pagto de conta de energia eletrica     | 06/12/2019 a 05-01-2020 |
| 7  | 2020-01-10 |           98.99         | Pagto de conta de telefone             | dezembro/2019           |
| 8  | 2020-01-12 |          201.40         | Compras na farmacia YYY                |                         |
| 9  | 2020-01-28 |          150.00         | Renda de servico de montagem           |                         |
| 10 | 2020-01-29 |          250.00         | Saque em conta corrente                |                         |
| 11 | 2020-01-30 |           29.89         | Pagto de imposto territorial           | 2020                    |
| 12 | 2020-01-30 |          100.00         | Deposito em aplicacao financeira       |                         |
| 13 | 2020-01-31 |           17.32         | Rendimento de aplicacao financeira     | 31-01-2020              |
| 14 | 2020-02-05 |         1950.11         | Proventos                              | janeiro/2020            |
| 15 | 2020-02-06 |         1267.12         | Compras no supermercado XXX            |                         |
| 16 | 2020-02-09 |           84.71         | Pagto de conta de energia eletrica     | 06/01/2020 a 05/02/2020 |
| 17 | 2020-02-10 |           98.99         | Pagto de conta de telefone             | janeiro/2020            |
| 18 | 2020-02-14 |          140.00         | Venda de maquina usada                 |                         |
| 19 | 2020-02-16 |           41.27         | Despesa na farmacia YYY                |                         |
| 20 | 2020-02-28 |           16.27         | Rendimento de aplicacao financeira     | 28/02/2020              |
| 21 | 2020-03-05 |         1940.42         | Proventos                              | fevereiro/2020          |
| 22 | 2020-03-06 |          807.20         | Compras no supermercado ZZZ            |                         |
| 23 | 2020-03-09 |           80.44         | Pagto de conta de energia eletrica     | 06/02/2020 a 05/03/2020 |
| 24 | 2020-03-10 |           98.99         | Pagto de conta de telefone             | fevereiro/2020          |
| 25 | 2020-03-26 |          407.70         | Compras no supermercado XXX            |                         |
| 26 | 2020-03-27 |          140.00         | Deposito em conta corrente             |                         |
| 27 | 2020-03-28 |           50.40         | Despesa de transporte                  |                         |
| 28 | 2020-03-29 |          107.00         | Pagto de servico de limpeza            |                         |
| 29 | 2020-03-30 |         1000.00         | Saque em aplicacao financeira          |                         |
| 30 | 2020-03-31 |           16.22         | Rendimento de aplicacao financeira     | 31/03/2020              |
+----+------------+-------------------------+----------------------------------------+-------------------------+
</pre>

### Os saldos iniciais ou de abertura

Na tabela **`entrada`** serão registrados 3 saldos iniciais:
```sqlite
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(1, 8,  75.40,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(1, 2,  75.40,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(2, 8, 482.86,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(2, 3, 482.86,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(3, 8,4877.14,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(3, 4,4877.14,'D');

```
<pre>
<b>sqlite></b> .mode table
<b>sqlite></b> SELECT id, transacao_id, conta_id, printf('%15.2f', valor), dcs FROM entrada;
+----+--------------+----------+-------------------------+-----+
| id | transacao_id | conta_id | printf('%15.2f', valor) | dcs |
+----+--------------+----------+-------------------------+-----+
| 1  | 1            | 8        |           75.40         | C   |
| 2  | 1            | 2        |           75.40         | D   |
| 3  | 2            | 8        |          482.86         | C   |
| 4  | 2            | 3        |          482.86         | D   |
| 5  | 3            | 8        |         4877.14         | C   |
| 6  | 3            | 4        |         4877.14         | D   |
+----+--------------+----------+-------------------------+-----+
</pre>

Depois, a primeira demonstração pode ser obtida assim:
<pre>
<b>sqlite></b> SELECT conta.codigo AS Codigo, conta.descricao AS Conta, 
  printf('%15.2f',ABS(SUM(IIF(entrada.dcs = "D" OR (entrada.dcs = "S" and conta.natureza = "D"), -1 * entrada.valor,
                          IIF(entrada.dcs = "C" OR (entrada.dcs = "S" and conta.natureza = "C"),entrada.valor,0))))) AS Saldo, 
  conta.natureza AS 'D/C' 
FROM entrada 
  INNER JOIN conta ON entrada.conta_id = conta.id 
  INNER JOIN transacao ON entrada.transacao_id = transacao.id 
WHERE 
  transacao.data = '2020-01-01' 
GROUP BY 
  conta_id 
HAVING 
  transacao.data <= '2020-01-01' 
ORDER BY 
  Codigo;
+--------+----------------------+-----------------+-----+
| Codigo |        Conta         |      Saldo      | D/C |
+--------+----------------------+-----------------+-----+
| 11110  | Dinheiro em Carteira |           75.40 | D   |
| 11120  | Conta Bancaria       |          482.86 | D   |
| 11130  | Aplicacao Financeira |         4877.14 | D   |
| 23199  | Saldos de Abertura   |         5435.40 | C   |
+--------+----------------------+-----------------+-----+
</pre>

## Os lançamentos comuns

Para fazer os lançamentos para o restante das transações, devemos observar os seguintes fatos:

* O salário é depositado na conta bancária;
* A renda de servico realizado e o pagamento pela venda da maquina foram feitos com dinheiro em espécie;
* As compras em supermercados geralmente são feitas através de débito automático na conta corrente;
* As contas de energia elétrica, de telefone e os impostos também são pagos através da conta corrente no banco;
* As compras na farmácia, o pagamento pelo serviço de limpeza e o gasto com transporte foram feitos com dinheiro em espécie.

```sqlite
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 4,20, 1940.42,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 4, 3, 1940.42,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 5, 3,  967.12,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 5,14,  967.12,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 6, 3,   80.78,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 6,10,   80.78,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 7, 3,   98.99,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 7,12,   98.99,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 8, 2,  201.40,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 8,15,  201.40,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 9,21,  150.00,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES( 9, 2,  150.00,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(10, 3,  250.00,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(10, 2,  250.00,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(11, 3,   29.89,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(11,18,   29.89,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(12, 2,  100.00,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(12, 4,  100.00,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(13, 4,   17.32,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(13, 3,   17.32,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(14,20, 1950.11,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(14, 3, 1950.11,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(15, 3, 1267.12,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(15,14, 1267.12,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(16, 3,   84.71,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(16,10,   84.71,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(17, 3,   98.99,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(17,12,   98.99,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(18,22,  140.00,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(18, 2,  140.00,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(19, 2,   41.27,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(19,15,   41.27,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(20, 4,   16.27,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(20, 3,   16.27,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(21,20, 1940.42,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(21, 3, 1940.42,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(22, 3,  807.20,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(22,14,  807.20,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(23, 3,   80.44,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(23,10,   80.44,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(24, 3,   98.99,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(24,12,   98.99,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(25, 3,  407.70,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(25,14,  407.70,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(26, 2,  140.00,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(26, 4,  140.00,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(27, 2,   50.40,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(27,13,   50.40,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(28, 2,  107.00,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(28,17,  107.00,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(29, 4, 1000.00,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(29, 2, 1000.00,'D');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(30, 4,   16.22,'C');
INSERT INTO entrada (transacao_id, conta_id, valor, dcs) VALUES(30, 3,   16.22,'D');

```
<pre>
<b>sqlite></b> SELECT id, transacao_id, conta_id, printf('%15.2f',valor), dcs FROM entrada;
+----+--------------+----------+------------------------+-----+
| id | transacao_id | conta_id | printf('%15.2f',valor) | dcs |
+----+--------------+----------+------------------------+-----+
| 1  | 1            | 8        |           75.40        | C   |
| 2  | 1            | 2        |           75.40        | D   |
| 3  | 2            | 8        |          482.86        | C   |
| 4  | 2            | 3        |          482.86        | D   |
| 5  | 3            | 8        |         4877.14        | C   |
| 6  | 3            | 4        |         4877.14        | D   |
| 7  | 4            | 20       |         1940.42        | C   |
| 8  | 4            | 3        |         1940.42        | D   |
| 9  | 5            | 3        |          967.12        | C   |
| 10 | 5            | 14       |          967.12        | D   |
| 11 | 6            | 3        |           80.78        | C   |
| 12 | 6            | 10       |           80.78        | D   |
| 13 | 7            | 3        |           98.99        | C   |
| 14 | 7            | 12       |           98.99        | D   |
| 15 | 8            | 2        |          201.40        | C   |
| 16 | 8            | 15       |          201.40        | D   |
| 17 | 9            | 21       |          150.00        | C   |
| 18 | 9            | 2        |          150.00        | D   |
| 19 | 10           | 3        |          250.00        | C   |
| 20 | 10           | 2        |          250.00        | D   |
| 21 | 11           | 3        |           29.89        | C   |
| 22 | 11           | 18       |           29.89        | D   |
| 23 | 12           | 2        |          100.00        | C   |
| 24 | 12           | 4        |          100.00        | D   |
| 25 | 13           | 4        |           17.32        | C   |
| 26 | 13           | 3        |           17.32        | D   |
| 27 | 14           | 20       |         1950.11        | C   |
| 28 | 14           | 3        |         1950.11        | D   |
| 29 | 15           | 3        |         1267.12        | C   |
| 30 | 15           | 14       |         1267.12        | D   |
| 31 | 16           | 3        |           84.71        | C   |
| 32 | 16           | 10       |           84.71        | D   |
| 33 | 17           | 3        |           98.99        | C   |
| 34 | 17           | 12       |           98.99        | D   |
| 35 | 18           | 22       |          140.00        | C   |
| 36 | 18           | 2        |          140.00        | D   |
| 37 | 19           | 2        |           41.27        | C   |
| 38 | 19           | 15       |           41.27        | D   |
| 39 | 20           | 4        |           16.27        | C   |
| 40 | 20           | 3        |           16.27        | D   |
| 41 | 21           | 20       |         1940.42        | C   |
| 42 | 21           | 3        |         1940.42        | D   |
| 43 | 22           | 3        |          807.20        | C   |
| 44 | 22           | 14       |          807.20        | D   |
| 45 | 23           | 3        |           80.44        | C   |
| 46 | 23           | 10       |           80.44        | D   |
| 47 | 24           | 3        |           98.99        | C   |
| 48 | 24           | 12       |           98.99        | D   |
| 49 | 25           | 3        |          407.70        | C   |
| 50 | 25           | 14       |          407.70        | D   |
| 51 | 26           | 2        |          140.00        | C   |
| 52 | 26           | 4        |          140.00        | D   |
| 53 | 27           | 2        |           50.40        | C   |
| 54 | 27           | 13       |           50.40        | D   |
| 55 | 28           | 2        |          107.00        | C   |
| 56 | 28           | 17       |          107.00        | D   |
| 57 | 29           | 4        |         1000.00        | C   |
| 58 | 29           | 2        |         1000.00        | D   |
| 59 | 30           | 4        |           16.22        | C   |
| 60 | 30           | 3        |           16.22        | D   |
+----+--------------+----------+------------------------+-----+

<b>sqlite></b> SELECT conta.codigo AS Codigo, conta.descricao AS Conta, 
  printf('%15.2f',ABS(SUM(IIF(entrada.dcs = "D" OR (entrada.dcs = "S" and conta.natureza = "D"), (-1) * entrada.valor,
                          IIF(entrada.dcs = "C" OR (entrada.dcs = "S" and conta.natureza = "C"),entrada.valor,0))))) AS Saldo, 
  conta.natureza AS 'D/C' 
FROM entrada 
  INNER JOIN conta ON entrada.conta_id = conta.id 
  INNER JOIN transacao ON entrada.transacao_id = transacao.id 
GROUP BY 
  conta_id 
HAVING 
  transacao.data <= '2020-03-31' and conta.codigo < 99999
ORDER BY 
  Codigo;
+--------+----------------------+-----------------+-----+
| Codigo |        Conta         |      Saldo      | D/C |
+--------+----------------------+-----------------+-----+
| 11110  | Dinheiro em Carteira |          975.33 | D   |
| 11120  | Conta Bancaria       |         2091.69 | D   |
| 11130  | Aplicacao Financeira |         4067.33 | D   |
| 23199  | Saldos de Abertura   |         5435.40 | C   |
| 31111  | Energia Eletrica     |          245.93 | D   |
| 31113  | Telefone             |          296.97 | D   |
| 31150  | Transportes          |           50.40 | D   |
| 31161  | Supermercados        |         3449.14 | D   |
| 31162  | Saude                |          242.67 | D   |
| 31180  | Servicos Diversos    |          107.00 | D   |
| 31190  | Impostos             |           29.89 | D   |
| 41110  | Salarios e Proventos |         5830.95 | C   |
| 41120  | Servicos Prestados   |          150.00 | C   |
| 41150  | Outras Receitas      |          140.00 | C   |
+--------+----------------------+-----------------+-----+
</pre>

## Parte 4 - Análise dos dados

<!--
Frequencias de lancancmentos de acordo com o historico:
select conta.id, entrada.dcs, transacao.historico, conta.descricao, count(conta.codigo) as TotalFreq from entrada inner join conta on entrada.conta_id = conta.id inner join transacao on entrada.transacao_id = transacao.id group by transacao.historico, conta.codigo order by transacao.historico, TotalFreq desc, entrada.dcs ;
-->

<!--
sqlite> delete from transacao;
sqlite> delete from sqlite_sequence where name='transacao';
sqlite> delete from conta;
sqlite> delete from sqlite_sequence where name='conta';
sqlite> delete from entrada;
sqlite> delete from sqlite_sequence where name='entrada';
-->
## Parte 5 - Análise da estrutura do banco de dados

# Referências