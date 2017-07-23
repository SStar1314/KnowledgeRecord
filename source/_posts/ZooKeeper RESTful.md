---
title: ZooKeeper RESTful
tags: ZooKeeper
categories: 分布式协同
---

### Enable ZooKeeper RESTful service：
``` bash
     vim  rest.properties
     cd   /root/zookeeper-3.4.8/src/contrib/rest
     nohup ant run &    
     curl -H'Accept: application/json' http://localhost:9998/znodes/v1/                    json
     curl -H'Accept: application/xml' http://localhost:9998/znodes/v1/                    xml
```
### RESTful 用法：
#get children of the root node
```bash
curl http://localhost:9998/znodes/v1/?view=children
```
#get "/cluster1/leader" as xml (default is json)
```bash
curl -H'Accept: application/xml' http://localhost:9998/znodes/v1/cluster1/leader
```
#get the data as text
```bash
curl -w "\n%{http_code}\n" "http://localhost:9998/znodes/v1/cluster1/leader?dataformat=utf8"
```
#set a node (data.txt contains the ascii text you want to set on the node)
```bash
curl -T data.txt -w "\n%{http_code}\n" "http://localhost:9998/znodes/v1/cluster1/leader?dataformat=utf8"
```
#create a node
```bash
curl -d "data1" -H'Content-Type: application/octet-stream' -w "\n%{http_code}\n" "http://localhost:9998/znodes/v1/?op=create&name=cluster2&dataformat=utf8"
```
```bash
curl -d "data2" -H'Content-Type: application/octet-stream' -w "\n%{http_code}\n" "http://localhost:9998/znodes/v1/cluster2?op=create&name=leader&dataformat=utf8"
```
#create a new session
```bash
curl -d "" -H'Content-Type: application/octet-stream' -w "\n%{http_code}\n" "http://localhost:9998/sessions/v1/?op=create&expire=10"
```
#session heartbeat
```bash
curl -X "PUT" -H'Content-Type: application/octet-stream' -w "\n%{http_code}\n" "http://localhost:9998/sessions/v1/02dfdcc8-8667-4e53-a6f8-ca5c2b495a72"
```
#delete a session
```bash
curl -X "DELETE" -H'Content-Type: application/octet-stream' -w "\n%{http_code}\n" "http://localhost:9998/sessions/v1/02dfdcc8-8667-4e53-a6f8-ca5c2b495a72"
```

## service 启动命令
__RestAPI源码入门：__
RestAPI 入口main函数所在文件： __org.apache.zookeeper.server.jersey.RestMain__

__ZooKeeper Source Code 解析：__
1.zkServer 脚本启动命令：
```bash
ZOOMAIN="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=$JMXLOCALONLY org.apache.zookeeper.server.quorum.QuorumPeerMain"

nohup "$JAVA" "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
        -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=7778 \                         
        -cp "$CLASSPATH" $JVMFLAGS $ZOOMAIN "$ZOOCFG" > "$_ZOO_DAEMON_OUT" 2>&1 < /dev/null &

```
2.zkCli 脚本启动命令：
```bash
"$JAVA" "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
     -cp "$CLASSPATH" $CLIENT_JVMFLAGS $JVMFLAGS \
     org.apache.zookeeper.ZooKeeperMain "$@"
```
由上可知， ZooKeeper的server启动入口函数为 __QuorumPeerMain__ ，而client的启动入口函数为 __ZooKeeperMain__。
