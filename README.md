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

Para carga de dados no *data lake*, a primeira coisa a ser feita é se certificar que os serviços do HDFS e do Yarn foram inicializados corretamente, conforme mostrado:

```
[hadoop@dataserver ~]$ jps
4066 Jps
[hadoop@dataserver ~]$ start-dfs.sh 
Starting namenodes on [localhost]
Starting datanodes
Starting secondary namenodes [dataserver]
[hadoop@dataserver ~]$ start-yarn.sh 
Starting resourcemanager
Starting nodemanagers
[hadoop@dataserver ~]$ jps
4384 DataNode
5156 NodeManager
4598 SecondaryNameNode
4251 NameNode
5020 ResourceManager
6077 Jps

```

Pode-se utilizar o Apache Sqoop para fazer a carga dos dados do banco de dados relacional para o HDFS. Para isto basta executar o seguinte comando no terminal:

```
sqoop import-all-tables --connect jdbc:mysql://localhost:3306/adventureworks?serverTimezone=UTC --username root --password Dsahadoop@1 --m 1

```
O comando acima executado importa para o HDFS todas as 70 tabelas do banco de dados *adventureworks* rodando na máquina local, acessado pela porta 3306 (padrão do MySQL), através do driver JDBC. A saída do terminal pode ser observada no arquivo sqoop import-all-tables --c.log. Abaixo pode-se verificar que os dados foram transferidos para a estrutura /user/hadoop/ em formato texto, em um diretório para cada uma das tabelas do banco de dados.

```
[hadoop@dataserver ~]$ hdfs dfs -ls /user/hadoop
Found 70 items
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 12:59 /user/hadoop/address
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 12:59 /user/hadoop/addresstype
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:00 /user/hadoop/awbuildversion
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:00 /user/hadoop/billofmaterials
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:00 /user/hadoop/contact
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:01 /user/hadoop/contactcreditcard
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:01 /user/hadoop/contacttype
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:02 /user/hadoop/countryregion
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:02 /user/hadoop/countryregioncurrency
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:02 /user/hadoop/creditcard
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:03 /user/hadoop/culture
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:03 /user/hadoop/currency
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:03 /user/hadoop/currencyrate
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:04 /user/hadoop/customer
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:04 /user/hadoop/customeraddress
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:05 /user/hadoop/databaselog
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:05 /user/hadoop/department
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:05 /user/hadoop/document
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:06 /user/hadoop/employee
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:06 /user/hadoop/employeeaddress
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:07 /user/hadoop/employeedepartmenthistory
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:07 /user/hadoop/employeepayhistory
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:07 /user/hadoop/errorlog
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:08 /user/hadoop/illustration
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:08 /user/hadoop/individual
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:08 /user/hadoop/jobcandidate
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:09 /user/hadoop/location
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:09 /user/hadoop/product
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:10 /user/hadoop/productcategory
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:10 /user/hadoop/productcosthistory
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:10 /user/hadoop/productdescription
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:11 /user/hadoop/productdocument
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:11 /user/hadoop/productinventory
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:12 /user/hadoop/productlistpricehistory
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:12 /user/hadoop/productmodel
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:12 /user/hadoop/productmodelillustration
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:13 /user/hadoop/productmodelproductdescriptionculture
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:13 /user/hadoop/productphoto
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:14 /user/hadoop/productproductphoto
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:14 /user/hadoop/productreview
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:14 /user/hadoop/productsubcategory
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:15 /user/hadoop/productvendor
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:15 /user/hadoop/purchaseorderdetail
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:16 /user/hadoop/purchaseorderheader
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:16 /user/hadoop/salesorderdetail
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:16 /user/hadoop/salesorderheader
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:17 /user/hadoop/salesorderheadersalesreason
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:17 /user/hadoop/salesperson
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:18 /user/hadoop/salespersonquotahistory
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:18 /user/hadoop/salesreason
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:18 /user/hadoop/salestaxrate
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:19 /user/hadoop/salesterritory
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:19 /user/hadoop/salesterritoryhistory
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:20 /user/hadoop/scrapreason
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:20 /user/hadoop/shift
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:20 /user/hadoop/shipmethod
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:21 /user/hadoop/shoppingcartitem
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:21 /user/hadoop/specialoffer
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:21 /user/hadoop/specialofferproduct
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:22 /user/hadoop/stateprovince
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:22 /user/hadoop/store
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:23 /user/hadoop/storecontact
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:23 /user/hadoop/transactionhistory
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:23 /user/hadoop/transactionhistoryarchive
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:24 /user/hadoop/unitmeasure
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:24 /user/hadoop/vendor
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:25 /user/hadoop/vendoraddress
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:25 /user/hadoop/vendorcontact
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:26 /user/hadoop/workorder
drwxr-xr-x   - hadoop supergroup          0 2021-05-19 13:26 /user/hadoop/workorderrouting
[hadoop@dataserver ~]$ hdfs dfs -ls /user/hadoop/customer
Found 2 items
-rw-r--r--   1 hadoop supergroup          0 2021-05-19 13:04 /user/hadoop/customer/_SUCCESS
-rw-r--r--   1 hadoop supergroup    1746278 2021-05-19 13:04 /user/hadoop/customer/part-m-00000

```

## Carga de dados no *data warehouse*:

Contudo, utilizar o Apache Hive rodando sobre o HDFS permite uma maior facilidade na manipulação dos dados no cluster, pois pode-se utilizar a linguagem HQL, bastante similar ao já bem conhecido SQL padrão.

Antes de tudo, deve-se certificar que existe uma estrutura de diretórios no HDFS tal como:

```
[hadoop@dataserver ~]$ hdfs dfs -ls /user/hive
Found 1 items
drwxr-xr-x   - hadoop supergroup          0 2021-05-18 12:04 /user/hive/warehouse

```
Esta estrutura de diretórios é onde o Hive cria o *warehouse* no HDFS.

Com o Hive devidamente configurado e o banco de dados *adventureworks* carregado no MySQL, utiliza-se o Sqoop para carga dos dados no Apache Hive. Bastando para isto digitar o comando no terminal:

```
sqoop import-all-tables --connect jdbc:mysql://localhost:3306/adventureworks?serverTimezone=UTC --username root --password Dsahadoop@1 --hive-import
```

```
[hadoop@dataserver bin]$ schematool -dbType derby -initSchema
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/hive/lib/log4j-slf4j-impl-2.10.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/opt/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Metastore connection URL:	 jdbc:derby:;databaseName=metastore_db;create=true
Metastore Connection Driver :	 org.apache.derby.jdbc.EmbeddedDriver
Metastore connection User:	 APP
Starting metastore schema initialization to 3.1.0
Initialization script hive-schema-3.1.0.derby.sql


Initialization script completed
schemaTool completed

```

Último teste:
[hadoop@dataserver ~]$ sqoop import --connect jdbc:mysql://localhost:3306/adventureworks?serverTimezone=UTC --username root --password Dsahadoop@1 --table customer --hive-import --hive-database adventureworks --hive-table customer ----map-column-hive rowguid=binary --target-dir /user/hive/warehouse/adventureworks --m 1
Error: Could not find or load main class org.apache.hadoop.hbase.util.GetJavaProperty
2021-05-19 14:04:15,702 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
2021-05-19 14:04:15,904 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
2021-05-19 14:04:15,905 ERROR tool.BaseSqoopTool: Error parsing arguments for import:
2021-05-19 14:04:15,905 ERROR tool.BaseSqoopTool: Unrecognized argument: ----map-column-hive
2021-05-19 14:04:15,905 ERROR tool.BaseSqoopTool: Unrecognized argument: rowguid=binary
2021-05-19 14:04:15,905 ERROR tool.BaseSqoopTool: Unrecognized argument: --target-dir
2021-05-19 14:04:15,905 ERROR tool.BaseSqoopTool: Unrecognized argument: /user/hive/warehouse/adventureworks
2021-05-19 14:04:15,905 ERROR tool.BaseSqoopTool: Unrecognized argument: --m
2021-05-19 14:04:15,905 ERROR tool.BaseSqoopTool: Unrecognized argument: 1
