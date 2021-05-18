# Data Warehouse com Apache Hive
### Projeto nº5 da Formação Cientista de Dados da Data Science Academy

## Introdução

O objetivo deste projeto é implementar um *data warehouse* rodando sobre um *data lake* com Apache Hadoop, transferindo os dados de um banco de dados MySQL para o Apache Hive rodando sobre o sistema de arquivos HDFS de um cluster Hadoop, permitindo manipulações com o Apache Spark.

O projeto será executado em três passos:
- Passo 1 - carregar o banco de dados no MySQL;
- Passo 2 - transferir o banco de dados do MySQL para o Hive;
- Passo 3 - processar os dados no Hive com o Spark;

Requisitos para execução:
- Cluster Apache Hadoop instalado com Apache Hive, Apache Sqoop e Apache Spark;
- RDBMS MySQL instalado e um banco de dados a ser transferido para o cluster.

## Carga de dados no MySQL:

Dado o banco de dados AWBackup.sql anexo ao projeto, para fazer a carga deste banco de dados no MySQL basta digitar no terminal o seguinte comando:

`mysql -u root -p < ./AWBackup.sql`

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

