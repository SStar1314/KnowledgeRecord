---
title: HDFS ViewFS (Based on Federation)
tags: Hadoop
categories: bigdata
---

## HDFS ViewFs
The View File System (ViewFs) provides a way to manage multiple Hadoop file system namespaces or namespace volumes.

### How The Clusters Look

Suppose there are multiple cluster, each cluster has one or more namenodes. Each namenode has its own namespace, and a namenode belongs to one and only one cluster.

### A Global Namespace Per Cluster Using ViewFs

ViewFs implements the Hadoop file system interface just like HDFS and the local file system. 代码位于Hadoop HDFS src 工程的 ViewFS继承类。
![](/images/Hadoop_ViewFS_1.png)

```xml
<property>   
  <name>fs.default.name</name>   
  <value>viewfs://clusterX</value>
</property>
```
