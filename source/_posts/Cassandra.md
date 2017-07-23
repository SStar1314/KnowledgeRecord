---
title: Cassandra 数据库
tags: Cassandra
categories: NoSQL
---
### Cassandra 数据库
Apache Cassandra是一套开源分布式NoSQL数据库系统。它最初由Facebook开发，用于储存收件箱等简单格式数据，集Google BigTable的数据模型与AmazonDynamo的完全分布式架构于一身。Facebook于2008将 Cassandra 开源，此后，由于Cassandra良好的可扩展性和性能，被Apple, Comcast,Instagram, Spotify, eBay, Rackspace, Netflix等知名网站所采用，成为了一种流行的分布式结构化数据存储方案。

在数据库排行榜“DB-Engines Ranking”中，Cassandra排在第七位，是非关系型数据库中排名第二高的（仅次于MongoDB）。

### 数据模型
Cassandra使用了Google 设计的 BigTable的数据模型，Cassandra使用的是宽列存储模型(Wide Column Stores)，每行数据由row key唯一标识之后，可以有最多20亿个列，每个列由一个column key标识，每个column key下对应若干value。这种模型可以理解为是一个二维的key-value存储，即整个数据模型被定义成一个类似 map< key1, map< key2,value>>的类型。

### 存储模型
与BigTable和其模仿者HBase不同，Cassandra的数据并不存储在分布式文件系统如GFS或HDFS中，而是直接存于本地。与BigTable一样，Cassandra也是日志型数据库，即把新写入的数据存储在内存的Memtable中并通过磁盘中的CommitLog来做持久化，内存填满后将数据按照key的顺序写进一个只读文件SSTable中，每次读取数据时将所有SSTable和内存中的数据进行查找和合并。这种系统的特点是写入比读取更快，因为写入一条数据是顺序计入commit log中，不需要随机读取磁盘以及搜索。

### 与类似开源系统的比较

HBase是Apache Hadoop项目的一个子项目，是Google BigTable的一个克隆，与Cassandra一样，它们都使用了BigTable的列族式的数据模型，但是：

- Cassandra只有一种节点，而HBase有多种不同角色，除了处理读写请求的region server之外，其架构在一套完整的HDFS分布式文件系统之上，并需要ZooKeeper来同步集群状态，部署上Cassandra更简单。
- Cassandra的数据一致性策略是可配置的，可选择是强一致性还是性能更高的最终一致性；而HBase总是强一致性的。
- Cassandra通过一致性哈希来决定一行数据存储在哪些节点，靠概率上的平均来实现负载均衡；而HBase每段数据(region)只有一个节点负责处理，由master来动态分配一个region是否大到需要拆分成两个，同时会将过热的节点上的一些region动态的分配给负载较低的节点，因此实现动态的负载均衡。
- 因为每个region同时只能有一个节点处理，一旦这个节点无响应，在系统将这个节点的所有region转移到其他节点之前这些数据便无法读写，加上master也只有一个节点，备用master的恢复也需要时间，因此HBase在一定程度上有单点问题；而Cassandra无单点问题。
- Cassandra的读写性能优于HBase。

### Cassandra 服务启动
- 1.启动服务
```bash
./cassandra   org.apache.cassandra.service.CassandraDaemon
```
- 2.启动用户交互              实际启动cqlsh.py来进行交互
```bash
./cqlsh          
```
Cassandra Shell 命令：  http://www.w3ii.com/cassandra/cassandra_shell_commands.html
```bash
./cassandra -f           前台启动 cassandra
```
- 3.停止服务
```bash
kill  pid   
```
### Cql 使用
__创建使用Cqlsh一个密钥空间__
http://www.w3ii.com/cassandra/cassandra_create_keyspace.html
```sql
CREATE KEYSPACE tutorialspoint
WITH replication = {'class':'SimpleStrategy', 'replication_factor' : 3};
DESCRIBE keyspaces;
```
__使用Java创建API密钥空间一__
http://www.w3ii.com/cassandra/cassandra_create_keyspace.html

__改变使用Cqlsh KEYSPACE__
http://www.w3ii.com/cassandra/cassandra_alter_keyspace.html
```bash
ALTER KEYSPACE "KeySpace Name" WITH replication = {'class': 'Strategy name', 'replication_factor' : 'No.Of  replicas'};
ALTER KEYSPACE tutorialspoint WITH replication = {'class':'SimpleStrategy', 'replication_factor' : 2};
```
__测试密钥空间的durable_writes属性__
```bash
cqlsh> SELECT * FROM system_schema.keyspaces;
ALTER KEYSPACE test
WITH REPLICATION = {'class' : 'NetworkTopologyStrategy', 'datacenter1' : 3}
AND DURABLE_WRITES = true;
```
__删除使用Cqlsh一个密钥空间__
```bash
DROP KEYSPACE tutorialspoint;
```
__Cassandra创建表__
```bash
USE tutorialspoint;
cqlsh:tutorialspoint>; CREATE TABLE emp(
   emp_id int PRIMARY KEY,
   emp_name text,
   emp_city text,
   emp_sal varint,
   emp_phone varint
   );
cqlsh:tutorialspoint> select * from emp;
```
__Cassandra修改表__
```bash
cqlsh:tutorialspoint> ALTER TABLE emp
   ... ADD emp_email text;
```
__Cassandra删除表__
```bash
cqlsh:tutorialspoint> DROP TABLE emp;
cqlsh:tutorialspoint> DESCRIBE COLUMNFAMILIES;
```
__Cassandra截断表__
```bash
cqlsh:tp> TRUNCATE student;
```
__Cassandra创建索引__
```bash
cqlsh:tutorialspoint> CREATE INDEX name ON emp1 (emp_name);
```
Cassandra DROP INDEX
```bash
cqlsh:tp> drop index name;
```
__Cassandra批量__
```bash
cqlsh:tutorialspoint> BEGIN BATCH
     INSERT INTO emp (emp_id, emp_city, emp_name, emp_phone, emp_sal) values(  4,'Pune','rajeev',9848022331, 30000);
     UPDATE emp SET emp_sal = 50000 WHERE emp_id =3;
     DELETE emp_city FROM emp WHERE emp_id = 2;
     APPLY BATCH;
```
__Cassandra创建数据__
```bash
cqlsh:tutorialspoint> INSERT INTO emp (emp_id, emp_name, emp_city,
   emp_phone, emp_sal) VALUES(1,'ram', 'Hyderabad', 9848022338, 50000);
cqlsh:tutorialspoint> INSERT INTO emp (emp_id, emp_name, emp_city,
   emp_phone, emp_sal) VALUES(2,'robin', 'Hyderabad', 9848022339, 40000);
cqlsh:tutorialspoint> INSERT INTO emp (emp_id, emp_name, emp_city,
   emp_phone, emp_sal) VALUES(3,'rahman', 'Chennai', 9848022330, 45000);
```
__Cassandra更新数据__
```bash
cqlsh:tutorialspoint> UPDATE emp SET emp_city='Delhi',emp_sal=50000
   WHERE emp_id=2;
```
__Cassandra删除数据__
```bash
cqlsh:tutorialspoint> DELETE emp_sal FROM emp WHERE emp_id=3;
```
__Cassandra CQL集合__
```bash
cqlsh:tutorialspoint> CREATE TABLE data(name text PRIMARY KEY, email list<text>);
cqlsh:tutorialspoint> INSERT INTO data(name, email) VALUES ('ramu', ['abc@gmail.com','cba@yahoo.com'])
cqlsh:tutorialspoint> UPDATE data SET email = email +['xyz@w3ii.com'] where name = 'ramu';

cqlsh:tutorialspoint> CREATE TABLE data2 (name text PRIMARY KEY, phone set<varint>);

cqlsh:tutorialspoint> INSERT INTO data2(name, phone)VALUES ('rahman',    {9848022338,9848022339});

cqlsh:tutorialspoint> UPDATE data2 SET phone = phone + {9848022330} where name = 'rahman';

cqlsh:tutorialspoint> CREATE TABLE data3 (name text PRIMARY KEY, address map<timestamp, text>);

cqlsh:tutorialspoint> INSERT INTO data3 (name, address) VALUES ('robin', {'home' : 'hyderabad' , 'office' : 'Delhi' } );

cqlsh:tutorialspoint> UPDATE data3 SET address = address+{'office':'mumbai'} WHERE name = 'robin';
```

### Cassandra 安装 及 源代码分析
__service 启动入口函数：__
```bash
org.apache.cassandra.service.CassandraDaemon
```
__installation:__
http://cassandra.apache.org/doc/latest/getting_started/installing.html

__import源代码进Eclipse：__
- 1.ant build
- 2.ant generate-eclipse-files

还有其它操作：
- 执行ant avro-generate
- 执行ant gen-thrift-java
- 执行ant generate-eclipse-files
