---
title: Maven
tags: Maven
categories: Java
---

### 使用maven创建一个project：
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
groupId 指: project名称，及生成的war包名称。
artifactId 指: 不携带version的war包名称。

Build project：
```bash
mvn package
```

### Running Maven Tools:
Maven Phases:
* validate: validate the project is correct and all necessary information is available
* compile: compile the source code of the project
* test: test the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed
* package: take the compiled code and package it in its distributable format, such as a JAR.
* integration-test: process and deploy the package if necessary into an environment where integration tests can be run
* verify: run any checks to verify the package is valid and meets quality criteria
* install: install the package into the local repository, for use as a dependency in other projects locally
* deploy: done in an integration or release environment, copies the final package to the remote repository for sharing with other developers and projects.

There are two other Maven lifecycles of note beyond the default list above
* clean: cleans up artifacts created by prior builds
* site: generates site documentation for this project

Detail usage: http://maven.apache.org/guides/getting-started/index.html
