---
title: HDFS Offline Image Viewer (解析fsimage file)
tags: Hadoop
categories: bigdata
---

## HDFS Offline Image Viewer
The Offiline Image Viewer is a tool to dump the contents of hdfs fsimage files.

command :
- Web Processor
Web processer launches a HTTP server :     
```bash
bin/hdfs oiv -i fsimage
```
Users can access the viewer and get information of the fsimage:     
```bash
bin/hdfs dfs -ls webhdfs://127.0.0.1:5978/
```

- XML Processor
XML processer  is used to dump all the contents in the fsimage.
```bash
bin/hdfs oiv -p XML -i fsimage -o fsimage.xml
```
