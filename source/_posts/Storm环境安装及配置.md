---
title: Storm(流计算)环境安装及配置
tags: 大数据
categories: bigdata
---

### Storm 集群安装和配置
__Summary of the steps for setting up a Storm cluster:__
- Set up a Zookeeper cluster
- Install dependencies on Nimbus and worker machines
- Download and extract a Storm release to Nimbus and worker machines
- Fill in mandatory configurations into storm.yaml
- Launch daemons under supervision using "storm" script and a supervisor of your choice

__配置：__
![](/images/Storm_Config.png)

__源代码入口：__
```bash
bin/storm nimbus                                   org.apache.storm.daemon.nimbus
bin/storm supervisor                              org.apache.storm.daemon.supervisor
bin/storm ui                                             org.apache.storm.ui.core
```
__运行测试案例：__
```bash
bin/storm   jar   examples/storm-starter/storm-starter-topologies-1.0.2.jar    org.apache.storm.starter.WordCountTopology
```
__nimbus 与 supervisor之间通过zookeeper 进行通信__
```bash
[zk: localhost:2181(CONNECTED) 9] ls /storm
[backpressure, workerbeats, nimbuses, supervisors, errors, logconfigs, storms, assignments, leader-lock, blobstore]
```

### Integration With External Systems, and Other Libraries

- Flux Data Driven Topology Builder
- Apache Kafka Integration
- Apache HBase Integration
- Apache HDFS Integration
- Apache Hive Integration
- JDBC Integration
- Redis Integration
- Event Hubs Intergration
- Kestrel Intergration
