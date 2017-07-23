---
title: Mesos
tags: Mesos
categories: Mesos
---

## Mesos Installation

![](/images/mesos_1.png)

获取Mesos源码或者tar包：
```bash
$ wget http://www.apache.org/dist/mesos/0.28.2/mesos-0.28.2.tar.gz
$ tar -zxf mesos-0.28.2.tar.gz
$ git clone https://git-wip-us.apache.org/repos/asf/mesos.git
```
Mesos 安装（Ubuntu14.04）：
#### Update the packages.
$ sudo apt-get update

#### Install a few utility tools.
$ sudo apt-get install -y tar wget git

#### Install the latest OpenJDK.
$ sudo apt-get install -y openjdk-7-jdk

#### Install autotools (Only necessary if building from git repository).
$ sudo apt-get install -y autoconf libtool

#### Install other Mesos dependencies.
$ sudo apt-get -y install build-essential python-dev libcurl4-nss-dev libsasl2-dev libsasl2-modules maven libapr1-dev libsvn-dev

Building Mesos(如果下载了tar.gz的Mesos包，此步可以被跳过)：

#### Change working directory.
$ cd mesos

#### Bootstrap (Only required if building from git repository).
$ ./bootstrap

#### Configure and build.
$ mkdir build
$ cd build
$ ../configure
$ make

#### Run test suite.
$ make check

#### Install (Optional).
$ make install

### 搭建Mesos Cluster命令：
#### Change into build directory.
$ cd build

#### Start mesos master (Ensure work directory exists and has proper permissions).
$ ./bin/mesos-master.sh —ip=$(ip) --work_dir=/var/lib/mesos

#### Start mesos agent.
$ ./bin/mesos-agent.sh —master=$(master_ip):5050 --work_dir=/var/lib/mesos

#### Visit the mesos web page.
$ http://$(master_ip):5050

#### Run C++ framework (Exits after successfully running some tasks.).
$ ./src/test-framework --master=$(master_ip):5050

#### Run Java framework (Exits after successfully running some tasks.).
$ ./src/examples/java/test-framework $(master_ip):5050

#### Run Python framework (Exits after successfully running some tasks.).
$ ./src/examples/python/test-framework $(master_ip):5050
