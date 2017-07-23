---
title: Spark Quick Start examples
tags: Spark
categories: bigdata
---
### Spark Quick Start

__Scala:__
``` scala
./bin/spark-shell
scala> val textFile = sc.textFile("README.md")
scala> textFile.count()
scala> textFile.first()
scala> val linesWithSpark = textFile.filter(line => line.contains("Spark"))
scala> textFile.filter(line => line.contains("Spark")).count()
```
more on RDD operations:
``` scala
scala> textFile.map(line => line.split(" ").size).reduce((a, b) => if (a > b) a else b)
scala> import java.lang.Math
scala> textFile.map(line => line.split(" ").size).reduce((a, b) => Math.max(a, b))
scala> val wordCounts = textFile.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey((a, b) => a + b)
scala> wordCounts.collect()
```
Self-Contained  Applications:
``` scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf
object SimpleApp {
  def main(args: Array[String]) {
    val logFile = "YOUR_SPARK_HOME/README.md" // Should be some file on your system
    val conf = new SparkConf().setAppName("Simple Application")
    val sc = new SparkContext(conf)
    val logData = sc.textFile(logFile, 2).cache()
    val numAs = logData.filter(line => line.contains("a")).count()
    val numBs = logData.filter(line => line.contains("b")).count()
    println("Lines with a: %s, Lines with b: %s".format(numAs, numBs))
  }
}
```
SparkConf:
```
name := "Simple Project"
version := "1.0"
scalaVersion := "2.11.7"
libraryDependencies += "org.apache.spark" %% "spark-core" % "2.0.1"
```
__Python:__
``` python
./bin/pyspark
>>> textFile = sc.textFile("README.md")
>>> textFile.count()
>>> textFile.first()
>>> linesWithSpark = textFile.filter(lambda line: "Spark" in line)
>>> textFile.filter(lambda line: "Spark" in line).count()
```
more on RDD operations:
``` python
>>> textFile.map(lambda line: len(line.split())).reduce(lambda a, b: a if (a > b) else b)
>>> def max(a, b):
...     if a > b:
...         return a
...     else:
...         return b
...
>>> textFile.map(lambda line: len(line.split())).reduce(max)
>>> wordCounts = textFile.flatMap(lambda line: line.split()).map(lambda word: (word, 1)).reduceByKey(lambda a, b: a+b)
>>> wordCounts.collect()
```
Self-Contained  Applications:
``` python
from pyspark import SparkContext

logFile = "YOUR_SPARK_HOME/README.md"  # Should be some file on your system
sc = SparkContext("local", "Simple App")
logData = sc.textFile(logFile).cache()
numAs = logData.filter(lambda s: 'a' in s).count()
numBs = logData.filter(lambda s: 'b' in s).count()
print("Lines with a: %i, lines with b: %i" % (numAs, numBs))

$ YOUR_SPARK_HOME/bin/spark-submit \
  --master local[4] \
  SimpleApp.py
```

## Spark Programming Guide
参考： http://spark.apache.org/docs/latest/programming-guide.html
