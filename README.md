# Data Warehouse com Apache Hive
### Projeto nº5 da Formação Cientista de Dados da Data Science Academy

## Introdução

O objetivo deste projeto é implementar um *data warehouse* rodando sobre um *data lake* com Apache Hadoop, transferindo os dados de um banco de dados MySQL para o Apache Hive rodando sobre o sistema de arquivos HDFS de um cluster Hadoop, permitindo manipulações com o Apache Spark.

O projeto será executado em três passos:
- Passo 1 - carregar o banco de dados no MySQL;
- Passo 2 - transferir o banco de dados do MySQL para o Hive;
- Passo 3 - processar os dados no Hive com o Spark;

Requisitos para execução do projeto como descrito neste documento:
- Sistema operacional Linux;
- Cluster Apache Hadoop instalado com Apache Hive, Apache Sqoop e Apache Spark;
- RDBMS MySQL instalado e um banco de dados a ser transferido para o cluster;
- JDBC MySQL instalado para fazer a conexão entre o MySQL e o cluster Hadoop.

## Carga de dados no MySQL:

Dado o banco de dados AWBackup.sql anexo ao projeto, para fazer a carga deste banco de dados no MySQL basta digitar no terminal o seguinte comando:

`mysql -u root -p < ./AWBackup.sql`

Alternativamente, pode-se executar o script import_adventurework.sh.

Verificando se foi criado o novo banco de dados no MySQL:
![Figura 1](./img/fig_1.png)

```
mysql> use adventureworks;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+---------------------------------------+
| Tables_in_adventureworks              |
+---------------------------------------+
| address                               |
| addresstype                           |
| awbuildversion                        |
| billofmaterials                       |
| contact                               |
| contactcreditcard                     |
| contacttype                           |
| countryregion                         |
| countryregioncurrency                 |
| creditcard                            |
| culture                               |
| currency                              |
| currencyrate                          |
| customer                              |
| customeraddress                       |
| databaselog                           |
| department                            |
| document                              |
| employee                              |
| employeeaddress                       |
| employeedepartmenthistory             |
| employeepayhistory                    |
| errorlog                              |
| illustration                          |
| individual                            |
| jobcandidate                          |
| location                              |
| product                               |
| productcategory                       |
| productcosthistory                    |
| productdescription                    |
| productdocument                       |
| productinventory                      |
| productlistpricehistory               |
| productmodel                          |
| productmodelillustration              |
| productmodelproductdescriptionculture |
| productphoto                          |
| productproductphoto                   |
| productreview                         |
| productsubcategory                    |
| productvendor                         |
| purchaseorderdetail                   |
| purchaseorderheader                   |
| salesorderdetail                      |
| salesorderheader                      |
| salesorderheadersalesreason           |
| salesperson                           |
| salespersonquotahistory               |
| salesreason                           |
| salestaxrate                          |
| salesterritory                        |
| salesterritoryhistory                 |
| scrapreason                           |
| shift                                 |
| shipmethod                            |
| shoppingcartitem                      |
| specialoffer                          |
| specialofferproduct                   |
| stateprovince                         |
| store                                 |
| storecontact                          |
| transactionhistory                    |
| transactionhistoryarchive             |
| unitmeasure                           |
| vendor                                |
| vendoraddress                         |
| vendorcontact                         |
| workorder                             |
| workorderrouting                      |
+---------------------------------------+
70 rows in set (0.00 sec)

```
Constata-se, então, que o banco de dados está acessível localmente através do MySQL RDBMS, onde constatou-se que possui 70 tabelas, e localmente pode-se fazer manipulações nestes dados através de linguagem SQL.


## Carga de dados no *data lake*:

Pode-se utilizar o Apache Sqoop para fazer a carga dos dados do banco de dados relacional para o HDFS. Para isto basta executar o seguinte comando no terminal:

```
sqoop import-all-tables --connect jdbc:mysql://localhost:3306/adventureworks?serverTimezone=UTC --username root --password Dsahadoop@1 --m 1

```
O comando acima executado importa para o HDFS todas as 70 tabelas do banco de dados *adventureworks* rodando na máquina local, acessado pela porta 3306 (padrão do MySQL), através do driver JDBC. A saída do terminal pode ser observada no arquivo sqoop import-all-tables --c.log. Abaixo pode-se verificar que os dados foram transferidos para a estrutura /user/hadoop/ em formato texto, em um diretório para cada uma das tabelas do banco de dados.

```
[hadoop@dataserver ~]$ hdfs dfs -ls /user/hadoop
Found 71 items
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:38 /user/hadoop/address
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:38 /user/hadoop/addresstype
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:39 /user/hadoop/awbuildversion
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:39 /user/hadoop/billofmaterials
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:40 /user/hadoop/contact
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:40 /user/hadoop/contactcreditcard
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:40 /user/hadoop/contacttype
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:41 /user/hadoop/countryregion
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:41 /user/hadoop/countryregioncurrency
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:41 /user/hadoop/creditcard
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:42 /user/hadoop/culture
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:42 /user/hadoop/currency
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:43 /user/hadoop/currencyrate
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:43 /user/hadoop/customer
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:43 /user/hadoop/customeraddress
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:44 /user/hadoop/databaselog
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:44 /user/hadoop/department
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:44 /user/hadoop/document
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:45 /user/hadoop/employee
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:45 /user/hadoop/employeeaddress
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:45 /user/hadoop/employeedepartmenthistory
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:46 /user/hadoop/employeepayhistory
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:46 /user/hadoop/errorlog
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:47 /user/hadoop/illustration
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:47 /user/hadoop/individual
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:47 /user/hadoop/jobcandidate
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:48 /user/hadoop/location
drwxrwxrwx   - hadoop supergroup          0 2021-04-01 12:54 /user/hadoop/pet
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:48 /user/hadoop/product
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:48 /user/hadoop/productcategory
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:49 /user/hadoop/productcosthistory
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:49 /user/hadoop/productdescription
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:49 /user/hadoop/productdocument
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:50 /user/hadoop/productinventory
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:50 /user/hadoop/productlistpricehistory
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:51 /user/hadoop/productmodel
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:51 /user/hadoop/productmodelillustration
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:51 /user/hadoop/productmodelproductdescriptionculture
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:52 /user/hadoop/productphoto
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:52 /user/hadoop/productproductphoto
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:52 /user/hadoop/productreview
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:53 /user/hadoop/productsubcategory
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:53 /user/hadoop/productvendor
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:53 /user/hadoop/purchaseorderdetail
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:54 /user/hadoop/purchaseorderheader
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:54 /user/hadoop/salesorderdetail
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:55 /user/hadoop/salesorderheader
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:55 /user/hadoop/salesorderheadersalesreason
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:55 /user/hadoop/salesperson
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:56 /user/hadoop/salespersonquotahistory
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:56 /user/hadoop/salesreason
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:56 /user/hadoop/salestaxrate
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:57 /user/hadoop/salesterritory
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:57 /user/hadoop/salesterritoryhistory
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:57 /user/hadoop/scrapreason
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:58 /user/hadoop/shift
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:58 /user/hadoop/shipmethod
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:58 /user/hadoop/shoppingcartitem
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:59 /user/hadoop/specialoffer
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:59 /user/hadoop/specialofferproduct
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:59 /user/hadoop/stateprovince
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 13:00 /user/hadoop/store
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 13:00 /user/hadoop/storecontact
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 13:01 /user/hadoop/transactionhistory
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 13:01 /user/hadoop/transactionhistoryarchive
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 13:01 /user/hadoop/unitmeasure
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 13:02 /user/hadoop/vendor
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 13:02 /user/hadoop/vendoraddress
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 13:02 /user/hadoop/vendorcontact
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 13:03 /user/hadoop/workorder
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 13:03 /user/hadoop/workorderrouting
[hadoop@dataserver ~]$ hdfs dfs -ls /user/hadoop/store
Found 2 items
-rw-r--r--   1 hadoop supergroup          0 2021-05-18 13:00 /user/hadoop/store/_SUCCESS
-rw-r--r--   1 hadoop supergroup     363123 2021-05-18 13:00 /user/hadoop/store/part-m-00000
```

## Carga de dados no *data warehouse*:

Contudo, utilizar o Apache Hive para 

Deve-se certificar que existe uma estrutura de diretórios no HDFS tal como:

```
[hadoop@dataserver ~]$ hdfs dfs -ls /user/hive
Found 1 items
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:04 /user/hive/warehouse

```
Esta estrutura de diretórios é onde o Hive cria o *warehouse* no HDFS.

Com o Hive devidamente configurado e o banco de dados *adventureworks* carregado no MySQL, utiliza-se o Sqoop para carga dos dados no Apache Hive. Bastando para isto digitar o comando no terminal:

