---
title: 'Building a real-time big data pipeline (10: Spark Streaming, Kafka, Java)'
date: 2021-01-19
permalink: /posts/2021/01/blog-post-kafka-spark-streaming/
header:
  teaser: ""
tags:
  - big data
  - apache kafka
  - kafka
  - real time data pipelines 
  - java
  - docker
  - spark streaming 
  - YAML
  - zookeeper
  - bioinformatics
  - emory University 

---  
*Updated on January 19, 2021*  

[Apache Kafka](https://kafka.apache.org/) is a scalable, high performance and low latency platform for handling of real-time data feeds. Kafka allows reading and writing streams of data like a messaging system; written in Scala and Java.  

Kafka requires [Apache Zookeeper](https://zookeeper.apache.org/) to run. Kafka v2.5.0 (scala v2.12 build) and zookeeper (v3.4.13) were installed using docker.  

See my other blogs for installations [Kafka and Zookeeper with Docker](https://adinasarapu.github.io/posts/2020/01/blog-post-kafka/).  
 
Once we start Zookeeper and Kafka locally, we can proceed to create our first topic, named “mytopic”:  

```
bash-4.4# ./kafka-topics.sh \  
   --create \  
   --topic mytopic \  
   --partitions 1 \  
   --replication-factor 1 \  
   --bootstrap-server localhost:9092  
```  

*Spark Streaming* is an extension of the core *Apache Spark* platform that enables scalable, high-throughput, fault-tolerant processing of data streams; written in Scala but offers Scala, Java, R and Python APIs to work with. It takes data from the sources like Kafka, Flume, Kinesis, HDFS, S3 or Twitter. This data can be further processed using complex algorithms. The final output, which is the processed data can be pushed out to destinations such as HDFS filesystems, databases, and live dashboards. Spark Streaming allows you to use *Machine Learning* applications to the data streams for advanced data processing. Spark uses Hadoop's client libraries for distributed storage (HDFS) and resource management (YARN).  

Figure source [https://www.cuelogic.com](https://www.cuelogic.com/blog/analyzing-data-streaming-using-spark-vs-kafka)  
![spark-streaming](/images/spark-streaming.png)  

*Java Application*  

Spark Streaming provides a high-level abstraction that represents a continuous data stream. This abstraction of the data stream is called discretized stream or DStream. This DStream can either be created from the data streams from the sources such as Kafka, Flume, and Kinesis or other DStreams by applying high-level operations on them.

Figure source [https://www.cuelogic.com](https://www.cuelogic.com/blog/analyzing-data-streaming-using-spark-vs-kafka)  
![DStreams-sequences](/images/DStreams-sequences.png)  

Create a new Maven enabled project in Eclipse IDE.  
Update pom.xml file  

```  
<dependencies>
 <dependency>  
  <groupId>org.apache.spark</groupId>  
  <artifactId>spark-core_2.12</artifactId>  
  <version>3.0.0</version>  
 </dependency>  
 <dependency>  
  <groupId>org.apache.spark</groupId>  
  <artifactId>spark-streaming_2.12</artifactId>  
  <version>3.0.0</version>  
  <scope>provided</scope>  
 </dependency>  
  <dependency>  
  <groupId>org.apache.spark</groupId>  
  <artifactId>spark-streaming-kafka-0-10_2.12</artifactId>  
  <version>3.0.0</version>  
 </dependency>  
 <dependency>  
  <groupId>org.apache.spark</groupId>  
  <artifactId>spark-sql_2.12</artifactId>  
  <version>3.0.0</version>  
 </dependency>  
</dependencies>
```

Create the following Java class  

```
package com.example.spark;  

import java.util.Arrays;  
import java.util.Collection;  
import java.util.HashMap;  
import java.util.Map;  

import org.apache.kafka.clients.consumer.ConsumerRecord;  
import org.apache.spark.SparkConf;  
import org.apache.spark.streaming.Durations;  
import org.apache.spark.streaming.api.java.JavaDStream;  
import org.apache.spark.streaming.api.java.JavaInputDStream;  
import org.apache.spark.streaming.api.java.JavaStreamingContext;  
import org.apache.spark.streaming.kafka010.ConsumerStrategies;  
import org.apache.spark.streaming.kafka010.KafkaUtils;  
import org.apache.spark.streaming.kafka010.LocationStrategies;  

import org.apache.kafka.common.serialization.StringDeserializer;  

import org.apache.log4j.Level;  
import org.apache.log4j.Logger;  

public class JavaSparkApp {  
	public static void main(String[] args) throws InterruptedException {  
		// Firstly, we'll begin by initializing the JavaStreamingContext which is the entry point for all Spark Streaming applications:  
		Logger.getLogger("org").setLevel(Level.ALL);  
		Logger.getLogger("akka").setLevel(Level.ALL);  

		SparkConf sparkConf = new SparkConf();  
		sparkConf.setMaster("local");  
		sparkConf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer");  
		sparkConf.setAppName("WordCountingApp");  

		JavaStreamingContext streamingContext = new JavaStreamingContext(sparkConf, Durations.seconds(1));  

		// Now, we can connect to the Kafka topic from the JavaStreamingContext.  
	
		Map<String, Object> kafkaParams = new HashMap<>();  
		kafkaParams.put("bootstrap.servers", "localhost:9092");  
		kafkaParams.put("key.deserializer", StringDeserializer.class);  
		kafkaParams.put("value.deserializer", StringDeserializer.class);  
		kafkaParams.put("group.id", "group_test2");  
		kafkaParams.put("auto.offset.reset", "latest");  
		kafkaParams.put("enable.auto.commit", false);  

		Collection<String> topics = Arrays.asList("mytopic");  

		JavaInputDStream<ConsumerRecord<String, String>> messages = KafkaUtils.createDirectStream(streamingContext, 
			LocationStrategies.PreferConsistent(),  
			ConsumerStrategies.<String, String>Subscribe(topics, kafkaParams));  
		JavaDStream<String> data = messages.map(v -> { return v.value(); });  
		
		data.print();  
		
		streamingContext.start();  
		streamingContext.awaitTermination();  
	}  
}  
```  

Compile/run the above cretaed Java application and run the console producer client to write a few events into your topic.  

```  
>bash-4.4# ./kafka-console-producer.sh  
	--broker-list localhost:9092  
	--topic mytopic
>hello  
>here is my message  
```  

we should see the messages like ...  
```  
-------------------------------------------  
Time: 1593889650000 ms  
-------------------------------------------  
hello  

-------------------------------------------  
Time: 1593889670000 ms  
-------------------------------------------  
here is my message  
```

You can shut down docker-compose by executing the following command in another terminal.  

```  
bash-4.4# exit  

$docker-compose down  
```
