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

Apache Spark MLlib [^1] [^2] [^3] is a distributed framework that provides many utilities useful for **machine learning** tasks, such as: Classification, Regression, Clustering, Dimentionality reduction and, Linear algebra, statistics and data handling. Python is a popular programming language with a number of packages that support data processing and machine learning tasks. The Spark Python API (PySpark) exposes the Spark programming model to Python.  

[PySpark (Python on Spark)](https://spark.apache.org/docs/latest/api/python/index.html)  

Python or R DataFrames exist on one machine rather than multiple machines. If you want to do distributed computation, then you’ll need to perform operations on Spark dataframes, and not using Python or R data types. This has been achieved by taking advantage of the SparkR[^4] or PySpark APIs. Spark's data frame object can be thought of as a table distributed across a cluster and has functionality that is similar to dataframe in R.  

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

## 2. PySpark, `spark session` in Python environment  

a. Configuring Eclipse with PySpark and Hadoop  

[Configuring Eclipse with Python and Spark on Hadoop](https://enahwe.wordpress.com)  

Create Spark session   
```  
from pyspark.sql import SparkSession  
from pyspark.sql.functions import when   

spark = SparkSession.builder.master('local').appName("HDFS").getOrCreate()  
```  

Convert list to data frame  
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

Replace Yes or No with 1 or 0  
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