---
title: Spark Cluster 三种模式
tags: Spark
categories: bigdata
---
## Spark Cluster 三种模式
- __Standalone__ – a simple cluster manager included with Spark that makes it easy to set up a cluster.
- __Apache Mesos__ – a general cluster manager that can also run Hadoop MapReduce and service applications.
- __Hadoop YARN__ – the resource manager in Hadoop 2.

### Standalone 模式：
start Master:
```bash
./sbin/start-master.sh
```
start Slave:
```bash
./sbin/start-slave.sh <master-spark-URL>
```

- sbin/start-master.sh - Starts a master instance on the machine the script is executed on.
- sbin/start-slaves.sh - Starts a slave instance on each machine specified in the conf/slaves file.
- sbin/start-slave.sh - Starts a slave instance on the machine the script is executed on.
- sbin/start-all.sh - Starts both a master and a number of slaves as described above.
- sbin/stop-master.sh - Stops the master that was started via the bin/start-master.sh script.
- sbin/stop-slaves.sh - Stops all slave instances on the machines specified in the conf/slaves file.
- sbin/stop-all.sh - Stops both the master and the slaves as described above.

Connecting an Application to the Cluster:
```bash
./bin/spark-shell --master spark://IP:PORT
```
Launching Spark Applications:
```bash
./bin/spark-class org.apache.spark.deploy.Client kill <master url> <driver ID>
```
Resource Scheduling:
```Scala
val conf = new SparkConf()
             .setMaster(...)
             .setAppName(...)
             .set("spark.cores.max", "10")
val sc = new SparkContext(conf)
```
### Running Spark on Mesos 模式：
参考： http://spark.apache.org/docs/latest/running-on-mesos.html

__Installing Mesos:__
- Spark 2.0.1 is designed for use with Mesos 0.21.0. http://mesos.apache.org/gettingstarted/
__Connecting  Spark  to  Mesos:__
- To use Mesos from Spark, you need a Spark binary package available in a place accessible by Mesos, and a Spark driver program configured to connect to Mesos.
__Uploading  Spark Package:__
- Download a Spark binary package from the Spark download page
- Upload to hdfs/http/s3
__To host on HDFS, use the Hadoop fs put command:__
`hadoop fs -put spark-2.0.1.tar.gz /path/to/spark-2.0.1.tar.gz`
__Using a  Mesos Master URL:__
- The Master URLs for Mesos are in the form `mesos://host:5050` for a single-master Mesos cluster, or `mesos://zk://host1:2181,host2:2181,host3:2181/mesos` for a multi-master Mesos cluster using ZooKeeper.
__Client Mode:__
```Scala
val conf = new SparkConf()
       .setMaster("mesos://HOST:5050")
       .setAppName("My app")
       .set("spark.executor.uri", "<path to spark-2.0.1.tar.gz uploaded above>")
val sc = new SparkContext(conf)
     ./bin/spark-shell --master mesos://host:5050
```

__Cluster Mode:__
``` bash
./bin/spark-submit \
       --class org.apache.spark.examples.SparkPi \
       --master mesos://207.184.161.138:7077 \
       --deploy-mode cluster \
       --supervise \
       --executor-memory 20G \
       --total-executor-cores 100 \
       http://path/to/examples.jar \
       1000 \
```

### Running Spark on YARN 模式：
__Cluster mode:__
```bash
$ ./bin/spark-submit --class path.to.your.Class --master yarn --deploy-mode cluster [options] <app jar> [app options]

$ ./bin/spark-submit --class org.apache.spark.examples.SparkPi \
    --master yarn \
    --deploy-mode cluster \
    --driver-memory 4g \
    --executor-memory 2g \
    --executor-cores 1 \
    --queue thequeue \
    lib/spark-examples*.jar \
    10
```
__Client mode:__
```bash
$ ./bin/spark-shell --master yarn --deploy-mode client
```
