---
title: 'Building a real-time big data pipeline (part 2: Spark, Hadoop)'
date: 2020-02-09
permalink: /posts/2020/02/blog-post-spark/
tags:
  - big data
  - apache spark
  - real time data pipelines 
  - scala
  - hadoop   

---  
Apache Spark[^1] is a unified analytics engine for large-scale data processing.

**What exactly is Apache Spark?**  

A Cluster computing engine and a set of libraries, application programming interfaces (APIs) together make apache spark. The spark core itself has two parts. 1. Computing engine and 2. Spark Core APIs (Scala, Java, Python and R).  

**Apache Spark Ecosystem**  
```
+--------+-----------+-------+----------+
| SQL	 | Streaming | MLlib |	GraphX 	|
|---------------------------------------|
|	Spark Core API			|	
|---------------------------------------|
| Scala	| Python    |	Java |	R	|
|---------------------------------------|	
|	Compute Engine			|
+---------------------------------------+
```  
**Computing Engine**  
Apache Hadoop[^2] offers distributed storage (HDFS), resource manager (YARN)  and computing framework (Map Reduce). Apache Spark is a distributed processing engine but it doesn’t come with inbuilt cluster resource manager and distributed storage system. We have to plugin a cluster manager and a storage system of our choice. We can use YARN, Mesos, and Kubernetes as a cluster manager for Apache Spark. Similarly, for the storage system we can use HDFS, Amazon S3, Google cloud storage or Cassandra File System (CFS) and more. The compute engine provides some basic functionality like memory management, task scheduling, fault recovery and most importantly interacting with the cluster manager and storage system.  

**Core APIs**
Spark core consists of structured API and unstructured API. Structured API consists of Data Frames and Data Sets. Unstructured APIs consists of RDDs, accumulators and broadcast variables[^3]. These core APIs are available as Scala, Java, Python and R.    
Outside of Spark Core we have 4 sets of libraries and packages, Spark SQL, Spark Streaming, MLlib and Graphx. They directly depend on Spark Core APIs to achieve distributed processing.  

Typical Spark Application Process Flow: Apache Spark reads some data from source and load it into a Spark. There are 3 alternatives to hold data in Spark. 1) Data Frame 2) Data Set and 3) RDD. Latest Spark release recommended to use Data Frame and Data Set. Both of them are compiled down in RDD.  
 
**RDD: Resilient Distributed Data Set** [^3] Spark RDD is a resilient, partitioned, distributed and immutable collection of data.  
**Resilient** – RDDs are fault tolerant.  
**Partitioned** – Spark breaks the RDD into smaller chunks of data. These pieces are called partitions.  
**Distributed** – Instead of keeping these partitions on a single machine, Spark spreads them across the cluster.  
**Immutable** – Once defined, you can’t change them. So Spark RDD is a read-only data structure.  

For RDDs vs DataFrames and Datasets - When to use them and why. Read [^4].  

We can create RDD using two methods. 1.Load some data from a source or 2. Create an RDD by transforming another RDD  

**Step 1**: Hadoop installation  

See tutorial on [How to Install and Configure Hadoop on Mac](https://www.quickprogrammingtips.com/big-data/how-to-install-hadoop-on-mac-os-x-el-capitan.html)  

Update your `~/.bash_profile` file, which is a configuration file for configuring user environments.  

```
$vi ~/.bash_profile  
export HADOOP_HOME=/Users/adinasarapu/Documents/hadoop-3.1.3  
export PATH=$PATH:$HADOOP_HOME/bin  
```  

**Start Hadoop**  

```
$bash sbin/start-dfs.sh 
Starting namenodes on [localhost]
Starting datanodes
Starting secondary namenodes [Ashoks-MacBook-Pro.local]
```

**Verify Hadoop installation**  
```
$ jps
61073 ResourceManager
82025 SecondaryNameNode
61177 NodeManager
81882 DataNode
82303 Jps
81774 NameNode
```

**Create user**  

`$hadoop fs -mkdir -p /user/adinasarapu`  

**Move file to HDFS**  

`$hadoop fs -copyFromLocal samples.csv /user/adinasarapu`  

Now the data file is at real distributed storage. The file location at HDFS is `hdfs://localhost:9000/user/adinasarapu/ samples.csv`  

**List files moved**  
`$ hadoop fs -ls /user/adinasarapu`    
```
Found 3 items
-rw-r--r--   1 adinasarapu supergroup  110252495 2020-02-08 17:04 /user/adinasarapu/flist.txt
-rw-r--r--   1 adinasarapu supergroup       1318 2020-02-09 14:47 /user/adinasarapu/samples.csv
-rw-r--r--   1 adinasarapu supergroup     303684 2020-02-09 08:21 /user/adinasarapu/survey.csv
```

**Apache Spark installation**  

See tutorial on [Installing Apache Spark ... on macOS](https://medium.com/luckspark/installing-spark-2-3-0-on-macos-high-sierra-276a127b8b85)  
Update your `~/.bash_profile` file, which is a configuration file for configuring user environments.  

```
$vi ~/.bash_profile  
export SPARK_HOME=/Users/adinasarapu/Documents/spark-3.0.0-preview2-bin-hadoop3.2  
export PATH=$PATH:$SPARK_HOME/bin  
```

**Start Spark shell**  
```  
$spark-shell  
20/02/09 15:03:23 ..  
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties  
Setting default log level to "WARN".  
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).  
Spark context Web UI available at http://192.168.0.5:4040  
Spark context available as 'sc' (master = local[*], app id = local-1581278612106).  
Spark session available as 'spark'.  
Welcome to  
      ____              __  
     / __/__  ___ _____/ /__  
    _\ \/ _ \/ _ `/ __/  '_/  
   /___/ .__/\_,_/_/ /_/\_\   version 3.0.0-preview2  
      /_/  
           
Using Scala version 2.12.10 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_102)  
Type in expressions to have them evaluated.  
Type :help for more information.  
```  

**Read CSV data**  
``` 
scala> val df = spark.read.options(Map(  
	"header" -> "true",  
	"inferSchema"->"true",  
	"nullValue"->"NA",  
	"timestampFormat"->"MM-dd-yyyy",  
	"mode"->"failfast")).csv("hdfs://localhost:9000/user/adinasarapu/samples.csv")  

df: org.apache.spark.sql.DataFrame = [Sample: string, p16: string ... 7 more fields]
```  

**Check the number of partitions**  
```
scala> df.rdd.getNumPartitions  
res4: Int = 1  
```

**Set/increase Number of partitions to 3**  
```  
scala> val df2 = df.repartition(3).toDF  
df2: org.apache.spark.sql.DataFrame = [Sample: string, p16: string ... 7 more fields]  
```
**Recheck the number of partitions**  
```
scala> df2.rdd.getNumPartitions  
res5: Int = 3  
```  

**SQL like operation**   
Data Frame follows row and column structure like a database table. Data Frame compiles down to RDDs. RDDs are immutable and once loaded you can’t modify it. However, you can perform the following operations a) Transformation and b) Actions. Spark Data Frames carries the same legacy from RDDs. Like RDDs, Spark Data Frames are immutable. You can perform transformation and actions on Data Frames.  

```
scala> df.select("Sample","Age","Sex","Anatomy").filter("Age < 55").show  
  
+------+---+------+-------+  
|Sample|Age|   Sex|Anatomy|  
+------+---+------+-------+  
|GHN-57| 50|female|    BOT|  
|GHN-39| 51|  male|    BOT|  
|GHN-60| 41|  male|    BOT|  
|GHN-64| 49|  male|    BOT|  
|GHN-77| 53|  male|    BOT|  
|GHN-66| 52|  male| Tonsil|  
|GHN-67| 54|  male| Tonsil|  
|GHN-68| 51|  male| Tonsil|  
|GHN-80| 54|  male| Tonsil|  
|GHN-83| 54|  male| Tonsil|  
+------+---+------+-------+
```  
```  
scala> val df1 = df.select($"Sex",$"Radiation")  
df1: org.apache.spark.sql.DataFrame = [Sex: string, Radiation: string]  

scala> df1.show  
+------+---------+  
|   Sex|Radiation|  
+------+---------+  
|female|        Y|  
|female|        Y|  
|  male|        Y|  
|  male|        N|  
|  male|        Y|  
|  male|        Y|  
|  male|        Y|  
|  male|        Y|  
|  male|        Y|  
|  male|        Y|  
|  male|        N|  
|  male|        N|  
|  male|  Unknown|  
|  male|        Y|  
|female|        Y|  
|  male|        Y|  
|  male|        Y|  
|  male|        Y|  
|  male|        Y|  
|  male|        N|  
+------+---------+  
only showing top 20 rows  
```  

```  
scala> val df2 = df1.select($"Sex",   
		(when($"Radiation" === "Y",1).otherwise(0)).alias("All-Yes"),  
		(when($"Radiation" === "N",1).otherwise(0)).alias("All-Nos"),  
		(when($"Radiation" === "Unknown",1).otherwise(0)).alias("All-Unknown"))  
df2: org.apache.spark.sql.DataFrame = [Sex: string, All-Yes: int ... 2 more fields]  

scala> df2.show  
+------+-------+-------+-----------+  
|   Sex|All-Yes|All-Nos|All-Unknown|  
+------+-------+-------+-----------+  
|female|      1|      0|          0|  
|female|      1|      0|          0|  
|  male|      1|      0|          0|  
|  male|      0|      1|          0|  
|  male|      1|      0|          0|  
|  male|      1|      0|          0|  
|  male|      1|      0|          0|  
|  male|      1|      0|          0|  
|  male|      1|      0|          0|  
|  male|      1|      0|          0|  
|  male|      0|      1|          0|  
|  male|      0|      1|          0|  
|  male|      0|      0|          1|  
|  male|      1|      0|          0|  
|female|      1|      0|          0|  
|  male|      1|      0|          0|  
|  male|      1|      0|          0|  
|  male|      1|      0|          0|  
|  male|      1|      0|          0|  
|  male|      0|      1|          0|  
+------+-------+-------+-----------+  
only showing top 20 rows  
```  

**Scala user-defined function (UDF)**  

```  
def parseSex(g: String) = {  
 	g.toLowerCase match {   
			case "male"  => “Male”  
			case "female" => “Female”   
			case _ => “Other”  
	}  
}   

scala> val parseSexUDF = udf(parseSex _)  

parseGenderUDF: org.apache.spark.sql.expressions.UserDefinedFunction = SparkUserDefinedFunction($Lambda$3628/1633612848@4ee80d4c,StringType,List(Some(Schema(StringType,true))),None,true,true)  
```  
```  
scala> val df3 = df2.select((parseSexUDF($"Sex")).alias("Sex"),$"All-Yes",$"All-Nos",$"All-Unknown")  
df3: org.apache.spark.sql.DataFrame = [Sex: string, All-Yes: int ... 2 more fields]  
```

```  
scala> val df4 = df3.groupBy("Sex").agg(sum($"All-Yes"), sum($"All-Nos"), sum($"All-Unknown"))  
df4: org.apache.spark.sql.DataFrame = [Sex: string, sum(All-Yes): bigint ... 2 more fields]  

scala> df4.show  
+------+------------+------------+----------------+  
|   Sex|sum(All-Yes)|sum(All-Nos)|sum(All-Unknown)|  
+------+------------+------------+----------------+  
|Female|           3|           0|               0|  
|  Male|          18|           5|               1|  
+------+------------+------------+----------------+  
```

```  
scala> val df5 = df4.filter($"Sex" =!= "Unknown")  
df5: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [Sex: string, sum(All-Yes): bigint ... 2 more fields]  
  
scala> df5.collect()  
res22: Array[org.apache.spark.sql.Row] = Array([Male,18,5,1])  

scala> df5.show  
+----+------------+------------+----------------+  
| Sex|sum(All-Yes)|sum(All-Nos)|sum(All-Unknown)|  
+----+------------+------------+----------------+  
|Male|          18|           5|               1|  
+----+------------+------------+----------------+  
```

[^1]: [Apache Spark](https://spark.apache.org)
[^2]: [Apache Hadoop](https://hadoop.apache.org)
[^3]: [Learning Journal](https://www.learningjournal.guru/courses/spark/spark-foundation-training/)
[^4]: [RDDs vs Data Frames and Data Sets](https://databricks.com/blog/2016/07/14/a-tale-of-three-apache-spark-apis-rdds-dataframes-and-datasets.html) A Tale of Three Apache Spark APIs: RDDs vs DataFrames and Datasets - When to use them and why
## References
