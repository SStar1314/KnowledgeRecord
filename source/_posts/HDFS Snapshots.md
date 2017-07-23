---
title: HDFS Snapshots
tags: Hadoop
categories: bigdata
---

## HDFS Snapshots
Snapshots功能非常重要，可以保证HDFS在出现异常情况时可以进行恢复。 Snapshots可以使用在整个HDFS系统上，也可以只对其中的部分文件目录。

HDFS Snapshots are read-only point-in-time copies of file system. Snapshots can be taken on a subtree of the file system or the entire file system.

The HDFS path should be Snapshottable.
```bash
Snapshots Paths “.snapshot” is used for accessing its snapshots.                hadoop  fs -ls /foo/.snapshot
Allow Snapshots command:                   hdfs  dfsadmin -allowSnapshot <path>
Disallow Snapshots command:             hdfs  dfsadmin  -disallowSnapshot <path>   
Create Snapshots command:                  hdfs dfs -createSnapshot <path> [<snapshotName>]
Delete Snapshots command:                  hdfs dfs -deleteSnapshot <path> [<snapshotName>]
Rename Snapshots command:                  hdfs dfs -renameSnapshot <path> <oldName> <newName>
Get Snapshottable Directory Listing command:           hdfs lsSnapshottableDir
Get Snapshots Difference Report command:                 hdfs snapshotDiff <path> <fromSnapshot> <toSnapshot>
```
