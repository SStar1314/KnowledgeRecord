---
title: Hadoop 源码入口
tags: Hadoop
categories: bigdata
---
### improt Hadoop源代码到Eclipse里面：
- 1. mvn clean install -DskipTests
- 2. mvn eclipse:eclipse -DdownloadSources -DdownloadJavadocs

## Hadoop 源码入口
这些是Hadoop 源码阅读的入口位置，可以通过这些main函数看进源码实现。

``` bash
Hadoop command:
    # the core commands
    if [ "$COMMAND" = "fs" ] ; then
      CLASS=org.apache.hadoop.fs.FsShell
    elif [ "$COMMAND" = "version" ] ; then
      CLASS=org.apache.hadoop.util.VersionInfo
    elif [ "$COMMAND" = "jar" ] ; then
      CLASS=org.apache.hadoop.util.RunJar
      if [[ -n "${YARN_OPTS}" ]] || [[ -n "${YARN_CLIENT_OPTS}" ]]; then
        echo "WARNING: Use \"yarn jar\" to launch YARN applications." 1>&2
      fi
    elif [ "$COMMAND" = "key" ] ; then
      CLASS=org.apache.hadoop.crypto.key.KeyShell
    elif [ "$COMMAND" = "checknative" ] ; then
      CLASS=org.apache.hadoop.util.NativeLibraryChecker
    elif [ "$COMMAND" = "distcp" ] ; then
      CLASS=org.apache.hadoop.tools.DistCp
      CLASSPATH=${CLASSPATH}:${TOOL_PATH}
    elif [ "$COMMAND" = "daemonlog" ] ; then
      CLASS=org.apache.hadoop.log.LogLevel
    elif [ "$COMMAND" = "archive" ] ; then
      CLASS=org.apache.hadoop.tools.HadoopArchives
      CLASSPATH=${CLASSPATH}:${TOOL_PATH}
    elif [ "$COMMAND" = "credential" ] ; then
      CLASS=org.apache.hadoop.security.alias.CredentialShell
    elif [ "$COMMAND" = "trace" ] ; then
      CLASS=org.apache.hadoop.tracing.TraceAdmin
    elif [ "$COMMAND" = "classpath" ] ; then
      if [ "$#" -gt 1 ]; then
        CLASS=org.apache.hadoop.util.Classpath
```
``` bash
Hdfs command:
if [ "$COMMAND" = "namenode" ] ; then
  CLASS='org.apache.hadoop.hdfs.server.namenode.NameNode'
  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_NAMENODE_OPTS"
elif [ "$COMMAND" = "zkfc" ] ; then
  CLASS='org.apache.hadoop.hdfs.tools.DFSZKFailoverController'
  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_ZKFC_OPTS"
elif [ "$COMMAND" = "secondarynamenode" ] ; then
  CLASS='org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode'
  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_SECONDARYNAMENODE_OPTS"
elif [ "$COMMAND" = "datanode" ] ; then
  CLASS='org.apache.hadoop.hdfs.server.datanode.DataNode'
  if [ "$starting_secure_dn" = "true" ]; then
    HADOOP_OPTS="$HADOOP_OPTS -jvm server $HADOOP_DATANODE_OPTS"
  else
    HADOOP_OPTS="$HADOOP_OPTS -server $HADOOP_DATANODE_OPTS"
  fi
elif [ "$COMMAND" = "journalnode" ] ; then
  CLASS='org.apache.hadoop.hdfs.qjournal.server.JournalNode'
  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_JOURNALNODE_OPTS"
elif [ "$COMMAND" = "dfs" ] ; then
  CLASS=org.apache.hadoop.fs.FsShell
  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_CLIENT_OPTS"
elif [ "$COMMAND" = "dfsadmin" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.DFSAdmin
  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_CLIENT_OPTS"
elif [ "$COMMAND" = "haadmin" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.DFSHAAdmin
  CLASSPATH=${CLASSPATH}:${TOOL_PATH}
  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_CLIENT_OPTS"
elif [ "$COMMAND" = "fsck" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.DFSck
  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_CLIENT_OPTS"
elif [ "$COMMAND" = "balancer" ] ; then
  CLASS=org.apache.hadoop.hdfs.server.balancer.Balancer
  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_BALANCER_OPTS"
elif [ "$COMMAND" = "mover" ] ; then
  CLASS=org.apache.hadoop.hdfs.server.mover.Mover
  HADOOP_OPTS="${HADOOP_OPTS} ${HADOOP_MOVER_OPTS}"
elif [ "$COMMAND" = "storagepolicies" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.StoragePolicyAdmin
elif [ "$COMMAND" = "jmxget" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.JMXGet
elif [ "$COMMAND" = "oiv" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.offlineImageViewer.OfflineImageViewerPB
elif [ "$COMMAND" = "oiv_legacy" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.offlineImageViewer.OfflineImageViewer
elif [ "$COMMAND" = "oev" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.offlineEditsViewer.OfflineEditsViewer
elif [ "$COMMAND" = "fetchdt" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.DelegationTokenFetcher
elif [ "$COMMAND" = "getconf" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.GetConf
elif [ "$COMMAND" = "groups" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.GetGroups
elif [ "$COMMAND" = "snapshotDiff" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.snapshot.SnapshotDiff
elif [ "$COMMAND" = "lsSnapshottableDir" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.snapshot.LsSnapshottableDir
elif [ "$COMMAND" = "portmap" ] ; then
  CLASS=org.apache.hadoop.portmap.Portmap
  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_PORTMAP_OPTS"
elif [ "$COMMAND" = "nfs3" ] ; then
  CLASS=org.apache.hadoop.hdfs.nfs.nfs3.Nfs3
  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_NFS3_OPTS"
elif [ "$COMMAND" = "cacheadmin" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.CacheAdmin
elif [ "$COMMAND" = "crypto" ] ; then
  CLASS=org.apache.hadoop.hdfs.tools.CryptoAdmin
elif [ "$COMMAND" = "version" ] ; then
  CLASS=org.apache.hadoop.util.VersionInfo
elif [ "$COMMAND" = "debug" ]; then
  CLASS=org.apache.hadoop.hdfs.tools.DebugAdmin
elif [ "$COMMAND" = "classpath" ]; then
  if [ "$#" -gt 0 ]; then
    CLASS=org.apache.hadoop.util.Classpath
```
``` bash
org.apache.hadoop.hdfs.server.datanode.SecureDataNodeStarter
org.apache.hadoop.hdfs.nfs.nfs3.PrivilegedNfsGatewayStarter
```
``` bash
Yarn:
elif [ "$COMMAND" = "rmadmin" ] ; then
  CLASS='org.apache.hadoop.yarn.client.cli.RMAdminCLI'
  YARN_OPTS="$YARN_OPTS $YARN_CLIENT_OPTS"
elif [ "$COMMAND" = "scmadmin" ] ; then
  CLASS='org.apache.hadoop.yarn.client.SCMAdmin'
  YARN_OPTS="$YARN_OPTS $YARN_CLIENT_OPTS"
elif [ "$COMMAND" = "application" ] ||
     [ "$COMMAND" = "applicationattempt" ] ||
     [ "$COMMAND" = "container" ]; then
  CLASS=org.apache.hadoop.yarn.client.cli.ApplicationCLI
  YARN_OPTS="$YARN_OPTS $YARN_CLIENT_OPTS"
  set -- $COMMAND $@
elif [ "$COMMAND" = "node" ] ; then
  CLASS=org.apache.hadoop.yarn.client.cli.NodeCLI
  YARN_OPTS="$YARN_OPTS $YARN_CLIENT_OPTS"
elif [ "$COMMAND" = "queue" ] ; then
  CLASS=org.apache.hadoop.yarn.client.cli.QueueCLI
  YARN_OPTS="$YARN_OPTS $YARN_CLIENT_OPTS"
elif [ "$COMMAND" = "resourcemanager" ] ; then
  CLASSPATH=${CLASSPATH}:$YARN_CONF_DIR/rm-config/log4j.properties
  CLASS='org.apache.hadoop.yarn.server.resourcemanager.ResourceManager’               main     —> serviceInit —> serviceStart
  YARN_OPTS="$YARN_OPTS $YARN_RESOURCEMANAGER_OPTS"
  if [ "$YARN_RESOURCEMANAGER_HEAPSIZE" != "" ]; then
    JAVA_HEAP_MAX="-Xmx""$YARN_RESOURCEMANAGER_HEAPSIZE""m"
  fi
elif [ "$COMMAND" = "historyserver" ] ; then
  echo "DEPRECATED: Use of this command to start the timeline server is deprecated." 1>&2
  echo "Instead use the timelineserver command for it." 1>&2
  CLASSPATH=${CLASSPATH}:$YARN_CONF_DIR/ahs-config/log4j.properties
  CLASS='org.apache.hadoop.yarn.server.applicationhistoryservice.ApplicationHistoryServer'
  YARN_OPTS="$YARN_OPTS $YARN_HISTORYSERVER_OPTS"
  if [ "$YARN_HISTORYSERVER_HEAPSIZE" != "" ]; then
    JAVA_HEAP_MAX="-Xmx""$YARN_HISTORYSERVER_HEAPSIZE""m"
  fi
elif [ "$COMMAND" = "timelineserver" ] ; then
  CLASSPATH=${CLASSPATH}:$YARN_CONF_DIR/timelineserver-config/log4j.properties
  CLASS='org.apache.hadoop.yarn.server.applicationhistoryservice.ApplicationHistoryServer'
  YARN_OPTS="$YARN_OPTS $YARN_TIMELINESERVER_OPTS"
  if [ "$YARN_TIMELINESERVER_HEAPSIZE" != "" ]; then
    JAVA_HEAP_MAX="-Xmx""$YARN_TIMELINESERVER_HEAPSIZE""m"
  fi
elif [ "$COMMAND" = "sharedcachemanager" ] ; then
  CLASSPATH=${CLASSPATH}:$YARN_CONF_DIR/scm-config/log4j.properties
  CLASS='org.apache.hadoop.yarn.server.sharedcachemanager.SharedCacheManager'
  YARN_OPTS="$YARN_OPTS $YARN_SHAREDCACHEMANAGER_OPTS"
  YARN_OPTS="$YARN_OPTS $YARN_SHAREDCACHEMANAGER_OPTS"
  if [ "$YARN_SHAREDCACHEMANAGER_HEAPSIZE" != "" ]; then
    JAVA_HEAP_MAX="-Xmx""$YARN_SHAREDCACHEMANAGER_HEAPSIZE""m"
  fi
elif [ "$COMMAND" = "nodemanager" ] ; then
  CLASSPATH=${CLASSPATH}:$YARN_CONF_DIR/nm-config/log4j.properties
  CLASS='org.apache.hadoop.yarn.server.nodemanager.NodeManager'                      main     —> serviceInit —> serviceStart
  YARN_OPTS="$YARN_OPTS -server $YARN_NODEMANAGER_OPTS"
  if [ "$YARN_NODEMANAGER_HEAPSIZE" != "" ]; then
    JAVA_HEAP_MAX="-Xmx""$YARN_NODEMANAGER_HEAPSIZE""m"
  fi
elif [ "$COMMAND" = "proxyserver" ] ; then
  CLASS='org.apache.hadoop.yarn.server.webproxy.WebAppProxyServer'
  YARN_OPTS="$YARN_OPTS $YARN_PROXYSERVER_OPTS"
  if [ "$YARN_PROXYSERVER_HEAPSIZE" != "" ]; then
    JAVA_HEAP_MAX="-Xmx""$YARN_PROXYSERVER_HEAPSIZE""m"
  fi
elif [ "$COMMAND" = "version" ] ; then
  CLASS=org.apache.hadoop.util.VersionInfo
  YARN_OPTS="$YARN_OPTS $YARN_CLIENT_OPTS"
elif [ "$COMMAND" = "jar" ] ; then
  CLASS=org.apache.hadoop.util.RunJar
  YARN_OPTS="$YARN_OPTS $YARN_CLIENT_OPTS"
elif [ "$COMMAND" = "logs" ] ; then
  CLASS=org.apache.hadoop.yarn.client.cli.LogsCLI
  YARN_OPTS="$YARN_OPTS $YARN_CLIENT_OPTS"
elif [ "$COMMAND" = "daemonlog" ] ; then
  CLASS=org.apache.hadoop.log.LogLevel
  YARN_OPTS="$YARN_OPTS $YARN_CLIENT_OPTS"
elif [ "$COMMAND" = "cluster" ] ; then
  CLASS=org.apache.hadoop.yarn.client.cli.ClusterCLI
  YARN_OPTS="$YARN_OPTS $YARN_CLIENT_OPTS"
else
  CLASS=$COMMAND
fi
```
