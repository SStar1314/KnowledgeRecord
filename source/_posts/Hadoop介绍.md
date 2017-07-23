---
title: Hadoop介绍
tags: Hadoop
categories: bigdata
---

## Hadoop主要包含以下部分
- __Hadoop Common__: The common utilities that support the other Hadoop modules.
- __Hadoop Distributed File System (HDFS™)__: A distributed file system that provides high-throughput access to application data.
- __Hadoop YARN__: A framework for job scheduling and cluster resource management.
- __Hadoop MapReduce__: A YARN-based system for parallel processing of large data sets.


## Hadoop生态圈
- __Ambari™__: 部署服务. A web-based tool for provisioning, managing, and monitoring Apache Hadoop clusters which includes support for Hadoop HDFS, Hadoop MapReduce, Hive, HCatalog, HBase, ZooKeeper, Oozie, Pig and Sqoop. Ambari also provides a dashboard for viewing cluster health such as heatmaps and ability to view MapReduce, Pig and Hive applications visually alongwith features to diagnose their performance characteristics in a user-friendly manner.
- __Avro™__: 数据序列化. A data serialization system.
- __Cassandra™__: 分布式NoSQL. A scalable multi-master database with no single points of failure.
- __Chukwa™__: 数据收集. A data collection system for managing large distributed systems.
- __HBase™__: 分布式NoSQL. A scalable, distributed database that supports structured data storage for large tables.
- __Hive™__: 数据仓库. A data warehouse infrastructure that provides data summarization and ad hoc querying.
- __Mahout™:__ 数据挖掘. A Scalable machine learning and data mining library.
- __Pig™__: 可转化为MapReduce的数据查询. A high-level data-flow language and execution framework for parallel computation.
- __Spark™__: 内存加速的运算框架. A fast and general compute engine for Hadoop data. Spark provides a simple and expressive programming model that supports a wide range of applications, including ETL, machine learning, stream processing, and graph computation.
- __Tez™__: A generalized data-flow programming framework, built on Hadoop YARN, which provides a powerful and flexible engine to execute an arbitrary DAG of tasks to process data for both batch and interactive use-cases. Tez is being adopted by Hive™, Pig™ and other frameworks in the Hadoop ecosystem, and also by other commercial software (e.g. ETL tools), to replace Hadoop™ MapReduce as the underlying execution engine.
- __ZooKeeper™__: 分布式协同框架. A high-performance coordination service for distributed applications.
