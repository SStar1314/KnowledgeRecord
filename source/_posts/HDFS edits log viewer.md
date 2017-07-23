---
title: HDFS Offline Edits Viewer (解析edits log 文件)
tags: Hadoop
categories: bigdata
---

## HDFS Offline Edits Viewer
Offline Edits Viewer is a tool to parse the Edits log file.

command:
```bash
hdfs oev -i edits -o edits.xml
```

__Hadoop cluster recovery:__ This can be done by converting the binary edits to XML, edit it manually and then convert it back to binary.
 
