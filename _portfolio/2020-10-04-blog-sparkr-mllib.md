---
title: 'Building a real-time big data pipeline (part 8: Spark MLlib, R, Regression)'
date: 2020-10-04
permalink: /posts/2020/10/blog-post-sparkr-mllib/
tags:
  - big data
  - Spark MLlib 
  - sparkR   
  - Machine Learning
  - Generalized Linear Regression
  - bioinformatics  
  - Hadoop Distributed File System
  - Emory Uiversity

---  
*Updated on October 04, 2020*  

Apache Spark MLlib [^1] [^2] [^3] is a distributed framework that provides many utilities useful for **machine learning** tasks, such as: Classification, Regression, Clustering, Dimentionality reduction and, Linear algebra, statistics and data handling. R is single threaded and it is often impractical to use R on large datasets. To address R’s scalability issue, the Spark community developed SparkR package which is based on a distributed data frame that enables structured data processing with a syntax familiar to R users.  

[SparkR (R on Spark)](https://spark.apache.org/docs/3.0.0/sparkr.html#overview)  

_"SparkR is an R package that provides a light-weight frontend to use Apache Spark from R. In Spark 3.0.0, SparkR provides a distributed data frame implementation that supports operations like selection, filtering, aggregation etc. (similar to R data frames, dplyr) but on large datasets. SparkR also supports distributed machine learning using MLlib"._  

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

## 2. SparkR, `spark session` in R environment  

a. Start sparkR shell from Spark installation

cd to spark installation directory and run `sparkR` command   
```  
cd bigdata/spark-3.0.0-bin-hadoop3.2/bin/  
>sparkR

...
...
...

 Welcome to
    ____              __ 
   / __/__  ___ _____/ /__ 
  _\ \/ _ \/ _ `/ __/  '_/ 
 /___/ .__/\_,_/_/ /_/\_\   version  3.0.0 
    /_/ 


 SparkSession available as 'spark'.
During startup - Warning message:
In SparkR::sparkR.session() :
  Version mismatch between Spark JVM and SparkR package. JVM version was 3.0.0 , while R package version was 2.4.6
```  
b. Install `SparkR` package   
```  
install.packages(remotes)  
library(remotes)
install_github(“cran/SparkR”)
```  
c. Open RStudio and set SPARK_HOME and JAVA_HOME  

Check java version `system("java -version")`  
If you see an error like "R looking for the wrong java version" set `Sys.setenv(JAVA_HOME=java_path)`  
```
Error in checkJavaVersion() : 
  Java version 8 is required for this package; found version: 14.0.2
``` 

```  
Sys.setenv(SPARK_HOME = "/Users/adinasa/bigdata/spark-3.0.0-bin-hadoop3.2")
.libPaths(c(file.path(Sys.getenv("SPARK_HOME"), "R", "lib"), .libPaths()))

java_path <- normalizePath("/Library/Java/JavaVirtualMachines/jdk1.8.0_261.jdk/Contents/Home")
Sys.setenv(JAVA_HOME=java_path)

# load 
library(SparkR)

# SparkSession is the entry point into SparkR. 
SparkR::sparkR.session(master = "local", sparkHome = Sys.getenv("SPARK_HOME"), appName = "SparkR", spark.executor.memory = "2g")

# a list of config values with keys as their names
sparkR.conf()

# In Spark 2.0 csv is natively supported so you should be able to do something like this:
s.df <- SparkR::read.df("hdfs://localhost:9000/user/adinasarapu/samples_proteomics.csv", source = "csv", header = "true")

# SQL like query
createOrReplaceTempView(s.df, "df_view")
new_df <- SparkR::sql("SELECT * FROM df_view WHERE Age < 51")

# Spark Dataframe to R DataFrame
r.df <- SparkR::as.data.frame(new_df)

r.df
```

Output,   

|SampleID 	 | Disease  | Genetic | Age  | Sex   |
| -------------- | -------- | ------- | ---- | ----- | 
| D_27		 | Yes      | No      | 50   | female|
| DG_38          | Yes      | Yes     | 46   | female|
| Control_16     | No       | No      | 41   | female|
| Control_41     | No       | No      | 43   | female|
| Control_46     | No       | No      | 42   | female| 

Regression analysis script will be added soon!!!  

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

[^1]: [Apache Spark](https://spark.apache.org)     
[^2]: [Spark MLlib: RDD-based API](https://spark.apache.org/docs/3.0.0/mllib-guide.html)
[^3]: [A tale of Spark Session and Spark Context](https://medium.com/@achilleus/spark-session-10d0d66d1d24)  
[^4]: [Apache Hadoop](https://hadoop.apache.org)
[^5]: [NotSerializableException](https://stackoverflow.com/questions/55993313/not-serialazable-exception-while-running-linear-regression-scala-2-12)  

## References  
