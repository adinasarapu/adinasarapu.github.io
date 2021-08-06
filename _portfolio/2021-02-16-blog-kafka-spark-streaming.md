---
title: 'Building a real-time big data pipeline (11: Spark SQL Streaming, Kafka, Python)'
date: 2021-02-16
permalink: /posts/2021/02/blog-post-kafka-spark-streaming/
header:
  teaser: ""
tags:
  - big data
  - apache kafka
  - kafka
  - real time data pipelines 
  - Python
  - docker
  - spark sql
  - spark streaming 
  - YAML
  - zookeeper
  - bioinformatics
  - emory University 

---  
*Updated on August 06, 2021*  

*Apache Spark* is a general-purpose, in-memory cluster computing engine for large scale data processing. Spark can also work with Hadoop and it's modules. Spark uses Hadoop’s client libraries for distributed storage (HDFS) and resource management (YARN). The real-time data processing capability makes Spark a top choice for big data analytics. Spark provides APIs in Java, Scala, Python and R. It also supports libraries such as *Spark SQL* for structured data processing, *MLlib* for machine learning, *GraphX* for computing graphs, and *Spark Streaming* for stream computing.  

*Apache Spark Streaming*, a separate library from the core Apache Spark platform, enables scalable, high-throughput, fault-tolerant processing of data streams; written in Scala but offers Scala, Java, R and Python APIs to work with. It takes data from the sources like Kafka, Flume, Kinesis, HDFS, S3 or Twitter. Spark Streaming utilizes the discretized streams or DStream to divide the data into chunks before processing it. A DStream is a sequence of Resilient Data Sets or RDDs.  

*Spark Structured Streaming* is a scalable and fault-tolerant stream processing engine built on the *Spark SQL engine* library. This streaming model is based on the Dataset and DataFrame APIs, consumable in Java, Scala, Python, and R.  

### Apache Spark Streaming vs Spark Structured Streaming   
Apache Spark Streaming uses DStreams, while Spark Structured Streaming uses DataFrames to process the streams of data pouring into the analytics engine. Since Spark’s Structured Streaming model is an extension built on top of the Apache Spark’s DStreams construct, no need to access the RDD blocks directly.  

Python or R data frames exist on one machine rather than multiple machines. If you want to do distributed computation, then you’ll need to perform operations on Spark data frames. Spark’s data frame object can be thought of as a table distributed across a cluster and has functionality that is similar to a TABLE in relational database or, a data frame in R/Python. This has been achieved by taking advantage of the SparkR (R on Spark) or PySpark (Python on Spark) APIs.  

### Streaming from Kafka  

The Kafka cluster stores streams of records in categories called *topics*. See my other blog for installation and starting a kafka service [Kafka and Zookeeper with Docker](https://adinasarapu.github.io/posts/2020/01/blog-post-kafka/).  

Once we start zookeeper and kafka locally, we can proceed to create our first topic, named *mytopic*. The producer clients can then publish streams of data (messages) to the said topic (mytopic) and consumers can read the said datastream, if they are subscribed to that particular topic.  

Run the following commands in the directory where docker-compose.yml file is present.  

```
docker-compose up -d
docker-compose exec kafka bash
```  

```
cd /opt/kafka/bin/

bash-4.4# ./kafka-topics.sh \  
   --create \  
   --topic mytopic \  
   --partitions 1 \  
   --replication-factor 1 \  
   --bootstrap-server localhost:9092  
```

### Python Application:  

See [Configuring Eclipse IDE for Python and Spark](https://enahwe.wordpress.com)  

*Create a Eclipse Python project*  

Create a src folder:  
To add a source folder in order to create your Python source, right-click on the project icon and do: New > Folder   
Name the new folder “src”, then click on the button [Finish].  

Create a conf folder:  
To add a source folder in order to create your Python source, right-click on the project icone and do: New > Folder  
Name the new folder “conf”, then click on the button [Finish].  

Create the new project:  
Check that you are on the PyDev perspective.  
Go to the Eclipse menu File > New > PyDev project  
Name your new project “PySparkProject”, then click on the button [Finish].  

Create your source code:  
To add your new Python source, right-click on the source folder icon and do: New > PyDev Module.  
Name the new Python source “PySparkStreaming”, then click on the button [Finish], then click on the button [OK].  

Execute the code:  
To execute the following code, right-click on the Python module “PySparkStreaming.py”, then choose Run As > 1 Python Run.  

```
from pyspark.sql import SparkSession
from pyspark.sql.functions import explode
from pyspark.sql.functions import split

spark = SparkSession.builder.appName('SparkStreamApp').getOrCreate()

# default for startingOffsets is "latest", but "earliest" allows rewind for missed alerts  
df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "mytopic") \
    .option("startingOffsets", "earliest") \
    .load()

# df is the raw data stream, in "kafka" format.

ds = df.selectExpr("CAST(value AS STRING)")

# Split the lines into words
words = df.select(
   explode(
       split(df.value, " ")
   ).alias("word")
)

# Generate running word count
wordCounts = words.groupBy("word").count()

query = wordCounts \
    .writeStream \
    .outputMode("complete") \
    .format("console") \
    .start()

query.awaitTermination()
```

Once you compile/run the above created simple Python code; run the following console producer to write a few events into your topic.  

```
bash-4.4# ./kafka-console-producer.sh --broker-list localhost:9092 --topic mytopic
>Apache Spark Streaming computing
>Apache Spark Streaming computing
>Apache Spark Streaming computing
>Apache Spark Streaming
>Apache Streaming computing
``` 

### Results:  

```
+---------+-----+
|     word|count|
+---------+-----+
|   Apache|    5|
|computing|    4|
|   Spark |    4|
|Streaming|    5|
+---------+-----+  
only showing top 20 rows
```  

## Useful links 

[DStreams vs. DataFrames: Two Flavors of Spark Streaming](https://www.qubole.com/blog/dstreams-vs-dataframes-two-flavors-of-spark-streaming/)  

