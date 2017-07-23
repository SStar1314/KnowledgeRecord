---
title: HDFS Federation(联邦)
tags: Hadoop
categories: bigdata
---

## HDFS Federation(联邦)
在前面的文章介绍过，Hadoop的Federation是将整个文件系统划分为子集，每一个Federation中的NameNode负责管理其中一个子集，整个文件系统由这些子集通过挂载mount的方式构建。 Federation与HA结合使用。

官方doc: https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/Federation.html

### HDFS has two main layers:
__Namespace :__
    1. Consists of directories, files and blocks.
    2. It supports all the namespace related file system operations such as create, delete, modify and list files and directories.            
__Block Storage Services ,  this has two parts :__
    1. Block Management (performed in the Namenode)
      Provides Datanode cluster membership by handling registrations, and periodic heart beats.
      Processes block reports and maintains location of blocks.
      Supports block related operations such as create, delete, modify and get block location.
      Manages replica placement, block replication for under replicated blocks, and deletes blocks that are over replicated.   
    2. Storage   -   is provided by Datanodes by storing blocks on the local file system and allowing read/write access.

### Multiple Namenodes/Namespaces
Federation uses multiple independent Namenodes/namespaces to scale the name service horizontally. __The Namenodes are federated; the Namenodes are independent and do not require coordination with each other.__ The Datanodes are used as common storage for blocks by all the Namespaces. Each Datanode registers with all the Namenodes in the cluster. Datanodes send periodic heartbeats and block reports.
![](/images/HDFS_Fedaration_1.png)

### Federation Configuration
Federation configuration is __backward compatible__ and allows existing single Namenode configuration to work without any change.
__Step 1:__ Add the dfs.nameservices parameter to your configuration and configure it with a list of comma separated NameServiceIDs. This will be used by the Datanodes to determine the Namenodes in the cluster.
__Step 2:__ For each Namenode and Secondary Namenode/BackupNode/Checkpointer add the following configuration parameters suffixed with the corresponding NameServiceID into the common configuration file:
__Namenode__
- dfs.namenode.rpc-address
- dfs.namenode.servicerpc-address
- dfs.namenode.http-address
- dfs.namenode.https-address
- dfs.namenode.keytab.file
- dfs.namenode.name.dir
- dfs.namenode.edits.dir
- dfs.namenode.checkpoint.dir
- dfs.namenode.checkpoint.edits.dir
__Secondary Namenode__
- dfs.namenode.secondary.http-address
- dfs.secondary.namenode.keytab.file
__BackupNode__
- dfs.namenode.backup.address
- dfs.secondary.namenode.keytab.file

#### hdfs-site.xml:
```xml
<configuration>
  <property>
    <name>dfs.nameservices</name>
    <value>ns1,ns2</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.ns1</name>
    <value>nn-host1:rpc-port</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.ns1</name>
    <value>nn-host1:http-port</value>
  </property>
  <property>
    <name>dfs.namenode.secondaryhttp-address.ns1</name>
    <value>snn-host1:http-port</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.ns2</name>
    <value>nn-host2:rpc-port</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.ns2</name>
    <value>nn-host2:http-port</value>
  </property>
  <property>
    <name>dfs.namenode.secondaryhttp-address.ns2</name>
    <value>snn-host2:http-port</value>
  </property>
</configuration>
```

#### Formatting Namenodes

Step 1: Format a Namenode:
```bash
[hdfs]$ $HADOOP_PREFIX/bin/hdfs namenode -format [-clusterId <cluster_id>]
```
Step 2: Format additional Namenodes
```bash
[hdfs]$ $HADOOP_PREFIX/bin/hdfs namenode -format -clusterId <cluster_id>
```
#### Upgrading from an older release and configuring federation
Older releases only support a single Namenode, after Upgrade the cluster to newer release in order to enable federation.
```bash
[hdfs]$ $HADOOP_PREFIX/bin/hdfs start namenode --config $HADOOP_CONF_DIR  -upgrade -clusterId <cluster_ID>
```
#### Adding a new Namenode to an existing HDFS cluster
Perform the following steps:
- Add dfs.nameservices to the configuration.
- Update the configuration with the NameServiceID suffix. Configuration key names changed post release 0.20. You must use the new configuration parameter names in order to use federation.
- Add the new Namenode related config to the configuration file.
- Propagate the configuration file to the all the nodes in the cluster.
- Start the new Namenode and Secondary/Backup.
- Refresh the Datanodes to pickup the newly added Namenode by running the following command against all the Datanodes in the cluster:
```bash
[hdfs]$ $HADOOP_PREFIX/bin/hdfs dfsadmin -refreshNameNodes <datanode_host_name>:<datanode_rpc_port>
```
