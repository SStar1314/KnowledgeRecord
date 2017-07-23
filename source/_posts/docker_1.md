---
title: Docker
tags: Docker
categories: 容器技术
---

## Docker

### Docker 内核特性
容器 ＝ Cgroup ＋ Namespace ＋ rootfs ＋ 容器引擎

__Docker 内核特性：__
1. Namespace：访问隔离。
2. Cgroup：资源控制。
3. rootfs：文件系统隔离。
4. 容器引擎：生命周期控制。

#### Cgroup 实现子系统：
1.devices：设备权限控制。
2.cpuset：分配制定的CPU和内存节点。
3.cpu：控制CPU占用率。
4.cpuacct：统计CPU使用情况。
5.memory：限制内存的使用上限。
6.freezer：冻结（暂停）Cgroup中的进程。
7.net_cls：配合tc（traffic controller）限制网络带宽。
8.net_prio：设置进程的网络流量优先级。
9.huge_tlb：限制HugeTLB的使用。
10.perf_event：运行Perf工具基于Cgroup分组做性能检测。

#### Linux内核中的6种Namespace：
1.IPC：隔离System V IPC和POSIX消息队列。
2.Network：隔离网络资源。
3.Mount：隔离文件系统挂载点。
4.PID：隔离进程ID。
5.UTS：隔离主机名和域名。
6.User：隔离用户ID和组ID。

### Docker 镜像命令
Docker镜像：
```bash
docker images
docker images --help, docker images --filter
docker search image-name
dockviz images -t    需要先调用：alias dockviz="docker run --rm -v /var/run/docker.sock:/var/run/docker.sock nate/dockviz"
docker pull image-name     下载镜像
docker save -o image-name.tar image-name   导出镜像
docker load -i image-name.tar      导入镜像
docker inspect image-name          查看容器和镜像详细信息
docker commit           增量式的生产镜像
docker history image-name           查看image layer纪录
docker push           上传镜像   （docker push localhost:5000/official/ubuntu:14.04）
```

从image repositories中查看所有images信息： cat /var/lib/docker/image/aufs/repositories.json  | python -m json.tool
启动Docker daemon，测试image layer之间的关系：docker daemon -D -s overlay -g /var/lib/docker/

### 部署docker私有仓库：
```bash
docker run -d --hostname localhost --name registry-v2 -v /opt/data/distribution:/var/lib/registry/docker/registry/v2 -p 5000:5000 registry:2.0
```

```bash
向私有仓库push image：
docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
docker push NAME[:TAG]
docker tag 8dbd9e392a96 localhost.localdomain:5000/ubuntu
docker push localhost.localdomain:5000/ubuntu
docker pull localhost:5000/ubuntu
docker stop registry && docker rm -v registry
验证私有仓库：curl -X GET http://localhost:5000/v2/
查找镜像： curl -X GET http://localhost:5000/v2/foo/bar/tags/list  
     例如：  curl -X GET http://localhost:5000/v2/busybox/tags/list
            {"name":"busybox","tags":["latest"]}
     通过tags查找manifest： curl -X GET http://localhost:5000/v2/busybox/manifests/latest
创建带Auth的私有仓库：
docker run -d -p 5000:5000 --restart=always --name registry
     \ -v `pwd`/auth:/auth
     \-e"REGISTRY_AUTH=htpasswd"
     \-e"REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm"
     \-eREGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
     \ -v `pwd`/certs:/certs
     \-eREGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt
     \-eREGISTRY_HTTP_TLS_KEY=/certs/domain.key
     \ registry:2
登录registry：docker login myregistrydomain.com:5000
registry yml文件内容：
registry:
     restart:always
     image:registry:2
     ports:
          -5000:5000
     environment:
          REGISTRY_HTTP_TLS_CERTIFICATE:/certs/domain.crt
          REGISTRY_HTTP_TLS_KEY:/certs/domain.key
          REGISTRY_AUTH:htpasswd
          REGISTRY_AUTH_HTPASSWD_PATH:/auth/htpasswd
          REGISTRY_AUTH_HTPASSWD_REALM:Registry Realm
     volumes:
          -/path/data:/var/lib/registry
          -/path/certs:/certs
          -/path/auth:/auth
docker-compose up -d
```

### Build my own Docker image:
```bash
1.Write a Dockerfile:
FROM docker/whalesay:latest
RUN apt-get -y update && apt-get install -y fortunes
CMD /usr/games/fortune -a | cowsay
2.Build an image from my Dockerfile:
docker build -t docker-whale .
3.Run new docker-whale:
docker images
docker run docker-whale
```
