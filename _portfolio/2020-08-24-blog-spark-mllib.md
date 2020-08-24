---
title: 'Building a real-time big data pipeline (part 7: Hadoop, Spark Machine Learning)'
date: 2020-08-24
permalink: /posts/2020/08/blog-post-spark-mllib/
tags:
  - big data
  - Spark MLlib 
  - scala   
  - Machine Learning
  - bioinformatics
  - Hadoop Distributed File System
  - Emory Uiversity

---  
*Updated on August 24, 2020*  

Apache Spark MLlib [^1] [^2] [^3] is a distributed framework that provides many utilities useful for **machine learning** tasks, such as:  

1. Classification  
2. Regression  
3. Clustering  
4. Modeling  
5. Dimentionality reduction  
6. Linear algebra, statistics and data handling   

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

[Installing sbt on macOS](https://www.scala-sbt.org/1.x/docs/Installing-sbt-on-Mac.html)  
```  
$brew install sbt
```  

*Create the project*  

cd to an empy directory
```
cd /Users/adinasa/Documents/bigdata/proteomics  
```  

Run the following Unix shell script which creates the initial set of files and directories you’ll want for most projects:  

`cretate_sbt_project.sh`  

```  
#!/bin/sh  

PROJ_DIR="PROJ_DIR="/Users/adinasa/Documents/bigdata/proteomics""

cd $PROJ_DIR

mkdir -p src/{main,test}/{java,resources,scala}  
mkdir lib project target  

# create an initial build.sbt file  
echo 'name := "MyProject"  
version := "1.0"  
scalaVersion := "2.12.12"' > build.sbt  
```  

If you have the tree command on your system and run it from the current directory, you’ll see that the basic directory structure looks like this:  

```  
.		
├── build.sbt				
├── lib		
├── project		
├── src		
│   ├── main		
│   │   ├── java		
│   │   ├── resources		
│   │   └── scala	
│   └── test		
│       ├── java		
│       ├── resources		
│       └── scala		
└── target		
```  

*build.sbt*  

The `build.sbt` file is SBT’s basic configuration file. You define most settings that SBT needs in this file, including specifying library dependencies, repositories, and any other basic settings your project requires.  

The dependencies are in Maven format, with `%` separating the parts. The `%%` means that it will automatically add the specific Scala version to the dependency name.  

Update the `build.sbt` file for Hadoop, Spark core and Spark Machine Learning libs [^1] [^2] [^3] [^4] [^5].   
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
import org.apache.spark.sql.Dataset;  
import org.apache.spark.sql.Row;  
import org.apache.spark.sql.SparkSession;

public class JavaExampleMain {  
	// main methos
	public static void main(String[] args) {  
		FileStatus[] fileStatus = filesFromHadoop();  
		SparkSession spark = sparkSession();  
		DatasetForML dataset = new DatasetForML();  
		dataset.setFileStatus(fileStatus);  
		dataset.setSpark(spark);  
		Dataset<Row> dataset2 = dataset.createDataSet();  
		
		//
		GeneralizedLinearRegression glr = new GeneralizedLinearRegression()  
			.setFamily("gaussian")  
			.setLink("identity")  
			.setMaxIter(10)  
			.setRegParam(0.3);  
		
		// Fit the model  
		GeneralizedLinearRegressionModel model = glr.fit(dataset2);  
		
		// Print the coefficients and intercept for generalized linear regression model  
		System.out.println("Coefficients: " + model.coefficients());  
		System.out.println("Intercept: " + model.intercept());  
		
		spark.stop();  
	}
	
	// method for SparkSession
	private static SparkSession sparkSession() {

		// The entry point to programming Spark with the Dataset and DataFrame API.  
		// The builder can be used to create a new session  
		SparkSession spark = SparkSession  
			.builder()  
			.master("local").appName("Java Spark SQL Example")  
			.config("spark.serializer", "org.apache.spark.serializer.KryoSerializer")  
			.getOrCreate();  
		return spark;  
	}  

	// Hadoop connection  
	private static FileStatus[] filesFromHadoop() {  
		Logger rootLogger = LogManager.getRootLogger();
		rootLogger.setLevel(Level.WARN);  

		Configuration conf = new Configuration();  
		URI uri = null;  
		try {  
			uri = new URI("hdfs://localhost:9000/user/adinasarapu");  
		} catch (URISyntaxException e1) {  
			e1.printStackTrace();  
		}  
		FileSystem fs = null;  
		try {  
			fs = FileSystem.get(uri, conf);  
		} catch (IOException e) {  
			e.printStackTrace();  
		}  
			FileStatus[] fileStatus = null;  
		try {  
			fileStatus = fs.listStatus(new Path(uri));  
		} catch (FileNotFoundException e) {  
			e.printStackTrace();  
		} catch (IOException e) {  
			e.printStackTrace();  
		}  
		return fileStatus;  
	}  
}  
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

	FileStatus[] fileStatus = null;  
	SparkSession spark = null;  

	public DatasetForML() {	}  

	public FileStatus[] getFileStatus() {  
		return fileStatus;  
	}  

	public void setFileStatus(FileStatus[] fileStatus) {  
		this.fileStatus = fileStatus;  
	}  

	public SparkSession getSpark() {  
		return spark;  
	}  
	
	public void setSpark(SparkSession spark) {  
		this.spark = spark;  
	}  
	
	public Dataset<Row> createDataSet() {  
		 
		Dataset<Row> dataset = null;  
		String data_file = null;  
		String samples_file = null;
		
		// Check if file exists at the given location
		for (FileStatus status : fileStatus) {  
			String file_name = status.getPath().toString();  
			File f = new File(file_name);
  
			if(f.getName().startsWith("data_")) {  
				data_file = file_name;    
			}  

			if(f.getName().startsWith("samples_")) {  
				samples_file = file_name;  
			}  
		}  

		Dataset<Row> df_sample = spark.read()  
				.option("inferSchema","true")  
				.option("header", "true")  
				.csv(samples_file)  
				.withColumnRenamed("Disease", "label");  
		
		Dataset<Row> df_sample2 = df_sample.withColumn("label", when(col("label").isNotNull()  
				.and(col("label").equalTo(lit("Yes"))),lit(1))
				.otherwise(lit(0)));  
		
		Dataset<Row> df_sample3 = df_sample2.select("SampleID","label", "Age");  

		String[] myStrings = {"label","Age"};  
		VectorAssembler VA = new  VectorAssembler()  
				.setInputCols(myStrings)  
				.setOutputCol("features");  
		dataset = VA.transform(df_sample3);  
		return dataset;  
	}  
}  
```  

Run the project:`sbt run`  
```  
[info] welcome to sbt 1.3.13 (Oracle Corporation Java 1.8.0_261)  
[info] loading settings for project proteomics-build from plugins.sbt ...  
[info] loading project definition from /Users/adinasa/Documents/bigdata/proteomics/project  
[info] loading settings for project proteomics from built.sbt ...  
[info] set current project to MyProject (in build file:/Users/adinasa/Documents/bigdata/proteomics/)  
...  
...  
...  
Coefficients: [0.5885374357547469,0.004125098728643811]  
Intercept: 0.01456465954989681  
```

Finally,
*Shutting Down the HDFS*:
You can stop all the daemons using the command `stop-all.sh`. You can also start or stop each daemon separately.

```
$bash stop-all.sh
Stopping namenodes on [localhost]
Stopping datanodes
Stopping secondary namenodes [Ashoks-MacBook-Pro.2.local]
Stopping nodemanagers
Stopping resourcemanager
```

[^1]: [Apache Spark](https://spark.apache.org)     
[^2]: [Spark MLlib: RDD-based API](https://spark.apache.org/docs/3.0.0/mllib-guide.html)
[^3]: [A tale of Spark Session and Spark Context](https://medium.com/@achilleus/spark-session-10d0d66d1d24)  
[^4]: [Apache Hadoop](https://hadoop.apache.org)
[^5]: [NotSerializableException](https://stackoverflow.com/questions/55993313/not-serialazable-exception-while-running-linear-regression-scala-2-12)  

## References  
