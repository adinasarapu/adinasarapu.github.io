---
title: 'Building a real-time big data pipeline (part 1: Kafka)'
date: 2020-01-26
permalink: /posts/2020/01/blog-post-kafka/
tags:
  - big data
  - apache kafka
  - real time data pipelines 
  - java
  - docker
  - Spring Boot 
  - YAML
  - Zookeeper  

---  
[Kafka](https://kafka.apache.org) is used for building real-time data pipelines and streaming apps.  

What is Kafka? [Getting started with kafka](https://success.docker.com/article/getting-started-with-kafka) says “Kafka is a distributed append log; in a simplistic view it is like a file on a filesystem. Producers can append data (echo 'data' >> file.dat), and consumers subscribe to a certain file (tail -f file.dat). In addition, Kafka provides an ever-increasing counter and a timestamp for each consumed message. Kafka uses Zookeeper (simplified: solid, reliable, transactional key/value store) to keep track of the state of producers, topics, and consumers.”  

**Kafka for local development of applications**:  
There are multiple ways of running Kafka locally for development of apps but the easiest method is by docker-compose. To download Docker Desktop, go to [Docker Hub](https://hub.docker.com/) and sign In with your Docker ID.  

Docker compose facilitates installing Kafka and Zookeeper with the help of `docker-compose.yml` file.  

```
version: '2'  
services:  
  zookeeper:  
    image: wurstmeister/zookeeper  
    ports:  
      - "2181:2181"  
  kafka:  
   image: wurstmeister/kafka  
    ports:  
      - "9092:9092"  
    environment:  
     KAFKA_ADVERTISED_HOST_NAME: localhost  
     KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181  
     KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"  
    volumes:  
     - /var/run/docker.sock:/var/run/docker.sock
```  

**1. Start the Kafka service**  
Open a terminal, go where you have the docker-compose.yml file, and execute the following command. This command starts the docker-compose engine, and it downloads the images and runs them.  

```
$docker-compose up -d  
```   

```
Starting kafka_example_zookeeper_1 ... done  
Starting kafka_example_kafka_1     ... done  
```  

To list running docker containers, run the following command  

```  
$docker-compose ps  
```   
```   
Name				Command				State	Ports  
kafka_example_kafka_1		start-kafka.sh			Up	0.0.0.0:9092->9092/tcp                              
kafka_example_zookeeper_1	/bin/sh -c /usr/sbin/sshd  ...	Up	0.0.0.0:2181->2181/tcp, 22/tcp, 2888/tcp, 3888/tcp  
```  

You can shut down docker-compose by executing the following command in another terminal.  

```
$docker-compose down  
```  
```  
Stopping kafka_example_zookeeper_1 ... done  
Stopping kafka_example_kafka_1     ... done  
Removing kafka_example_zookeeper_1 ... done  
Removing kafka_example_kafka_1     ... done  
Removing network kafka_example_default  
```  

Using the following command check the ZooKeeper logs to verify that ZooKeeper is working and healthy.  

```  
$docker-compose logs zookeeper | grep -i binding  
```  

Next, check the Kafka logs to verify that broker is working and healthy.  

```  
$docker-compose logs kafka | grep -i started  
```  

**2. Create a topic**  
The Kafka cluster stores streams of records in categories called topics. A topic can have zero, one, or many consumers that subscribe to the data written to it. `docker-compose exec` command by default allocates a TTY, so that you can use such a command to get an interactive prompt. Go into directory where docker-compose.yml file present, and execute it as  

```  
$docker-compose exec kafka bash  
```  

(for zookeeper `$docker-compose exec zookeeper bash`)  

Change the directory to /opt/kafka/bin where you find scripts such as `kafka-topics.sh`.  
`cd /opt/kafka/bin`  

Create, list or delete existing topics:  

```
bash-4.4# ./kafka-topics.sh \  
 --create \  
 --topic test1 \  
 --partitions 1 \  
 --replication-factor 1 \  
 --bootstrap-server localhost:9092  
```  

```
bash-4.4# ./kafka-topics.sh \
 --list \
 --bootstrap-server localhost:9092  
```

If necessary, delete a topic using the following command.  

```
bash-4.4# ./kafka-topics.sh \
 --delete \
 --topic test1 \
 --bootstrap-server localhost:9092  
```  

**3. Producer and Consumer**  
A Kafka producer is an object that consists of a pool of buffer space that holds records that haven't yet been transmitted to the server.  

The following is a producer command line to read data from standard input and write it to a Kafka topic.  

```
bash-4.4# ./kafka-console-producer.sh --broker-list localhost:9092 --topic test1  
>Hello  
>World  
^C
```

The following is a command line to read data from a Kafka topic and write it to standard output.  
```  
bash-4.4# ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test1 --from-beginning  
Hello  
World  
^CProcessed a total of 2 messages  
```  

**Another way of reading data from a Kafka topic is by simply using a Spring Boot application** (Here I call this project as SpringBootKafka).  

The following demonstrates how to receive messages from Kafka topic. First in this blog I create a Spring Kafka Consumer, which is able to listen the messages sent to a Kafka topic. Then in next blog I create a Spring Kafka Producer, which is able to send messages to a Kafka topic.  

The first step to create a simple Spring Boot maven Application is [Starting with Spring Initializr](https://spring.io/guides/gs/spring-boot/) and make sure to have spring-kafka dependency to `pom.xml`.  

```  
<dependency>  
  <groupId>org.springframework.kafka</groupId>  
  <artifactId>spring-kafka</artifactId>  
</dependency>  
```  

`SpringBootKafkaApplication.java` class (The Spring Initializr creates the following simple application class for you)

```    
@SpringBootApplication  
public class SpringBootKafkaApplication {  
	public static void main(String[] args) {  
		SpringApplication.run(SpringBootApplication.class, args);  
	}  
}  
```

In Spring Boot, properties are kept in the `application.properties` file under the classpath. The application.properties file is located in the `src/main/resources` directory. Change application.properties file to `application.yml`, then add the following content.  

```  
spring:  
  kafka:  
consumer:  
  bootstrap-servers: localhost:9092  
  group-id: group_test1  
  auto-offset-reset: earliest  
  key-deserializer: org.apache.kafka.common.serialization.StringDeserializer  
  value-deserializer: org.apache.kafka.common.serialization.StringDeserializer  
```  

Create a Spring Kafka Consumer class:  

Create a class called `KafkaConsumer.java` and add a method with the @KakfaListener annotation.  

```  
@Service  
public class KafkaConsumer {  
	@KafkaListener(id = "group_test1", topics = "test1")  
	public void consumeMessage(String message) {  
		System.out.println("Consumed message: " + message);  
	}  
}
```  

How to run Spring Boot web application in Eclipse?  

In eclipse Project Explorer, right click the project name -> select "Run As" -> "Maven Build..."  
In the goals, enter `spring-boot:run`  
then click Run button.  

If you have Spring Tool Suite (STS) plug-in, you see a "Spring Boot App" option under Run As.  

Start Kafka and Zookeeper: `$docker-compose up -d`  
Produce message using the Kafka console producer: `$docker-compose exec kafka bash`  
Once inside the container `cd /opt/kafka/bin`  
Run the following console producer which will enable you to send messages to Kafka:  

```  
bash-4.4# ./kafka-console-producer.sh --broker-list localhost:9092 --topic test1  
>Hello  
>World  
^C  
```  

Try sending a few messages like above (Hello, World etc) and watch the application standard output in the Eclipse shell where you are running your Spring Boot application.  

Eclipse Console:  
```  
  .   ____          _            __ _ _  
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \  
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \  
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )  
  '  |____| .__|_| |_|_| |_\__, | / / / /  
 =========|_|==============|___/=/_/_/_/  

:: Spring Boot ::        (v2.2.4.RELEASE)  
2020-01-26 14:26:55.205  INFO 11137 --- [           main] c.e.d.SpringBootKafkaConsumerApplication : Starting   
…  
…  
…
o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-1, groupId=simpleconsumer] Setting newly assigned partitions: test1-0  
2020-01-26 14:26:56.384  INFO 11137 --- [econsumer-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-1, groupId=simpleconsumer] Found no committed offset for partition test1-0  
2020-01-26 14:26:56.408  INFO 11137 --- [econsumer-0-C-1] o.a.k.c.c.internals.SubscriptionState    : [Consumer clientId=consumer-1, groupId=simpleconsumer] Resetting offset for partition test1-0 to offset 2.  
2020-01-26 14:26:56.477  INFO 11137 --- [econsumer-0-C-1] o.s.k.l.KafkaMessageListenerContainer    : simpleconsumer: partitions assigned: [test1-0]  
Got message: hello  
Got message: world  
```  

The following code demonstrates how to send and receive messages from Kafka topic. The above `KafkaConsumer.java` receives messages that were sent to a Kafka topic. The followng `KafkaProducer.java` send messages to a Kafka topic.          

Make sure to have spring-web dependency to `pom.xml`.

```  
<dependency>  
  <groupId>org.springframework.boot</groupId>  
  <artifactId>spring-boot-starter-web</artifactId>  
</dependency>  
```  

Add two new java classes `KafkaController.java` and `KafkaProducer.java`  

```
@RestController  
@RequestMapping(value="/kafka")  
public class KafkaController {  
	private final KafkaProducer producer;  

	@Autowired  
	KafkaController(KafkaProducer producer){  
		this.producer = producer;  
	}  

	@PostMapping(value="/publish")  
	public void messagePrint(@RequestParam(value="message", required = false) String message) {  
		this.producer.sendMessage(message);  
	}  
}  
```  

```  
@Service  
public class KafkaProducer {  
	private static final String TOPIC = "test1";  

	@Autowired  
	private KafkaTemplate<String, String> kafkaTemplate;  
	
	public void sendMessage(String message) {  
		kafkaTemplate.send(TOPIC, message);  
		System.out.println("Produced message: " + message);  
	}  
}  
```  

Update `application.yml` file.  

```
server:
  port: 8080
spring:
  kafka:
   consumer:
    bootstrap-servers: localhost:9092
    group-id: group_test1
    auto-offset-reset: earliest
    key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    value-deserializer: org.apache.kafka.common.serialization.StringDeserializer  
   producer:  
    bootstrap-servers: localhost:9092  
    key-serializer: org.apache.kafka.common.serialization.StringSerializer  
    value-serializer: org.apache.kafka.common.serialization.StringSerializer  
```

Run Spring Boot web application (see How to run Spring Boot web application in Eclipse?)


Make POST request using [Postman](https://www.getpostman.com).  

Select `POST` and use the API `http://localhost:8080/kafka/publish`  
**Body**: form-data  **KEY**: message  **VALUE**: hello
 
Finally click **send**.  


See Eclipse Console for messages:  

```
...  
...  
...  
2020-01-27 13:12:06.911  INFO 31822 --- [nio-8080-exec-2] o.a.kafka.common.utils.AppInfoParser     : Kafka version: 2.3.1  
2020-01-27 13:12:06.912  INFO 31822 --- [nio-8080-exec-2] o.a.kafka.common.utils.AppInfoParser     : Kafka commitId: 18a913733fb71c01  
2020-01-27 13:12:06.912  INFO 31822 --- [nio-8080-exec-2] o.a.kafka.common.utils.AppInfoParser     : Kafka startTimeMs: 1580148726911  
2020-01-27 13:12:06.947  INFO 31822 --- [ad | producer-1] org.apache.kafka.clients.Metadata        : [Producer clientId=producer-1] Cluster ID: 6R8O95IPSfGoifR4zzwM6g  
Produced message: hello  
Consumed message: hello
```  
