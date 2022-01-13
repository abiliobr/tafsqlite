# Teste de uso de SQLite para registros financeiros

Neste projeto, vamos testar o uso do SQLite para a gerência de informações financeiras de nível pessoal ou doméstico, tais como receitas de salários ou serviços prestados, despesas com alimentação ou transporte, etc. Os passos necessários para o teste são:

1. Projeto de banco de dados de registro de finanças
3. Inserção de dados fictícios
4. Análise de dados

## Parte 1 - O projeto de banco de dados
Inspirado nas ideias apresentadas em https://stackoverflow.com/questions/2494343/database-schema-design-for-a-double-entry-accounting-system e em https://medium.com/@RobertKhou/double-entry-accounting-in-a-relational-database-2b7838a5d7f8, o esquema inicial para as tabelas poderá ficar assim:

- Tabela: ``transacao``
```
     - id
     - data
     - valor
     - historico
     - referente
```
- Tabela: conta
```
     - id
     - descricao
     - dc
     - codigo
```
- Tabela: partida
```
     - id
     - conta_id
     - transacao_id
     - valor
```
A tabela ``partida`` representa os registros financeiros de uma transação associados a contas na tabela ``conta``, segundo o método contábil das "partidas dobradas". Portanto, a tabela ``partida`` representa o relacionamento "muitos-para-muitos" no esquema abaixo:
```
transacao -----< partida >----- conta
```
### Criação do banco de dados e tabelas

Usando o SQLite, foram executadas as seguintes DDLs para as 3 tabelas:
<pre>
<b>
$ sqlite3 tafsqlite.db3
sqlite>
</b>
CREATE TABLE transacao (
    id        INTEGER         PRIMARY KEY AUTOINCREMENT
                              NOT NULL,
    data      DATE            NOT NULL,
    valor     DECIMAL (15, 2) NOT NULL,
    historico VARCHAR (100)   NOT NULL,
    referente VARCHAR (50) 
);
<b>
sqlite>
</b>
CREATE TABLE conta (
    id        INTEGER       PRIMARY KEY AUTOINCREMENT
                            NOT NULL,
    descricao VARCHAR (100) NOT NULL,
    dc        VARCHAR (1)   NOT NULL
                            CHECK (dc IN ("C", "D") ),
    codigo    VARCHAR (3)   NOT NULL
);
<b>
sqlite>
</b>
CREATE TABLE partida (
    id           INTEGER         PRIMARY KEY AUTOINCREMENT
                                 NOT NULL,
    transacao_id INTEGER         REFERENCES transacao (id) ON DELETE CASCADE
                                                           ON UPDATE CASCADE
                                 NOT NULL,
    conta_id     INTEGER         REFERENCES conta (id) ON DELETE CASCADE
                                                       ON UPDATE CASCADE
                                 NOT NULL,
    valor        DECIMAL (15, 2) NOT NULL
);
<b>
sqlite> .tables
conta      partida    transacao

sqlite> .quit
</b>
</pre>
## Parte 2 - Inserção de dados nas tabelas

Começando pela tabela ``transacao``, vamos simular os seguintes eventos fictícios:
1. Recebimento de salário de 2.740,42 da empresa Contabilidade Pacioli em 05/01/2020 referente ao mês de dezembro/2019;
2. Recebimento de uma venda de uma bicicleta usada no valor de 450,00 no dia 10/01/2020 para um vizinho;
3. Ganho de 17,32 num rendimento de aplicacao financeira no final do mês de janeiro de 2020;