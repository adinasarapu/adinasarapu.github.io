---
title: 'Building a real-time big data pipeline (part 5: Cassandra, NoSQL Database)'
date: 2020-08-08
permalink: /posts/2020/08/blog-post-spark-streaming-cassandra/
tags:
  - big data
  - apache cassandra
  - real time data pipelines 
  - Java
  - NoSQL
  - Bioinformatics
  - Emory University 

---  
*Updated on August 08, 2020*  

[Apache Cassandra](https://cassandra.apache.org) is a distributed NoSQL database which is used for handling Big data and real-time web applications. NoSQL stands for "Not Only SQL" or "Not SQL". NoSQL database is a non-relational data management system, that does not require a fixed schema.  
 
Why NoSQL?  
The system response time becomes slow when you use RDBMS for massive volumes of data. The alternative for this issue is to distribute database load on multiple hosts whenever the load increase.  
 
NoSQL databases are mainly categorized into four types:  
1. key-value pair (e.g Redis, Dynamo)  
2. column-oriented (e.g HBase, Cassandra)  
3. graph-based (e.g . Neo4J)  
4. document-oriented (e.g . MongoDB)  

Every category has its unique attributes and limitations. None of the above-specified database is better to solve all the problems. Users should select the database based on their product need.  

How to connect Cassandra to localhost using cqlsh?  

**Step 1: Check installed java versions**  

`/usr/libexec/java_home -V`  

```  
Matching Java Virtual Machines (2):  
14.0.2, x86_64:	"Java SE 14.0.2"	/Library/Java/JavaVirtualMachines/jdk-14.0.2.jdk/Contents/Home  
1.8.0_261, x86_64:	"Java SE 8"	/Library/Java/JavaVirtualMachines/jdk1.8.0_261.jdk/Contents/Home  
```  

In my case, cassandra is not working with Java 14.0.2, so I changed default Java JVM to Java 1.8.  

Add JAVA_HOME to ~/.bash_profile.  

```  
export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.8.0_261.jdk/Contents/Home"  
```

Reload .bash_profile `source ~/.bash_profile`  


**Step 2: Download [Cassandra](https://cassandra.apache.org/download/)**  

`Add CASSANDRA_HOME to ~/.bash_profile`  

```
export CASSANDRA_HOME="/Users/adinasa/bigdata/apache-cassandra-3.11.7"  
export PATH="$PATH:$CASSANDRA_HOME/bin"
```  

Reload .bash_profile `source ~/.bash_profile`  

**Step 3: Start Cassandra and cqlsh**  

To start Cassandra run the following command at terminal `cassandra -f`  

`cqlsh` is a command line shell for interacting with Cassandra through CQL (the CassandraQuery Language).  

You may need to reload your `~/.bash_profile`

```  
source ~/.bash_profile  
```  

Run the following command at terminal `cqlsh`  

```  
Connected to Test Cluster at 127.0.0.1:9042.  
[cqlsh 5.0.1 | Cassandra 3.11.7 | CQL spec 3.4.4 | Native protocol v4]  
Use HELP for help.  
cqlsh>  
```  

Complete list of CQL commands are available at [cqlCommandsTOC](https://docs.datastax.com/en/cql-oss/3.x/cql/cql_reference/cqlCommandsTOC.html)  

```
cqlsh> DESCRIBE KEYSPACES;  
cqlsh> USE system_auth; DESC TABLES  
cqlsh> DESCRIBE TABLE system_auth.role_members  
```  

**Step 4: Java application**  

Apart from the CQL shell, another way of connecting to Cassandra is via a programming language driver. Created a Maven project in Eclipse and update the pom.xml file with the following dependencies.  

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
  <groupId>com.datastax.cassandra</groupId>  
  <artifactId>cassandra-driver-extras</artifactId>  
  <version>3.10.0</version>  
 </dependency>  
</dependencies>  
```  

Create a class called `Cassandra.java`  

```  
import org.apache.log4j.BasicConfigurator;  
import com.datastax.driver.core.Cluster;  
import com.datastax.driver.core.Session;  

public class Cassandra {  
	public static void main(String[] args) {  
		BasicConfigurator.configure();
		Cluster cluster = Cluster.builder().withoutJMXReporting()
				.addContactPoint("127.0.0.1")
				.withPort(9042)
				.build();
		Session session = cluster.newSession();		
		session.execute("SELECT * from system_auth.role_members");  
	}  
}  
```  
How to run Spring Boot web application in Eclipse?  
In eclipse Project Explorer, right click the project name -> select “Run As” -> “Java Application”
