---
title: HDFS安装(三种模式)
tags: Hadoop
categories: bigdata
---

## HDFS 分布式文件系统
- __Hadoop Distributed File System (HDFS™)__: A distributed file system that provides high-throughput access to application data.

## Hadoop HDFS 2.x 安装
Hadoop HDFS 2.x 包含了3种安装模式：
1. Standalone. 独立模式
2. Pseudo-Distributed Operation. 伪分布式
3. Cluster. 集群

### Standalone
默认情况下，Hadoop被配置成以非分布式模式运行的一个独立Java进程。
``` bash
     export JAVA_HOME=/usr/local/java/openjdk1.8/
     cd  hadoop_home/
     mkdir input
     cp etc/hadoop/*.xml input
     bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep input output 'dfs[a-z.]+'
     cat output/*
```
### Pseudo-Distributed Operation
Hadoop可以在单节点上以所谓的伪分布式模式运行，此时每一个Hadoop守护进程都作为一个独立的Java进程运行。
#### 配置
__不配置Yarn__
__etc/hadoop/core-site.xml:__
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```
__etc/hadoop/hdfs-site.xml:__
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
</configuration>
```
__Setup passphraseless ssh login:__
``` bash
ssh localhost
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```
----------------------------------------------
#### 伪分布式HDFS初始化及使用命令
The following instructions are to run a MapReduce job locally.
1. Format the filesystem: 初始化!!!
``` bash
bin/hdfs namenode -format
```
2. Start NameNode daemon and DataNode daemon:
```bash
sbin/start-dfs.sh
```
The hadoop daemon log output is written to the $(HADOOP_LOG_DIR) directory (defaults to $(HADOOP_HOME)/logs).
3. Browse the web interface for the NameNode; by default it is available at:
    NameNode - http://localhost:50070/
4. Make the HDFS directories required to execute MapReduce jobs:
```bash
bin/hdfs dfs -mkdir /user
bin/hdfs dfs -mkdir /user/<username>
```
5. Copy the input files into the distributed filesystem:
```bash
bin/hdfs dfs -put etc/hadoop input
```
6. Run some of the examples provided:
```bash
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep input output 'dfs[a-z.]+'
```
7. Examine the output files: Copy the output files from the distributed filesystem to the local filesystem and examine them:
```bash
bin/hdfs dfs -get output output
cat output/*
```
or
View the output files on the distributed filesystem:
```bash
bin/hdfs dfs -cat output/*
```
8. When you’re done, stop the daemons with:
```bash
sbin/stop-dfs.sh
```
#### 配置Yarn
__配置Yarn on a Single Node__
__etc/hadoop/mapred-site.xml__
```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```
__etc/hadoop/yarn-site.xml:__
```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```
1. Start ResourceManager daemon and NodeManager daemon:
```bash
sbin/start-yarn.sh
```
2. Browse the web interface for the ResourceManager; by default it is available at:
ResourceManager - http://localhost:8088/
3. Run a MapReduce job.
与不配置Yarn的HDFS使用命令类似，可参考上面的例子。
4. When you’re done, stop the daemons with:
```bash
sbin/stop-yarn.sh
```

### Cluster Setup
参考：http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html
下载 hadoop2.7.3 版本的压缩包，解压缩到master节点上， 解压路径为 ${Hadoop_Install} .
配置 hadoop cluster 中各个节点之间的passwordless 无密码访问。

#### Configure Hadoop Cluster
到 ${Hadoop_Install}/etc/hadoop/ 目录下 编辑配置文件： __core-site.xml__  __hdfs-site.xml__  __mapred-site.xml__  __yarn-site.xml__ .
__core-site.xml :  configure important parameters__
```xml
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://hadoop-nn:9000</value>
        </property>
        <property>
                <name>dfs.permissions</name>
                <value>false</value>
        </property>
</configuration>
```
__hdfs-site.xml :  configure for NameNode + DataNode__
```xml
<configuration>
        <property>
                <name>dfs.data.dir</name>
                <value>/opt/hadoop/dfs/name/data</value>
                <final>true</final>
        </property>
        <property>
                <name>dfs.name.dir</name>
                <value>/opt/hadoop/dfs/name</value>
                <final>true</final>
        </property>
        <property>
                <name>dfs.blocksize</name>
                <value>10240</value>
        </property>
        <property>
                <name>dfs.replication</name>
                <value>2</value>
        </property>
</configuration>
```
__mapred-site.xml :  Configure for MapReduce Applications + MapReduce JobHistory Server__
```xml
<configuration>
        <property>
                <name>mapred.framework.name</name>
                <value>yarn</value>
        </property>
</configuration>
```
__yarn-site.xml :  Configure for ResourceManager + NodeManager + History Server__
```xml
<configuration>
        <property>
                <name>yarn.resourcemanager.resource-tracker.address</name>
                <value>hadoop-nn:8025</value>
        </property>
        <property>
                <name>yarn.resourcemanager.scheduler.address</name>
                <value>hadoop-nn:8035</value>
        </property>
        <property>
                <name>yarn.resourcemanager.address</name>
                <value>hadoop-nn:8050</value>
        </property>
</configuration>
```

#### Hadoop Cluster Startup
__Format a new distributed filesystem:__
```bash
[hdfs]$ $HADOOP_PREFIX/bin/hdfs namenode -format <cluster_name>
```
会生成一个name文件夹，里面存储fsimage和editlog文件，记录整个cluster中的文件系统。
__Start HDFS NameNode :__
```bash
[hdfs]$ $HADOOP_PREFIX/sbin/hadoop-daemon.sh --config $HADOOP_CONF_DIR --script hdfs start namenode
```
__Start HDFS DataNode :__
```bash
[hdfs]$ $HADOOP_PREFIX/sbin/hadoop-daemons.sh --config $HADOOP_CONF_DIR --script hdfs start datanode
```
__Start all Hadoop slaves * :__
```bash
[hdfs]$ $HADOOP_PREFIX/sbin/start-dfs.sh
```
__Start  Yarn  ResourceManager :__
```bash
[yarn]$ $HADOOP_YARN_HOME/sbin/yarn-daemon.sh --config $HADOOP_CONF_DIR start resourcemanager
```
__Start  Yarn  NodeManager :__
```bash
[yarn]$ $HADOOP_YARN_HOME/sbin/yarn-daemons.sh --config $HADOOP_CONF_DIR start nodemanager
```
__Start  Yarn  WebAppProxy server if necessary:__
```bash
[yarn]$ $HADOOP_YARN_HOME/sbin/yarn-daemon.sh --config $HADOOP_CONF_DIR start proxyserver
```
__Start all  Yarn  slaves *:__
```bash
[yarn]$ $HADOOP_PREFIX/sbin/start-yarn.sh
```
__Start MapReduce JobHistory server :__
```bash
[mapred]$ $HADOOP_PREFIX/sbin/mr-jobhistory-daemon.sh --config $HADOOP_CONF_DIR start historyserver
```

#### Hadoop Cluster Web Interfaces
```
NameNode     http://hadoop_namenode:50070/
ResourceManager     http://hadoop_resourcemanager:8088/
MapReduce JobHistory Server     http://jobhistory_serevr:19888/
```

#### Hadoop Cluster exclude/decommision Datanodes
__configure  hdfs-site.xml :__
```xml
<property>
  <name>dfs.hosts.exclude</name>
  <value>/home/hadoop/hdfs_exclude.txt</value>
  <description>DFS exclude</description>
</property>  
```
Then write the decommission data node(slave2) to __hdfs_exclude.txt__ file.
Last, force configure reload:
```bash
hadoop dfsadmin  -refreshNodes
hadoop dfsadmin  -report
```
