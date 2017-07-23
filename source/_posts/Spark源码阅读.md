---
title: Spark源码阅读
tags: Spark
categories: bigdata
---

### Spark 源代码 import 进 Eclipse
参考： https://cwiki.apache.org/confluence/display/SPARK/Useful+Developer+Tools

到Spark源代码目录下，运行：
```bash
mvn -DskipTests clean package
mvn eclipse:eclipse
```

Spark 是由 Scala语言编写，而Scala语言的语法糖太多，对于Scala新手的我来说确实困难，花费时间会较多，先暂时放一放。
