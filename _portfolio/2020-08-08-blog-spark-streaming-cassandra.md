---
title: 'Building a real-time big data pipeline (part 5: Cassandra, CQL, Java)'
date: 2020-08-08
permalink: /posts/2020/08/blog-post-spark-streaming-cassandra/
tags:
  - big data
  - apache cassandra
  - real time data pipelines 
  - Java
  - NoSQL
  - CQL
  - Bioinformatics
  - Emory University 

---  
*Updated on August 13, 2020*  

[Apache Cassandra](https://cassandra.apache.org) is a distributed NoSQL database (DB) which is used for handling Big data and real-time web applications. NoSQL stands for "Not Only SQL" or "Not SQL". NoSQL database is a non-relational data management system, that does not require a fixed schema.  
 
Why NoSQL?  
The system response time becomes slow when you use RDBMS for massive volumes of data. The alternative for this issue is to distribute DB load on multiple hosts whenever the load increase.  
 
NoSQL databases are mainly categorized into four types:  
1. key-value pair (e.g Redis, Dynamo)  
2. column-oriented (e.g HBase, Cassandra)  
3. graph-based (e.g . Neo4J)  
4. document-oriented (e.g . MongoDB)  

Every category has its unique attributes and limitations. None of the above-specified DB is better to solve all the problems. Users should select the DB based on their product need.  

[Cassandra is a good choice](https://stackoverflow.com/questions/2634955/when-not-to-use-cassandra) if  
1. You don't require the ACID properties from your DB.  
2. There would be massive and huge number of writes on the DB.  
3. There is a requirement to integrate with Big Data, Hadoop, Hive and Spark.  
4. There is a need of real time data analytics and report generations.  
5. There is a requirement of impressive fault tolerant mechanism.  
6. There is a requirement of homogenous system.  
7. There is a requirement of lots of customization for tuning. 

**Step 1: Check installed java versions**  

`/usr/libexec/java_home -V`  

```  
Matching Java Virtual Machines (2):  
14.0.2, x86_64:		"Java SE 14.0.2"	/Library/Java/JavaVirtualMachines/jdk-14.0.2.jdk/Contents/Home  
1.8.0_261, x86_64:	"Java SE 8"		/Library/Java/JavaVirtualMachines/jdk1.8.0_261.jdk/Contents/Home  
```  

In my case, cassandra is not working with Java 14.0.2, so I changed default Java JVM to Java 1.8.  

Add JAVA_HOME to `~/.bash_profile`  

```  
export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.8.0_261.jdk/Contents/Home"  
```

Reload .bash_profile using `source ~/.bash_profile`  


**Step 2: Download [Cassandra](https://cassandra.apache.org/download/)**  

Add CASSANDRA_HOME to `~/.bash_profile`  

```
export CASSANDRA_HOME="/Users/adinasa/bigdata/apache-cassandra-3.11.7"  
export PATH="$PATH:$CASSANDRA_HOME/bin"
```  

Reload .bash_profile using `source ~/.bash_profile`  

**Step 3: Start Cassandra and cqlsh**  

To start Cassandra, run `cassandra -f`  

`cqlsh` is a command line shell for interacting with Cassandra through CQL (the CassandraQuery Language).  

You may need to reload your .bash_profile using `source ~/.bash_profile`  

Run the following command at terminal `cqlsh`  

```  
Connected to Test Cluster at 127.0.0.1:9042.  
[cqlsh 5.0.1 | Cassandra 3.11.7 | CQL spec 3.4.4 | Native protocol v4]  
Use HELP for help.  
cqlsh>  
```  

Complete list of CQL commands are available at [cqlCommandsTOC](https://docs.datastax.com/en/cql-oss/3.x/cql/cql_reference/cqlCommandsTOC.html)  

A **keyspace** is the top-level DB object that controls the replication for the object it contains at each datacenter in the cluster. Keyspaces contain tables, materialized views and user-defined types, functions and aggregates.  

```
cqlsh> DESCRIBE KEYSPACES;  
cqlsh> USE system_auth; DESC TABLES  
cqlsh> DESC TABLE system_auth.role_members  
```  

Create a KEYSPACE in a single node cluster environment    

```  
cqlsh> CREATE KEYSPACE IF NOT EXISTS cancer WITH replication = {'class' : 'SimpleStrategy', 'replication_factor':1};  

cqlsh> DESC KEYSPACES;  

cancer  system_schema  system_auth  system  system_distributed  system_traces  
```

Create a TABLE in a KEYSPACE and update with data.  

```  
cqlsh>DROP TABLE IF EXISTS cancer.meta_data;

cqlsh>CREATE TABLE cancer.meta_data (ID text, AGE int, details_ map<text,text>, PRIMARY KEY (ID,AGE));  

cqlsh>INSERT INTO cancer.meta_data (ID, AGE, details_) VALUES ('GHN-1', 40,{'HPV':'negative', 'bday':'02/07/1980', 'blist_nation':'USA'});  
cqlsh>INSERT INTO cancer.meta_data (ID, AGE, details_) VALUES ('GHN-2', 35,{'HPV':'positive', 'bday':'05/07/1985', 'blist_nation':'UK'});  
cqlsh>INSERT INTO cancer.meta_data (ID, AGE, details_) VALUES ('GHN-3', 30,{'HPV':'negative', 'bday':'23/07/1990', 'blist_nation':'AUS'});  
cqlsh>INSERT INTO cancer.meta_data (ID, AGE, details_) VALUES ('GHN-4', 44,{'HPV':'negative', 'bday':'19/07/1976', 'blist_nation':'USA'});  
cqlsh>INSERT INTO cancer.meta_data (ID, AGE, details_) VALUES ('GHN-5', 25,{'HPV':'negative', 'bday':'05/07/1995', 'blist_nation':'UK'});  
cqlsh>INSERT INTO cancer.meta_data (ID, AGE, details_) VALUES ('GHN-6', 40,{'HPV':'positive', 'bday':'12/07/1980', 'blist_nation':'USA'});  
```  

Query a TABLE.  
```  
cqlsh> SELECT * FROM cancer.meta_data;  

 id    | age | details_  
-------+-----+------------------------------------------------------------------  
 GHN-1 |  40 | {'HPV': 'negative', 'bday': '02/07/1980', 'blist_nation': 'USA'}  
 GHN-2 |  35 |  {'HPV': 'positive', 'bday': '05/07/1985', 'blist_nation': 'UK'}  
 GHN-6 |  40 | {'HPV': 'positive', 'bday': '12/07/1980', 'blist_nation': 'USA'}  
 GHN-5 |  25 |  {'HPV': 'negative', 'bday': '05/07/1995', 'blist_nation': 'UK'}  
 GHN-3 |  30 | {'HPV': 'negative', 'bday': '23/07/1990', 'blist_nation': 'AUS'}  
 GHN-4 |  44 | {'HPV': 'negative', 'bday': '19/07/1976', 'blist_nation': 'USA'}  

(6 rows)  

cqlsh> SELECT COUNT(*) FROM cancer.meta_data;  

 count  
-------  
    6  

(1 rows)  

Warnings :  
Aggregation query used without partition key  

cqlsh> SELECT * FROM cancer.meta_data WHERE id = 'GHN-1';  

 id    | age | details_  
-------+-----+------------------------------------------------------------------  
 GHN-1 |  40 | {'HPV': 'negative', 'bday': '02/07/1980', 'blist_nation': 'USA'}    

(1 rows)  

cqlsh> SELECT * FROM cancer.meta_data WHERE id IN ('GHN-1', 'GHN-2');  

 id    | age | details_  
-------+-----+------------------------------------------------------------------  
 GHN-1 |  40 | {'HPV': 'negative', 'bday': '02/07/1980', 'blist_nation': 'USA'}  
 GHN-2 |  35 |  {'HPV': 'positive', 'bday': '05/07/1985', 'blist_nation': 'UK'}  

(2 rows)   
```    

*The partition key columns support only two operators i.e = and IN.*  

For map collections Cassandra allows creation of an index on keys, values or entries.  

```  
cqlsh>CREATE INDEX meta_index ON cancer.meta_data (ENTRIES(details_));  

cqlsh> SELECT * FROM cancer.meta_data WHERE details_['blist_nation'] = 'UK';  

 id    | age | details_  
-------+-----+-----------------------------------------------------------------  
 GHN-2 |  35 | {'HPV': 'positive', 'bday': '05/07/1985', 'blist_nation': 'UK'}  
 GHN-5 |  25 | {'HPV': 'negative', 'bday': '05/07/1995', 'blist_nation': 'UK'}  

(2 rows)  
```  
*The OR operator is not supported by CQL. You can only use AND operator with primary key columns (partition key and clustering columns keys).*  

```  
cqlsh> SELECT * FROM cancer.meta_data WHERE id IN ('GHN-1','GHN-2','GHN-3') AND age >= 35;  

 id    | age | details_  
-------+-----+------------------------------------------------------------------  
 GHN-1 |  40 | {'HPV': 'negative', 'bday': '02/07/1980', 'blist_nation': 'USA'}  
 GHN-2 |  35 |  {'HPV': 'positive', 'bday': '05/07/1985', 'blist_nation': 'UK'}  

(2 rows)  
```  

**Step 4: Java application**  

Apart from the CQL shell, another way of connecting to Cassandra is *via* a programming language driver. Here I am using Datastax's [Java-Driver](https://github.com/datastax/java-driver). For a [list of available client drivers](https://cassandra.apache.org/doc/latest/getting_started/drivers.html).  

```
cqlsh> select release_version from system.local;

release_version
-----------------
        3.11.7

(1 rows)  
```  

Created a Maven project in Eclipse and update the pom.xml file for the following dependencies.  

```
<dependencies>  
 <dependency>  
  <groupId>com.datastax.oss</groupId>  
  <artifactId>java-driver-core</artifactId>  
  <version>4.8.0</version>  
 </dependency>  

 <dependency>  
  <groupId>com.datastax.cassandra</groupId>  
  <artifactId>cassandra-driver-core</artifactId>  
  <version>3.10.0</version>  
 </dependency>  
 
 <dependency>  
  <groupId>com.datastax.cassandra</groupId>  
  <artifactId>cassandra-driver-mapping</artifactId>  
  <version>3.10.0</version>  
 </dependency>  
 
 <dependency>  
  <groupId>com.datastax.oss</groupId>  
  <artifactId>java-driver-query-builder</artifactId>  
  <version>4.8.0</version>  
 </dependency>  

 <dependency>  
  <groupId>com.datastax.cassandra</groupId>  
  <artifactId>cassandra-driver-extras</artifactId>  
  <version>3.10.0</version>  
 </dependency>  
</dependencies>  
```  

Refer to each module's manual for more details ([core](https://github.com/datastax/java-driver/blob/4.x/manual/core), [query builder](https://github.com/datastax/java-driver/blob/4.x/manual/query_builder), [mapper](https://github.com/datastax/java-driver/blob/4.x/manual/mapper)).  


The **core** module handles cluster connectivity and request execution. Here's a short program that connects to Cassandra and executes a query:  

```  
import com.datastax.oss.driver.api.core.CqlSession;  
import com.datastax.oss.driver.api.core.cql.ResultSet;  
import com.datastax.oss.driver.api.core.cql.Row;  

public class CassandraCore {  
	public static void main(String[] args) {  
		# If you don't specify any contact point, the driver defaults to 127.0.0.1:9042  
		CqlSession session = CqlSession.builder().build();  
		ResultSet rs = session.execute("select release_version from system.local");  
		Row row = rs.one();  
		System.out.println(row.getString("release_version"));  
	}  
} 
```  

The **query builder** is a utility to generate CQL queries programmatically. Here's a short program that connects to Cassandra and, creates and executes a query:    

```  
import static com.datastax.oss.driver.api.querybuilder.QueryBuilder.*;  

import com.datastax.oss.driver.api.core.CqlSession;  
import com.datastax.oss.driver.api.core.cql.ResultSet;  
import com.datastax.oss.driver.api.core.cql.Row;  
import com.datastax.oss.driver.api.core.cql.SimpleStatement;  
import com.datastax.oss.driver.api.querybuilder.select.Select;  

public class CassandraQueryBuilder {  
	public static void main(String[] args) {  
		CqlSession session = CqlSession.builder().build();  
		// SELECT release_version FROM system.local  
		Select query = selectFrom("system", "local").column("release_version");   
		SimpleStatement statement = query.build();  
		ResultSet rs = session.execute(statement);  
		Row row = rs.one();  
		System.out.println(row.getString("release_version"));  
	}  
}  
```  

How to run Spring Boot web application in Eclipse?  
In eclipse Project Explorer, right click the project name -> select “Run As” -> “Java Application”
