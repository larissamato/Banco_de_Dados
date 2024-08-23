## 1.INTRODUÇÃO

​	Esse documento apresenta informações sobre o banco de dados do projeto Big Data de uma atividade da faculdade, sendo elas, descrições gerais, dicionário de dados, modelo entidade relacionamento, esquema relacional, normalização, exemplos de inserções, SQLs avançadas e suas representações gráficas. Os dados obtidos são de uma base aberta do IBAMA denominado: Emissões de Poluentes Atmosféricos por meio de um arquivo de extensão .csv.

​	As informações tratadas se referem a pessoas jurídicas cadastradas por município no formulário Técnico Federal de Atividades Potencialmente Poluidoras e Utilizadoras de Recursos Naturais – CTF/APP, juntamente com a sua atividade desenvolvida, situação cadastral, poluentes emitidos, suas quantidades, e por fim, a metodologia utilizada para obter esses dados.

## 2.DICIONÁRIO DE DADOS

**Local**

| **Atributo** | **Classe**             | **Domínio** | **Tamanho** | **Descrição**                                                |
| ------------ | ---------------------- | ----------- | ----------- | ------------------------------------------------------------ |
| CodLocal     | chave 		primária | Inteiro     | -           | Atributo 		identificador da tabela Local. Not Null     |
| Estado       | simples                | Varchar     | (30)        | O 		estado do local da empresa. Not Null               |
| Cidade       | simples                | Varchar     | (50)        | A 		cidade pertencente ao Estado do local da empresa. Not Null |

**Empresa**

| **Atributo** | **Classe**                | **Domínio** | **Tamanho** | **Descrição**                                               |
| ------------ | ------------------------- | ----------- | ----------- | ----------------------------------------------------------- |
| Cnpj         | chave primária            | Varchar     | (18)        | Atributo 		identificador da pessoa jurídica. Not Null |
| CodLocalRef  | chave 		estrangeira | Inteiro     | -           | Local da 		empresa. Not Null                          |
| Nome         | simples                   | Varchar     | (70)        | Nome da 		empresa. Not Null                           |
| Categoria    | simples                   | Varchar     | (70)        | Categoria na 		qual a empresa está inserida. Not Null |

**Detalhe**

| **Atributo**   | **Classe**                | **Domínio** | **Tamanho** | **Descrição**                                                |
| -------------- | ------------------------- | ----------- | ----------- | ------------------------------------------------------------ |
| CodDetalhe     | chave primária            | Inteiro     | -           | Atributo 		identificador da relação da Pesquisa com a Empresa |
| CodEmpresaRef  | chave 		estrangeira | Varchar     | (18)        | Referente ao 		cnpj da empresa                         |
| CodPesquisaRef | chave 		estrangeira | Inteiro     | -           | Referente a 		pesquisa anual                           |
| Segmento       | simples                   | Varchar     | (300)       | Detalhe sobre 		qual a atividade específica da empresa daquele ano |
| Situacao       | simples                   | Varchar     | (30)        | Descreve a 		situação da empresa como ativa, encerrada ou em análise 		cadastral daquele ano |

**Pesquisa**

| **Atributo** | **Classe**     | **Domínio** | **Tamanho** | **Descrição**                                               |
| ------------ | -------------- | ----------- | ----------- | ----------------------------------------------------------- |
| CodPesquisa  | chave primária | Inteiro     | -           | Atributo 		identificador da pesquisa                  |
| Ano          | simples        | Inteiro     | -           | Ano da 		obtenção dos dados                           |
| Poluente     | simples        | Varchar     | (30)        | Substâncias 		poluentes emitidas                      |
| Metodologia  | simples        | Varchar     | (10)        | Como foi 		obtido os dados                            |
| Quantidade   | simples        | Float       | -           | Quantitativo 		de emissões de cada empresa anualmente |



## 3.NORMALIZAÇÃO

​	A normalização foi realizada a partir da tabela obtida na base de dados abertos, sendo ela: Tabela Relatório (Cnpj, Razão Social, Estado, Município, Categoria de Atividade, Código do Detalhe, Segmento , Ano, Poluente emitido, Quantidade, Metodologia utilizada, Situação Cadastral).

​	A primeira e a terceira forma normal não estão presente nesse exemplo, por haver atributos multivalorados e dependência transitiva. Dessa forma houve a seguinte divisão de dependência:

(Cnpj) → Razão Social, Categoria da Atividade

(CodDetalhe) → Situação Cadastral, Segmento 

(CodLocal) → Estado , Município 

(CodPesquisa) → Ano, Poluente, Quantidade, Metodologia 

​	As tabelas portanto passaram a ficar da seguinte forma:

Empresa (Cnpj, Razão Social, Categoria da Atividade, CodLocal)

Detalhe (CodDetalhe, Situação Cadastral, Segmento, Cnpj, CodPesquisa)

Local (CodLocal, Estado, Município)

Pesquisa (CodPesquisa, Ano, Poluente, Quantidade, Metodologia)

​	Logo depois dessa mudança todas as tabelas se apresentam na segunda forma normal, por não terem chaves primárias compostas.

## 4. CRIAÇÃO

```sql
create table Local(codLocal int, estado varchar(30), cidade varchar(50),primary key(codLocal));

create table Empresa (cnpj Varchar(18), nome varchar(70), categoria varchar(70),codLocalRef int, primary key(cnpj),Foreign Key(codLocalRef) REFERENCES Local(codLocal));

create table Detalhe(codDetalhe int, codEmpresaRef Varchar(18),segmento varchar(300),situacao varchar(30),  codPesquisaRef int, primary key(codDetalhe), Foreign Key (codEmpresaRef) REFERENCES Empresa(cnpj), Foreign Key(codPesquisaRef) REFERENCES Pesquisa(codPesquisa));

create table Pesquisa (codPesquisa int, ano int, poluente varchar(30),metodologia varchar(10), quantidade float, primary key (codPesquisa));
```



## 5.INSERÇÕES

Abaixo estará um exemplo de inserção para cada tabela do BigData:

```sql
insert into Local (codLocal, estado,cidade) values (2, 'Goiás', 'Abadia de Goiás');
```

```sql
insert into Empresa (cnpj, nome, categoria, codLocalRef) values ( '00.012.377/0001-60 ','CEREAL COM .EXP. E REPRES. AGROPECUÁRIA S/A','Transporte,Terminais,   Depósitos e Comércio', 199);
```

```sql
insert intoDetalhe (codDetalhe, codEmpresaRef, segmento, situacao, codPesquisaRef) values (4, '00.075.140/0001-29',  'Fabricação de chapas, placas de madeira aglomerada, prensada e compensada',  'Ativa', 4);
```

```sql
insert into Pesquisa (codPesquisa, ano, poluente, metodologia, quantidade) values (1, 2013, 'Material Particulado (MP)',  'Estimativa', 0.1);
```



## 6.SQLS AVANÇADAS

```sql
select ano, SUM(quantidade) as Total from Pesquisa group by ano;
```

```sql
select l.estado, count(*) from Empresa as e inner join Local as l on e.codLocalRef=l.codLocal group by l.estado;


```

```sql
select p.quantidade as Total, e.nome as Nome,p.ano as Ano, p.poluente as Poluente from Detalhe as d left join Pesquisa as p on p.codPesquisa=d.codPesquisaRef inner join Empresa as e on d.CodEmpresaRef=e.cnpj where Ano=2020  

ORDER BY Total DESC;
```



