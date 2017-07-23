---
title: ZooKeeper简介
tags: ZooKeeper
categories: 分布式协同
---

## ZooKeeper导图
__ZooKeeper 服务:__
![](/images/ZooKeeper_1.png)

__ZooKeeper 名字空间:__
![](/images/ZooKeeper_2.png)

## 推荐一本书
《ZooKeeper分布式过程协同技术详解》

## 基于 Paxos 算法
wiki: https://en.wikipedia.org/wiki/Paxos_(computer_science)


## ZooKeeper 支持的api
__ZooKeeper API:__
1.create /path data
2.delete /path
3.exists /path
4.setData /path data
5.getData /path
6.getChildren /path

## 单点ZooKeeper 与 ZooKeeper集群
__ZooKeeper服务器的两种工作模式： 独立模式(standalone)和仲裁模式(quorum)__
__独立模式(standalone)__: 单独的ZooKeeper服务器。
__仲裁模式(quorum)__: ZooKeeper集合(ZooKeeper ensemble)。 仲裁模式中，为减少ZooKeeper Server数据同步的延迟，法定人数 的概念被使用，法定人数的大小设置非常重要。

__仲裁模式试验：__
__configure 文件：__
_z1.cfg_
```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/root/zookeeper-3.4.8/conf/z1/data
clientPort=2181
server.1=127.0.0.1:2222:2223
server.2=127.0.0.1:3333:3334
server.3=127.0.0.1:4444:4445
```
_z2.cfg_
```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/root/zookeeper-3.4.8/conf/z2/data
clientPort=2182
server.1=127.0.0.1:2222:2223
server.2=127.0.0.1:3333:3334
server.3=127.0.0.1:4444:4445
```
_z3.cfg_
```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/root/zookeeper-3.4.8/conf/z3/data
clientPort=2183
server.1=127.0.0.1:2222:2223
server.2=127.0.0.1:3333:3334
server.3=127.0.0.1:4444:4445
```
_启动3个节点命令：_
```
/root/zookeeper-3.4.8/bin/zkServer.sh  start  z1/z1.cfg
/root/zookeeper-3.4.8/bin/zkServer.sh  start  z2/z2.cfg
/root/zookeeper-3.4.8/bin/zkServer.sh  start  z3/z3.cfg
```
_连接集群：_
```
/bin/zkCli.sh -server 127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
```
创建 __ephemeral__ 节点：
```
create -e /master "master.example.com"
```
创建 __sequential__ 节点：
```
create -s /master/task-  “cmd"
```
__WatchedEvent__ 数据结构包含以下信息：
```
ZooKeeper会话状态(KeeperState)：Disconnected, SyncConnected, AuthFailed, ConnectedReadOnly, SaslAuthenticated, Expired.
事件类型(EventType)：NodeCreated, NodeDeleted, NodeDataChanged, NodeChildrenChanged, None.
如果事件类型不是None时，返回一个znode路径.
Watch监视点设置：
NodeCreated：通过exists调用设置监视点。
NodeDeleted：通过exists或getData调用设置监视点。
NodeDataChanged：通过exists或getData调用设置监视点。
NodeChildrenChanged：通过getChildren调用设置监视点。
```
