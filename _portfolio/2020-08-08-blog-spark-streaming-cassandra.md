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
*Updated on August 16, 2020*  

[Apache Cassandra](https://cassandra.apache.org) is a distributed NoSQL database (DB) which is used for handling Big data and real-time web applications. NoSQL stands for "Not Only SQL" or "Not SQL". NoSQL database is a non-relational data management system, that does not require a fixed schema.  
 
Why NoSQL?  
The system response time becomes slow when you use RDBMS for massive volumes of data. The alternative for this issue is to distribute DB load on multiple hosts whenever the load increase.  
 
NoSQL databases are mainly categorized into four types:  
1. key-value pair (e.g Redis, Dynamo)  
2. column-oriented (e.g HBase, Cassandra)  
3. graph-based (e.g Neo4J)  
4. document-oriented (e.g MongoDB)  

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
cqlsh>INSERT INTO cancer.meta_data (ID, AGE, details_) VALUES ('GHN-4', 44,{'HPV':'positive', 'bday':'19/07/1976', 'blist_nation':'UK'});  
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
 GHN-4 |  44 | {'HPV': 'positive', 'bday': '19/07/1976', 'blist_nation': 'UK'}  

(6 rows)  

Use the UPDATE command to insert values into the map.  
cqlsh> UPDATE cancer.meta_data SET details_ = details_ + {'HPV': 'negative', 'bday': '19/07/1976', 'blist_nation':'USA'} WHERE id = 'GHN-4' AND age=44;  
Set a specific element of map using the UPDATE command.  
UPDATE cancer.meta_data SET details_['HPV'] = 'negative' WHERE id = 'GHN-4' AND age=44;  

cqlsh> SELECT COUNT(*) FROM cancer.meta_data;  

 count  
-------  
    6  

(1 rows)  

Warnings :  
Aggregation query used without partition key  
```  
[Partition Key vs Composite Key vs Clustering Columns in Cassandra](https://www.bmc.com/blogs/cassandra-clustering-columns-partition-composite-key/)  

The partition key columns support only two operators: '=' and 'IN'.  
```  
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

Write a simple Java program for the following cql query.  
```
cqlsh> SELECT * FROM cancer.meta_data WHERE id = 'GHN-1';  

 id    | age | details_  
-------+-----+------------------------------------------------------------------  
 GHN-1 |  40 | {'HPV': 'negative', 'bday': '02/07/1980', 'blist_nation': 'USA'}  
```  

Created a **Maven project** in Eclipse and update the **pom.xml** file for the following dependencies.  

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
  <version>4.0.0</version>  
 </dependency>  

 <dependency>  
  <groupId>com.datastax.oss</groupId>  
  <artifactId>java-driver-query-builder</artifactId>  
  <version>4.8.0</version>  
 </dependency>   
</dependencies>  
```  
Refer to each module's manual for more details ([core](https://github.com/datastax/java-driver/blob/4.x/manual/core), [query builder](https://github.com/datastax/java-driver/blob/4.x/manual/query_builder), [mapper](https://github.com/datastax/java-driver/blob/4.x/manual/mapper)).  

**Java example code 1**  

The **core** module handles cluster connectivity and request execution. Here's a short program that connects to Cassandra and executes a query:  

```  
import com.datastax.oss.driver.api.core.CqlSession;  
import com.datastax.oss.driver.api.core.cql.ResultSet;  
import com.datastax.oss.driver.api.core.cql.Row;  

public class CassandraCore {  
	public static void main(String[] args) {  
		// If you don't specify any contact point, the driver defaults to 127.0.0.1:9042  

		CqlSession session = CqlSession.builder().build();  
		ResultSet rs = session.execute("SELECT * FROM cancer.meta_data WHERE id = 'GHN-1'");  
		Row row = rs.one();  

		System.out.println("ID: " + row.getString("id"));  
		System.out.println("Age: " + row.getInt("age"));  

		Map<String, String> m = row.getMap("details_",String.class,String.class);  
		System.out.println("HPV: "+ m.get("HPV"));  
		System.out.println("bday: "+ m.get("bday"));  
		System.out.println("blist_nation: "+ m.get("blist_nation"));  
	}  
} 
```  

Output of the above program is  
```  
ID: GHN-1  
Age: 40  
HPV: negative  
bday: 02/07/1980  
blist_nation: USA
```  

**Java example code 2**  

The **query builder** is a utility to generate CQL queries programmatically. Here's a short program that connects to Cassandra and, creates and executes a query:    

```  
import static com.datastax.oss.driver.api.querybuilder.QueryBuilder.*;  

import java.util.Map;  

import com.datastax.oss.driver.api.core.CqlSession;  
import com.datastax.oss.driver.api.core.cql.PreparedStatement;  
import com.datastax.oss.driver.api.core.cql.ResultSet;  
import com.datastax.oss.driver.api.core.cql.Row;  
import com.datastax.oss.driver.api.querybuilder.select.Select;  

public class CassandraQueryBuilder {  
	public static void main(String[] args) {  
		CqlSession session = CqlSession.builder().build();    
		// SELECT * == all()     
		Select query = selectFrom("cancer", "meta_data").all().whereColumn("id").isEqualTo(bindMarker());  
		PreparedStatement preparedQuery = session.prepare(query.build());  
		ResultSet rs = session.execute(preparedQuery.bind("GHN-2"));  
		Row row = rs.one();  
		System.out.println("ID: " + row.getString("id"));   
		System.out.println("Age: " + row.getInt("age"));   
		Map<String, String> m = row.getMap("details_",String.class,String.class);  
		System.out.println("HPV: "+ m.get("HPV"));  
		System.out.println("bday: "+ m.get("bday"));  
		System.out.println("blist_nation: "+ m.get("blist_nation"));	  
	}  
}  
```  

Output of the above program is  
```  
ID: GHN-2  
Age: 35  
HPV: positive  
bday: 05/07/1985  
blist_nation: UK  
```
**Java example code 3**  

The **[mapper](https://docs.datastax.com/en/developer/java-driver/4.3/manual/mapper/)** generates the boilerplate to execute queries and convert the results into application-level objects. For a quick overview of mapper features, we are going to build a trivial example based on the schema `cancer.meta_data`:  

First, update the **pom.xml** file with the following dependency.  

```  
<dependency>  
  <groupId>com.datastax.oss</groupId>  
  <artifactId>java-driver-mapper-processor</artifactId>  
  <version>4.8.0</version>  
</dependency>  
```  

Create the following Java classes/interfaces.  

**Entity class**: MetaData.java    
This is a simple data container that will represent a row in the *meta_data* table. We use mapper annotations to mark the class as an entity, and indicate which field(s) correspond to the primary key. Entity classes must have a no-arg constructor; note that, because we also have a constructor that takes all the fields, we have to define the no-arg constructor explicitly. We use mapper annotations to mark the class as an entity, and indicate which field(s) correspond to the primary key.  

More annotations are available; for more details, see [Entities](https://docs.datastax.com/en/developer/java-driver/4.3/manual/mapper/entities/).  

```  
import java.util.Map;    
import com.datastax.oss.driver.api.mapper.annotations.CqlName;  
import com.datastax.oss.driver.api.mapper.annotations.Entity;  
import com.datastax.oss.driver.api.mapper.annotations.PartitionKey;  

@Entity  
public class MetaData {  
	
	@PartitionKey  
	@CqlName("id")  
	private String id;  
	
	@CqlName("age")  
	private Integer age;  

	@CqlName("details_")  
	private Map<String, String> details;  
	
	public MetaData() {}  
	
	public MetaData(String id, Integer age, Map<String, String> details) {  
		  
		this.id = id;  
		this.age = age;  
		this.details = details;  
	}  

	public String getId() {  
		return id;  
	}  

	public void setId(String id) {  
		this.id = id;  
	}
	
	public Integer getAge() {  
		return age;  
	}

	public void setAge(Integer age) {  
		this.age = age;  
	}

	public Map<String, String> getDetails() {  
		return details;  
	}

	public void setDetails(Map<String, String> details) {  
		this.details = details;  
	}  
}
```  

**DAO interface**:MetaDataDAO  
A DAO defines a set of query methods. Again, mapper annotations are used to mark the interface, and indicate what kind of request each method should execute.  

For the full list of available query types, see [DAOs](https://docs.datastax.com/en/developer/java-driver/4.3/manual/mapper/daos/).  

```  
import com.datastax.oss.driver.api.mapper.annotations.Dao;  
import com.datastax.oss.driver.api.mapper.annotations.Delete;  
import com.datastax.oss.driver.api.mapper.annotations.Insert;  
import com.datastax.oss.driver.api.mapper.annotations.Select;

@Dao   
public interface MetaDataDAO {  
	
	@Select  
	MetaData findById(String personId);  
	
	@Insert  
	void save(MetaData metaData);  
	
	@Delete  
	void delete(MetaData metaData);  
}
```  

**Mapper interface**: PersonMapper.java  
This is the top-level entry point to mapper features, that allows you to obtain DAO instances.  

For more details, see [Mapper](https://docs.datastax.com/en/developer/java-driver/4.3/manual/mapper/mapper/).  

```  
import com.datastax.oss.driver.api.core.CqlIdentifier;  
import com.datastax.oss.driver.api.mapper.annotations.DaoFactory;  
import com.datastax.oss.driver.api.mapper.annotations.DaoKeyspace;  
import com.datastax.oss.driver.api.mapper.annotations.Mapper;  

@Mapper  
public interface PersonMapper {  
	
	@DaoFactory  
	MetaDataDAO personDao(@DaoKeyspace CqlIdentifier keyspace);   
}
```  

**Generating an additional code using annotation processing**:  
Annotation processing is a common technique in modern frameworks, and is generally well supported by build tools and IDEs; See [Configuring the annotation processor](https://docs.datastax.com/en/developer/java-driver/4.3/manual/mapper/config/).  

The mapper’s annotation processor hooks into the Java compiler, and generates additional source files from your annotated classes before the main compilation happens. It is contained in the *java-driver-mapper-processor* artifact.  

Updata the **pom.xml** file.  

The processor runs every time you execute the *mvn compile* phase.  

```  
<dependency>  
  <groupId>com.datastax.oss</groupId>  
  <artifactId>java-driver-mapper-processor</artifactId>  
  <version>4.8.0</version>  
</dependency>  

<build>  
  <plugins>  
     <plugin>  
      <artifactId>maven-compiler-plugin</artifactId>  
      <version>3.8.1</version>  
      <configuration>  
        <source>1.8</source> <!-- (or higher) -->  
        <target>1.8</target> <!-- (or higher) -->  
        <annotationProcessorPaths>  
         <path>  
            <groupId>com.datastax.oss</groupId>  
            <artifactId>java-driver-mapper-processor</artifactId>   
            <version>4.8.0</version>  
         </path>  
       </annotationProcessorPaths>  
     </configuration>  
   </plugin>  
  </plugins>  
</build>  
```  

With the above configuration, these files are in **target/generated-sources/annotations** directory of Eclipse. Make sure that directory is marked as a source folder in your IDE 
(for example, in Eclipse IDE, this might require right-clicking on your pom.xml and selecting “Maven > Update Project”).  

![eclipse-annotations](/images/eclipse-annotations.png)

One of the classes generated during annotation processing is *PersonMapperBuilder.java*. It allows you to initialize a mapper instance by wrapping a core driver session:  


**Java main** class: MetaDataMain.java  
The main() method is the entry point into the application.  

```  
import java.util.HashMap;  
import java.util.Map;  
import com.datastax.oss.driver.api.core.CqlIdentifier;  
import com.datastax.oss.driver.api.core.CqlSession;  

public class MetaDataMain {  

	public static void main(String[] args) {  
	
		CqlSession session = CqlSession.builder().build();  
		PersonMapper personMapper = new PersonMapperBuilder(session).build();  	
		MetaDataDAO dao = personMapper.personDao(CqlIdentifier.fromCql("cancer"));  

		// retrieve data from DB  
		MetaData md = dao.findById("GHN-1");  
		System.out.println(md.getAge().toString());  
		Map<String, String> details = md.getDetails();  
		System.out.println(details.get("HPV"));  
		System.out.println(details.get("bday"));  
		System.out.println(details.get("blist_nation"));  
		
		// update DB with new data  
		Map<String, String> map = new HashMap<>();  
		map.put("HPV", "negative");  
		map.put("bday", "01/02/2000");  
		map.put("blist_nation", "Italy");  
		dao.save(new MetaData("GHN-8", 20,map));  	
	}  
}  
```  

Makesure to check the newly added row in DB i.e "GHN-8 ..."  
 
```  
cqlsh> SELECT * FROM cancer.meta_data;  
  
 id    | age | details_  
-------+-----+--------------------------------------------------------------------  
 GHN-1 |  40 |   {'HPV': 'negative', 'bday': '02/07/1980', 'blist_nation': 'USA'}  
 GHN-2 |  35 |    {'HPV': 'positive', 'bday': '05/07/1985', 'blist_nation': 'UK'}  
 GHN-6 |  40 |   {'HPV': 'positive', 'bday': '12/07/1980', 'blist_nation': 'USA'}  
 GHN-5 |  25 |    {'HPV': 'negative', 'bday': '05/07/1995', 'blist_nation': 'UK'}  
 GHN-3 |  30 |   {'HPV': 'negative', 'bday': '23/07/1990', 'blist_nation': 'AUS'}  
 GHN-8 |  20 | {'HPV': 'negative', 'bday': '01/02/2000', 'blist_nation': 'Italy'}  
 GHN-4 |  44 |   {'HPV': 'negative', 'bday': '19/07/1976', 'blist_nation': 'USA'}  

(7 rows)  
```   


How to run Java Main class in Eclipse?  
In eclipse Project Explorer, right click the Main class -> select “Run As” -> “Java Application”  

Further Reading:  
[How to Setup a Cassandra Cluster](https://www.bmc.com/blogs/setup-cassandra-cluster/)  

