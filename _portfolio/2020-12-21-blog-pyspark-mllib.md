---
title: 'Building a real-time big data pipeline (part 9: Spark MLlib, Python, Regression)'
date: 2020-12-21
permalink: /posts/2020/12/blog-post-pyspark-mllib/
tags:
  - big data
  - Spark MLlib 
  - pyspark   
  - Machine Learning
  - Generalized Linear Regression
  - bioinformatics  
  - Hadoop Distributed File System
  - Emory Uiversity

---  
*Updated on December 21, 2020*  

Apache Spark MLlib [^1] [^2] [^3] is a distributed framework that provides many utilities useful for **machine learning** tasks, such as: classification, regression, clustering, dimentionality reduction and, linear algebra and statistics. Python is a general purpose popular programming language with a number of packages that support data processing and machine learning tasks. The Spark Python API (PySpark) exposes the Spark programming model to Python.   

[PySpark (Python on Spark)](https://spark.apache.org/docs/latest/api/python/index.html)  

Python, or R, DataFrame exists on one machine rather than multiple machines. If you want to do distributed computation, then you’ll need to perform operations on Spark dataframes, and not using Python or R dataframe. This has been achieved by taking advantage of the SparkR[^4] or PySpark APIs. Spark's dataframe object can be thought of as a table distributed across a cluster and has functionality that is similar to dataframe in R or python.  

## 1. Start Hadoop/HDFS  

The following command will start the namenode as well as the data nodes as cluster [^3].    
```  
cd /Users/adinasa/bigdata/hadoop-3.2.1  
$bash sbin/start-dfs.sh  
```  

Create a directory  
```  
$source ~/.bash_profile  
$hadoop fs -mkdir -p /user/adinasarapu
```  

Insert data into HDFS: Use `-copyFromLocal` command to move one or more files from local location to HDFS.  
```
$hadoop fs -copyFromLocal *.csv /user/adinasarapu
```  

Verify the files using `ls` command.  
```    
$hadoop fs -ls /user/adinasarapu  
-rw-r--r--   1 adinasa supergroup     164762 2020-08-18 17:39 /user/adinasarapu/data_proteomics.csv  
-rw-r--r--   1 adinasa supergroup        786 2020-08-18 17:39 /user/adinasarapu/samples_proteomics.csv  
```  

Start YARN with the script: `start-yarn.sh`    
```  
$bash start-yarn.sh  
Starting resourcemanager  
Starting nodemanagers  
```  

Check the list of Java processes running in your system by using the command `jps`. If you are able to see the Hadoop daemons running after executing the jps command, we can safely assume that the Hadoop cluster is running.  

```  
$jps  
96899 NodeManager  
91702 SecondaryNameNode  
96790 ResourceManager  
97240 Jps  
91437 NameNode  
91550 DataNode  
```  

Open a web browser to see your configurations for the current session.  

*Web UI*  
for HDFS: http://localhost:9870  
for YARN Resource Manager: http://localhost:8088 

## 2. PySpark, spark session in Python environment  

Configuring Eclipse IDE with PySpark and Hadoop: Here are the instructions for [Configuring Eclipse with Python and Spark on Hadoop](https://enahwe.wordpress.com)  

PySpark communicates with the Scala-based Spark via the [Py4J library](https://www.py4j.org). Py4J isn’t specific to PySpark or Spark. Py4J allows any Python program to talk to JVM-based code.  

Creating a spark context: _"The entry-point of any PySpark program is a SparkContext object. This object allows you to connect to a Spark cluster and create RDDs. The local[\*] string is a special string denoting that you’re using a local cluster, which is another way of saying you’re running in single-machine mode. The * tells Spark to create as many worker threads as logical cores on your machine. Creating a SparkContext can be more involved when you’re using a cluster. To connect to a Spark cluster, you might need to handle authentication and a few other pieces of information specific to your cluster"_ [https://realpython.com](https://realpython.com/pyspark-intro/)  

```  
from pyspark.conf import SparkConf
from pyspark.context import SparkContext
from pyspark.sql import SparkSession
from pyspark.sql.functions import when   

conf = SparkConf()
conf.setMaster('local[*]')
conf.setAppName('MyApp')
# conf.set(key, value)
conf.set("spark.executor.memory", '4g')
conf.set('spark.executor.cores', '1')
conf.set('spark.cores.max', '1')
conf.set("spark.driver.memory",'4g')

SparkContext().stop()

spark = SparkSession.builder.config(conf=conf).getOrCreate()
```  

In PySpark, SparkContext is available as `sc` by default. To create a new SparkContext, first you need to stop the default SparkContext.  
```
sc = spark.sparkContext
txt = sc.textFile('hdfs://localhost:9000/user/adinasarapu/samples_proteomics.csv')
print(txt.collect())
```  
You can start creating Resilient Distributed Datasets (RDDs) once you have a SparkContext. One way to create RDDs is to read a file with textFile(). RDDs are one of the foundational data structures in Spark. A single RDD can be divided into multiple logical partitions so that these partitions can be stored and processed on different machines of a cluster. RDDs are immutable (read-only) in nature. You cannot change an original RDD, but you can create new RDDs by performing operations, like transformations, on an existing RDD. An RDD in Spark can be cached and used again for future transformations. RDDs are said to be lazily evaluated, i.e., they delay the evaluation until it is really needed.    

[What are the Limitations of RDD in Apache Spark?](https://techvidvan.com/tutorials/spark-rdd-features/)  
RDD does not provide schema view of data. It has no provision for handling structured data. Dataset and DataFrame provide the Schema view of data. DataFrame is a distributed collection of data organized into named columns. Spark DataFrames can be created from various sources, such as external files or databases, or the existing RDDs. DataFrames allow the processing of huge amounts of data. Datasets are an extension of the DataFrame APIs in Spark. In addition to the features of DataFrames and RDDs, datasets provide various other functionalities. They provide an object-oriented programming interface, which includes the concepts of classes and objects.

Creating Spark DataFrame from CSV file:  
```
df = spark.read.format('csv').option('header',True).option('multiLine', True).load('hdfs://localhost:9000/user/adinasarapu/samples_proteomics.csv')  
df.show()  
```

```    
+--------+-------+-------+---+------+  
|SampleID|Disease|Genetic|Age|   Sex|  
+--------+-------+-------+---+------+  
|     D_2|    Yes|     No| 63|female|  
|     D_7|    Yes|     No| 78|female|  
|    D_12|    Yes|     No| 65|female|  
|    D_17|    Yes|     No| 58|female|  
|    D_22|    Yes|     No| 65|female|  
|    D_27|    Yes|     No| 50|female|  
|    D_32|    Yes|     No| 67|female|  
|    D_37|    Yes|     No| 80|female|  
|    D_42|    Yes|     No| 66|female|  
|    D_47|    Yes|     No| 64|female|  
|    DG_3|    Yes|    Yes| 76|female|  
|    DG_8|    Yes|    Yes| 56|female|  
|   DG_13|    Yes|    Yes| 81|female|  
|   DG_18|    Yes|    Yes| 69|female|  
|   DG_23|    Yes|    Yes| 60|female|  
|   DG_28|    Yes|    Yes| 63|female|  
|   DG_33|    Yes|    Yes| 70|female|  
|   DG_38|    Yes|    Yes| 46|female|  
|   DG_43|    Yes|    Yes| 65|female|  
|   DG_48|    Yes|    Yes| 77|female|  
+--------+-------+-------+---+------+  
only showing top 20 rows  
```  

```  
print(df)  
DataFrame[SampleID: string, Disease: string, Genetic: string, Age: string, Sex: string]  
```

Replacing Yes or No with 1 or 0  
```  
newDf = df.withColumn('Disease', when(df['Disease'] == 'Yes', 1).otherwise(0))  
newDf = newDf.withColumn('Genetic', when(df['Genetic'] == 'Yes', 1).otherwise(0))  
```  
```  
newDf.show()  

+--------+-------+-------+---+------+  
|SampleID|Disease|Genetic|Age|   Sex|  
+--------+-------+-------+---+------+  
|     D_2|      1|      0| 63|female|  
|     D_7|      1|      0| 78|female|  
|    D_12|      1|      0| 65|female|  
|    D_17|      1|      0| 58|female|  
|    D_22|      1|      0| 65|female|  
|    D_27|      1|      0| 50|female|  
|    D_32|      1|      0| 67|female|  
|    D_37|      1|      0| 80|female|  
|    D_42|      1|      0| 66|female|  
|    D_47|      1|      0| 64|female|  
|    DG_3|      1|      1| 76|female|  
|    DG_8|      1|      1| 56|female|  
|   DG_13|      1|      1| 81|female|  
|   DG_18|      1|      1| 69|female|  
|   DG_23|      1|      1| 60|female|  
|   DG_28|      1|      1| 63|female|  
|   DG_33|      1|      1| 70|female|  
|   DG_38|      1|      1| 46|female|  
|   DG_43|      1|      1| 65|female|  
|   DG_48|      1|      1| 77|female|  
+--------+-------+-------+---+------+  
only showing top 20 rows  
```  

Change Column type and select required columns for model building  
```  
newDf = newDf.withColumn("Disease",newDf["Disease"].cast('double'))  
newDf = newDf.withColumn("Genetic",newDf["Genetic"].cast('double'))  
newDf = newDf.withColumn("Age",newDf["Age"].cast('double'))  
```  
```  
newDf.show()    
+--------+-------+-------+----+------+  
|SampleID|Disease|Genetic| Age|   Sex|  
+--------+-------+-------+----+------+  
|     D_2|    1.0|    0.0|63.0|female|  
|     D_7|    1.0|    0.0|78.0|female|  
|    D_12|    1.0|    0.0|65.0|female|  
|    D_17|    1.0|    0.0|58.0|female|  
|    D_22|    1.0|    0.0|65.0|female|  
|    D_27|    1.0|    0.0|50.0|female|  
|    D_32|    1.0|    0.0|67.0|female|  
|    D_37|    1.0|    0.0|80.0|female|  
|    D_42|    1.0|    0.0|66.0|female|  
|    D_47|    1.0|    0.0|64.0|female|  
|    DG_3|    1.0|    1.0|76.0|female|  
|    DG_8|    1.0|    1.0|56.0|female|  
|   DG_13|    1.0|    1.0|81.0|female|  
|   DG_18|    1.0|    1.0|69.0|female|  
|   DG_23|    1.0|    1.0|60.0|female|  
|   DG_28|    1.0|    1.0|63.0|female|  
|   DG_33|    1.0|    1.0|70.0|female|  
|   DG_38|    1.0|    1.0|46.0|female|  
|   DG_43|    1.0|    1.0|65.0|female|  
|   DG_48|    1.0|    1.0|77.0|female|  
+--------+-------+-------+----+------+  
only showing top 20 rows  
```  

Prepare data for Machine Learning. And we need two columns only — features and label(“Disease”)  
```  
from pyspark.ml.feature import VectorAssembler  
vectorAssembler = VectorAssembler(inputCols = ['Genetic','Age'], outputCol = 'features')  
vhouse_df = vectorAssembler.transform(newDf)  
```  

Subset Dataset  
```  
vhouse_df = vhouse_df.select(['features', 'Disease'])  
vhouse_df.show()  
```

```  
from pyspark.ml.regression import GeneralizedLinearRegression  
glr = GeneralizedLinearRegression(featuresCol = "features", labelCol="Disease", maxIter=10, regParam=0.3, family="gaussian", link="identity")  
lr_model = glr.fit(vhouse_df)  
```  

Summarize the model over the training set  
```
trainingSummary = lr_model.summary  
print("Coefficients: " + str(lr_model.coefficients))  
print("Intercept: " + str(lr_model.intercept))
```  

Results  
```
Coefficients: [0.2797034064149404,0.008458887145555321]  
Intercept: 0.04080427059655318
```  

Finally, shutting down the HDFS  
You can stop all the daemons using the command `stop-all.sh`. You can also start or stop each daemon separately.  

```
$bash stop-all.sh
Stopping namenodes on [localhost]
Stopping datanodes
Stopping secondary namenodes [Ashoks-MacBook-Pro.2.local]
Stopping nodemanagers
Stopping resourcemanager
```

Further reading...  
[Logistic Regression in Spark ML](https://medium.com/@dhiraj.p.rai/logistic-regression-in-spark-ml-8a95b5f5434c)  
[Logistic Regression with Apache Spark](https://medium.com/rahasak/logistic-regression-with-apache-spark-b7ec4c98cfcd)  
[Feature Transformation](https://towardsdatascience.com/apache-spark-mllib-tutorial-7aba8a1dce6e)  
[PySpark: Apache Spark with Python](https://intellipaat.com/blog/tutorial/spark-tutorial/pyspark-tutorial/#_SparkContext)  
[SparkR and Sparking Water](https://rpubs.com/wendyu/sparkr)  
[Integrate SparkR and R for Better Data Science Workflow](https://blog.cloudera.com/integrate-sparkr-and-r-for-better-data-science-workflow/)  
[A Compelling Case for SparkR](https://cosminsanda.com/posts/a-compelling-case-for-sparkr/)  
[Spark – How to change column type?](https://sparkbyexamples.com/spark/spark-change-dataframe-column-type/)  
[SparkRext - SparkR extension for closer to dplyr](https://rstudio-pubs-static.s3.amazonaws.com/91559_b0a439e19f6044a9b462d0aa7b5081a2.html)  

[^1]: [Apache Spark](https://spark.apache.org)     
[^2]: [Spark MLlib: RDD-based API](https://spark.apache.org/docs/3.0.0/mllib-guide.html)
[^3]: [A tale of Spark Session and Spark Context](https://medium.com/@achilleus/spark-session-10d0d66d1d24)  
[^4]: [SparkR: Scaling R Programs with Spark](https://cs.stanford.edu/~matei/papers/2016/sigmod_sparkr.pdf)  

## References  
