---
title: 'Building a real-time big data pipeline (part 7: Hadoop, Spark MLlib, Generalized Linear Regression)'
date: 2020-08-24
permalink: /posts/2020/08/blog-post-spark-mllib/
tags:
  - big data
  - Spark MLlib 
  - scala   
  - Machine Learning
  - Generalized Linear Regression
  - bioinformatics  
  - Hadoop Distributed File System
  - Emory Uiversity

---  
*Updated on September 20, 2020*  

Apache Spark MLlib [^1] [^2] [^3] is a distributed framework that provides many utilities useful for **machine learning** tasks, such as:  

1. Classification  
2. Regression  
3. Clustering  
4. Dimentionality reduction  
5. Linear algebra, statistics and data handling   

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

## 2. The Scala Build Tool (SBT)  

SBT is an interactive build tool for Scala, Java, and more. It requires Java 1.8 or later.  

[How to install, and create an initial set of files and directories for any SBT project](https://adinasarapu.github.io/posts/2020/08/blog-post-spark-sbt/)  

Update the `build.sbt` file for Hadoop, Spark core and Spark Machine Learning libraries [^1] [^2] [^3] [^4] [^5]. Use Maven repository to get any `libraryDependencies`. See [Spark Project ML Library](https://mvnrepository.com/artifact/org.apache.spark/spark-mllib) at Maven repository.  
```  
name := "MyProject"
version := "1.0"

// Scala version above 2.12.8 is necessary to prevent "Task not serializable: java.io.NotSerializableException ..."  
scalaVersion := "2.12.12"

mainClass := Some("com.test.rdd.JavaExampleMain")  

libraryDependencies += "org.apache.hadoop" % "hadoop-common" % "3.2.1";  
libraryDependencies += "org.apache.hadoop" % "hadoop-hdfs-client" % "3.2.1";  
libraryDependencies += "org.apache.spark" %% "spark-core" % "3.0.0";  
libraryDependencies += "org.apache.spark" %% "spark-sql" % "3.0.0";  
libraryDependencies += "org.apache.spark" %% "spark-mllib" % "3.0.0"  
```  

## 3. Java application  

`GeneralizedLinearRegression` is a regression algorithm. There are two steps to successfully run this application. 

First to fit a Generalized Linear Model (GLM), use a symbolic description of the linear predictor (link function) and a description of the error distribution (family) from the following table.

For example, `GeneralizedLinearRegression glr = new GeneralizedLinearRegression().setFamily("gaussian").setLink("identity")`    
  
| Family (Error distribution) 	| Link function (Linear predictor) |  
| ----------------------------- | -------------------------------- |  
| gaussian			| identity, log, inverse	   |  
| binomial			| logit, probit, cloglog	   |  
| poisson			| log, identity, sqrt		   |  
| gamma				| inverse, identity, log	   |  

Second, relate column names to model parameters (label and features).  
`label`: dependent variable in the model  
`features` is a vector with independent variables in the model  

Create a file named `JavaExampleMain.java` in the `src/main/java` directory.  
```  
package com.test.rdd; 
 
import static org.apache.spark.sql.functions.col;  
import static org.apache.spark.sql.functions.lit;  
import static org.apache.spark.sql.functions.when;  
import java.io.File;  
import java.io.FileNotFoundException;  
import java.io.IOException;  
import java.net.URI;  
import java.net.URISyntaxException;  
import org.apache.hadoop.conf.Configuration;  
import org.apache.hadoop.fs.FileStatus;  
import org.apache.hadoop.fs.FileSystem;  
import org.apache.hadoop.fs.Path;  
import org.apache.log4j.Level;  
import org.apache.log4j.LogManager;  
import org.apache.log4j.Logger;  
import org.apache.spark.ml.feature.VectorAssembler;  
import org.apache.spark.ml.regression.GeneralizedLinearRegression;  
import org.apache.spark.ml.regression.GeneralizedLinearRegressionModel;  
import org.apache.spark.ml.regression.GeneralizedLinearRegressionTrainingSummary;  
import org.apache.spark.sql.Dataset;  
import org.apache.spark.sql.Row;  
import org.apache.spark.sql.SparkSession;  

public class JavaExampleMain {  

    public static void main(String[] args) {  

	// Hadoop: get file-list from HDFS  

	URI uri = null;  
	try {  
		uri = new URI("hdfs://localhost:9000/user/adinasarapu");  
	} catch (URISyntaxException e1) {  
		e1.printStackTrace();  
	}  
	
	FileStatus[] fileStatus = filesFromHadoop(uri);

        // This is the entry point to programming Spark with the Dataset and DataFrame API.  
	// The builder can also be used to create a new session  
	// If you didn't specify serialization in spark context you are using the default java serialization...  
	// Spark runs on YARN, in cluster mode. spark.serializer is set to org.apache.spark.serializer.KryoSerializer  

        SparkSession spark = SparkSession  
		.builder()  
		.master("local").appName("Java Spark SQL Example")  
		.config("spark.serializer", "org.apache.spark.serializer.KryoSerializer")  
		.getOrCreate();  

        // Create Dataset<Row> object  
        
	DatasetForML dataset = new DatasetForML();  
	dataset.setFileStatus(fileStatus);  
	dataset.setSpark(spark);  
	Dataset<Row> dataset2 = dataset.createDataSet();  

	// Here you can also set label column and features columns  
	// label: dependent variable in the model  
	// features is a vector and independent variables in the model  

       GeneralizedLinearRegression glr = new GeneralizedLinearRegression()  
		.setFamily("gaussian")  
		.setLink("identity")  
		.setMaxIter(10)  
		.setRegParam(0.3);  

                // .setLabelCol("label")  
		// .setFeaturesCol("features");  

	// Fit the model  

	GeneralizedLinearRegressionModel model = glr.fit(dataset2);  

        // Print the coefficients and intercept for generalized linear regression model  

        GeneralizedLinearRegressionTrainingSummary summary = model.summary();  
        System.out.println(summary.toString());  
```   

Create a file named `DatasetForML.java` in the `src/main/java` directory.  

```  
package com.test.rdd;  

import static org.apache.spark.sql.functions.col;  
import static org.apache.spark.sql.functions.lit;  
import static org.apache.spark.sql.functions.when;  
import java.io.File;  
import org.apache.hadoop.fs.FileStatus;  
import org.apache.spark.ml.feature.VectorAssembler;  
import org.apache.spark.sql.Dataset;  
import org.apache.spark.sql.Row;  
import org.apache.spark.sql.SparkSession;  

public class DatasetForML {  
    private static final long serialVersionUID = 1L; 
    
    // initialize with null
    FileStatus[] fileStatus = null;  
    SparkSession spark = null;  
	
    // No-argument constructor  
    public DatasetForML() { }  

    public void setFileStatus(FileStatus[] fileStatus) {  
       this.fileStatus = fileStatus;   
    }  

    public void setSpark(SparkSession spark) {
       this.spark = spark;
    }

    public FileStatus[] getFileStatus() {  return fileStatus; }  

    public SparkSession getSpark() {  return spark; }     

    public Dataset<Row> createDataSet() {   
       
       Dataset<Row> dataset = null;   
       String data_file = null;   
       String samples_file = null;  
     
       for (FileStatus status : fileStatus) {  
           String file_name = status.getPath().toString();  
           File f = new File(file_name);  
           if (f.getName().startsWith("data_")) {  
              data_file = file_name;  
           }  
           if (f.getName().startsWith("samples_")) {  
              samples_file = file_name;
           }  
       }  
 
       // Read samples file as Dataset<Row>  
       // Replace column name Disease with label as .withColumnRenamed("Disease", "label")  
       // Now label column is a target (acts as dependent variable in the model)  
       // Optionally you can use setLabelCol("Disease") with GeneralizedLinearRegression class  

       Dataset<Row> df_sample = spark.read()  
		.option("inferSchema", "true")  
		.option("header", "true")  
		.csv(samples_file)  
		.withColumnRenamed("Disease", "label");  

       // Replace Yes or NO with 1 or 0  

       Dataset<Row> df_sample2 = df_sample  
		.withColumn("label", when(col("label").isNotNull()  
			.and(col("label").equalTo(lit("Yes"))), lit(1)).otherwise(lit(0)))  
		.withColumn("Genetic",when(col("Genetic").isNotNull()  
			.and(col("Genetic").equalTo(lit("Yes"))),lit(1)).otherwise(lit(0)));  

       // Subset the Dataset  

       Dataset<Row> df_sample3 = df_sample2.select("SampleID", "label", "Genetic","Age");  
       
       // df_sample3.show();  
       // df_sample3.printSchema();  

       // VectorAssembler is a transformer that combines a given list of columns into a single vector column  
       // We want to combine Genetic and Age into a single feature vector called features (acts as independent variables in the model)  
       // Optionally you can use setFearuresCol("name_of_a_vector") with GeneralizedLinearRegression class

       String[] myStrings = {"Genetic", "Age"};  
       VectorAssembler VA = new VectorAssembler().setInputCols(myStrings).setOutputCol("features");  
       dataset = VA.transform(df_sample3);  

       // dataset.show();
       return dataset;    
}  
```  

## 4. Code compilation and results  

To compile and run the project use either SBT, IntelliJ IDEA or Eclipse IDE  

For `SBT`,  

run `sbt run` in the directory where `build.sbt` file is present.  

For Intellij IDEA,   
1. Download and intall IntelliJ IDEA  
2. Add the Scala plugin (IntelliJ IDEA -> Preferences -> Plugins)  
3. Import an SBT project (From the `Welcome Screen` or `File` -> Open; Browse to and select the top-level folder of your sbt project, and click OK)  

For [Eclipse IDE](https://adinasarapu.github.io/posts/2020/08/blog-post-spark-sbt/)  

Results  
```  
Coefficients:  
	Feature     Estimate  Std Error T Value	P Value  
	(Intercept) 0.0408    0.3501	0.1165	0.9081  
	Genetic     0.2797    0.1278	2.1878	0.0375  
	Age   	    0.0085    0.0055	1.5454	0.1339  

(Dispersion parameter for gaussian family taken to be 0.1753)  
	Null deviance: 6.6667 on 27 degrees of freedom  
	Residual deviance: 4.7321 on 27 degrees of freedom  
AIC: 37.7313  
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

[^1]: [Apache Spark](https://spark.apache.org)     
[^2]: [Spark MLlib: RDD-based API](https://spark.apache.org/docs/3.0.0/mllib-guide.html)
[^3]: [A tale of Spark Session and Spark Context](https://medium.com/@achilleus/spark-session-10d0d66d1d24)  
[^4]: [Apache Hadoop](https://hadoop.apache.org)
[^5]: [NotSerializableException](https://stackoverflow.com/questions/55993313/not-serialazable-exception-while-running-linear-regression-scala-2-12)  

## References  
